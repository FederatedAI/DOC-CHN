# 纵向LR的代码实现分析
<br>
在federatedml目录下的算法中，逻辑回归是比较容易理解并且实现步骤也比较易懂的，所以本文从逻辑回归中的纵向LR下手，具体带领读者分析其代码实现。

<br>
<br>

#### ` 一、原理分析`
![1](https://img-blog.csdnimg.cn/20200811173412769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
正如上图，我们可以看到，LR的损失函数的梯度是指数形式，由于不能进行同态加密，所以用泰勒展开为多项式。这个多项式的表达式就是我们上图中的红色框乘以x，而这个红框内的表达式我们亲切的用d来表示它。
可见：d中含有y，所以d的表达式只有guest能算，而host不能算。
（1）委屈的host只能将||wx||传给guest，也就是我们的**第一步**。
（2）guest收到了host传来的||wx||后，合在一起开心的算出了||d||，并将它分享给委屈的host，这也就是我们的**第二步**。
（3）两边都拿到了||d||自然也就可以根据图上的公式计算损失函数的梯度了，但是他们发现好家伙，这个||d||是加密过的，得靠arbiter解密才行，所以他们就先传回给arbiter，再接收，也就是我们的**第三步和第四步**

**至此，原理图已经分析完毕，接下来可以看看具体的代码实现**

#### ` 二、代码实现`
- 首先进入 /fate/federatedml/linear_model/logistic_regression目录下，可以看到guest,host和arbiter三方的代码，我们拿guest举例（guest是先收host的||wx||，再计算||d||，再回传给host后两边在分别计算梯度发给arbiter解密后再拿回来）
下图，我们找到compute_gradient_procedure这个函数，可以清楚看到每一步的过程（如下图）
![2](https://img-blog.csdnimg.cn/20200811174832370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
从整体上看便是上图了，从细节看我们可以分别点开每个函数进一步分析：
#### `(1)得到host方传来的||wx||————解析get_host_forward函数`
![3](https://img-blog.csdnimg.cn/20200811180348614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
先看蓝色框想找host_forward_transfer对象，于是找到了红色框发现是下面这个register_gradient_proceduce方法实现的（注意这个方法不在当前文件，它是子类）
那么红色框这个函数又是在哪里实现的呢？（如下图）
![6](https://img-blog.csdnimg.cn/20200811180644314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
所以大体上就是host传给了我们的中转站Variable对象，然后我们的guest又从中转站取了出来，这便是步骤1的详解。

#### `(2)计算||d||————解析compute_and_aggregate_forwards函数`
这个函数如果我们直接点进去会发现并没有实现，就很奇怪
![7](https://img-blog.csdnimg.cn/20200811181408149.png)
思来想去，父债子还，找找他的儿子吧，发现果然他的子类实现了这个方法，子类的路径我贴在这里，免去大家寻找：
![8](https://img-blog.csdnimg.cn/20200811181525539.png)
剩下的就是套公式计算了，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811181044610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
#### `(3)将||d||发给host————解析remote_fore_gradient函数`
![8](https://img-blog.csdnimg.cn/20200811182224863.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
从上往下看
函数1和函数2 都是federatedml / optim / gradient /hetero_linear_model_gradient.py中
函数3 是federatedml / optim / gradient /hetero_lr_gradient_and_loss.py中
函数4 是 federatedml / transfer_variable / transfer_class / hetero_linr_transfer_variable.py
过程和从host接收类似，也是经过了中转站Variable。

#### `(4)计算梯度`
![9](https://img-blog.csdnimg.cn/20200811183320283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
#### `(5)接收解密后的梯度`
![10](https://img-blog.csdnimg.cn/20200811183443777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
**本文到此结束，总的来说，纵向没有横向的简易，详见下篇横向LR的代码详解。**
