Windows中利用pycharm连接docker的挂载文件，进而在本地修改FATE代码
=============
<br/>


# 一、 Docker容器的挂载
拿FATE单机版（docker部署）举例：https://github.com/FederatedAI/FATE/blob/master/standalone-deploy/doc/Fate-standalone_deployment_guide_zh.md
<br/>
![docker单机版部署](https://img-blog.csdnimg.cn/2020080522491512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
<br/>
关于docker和docker-compose的安装不再赘述，这里默认已经下载好上图中的压缩包并且解压好了。
![2](https://img-blog.csdnimg.cn/20200805224946720.png)

进入后有上图中的几个文件，其中docker_standalone.yml便可以配置我们的容器文件的挂载，但是在经过数次尝试后，会发现尽管容器的挂载已经成功，但是本地的挂载目录下仍为空。
解决此问题的方法是我们可以先将容器对应的目录拷贝到本地，然后再进行docker_standalone.yml的配置。
## `step1:先执行install_standalone_docker.sh将容器安装好`
命令为：bash install_standalone_docker.sh
执行后使用 docker ps 检测一下是否成功：
![3](https://img-blog.csdnimg.cn/20200805225013292.png)
我本次的docker_python的容器ID为14c1d03c062f （后边使用14c简写），我们看一下容器内的目录：
![4](https://img-blog.csdnimg.cn/20200805225024837.png)

我希望将 /fate 整个目录都挂载到我的服务器本地，在我们使用exit退出容器后，在当前目录下可以创建一个 my_fate 目录（名字可以自己定），然后进入该目录
![5](https://img-blog.csdnimg.cn/20200805225042872.png)

## `step2：使用docker cp命令将容器中的目录copy到本地`

![6](https://img-blog.csdnimg.cn/20200805225054341.png)

这时候上图中的fate就和我们在容器中看到的fate目录是一致的了

## `step3：修改docker_standalone.yml文件后重新执行install_standalone_docker.sh`
如下图，使用 vi docker_standalone.yml进入后添加绿色框这一行内容，并保存
![7](https://img-blog.csdnimg.cn/20200805225113900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)


然后重新执行 bash install_standalone_docker.sh后
使用docker ps查看当前的容器id（有可能容器id和刚刚不同）
![8](https://img-blog.csdnimg.cn/20200805225153458.png)
可见现在我的容器id是b86bdef20d95（简称b86）
## `step4：验证是否挂载成功`
使用docker inspect b86查看挂载情况，找到Mounts字段如下就代表已经成功
![9](https://img-blog.csdnimg.cn/20200805225258975.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)

这时候我们修改 my_fate下的fate目录中的文件时，容器中也会相应的改变，如我们添加一个test.txt并写上hello world后容器也会多出一条：
![10](https://img-blog.csdnimg.cn/20200805225352891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
# 二、 pycharm连接linux服务器
本节中进行的是我们在windows本地如何连接刚刚的挂载路径，从而达到本地pycharm上直接修改容器中的代码，而不用再在黑黑的vi中编辑。

## `step1：下载一个专业版的pycharm`
专业的我们必然要用专业的工具，主要是社区版不太得行。专业版pycharm的下载我是通过学生邮箱直接注册好便可下载，至于是否还有其余下载途径还请自行摸索。

## `step2：创建新项目并连接服务器的指定目录（详细步骤如下）`
### （1）创建project
![11](https://img-blog.csdnimg.cn/20200805225414407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
### （2）设置interpreter
![12](https://img-blog.csdnimg.cn/20200805225432873.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
![13](https://img-blog.csdnimg.cn/2020080522544910.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
![14](https://img-blog.csdnimg.cn/20200805225509321.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
填好你的服务器地址端口名字后下一步
![15](https://img-blog.csdnimg.cn/2020080522552418.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
![16](https://img-blog.csdnimg.cn/20200805225536958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
到这步时候，红色框里就是指配对服务器的哪个目录，我们选择刚刚配置的那个挂载目录即可
![17](https://img-blog.csdnimg.cn/20200805225551194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)

这个文件夹的图标你放上去才能显示出来

![18](https://img-blog.csdnimg.cn/20200805225604198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)

![19](https://img-blog.csdnimg.cn/20200805225616353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)

我选择好刚刚创建的my_fate下的fate目录后确定，然后一路ok、finish就好了

## `step3：download和upload`
无论是上传还是下载，我们都需要选中某个文件或文件夹才有效，比如我们现在项目中什么也没有时候，我们可以选择项目名后download即可
![20](https://img-blog.csdnimg.cn/20200805225630178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
## `step4：友情提示区`
### (1)  自动上传
![21](https://img-blog.csdnimg.cn/20200805225647367.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
![22](https://img-blog.csdnimg.cn/20200805225703686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
点进去后用这个小对勾选择你要选择的默认服务器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805225719994.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)

再把这个选上后，每次修改后使用ctrl+s保存即可自动上传，十分方便
### （2）这个选项不取消有可能会报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805225734350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805225748422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
## `（3）连接的目标目录有可能文件有读写权限导致无法上传，注意自行修改`

到此为止，本教程结束！
本教程的功能主要是方便在本地pycharm中修改代码并上传和下载，如果要运行还需使用fate_flow提供的api，但本人在此建议可以简单的写一个脚本，这样可以达到重复利用，在此提供范例：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805225801776.png)

```xml
python /fate/fate_flow/fate_flow_client.py -f upload -c /fate/examples/federatedml-1.x-examples/homo_logistic_regression/upload_data_guest.json
python /fate/fate_flow/fate_flow_client.py -f upload -c /fate/examples/federatedml-1.x-examples/homo_logistic_regression/upload_data_host.json
python /fate/fate_flow/fate_flow_client.py -f submit_job -d /fate/examples/federatedml-1.x-examples/homo_logistic_regression/test_homolr_train_job_dsl.json -c /fate/examples/federatedml-1.x-examples/homo_logistic_regression/test_homolr_train_job_conf.json
```
## 三、 关于集群版KUBEFATE的挂载（使用docker的话）
tips：集群版的容器confs-xxxx_python_1生成时候就已经自动挂载过了，我们可以直接通过docker inspect命令查看他的挂载目录并用pycharm连接（步骤同上）
