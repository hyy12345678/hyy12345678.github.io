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
        android.signingConfigs.release.storeFile = file(props['STORE_FILE'])
        android.signingConfigs.release.storePassword = props['STORE_PASSWORD']
        android.signingConfigs.release.keyAlias = props['KEY_ALIAS']
        android.signingConfigs.release.keyPassword = props['KEY_PASSWORD']
    } else {
        android.buildTypes.release.signingConfig = null
    }
} else {
    android.buildTypes.release.signingConfig = null
}
```

