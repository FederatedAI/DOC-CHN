## FATEboard的使用排雷
![1](https://img-blog.csdnimg.cn/20200811104335491.png)<br>

- ##### 容器刚启动就挂掉（可能是内存不够）
- ##### 容器的访问地址不明确


## `一、容器刚启动就挂掉`
在我们开心的启动了docker容器后，发现Network error，如果用（shift+F5）刷新后页面甚至直接没了。这时候我们首先要 **考虑一下Fateboard的容器是否启动** 。
我们使用 docker ps 命令查看一下当前启动的容器，如果发现只有fate_python启动而fateboard的容器没有启动（如下）：
![2](https://img-blog.csdnimg.cn/20200811105752237.png)
则说明该容器挂掉了，那么我们可以打开fateboard的error.log，我这里记录的是一个比较常见的问题，就是我们可能使用虚拟机或者自己买的学生版云服务器时候，内存比较小，这时候这两个容器就无法一起启动。
打开error日志后在vi下匹配memory发现如下（应该就是内存不够的问题了）：
![3](https://img-blog.csdnimg.cn/20200811110200715.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
**解决方法：换一个内存够4G的环境重新部署即可**


## `二、容器的访问地址不明确`
当我们万事俱备了，那么我们应该在浏览器输入的访问地址是什么呢？


我将地址分为以下两个类别：

- 类别1 ：直接连接云服务器或者在内网直接连接实验室服务器
		 	
		 	这种情况我们直接在浏览器输入服务器的地址IP:8080即可访问

- 类别2 ：在家办公的2020我们想要返校连接校内服务器变得困难，所以我们采用反向代理（跳板机）连接校内服务器，这时候我们直接使用跳板机的 IP:8080 无法访问

			这种情况我推荐大家可以使用MobaXterm，如下图介绍：
			
![4](https://img-blog.csdnimg.cn/20200811111457332.png)
			下载好后点击上图红框进入设置界面后选择“New SSH tunnel”，根据下图填写地址等：
			![5](https://img-blog.csdnimg.cn/20200811111929704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqczk3NTU4NDcxNA==,size_16,color_FFFFFF,t_70)
上图中除了数字123以外其余都按照图填写即可，左边的8080代表本地的8080端口，右边的127.0.0.1和8080代表目标机的8080端口（也就是本地8080对应上了目标的8080，也可以自行更改）
数字1：跳板机的IP地址
数字2：跳板机的用户名
数字3：跳板机的端口号

配置好打开后，我们在浏览器中输入 localhost:8080 即可访问