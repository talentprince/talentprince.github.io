---
layout: post
title: "Robolectric in Android Studio"
date: 2014-12-15 15:07:06 +0800
comments: true
categories: Android
description: Robolectric in Android Studio
keywords: AndroidStudio,robolectric,test,android,unit test
tag: [android, robolectric]
---

Robolectric
-----------

Robolectric is a unit test framework that de-fangs the Android SDK jar so you can test-drive the development of your Android app. Tests run inside the JVM on your workstation in seconds. With Robolectric you can write tests like this:

```java
  // Test class for MyActivity @RunWith(RobolectricTestRunner.class)
  public class MyActivityTest {
 
    @Test   public void clickingButton_shouldChangeResultsViewText()
  throws Exception {
    Activity activity = Robolectric.buildActivity(MyActivity.class).create().get();
      Button pressMeButton = (Button) activity.findViewById(R.id.press_me_button);
      TextView results = (TextView) activity.findViewById(R.id.results_text_view);
  
      pressMeButton.performClick();
      String resultsText = results.getText().toString();
      assertThat(resultsText, equalTo("Testing Android Rocks!"));   } }
```

Robolectric makes this possible by rewriting Android SDK classes as they’re being loaded and making it possible for them to run on a regular JVM. 

----------

<!--more-->

Setup in Android Studio
-----------------------

#### Step 1: 

读取sdk所在目录, 通过local.properties   

```
  repositories {  
  def androidHome = System.getenv("ANDROID_HOME")  
   if(androidHome == null) {   
     Properties props = new Properties()    
    props.load(new FileInputStream("${projectDir.getParent()}/local.properties"))    
      androidHome = props.getProperty("sdk.dir")    
   }   
   maven {    
       url "$androidHome/extras/android/m2repository/"    
   }   
  }
```

#### Step 2: 

设置robolectric的配置文件,指定res与manifest所在目录  

```
  manifest: xxxxxxx   
  resourceDir: xxxxx  
```

```
  sourceSets {  
  test {  
      // Fix for running tests through AndroidStudio   
      // Make sure our resources are on our classpath    
      output.dir(output.resourcesDir, builtBy: "processTestResources")   
    }   
  }   
```

#### Step 3 
配置Debug属性

>  增加Gradle任务并设置JUnit运行目录, 添加增加的Gradle任务在Make之前,进行编译
>  cmd + , 进入工程配置
![Gradle Set](/images/gradle_set.png)
![Junit Set](/images/junit_set.png)


------------

如果不想通过配置文件指定位置,可以通过重写方法,写死路径来指定Android项目的Manifest与Res位置
如:


```java
    public class ApplicationTestRunner extends RobolectricTestRunner {

        //Maximun SDK Robolectric will compile (issues with SDK > 18)
        private static final int MAX_SDK_SUPPORTED_BY_ROBOLECTRIC = 18;

        private static final String ANDROID_MANIFEST_PATH = "../app/build/intermediates/manifests/full/unittest/debug/AndroidManifest.xml";
        private static final String ANDROID_MANIFEST_RES_PATH = "../app/build/intermediates/res/unittest/debug";

        /**
         * Call this constructor to specify the location of resources and AndroidManifest.xml.
         *
         * @throws org.junit.runners.model.InitializationError
         */
        public ApplicationTestRunner(Class<?> testClass) throws InitializationError {
            super(testClass);
        }

        @Override
        protected AndroidManifest getAppManifest(Config config) {
            return new AndroidManifest(Fs.fileFromPath(ANDROID_MANIFEST_PATH),
                    Fs.fileFromPath(ANDROID_MANIFEST_RES_PATH)) {
                @Override
                public int getTargetSdkVersion() {
                    return MAX_SDK_SUPPORTED_BY_ROBOLECTRIC;
                }
            };
        }
    }
```
 
----------

All done


> 参考
> http://robolectric.org/


 


