此处会列出算法中神经网络模块的常见问题以及相应的注意事项和解决方案。

常见问题
========

1. | **Q: 请问横向nn有参考资料或者论文吗？**
   | A:
     欢迎参考此\ `文件 <https://storage.googleapis.com/pub-tools-public-publication-data/pdf/ae87385258d90b9e48377ed49d83d467b45d5776.pdf>`__

2. | **Q：请问现在NN模块下支持哪些算法呢？**
   | A:
     现在nn目录下已经提供了横向(homo)/纵向(hetero)的nn实现。除了全连接网络外，还支持卷积神经网络。

3. | **Q：现在支持哪些backend呢？能否使用GPU加速？**
   | A: 目前fate支持了tf,
     tf-keras以及pytorch作为backend。目前gpu计算加速的主要是横向的DNN算法，直接走的是tensorflow。

4. | **Q：目前支持预训练模型么？**
   | A: 还不支持预训练的模型使用，后面会支持。
     这里涉及个问题是预训练模型如果以pb的形式发送给其他方，会有执行任意代码的风险，需要考虑下

5. | **Q：fateboard上能够展示神经网络的结构么？**
   | A:
     目前还不支持，我们后面会把tensorboard嵌入进来，进行神经网络可视化展示
