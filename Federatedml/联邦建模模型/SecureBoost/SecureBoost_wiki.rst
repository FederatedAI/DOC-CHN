此此处会列出算法中SecureBoost的常见问题以及相应的注意事项和解决方案。

常见问题
========

1.  **Q: param中min_sample_split与min_leaf_node的区别?** A:
    min_leaf_node是现在这个点能不能分裂；min_sample_split是限制分裂后的两边样本数至少达到多少

2.  **Q: 请问现在secureboost支持哪些类型的任务呢?** A:
    目前，secureboost支持横向，纵向场景下的二分类，回归，多分类的任务

3.  **Q: SecureBoost的论文中 预测的时候 走到相应的分支需要对应party
    告知这个分支走向是left 还是right，是这样么?** A:
    是的，预测的时候也需要多方一起合作

4.  **Q: 可以关掉secureboost的加密么?** A:
    这个是不支持的，但是如果有这个需求，可以到federatedml/tree/hetero_secureboosting_tree_guest.py
    下修改generate_encrypter

5.  **Q: secureboost现在是否支持缺失值?** A:
    1.2开始secureboost支持缺失值，目前样例数据中也有带缺失值的\ `数据 <https://github.com/FederatedAI/FATE/blob/master/examples/data/ionosphere_scale_a.csv>`__
    可以试验。

6.  | **Q:
      validation_freqs这个参数会影响训练过程的secureboost中树的分裂嘛？还是只是用来展示当前模型在验证集上的效果？**
    | A: 是用来展示的。

7.  | **Q:
      SecureBoost在第一棵树第一个节点分裂时，传输的gi是否会泄露label信息？**
    | A:
      同态加密对每次y的加密都会不一样的，所以host拿不到gi的信息。如果还有顾虑的话，可以按照paper上面描述，第一个树完全用guest特征来建树。
      fate-1.5会推出这个feature.

8.  | **Q:
      SecureBoost在传输信息中加入混淆之后，是否会影响host这边的分桶(get_histogram)？**
    | A: 加的是r\ :sup:`n(r是混淆），r`\ n \* lambda % (n^2) =
      1，刚好等于phi(n^2)

9.  | **Q: 在Secureboost中每轮训练的模型该如何导出?**
    | A:
      目前是不支持checkpoint的，checkpoint和warmstart已经在我们的规划当中了。

10. | **Q:
      为什么调整encrypted_mode_calculator_param.mode参数对boost运行时间改进效果不明显？
      如何提升效率？**
    | A:
      boost的计算里面，直方图的加法占了很大一部分，同态加密的加法在boost里面体现了出来，可以换下IterativeAffine，会快一点。
