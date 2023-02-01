---
title: 使用Anchor来简化YAML
date: 2023-02-01 13:20:57
tags: [ops, yaml]
categories: Operation
description: YAML Achors
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1675229367/yaml_vqy0ae.webp
---

通过配置文件来对我们的系统进行配置在任何系统里都是一个常见的操作, 我们平时最普遍使用的文件格式便是[YAML](https://en.wikipedia.org/wiki/YAML). 很多时候, 我们发现一些通用的配置在不同场景中得来回重复的使用, 或者只有个别字段发生了变化. 在这种情况下, 我们其实可以使用YAML的`Anchor`(&)跟`Alias`(*)来复用我们的配置.

<!--more-->

首先我们可以使用Anchor来标记我们期望复用的配置, 我们知道YAML的结构其实也是一种树的结构, 那么Anchor标记的就可以是树里面任意树枝下的整个叶子结点.

比如:

```
definitions:
  steps:
    - step: &build-test
        name: Build and Test
        script: 
          - ./gradlew build
        artifacts:
          - build/**
```

我们将`step`这个树枝标记为一个Anchor `&build-test`, 那么它就代表了整个`step`下的所有叶子节点, 包括`script`与`artifects`.

### Anchor与Alias

如果在其他的定义过程中, 想复用这个step的配置, 那么就变得简单了, 我们只需要使用`Alias`来引用即可.

```
pipelines:
  branches:
    dev:
    - step: *build-test
    main:
    - step: *build-test
```

这样就很容易实现同样配置的复用, 你的Anchor跟Alias可以定义到一个文件, 也可分开定义, 通过一些小trick再讲他们整合, 比如:

```
cat anchors.yaml pipeline.yaml | buildkite-agent pipeline upload
```

```
cat anchors.yaml config.yaml | > full_config.yaml
```

### 复用与复写

在编程语言中, 我们经常会提到这两个词, 当然YAML已经可以复用了, 那当然也可以复写喽.

如果我们在复用的过程中需要改变一些叶子节点的配置, 对其进行重新定义, 那么可以使用`<<`来进行.

比如:

```
- step: 
  << *build-test
  name: Buid & Test on main
```

如果想要通过复写来完全清除掉某个复用的配置, 我们可以使用`~`.

比如:

```
    - step: 
      << *build-test
      artifacts: ~
```

虽然配置文件复制起来更容易, 但是何不尝试一下复用与复写来简化之呢 ...