# FATE-1.5.0v-docker单机版部署步骤
操作系统：centOS服务器
1、首先确保系统已经安装好docker和docker-compose
2、如下命令检查8080、9360和9380端口是否已被占用

```shell
netstat -tln
```
结果如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119164953393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
存在8080、9360和9380说明这几个端口已经被占用。如果确定不影响其他功能，可以将这几个端口释放，释放方法如下。
找到对应端口在系统中的进程 ID(PID)，输入如下命令：

```shell
 lsof -i :8080
 lsof -i :9360
 lsof -i :9380
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119170018306.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
使用 kill -9 [PID] 命令结束进程即可。最后可以再次用netstat -tln查看端口是否释放成功。

3、从微众银行官方下载安装包
如果服务器可以访问网络可以如下命令直接下载
```shell
wget https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/docker_standalone-fate-1.5.0.tar.gz
```
如果服务器不能访问网络或者网速太慢不想等，可以在本地浏览器直接放上上面的安装包链接就会自动下载，下载到本地后再从本地传到服务器上，这样会更快。

4、解压和部署
部署前确保docker服务是开启的，然后执行下面命令解压并部署。
```shell
tar -xzvf docker_standalone-fate-1.5.0.tar.gz
cd docker_standalone-fate-1.5.0
bash install_standalone_docker.sh
```
5、检查
部署完成后，容器会自动启动，并且已经做好了端口映射，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119175858122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

6、测试
（1）单元测试
```shell
docker exec -it fate_python bash
bash ./python/federatedml/test/run_test.sh
```
需要等待一段时间执行一系列的测试，最后，如果测试都成功，会显示
```
there are 0 failed test
```
如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119181324785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

（2）Toy测试
```shell
docker exec -it fate_python bash
python ./examples/toy_example/run_toy_example.py 10000 10000 0
```

如果成功如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119181626349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/202011191816369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

浏览器输入：http://hostip:8080  访问FateBoard，如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119182011858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)