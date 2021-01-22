# FATE 测试

本文档提供一些用于FATE测试的有效工具。

[![tutorial](https://github.com/FederatedAI/FATE/raw/master/python/fate_test/images/tutorial.gif)](https://github.com/FederatedAI/FATE/blob/master/python/fate_test/images/tutorial.gif)

## 快速开始

1. （可选）创建`python`虚拟环境

   ```
   python -m venv venv
   source venv/bin/activate
   pip install -U pip
   ```

2. 安装`fate_test`包

   ```
   pip install fate_test
   fate_test --help
   ```

3. 编辑默认的`fate_test_config.yaml`

   ```
   # 使用系统默认编辑器编辑优先级配置文件
   # 根据注释填写一些字段
   fate_test config edit
   ```

4. 配置`FATE-Pipeline `和`FATE-Flow`命令行服务器设置

```
# 配置 FATE-Pipeline 和 FATE-Flow
pipeline init --port 9380 --ip 127.0.0.1
# 配置 FATE-Flow 命令行服务器设置
flow init --port 9380 --ip 127.0.0.1
```

5. 运行`fate_test`组件

```
fate_test suite -i <path contains *testsuite.json>
```

6. 运行`fate_test`基准测试

```
fate_test benchmark-quality -i <path contains *benchmark.json>
```

7. 有用的日志或异常将保存到日志目录中，并在最后一步中显示命名空间



## 开发安装

在开发过程中使用可编辑模式更加方便：将步骤2替换为以下步骤

```
pip install -e ${FATE}/python/fate_client && pip install -e ${FATE}/python/fate_test
```



## 命令类型

- 组件：用于运行测试组件，即FATE任务的集合

  ```
  fate_test suite -i <path contains *testsuite.json>
  ```

- `benchmark-quality`用于比较`FATE`和其他机器学习系统之间的模型效果

  ```
  fate_test benchmark-quality -i <path contains *benchmark.json>
  ```



## 配置示例

1. 不需要 SSH 通道的情况：

   - 9999, service: service_a
   - 10000, service: service_b

   `service_a`，`service_b`都可以直接请求：

   ```
   work_mode: 1 # 0 for standalone, 1 for cluster
   data_base_dir: <path_to_data>
   parties:
     guest: [10000]
     host: [9999, 10000]
     arbiter: [9999]
   services:
     - flow_services:
       - {address: service_a, parties: [9999]}
       - {address: service_b, parties: [10000]}
   ```

2. 需要 SSH 通道的情况：

   - 9999, service: service_a
   - 10000, service: service_b

   `service_a`可以直接请求，而`service_b`不可以， 但你可以在其他节点请求`service_b`，比如说B节点：

   ```
   work_mode: 0 # 0 for standalone, 1 for cluster
   data_base_dir: <path_to_data>
   parties:
     guest: [10000]
     host: [9999, 10000]
     arbiter: [9999]
   services:
     - flow_services:
       - {address: service_a, parties: [9999]}
     - flow_services:
       - {address: service_b, parties: [10000]}
       ssh_tunnel: # optional
       enable: true
       ssh_address: <ssh_ip_to_B>:<ssh_port_to_B>
       ssh_username: <ssh_username_to B>
       ssh_password: # optional
       ssh_priv_key: "~/.ssh/id_rsa"
   ```



## 测试组件

测试组件用于按顺序运行一系列测试任务。 用于任务的数据可以在提交任务之前上载，并在任务完成后清除。该工具对于FATE的发布测试很有用。



### 命令选项

```
fate_test suite --help
```

1. include:

   ```
   fate_test suite -i <path1 contains *testsuite.json>
   ```

   在`path1`中运行测试组件。

2. exclude:

   ```
   fate_test suite -i <path1 contains *testsuite.json> -e <path2 to exclude> -e <path3 to exclude> ...
   ```

   在`path1`中运行测试组件，但不在`path2`和`path3`中运行。

3. glob:

   ```
   fate_test suite -i <path1 contains *testsuite.json> -g "hetero*"
   ```

   在以`path1`和`hetero`开头的子目录中运行测试组件。

4. replace:

   ```
   fate_test suite -i <path1 contains *testsuite.json> -r '{"maxIter": 5}'
   ```

   在数据配置文件、配置文件、dsl文件中找到所有键为`maxIter`的键值对，并将值替换为5。

5. skip-data:

   ```
   fate_test suite -i <path1 contains *testsuite.json> --skip-data
   ```

   在`path1`中运行测试组件，而不上传`benchmark.json`中指定的数据。

6. yes:

   ```
   fate_test suite -i <path1 contains *testsuite.json> --yes
   ```

   直接在`path1`中运行测试组件，跳过用户再次确认选项。

7. skip-dsl-jobs:

   ```
   fate_test suite -i <path1 contains *testsuite.json> --skip-dsl-jobs
   ```

   在`path1`中运行测试组件，但跳过测试组件中的所有的tasks。当仅需要pipeline任务时，这个命令很有用。

8. skip-pipeline-jobs:

   ```
   fate_test suite -i <path1 contains *testsuite.json> --skip-pipeline-jobs
   ```

   在`path1`中运行测试组件，但跳过测试组件中的所有的pipeline任务。当仅需要dsl任务时，这个命令很有用。

## Benchmark Quality

`benchmark-quality`用于比较`FATE`和其他机器学习系统之间的模型效果。 基准程序为每个基准任务组生成一个指标比较摘要。

```
fate_test benchmark-quality -i examples/benchmark_quality/hetero_linear_regression
+-------+--------------------------------------------------------------+
|  Data |                             Name                             |
+-------+--------------------------------------------------------------+
| train | {'guest': 'motor_hetero_guest', 'host': 'motor_hetero_host'} |
|  test | {'guest': 'motor_hetero_guest', 'host': 'motor_hetero_host'} |
+-------+--------------------------------------------------------------+
+------------------------------------+--------------------+--------------------+-------------------------+---------------------+
|             Model Name             | explained_variance |      r2_score      | root_mean_squared_error |  mean_squared_error |
+------------------------------------+--------------------+--------------------+-------------------------+---------------------+
| local-linear_regression-regression | 0.9035168452250094 | 0.9035070863155368 |   0.31340413289880553   | 0.09822215051805216 |
| FATE-linear_regression-regression  | 0.903146386539082  | 0.9031411831961411 |    0.3139977881119483   | 0.09859461093919596 |
+------------------------------------+--------------------+--------------------+-------------------------+---------------------+
+-------------------------+-----------+
|          Metric         | All Match |
+-------------------------+-----------+
|    explained_variance   |    True   |
|         r2_score        |    True   |
| root_mean_squared_error |    True   |
|    mean_squared_error   |    True   |
+-------------------------+-----------+
```



### 命令选项

使用以下命令显示帮助消息：

```
fate_test benchmark-quality --help
```

1. include:

   ```
   fate_test benchmark-quality -i <path1 contains *benchmark.json>
   ```

   在`path1`中运行基准测试组件。

2. exclude:

   ```
   fate_test benchmark-quality -i <path1 contains *benchmark.json> -e <path2 to exclude> -e <path3 to exclude> ...
   ```

   在`path1`中运行基准测试组件，但不在`path2`和`path3`中运行。

3. glob:

   ```
   fate_test benchmark-quality -i <path1 contains *benchmark.json> -g "hetero*"
   ```

   在以`path1`和`hetero`开头的子目录中运行基准测试组件。

4. tol:

   ```
   fate_test benchmark-quality -i <path1 contains *benchmark.json> -t 1e-3
   ```

   在`path1`中运行基准测试组件，并将度量差异的绝对值设置为0.001。如果度量之间的绝对差小于`tol `，则认为度量几乎相等。查看基准测试组件[编写指南](https://github.com/FederatedAI/FATE/blob/master/python/fate_test/README.rst#benchmark-testsuite)设置`tol`。

5. skip-data:

   ```
   fate_test benchmark-quality -i <path1 contains *benchmark.json> --skip-data
   ```

   在`path1`中运行基准测试组件，而不上传`benchmark.json`中指定的数据。

6. yes:

   ```
   fate_test benchmark-quality -i <path1 contains *benchmark.json> --yes
   ```

   直接在`path1`中运行基准测试组件，跳过用户再次确认选项。
   
   

### 基准测试组件

任务的配置应在基准测试组件中指定，该组件的文件名以` benchmark.json`结尾。 有关基准测试组件示例的信息，请参考[此处](https://github.com/FederatedAI/FATE/tree/master/examples/benchmark_quality)。

一个基准测绘组件包括以下几个元素:

- data: 运行`FATE`任务之前要上传的本地数据列表

  - file: 要上传的原始数据文件的路径，应该是testsuite或FATE安装路径的相对路径
  - head: 文件是否包含header
  - partition: 数据存储的分组个数
  - table_name: 存储中的表名
  - namespace: 存储中表的命名空间
  - role: 按照`fate_test.config`中的设定，哪个角色上载数据； 命名格式为：`“ {role_type} _ {role_index}”`，索引从0开始

  ```
  "data": [
      {
          "file": "examples/data/motor_hetero_host.csv",
          "head": 1,
          "partition": 8,
          "table_name": "motor_hetero_host",
          "namespace": "experiment",
          "role": "host_0"
      }
  ]
  ```

- job group: 每个组包括任意数量的任务，以及指向相应脚本和配置的路径

  - job: 要运行的任务名称，在每个组列表中必须唯一

    - script: [测试脚本](https://github.com/FederatedAI/FATE/blob/master/python/fate_test/README.rst#testing-script)的路径, 应该是testsuite的相对路径
    - conf: 脚本的任务配置文件的路径，应是testsuite的相对路径

    ```
    "local": {
         "script": "./local-linr.py",
         "conf": "./linr_config.yaml"
    }
    ```

  - compare_setting: 用于质量指标比较的其他设置，目前仅需要`relative_tol`

    如果指标$a$和$b$满足$|a-b| <= max(relative\_tol * max(|a|,|b|),  absolute\_tol)$，认为他们几乎相等。在下面的示例中，如果local和FATE任务的度量标准相对差值小于$0.05 *max(abs(local\_metric), abs(pipeline\_metric))$，就认为他们几乎相等。

  ```
  "linear_regression-regression": {
      "local": {
          "script": "./local-linr.py",
          "conf": "./linr_config.yaml"
      },
      "FATE": {
          "script": "./fate-linr.py",
          "conf": "./linr_config.yaml"
      },
      "compare_setting": {
          "relative_tol": 0.01
      }
  }
  ```



### 测试脚本

所有作业脚本都必须具有`Main`函数作为执行作业的入口； 脚本应返回两个字典：一个是带有数据信息键值对：`{data_type}:{data_name_dictionary}`； 第二个包含`{metric_name}:{metric_value}`用于度量比较的键值对。

默认情况下，最终数据摘要显示名为“ FATE”的任务输出； 如果不存在这样的作业，则显示第一个作业返回的数据信息。 为了清晰呈现，我们建议用户遵循[常规](https://github.com/FederatedAI/FATE/blob/master/examples/data/README.md#data-set-naming-rule) 命名。 对于多主机任务，请考虑对主机进行如下编号：

```
{'guest': 'default_credit_homo_guest',
 'host_1': 'default_credit_homo_host_1',
 'host_2': 'default_credit_homo_host_2'}
```

比较返回的相同键的质量指标。 请注意，只能比较值为**实数**的指标。

- FATE脚本:

  `Main`应该有三个输入:

  - config: 任务配置, 从 `fate_test_config.yaml`加载的[JobConfig](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/utils/tools.py#L64) 对象
  - param: 任务参数设置, 从基准测试组件的指定的`conf`文件中加载的字典
  - namespace: 命名空间后缀, 用户指定的 *namespace* 或者使用时间戳生成的*namespace


- 非FATE脚本:

   `Main`应该有两个输入:

  - param: 任务参数设置, 从基准测试组件的指定的`conf`文件中加载的字典
  - (optional) config: 任务配置, 从 `fate_test_config.yaml`加载的[JobConfig](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/utils/tools.py#L64) 对象

注意FATE和非FATE脚本中的`Main`都可以设置为零输入参数。

## Data子命令

Data子命令用于上传或删除组件中的数据集。

### command options

```
fate_test data --help
```

1. include:

   ```
   fate_test data [upload|delete] -i <path1 contains *testsuite.json>
   ```

   在`path1`中的测试组件中上载/删除数据集

2. exclude:

   ```
   fate_test data [upload|delete] -i <path1 contains *testsuite.json> -e <path2 to exclude> -e <path3 to exclude> ...
   ```

   在`path1`中的测试组件中上载/删除数据集，但在`path2`、`path3`中不上传/下载数据集

3. glob:

   ```
   fate_test data [upload|delete] -i <path1 contains *testsuite.json> -g "hetero*"
   ```

   上传/删除`path1`中以`hetero`开头的子目录中测试组件中的数据集

## 完整的命令选项

```
.. click:: fate_test.scripts.cli:cli
  :prog: fate_test
  :show-nested:
```