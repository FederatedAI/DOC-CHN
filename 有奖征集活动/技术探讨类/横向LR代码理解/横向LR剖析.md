# 横向LR的代码理解
<br>
横向LR以及其他类似的横向联邦学习的代码框架比较清晰，相对于纵向逻辑更好理清
整体的横向LR的结构如图：
<br>

![1](https://img-blog.csdnimg.cn/20200812163817441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
我们可以看到逻辑就是A和B在本地训练若干轮(aggregate_iters)后来一次向Arbiter的聚合，之后收到Arbiter的回复后便结束了一轮。所以，我们想要理清整个横向LR的代码框架可以从client怎么向arbiter传送数据以及arbiter怎么拿到传来的数据这两方面考虑。至于arbiter反过来传给client以及client的接收和前面两步类似，所以只需了解前两步即可。
## `一、client向arbiter传送过程`
还是以Guest方为例（Host方类似），找到federatedml / linear_model / logistic_regression / homo_logsitic_regression/homo_lr_guest.py文件后观察其fit函数，也就是训练函数，可以看到其结构如下：
![2](https://img-blog.csdnimg.cn/20200813105356949.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
下方绿色部分是指guest在本地训练的部分，上分红色部分是指将模型聚合（也就是发送给arbiter方的部分），从上方红色框的条件语句可以看到，当我们当前轮数满足了aggregate_iters的整数倍（也就是每隔aggregate_iters聚合一次）或者达到最大轮数时候，我们的guest方会向arbiter发送一次，我们这部分内容就重点关注是如何发送的。
`step 1： 了解model_weights变量`

```python
weight = self.aggregator.aggregate_then_get(model_weights, degree=degree,suffix=self.n_iter_)
```
传输的源头起于上面这行代码，但传入的参数model_weights究竟是什么东西呢，如果我们想查看它甚至是想修改它应该怎么做呢？
model_weights的起源是

```python
self.model_weights = self._init_model_variables(data_instances)
```
不难看出它是一个对象类，具体打印出type是
<class 'federatedml.framework.weights.TransferableWeights'>，我们具体进入这个类可以发现当我们调用其 unboxed 方法可以将这个对象类中的值取出来，而这个取出来的值是numpy类型，这个类型是可以供我们修改的。

`step 2 : 进入aggregate_then_get函数`
![3](https://img-blog.csdnimg.cn/20200813123158970.png#pic_center)
可以看到这个函数分为发和收两部分，我们上边也提到了我们只看client向arbiter这一趟线，所以我们只关注  **send_model**  这个函数即可。
顺便一提，此函数所在的位置 federatedml / framework / homo / procedure / aggregator.py中，这个aggregator.py可以说是整个横向联邦的大管家，我们交接的主要函数都是通过它来转手的，而且看过往期教学视频的同学应该也知道，这个函数正是横向联邦的核心。

`step 3:   进入send_model函数`
![4](https://img-blog.csdnimg.cn/20200813123840315.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
先看secure_aggregate，它调用了上面的_func，从形参的字面意思可以看出它是把_func作为一个发射器了，我们先看一下secure_aggregate是什么样子

`step 4 查看secure_aggregate函数`
![5](https://img-blog.csdnimg.cn/2020081312403715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
1、degree行是说是否有权重的相乘
2、enable_secure_aggregate行是说要不要加密（如果我们开始修改了model_weights的值，这里很可能会带上加密后改变原有值，所以如果数值变了不妨在这儿找找看）
3、remote_weights行调用的for_remote函数是将我们的
<class 'federatedml.linear_model.linear_model_weight.LinearModelWeights'>类型的model_weights转变为:<class 'federatedml.framework.weights.TransferableWeights'>以便后续一些操作

所以上面这个函数就是包装了一下我们的model_weights然后就利用_func发射出去了，我们接下来看一下这个发射函数什么样子：
`step 5 : 查看_func函数`
![6](https://img-blog.csdnimg.cn/20200813124633616.png#pic_center)
点击进入_model_scatter.send_model函数
![7](https://img-blog.csdnimg.cn/20200813124730735.png#pic_center)
继续进入remote_parties(这里边我认为是remote函数是传递功能)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813124835621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
`step 6 进入remote函数`
进入后会惊奇的发现这个函数啥也没写
![7](https://img-blog.csdnimg.cn/20200813125113368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
对此我也困惑了许久，后来找到了实现它的函数位置（对于单机版的用户来说它在arch / api / impl / based_1x / federation_standalone.py中）
![具体如上图](https://img-blog.csdnimg.cn/20200813125326439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
至此，client向arbiter传送的过程大体结束，接下来介绍一下arbiter如何收下这些数据。


## `二、arbiter接收client的数据`

![1](https://img-blog.csdnimg.cn/20200813125729614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
打开homo_lr_arbiter,py发现和guest的结构类似，我们下手的目标便是上图中红色框中的内容。

`step 1 : 打开aggregator.aggregate_and_broadcast函数`
![2](https://img-blog.csdnimg.cn/20200813125929360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
因为我们想要看arbiter如何接收数据，所以我们应该关注上图红色框中的aggregate_model函数

`step 2 : 打开aggregate_model函数`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813130311352.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
第一行models的获取便是我们要关注的核心部分，经过测试我发现他是一个generator对象，next(models)[0]代表每次生成的model，next(models)[1]代表每次生成的degree，但是在这儿我重要的话说三遍！！！！！千万千万千万不要用 LOGGER.debug打印了next(models)[0]后不将LOGGER语句注释掉，因为如果用LOGGER调用过一次next，它下边reduce函数就会从下一次调用，就会匹配不上，此处bug本人曾改了许久才发现了问题。
好了废话不多说，我们继续打开models的创造者

`step 3 : 打开get_models_for_aggregate函数`
![6](https://img-blog.csdnimg.cn/20200813130733886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
看到yield便可知道，这是个生成器函数，第一次会弹出guest_model，后边会依次弹出各个host_model，在上图中的我的注释部分提到可以在“这儿”修改，是因为我之前用guest方举例，如果我们在guest方修改了数据，就可以从这个函数的这个地方拿出来看或者修改。
值得一提的就是，这里的guest_model类型是:<class 'federatedml.framework.weights.TransferableWeights'>，这个类对象可以通过unboxed方法提取它的值，但是！但是！但是！
#### `我们没有直接修改它的unboxed值的办法！所以我想了一招，直接去这个类中添加一个set方法后，调用set方法传入我们新改的值，如下：`
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081313154798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
之后再在上述guest_model的地方修改即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813131653405.png#pic_center)

到此为止，横向的LR结构的client向arbiter的单向流步骤大体已经讲完，从arbiter返回client类似，在此不过多赘述。
另外顺便提一下，在host方貌似数据都是加密过的，不能直接用unboxed看到其值，如果测试用的话，大家可以将conf文件中加密部分改为null即可显示数据。如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813131942916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70#pic_center)
至此，横向LR的代码理解篇结束，祝大家事业有成一帆风顺！