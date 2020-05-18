此处会列出算法中横向框架的常见问题以及相应的注意事项和解决方案。

常见问题
========

1. | **Q: arbiter端对多方二进制模型的聚合是怎样实现的？**
   | A:
     请参考横向相关\ `文档 <https://github.com/FederatedAI/FATE/tree/master/federatedml/framework/homo>`__

2. | **Q:
     横向联邦的中立方arbiter必须是可信的吗？如果不要求的话，模型聚合的时候，arbiter是否可以推出host/guest的原始数据信息吗？**
   | A: 横向nn是secret aggregation. Arbiter 解密时不知道每一方具体的模型
     只能得到平均模型 也就无法得到每一方的梯度。
