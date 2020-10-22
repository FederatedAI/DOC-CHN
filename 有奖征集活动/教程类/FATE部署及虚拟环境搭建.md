# FATE部署及其虚拟环境创建
## 本文主要目的是为[FATE/research/federated_object_detection_benchmark/](https://github.com/FederatedAI/FATE/tree/master/research/federated_object_detection_benchmark)（详细的代码解析可以参考[链接](https://github.com/FederatedAI/DOC-CHN/pull/43/files)，对应论文：《Federated-Benchmark: A Benchmark of Real-world Images Dataset for Federated Learning》）搭建虚拟环境，并在虚拟环境中配置pytorch-gpu版本，使得该目录下的代码可以正常执行。

### 安装EPLE
```bash
sudo yum install epel-release -y
```

### 换yum源（阿里云）
```bash
sudo wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

### 安装python3.6
```bash
yum install python36 -y yum install python36-devel -y
```
### 查看Python版本以确认正确安装
```bash
python3
```

### 安装pip3
```bash
yum install python36-pip -y
```
### 查看pip3版本 
```bash
pip3 -v
```

### 安装virtualenv 
```bash
pip3 install virtualenv –user
```

### 创建目录存放代码 
```bash
mkdir CV
```
### 进入创建的目录，创建虚拟环境 
```bash
cd CV 
virtualenv -p python3 venv
``` 
### 进入虚拟环境 
```bash
source venv/bin/activate
```
<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/%E8%BF%9B%E5%85%A5%E8%99%9A%E6%8B%9F%E7%8E%AF%E5%A2%83.jpg>

### 查看cuda版本
```bash
nvcc -V
```
<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/%E6%9F%A5%E7%9C%8Bcuda%E7%89%88%E6%9C%AC.jpg>

### 装带GPU的pytorch
进入[pytorch官网](https://pytorch.org/)，选择对应的cuda版本，复制命令安装。

<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/pytorch%E7%89%88%E6%9C%AC.jpg>

（安装过程出现timed out，原因是源不稳定，多试几次）

<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/pytorch%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90.jpg>

### 测试pytorch(GPU)是否安装成功
<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/%E6%B5%8B%E8%AF%95pytorch%E6%98%AF%E5%90%A6%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.jpg>

能看到上述结果，说明pytorch gpu版本安装成功。

### 安装cupy
```bash
pip3 install cupy_cuda90 //如果服务器上cuda是9.0版本
```
安装过程会特别慢，建议换pip3的源（清华）。
```bash
pip3 install pip -U
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/%E6%8D%A2tuna%E6%BA%90.jpg>

### 从github上拖代码:
```bash
wget https://github.com/FederatedAI/FATE.git
```
<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/github%E6%BA%90%E7%A0%81.jpg>

### 解压代码 
```bash
upzip FATE.zip
```

### 安装依赖 
```bash
pip install -r requirements.txt
```

<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/%E5%AE%89%E8%A3%85%E4%BE%9D%E8%B5%96.jpg>

出现上述报错，安装gcc。

```bash
 sudo yum install gcc-c++
 ```
 安装gcc后再次安装依赖，安装成功如下图所示。
 
 <img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/%E4%BE%9D%E8%B5%96%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.jpg>
