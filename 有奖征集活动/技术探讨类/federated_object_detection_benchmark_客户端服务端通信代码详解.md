# 论文《Federated-Benchmark: A Benchmark of Real-world Images Dataset for Federated Learning》客户端服务端通信过程代码详解

## 代码目录：FATE/research/federated_object_detection_benchmark/

### 本文解析：client的行为写在fl_client.py中，server的行为写在fl_server.py中。整个框架中，client和server通过socket IO通信。


# 初始化阶段

## 客户端hanlder注册
Client中__init__()函数中调用register_handlers, 示意图如下, 下图列出了所有在监听的handler
<img src="https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/aa.png">

其中分别注册了connect, disconnect, reconnect, init, request_update, stop_and_eval, check_client_resource 事件，对应的触发条件是self.sio.on函数的第一个参数，具体作用如下：

* connect 事件当连接时被调用，打印connect指示当前已经连接到服务器
* disconnect 事件当断开服务器时被调用，打印connect指示当前已断开
* reconnect 当重新连接到服务器时候被调用，指示已经重新连接到服务器
* init 当第一次连接到服务器的时候调用， 是client_wake_up的回调， 指示wakeup成功，开始加载本地模型，并通知服务器加载成功
* request_update 服务器发送reqest_update的时候调用， 在本地训练一轮并将参数使用pickle转化后再转化为socketio能够通讯的string格式发送给服务器
* stop_and_eval 在本函数中接收来自服务器的聚合模型，并使用本地的数据集评估后将评估结果发送给服务器
* check_client_resource 的作用是检查本地资源负载情况并发送给服务器， 以便于服务器调整资源策略

## 服务器handler注册

Server中__init__调用register_handler来注册处理器，示意图如下图：
<img src="https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/bb.png">

* connect 打印了客户端请求的sid，指示已经连接
* disconnect 打印客户端sid并将客户端从客户端列表中移除
* reconnect 打印重新连接的客户端sid
* client_wake_up 相应客户端唤醒的请求并命令客户端进行初始化
* client_ready 将客户端加入客户端列表， 并且当客户端数目达到要求的时候，开始check客户端的资源
* check_client_resource_done 客户端资源负载统计完毕，发回的请求，如果负载小于一定值，就将客户端加到一个选定客户端列表，如果超过半数的客户端达到要求，则开始训练， 否则将继续等待并检查客户端资源，直到要求满足
* client_update 收到客户端发过来的权值信息并聚合，聚合完毕后将权值发送给客户端，并指示客户端进行评估
* client_eval 收到客户端的评估指标并进行聚合， 计算服务器的loss等指标，计算完成后，进入下一轮训练




# 通信流程
Client通过__init__ ()函数向客户端发送‘client_wake_up’, 告诉server客户端已经唤醒。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/1.jpg>

server接收到消息后，发送'init',通知客户端初始化。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/2.jpg>

client接收到初始化消息后，通过on_init()函数进行本地模型初始化，并发向server发送'client_ready'，告诉server我准备好了。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/3.jpg>
<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/4.jpg>
<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/5.jpg>

客户端收到当前轮数后，通过on_check_client_resource(*args)检查自身负载是否满足条件，并返回客户端资源检查完毕的消息。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/6.jpg>

Server接收到消息后，如果框中部分大于0.5进入下一轮训练，若小于0.5则返回去检查客户端资源。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/7.jpg>

Server通过train_next_round()函数将聚合后的模型传给client，并通知client开始request_update更新本地模型。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/8.jpg>

Client收到request_update后先通过register_handles()函数中的on_connect()等函数打印客户端与服务端的连接状态，

<img src=https://raw.githubusercontent.com/Catherineylp/FATE-/master/client_regiter_handle.png>

然后通过on_request_update(*args)函数执行本地训练。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/9.jpg>

并将本地更新后的当前轮数，参数，训练集大小，loss传给server，

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/10.jpg>

同时通知server ‘client_update’。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/11.jpg>

Server收到本地更新的参数后，通过handle_client_update()函数执行参数聚合。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/12.jpg>

并通过stop_and_eval()函数向client发停止训练、开始验证的消息，进入验证阶段。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/13.jpg>

Client收到验证通知后，通过on_stop_and_eval(*args)函数基于本地验证集验证，并将验证结果返回给server。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/14.jpg>

Server接收到消息后，将所有客户端的loss，map，recall进行聚合得到全局模型的loss, map, recall。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/15.jpg>

并判断当前轮数是否大于最大轮数，若大于停止联邦学习过程，否则继续循环整个过程。

<img src=https://raw.githubusercontent.com/Catherineylp/federated_object_detection_benchmark_CodeAnalysis/master/16.jpg>
