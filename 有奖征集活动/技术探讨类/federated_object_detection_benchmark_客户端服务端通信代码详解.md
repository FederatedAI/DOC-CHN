# 论文《Federated-Benchmark: A Benchmark of Real-world Images Dataset for Federated Learning》客户端服务端通信过程代码详解

## 代码目录：FATE/research/federated_object_detection_benchmark/

### 本文解析：client的行为写在fl_client.py中，server的行为写在fl_server.py中。整个框架中，client和server通过socket IO通信。

<br>

Client首先通过__init__ ()函数向客户端发送‘client_wake_up’, 告诉server客户端已经唤醒。

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

Client收到request_update后通过on_request_update(*args)函数执行本地训练。

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


