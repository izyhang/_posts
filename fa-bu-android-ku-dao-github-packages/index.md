---
layout: post
title: 发布Android库到GitHub Packages
date: 2020-03-09 14:13:10
updated: 2020-03-09 14:13:10
tags:
  - libraries
  - gradle
  - github packages
categories: Android
---

原文：[https://proandroiddev.com/publishing-android-libraries-to-the-github-package-registry-part-1-7997be54ea5a](https://proandroiddev.com/publishing-android-libraries-to-the-github-package-registry-part-1-7997be54ea5a)

<!-- More -->

`GitHub`推出了自己的包管理服务，目前开源项目免费使用，提供`npm`、`gem`、`mvn`、`gradle`、`docker`、`dotnet`，更多信息可以在这里找到 [About GitHub Packages](https://help.github.com/en/packages/publishing-and-managing-packages/about-github-packages)

## Part 1 - 发布

### Step 1 - 生成 GitHub Personal Access Token

- 在你的 GitHub 账户里
- 进入 Settings -> Developer Settings -> Personal Access Tokens -> Generate new token
- 勾选`write:packages`和`read:packages`生成一个 Token
  ![](1.jpeg)
- 生成 Token 后记得复制保存，该 Token 只显示在这一次，之后只能重新生成新的 Token

### Step 2 - 配置 Token 到你的项目

- 在你的项目根目录新建一个`.properties`文件，比如`github.properties`
- 添加两个属性`gpr.usr=GITHUB_USERID` `gpr.key=PERSONAL_ACCESS_TOKEN`
- `GITHUB_USERID`替换成你的 GitHub 账户 Id，`PERSONAL_ACCESS_TOKEN`替换成 Step 1 生成的 Token
- 如果你的项目是开源项目，记得将`.properties`文件添加到`.gitignore`，以免泄露 Token

> 你也可以通过环境变量来传入`GPR_USER`和`GPR_API_KEY`，就不需要`.properties`文件

### Step 3 - 更新 library module 的 build.gradle

- 将下方代码添加到 library module 的 build.gradle 文件

```groovy
apply plugin: 'maven-publish' // 添加到library build.gradle的最上面

def githubProperties = new Properties()
githubProperties.load(new FileInputStream(rootProject.file("github.properties")))

def getVersionName = { ->
 return "1.0.2" // 仓库版本
}

def getArtificatId = { ->
 return "sampleAndroidLib" // 仓库Id
}

publishing {
 publications {
   bar(MavenPublication) {
      groupId 'com.enefce.libraries' // 仓库组Id
      artifactId getArtificatId()
      version getVersionName()
      artifact("$buildDir/outputs/aar/${getArtificatId()}-release.aar")
    }
 }

repositories {
        maven {
               name = "GitHubPackages"
        /** Configure path of your package repository on Github
         ** GITHUB_USERID替换成你的GitHub账户Id
         ** REPOSITORY替换成你的项目名称
        */
 url = uri("https://maven.pkg.github.com/GITHUB_USERID/REPOSITORY")
credentials {
        /** Create github.properties in root project folder file with
         ** gpr.usr=GITHUB_USER_ID & gpr.key=PERSONAL_ACCESS_TOKEN
         ** Set env variable GPR_USER & GPR_API_KEY if not adding a properties file**/

         username = githubProperties['gpr.usr'] ?: System.getenv("GPR_USER")
         password = githubProperties['gpr.key'] ?: System.getenv("GPR_API_KEY")
      }
    }
  }
}
```

### Step 4 - 执行发布

> 在执行发布之前需要确保编译过，也就是`build`命令，并能在`build/outputs/aar/`路径找到编译后的 aar 文件

- 在你的 library module 任务下找到`publishing/publish`并执行，或者直接命令行

```shell
$ gradle publish
```

- 当任务成功后就可以在你的 GitHub 账户下的`Packages`栏找到该包
  ![](2.jpeg)
- 如果任务失败，可以跟`–stacktrace`、`–info`或者`-debug`一起执行查看日志

> 注意：

## Part 2 - 安装

> 如果是私有项目还需要生成一个`read:packages`的 Token，生成步骤见 Part 1

### Step 1 - 更新项目的根 build.gradle

- 将下方代码添加到根 build.gradle 文件

```groovy
allprojects {
    repositories {
        maven {
          name = "GitHubPackages"
          /**
          ** GITHUB_USERID替换成刚才发布包的GitHub账号Id
          ** REPOSITORY替换成刚才发布包的项目名称
          */
          url = uri("https://maven.pkg.github.com/GITHUB_USERID/REPOSITORY")
        }
    }
}
```

- 添加依赖到 app.module 文件

```groovy
dependencies {
  implementation 'com.example:package'
}
```

- 同步

## Part 3 - 便携发布

```groovy
apply from: 'https://raw.githubusercontent.com/izyhang/GitHub-Packages-Publish/master/publish.gradle'
```
