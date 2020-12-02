# Standalone-Docker-FATE-1.5.0v torch横向正弦拟合
## 1、生成原始正弦实验数据
```shell
docker exec -it fate_python bash
cd /fate/examples/data
vim generate_sine_data.py
```
```python
import pandas as pd
import numpy as np
x = np.linspace(0, 64*np.pi, 50000)
y = np.sin(x)
pd.DataFrame(y.reshape(5000,10), columns=['x'+str(x) for x in range(9)]+['y']).to_csv('sine_data.csv', index=False)
```
## 2、数据横向切分
```shell
vim sine_data_homo_split.py
```
```python
import pandas as pd
data = pd.read_csv('sine_data.csv')
data.loc[:2499,:].to_csv('sine_data_guest.csv', index=False)
data.loc[2500:,:].to_csv('sine_data_host.csv', index=False)
```
## 3、配置数据上传json文件
```shell
cd /fate/examples/dsl/v1/
vim upload_sine_data_guest.json
```
```json
{
	"file": "examples/data/sine_data_guest.csv",
	"head": 1,
	"partition": 16,
	"work_mode": 0,
	"table_name": "sine_data_guest",
	"namespace": "experiment"
}
```
```shell
vim upload_sine_data_host.json
```
```json
{
	"file": "examples/data/sine_data_host.csv",
	"head": 1,
	"partition": 16,
	"work_mode": 0,
	"table_name": "sine_data_host",
	"namespace": "experiment"
}
```
## 4、数据上传
```shell
cd /fate/python/fate_flow/
python fate_flow_client.py -f upload -c /fate/examples/dsl/v1/upload_sine_data_guest.json
python fate_flow_client.py -f upload -c /fate/examples/dsl/v1/upload_sine_data_host.json
```
## 5、配置模型参数
```shell
cd /fate/examples/dsl/v1/homo_nn/
vim pytorch_homo_dnn_single_layer.json
```
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
            10000
        ],
        "arbiter": [
            10000
        ]
    },
    "role_parameters": {
        "guest": {
            "args": {
                "data": {
                    "train_data": [
                        {
                            "name": "sine_data_guest",
                            "namespace": "experiment"
                        }
                    ]
                }
            },
            "dataio_0": {
                "with_label": [
                    true
                ],
                "label_name": [
                    "y"
                ],
                "label_type": [
                    "float"
                ],
                "output_format": [
                    "dense"
                ]
            }
        },
        "host": {
            "args": {
                "data": {
                    "train_data": [
                        {
                            "name": "sine_data_host",
                            "namespace": "experiment"
                        }
                    ]
                }
            },
            "dataio_0": {
                "with_label": [
                    true
                ],
                "label_name": [
                    "y"
                ],
                "label_type": [
                    "float"
                ],
                "output_format": [
                    "dense"
                ]
            }
        }
    },
    "algorithm_parameters": {
        "homo_nn_0": {
            "config_type": "pytorch",
            "nn_define": [
                {
                    "layer": "Linear",
                    "name": "line",
                    "type": "normal",
                    "config": [
                        8,
                        1
                    ]
                },
                {
                    "layer": "Relu",
                    "type": "activate",
                    "name": "relu"
                }
            ],
            "batch_size": -1,
            "optimizer": {
                "optimizer": "Adam",
                "lr": 0.05
            },
            "early_stop": {
                "early_stop": "diff",
                "eps": 0.0001
            },
            "loss": "MSELoss",
            "metrics": [
                "mse"
            ],
            "max_iter": 2
        }
    }
}
```
## 6、提交任务
```shell
cd /fate/python/fate_flow/
python fate_flow_client.py -f submit_job -c /fate/examples/dsl/v1/homo_nn/pytorch_homo_dnn_single_layer.json -d /fate/examples/dsl/v1/homo_nn/test_homo_nn_train_then_predict.json
```
返回信息如下，任务提交成功
```json
{
    "data": {
        "board_url": "http://0.0.0.0:8080/index.html#/dashboard?job_id=2020120206452096971523&role=guest&party_id=10000",
        "job_dsl_path": "/fate/jobs/2020120206452096971523/job_dsl.json",
        "job_id": "2020120206452096971523",
        "job_runtime_conf_on_party_path": "/fate/jobs/2020120206452096971523/guest/job_runtime_on_party_conf.json",
        "job_runtime_conf_path": "/fate/jobs/2020120206452096971523/job_runtime_conf.json",
        "logs_directory": "/fate/logs/2020120206452096971523",
        "model_info": {
            "model_id": "arbiter-10000#guest-10000#host-10000#model",
            "model_version": "2020120206452096971523"
        },
        "pipeline_dsl_path": "/fate/jobs/2020120206452096971523/pipeline_dsl.json",
        "train_runtime_conf_path": "/fate/jobs/2020120206452096971523/train_runtime_conf.json"
    },
    "jobId": "2020120206452096971523",
    "retcode": 0,
    "retmsg": "success"
}
```
## 7、查看FATEBoard
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201202151203856.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

