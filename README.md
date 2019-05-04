# Nexus使用

## 一、简介

在内网环境下，基于容器管理系统Kubernetes引入DevOps。本文介绍的是使用社区版nexus为Jenkins自动构建Docker镜像提供依赖下载的私有仓库。



## 二、私有仓库使用

### 1、maven

#### 上传

将本地的第三方依赖库批量上传至仓库，修改本地maven的`setting.xml`配置文件。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">     
    <localRepository>localpath</localRepository>
	<proxies>
         <proxy>
             <id>optional</id>
             <active>true</active>
             <protocol>http</protocol>
             <username>XXXXX</username>
             <password>XXXXX</password>
             <host>XXX.XXX.XXX.XXX</host>
             <port>XXX</port>
             <nonProxyHosts>XXX|nexus.XXX.com</nonProxyHosts>
          </proxy>
    </proxies>
    <server>
        <id>maven-releases</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</settings>
```

参考[批量上传 Jar 包到 Maven 私服的工具](https://blog.csdn.net/isea533/article/details/77197017)，为`maven`目录下的java代码配置运行参数，运行`UploadJAR.java`,将本地仓库的第三方依赖包上传至私有仓库。

> 由于使用`settings.xml`指向的本地仓库运行`UploadJAR.java`时，会出现`Cannot deploy artifact from the local repository`的错误。建议首先为项目新建一个本地仓库；然后修改`settings.xml`，指向这个本地仓库；编译项目，将需要的依赖下到本地，然后将本地仓库修改为原来的地址。最后修改`UploadJAR.java`代码中的main函数中的`deploy(new File("F:\\.m2\\repository").listFiles());`，将其指向新建本地仓库的路径，运行代码。

#### 下载

修改本地maven的`setting.xml`配置文件。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <proxies>
         <proxy>
             <id>optional</id>
             <active>true</active>
             <protocol>http</protocol>
             <username>XXXXX</username>
             <password>XXXXX</password>
             <host>XXX.XXX.XXX.XXX</host>
             <port>XXX</port>
             <nonProxyHosts>XXX|nexus.XXX.com</nonProxyHosts>
          </proxy>
    </proxies>
    <mirrors>
		<mirror>
			<id>nexus</id>
			<mirrorOf>*</mirrorOf>
			<name>CorePro Nexus</name>
            <url>http://nexus.XXX.com/repository/maven-public/</url>
        </mirror>
	</mirrors>
    <profile>
      <id>nexus</id>
      <repositories> 
        <repository> 
            <id>nexus</id> 
            <name>local private nexus</name> 
            <url>http://nexus.XXX.com/repository/maven-public/</url> 
            <releases><enabled>true</enabled><updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy></releases> 
            <snapshots><enabled>false</enabled></snapshots> 
        </repository>        
      </repositories> 
      <pluginRepositories> 
        <pluginRepository> 
            <id>nexus</id> 
            <name>local private nexus</name> 
            <url>http://nexus.XXX.com/repository/maven-public/</url> 
            <releases><enabled>true</enabled><updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy></releases> 
            <snapshots><enabled>false</enabled></snapshots> 
        </pluginRepository>        
       </pluginRepositories> 
    </profile>
    <activeProfiles>
		<activeProfile>nexus</activeProfile>
	</activeProfiles>
</settings> 
```

可为maven而配置多个仓库源，但注意`mirrors`节点的`mirror`填写顺序，访问的顺序按填写顺序来。即保证优先使用私有仓库，然后中央仓库的国内代理。

### 2、PyPI

#### 上传

python项目的需要在[PyPI](https://pypi.org/)找到依赖包的编译文件或源码压缩文件到本地`pip-package`文件夹中。

安装python的twine包 

```powershell
$ pip install twine
```

上传本地依赖包至私有仓库

```powershell
$ twine upload --repository-url <nexus-repository-url>  pip-package/*
```

#### 下载

linux环境下，修改~/.pip/pip.conf

```powershell
[global]
index-url = nexus.XXX.com/repository/pypi-group/simple
[install]
trusted-host=nexus
```

### 3、yum

需要宿主机可以连接外网。其他配置参考: [nexus3配置Yum源](https://www.cnblogs.com/ethanw97m/p/9970565.html)

### 4、apt

插件安装参考: [Nexus repository APT plugin](https://github.com/sonatype-nexus-community/nexus-repository-apt)

需要宿主机可以连接外网。其他配置参考: [Ubuntu修改apt-get源](https://www.cnblogs.com/TechSnail/p/7754969.html)

需要注意的是选用Docker镜像的系统版本，不同的系统版本需要使用不同的apt-get源。

### 5、helm

插件安装参考: [Nexus Repository Helm Format](https://github.com/sonatype-nexus-community/nexus-repository-helm)

注意：http://nexus.XXX.com/repository/helm-hosted/index.yaml下的文件格式

```yaml
apiVersion: '1.0'
entries:
  mongodb:
  - created: 2019-03-22T07:25:08.528Z
    description: NoSQL document-oriented database that stores JSON-like documents
      with dynamic schemas, simplifying the integration of data in content-driven
      applications.
    digest: d30920c594809a38bc9cb06d240b390159c638f0a014c426118c802385000ab0
    icon: https://bitnami.com/assets/stacks/mongodb/img/mongodb-stack-220x234.png
    maintainers:
    - email: containers@bitnami.com
      name: Bitnami
    name: mongodb
    sources:
    - https://github.com/bitnami/bitnami-docker-mongodb
    urls:
    - mongodb-5.2.0.tgz
    version: 5.2.0
generated: 2019-04-27T03:38:09.147Z
```

`urls`对应的路径为相对路径，与helm的Chart的`urls`绝对路径不同。可能会导致如python-helm或golang-helm等个语言客户端无法下载helm安装包。

## 三、docker配置

#### 背景知识简介

容器中的DNS名称解析优先级顺序为：

- 内置DNS服务器127.0.0.11。
- 通过--dns等参数为容器配置的DNS服务器。
- docker守护进程的--dns服务配置（默认为8.8.8.8和8.8.4.4）
- 宿主机上的DNS设置。

 当我们在build镜像的时候，通过修改/etc/docker/daemon.json文件来实现DNS参数设置。

```json
{
    "dns":["xx.xx.xx.xx"]
}
```

#### docker构建改造

在Dockerfile中，会出现例如

```dockerfile
FROM XXXX

RUN apt-get update -y  && apt-get  install vim -y
RUN yum update -y  && yum install vim -y
RUN pip install redis
```

将Dockerfile使用的基础镜像做上述相应改造后，还需要为Docker deamon做DNS解析，让docker在构建镜像时能解析内网仓库地址`nexus.xxx.com` 。

修改/etc/docker/daemon.json文件

```
{
    "dns":["nexus.xxx.com"]
}
```

## 四、其他依赖

本文只列举了几个nexus常用仓库的使用方法，其他插件的使用可参考[官网](https://www.sonatype.com/download-oss-sonatype)介绍。

## 参考文档

- [Maven私服Nexus 3.x搭建](https://www.jianshu.com/p/1898f29ce1ca)
- [使用 Nexus 搭建 PyPi 私服及上传](https://blog.csdn.net/m0_37607365/article/details/79998955)
- [Daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)

