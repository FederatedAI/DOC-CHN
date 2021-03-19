# FederatedAI / DOC-CHN GitHub PR提交流程
## 1、Fork DOC-CHN仓库
1、找到Federated AI Ecosystem下的DOC-CHN仓库，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121123152719.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
2、点击上图中的DOC-CHN进入仓库，进入后再点击右上角的Fork，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121123848428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
3、点击完Fork，DOC-CHN仓库会自动拷贝一份进入自己名下的GitHub仓库下，页面自动跳转到自己的DOC-CHN仓库，如下图所示，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121130356380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

## 2、拷贝自己的DOC-CHN远程仓库到本地电脑
1、这里假定Git Bash已经安装好了，并且GitHub的ssh key等的相关配置都已经配置好了，在此不做赘述（笔者本地电脑为Win10环境）
2、在自己的DOC-CHN远程仓库界面按照图中箭头顺序依次点击之后就会复制自己的DOC-CHN仓库的ssh链接（此链接要用来把自己的GitHub DOC-CHN远程仓库拷贝到自己的本地电脑）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121124725371.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
3、找一个自己想要存放DOC-CHN仓库的目录，这里以PR-test为例，进入PR-test目录，在空白处点击鼠标右键，然后点击Git Bash Here，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121131854460.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
点击后会自动弹出Git Bash终端，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112113240881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
4、在终端里输入git clone ，然后点击鼠标右键，再点击Paste，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121132734149.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
点击完Paste，就把自己的DOC-CHN远程仓库的ssh链接复制进入了终端，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121132746175.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
然后，点击回车运行，稍等片刻，DOC-CHN远程仓库就会被拷贝到当前目录（这里是PR-test），过程如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121143932912.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121144027359.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

## 3、与Federated AI Ecosystem下的DOC-CHN远程仓库建立链接
1、查看本地DOC-CHN仓库与哪些远程仓库建立了链接，切换进本地DOC-CHN目录并输入如下命令

```shell
git remote -v 
```
如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121145313125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
可以看到，本地仓库只与自己的GitHub远程仓库建立了链接，没有与Federated AI Ecosystem下的DOC-CHN远程仓库建立链接。
2、切换到Federated AI Ecosystem下的DOC-CHN远程仓库界面并复制其ssh链接，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121145959776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121150431449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
3、在Git Bash中输入如下命令
```shell
git remote add upstream git@github.com:FederatedAI/DOC-CHN.git
```
其中"git@github.com:FederatedAI/DOC-CHN.git"就是刚才复制的Federated AI Ecosystem下的DOC-CHN远程仓库的ssh链接。
运行后，再执行

```shell
git remote -v
```
如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121151216631.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
此时就已经和Federated AI Ecosystem下的DOC-CHN远程仓库建立了链接。
## 4、新建分支、增添文件并提交到自己的GitHub远程仓库
1、输入如下命令新建一个名为new-branch-name的分支
```shell
git checkout -b new-branch-name
```
运行后会自动切换到新分支下（括号中为分支名），如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121152010257.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
也可以输入
```shell
git branch 
```
查看当前所在分支，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121152426107.png#pic_center)
2、增添文件
假设有文件要增添进"./有奖征集活动/教程类/"目录，可以用cp命令或直接鼠标复制把文件复制进"教程类"目录，然后可以输入如下命令查看当前新添文件的状态
```shell
git status -s
```
如下图（我这里新添的文件是test.txt，文件前面有两个问号说明此时文件还没有被添加进分支的暂存区）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121153959467.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
接下来执行如下命令之一进行添加
```shell
git add test.txt
```
或
```shell
git add .
```
第二个命令中的“.”代表添加所有当前目录下没有被添加的新文件到暂存区（有的时候要一次性添加很多文件）
添加完了之后再执行
```shell
git status -s
```
查看当前目录所有新添文件的状态，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121154641462.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
如上图所示，成功添加后的文件前面显示为“A”
如果我们直接对文件进行修改，修改后的文件的状态前面会多一个“M”，这时候需要再重新添加一下，如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121155107771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
3、提交
输入如下命令进行提交
```shell
git commit -m "message"
git push origin new-branch-name
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121163009908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
然后查看自己的GitHub仓库会发现多了一个new-branch-name分支，就是刚刚提交的，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121163233977.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
## 5、提交PR（Pull Request）
1、点击如下图箭头所示的Pull request 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121163612170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
点击后进入如下页面，看一下图中箭头所指的地方，一般不会有错
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121164037553.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
上图的网页下拉，如下图，红色箭头可以看到哪些文件发生了改变，哪些增添和删除的内容，点击黑色箭头所指的Create pull request就创建了一个PR
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121164405796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
点击上图黑色箭头所指的Create pull request后，页面就跳转到如下图所示，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121164640571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
然而，如果显示上图中的报错，点击DCO那一行右边的"Details"会进入如下页面
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121165130176.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
按照上图页面的提示在Git Bash先运行
```shell
git commit --amend --signoff
```
运行后会出现如下提示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121165841822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
输入“:q”退出提示，然后再输入如下命令

```shell
git push --force-with-lease origin new-branch-name
```
运行完如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121170237421.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
最后找到FederatedAI / DOC-CHN下面的刚才报错的页面，如下图，没有报错，全部通过，就成功向FederatedAI / DOC-CHN提交了PR。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112117062028.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)