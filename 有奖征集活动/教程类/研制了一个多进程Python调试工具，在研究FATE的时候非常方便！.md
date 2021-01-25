# 研制了一个多进程Python调试工具，在研究FATE的时候非常方便！
## 1. FATE研究背景
相信已经部署过FATE的朋友都知道推荐的部署方式之一是用docker和docker-compose的方法，建议的部署系统是CentOS，并且对系统的配置要求还比较高，因此不管是单机版还是集群版一般都是部署在Linux服务器上的docker容器中。这样的话如果配置远程开发的集成开发环境（IDE），不仅要登入服务器还要能连进容器里，配置起来比较麻烦（个人没试过这种方法，感觉就比较麻烦，并且个人感觉Linux下的vim就很好用，能把vim使用好其实非常方便），因此个人都是用vim编辑代码。

## 2. Python自带Pdb调试工具
大部分人使用Python应该都习惯了使用IDE，常见的如PyCharm, VSCode, Eclipes等，因此可能很少人知道还有Pdb这么个调试工具，Pdb是一个Python自带的模块，使用的时候直接import就行，个人猜想IDE里面的调试功能的一系列按钮背后可能就是执行的Pdb的程序，毕竟IDE只是一种GUI交互而已。Python自带的Pdb调试工具使用方法和常用命令解释如下。
这里使用FATE-docker-standalone-1.5.0里面的/fate/python/fate_flow/fate_flow_client.py的代码为例，如下图所示的箭头处插入断点
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125112228648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
保存后运行如下代码提交一个job

```bash
python fate_flow_client.py -f submit_job -c /fate/examples/dsl/v1/hetero_secureboost/test_secureboost_train_binary_conf.json -d /fate/examples/dsl/v1/hetero_secureboost/test_secureboost_train_dsl.json
```
运行后返回如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125112851581.png#pic_center)

可以看出程序停在了parser = argparse.ArgumentParser()，这句还没有执行。然后在(Pdb)提示右边可以输入调试命令并按回车即可，这里解释几个常用的调试命令如下。

(1) "l": list, 呈现断点处那行代码的前后几行代码，如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125141257808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
(2) "n": next, 执行当前代码，把箭头移向下一行代码，如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125141549663.png#pic_center)
(3) "p": print, 打印变量，用法：p 变量名，如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125141958860.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
(4) "s": step, 进入函数内部调试，如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125142247852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
(5) "q": quit, 退出Pdb调试，如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125142428380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

## 3. FATE中的后台Python进程如何调试？
在FATE服务程序存在很多Python后台服务进程，前面的示例代码fate_flow_client.py实际上是在请求服务，服务请求成功的前提是服务程序已经在后台运行，可以使用命令ps -ef查看后台运行的进程，如下所示，其中箭头所指的就是fate_flow的服务进程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012514324547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

/fate/python/fate_flow/fate_flow_server.py在容器启动（或重启）的时候就会随着自动运行，原因是在创建镜像的时候镜像创建者在Dockerfile中设置了docker-entrypoint.sh，我在部署单机FATE-docker-1.5.0的时候是直接从微众银行官方链接（https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/docker_standalone-fate-1.5.0.tar.gz ） 下载的压缩包，压缩包里面有python.tar的python镜像压缩文件，执行部署脚本的时候里面实际上是在执行docker load载入镜像（具体部署过程可以参看：https://mp.weixin.qq.com/s/13gTDyyHO5EcGBdZA9yoeg ） 因此看不到利用Dockerfile构建镜像的过程，并且镜像原始创建者也没有把Dockerfile放进python镜像，不过我还是在官方的github fate仓库中找到了Dockerfile（https://github.com/FederatedAI/FATE/blob/master/standalone-deploy/docker/python/Dockerfile ），可以看到Dockerfile的最后一句设置了ENTRYPOINT ["docker-entrypoint.sh"]，docker-entrypoint.sh的内容在这个链接（https://github.com/FederatedAI/FATE/blob/master/standalone-deploy/docker/python/docker-entrypoint.sh ）可以看到，也就是容器启动（或重启）的时候就会自动执行docker-entrypoint.sh，而这里面执行了fate_flow_server.py服务。

### 一个简单取巧的方法可以避免fate_flow_server.py随着容器启动（或重启）而运行
既然fate_flow_server.py写在docker-entrypoint.sh中，那么只要找到docker_entrypoint.sh并修改其中的执行fate_flow_server.py这一句的代码即可避免服务启动。由于不太清楚这个脚本中前后的几句shell命令的意思，经过尝试后发现不能直接注释掉，也不能随便运行一个可以停止的py进程，这些都会导致容器无法正常启动，那么和服务进程性质最像的就是死循环，让死循环进程一直在后台运行（机器怎么会知道死循环进程是不是服务进程呢），于是写了死循环/fate/python/fate_flow/dead_circle.py如下：
```python
while True:
    if True:
        pass
```
然后执行
```bash
find / -name docker-entrypoint.sh
```
找到/usr/local/bin/docker-entrypoint.sh并打开，加上dead_circle.py的执行语句，并注释掉fate_flow_server.py的执行语句，如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125152836700.png#pic_center)
保存退出后重启容器即可，重启容器后利用ps -ef查看后台进程会发现fate_flow_server.py没有在后台运行，这样我们就可以手动启动这个服务了，只需要执行
```bash
cd /fate/python/fate_flow
python fate_flow_server.py
```
就可以了，这样服务启动后就会在终端显示服务信息，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125153736793.png#pic_center)
光标停在服务进程中没有返回，服务正在后台运行，等待请求，一旦发生请求这里就会打印服务信息，因此可以另外打开一个终端重新进入这个容器并发起请求。

前面提到的Pdb调试方法必须通过终端打印出来才能进行交互式的调试，现在服务信息也可以在终端打印出来了，也就是说只要把Pdb的断点设置在服务进程里面并重启服务进程就可以对服务进程进行调试了，以下提交一个submit job来演示。

大概理解fate_flow_server.py的代码之后，易知submit请求后会通过路由到/fate/python/fate_flow/apps/job_app.py里面的submit_job函数，在这个函数里面设置断点，如下所示，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125155403262.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
保存并退出后，先杀掉当前服务进程，通过ps -ef查看PID，然后kill -9 PID，再重新手动运行fate_flow_server.py即可。

然后执行submit请求如下：
```bash
cd /fate/python/fate_flow
python fate_flow_client.py -f submit_job -c /fate/examples/dsl/v1/hetero_secureboost/test_secureboost_train_binary_conf.json -d /fate/examples/dsl/v1/hetero_secureboost/test_secureboost_train_dsl.json
```
在服务进程信息终端会看到程序停在了Pdb设置的断点处，如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125160444511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
接下来就可以正常使用Pdb调试程序了。

### 服务中存在嵌套请求现象，手动启动服务也没用，通过有名管道来调试后台进程
本以为调试的问题通过死循环顶替服务进程自动启动而巧妙的解决了，没想到调试到后面发现在服务中也存在request请求，也就是说存在请求和服务的嵌套？不太清楚这后面的请求和服务的架构到底是怎么样的，我只是想调试看看算法部分的代码，通过FATEBoard的log可以看出哪些Py脚本是与算法相关的，比如在hetero_secureboost例子里面，从log可以看出有执行boosting.py这个文件，但是直接用Pdb在boosting.py里面设置断点，在服务端终端并没有显示出Pdb的调试交互提示，而整个服务会一直停在那里，也许这是请求服务的嵌套引起的，我不知道从submit的请求到boosting.py中间有多少次请求嵌套，也或许不是请求嵌套而是别的某种方式触发的算法代码的执行，反正不管怎么样这些肯定都是在后台执行的进程，于是有了通过Pdb和有名管道研究Python多进程的调试方法。

最后研制出来了mpdb工具，使用起来也比较简单，和使用Pdb一样，我在个人的github上建了一个仓库：https://github.com/huaqing89/mpdb

#### 使用方法和使用效果
以调试boosting.py为例，还是以提交hetero_secureboost的例子来说明，提交的主要命令如下：
```bash
cd /fate/python/fate_flow
python fate_flow_client.py -f submit_job -c /fate/examples/dsl/v1/hetero_secureboost/test_secureboost_train_binary_conf.json -d /fate/examples/dsl/v1/hetero_secureboost/test_secureboost_train_dsl.json
```
可以知道这是在提交submit请求，对于boosting.py来讲其主程序应该是服务程序fate_flow_server.py，fate_flow_server.py是在/fate/python/fate_flow下开始运行的，其主目录就是/fate/python/fate_flow，在我的mpdb工具里面有一个mpdb_cmd.py的文件，这个文件必须放在被调试程序的主目录下面，也就是说这是一个游走的文件，当需要被调试的程序的主目录发生变化需要把它移动到被调试程序的主目录下面，因此这里被调试的主程序是fate_flow_server.py，把mpdb_cmd先放进/fate/python/fate_flow中。

然后我们想调试的代码是在boosting.py中，执行
```bash
find / -name boosting.py 
```
找到boosting.py所在位置：/fate/python/federatedml/ensemble/boosting/boosting_core/boosting.py
在FATEBoard日志中有显示执行boosting.py的行号，如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125170520604.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
通过vim打开boosting.py具体到某一行，如下（比如167行）
```bash
vim +167 /fate/python/federatedml/ensemble/boosting/boosting_core/boosting.py
```
打开后光标停留的位置就是167行，也可以通过vim的底线命令模式输入:set nu显示行号，更直观。然后利用mpdb设置断点如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125171006819.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
可以看到mpdb设置断点和pdb设置断点几乎一模一样，就是先导入模块，然后调用set_trace()。断点设置完后保存退出再杀掉当前服务并重新手动启动服务，最后再提交submit请求。

请求提交完后返回信息如下表示请求提交成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125171800875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

此时服务信息一直停留在update的状态，如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125171926138.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

而FATEBoard的进度一直停留在66%，如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125172044652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

以上表明程序停在了断点处，如果我们用的是pdb设置断点，进度也是停在66%的地方，程序是中断了，但是pdb没有办法显示出调试的交互界面，所以没办法调试，但是现在我们用的mpdb，因此可以通过运行刚才放进/fate/python/fate_flow中的mpdb_cmd.py来获得调试交互界面，很简单，就是运行如下命令
```bash
cd /fate/python/fate_flow
python mpdb_cmd.py
```
运行后就可以获得pdb的交互调试界面了，如下所示，剩下的调试命令就是pdb的调试命令，和pdb一模一样的，mpdb背后就是调用的pdb模块。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125172654608.png#pic_center)
输入"l"可以看到如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125172939319.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

#### mpdb的原理
m代表Multiprocess，全部合起来就是Multiprocess Python Debugger. 核心文件就3个py文件如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125173835753.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

mpdb.py内容如下所示，主要是导入了pdb模块，利用了pdb模块利用管道进行进程间通信的功能。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125174038717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

mpdb_cmd.py内容如下，主要是管道另一端的数据读取、打印和写入等。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125174338677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

install.py内容如下，主要是在Python的sys.path中寻找pdb.py所在目录，并把mpdb.py放进pdb.py所在目录。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210125175825333.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)



