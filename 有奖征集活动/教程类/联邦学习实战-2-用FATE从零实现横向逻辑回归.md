---
title: 联邦学习实战-2-用FATE从零实现横向逻辑回归
tags:
  - null
categories:
  - technical
  - null
toc: true
declare: true
date: 2021-08-13 16:05:31
---

# 一、安装FATE平台

FATE是微众银行开源的联邦学习框架

> 官方地址：https://fate.fedai.org
>
> 文档：https://fate.readthedocs.io/en/latest/_build_temp/standalone-deploy/doc/Fate-standalone_deployment_guide_zh.html
>
> https://github.com/FederatedAI/Practicing-Federated-Learning/tree/main/chapter05_FATE_HFL

<!-- more -->

> <font color='#e54d42'>注意：书本《联邦学习实战》使用的FATE是1.4.0版本，而我使用的是1.6.0版本（2021-8-13），文中的配置文件中的注释在使用时要去除</font>

采用在[主机安装FATE的安装模式](https://fate.readthedocs.io/en/latest/_build_temp/standalone-deploy/doc/Fate-standalone_deployment_guide_zh.html#id1)

官方文档中的命令都是用`sh`运行，因为本机的环境不同，所以采用`bash`

```shell
wget https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/standalone_fate_master_1.6.0.tar.gz

tar -xzvf  standalone_fate_master_1.6.0.tar.gz

cd standalone_fate_master_1.6.0
bash init.sh init
```

> 问题：service.sh: [[ not found
>
> 解决：修改`init.sh`文件中的`sh`命令为`bash`

启动成功后：

![vXfbnF](http://xwjpics.gumptlu.work/qinniu_uPic/vXfbnF.png)

测试：

```shell
cd standalone_fate_master_1.6.0
source bin/init_env.sh
bash ./python/federatedml/test/run_test.sh
```

会输出一系列的测试输出，最终：

![O5UQVi](http://xwjpics.gumptlu.work/qinniu_uPic/O5UQVi.png)

toy测试

```shell
python ./examples/toy_example/run_toy_example.py 10000 10000 0
```

运行起来之后可以看到：

![MdTqV9](http://xwjpics.gumptlu.work/qinniu_uPic/MdTqV9.png)

![KESqxK](http://xwjpics.gumptlu.work/qinniu_uPic/KESqxK.png)

控制台（`IP:8080`）：

![image-20210815163001353](http://xwjpics.gumptlu.work/qinniu_uPic/image-20210815163001353.png)

安装FATE-Client和FATE-Test

为方便使用FATE，我们提供了便捷的交互工具[FATE-Client](https://fate.readthedocs.io/en/latest/_build_temp/python/fate_client)以及测试工具[FATE-Test](https://fate.readthedocs.io/en/latest/_build_temp/python/fate_test).

请在环境内使用以下指令安装：

```shell
python -m pip install fate-client
python -m pip install fate-test
```

# 二、切分数据集

数据集：威斯康星州临床科学中心开源的乳腺癌肿瘤数据集

`from sklearn.datasets import load_breast_cancer`

为了模拟横向联邦建模的场景，我们首先在本地将乳腺癌数据集切分为特征相同的横向联邦形式，当前的breast数据集有569条样本，我们将**前面的469条作为训练样本，后面的100条作为评估测试样本**。

- 从469条训练样本中，选取**前200条作为公司A的本地数据**，保存为breast_1_train.csv，将**剩余的269条数据作为公司B的本地数据**，保存为breast_2_train.csv。
- 测试数据集可以不需要切分，两个参与方使用相同的一份测试数据即可，文件命名为breast_eval.csv。

`splitDataset.py`

```python
from sklearn.datasets import load_breast_cancer
import pandas as pd


breast_dataset = load_breast_cancer()

breast = pd.DataFrame(breast_dataset.data, columns=breast_dataset.feature_names)

breast = (breast-breast.mean())/(breast.std())

col_names = breast.columns.values.tolist()



columns = {}
for idx, n in enumerate(col_names):
	columns[n] = "x%d"%idx

breast = breast.rename(columns=columns)

breast['y'] = breast_dataset.target

breast['idx'] = range(breast.shape[0])

idx = breast['idx']

breast.drop(labels=['idx'], axis=1, inplace = True)

breast.insert(0, 'idx', idx)

breast = breast.sample(frac=1)

train = breast.iloc[:469]

eval = breast.iloc[469:]

breast_1_train = train.iloc[:200]



breast_1_train.to_csv('breast_1_train.csv', index=False, header=True)


breast_2_train = train.iloc[200:]

breast_2_train.to_csv('breast_2_train.csv', index=False, header=True)

eval.to_csv('breast_eval.csv', index=False, header=True)
```

> <font color='#e54d42'>**注意：官方地址中的csv文件是有问题的，要运行这个切割数据集自己生成新的数据集文件**</font>

# 三、利用FATE构建横向联邦学习Pipeline

用FATE构建横向联邦学习Pipeline，涉及到三个方面的工作：

- 数据转换输入

  将数据文件（csv、txt）等转换成为FATE独有的基础数据结构DTable，从而方便进一步的操作

  <img src="http://xwjpics.gumptlu.work/qinniu_uPic/8NZB8C.png" alt="8NZB8C" style="zoom:50%;" />

- 模型训练

  FATE以模型训练构建流水线pipeline

- 模型评估：（可选）

  评估模型性能的手段

> <font color='#39b54a'>如果只使用框架自带的模型，那么其实我们需要修改的就是配置文件即可</font>

## 3.1 数据转换输入

新建一个项目目录`chapter05`（我这里创建的叫`chapter05`因为对应书第五章内容，可自行修改）

将数据集放到该目录下的`data`目录中：

![2mPZUP](http://xwjpics.gumptlu.work/qinniu_uPic/2mPZUP.png)

完成数据转换，需要一个配置文件和一个启动程序：

### 配置文件

`upload_data.json`文件编写配置，内容为：

```json
{
  "file": "data/breast_1_train.csv",
  "head": 1,
  "partition": 16,
  "work_mode": 0,
  "table_name": "homo_breast_1_train",
  "namespace": "homo_host_breast_train"
}
```

字段说明：

1. file: 数据文件路径，**相对于当前所在路径**
2. head: 指定数据文件是否包含表头
3. partition: 指定用于存储数据的分区数
4. work_mode: 指定工作模式，0代表单机版，1代表集群版
5. table_name：转换为DTable格式的表名
6. namespace: DTable格式的表名对应的命名空间
7. backend: 指定后端，0代表EGGROLL， 1代表SPARK加RabbitMQ， 2代表SPARK加Pulsar （暂时未用到）

### fate-flow上传数据

使用fate-flow上传数据。从FATE-1.5开始，推荐使用 [FATE-Flow Client Command Line](https://fate.readthedocs.io/en/latest/_build_temp/python/fate_client/flow_client/README.html) 执行FATE-Flow任务。

> 提示：
>
> ```shell
> {                                                                                                                 
>  "retcode": 100,                                                                                               
>  "retmsg": "Fate flow CLI has not been initialized yet or configured incorrectly. Please initialize it before using CLI at the first time. And make sure the address of fate flow server is configured correctly. The configuration file path is: /ai/xwj/federate_learning/fate_dir/standalone_fate_master_1.6.0/venv/lib/python3.6/site-packages/flow_client/settings.yaml."                                                                                       
> }      
> ```
>
> 解决：先初始化flow

使用`flow`首先要初始化一下：

[flow init文档](https://fate.readthedocs.io/en/latest/_build_temp/python/fate_client/flow_client/README_zh.html#init)

Fate Flow 命令行初始化命令。用户可选择提供fate服务器配置文件路径或指定fate服务器ip地址及端口进行初始化。注意：若用户同时使用上述两种方式进行初始化，CLI将优先读取配置文件内容，而用户所配置的服务器ip地址及端口信息将被忽略。

因为之前init命令直接已经启动了flow，所以直接初始化即可

```shell
flow init --ip 127.0.0.1 --port 9380
```

```shell
{
    "retcode": 0,
    "retmsg": "Fate Flow CLI has been initialized successfully."
}
```

因为之前

上传命令会自动将原始的本地文件转为DTable格式

```shell
flow data upload -c upload_data.json
```

> <font color='#39b54a'>注意，我使用的是新版本，新版本支持命令行直接使用</font>

> 如果提示没有flow命令，那么可能没有使用提供的conda环境，解决如下：
>
> `source bin/init_env.sh`

显示结果：

```json
{
    "data": {
        "board_url": "http://127.0.0.1:8080/index.html#/dashboard?job_id=202108151639113656812&role=local&party_id=0",
        "job_dsl_path": "/home/jiangbo/FATE/standalone_fate_master_1.6.0/jobs/202108151639113656812/job_dsl.json",
        "job_id": "202108151639113656812",
        "job_runtime_conf_on_party_path": "/home/jiangbo/FATE/standalone_fate_master_1.6.0/jobs/202108151639113656812/local/job_runtime_on_party_conf.json",
        "job_runtime_conf_path": "/home/jiangbo/FATE/standalone_fate_master_1.6.0/jobs/202108151639113656812/job_runtime_conf.json",
        "logs_directory": "/home/jiangbo/FATE/standalone_fate_master_1.6.0/logs/202108151639113656812",
        "model_info": {
            "model_id": "local-0#model",
            "model_version": "202108151639113656812"
        },
        "namespace": "homo_host_breast_train",
        "pipeline_dsl_path": "/home/jiangbo/FATE/standalone_fate_master_1.6.0/jobs/202108151639113656812/pipeline_dsl.json",
        "table_name": "homo_breast_1_train",
        "train_runtime_conf_path": "/home/jiangbo/FATE/standalone_fate_master_1.6.0/jobs/202108151639113656812/train_runtime_conf.json"
    },
    "jobId": "202108151639113656812",
    "retcode": 0,
    "retmsg": "success"
}
```

访问`board_url`可以在浏览器中看到对应的详情：

![image-20210815164323044](http://xwjpics.gumptlu.work/qinniu_uPic/image-20210815164323044.png)

对于breast_2_train.csv以及测试集合同样的过程：

测试集上传到两个参与方，并各自转换为DTable格式

```json

{
  "file": "data/breast_2_train.csv",
  "head": 1,
  "partition": 16,
  "work_mode": 0,
  "table_name": "homo_breast_2_train",
  "namespace": "homo_guest_breast_train"
}

// host

{
  "file": "data/breast_eval.csv",
  "head": 1,
  "partition": 16,
  "work_mode": 0,
  "table_name": "homo_breast_1_train",
  "namespace": "homo_host_breast_eval"
}

// guest

{
  "file": "data/breast_eval.csv",
  "head": 1,
  "partition": 16,
  "work_mode": 0,
  "table_name": "homo_breast_2_train",
  "namespace": "homo_guest_breast_eval"
}
```



## 3.2 模型训练

借助FATE，我们可以使用组件的方式来构建联邦学习，而不需要用户从新开始编码，FATE构建联邦学习Pipeline是通过**自定义dsl和conf两个配置文件**来实现：

[DSL 配置和运行配置 V1](https://fate.readthedocs.io/en/latest/_build_temp/doc/dsl_conf_v1_setting_guide_zh.html)

- dsl文件：用来描述任务模块，将任务模块以有向无环图（DAG）的形式组合在一起。
- conf文件：设置各个组件的参数，比如输入模块的数据表名；算法模块的学习率、batch大小、迭代次数等。

我们本实验使用的是模型：`逻辑回归`

进入`examples/dsl/v1/homo_logistic_regression`文件夹中：

**使用`test_homolr_train_job_dsl.json`、`test_homolr_train_job_conf.json`两个文件来辅助构建横向联邦学习模型**

### DSL配置文件

为了让任务模型的构建更加灵活，目前 FATE 使用了一套**自定的领域特定语言 (DSL)** 来描述任务。在 DSL 中，各种**任务模块**（例如数据读写 data_io，特征工程 feature-engineering， 回归 regression，分类 classification）**可以通向一个有向无环图 （DAG） 组织起来**。通过各种方式，用户可以根据自身的需要，灵活地组合各种算法模块。

DSL 的配置文件采用 **json 格式**，实际上，整个配置文件就是一个 json 对象 （dict）。在这个 dict 的第一级是 “components”，用来表示这个任务将会使用到的各个模块。用户需要使用模块名加数字 _num 作为对应模块的 key，例如 dataio_0，并且数字应从 0 开始计数。

具体的参数见官方文档连接（上面）

`test_homolr_train_job_dsl.json`:

定义了四个模块已经构成了基本的联邦学习流水线（`feature_scale_0`为1.6.0新增），可直接使用(复制一份)

```json
{
    "components" : {
      	// 数据IO组件，用于将本地数据转为DTable
        "dataio_0": {  		
            "module": "DataIO",
            "input": {
                "data": {
                    "data": [
                        "args.train_data"
                    ]
                }
            },
            "output": {
                "data": ["train"],
                "model": ["dataio"]
            }
         },
      	// 特征工程模块
        "feature_scale_0": {
            "module": "FeatureScale",
            "input": {
                "data": {
                    "data": [
                        "dataio_0.train"
                    ]
                }
            },
            "output": {
                "data": ["train"],
                "model": ["feature_scale"]
            }
        },
      	// 横向逻辑回归组件
        "homo_lr_0": {
            "module": "HomoLR",
            "input": {
                "data": {
                    "train_data": [
                        "feature_scale_0.train"
                    ]
                }
            },
            "output": {
                "data": ["train"],
                "model": ["homolr"]
            }
        },
      	// 模型评估组件，如果未提供测试数据集则自动使用训练集
        "evaluation_0": {
            "module": "Evaluation",
            "input": {
                "data": {
                    "data": [
                        "homo_lr_0.train"
                    ]
                }
            }
        }
    }
}
```

这里的`args.train_data`是根据下面的运行配置`conf`来的

其次配置文件之间的指定关系如下：

![llixaM](http://xwjpics.gumptlu.work/qinniu_uPic/llixaM.png)

### 运行配置 Submit Runtime Conf

每个模块都有不同的参数需要配置，不同的 party 对于同一个模块的参数也可能有所区别。为了简化这种情况，对于每一个模块，FATE 会将所有 party 的不同参数保存到同一个运行配置文件（Submit Runtime Conf）中，并且所有的 party 都将共用这个配置文件。

除了 DSL 的配置文件之外，用户还需要准备一份运行配置（Submit Runtime Conf）用于设置各个组件的参数。

`test_homolr_train_job_conf.json` 修改如下：

```json
{		
  	// 创始人
    "initiator": {
        "role": "guest",
        "party_id": 10000
    },
    "job_parameters": {
        "work_mode": 0		// 0代表单机版，1代表集群版
    },
  	// 所有参与此任务的角色
  	// 每一个元素代表一种角色以及承担这个角色的party_id，是一个列表表示同一角色可能有多个实体ID
    "role": {
        "guest": [
            10000
        ],
        "host": [
            10000
        ],
      	// 仲裁者
        "arbiter": [
            10000
        ]
    },
  	// 对于权限角色的进一步参数配置
    "role_parameters": {
        "guest": {
            "args": {
                "data": {
                    "train_data": [
                        {
                            "name": "homo_breast_2_train",	 	// 修改DTable的表名
                            "namespace": "homo_guest_breast_train"	// 修改命名空间
                        }
                    ]
                }
            },
          	"dataio_0": {
              "label_name": ["y"]						// 添加标签对应的列属性名称
            }
        },
        "host": {
            "args": {
                "data": {
                    "train_data": [
                        {
                            "name": "homo_breast_1_train",				// 同理修改
                            "namespace": "homo_host_breast_train"
                        }
                    ]
                }
            },
          	"dataio_0": {
              "label_name": ["y"]						// 添加标签对应的列属性名称
            },
            "evaluation_0": {
                "need_run": [
                    false
                ]
            }
        }
    },
  	// 设置模型训练的超参数信息
    "algorithm_parameters": {
        "dataio_0": {
            "with_label": true,
            "label_name": "y",
            "label_type": "int",
            "output_format": "dense"
        },
        "homo_lr_0": {
            "penalty": "L2",
            "optimizer": "sgd",
            "tol": 1e-05,
            "alpha": 0.01,
            "max_iter": 10,
            "early_stop": "diff",
            "batch_size": 500,
            "learning_rate": 0.15,
            "decay": 1,
            "decay_sqrt": true,
            "init_param": {
                "init_method": "zeros"
            },
            "encrypt_param": {
                "method": null
            },
            "cv_param": {
                "n_splits": 4,
                "shuffle": true,
                "random_seed": 33,
                "need_cv": false
            }
        }
    }
}
```

关于角色，解释如下：

- arbiter是用来辅助多方完成联合建模的，它的主要作用是**聚合梯度或者模型**。比如纵向lr里面，各方将自己一半的梯度发送给arbiter，然后arbiter再联合优化等等。
- guest代表数据应用方。
- host是数据提供方。
- local是指本地任务, 该角色仅用于upload和download阶段中。

### 提交参数，训练模型 

Job `submit`

- *介绍*： 提交执行pipeline任务。
- *参数*：

| 编号 | 参数      | Flag_1 | Flag_2        | 必要参数 | 参数介绍                                                     |
| ---- | --------- | ------ | ------------- | -------- | ------------------------------------------------------------ |
| 1    | conf_path | `-c`   | `--conf-path` | 是       | 任务配置文件路径                                             |
| 2    | dsl_path  | `-d`   | `--dsl-path`  | 否       | DSL文件路径. 如果任务为预测任务，该字段可以不输入。另外，用户可以提供可用的自定义DSL文件用于执行预测任务。 |

执行：

```shell
flow job submit -c test_homolr_train_job_conf.json -d test_homolr_train_job_dsl.json
```

![dqV03m](http://xwjpics.gumptlu.work/qinniu_uPic/dqV03m.png)

![45Ss9k](http://xwjpics.gumptlu.work/qinniu_uPic/45Ss9k.png)

![COVndQ](http://xwjpics.gumptlu.work/qinniu_uPic/COVndQ.png)

> <font color='#e54d42'>注意：需要全部为勾号才说明运行成功</font>

可以看到运行的结果：

<img src="http://xwjpics.gumptlu.work/qinniu_uPic/JyApKE.png" alt="JyApKE" style="zoom:57%;" />

## 3.3 评估模型

为了评估模型的效果，我们重新修改配置以检查效果：

在上传数据时，我们已经将两个评估模型的数据上传，并且转换为DTable格式

现在我们修改`dsl`配置文件:

```json
{
    "components" : {
        "dataio_0": {
            "module": "DataIO",
            "input": {
                "data": {
                    "data": [
                        "args.train_data"
                    ]
                }
            },
            "output": {
                "data": ["train"],
                "model": ["dataio"]
            }
         },
        "dataio_1": {
            "module": "DataIO",
            "input": {
                "data": {
                    "data": [
                        "args.eval_data"
                    ]
                },
                "model": ["dataio_0.dataio"]
            },
            "output": {
                "data": ["eval_data"]
            }
        },
        "feature_scale_0": {
            "module": "FeatureScale",
            "input": {
                "data": {
                    "data": [
                        "dataio_0.train"
                    ]
                }
            },
            "output": {
                "data": ["train"],
                "model": ["feature_scale"]
            }
        },
        "feature_scale_1": {
            "module": "FeatureScale",
            "input": {
                "data": {
                    "data": [
                        "dataio_1.eval_data"
                    ]
                }
            },
            "output": {
                "data": ["eval_data"],
                "model": ["feature_scale"]
            }
        },
        "homo_lr_0": {
            "module": "HomoLR",
            "input": {
                "data": {
                    "train_data": [				// 注意这里是指定训练数据
                        "feature_scale_0.train"
                    ]
                }
            },
            "output": {
                "data": [
                    "train"
                ],
                "model": ["homolr"]
            }
        },
        "homo_lr_1": {
            "module": "HomoLR",
            "input": {
                "data": {
                    "eval_data": [				// 注意：这里是指定评估数据
                        "feature_scale_1.eval_data"
                    ]
                },
                "model": [								// 这里指定好模型
                    "homo_lr_0.homolr"
                ]
            },
            "output": {
                "data": [
                    "eval_data"
                ],
                "model": ["homolr"]
            }
        },
        "evaluation_0": {
            "module": "Evaluation",
            "input": {
                "data": {
                    "data": [
                        "homo_lr_0.train"
                    ]
                }
            }
        },
        "evaluation_1": {
            "module": "Evaluation",
            "input": {
                "data": {
                    "data": [
                        "homo_lr_1.eval_data"		// 指定评估数据集
                    ]
                }
            }
        }
    }
}
```

修改`conf`文件：

```json
{		
    ...
    "role_parameters": {
        "guest": {
            "args": {
                "data": {
                    "train_data": [
                        {
                            "name": "homo_breast_2_train",	 	
                            "namespace": "homo_guest_breast_train"		
                        }	
                    ],
                    "eval_data": [			// 添加测试集合
                        {
                            "name": "homo_breast_2_eval",
                            "namespace": "homo_guest_breast_eval"
                        }
                    ]
                }
            },
            "dataio_0": {
              "label_name": ["y"]					
            }
        },
        "host": {
            "args": {
                "data": {
                    "train_data": [
                        {
                            "name": "homo_breast_1_train",				
                            "namespace": "homo_host_breast_train"
                        }
                    ],
                    "eval_data": [		// 添加测试集合
                        {
                            "name": "homo_breast_1_eval",
                            "namespace": "homo_host_breast_eval"
                        }
                    ]
                }
            },
           ...
        }
    },
    ....
}

```

重新提交任务，查看结果：

```shell
flow job submit -c test_homolr_train_job_conf.json -d test_homolr_train_job_dsl.json
```

DAG图：

<img src="http://xwjpics.gumptlu.work/qinniu_uPic/7yZKiv.png" alt="7yZKiv" style="zoom:87%;" />

## 3.4 多参与方环境配置

上面的方案中有两个参与角色：

- guest代表数据应用方。
- host是数据提供方。

下面考虑更多的客户端参与，修改`conf`文件中的role添加更多的参与方：

（这里为了方便起见，就不再对数据集根据不同的host进行进一步的划分，测试集也是如此）

```json
{		
    "initiator": {
        "role": "guest",
        "party_id": 10000
    },
    "job_parameters": {
        "work_mode": 0	
    },
    "role": {
        "guest": [
            10000
        ],
        "host": [
            10000, 10001, 10002					// 新建两个Host
        ],
        "arbiter": [
            10000
        ]
    },
    "role_parameters": {
        "guest": {
            ...
        },
        "host": {
            "args": {
                "data": {
                  // 训练数据、测试数据的list都要与上面的host的list大小一样，一一对应
                    "train_data": [			
                        {
                            "name": "homo_breast_1_train",				
                            "namespace": "homo_host_breast_train"
                        },
                        {
                            "name": "homo_breast_1_train",
                            "namespace": "homo_host_breast_train"
                        },
                        {
                            "name": "homo_breast_1_train",
                            "namespace": "homo_host_breast_train"
                        }
                    ],
                    "eval_data": [
                        {
                            "name": "homo_breast_1_eval",
                            "namespace": "homo_host_breast_eval"
                        },
                        {
                            "name": "homo_breast_1_eval",
                            "namespace": "homo_host_breast_eval"
                        },
                        {
                            "name": "homo_breast_1_eval",
                            "namespace": "homo_host_breast_eval"
                        }
                    ]
                }
            },
          	// dataio_0 标签指定、测试集是否需要跑也是一样，大小一致、一一对应
            "dataio_0": {
              "label_name": ["y", "y", "y"]
            },
            "evaluation_0": {
                "need_run": [
                    false, false, false
                ]
            }
        }
    },
    ...
}
```

运行后效果如图：

![qceCFn](http://xwjpics.gumptlu.work/qinniu_uPic/qceCFn.png)

# Tips

## 1. 重新启动fate-flow和fateboard

在根目录下

```shell
bash ./python/fate_flow/service.sh restart
bash ./fateboard/service.sh restart
```

## 2. 'NoneType' object has no attribute 'count'

![RBWpmD](http://xwjpics.gumptlu.work/qinniu_uPic/RBWpmD.png)

解决：检查conf中的配置name、namespace与upload的配置是否一一匹配

## 3. list index out of range

![hC18Vd](http://xwjpics.gumptlu.work/qinniu_uPic/hC18Vd.png)

检查数据集分割（样本大小）是否处理好

