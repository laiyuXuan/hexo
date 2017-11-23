---
title: maven 笔记
date: 2016-01-16 15:58:31
tags: maven
categories: 其他
---

### 生成项目骨架
```python
mvn archetype:generate 
```
> 自动生成符合maven规定的项目框架结构，即：
>> 主代码在 src/main/java目录下；
>> 测试代码在src/main/test目录下；
>> 资源文件在src/main/resources目录下；
>> 在根目录下的pom.xml文件

这些都是在超级pom文件内配置好的，因为每个pom文件都默认继承一个超级pom文件，故默认生成的目录都会按照这个格式。遵照这个约定的好处就是可以较少许多配置，不用手动配置各个路径。（约定优于配置理念）
超级pom的路径在 $MAVEN_HOME/lib/maven-model-builder-x.x.x.jar 中的org/apache/maven/model/pom-4.0.0.xml

### 依赖范围
![依赖配置中scope标签各值的范围](http://7xrmhj.com1.z0.glb.clouddn.com/maven-scope.png)

### 依赖调解
> * 路径最短者优先原则
> * 第一声明者优先原则
e.g : 存在依赖关系A operation-web->trade-service->autopart-service.1.0 和 B operation-web -> autoparts-service.2.0，此时operation-web取的应该是autoparts-service.2.0。
```python
mvn dependency:list
```

>以上命令可以列出所有已解析的依赖，当存在多个依赖时可用于检查是否解析到正确的依赖。

### 镜像
> 修改settings.xml中的mirror标签，可以配置相关的镜像仓库。配合镜像的一些特殊配置，可以用来搭建maven私服。
> e.g: 
```xml
<!--匹配所有仓库，通常用来搭建私服，构建区域网内的中央仓库-->
<mirrorOf>*</mirrorOf> 
<!--匹配不在本机上的仓库-->
<mirrorOf>external:*</mirrorOf> 
<!--匹配repo1和repo2-->
<mirrorOf>repo1,repo2</mirrorOf> 
<!--匹配所有仓库，除了repo1-->
<mirrorOf>*, !repo1</mirrorOf> 
```

### 生命周期
* clean生命周期：pre-clean, clean, post-clean
* default生命周期：validate, initialize, ... complie, ... test-compile, ... test, ... install, deploy(部分阶段被省略)
* site生命周期：pre-site, site, post-site, site-deploy

### 绑定插件与生命周期
通常生命周期的每个阶段都会与一个插件的目标进行，e.g: default-clean阶段 默认与maven-complier-plugin的complie进行绑定。没有绑定目标的阶段将不会有实际性的操作。常见的生命周期，maven都会默认绑定了插件。此外，还可以自定义绑定。
e.g:
```xml
<build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-source-plugin</artifactId>
                    <version>2.1.1</version>
                    <executions>
	                    <execution>
		                    <id>attah-source</id>
		                    <phase>verify</phase>
		                    <goals>
			                    <goal>jar-no-fork</goal>
		                    </goals>
	                    </execution>
                    </executions>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
```

#### 裁剪反应堆
反应堆指在一个多模块的maven项目中，所有模块组成的构建结构。

可以通过以下命令构建指定的模块：
* `-am --also-make`  构建所列模块和依赖模块
* `-amd --also-make-dependents` 同时构建依赖于构建模块的模块
* `- pl --projects` 构建指定模块，多个模块用逗号隔开
* `- rf - resume-from` 从指定模块恢复反应堆

