---
title: 如何写一个自己的Kubernetes Operator
date: 2025-10-23 15:33:48
categories: DevOps
comments: true
description: 通过自定义kubenetes operator来处理你的CRD
keywords: kubernetes operator k8s controller
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1761278193/operator_yevakj.jpg
---

随着Kubernetes越发成熟并且被各大公司所应用，云原生(Cloud Native)这个名词也成为一个大家口头时常提起的名词，不能说没有k8s就没有云原生的今天，毕竟在这之前就已经有了云原生的所谓理念。但是显而易见的是，是k8s推动了云原生从理念到实现, 无论是微服务，容器化，编排，自恢复，IaC还是CI/CD，GitOps，Observability，我们今天所倡导的一切最佳实践，在k8s的世界里，可能就是一行命令，一个标签，一组配置。

<!--more-->

当然如何更好的将这些配置统筹起来，统一管理，降低维护的难度，将所谓的DevOps的工作分配给每一个Developer，让整个团队敏捷起来，我们必须得使用Custom Resource Definition，而使用了CRD我们就得开发对应的Operator或者说是Controller。还记得上古时期我写过一篇文章，叫做[如何自定义Kubernetes资源](https://talentprince.github.io/2020/12/14/Custom-your-own-k8s-resource/), 如果你还不知道为何我们要使用CRD，那可以在这个文章找到答案。不过当时这个文章是通过官方的Custom Controller Sample来实现的，多少有一些繁琐，那现在最佳的实践是通过kubebuild的一些简单命令就可以做到，大大降低了Boilerplate代码的手动维护，极大的提升了生产力，也使我们更有动力去维护自己的CRD来让Ops工作变的很简单。

如果你看了之前的文章，还想不到这玩意有什么用，那还可以去了解一下[AWS ACK](https://aws.amazon.com/blogs/containers/aws-controllers-for-kubernetes-ack/)是怎么工作的。不用Terraform，不用Pulumi，不用CDK，像管理你的Service App一样，在k8s Cluster里就能管理你的Infrastructure，做到IaC。

好了言归正传，让我看看如何来实现一个自己的k8s Operator吧。

### 准备阶段

首先需要Install Kubebuilder, 以Mac为例，亦可以参考官方[Quick Start](https://book.kubebuilder.io/quick-start)

```
curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)
chmod +x kubebuilder && sudo mv kubebuilder /usr/local/bin/
```

### 生成阶段

接着就可以创建Operator的工程了

```
mkdir my-operator && cd my-operator
kubebuilder init --domain=mydomain.com --repo=mydomain.com/my-operator
```

继续添加要自定义的CRD，没错，这玩意竟然也可以自动生成。

```
kubebuilder create api --group apps --version v1alpha1 --kind App
```

你自己需要做的就是，定义你的CRD里面的字段。比如在`api/v1alpha1/app_types.go`定义俩个。

```
type AppSpec struct {
    // Image to deploy
    Image string `json:"image"`
    // Number of replicas
    Replicas *int32 `json:"replicas"`
}

type AppStatus struct {
    AvailableReplicas int32 `json:"availableReplicas"`
}
```

然后轻轻的敲个命令就CRD就生成好了。

```
make generate
make manifests
```

没想到吧，这才没几秒钟，恭喜你，你已经获得了一个完整的Operator以及自己期望定义的CRD全套资源了。

### 实现阶段

其实这一切才刚刚开始，因为这个Controller还是空空如也，需要我们根据CRD的字段，来实现我们的业务逻辑。

拿我们这次定义的CRD来说，我们想简化k8s App的部署，只需要你提供docker image的名字与节点个数，就可以完成Deploy，所以我们就需要从CRD里面拿到字段的指，来映射到具体的资源里，比如这里我们需要Deployment。大家甚至可以大胆的想想，如果映射到AWS ACK上，那你是不是就可以封装自己的CRD来更简单的部署符合自己公司需要的AWS Infrastructure了。

不过这次我们还是先试试简单的，来创建一个k8s Deployment，在`controllers/app_controller.go`里填入代码。

```
func (r *AppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := log.FromContext(ctx)
    
    var app appsv1alpha1.App
    if err := r.Get(ctx, req.NamespacedName, &app); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // 根据App的字段定义Deployment
    deploy := &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name:      app.Name,
            Namespace: app.Namespace,
        },
        Spec: appsv1.DeploymentSpec{
            Replicas: app.Spec.Replicas,
            Selector: &metav1.LabelSelector{
                MatchLabels: map[string]string{"app": app.Name},
            },
            Template: corev1.PodTemplateSpec{
                ObjectMeta: metav1.ObjectMeta{
                    Labels: map[string]string{"app": app.Name},
                },
                Spec: corev1.PodSpec{
                    Containers: []corev1.Container{
                        {
                            Name:  app.Name,
                            Image: app.Spec.Image,
                        },
                    },
                },
            },
        },
    }

    // 设置Deploment的Owner是App，这样App被删除，Deployment也一并消失,当然如果Deployment状态更新也会触发App的Reconcile
    if err := ctrl.SetControllerReference(&app, deploy, r.Scheme); err != nil {
        return ctrl.Result{}, err
    }

    // 查看当前是否已经有Deployment资源，如果没找到就返回创建
    var existing appsv1.Deployment
    err := r.Get(ctx, types.NamespacedName{Name: deploy.Name, Namespace: deploy.Namespace}, &existing)
    if err != nil && apierrors.IsNotFound(err) {
        log.Info("Creating Deployment", "name", deploy.Name)
        return ctrl.Result{}, r.Create(ctx, deploy)
    } else if err != nil {
        return ctrl.Result{}, err
    }

    // 如果Replicas变了就更新
    if *existing.Spec.Replicas != *app.Spec.Replicas {
        existing.Spec.Replicas = app.Spec.Replicas
        return ctrl.Result{}, r.Update(ctx, &existing)
    }

    // Deployment状态更新，App Reconsile，同步更新App的状态，这个可以在Describe里看到
    app.Status.AvailableReplicas = existing.Status.AvailableReplicas
    if err := r.Status().Update(ctx, &app); err != nil {
        log.Error(err, "Failed to update App status")
        return ctrl.Result{}, err
    }

    return ctrl.Result{}, nil
}
```

还需要一些import

```
import (
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	types "k8s.io/apimachinery/pkg/types"
    appsv1 "k8s.io/api/apps/v1"
    corev1 "k8s.io/api/core/v1"
    apierrors "k8s.io/apimachinery/pkg/api/errors"
)
```

### 测试

```
make install
```
这一步需要你本地有k8s环境，它会将你的CRD部署到集群。

```
make run
```
这一步会将你的controller，也就是我们说的operator本地起起来。

在`config/samples/apps_v1alpha1_app.yaml`添加我们CRD字段的值用来测试，这里我们可以用nginx来试试，当然也可以用个其他小的镜像，主要是为了下载的快。

```
apiVersion: apps.mydomain.com/v1alpha1
kind: App
metadata:
  name: example-app
spec:
  image: nginx:1.27.2
  replicas: 2

```

随后手动apply到k8s集群，通过get pods就可以查看我们的App产生的Deployment(nginx)了。

```
kubectl apply -f config/samples/apps_v1alpha1_app.yaml
```
```
kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
app-sample-544c8d4f86-lw49d   1/1     Running   0          15m
app-sample-544c8d4f86-vx8t2   1/1     Running   0          15m
```

### 拓展

当然这一些都只是一个简单的operator demo，在实际的应用过程中，我们可能会创建不同的operator来实现不同的业务，譬如有Infrastructure的Operator，GitOps的Operator，权限管理的Operator。

CRD的背后往往也会更复杂一些，不只有一个孤零零的Depolyment，还需要有其他一些列的资源来配合，这样才能体现CRD的优势，一换多。这时一个最佳的实践就是根据类型不同的Controller处理不同的资源。

如:

```
internal/controllers/
    ├── deployment_controller.go  
    ├── service_account_controller.go     
    ├── service_controller.go    
    ├── gateway_controller.go 
    └── ..._controller.go  
```

### 写在最后

可见，通过kubebuild已经可以快速创建一个operator的模版，但是还是需要根据实际业务以及需求还定义符合需求的CRD，才能真正的提升我们的云原生治理能力。二话不说，[源码](https://github.com/talentprince/kubernetes-operator)奉上仅供参考。