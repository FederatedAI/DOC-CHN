此处会列出算法中关于GLM的常见问题以及相应的注意事项和解决方案。

常见问题
========

1. | **Q: homo_lr或者hetero_lr的原理？**
   | A:
     请参考github上的算法\ `介绍 <https://github.com/FederatedAI/FATE/blob/master/federatedml/linear_model/logistic_regression/README.md>`__.
     我们的官网上也有详细介绍。

2. | **Q: 目前1.1版本的
     纵向lr好像有点慢，一次迭代通信需要3次，再加上同态加密时间开销好像很大。请问目前有比较快的策略嘛？**
   | A: 相对1.0,
     1.1版本已经把llr通信量降了很多，此外lr参数加密模式那里可以采用用fast模式，会快很多，内部测差不多提升一倍。

3. | **Q:
     纵向逻辑回归我把数据换成了自己的数据，loss在训练过程中一直上升，最终报错，这个是什么问题导致的?**
   | A:
     可以检查下数据是不是没有做归一化，因为算法是在0点附近用泰勒展开拟合的，所以没有归一化的数据，误差会比较大，就容易梯度爆炸了

4. | **Q:
     逻辑回归时，guest一方在计算局部梯度时，原始数据中的表现值label直接参与计算了，然后将梯度值传给了host，这样会不会有label值泄露的安全隐患？**
   | A:
     这个数据是加密的，所以不会有泄漏风险，整个结果都是基于加密机制计算的，结果是密文。

5. | **Q: 逻辑回归训练时可以设置成不使用同态加密吗？**
   | A:
     1.1版本以前可以，1.1以后不行。1.1以前可以把lr参数里面的EncryptParam里面的method设置为None就可以。

6. | **Q: 逻辑回归目前有没有调整参数初始化的方法呢？**
   | A: 目前LR提供的初始化方法可以参考
     ,https://github.com/FederatedAI/FATE/blob/master/federatedml/param/logistic_regression_param.py#L30

7. | **Q: homo_lr 中 变量 re_encrypt_batches 的作用是什么呢？**
   | A:
     在横向计算泰勒梯度时，加密的数经过多次运算后容易溢出，因此需要解密再加密回来，这个re_encrypt_batches
     是用来控制多少个batch做一次re_encrypt的

8. | **Q: 发现aggregator
     中聚合模型/loss的时候是明文操作，这样会不会导致信息泄露呢？**
   | A:
     loss聚合只会聚合一个loss的信息，这个信息根据我们评估是不敏感的，无法反推数据，因此不会数据泄露
     模型聚合时候的解密是发生在arbiter端的，我们这个方案是三方方案，arbiter可以获得明文的模型。另外通过模型也无法反推数据，因此是安全的。

9. | **Q: hetero_lr 的 encrypted_mode_calculator fast
     选项是用了什么策略增加速度？**
   | A: 主要是利用了同态加密的性质，加速了加密的过程，比如[y] = [ x ] +
     (y - x)，这样不需要对y应用encrypt操作。
