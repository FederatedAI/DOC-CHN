# MySQL 异常导致 KubeFATE 部署失败的排查经历

今天通过 KubeFATE 部署节点时遇到了一个问题，经过一步步排除最终得以解决，在此记录一下整个过程。

我是跟着教程一步步做的，使用 docker-compose 方式部署两个节点。

- 先下载好镜像
- 然后编辑 `parties.conf` 文件
- 接着执行 `bash generate_config.sh`
- 最后执行 `bash docker_deploy.sh all `

按部就班没什么问题，等待执行结束后，分别在两个节点上 `docker ps` 查看，容器应该是都启动好了。

兴致勃勃进入容器 `docker exec -it confs-9999_python_1 bash`，准备进行操作，没想到几秒后就在容器里被 “弹” 出来了！

再 `docker ps` 一看发现这个 `confs-9999_python_1` 怎么重启了，难怪我被 “弹” 出来。又观察了一会儿，发现更不得了这个容器每隔几秒就重启一次，无限往复。

![](https://upload-images.jianshu.io/upload_images/7153030-419ba8b311d88d89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看到官方文档中好像有提到这种情况：
```
进入docker容器后马上又弹出来了。
解决办法：稍等一会再尝试。
因为python服务依赖其他所有服务的正常运行，然而第一次启动的时候MySQL需要初始化数据库，python服务的容器会出现几次重启，当MySQL等其他服务都运行正常之后，就可以正常执行了。

```

既然官方都说了，那就等一会儿呗。

等了足足 10 分钟，`python` 容器还是在无限重启，这肯定是有问题了，于是尝试查看 `python` 容器的日志输出，看看有没有报错。`docker logs -f confs-9999_python_1` 看到确实有错误提示，说是连接 mysql 的 `fate_flow` 库失败。

![image.png](https://upload-images.jianshu.io/upload_images/7153030-dc8afcc0d1bc423c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那就进 mysql 看看吧，在 `confs-9999_mysql_1` 容器中 `show databases` 发现果然没有 `fate_flow` 库。

![](https://upload-images.jianshu.io/upload_images/7153030-f4039e93998c36d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

尝试重新部署一下，看看 mysql 重新启动能不能将 `fate_flow` 库给初始化出来。
```
bash docker_deploy.sh --delete all
bash docker_deploy.sh all
```
发现问题依旧，再试了好几次，还是这样。mysql 里的初始化库没有建起来，导致 python 连接数据库时就不断失败报错重启。

最后干脆 `rm -rf  /data/projects/fate/` 将节点上容器用到的文件全删了。再次重新部署，成功了！所有容器都正常启动了起来。

**分析总结一下：**
- 猜测可能是第一次部署的时候发生了一些意外，导致 mysql 库没初始化成功。
- 虽然 mysql 没初始化成功，但是初始了一半的数据被保存在了 /data/projects/fate/confs-xxx/confs/mysql/data 目录中。
- 后面每次启动的时候数据都会被挂载到到 mysql 里去，这就导致虽然重新部署的服务，但 mysql 里没能重新初始化数据，一直是残缺的数据。
- 最后直接导致 python 启动的时候连接不到 mysql 里指定的库而不断报错重启。
- 解决方法就是把 data 删干净，重新部署，世界重新开始！


