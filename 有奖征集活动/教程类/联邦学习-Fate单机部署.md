

@[TOC](目录)
## 1. Fate介绍
FATE (Federated AI Technology Enabler) 是微众银行AI部门发起的开源项目，为联邦学习生态系统提供了可靠的安全计算框架。FATE项目使用多方安全计算 (MPC) 以及同态加密 (HE) 技术构建底层安全计算协议，以此支持不同种类的机器学习的安全计算，包括逻辑回归、基于树的算法、深度学习和迁移学习等。

Fate主要包含：

### 1.1 FederatedML
一个实用的、可扩展的联邦机器学习库。

Federatedml包括许多常见机器学习算法的实现以及必要的实用工具。所有模块均采用去耦模块化方法开发，以增强可扩展性。具体来说，提供：
1. FML算法：用于DataIO，数据预处理，特征工程和建模的联合机器学习算法。下面列出了更多详细信息。
2. 实用程序：启用联合学习的工具，例如加密工具，统计模块，参数定义以及传递变量自动生成器等。
3. 框架：用于开发新算法模块的工具包和基础模型。框架提供了可重用的功能，以标准化模块并使它们紧凑。
4. 安全协议：提供多种安全协议，以实现更安全的多方交互计算


### 1.2 FATE Serving
一个可扩展的、高性能的联邦学习模型服务系统。
FATE-Serving是针对联合学习模型的高性能工业化服务系统，专为生产环境而设计。
FATE-Serving现在支持
	• 高性能的在线联合学习算法。
	• 联合学习在线推理管道。
	• 动态加载联合学习模型。
	• 可以服务于多个模型或同一模型的多个版本。
	• 支持A / B测试实验模型。
	• 使用联合学习模型进行实时推理。
	• 支持多级缓存以获取远程方联合推断结果。
	• 支持用于生产部署的预处理，后处理和数据访问适配器。

原理：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220175943600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)





### 1.3 FATEFlow
FATEFlow是联邦学习建模Pipeline 调度和生命周期管理工具，为用户构建端到端的联邦学习pipeline生产服务。FATEFlow实现了pipeline的状态管理及运行的协同调度，同时自动追踪任务中产生的数据、模型、指标、日志等便于建模人员分析。另外，FATEFlow还提供了联邦机制下的模型一致性管理以及生产发布功能。

FATEFlow调度流程：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020022018000239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)



### 1.4 FATEBoard
FATEBoard是联邦学习建模的可视化工具，为终端用户可视化和度量模型训练的全过程，帮助用户更简单而高效地进行模型探索和模型理解。
FATEBoard由任务仪表盘、任务可视化、任务管理与日志管理等模块组成，支持模型训练过程全流程的跟踪、统计和监控等，并为模型运行状态、模型输出、日志追踪等提供了丰富的可视化呈现。FATEBoard可大大增强联邦建模的操作体验，让联邦建模更易于理解与实施，有利于建模人员持续对模型探索与优化。




### 1.5 Federated Network
联邦学习多方通信网络。

架构如下：
• 联邦学习方之间的跨站点通信
• 模组
	○ MetaService：元数据管理者和持有者
	○ Proxy：应用程序层传输端点
	○ Federation ：全局对象的抽象和实现，即各方之间要“联合”的数据
	○ Fate-Exchange: 负责通信

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180022669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)




### 1.6 KubeFATE
使用云本地技术管理联邦学习工作负载。

当前，KubeFATE支持通过Docker Compose和Kubernetes 进行FATE部署。






## 2.Fate部署架构说明
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180045158.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180056810.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)
接下来为了更好的学习Fate，先来个单机版部署


## 3.环境准备
|系统工具  |版本  |备注
|--|--|--|
| CentOS | 7 |配置：4核 8G 256G
| Docker |   18.09.4|
| Docker-compose |   1.23.2| 参考：[CentOS7安装Docker-compose推荐方案](https://blog.csdn.net/qq_28540443/article/details/104262822)|
| GO |   1.13.4| 参考：[CentOS7安装Go](https://blog.csdn.net/qq_28540443/article/details/104265094)

## 4. 部署
由于是通过Docker-Compose方式，下载官方部署脚本
打开控制台，输入以下命令：

```bash
 wget https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/docker_standalone-fate-1.2.0.tar.gz
```
解压包
```bash
tar -xvf docker_standalone-fate-1.2.0.tar.gz
```

执行部署脚本

```bash
 cd docker_standalone-fate-1.2.0 
```

```bash
bash install_standalone_docker.sh
```

执行成功后，查看当前启动容器

```bash
docker ps
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180225176.png)

fate_fateboard : 监控面板
fate_python： 单机版 Federated Network

## 5. 单机测试

运行测试脚本

```bash
CONTAINER_ID=`docker ps -aqf "name=fate_python"`
docker exec -t -i ${CONTAINER_ID} bash
bash ./federatedml/test/run_test.sh
```

测试完成，控制台输出如下结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180246206.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)


## 6.执行测试任务
进入fate_python容器

```bash
docker exec -it fate_python bash
```

进入example目录

```bash
cd /fate/examples/federatedml-1.x-examples
```

执行快速测试脚本

```bash
 python quick_run.py
```

执行完成后控制台输出如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180305497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)


## 7.查看测试任务执行情况
上面单机部署中，包含的其中一个容器fate_fateboard是监控面板
访问``http://ip:8080 ``
可以看到6执行的所有任务的情况
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180321100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)


同时提供执行JOB的详细日志

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220180338717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NTQwNDQz,size_16,color_FFFFFF,t_70)