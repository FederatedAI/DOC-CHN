此处会列出算法中关于交集模块的常见问题以及相应的注意事项和解决方案。

常见问题
========

1. | **Q:
     目前FATE采取的是RSA+Hash实现id对齐样本。请问有对齐id+timestamp的方案吗？**
   | A:
     目前提供的id对齐方案，详情可以参考相关\ `文档 <https://github.com/FederatedAI/FATE/tree/master/federatedml/statistic/intersect>`__\ 。
     id+timestamp对齐的方案已经在计划中，欢迎关注我们后续版本。

2. | **Q: 我看到Intersect模块报错了，log里面出现“Count of data_instance
     is 0”？**
   | A: 请检查DATAIO，是否有正确的输入数据集

3. | **Q: Intersect支持多方么？**
   | A: Intersect是支持多host的

4. | **Q: 关于Intersect有详细的介绍么？**
   | A: 请见
     `文档 <https://github.com/FederatedAI/FATE/tree/master/federatedml/statistic/intersect>`__
