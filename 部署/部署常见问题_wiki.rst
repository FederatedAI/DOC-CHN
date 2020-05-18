**1. 部署FATE，CentOS是必须吗 可以用Debian之类的其他linux发行版吗**

当前发布版本包括代码以及部署脚本针对centos做了比较完备的测试，不过系统没有要求特定的发行版特性，理论上其他发行版也可以运行，可能需要定制修改依赖包以及部署脚本，还可以使用Docker版本.

**2. cluster 部署，服务器要求：16core/32G
memory，这是建议配置，还是最低配置呢？**

建议生产配置，根据数据量决定，如果只是简单小数据量测试，4核8G就够，如果实际使用，建议尽量保持16核32G.

**3. 部署完后，按文档说明进行测试的时候出了“‘encoding’ is an invalid
keyword argument for this function”？**

python版本不对，请安装官网要求的python版本

**4.
单方，多方之间部署，需要ssh免密、mysql访问授权、以及关闭防火墙吗？**

单方和多方的部署都需要这些操作的，执行机器到待部署节点需要做免密，mysql需要授权给fateflow，roll所在服务器的访问权限

**5.
如果我不想用默认的app用户，以及相应的目录，只用改这个allinone_cluster_configurations.sh配置文件就可以了吧？我用自己的用户做免密，且这个用户有sudo权限，然后改allinone_cluster_configurations.sh配置文件就可以了是吧？**

是的，改这个allinone_cluster_configurations.sh配置文件即可

同时部署前做免密是和这个非app用户做免密，这个用户要有sudo权限

**6. toy测试显示任务处于waiting状态，问题可能如下:**

-
检查redis是否是否启动或者配置是否有问题，如果redis有问题，fateflow的日志里会有报错。

- Fateflow最多支持5个任务同时在跑，可以按如下方法把任务杀掉：

python fate_flow_client.py -f query_job -s waiting \| grep f_job_id \|
awk ‘{print $2}’ \| awk -F ‘"’ ‘{print $2}’ \| xargs -n1 python
fate_flow_client.py -f stop_job -j

- 重启fateflow

**7. C++编译报错(storage-service-cxx多次启动失败,且报错显示Illegal
instruction(core dumped))**

执行以下命令:

wget
https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/third_party_source.tar.gz

tar -xzf third_party_source.tar.gz

cd third_party/rocksdb

PORTABLE=1 make shared_lib

cp librocksdb.so.6.1.2
/data/projects/fate/eggroll/storage-service-cxx/third_party/rocksdb/librocksdb.so.6.1.2

然后重启storage-service-cxx服务

**8. processor没有成功启动**

排查方法：

- 检查egg.properties的配置；

- 检查用户有没有修改过services.sh，因为有一个配置应该是需要修改的。

- 在eggroll-egg.log中找到start
cmd，设置好PYTHONPATH后，单独拎出来运行看有没有问题。

**9.
如果重新部署，本机除了fate没有其他服务，可以把所有服务都kill掉，以免残留进程影响:**

ps -ef|grep java \| awk ‘{print $2}’\|xargs kill -9

ps -ef|grep python \|awk ‘{print $2}’\|xargs kill -9

ps -ef|grep process \|awk ‘{print $2}’\|xargs kill -9

ps -ef|grep redis \|awk ‘{print $2}’\|xargs kill -9

ps -ef|grep storage-serv \|awk ‘{print $2}’\|xargs kill -9

ps -ef|grep mysqld \|awk ’{print $2}

**10. toy测试报错: TypeError: must be real number, not NoneType**

原因可能是,guest拿host的结果为None，host的eggrol存在问题。

**11. FATE离线模块端口介绍**

federation联邦通讯（9394），proxy通讯转发（9370），fateboard可视化展示（8080），roll作业提交和数据汇总模块（8011），meta-service元数据存储模块（8590），storage-service-cxx节点存储模块（7778），egg用户自定义模块（7888），fate_flow任务管理模块（任务训练/上传下载数据，发布模型）-9360/9380。

**12. FATE离线服务管理**

a.根据配置，对应IP主机启动、停止、重启、查看对应的服务

b.根据实际情况执行cd /data/projects/fate && sh services moudle_name
start \| stop \| restart \| status

moudle_name参考11.介绍

c.启动服务和查看端口监听状态

**storage-service-cxx服务:**

cd /data/projects/fate/storage-service-cxx && sh service.sh restart &&
netstat -antp \| grep -i “listen” \| grep 7778

**egg服务：**

cd /data/projects/fate/egg && sh service.sh restart && netstat -antp \|
grep -i “listen” \| grep 7888

**roll服务：**

cd /data/projects/fate/roll && sh service.sh restart && netstat -antp \|
grep -i “listen” \| grep 8011

**meta-service服务：**

cd /data/projects/fate/meta-service && sh service.sh restart && netstat
-antp \| grep -i “listen” \| grep 8590

**federation服务：**

cd /data/projects/fate/federation && sh service.sh restart && netstat
-antp \| grep -i “listen” \| grep 9394

**fateboard服务：**

cd /data/projects/fate/fateboard && sh service.sh restart && netstat
-antp \| grep -i “listen” \| grep 8080

**proxy服务：**

cd /data/projects/fate/proxy && sh service.sh restart && netstat -antp
\| grep -i “listen” \| grep 9370

**fate_flow服务：**

cd /data/projects/fate/python/fate_flow && && sh service.sh restart &&
netstat -antp \| grep -i “listen” \| grep 9380

**13. toy测试排查思路**

先测对端：python
/data/projects/fate/python/examples/toy_example/run_toy_example.py 9999
10000 1

如果不通，再分别测试自己

a.guest端执行：python
/data/projects/fate/python/examples/toy_example/run_toy_example.py 9999
9999 1

b.host端执行：python
/data/projects/fate/python/examples/toy_example/run_toy_example.py 10000
10000 1

哪端不通就哪端有问题，检查下federation（9394）/fateboard（8080）/proxy（9370）/egg（7888）/

roll（7778）/meta-service（8590）/storage-service-cxx（7778）/fate_flow（9360/9380）是否正常

如果两端都通，检查下proxy服务以及路由配置（/data/projects/fate/proxy/conf/route_table.json）是否正确

**14. Permission denied**

权限不够，请检查当前用户是否拥有FATE的目录权限

参考命令：chmod -R app. /data/projects/fate

**15. No space left on device**

磁盘空间不足，请检查部署各个主机磁盘

参考命令：df -TH

**16. 端口不通**

如测试端口：telnet 192.168.0.1 8080

如果不通 请检查下防火墙策略

**17. Connection refused**

提示服务拒绝连接，请检查日志是否报错和服务是否已启动。

**18. fateflow服务起不来**

cd /data/projects/fate/python/fate_flow && python fate_flow_server.py

如出现连接不了mysql，先到目标主机确认mysql服务是否已经启动：lsof
-i:3306，ps -ef \| grep mysql

如没有就启动cd /data/projects/fate/common/mysql/mysql-8.0.13执行sh
service.sh start

检查下/data/projects/fate/python/fate_flow/settings.py中的DATABASE连接信息是否正确

检查下各个redis mysql egg roll storage-service-cxx
meta-service服务是否启动

（fate_flow依赖于MySQL redis egg roll storage-service-cxx
meta-service，只有这些模块启动成功，fate_flow才会正常运行）

**19. 日志提示8011 7888-java.lang.reflect……**

修改eggroll/egg/egg.properties中的eggroll.computing.processor.session.max.count=16的值修改成不大于该机器CPU个数，重启egg服务。

**20. proxy修改路由不更新**

修改路由配置信息 重启proxy服务

**21. 使用新数据库不使用fate自带的mysql**

第一步执行部署:

git clone https://github.com/FederatedAI/FATE.git

cd FATE/cluster-deploy/scripts

bash packaging.sh

bash deploy_cluster_multinode.sh build all

第二步执行停止mysql服务：

cd /data/projects/fate/common/mysql/mysql-8.0.13

sh service.sh stop

kill -9 \`lsof -i:3306 \| grep -i “LISTEN” \| awk ‘{print $2}’\`

第三步改配置文件:

1）修改/data/projects/fate/fateboard/conf/application.properties中“spring.datasource.url”-IP地址、端口、数据库名

“spring.datasource.username”-用户名、“spring.datasource.password”-密码

2）修改/data/projects/fate/python/fate_flow/settings.py中的DATABASE连接信息的IP地址、端口、数据库名、用户名密码

3）根据实际修改，如果不能建库，请修数据库名改为实际数据库

第四步引入其他mysql服务：

1）用户名和密码与上面保持一致，可以不用有建库权限，但必须要有create
insert table权限

2）拷贝创建数据库表文件、执行插入语句：

cd FATE/eggroll/framework/meta-service/src/main/resources

把create-meta-service.sql拷贝到mysql主机上

如果不能建库，但是要有对应的数据库，需要修改create-meta-service.sql中的数据库名

修改eggroll_meta为对应自己的数据库：如sed -i
‘s/eggroll_meta/实际数据库/g’ create-meta-service.sql

mysql -hip地址 -u用户名 -p进入mysql执行source create-meta-service.sql

use 实际数据库；

INSERT INTO node (ip, port, type, status) values (‘roll服务所在IP’,
‘8011’, ‘ROLL’, ‘HEALTHY’);

INSERT INTO node (ip, port, type, status) values (‘proxy服务所在IP’,
‘9370’, ‘PROXY’, ‘HEALTHY’);

以下可能有多个，需要根据实际配置插入数据库表

INSERT INTO node (ip, port, type, status) values (‘egg服务所在IP’,
‘7888’, ‘EGG’, ‘HEALTHY’);

INSERT INTO node (ip, port, type, status) values
(‘storage-service-cxx服务所在IP’, ‘7778’, ‘STORAGE’, ‘HEALTHY’)

第五步重启服务

cd /data/projects/fate && sh services.sh all restart

注意：忽略终端显示的mysql启动错误

**22. 集群版服务管理注意事项**

a.根据部署配置，对应机器启对应服务，不要执行all。

b.什么用户发布就什么用户启动服务，使用root发布就使用root启动服务，非root用户把进程kill
-9干掉，如果是僵尸进程，找到对应的父进程kill掉，然后重启。

c.涉及权限的问题，改属主，建议chown -R app. /data/projects。

d.如何查看端口监听，建议netstat -antp \| grep -i “listen” \| grep
上述端口。

e.启动服务建议不要执行all，参考3.一个个启动，不要图省事。

f.每个服务启动过程，都会有日志记录，见模块下logs。

e.fate_flow依赖于MySQL redis egg roll storage-service-cxx
meta-service，只有这些模块启动成功，fate_flow才会正常运行。

**23. 单机docker install模式部署**

**1.注意事项**

环境条件：docker和docker-compose环境要安装好

部署用户：root或者具有sudo权限的普通用户，普通用户需要加入到docker组

端口监听：fateboard-8080 fateflow-9360/9380 查看是否监听参考netstat
-antp \| grep 端口号

**2.部署发布**

2.1 按照官方GitHub文档进行发布

#获取安装包

wget
https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/docker_standalone-fate-1.3.0.tar.gz

tar -xvf docker_standalone-fate-1.3.0.tar.gz

#执行部署

cd docker_standalone-fate-1.3.0

root用户执行：bash install_standalone_docker.sh

普通用户执行：bash install_standalone_docker.sh

2.2 执行docker-compose发布

cd docker_standalone-fate-1.3.0

docker load < python.tar

docker load < fateboard.tar

mkdir -p fate/data

mkdir -p fate/log

tar xvf data.tar.gz -C fate

启动：nohup docker-compose -f docker_standalone.yml up &

停止： docker-compose -f docker_standalone.yml down

2.3 单机版使用docker发布

前提条件：镜像已经导入成功

docker run -it -p 8080:8080 –name docker_fateboard docker_fateboard

docker run -it -p 9360:9360 -p 9380:9380 –name docker_python
docker_python

**24. 执行脚本乱码**

安装dos2unix插件

如：执行下dos2unix test.sh

**25. 检查IP配置**

a.检查操作系统环境：/etc/hosts文件、主机名、域名

b.检查数据库

/data/projects/fate/common/mysql/mysql-8.0.13/bin/mysql -ufate_dev -p -S
/data/projects/fate/common/mysql/mysql-8.0.13/mysql.sock

输入密码后执行

select from eggroll_meta.node；看是否有ip端口信息

如果没有执行

INSERT INTO node (ip, port, type, status) values (‘roll服务所在IP’,
‘8011’, ‘ROLL’, ‘HEALTHY’);

INSERT INTO node (ip, port, type, status) values (‘proxy服务所在IP’,
‘9370’, ‘PROXY’, ‘HEALTHY’);

以下可能有多个，需要根据实际配置插入数据库表

INSERT INTO node (ip, port, type, status) values (‘egg服务所在IP’,
‘7888’, ‘EGG’, ‘HEALTHY’);

INSERT INTO node (ip, port, type, status) values
(‘storage-service-cxx服务所在IP’, ‘7778’, ‘STORAGE’, ‘HEALTHY’)

c.检查配置文件（IP地址和端口）

/data/projects/fate/eggroll/python/eggroll/conf/server_conf.json

/data/projects/fate/python/arch/conf/server_conf.json

d.检查下路由表

/data/projects/fate/proxy/conf/route_table.json

e.检查fateflow

/data/projects/fate/python/fate_flow/settings.py中的DATABASE信息

**26.1.2版本集群部署的四种方式有什么区别吗？**

a..allinone build是基于源码编译的，配置简化，一键发布；

b.allinone install是基于安装包的，不需要编译，配置简化；

c.多节点build基于源码编译的，配置服务自定义，根据实际需求错开服务部署；

d.多节点install基于安装包的，不需要编译，其他的同3。

**27.跨网络部署**

根据架构图，跨网络部署，A端对外端口是9370，B端对外端口是9370，开通防火墙哦，如果要部署exchange，那就开通到exchange节点防火墙哦，主要还是配置proxy路由表。还有网络带宽，因为传输数据。

**28.在单机版部署完后，怎么实现多个主机间的联合训练啊，服务器怎么部署**

a.建议使用集群版

b.单机就一台主机，服务角色就fateboard可视化展示面板， python
fateflow任务训练提交，集群，多台机器，角色很多。包括federation fateboard
proxy eggroll fateflow等

**29.9999单独跑toy可以，10000跑不通，几个服务也检查了都运行着。报错是这样的，这个能看出来是哪的问题么？**

日志提示“xxxx:7888…………There is no exception等”

1.2版本修改eggroll/egg/conf/egg.properties中的max.count改为4，然后重启egg
roll storage-service-cxx

**30.普通用户可以用fate吗**

单机主机版，支持普通用户，单机docker install
build需要root权限，另外普通用户需要加入到docker组，集群版支持。

**31.请问1.2版本和1.1版本的storage-service-cxx模块的third_party是一样的吗？部署1.2的时候遇到了跟1.1同样的问题，用1.1的third_party_source.tar.gz直接编译可以吗？**

不一样，如果不能启动storage-service-cxx，那就重新编译下第三库c++（参考第7问题）。

**32.请问cluster我新部署了一个节点，怎么让它加入之前的cluster中呢，这块需要在哪块改下配置吗？**

a.把partyA的proxy规划为exchange

b.修改partyC的路由信息，proxy/conf/route_table.json文件，默认路由指向partyA的IP和proxy端口

c.修改partyA的路由信息，proxy/conf/route_table.json文件，增加一条指向partyC的路由

**33.Fate必须部署在centos7.2或者ubantu16.04上面吗？centos6的兼容吗？**

是的，6应该会有问题，主要是依赖包的问题。

**34.集群部署，第5点“Configuration check”，是看哪些个文件？**

Github文档里面有个超链接：cluster-deploy/doc ，点击打开即可。

**35.请问我在7.3步Minimization test中执行完sh run.sh host
fast之后出现Connection
refused错误，是什么原因啊，我重启fate_flow服务之后仍然是报Connection
refuse的错。还有一个问题，task_manager的启动脚本的路径是哪里啊**

fateflow没有启动

需要检查9360和9380端口是否正常监听，应该是服务还没有启动

**36.FATE通过yum 安装的数据库,在跑auto-deploy.sh 报异常，我可以把 -S
/usr/mysql.sock  删掉么**

ps -ef \| grep
mysql，看看mysql.sock是否在usr目录下，正常默认安装的在/tmp目录下，不需要加-S参数

**37.我有两台主机，一台host（248），一台guest（249）。我在host上执行
auto-deploy.sh
后，为什么host的/data/projects/fate下面只有venv，而guest下面有很多。免密后，deploy成功了。 
现在host和guest的router_table.json文件内容不一致，有没有问题**

部署过程中是否有报错？是否有做免密？本机也需要自身对自身做免密，因为都是通过IP连的，看是否可以先做下免密然后再试下。

**38.如果两台服务器不相连，是否只需要开三个防火墙端口：一个9394federation、一个9360
fateflow、一个proxy9370**

proxy互通即可，但是每个party集群内部需要所有端口都可以互通。

**39.想问下cluster 得要2个party  1个服务器么？一定要centos
用Ubuntu可以不**

cluster需要两台服务器，每个party一台。可以用Ubuntu，这个部署脚本没测过，提供的脚本都是基于centos的，一些命令可能需要自己修改下。

**40. KubeFATE的pre-requisite就是
kubernetes的每个节点的docker里面有FATE的所有image是吗？**

需要外网环境，从docker hub上拉取镜像，然后本地worker节点都有镜像。

**41.请问下
单party部署，是指分别在两个服务器配置configurations.sh然后启动?**

一个就可以了。

**42.问一下“单party部署”  和 
“partyA+partyB同时部署”分别指的是什么？单party部署的话，没部署的一方如何提交数据啊？**

联合学习需要两方或者两方以上，单party部署指只部署一方，需要和其他方联合学习测试。A+B指的是同时部署两方，这两方可以进行联合学习。
是A也是B.

**43.我们重新部署的时候，有一台机器在sh /data/projects/fate/services.sh
all
status时，出现了启动python多进程过多，直接导致了系统崩溃，不知道是哪配置错了，另一台机器就是正常的。**

查看/tmp目录权限，看app用户是否有权限写入，没有权限的话需要修改。

**44.问个问题，我这边机器有点点问题，新建docker网络失败，部署docker版本的话，我能修改一些docker-compose.yml，把容器加到已经有的网络嘛？会不会影响各个容器直接的通信？**

可以的，不过建议解决新建docker网络失败的问题，应该docker的设置有问题。

**45.问一下，单方部署，指的就是但party部署吧。那么多方之间，还需要ssh免密、mysql访问授权、以及关闭防火墙吗？**

单方的部署是一样需要这些操作的，执行机器到待部署节点需要做免密，mysql需要授权给fateflow，roll所在服务器的访问权限。

**46.作为exchange的一方，角色是什么？ arbiter？guest？**

Exchange不作为任何角色，只是路由中转。

**47.如果采用A+B同时部署的方式部署集群，请问在SSH免密登录配置好后，两台服务器的部署步骤一定要严格遵守文档的顺序嘛？是否先部署好A再部署B
？**

同时部署就是A和B，部署脚本会顺序部署的。如果分别部署A和B，先部署哪个都可以。

**48.我这边有两台服务器f1和f2用来部署FATE集群版，f1与f2均可以wget及git
clone到代码，但是无法ping通外界网络。f1与f2之间也配备了SSH免密登录。请问这种情况下可以部署FATE环境嘛？或者说部署过程中要求的外网环境具体用来干什么的呢？**

拉取代码，java源码的构建需要外网拉取依赖报，拉取c++的依赖包，如果可以拉取，ping不通没关系。

**49.我部署的集群版，创建的mysql在登录fate_dev用户时需要指定ip登录，这个对启动fate_flow的服务有影响吗？ fate_flow的服务启不来，erroe.log中显示unknown
fate_flow database？**

部署的时候，数据库的路径填写是否正确？看看数据库是否可以正常登陆。

**50. submit_job时报9380拒绝连接。重启fate所有服务之后，fate_flow进程在，但是9380/9360端口都没开启。问一下，调整时间对fate有什么影响吗？9380为什么起不来了？重启fate_flow服务时，检查了fate_flow的日志，没发现什么异常**

把fate_flow的进程杀掉，然后重启下。

**51.目前在集群部署的时候,编译环节提示编译环境缺失库(liblz4.so.1),鉴于内网环境无法连接外网，需要提前进行安装，请教下，在哪里能找到编译所需要的依赖列表？**

sudo yum -y install gcc gcc-c++ make openssl-devel supervisor gmp-devel
mpfr-devel libmpc-devel libaio numactl autoconf automake libtool
libffi-devel snappy snappy-devel zlib zlib-devel bzip2 bzip2-devel
lz4-devel libasan lsof

**52.问一下，exchange可以多次转发嘛，比如这种情况，两方均有多个party，各自有exchange节点以便于自己测试，然后合作时各自的exchange的default.default项填对方，也就是：任务发起party的proxy->A
exchange->B exchange -> 任务合作party的proxy？**

可以的，路由信息配置（/data/projects/fate/proxy/conf/route_table.json）正确即可

**53.FATE可以支持Ubuntu for IoT这种linux系统么？**

当前发布版本包括代码以及部署脚本针对centos做了比较完备的测试，不过系统没有要求特定的发行版特性，理论上其他发行版也可以运行，可能需要定制修改依赖包以及部署脚本，还可以使用Docker版本.Ubuntu
for IoT没做过测试，他本身应该做过很多精简，支持的可能性不大。

**54.请问一下 这个准备工作里的网络互通
需要自己先设置成docker容器能跨主机通信，还是只要主机之间能互相ping通就行呀？**

1）需要可以ping通，ping通表示主机之间路由是通的。

2）主机之间还需要端口互通，部署机到待部署机需要通过ssh协议远程登陆部署，ssh默认端口是22，所以部署机到待部署机之间至少要保证ssh协议的端口是通的。

3）两台待部署机之间如果是两个party，通过9370端口通讯，也要保证两台主机9370端口是互通的。

**55.启动fateflow报错显示缺少eggroll模块**

当某节点没有部署eggroll时，需要把已部署eggroll的节点下/data/projects/fate/eggroll/python目录拷贝到该主机/data/projects/fate/eggroll即可。
