

@[TOC]
# 1.fate-serving介绍

 fate-serving是FATE的在线部分，在使用FATE进行联邦建模完成之后，可以使用fate-serving进行在线联合预测。


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407164559439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)


如上图所示，整个集群需要有几个组件

- serving-server
- serving-proxy
- zookeeper     3.4+
- redis     4.0+

**serving-server**

serving-server用于处理所有的在线预测请求、以及模型加载请求。serving-server需要从fate-flow加载模型成功之后才能对外提供服务。在FATE中建好模型之后，通过fate-flow的推送模型脚本可以将模型推送至serving-server。

推送成功之后，serving-server会将该模型相关的预测接口注册进zookeeper ，外部系统可以通过服务发现获取接口地址并调用。同时本地文件持久化该模型,以便在serving-server实例在集群中某些组件不可用的情况下，仍然能够从本地文件中恢复模型。



**serving-proxy**

serving-proxy 是serving-server的代理，对外提供了grpc接口以及http的接口，主要用于联邦预测请求的路由转发以及鉴权。在离线的联邦建模时，每一个参与方都会分配一个唯一的partId。serving-proxy维护了一个各参与方partId的路由表，并通过路由表中的信息来转发请求。





## 1.1 部署流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407164618896.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)



# 2.环境准备

| 系统           | 版本    | 备注                                                         |
| -------------- | ------- | ------------------------------------------------------------ |
| CentOS         | 7       |                                                              |
| Docker         | 18.09.4 |                                                              |
| Docker-compose | 1.23.2  | 参考：[CentOS7安装Docker-compose推荐方案](https://blog.csdn.net/qq_28540443/article/details/104262822) |
| Java           | 1.8     |                                                              |
| Maven          | 3.5.2   |                                                              |
| Zookeeper      | 3.4.13  |                                                              |
| Redis          | 5.0.2   | 参考[docker+docker compose 部署Redis](https://blog.csdn.net/qq_28540443/article/details/104732656) |
| Fate-Flow      | 1.3     | 单机版                                                       |



# 3.编译fate-serving fate-proxy



根据上面对在线联合预测架构说明，我们需要启动fate-serving以及fate-proxy两个服务。

1.clone源码到本地

源码地址：[https://github.com/FederatedAI/FATE-Serving](https://github.com/FederatedAI/FATE-Serving)

2.执行命令编译

```
 cd ~/FATE-Serving
 mvn clean package
```



编译成功后，结果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407165007570.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)



为了使用方便，将编译好的fate-serving以及fate-proxy包copy出来

fate-serving目录结构如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407165023340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)


| 文件/文件目录名         | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| bin                     | 存放fate-serving启动、停止、状态获取等相关文本common.sh，存放服务进程id |
| conf                    | 服务相关配置                                                 |
| fate-serving-server.jar | 服务编译的包                                                 |
| lib                     | 依赖包                                                       |
| service.sh              | 启动/停止等操作服务的命令脚本封装                            |
| logs                    | 存放日志                                                     |



fate-proxy目录结构如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407165038207.png)

结构说明参考fate-serving.



# 4. 配置并启动fate-serving

fate-serving的相关配置主要是在`` conf/serving-server.properties  `` 文件下

这边提供我的配置参考如下：

```
ip=0.0.0.0
port=8000
#serviceRoleName=serving
#inferenceWorkerThreadNum=10
# cache
#remoteModelInferenceResultCacheSwitch=true
# in-process cache
#modelCacheMaxSize=100
#remoteModelInferenceResultCacheTTL=300
#remoteModelInferenceResultCacheMaxSize=10000
#inferenceResultCacheTTL=30
#inferenceResultCacheCacheMaxSize=1000
# external cache
redis.ip=192.168.2.103
redis.port=6379
#redis.password=fate_dev
#redis.timeout=10
#redis.maxTotal=100
#redis.maxIdle=100
#external.remoteModelInferenceResultCacheTTL=86400
#external.remoteModelInferenceResultCacheDBIndex=0
#external.inferenceResultCacheTTL=300
#external.inferenceResultCacheDBIndex=0
#canCacheRetcode=0,102
#external.processCacheDBIndex=0

# adapter
OnlineDataAccessAdapter=MockAdapter
InferencePostProcessingAdapter=PassPostProcessing
InferencePreProcessingAdapter=PassPreProcessing
# external subsystem

# model transfer
model.transfer.url=http://127.0.0.1:9380/v1/model/transfer
# zk router
zk.url=zookeeper://localhost:2181
useRegister=true
useZkRouter=true
# zk acl
#acl.enable=false

```



主要配置服务端口、redis相关以及zookeeper相关，必须保证redis与zookeeper能连通。

然后愉快地启动fate-serving服务即可

具体命令如下

```
sh service.sh start
```



启动成功后，还得观察日志，``  tail -fn 300 logs/fate-serving-server.log``是否报错。

# 5. 配置fate-serving到fate-flow

要实现在线预测必须将经过fate-flow训练的模型通过api提供到fate-serving，因此需要在fate-flow配置fate-serving的服务地址。

`` docker exec -it fate_python bash`` 进入fate-flow容器

编辑配置文件``  vi arch/conf/server_conf.json``

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407165058223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)



保存成功后，`` docker restart  fate_python ``重新启动容器.

# 6. 配置并启动fate-proxy

客户端通过http协议与fate-proxy交互，并不是直接与fate-serving交互。

编辑 **fate-proxy**编译目录下的 `` conf/application.properties`` 如下：

```
 # same as partyid
coordinator=1000
server.port=8059
#inference.service.name=serving
## actuator
#management.server.port=10087
#management.endpoints.web.exposure.include=health,info,metrics

#random, consistent
#routeType=random

route.table=conf/route_table.json
#auth.file=/data/projects/fate-serving/serving-proxy/conf/auth_config.json

useZkRouter=true
zk.url=zookeeper://localhost:2181

```



主要配置项是zookeeper,zookeeper作为服务注册中心，当模型被推送到fate-serving后会产生一系列服务注册到zookeeper,fate-proxy通过zookeeper发现服务。

然后愉快地启动fate-proxy服务即可

具体命令如下

```
sh service.sh start
```

# 7.推送模型

经过2-6步骤我们已经准备就绪。

`` docker exec -it fate_python bash`` 进入fate-flow容器

编辑 `` examples/publish_load_model.json `` 

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-mVCDV10c-1586249085597)(C:\Users\lps\Documents\myblog\fate\1.fate-serving介绍.assets\image-20200407162703012.png)]

执行以下命令推送模型：

```
 python fate_flow_client.py -f load -c examples/publish_load_model.json
```

输出结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407165117162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)


# 8. 绑定模型

`` docker exec -it fate_python bash`` 进入fate-flow容器

编辑 `` examples/bind_model_service.json``

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407165134475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)

执行以下命令推送模型：

```
python fate_flow_client.py -f bind -c examples/bind_model_service.json
```

输出结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040716514857.png)

# 9. 在线预测

**接口地址：**

http://192.168.2.103:8059/federation/v1/inference

    1.ip:port 对应fate-proxy的ip:port
    
    2. 后面v1好像就是v1了，表示接口版本

  **请求方式**

POST

**请求参数**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407165238108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)

**返回结果**：



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407165254796.png)


显示按照提供参数80%可能性良性


对比我预测前的结果数据为 1 ，接近预测结果。

# 10.总结

以上的实践主要是单机版的，所以在在线联合预测上面并没有实践，后面将会进行补充，从架构来看，联合的核心主要是通过fate-proxy，通过配置路由表的方式，双方使用共同模型，由数据使用方（Guest）发起请求，对样本数据进行处理，再获取数据提供方（Host）对样本数据的处理结果，进行整合返回。

# 参考资料

[\[https://github.com/FederatedAI/FATE-Serving/wiki](https://github.com/FederatedAI/FATE-Serving/wiki)