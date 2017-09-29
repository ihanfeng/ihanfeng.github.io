---
title: How to install gradle on windows 10
date: 2017-09-29 22:00:00
tags:
  - Gradle
categories:
  - Gradle
---
# Gradle介绍

Gradle是一个基于JVM的构建工具，它提供了：

* 像Ant一样，通用灵活的构建工具
* 可以切换的，基于约定的构建框架
* 强大的多工程构建支持
* 基于Apache Ivy的强大的依赖管理
* 支持maven, Ivy仓库
* 支持传递性依赖管理，而不需要远程仓库或者是pom.xml和ivy.xml配置文件。
* 对Ant的任务做了很好的集成
* 基于Groovy，build脚本使用Groovy编写
* 有广泛的领域模型支持构建


# Gradle 概述
1. 基于声明和基于约定的构建。
2. 依赖型的编程语言。
3. 可以结构化构建，易于维护和理解。
4. 有高级的API允许你在构建执行的整个过程当中，对它的核心进行监视，或者是配置它的行为。
5. 有良好的扩展性。有增量构建功能来克服性能瓶颈问题。
6. 多项目构建的支持。
7. 多种方式的依赖管理。
8. 是第一个构建集成工具。集成了Ant, maven的功能。
9. 易于移值。
10. 脚本采用Groovy编写，易于维护。
11. 通过Gradle Wrapper允许你在没有安装Gradle的机器上进行Gradle构建。
12. 自由，开源。

# Gradle 安装

1. JDK安装 推荐JDK8
2. Gradle官网 [链接](https://gradle.org/) 下载地址[链接](http://services.gradle.org/distributions/) 目前最新版本 V4.2
3. 下载 [gradle-4.2-all.zip](http://services.gradle.org/distributions/gradle-4.2-all.zip)，解压到自定义路径下。
4. 配置环境变量。配置GRADLE_HOME到你的gradle根目录当中，然后把%GRADLE_HOME%/bin（linux或mac的是$GRADLE_HOME/bin）加到PATH的环境变量。
5. 配置完成之后，运行gradle -v，检查一下是否安装无误。如果安装正确，它会打印出Gradle的版本信息，包括它的构建信息，Groovy, Ant, Ivy, 当前JVM和当前系统的版本信息。

# Gradle仓库配置

## 仓库类型

* Maven central repository	这是Maven的中央仓库，无需配置，直接声明就可以使用。但不支持https协议访问
* Maven JCenter repository	JCenter中央仓库，实际也是是用的maven搭建的，但相比Maven仓库更友好，通过CDN分发，并且支持https访问。
* Maven local repository	Maven本地的仓库，可以通过本地配置文件进行配置
* Maven repository	常规的第三方Maven仓库，可设置访问Url
* Ivy repository	Ivy仓库，可以是本地仓库，也可以是远程仓库
* Flat directory repository	使用本地文件夹作为仓库

## 使用方法

1. Maven central repository

  在build.gradle中配置
  ``` groovy
  repositories {
      mavenCentral()
  }
  ```
  就可以直接使用了。

2. Maven JCenter repository
  ``` groovy
  repositories {
      jcenter()
  }
  ```
  这时使用jcenter仓库是通过https访问的，如果想切换成http协议访问，需要修改配置：
  ``` groovy
  repositories {
      jcenter {
          url "http://jcenter.bintray.com"
      }
  }
  ```

3. Local Maven repository
  可以使用Maven本地的仓库。默认情况下，本地仓库位于USER_HOME/.m2/repository（例如windows环境中，在C:\Users\NAME.m2.repository），同时可以通过USER_HOME/.m2/下的settings.xml配置文件修改默认路径位置。
  若使用本地仓库在build.gradle中进行如下配置：
  ``` groovy
  repositories {
      mavenLocal()
  }
  ```
4. Maven repositories

  第三方的配置也很简单，直接指明url即可：
  ``` groovy
  repositories {
      maven {
          url "http://repo.mycompany.com/maven2"
      }
  }
  ```

5. Ivy repository

  配置如下：
  ``` groovy
  repositories {
      ivy {
          url "http://repo.mycompany.com/repo"
      }
  }
  ```

6. Flat directory repository

  使用本地文件夹，这个也比较常用。直接在build.gradle中声明文件夹路径：
  ``` groovy
  repositories {
      flatDir {
          dirs 'lib'
      }
      flatDir {
          dirs 'lib1', 'lib2'
      }
  }
  ```

  使用本地文件夹时，就不支持配置元数据格式的信息了（POM文件）。并且Gradle会优先使用服务器仓库中的库文件：例如同时声明了jcenter和flatDir，当flatDir中的库文件同样在jcenter中存在，gradle会优先使用jcenter的。

## 使用阿里镜像
在 USER_HOME/.gradle/ 下面创建新文件 init.gradle，输入下面的内容并保存。
```
allprojects{
    repositories {
        def REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public/'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $REPOSITORY_URL."
                    remove repo
                }
            }
        }
        maven {
            url REPOSITORY_URL
        }
    }
}
```

## 修改本地仓库的位置

设置Grdle环境变量,需要重启系统生效
```
RADLE_USER_HOME=d:/gradle-4.2/repo/
```
