
# 纵向arbiter方nesterov_momentum_sgd优化演算分析
>在纵向联邦学习中，模型训练过程guest和host方局部梯度加密后发给了中间方arbiter，arbiter对其进行解密并作了优化操作后再将梯度返还给双方进行本地参数更新，本文将具体介绍arbier方如何运用nesterov_momentum_sgd这一方式对获取到的双方梯度进行更新。
## 一、Nesterov Momentum介绍
Nesterov Momentum即牛顿动量梯度下降，是sgd(随机梯度下降）的优化版，SGD方法的一个缺点是，其更新方向完全依赖于当前的batch，因而其更新十分不稳定。解决这一问题的一个简单的做法便是引入momentum。momentum即动量，它模拟的是物体运动时的惯性，即更新的时候在一定程度上保留之前更新的方向，同时利用当前batch的梯度微调最终的更新方向。这样一来，可以在一定程度上增加稳定性，从而学习地更快，并且还有一定摆脱局部最优的能力：在Nesterov Momentum中，先沿着之前积累的梯度走一步，然后再计算这一步的终点的梯度，利用该梯度进行校正，得到最终的更新方向。相当于在Nesterov中添加了一个校正因子，Nesterov Momentum的位置更新取决于之前积累的梯度和根据之前积累梯度走到的位置的梯度。公式如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020082715205927.png)
其中ε表示学习率，α表示动量参数,θ为初始参数，ν为初始速度。
## 二、FATE源码比对
**源码路径：federatedml\optim\optimizer.py**
Nesterov Momentum函数位于190—207行
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200827153227944.png)
其中参数与传统公式的对应关系及含义为
参数     | 含义
-------- | -----
nesterov_momentum_coeff  | 动量参数α
opt_m  | 初始速度v
grad | 梯度g
 learning_rate | 学习率ε
delta_grad|优化后梯度
v|更新后速度v
* 第200行是opt_m初始化为一个和收到的梯度维度一致的零矩阵
* 第201行为更新速度，对应公式中的compute velocity update（蓝框所示）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200827161752569.png)
* 第202行为优化梯度,公式为优化梯度=动量参数*初始速度-（1+动量参数）*速度
* 第203行是将更新后的v赋值于初始速度后再进行循环计算
------
## 三、实际计算演练
以下通过LOG中获取到的iter:0时的数据进行实际演练说明：

从arbiter端获取打印出来的日志如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020082718473135.png#pic_center)
learning-rate如图所示设为0.15,而动量参数从py文件可得知为0.9
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828100536943.png)

arbiter方获取到guest方和host方传来的优化前梯度信息，size list表示host方1个，guest方7个（6个梯度一个截距）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828113801446.png#pic_center)

arbier方获取的梯度进行优化：
根据前面获得的已知条件learning_rate为0.15，动量参数为0.9，比如对于优化前的梯度grad=2.91599744,套用源码第200行和第201、202行公式:
>v=0.9*0-0.15*2.91599744=-0.43739962
>由于第203行将v赋值给了opt_m,所以opt_m=-0.43739962

>优化后梯度delta_grad=0.9*0-(1+0.9)*-0.43739962=0.83105927

元组中的其他梯度值以此类推都是可以根据公式算出的数值与日志是可以一一对应上的~
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828113317772.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828113832246.png#pic_center)
arbiter方计算好优化后的梯度delta_grad，再根据之前的size list,把delta_grad拆成两个元组（一个包含1个梯度，一个包含6个梯度和1个截距）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828113850234.png#pic_center)
双方接收到arbiter发回的优化梯度后，根据优化梯度来更新模型参数，自此迭代下去直到模型收敛或达到设置的迭代次数上限，模型停止训练。






