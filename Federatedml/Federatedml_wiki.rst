此处会列出涉及算法的常见问题以及相应的注意事项和解决方案。

常见问题
========

1. | **Q: 每个算法组件的参数列表应该去哪里查看？**
   | A:
     我们在github上有参数列表\ `文件 <https://github.com/FederatedAI/FATE/tree/master/federatedml/param>`__.
     同时在example下各个算法中，也有例子的配置文件可供大家参考。
     后续会有专门的文档页面。

2. | **Q: federatedml相关的文档都有哪些？**
   | A:
     我们在github的federatedml文件夹下有各个算法的列表，对应算法有链接连到具体算法的文档中。
     具体可以参考：
     https://github.com/FederatedAI/FATE/tree/master/federatedml

3. | **Q: Guest,Host, Arbiter分别是什么?**
   | A:
     Guest表示数据应用方，Host是数据提供方，在纵向算法中，Guest是有标签y的一方。arbiter是用来辅助多方完成联合建模的，主要的作用是用来聚合梯度或者模型,比如纵向lr里面,各方将自己一半的梯度发送给arbiter，然后arbiter再联合优化等等,arbiter还参与以及分发公私钥，进行加解密服务等等

4. | **Q: 为什么只能是Guest发起任务?**
   | A:
     因为Guest是数据应用方，在纵向模型的建模中往往是有y的一方。通常整个建模任务通常是服务于y的业务逻辑的，因此也只有guest需要应用这一模型。同时，如果允许Host方任意发起建模流程，有可能会有Guest方的数据泄露风险。

5. | **Q: 整个建模任务的流程大概是怎么样的呢?**
   | A:
     双方建模的任务，一般都是guest端发起的。但是发起任务之前要求双方都要把自己的数据上传，同时在启动任务的脚本里要指定上传的table
     name和namespace。
     可以看min_test，流程是先让host上传数据，获取到table和namespace，然后在guest端输入进来，guest会进行
     (1)上传自己的数据

(2) 把自己的数据和host的数据的table和namespace都填进配置里面
    (3)发起建模任务

6.  **Q: 如何对建好的模型做离线预测? 离线预测也需要联邦吗？** A:
    在训练好模型以后，记录下model version和model
    id(可以在提交任务的runtime_conf下面找到，或者根据fate
    flow接口按照jobid查询到)，然后在example下，有一个样例的predict配置，将两项内容填进去，同时配好多方信息，再启动任务即可。
    具体流程可参考\ `文档 <https://github.com/FederatedAI/FATE/tree/master/examples/federatedml-1.0-examples#step3-start-your-predict-task>`__
    在纵向场景下，离线预测也是需要各方参与的。而在横向场景下，各方可以使用得到的模型在本地预测。

7.  | **Q: 现在哪些算法支持多个host建模了呢**
    | A:
      目前交集(Intersect)，横向和纵向的逻辑回归(Homo/Hetero-LR)，纵向的线性回归(Hetero-LinR)、纵向SecureBoost,联邦特征分箱和联邦特征选择均已支持多个数据提供方(Host)参与建模。

8.  | **Q: 多host的任务配置如何配**
    | A:
      各支持多host的模块均在\ `examples <https://github.com/FederatedAI/FATE/tree/master/examples>`__
      里提供多host配置样例。可以参考样例调整配置。

9.  | **Q: 如何使用Spark模式**
    | A: 1. 首先需要部署yarn环境，并把SPARK_HOME加入fate_flow启动脚本

    2. 在conf的job_parameters中加入”backend”: 1

10. | **Q: 支持哪些Spark提交模式**
    | A: 目前仅支持Spark-Client模式

11. | **Q: FATE计算流程里怎样调用spark？**
    | A: 存储跟federation依赖eggroll; fateflow用spark submit 提交任务。

12. | **Q: 最多可以多少方参与？ 性能如何？是否可以一方发起所有操作？**
    | A:
      目前横向可以支持多方，单机版本至少10个并行跑过没问题（分布式版本，目前没用那么多机器测试）。多方的性能数据暂时还没统计。目前就是FATE就是通过一方来发起所有任务，其他方不需要任何手动操作。

13. | **Q:
      如何对参与联邦的各方中的数据质量进行评价，以避免脏数据对联邦模型的影响?**
    | A:
      可以通过对加入方的数据对模型提升效果来评判。目前参与横向联邦的数据需要在各方中全部对齐，后续如何解决仅在本方中独有的数据参与联邦学习。这个问题可以通过联邦迁移学习来解决

14. | **Q: role parameter 与 algorithm parameter的区别？**
    | A:
      role_parameters表示的是各个party独有的参数，比如输入数据的表库名、以及对该数据的处理方法。
      algorithm_parameters表示的是算法各方共同的运行参数，大部分情况下算法参数都可以写在algorithm_parameters.

15. | **Q: 如何跑自己的数据呢？比如说使用纵向LR算法**
    | A: 首先，使用fate_flow上传自己的数据，记录下对应的table和table
      name，具体上传流程参考这里的upload
      guide:https://github.com/FederatedAI/FATE/tree/master/doc
      然后在example下有hetero_lr的文件夹，里面有例子的dsl和conf，把conf里面的数据table和table_name换成刚刚上传了的数据表
      然后利用fate_flow的接口启动任务即可，具体启动命令，在文件夹下有README

16. | **Q: 替换新数据后，为什么训练模型会报overflow/infinite错误？**
    | A:
      在计算梯度的时候，LR使用了0点附近的泰勒展开的，如果偏离0点太多，有可能误差会很大，导致浮点数爆炸了。可以通过数据归一化避免。可以在配置的dsl里面添加数据归一化模块，具体可以参考一下example里面的文件名带dsl的配置文件。

17. | **Q: Guest/Host可以有一方没有feature吗？**
    | A:
      目前纵向场景不支持某一方没有feature。如果guest只有id和y，可以考虑添加一列特征，值都是0.00001这样。不过这种做法存在一定风险。
      目前FATE的组件需要特征不为空：因为在纵向联邦场景，我们认为双方都需要有数据参与建模，如果某一方特征为空，我们更希望这被提前感知到。

18. | **Q:
      如果有一方的模型效果比较差或者有一方恶意的降低模型效果，最后会不会拉低多方的平均的模型效果？**
    | A: 可以参考数据并行的分布式训练来考虑：

    1. 一方模型效果比较差: 如果是这一方数据很差(脏数据),
       那看这一方的权重，会一定程度拉低效果；如果仅仅是这一方的数据比较偏，那其实影响有可能是正面的
    2. 一方恶意降低模型效果:
       用模型平均防范不了恶意攻击导致的建模效果降低，也有一些方法可以减少损害
       比如改用调和平均。不过我们这边的假设是honest but
       curious，这种不遵守协议的并不符合。

19. | **Q: FATE现在使用了哪些加密协议呢？**
    | A: 我们会根据不同算法特点选择合适的加密算法。
      目前我们除用到同态加密Paillier外，还实现了secret-share
      （SPDZ），将其用在纵向pearson相关性计算算法，DNN没有使用。

20. | **Q: 可以不加密使用FATE交互吗？**
    | A:
      FATE本身定位为一种安全的计算框架，我们会根据不同场景，不同算法设计不同加密机制，同一个算法也会支持不同等级的加密机制供用户选择。但是完全不加密的方式，目前FATE还不提供

21. | **Q: 各种examples使用样例数据的来源?**
    | A:
      请参照相关说明\ `文档 <https://github.com/FederatedAI/FATE/tree/master/examples/data>`__.

22. | **Q: 如何在job配置文件中设置某参数数值为空（None）？**
    | A: 配置文件中设为null即可。

23. | **Q: 在处理数据时报错 AttributeError:“NoneType object has no
      items”？**
    | A:
      前期版本中eggroll不支持单个超过32M数值，此问题已在1.3版本修正，更新版本后可解决此问题。

24. | **Q:
      在使用hetero联邦学习的时候，我想让host方来预测结果，请问有没有办法做到？**
    | A: 因为这么干有安全风险，我们现在是不支持的。

25. | **Q:
      是否所有算法都需要arbiter这一角色呢？有没有实现了没有第三方情况下的联邦学习呢？**
    | A: 目前hetero_secureboost, hetero_nn是不需要第三方的。

26. | **Q: 我该怎么看运行日志呢？**
    | A:
      federatedml的日志是可以在fateboard的dashboard下与部署文件夹下的logs文件夹中看到的。

27. | **Q:
      在使用hetero联邦学习的时候，我想让host方来预测结果，请问有没有办法做到？**
    | A: 因为这么干有安全风险，我们现在是不支持的。

28. | **Q: 模型训练好后只能离线预测吗？有没有办法将模型落地呢？**
    | A: 可以关注下我们的FATE-serving, 支持在线预测。
      目前支持的模型是HeteroLR:raw-latex:`\SecureBoost`，后续会支持更多在线模型预测。

29. | **Q: 横向联邦学习，和分布式训练是什么关系？**
    | A: 抛开加密部分，大体上训练流程是类似的，差异主要在：

    1. 数据假设不一样，
       分布式训练一般都是同分布的数据，联邦学习大多数都是非同分布的
    2. 引入信息保护的策略。

30. | **Q: y值参与计算和传输的只有training环节吧？**
    | A:
      不一定，特征工程里面也可能用到，在特征分箱里面，会将加密的label传给host

31. | **Q:
      部署时arbiter可以和host放一起吗？arbiter可以和guest放一起吗？一定需要arbiter参与吗？**
    | A:
      arbiter和host放一起是可以的。arbiter和guest放在一起，可能会造成模型和用户级别信息的泄漏，横向联邦场景中，有两种情况。
      一种是明文模型存在中心点，那么预测阶段自己方已有模型，无需协作方。一种是FATE已实现的横向LR加密训练，那么预测阶段需要持有模型方arbiter进行协作。

32. **Q: guest方可以只有label没有数据吗？** A:
    可以在guest那边，手动加一列X，X的取值很小，比如0.00001这样。但是只有y有一定泄露数据的风险。

33. **Q: 支持spark作为计算引擎的这个两点有介绍文档吗？** A:
    请见https://github.com/FederatedAI/FATE/tree/master/examples/federatedml-1.x-examples#use-spark

34. | **Q: 请问联邦学习纵向在预测的时候，一方数据欠缺能够预测吗？**
    | A:
      secureboost是可以支持的，当成缺失值。如果是HeteroLR，会看成特征值全0，这个可能会影响效果。

35. | **Q: transfer variable是能收发DTable吗？**
    | A: python object都是可以收发的。

36. | **Q: 提交predict任务支持评估预测结果吗？**
    | A: 可以的，需要在dsl里面evaluation对应的模块增加“need_deploy”:
      true。但是需要注意的是，纵向场景下，只有guest可以跑评估预测，所以host的“need_deploy”应该还是false。

37. | **Q: 所有的组件的run 都是从modelbase 开始的么？**
    | A:
      每个模型都需要有run:raw-latex:`\export`\_model:raw-latex:`\save`\_data的接口来给fate_flow调用，modelbase提供了一些共同的函数，现在各模型继承它，然后根据需要重载。

38. | **Q: 运行FATE的算法比较慢，可能是什么问题呢？**
    | A:
      有可能和partition的设置有关，我们默认的参数是8个partition的，如果比核数多，可能会导致慢了一倍，建议将partition改成和核数相同，会快一些
      ；或者可能因为加密较慢，可以调整为更快的加密算法，比如Secureboost可以使用iterativeAffine加密。

39. | **Q:
      请问“validation_freqs”参数的对数据的验证是指什么验证呢？类似于交叉验证的作用嘛？这个取值有没有建议的范围呢？**
    | A:
      这是指在训练过程中评估的频率，会使用验证集的数据，每过几轮，评估一下目前的效果这样。如果没有这个需求，可以不设置，不影响最终建模结果的。
