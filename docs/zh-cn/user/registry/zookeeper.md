# ZooKeeper注册中心

------

ZooKeeper是Seata组件中重要的注册中心实现。

## 预备工作

------

当您将ZooKeeper整合到您的Seata工程前，请确保已经启动ZooKeeper服务，并且项目能正常连接上ZooKeeper服务。如果您还不太清除ZooKeeper的基本使用的话，可先行参考相关
[ZooKeeper快速入门](https://cloud.spring.io/spring-cloud-zookeeper/reference/html/#spring-cloud-zookeeper-install)

。建议使用ZooKeeper 3.4.0及以上的版本。

## 快速上手

------

Seata 融合 ZooKeeper 注册中心的操作步骤非常简单，大致步骤可分为“增加 Maven 依赖”以及“配置注册中心“。

## 增加Maven依赖

------

首先，您需要将 Spring Cloud  的 Maven 依赖添加到您的项目 `pom.xml` 文件中，建议使用 Seata `1.4.1+`，版本对应关系请参考[版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/版本说明)

```pom
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>最新版</version>
</dependency>
<!-- spring cloud 相关依赖 -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-dependencies</artifactId>
	<!-- 此处版本采用你项目中的版本号，建议采用新版本 -->
	<version>${spring-cloud.version}</version>
</dependency>

<!-- zookeeper 此处版本号只是举例示意，实际项目应该遵循对应依赖版本-->
<dependency>
	<groupId>org.apache.zookeeper</groupId>
	<artifactId>zookeeper</artifactId>
	<version>3.6.2</version>
</dependency>
<!-- curator -->
<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-framework</artifactId>
	<version>4.3.0</version>
</dependency>
<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-recipes</artifactId>
	<version>4.3.0</version>
</dependency>
<dependency>
	<groupId>com.101tec</groupId>
	<artifactId>zkclient</artifactId>
	<version>0.11</version>
</dependency>
```

## 配置注册中心

------

### Seata服务端配置

在registry.conf中加入对应配置中心，其余[配置参考](https://cloud.spring.io/spring-cloud-zookeeper/reference/html/#spring-cloud-zookeeper-install)

```conf
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "zk"
  ...
  zk {
    cluster = "default"
    # zk单结点和多结点
    serverAddr = "127.0.0.1:2181"
    # serverAddr = "192.168.0.109:12181,192.168.0.109:22181,192.168.0.109:32181"
    sessionTimeout = 6000
    connectTimeout = 3000
    username = "root"
    password = "123456"
  }
  ...
}
```



#### Seata客户端配置

方法一：引入registry.conf文件, 该文件和服务端配置一样，其余[配置参考](https://cloud.spring.io/spring-cloud-zookeeper/reference/html/#spring-cloud-zookeeper-install)

```
registry {
  # file 、nacos 、eureka、redis、zk
  type = "zk"
  ...	
  zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    # serverAddr = "192.168.0.109:12181,192.168.0.109:22181,192.168.0.109:32181"
    session.timeout = 6000
    connect.timeout = 2000
  }
  ...
}
```

方法二：不引入registry文件，在application.properties中加入对应的配置中心。
需要在/registry/zk/demo（分布式项目有多模块的，demo为模块名）下新建一个key为seata服务端ip+端口的键值对(可使用zkui进行操作)，例如key为127.0.0.1:8091，其余[配置参考](https://cloud.spring.io/spring-cloud-zookeeper/reference/html/#spring-cloud-zookeeper-install)。

```pom
# seata分布式事务自动配置启用标志
seata.enabled=true
seata.enable-auto-data-source-proxy=true

# zk基本配置属性
seata.registry.type=zk
seata.registry.zk.cluster=default
seata.registry.zk.session-timeout=6000
seata.registry.zk.connect-timeout=3000
# zk多结点形式
#seata.registry.zk.server-addr=192.168.0.102:12181,192.168.0.102:22181,192.168.0.102:32181
# zk单结点形式
seata.registry.zk.server-addr=127.0.0.1:2181
# zk账号密码，没有可以不写
seata.registry.zk.username=root
seata.registry.zk.password=123456

# 事务组的部分配置
seata.application-id=${spring.application.name}
seata.tx-service-group=demo-seata-service-group
seata.service.enable-degrade=false
seata.service.vgroup-mapping.demo-seata-service-group=demo
seata.service.grouplist.demo=127.0.0.1:8091
```

启动 Seata-Server 后，会发现Server端的服务会被注册在zk中. 客户端配置完成后启动应用就可以正式体验 Seata 服务。
