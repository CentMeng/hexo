layout: android
title: studio配置AndroidAnnotations
date: 2016-07-23 11:32:15
categories:
- Android技术
tags:
- Android Studio
---


<img src="/img/data/androidstudio.png" />

- 修改全局gradle文件

```

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.2'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files

        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```
 - 修改app级gradle文件

```
apply plugin: 'com.android.application'
apply plugin: 'android-apt'
def AAVersion = '3.3.2'

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
    apt "org.androidannotations:androidannotations:$AAVersion"
    compile "org.androidannotations:androidannotations-api:$AAVersion"
}


apt {
    arguments {
        androidManifestFile variant.outputs[0].processResources.manifestFile
        // resourcePackageName "your.package.name"
        resourcePackageName 'com.luoteng.voicerecord'
    }
}

```

通过如下配置我们就可以使用android annotations了