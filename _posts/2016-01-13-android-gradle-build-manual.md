---
layout: post
title:  Android gradle build简明使用教程
date:   2016-01-13 16:05:32
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---

# Android gradle build简明使用教程

## 签名
gradle本身支持直接签名，只需要在build.gradle添加如下代码即可

```
signingConfigs {
        Neusoft {
            storeFile file("/Users/liuzuo/Documents/Resource/KeyStore/key_neusoft_si/neusoft.si.keystore")
            storePassword "BK3AGmHAmUzRsfo4S6sO"
            keyAlias "neusoft.si"
            keyPassword "BK3AGmHAmUzRsfo4S6sO"
        }
    }
    
buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.Neusoft
        }
        debug {
            
        }    
    }
```

一般填上上面的代码即可执行签名，但是这种方式不太安全，建议不要在build.gradle文件中写上签名文件的密码，因为build.gradle文件一般都会集成到代码的版本控制中，这样所有人都会有签名文件的密码。

所以应该把签名文件的密码隔离起来，写到一个配置文件中，此配置文件不包含在代码版本控制中，这样其他开发者就不会知道签名文件的密码。

gradle配置文件一般以.properties结束，我们先新建一个signing.properties文件，内容如下：  
```
STORE_FILE=yourapp.keystore
STORE_PASSWORD=your password
KEY_ALIAS=your alias
KEY_PASSWORD=your password
```
__注意没有字符串双引号""__

接下在guild.gradle文件中读取signing.properties配置文件，读取的代码如下：   
```
File propFile = file('signing.properties');
    if (propFile.exists()) {
        def Properties props = new Properties()
        props.load(new FileInputStream(propFile))
        if (props.containsKey('STORE_FILE') && props.containsKey('STORE_PASSWORD') &&
                props.containsKey('KEY_ALIAS') && props.containsKey('KEY_PASSWORD')) {
            android.signingConfigs.Neusoft.storeFile = file(props['STORE_FILE'])
            android.signingConfigs.Neusoft.storePassword = props['STORE_PASSWORD']
            android.signingConfigs.Neusoft.keyAlias = props['KEY_ALIAS']
            android.signingConfigs.Neusoft.keyPassword = props['KEY_PASSWORD']
        } else {
            android.buildTypes.Neusoft.signingConfig = null
        }
    } else {
        android.buildTypes.Neusoft.signingConfig = null
    }
```
代码很简单，就是读取文件，然后拿到签名需要的四个变量值分别赋值即可。

## 多渠道打包
由于国内Android市场众多渠道，为了统计每个渠道的下载及其它数据统计，就需要我们针对每个渠道单独打包。
gradle的多渠道打包很简单，因为gradle已经帮我们做好了很多基础功能。

下面以友盟统计为例说明，一般友盟统计在AndroidManifest.xml里面会有这么一段声明：
```
<meta-data
    android:name="UMENG_CHANNEL"
    android:value="CHANNEL_ID" />
```
其中CHANNEL_ID就是友盟的渠道标示，多渠道的实现一般就是通过修改CHANNEL_ID值来实现的。   
__接下来将一步一步来实现多渠道版本打包。__

1.在AndroidManifest.xml里配置PlaceHolder,用与在build.gradle文件中来替换成自己想要设置的值   
```
<meta-data
    android:name="UMENG_CHANNEL"
    android:value="${UMENG_CHANNEL_VALUE}" />
```
2.在build.gradle设置productFlavors，修改PlaceHolder的值
```
productFlavors {
        guanfang {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "guanfang"]
        }
    }
```
如果渠道较多可以批量修改   
```
productFlavors {
        guanfang {}
        miui {}
        wandoujia {}
}
 //批量处理
productFlavors.all { 
       flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name] 
}
```

按照上面两步即可编译打多渠道包了，命令是 ./gradlew assembleRelease，可以打包所有的多渠道包。   
如果只是想打单渠道包，则执行相应的task即可，如gradle assembleGuanfangStoreRelease就是打guanfang渠道的Release版本。  


## buildConfigField自定义配置
经常可能会遇到下面这种情况，就是Debug版本服务器和Release版本服务器通常不在一台服务器上，而测试希望可以同时发布两个服务器的版本用于测试，这个时候我们就需要修改代码，然后一个一个老老实实的发包。gradle提供buildConfigField配合多渠道打不同服务器版本的方法。
其实用法很简单,首先在相应的节点加上定义，比如：
```
buildTypes {
        release {

            buildConfigField "boolean", "SI_DEBUG", "false"

        }
        debug {

            buildConfigField "boolean", "SI_DEBUG", "true"
            
        }
    }
```

然后在代码中通过BuildConfig.SI_DEBUG调用即可。


## 一个完整的gradle脚本
下面给出沈阳医保项目完整的gradle的脚本，供大家参考：
```
apply plugin: 'com.android.application'

android {

    /**
     * 请在本文件同级目录下创建signing.properties文件，
     * signning.properties文件内容如下：
     * STORE_FILE=yourapp.keystore
     * STORE_PASSWORD=your password
     * KEY_ALIAS=your alias
     * KEY_PASSWORD=your password
     **/
    signingConfigs {
        Neusoft {

        }
    }

    File propFile = file('signing.properties');
    if (propFile.exists()) {
        def Properties props = new Properties()
        props.load(new FileInputStream(propFile))
        if (props.containsKey('STORE_FILE') && props.containsKey('STORE_PASSWORD') &&
                props.containsKey('KEY_ALIAS') && props.containsKey('KEY_PASSWORD')) {
            android.signingConfigs.Neusoft.storeFile = file(props['STORE_FILE'])
            android.signingConfigs.Neusoft.storePassword = props['STORE_PASSWORD']
            android.signingConfigs.Neusoft.keyAlias = props['KEY_ALIAS']
            android.signingConfigs.Neusoft.keyPassword = props['KEY_PASSWORD']
        } else {
            android.buildTypes.Neusoft.signingConfig = null
        }
    } else {
        android.buildTypes.Neusoft.signingConfig = null
    }



    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        applicationId "com.neusoft.syybgh"
        minSdkVersion 14
        targetSdkVersion 22
        versionCode 2
        versionName "1.0.1"

    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.Neusoft

            buildConfigField "boolean", "SI_DEBUG", "false"

            manifestPlaceholders = [BAIDU_MAP_API_KEY_VALUE:"heCqmgsv1TtcGLywRlpf1zUs",
                                    UMENG_APPKEY_VALUE:"56847f1667e58eb722004c51"]

        }
        debug {
            //signingConfig signingConfigs.Neusoft

            buildConfigField "boolean", "SI_DEBUG", "true"

            manifestPlaceholders = [BAIDU_MAP_API_KEY_VALUE:"heCqmgsv1TtcGLywRlpf1zUs",
                                    UMENG_APPKEY_VALUE:"56847f1667e58eb722004c51"]
        }
    }


    productFlavors{
        guanfang {}
    }

    //批量处理
    productFlavors.all{ flavor ->
        flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]

    }

    //关闭因第三方代码使用过时API导致编译停止
    lintOptions {
        abortOnError false
    }
    packagingOptions {
        exclude 'META-INF/DEPENDENCIES.txt'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/notice.txt'
        exclude 'META-INF/license.txt'
        exclude 'META-INF/dependencies.txt'
        exclude 'META-INF/LGPL2.1'
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])

    releaseCompile 'com.neusoft.si.android:base-core:1.0.2@aar'
    debugCompile 'com.neusoft.si.android:base-core-debug:1.0.2@aar'
    releaseCompile 'com.neusoft.si.android:base-ui-umeng:1.0.1@aar'
    debugCompile 'com.neusoft.si.android:base-ui-umeng-debug:1.0.1@aar'
    releaseCompile 'com.neusoft.si.android:base-net:1.1.2@aar'
    debugCompile 'com.neusoft.si.android:base-net-debug:1.1.2@aar'
    releaseCompile 'com.neusoft.si.android:base-database:1.0.1@aar'
    debugCompile 'com.neusoft.si.android:base-database-debug:1.0.1@aar'
    releaseCompile 'com.neusoft.si.android:base-update:1.0.2@aar'
    debugCompile 'com.neusoft.si.android:base-update-debug:1.0.2@aar'

    compile 'com.android.support:support-v4:22.2.1'
    compile 'com.j256.ormlite:ormlite-core:4.48'
    compile 'com.j256.ormlite:ormlite-android:4.48'
    compile 'com.fasterxml.jackson.core:jackson-core:2.6.1'
    compile 'com.fasterxml.jackson.core:jackson-annotations:2.6.1'
    compile 'com.fasterxml.jackson.core:jackson-databind:2.6.1'
    compile 'com.squareup.okio:okio:1.5.0'
    compile 'com.squareup.okhttp:okhttp:2.4.0'
    compile 'com.squareup.retrofit:retrofit:1.9.0'
    compile 'com.squareup.retrofit:converter-jackson:1.9.0'
    compile 'com.umeng.analytics:analytics:5.6.4'
}
```

好了，就是这么多了。
