# FATE Pipeline

Pipeline是一种高级的python API，它允许使用者以顺序方式对FATE任务进行设计、启动和查询。FATEPipeline具有用户友好的特点，且与FATE命令行工具保持一致。用户可以通过向Pipeline中添加组件来自定义任务工作流，然后通过一次调用即可启动任务。此外， Pipeline提供了训练后进行预测和查询信息的功能。通过运行[简单示例](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/demo/pipeline-mini-demo.py)可以了解FATEPipeline的工作方式。参与方id和工作模式的默认值可能需要根据部署设置进行修改。

```python
python pipeline-mini-demo.py config.yaml
```

更多有关Pipeline的示例，请参考[examples](https://github.com/FederatedAI/FATE/tree/master/examples/pipeline)。

## FATE任务是有向无环图

FATE任务是由算法组件节点组成的。FATEPipeline提供了易于使用的工具，用于配置任务的顺序和设置。FATE以模块化风格编写。模块被设计为具有输入输出数据和模型。因此，通过设置一个模块的输出为另一个模块的输入，可将两个模块相连接。通过观察FATE的各个模块如何处理数据，我们可以看出FATE任务实际上是由一系列子任务组成的。例如，在前文提及的[简单示例](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/demo/pipeline-mini-demo.py)中，guest方的数据首先通过*Reader*模块读取，然后导入至*DataIO*模块。通过将数据输入至*Intersection*模块，可寻找到guest方和host方的交叠id。最终，基于数据训练得到*HeteroLR*模型。每个提及的模块都是关于数据的一个小任务，组合起来就构成了一个模型训练任务。

除了给出的简单示例，一个任务可能包含多个数据集和模型。更多有关Pipeline的示例，请参考[examples](https://github.com/FederatedAI/FATE/tree/master/examples/pipeline)。

## 安装Pipeline

### PipelineCLI

在安装FATE客户端成功后，用户需为Pipeline配置服务器信息和日志目录。Pipeline提供了命令行工具用于快速设置。运行以下命令以获取更多信息。

```
pipeline init --help
```

## Pipeline接口

### 组件

FATE模块被包装在PipelineAPI的*component*中。每个组件可以输入输出*Data*和*Model*。组件参数可在初始化时便捷地进行设定。未指定的参数将采用默认值。所有组件均有可随意设置的*name*。组件的名称是其标识符，因此其名称在一个Pipeline中必须唯一。为便于追踪，我们建议每个组件的名称包含一个数字尾缀。

每个组件具有输入和/或输出*Data*和/或*Model*。有关如何使用组件的细节，请参考[guide](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/component/README.rst)。

一个采用指定参数值进行组件初始化的示例如下：

```
hetero_lr_0 = HeteroLR(name="hetero_lr_0", early_stop="weight_diff", max_iter=10, early_stopping_rounds=2, validation_freqs=2)
```

### 输入

[输入](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/component/README.rst)封装了组件全部的输入，包括*Data*和*Model*输入。要访问组件的*input*，请设置其*input*属性：

```
input_all = dataio_0.input
```

### 输出

[输出](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/component/README.rst)封装了组件的全部输出结果，包括*Data*和*Model*输出。要访问组件的*output*，请设置其*output*属性：

```
output_all = dataio_0.output
```

### 数据

*Data*包装了组件全部数据类型的输入输出。FATEPipeline包括包含5种*data*，每种用于不同场景。更多相关信息，请参考[here](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/component/README.rst)。

### 模型

*Model*定义了组件的模型输入和输出。类似于*Data*，两种*models*可用于不同目的。更多相关信息，请参考[here](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/component/README.rst)。

## 构建Pipeline

以下是构建Pipeline的概要指南。请参考[简单示例](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/demo/pipeline-mini-demo.py)以获取解释性的演示。

初始化Pipeline后，应制定任务参与方和发起方。以下是一个初始化Pipeline的示例：

```
pipeline = PipeLine()
pipeline.set_initiator(role='guest', party_id=9999)
pipeline.set_roles(guest=9999, host=10000, arbiter=10000)
```

需要设置*Reader*进行读取数据，以便其他组件可以处理数据。定义*Reader*组件：

```
reader_0 = Reader(name="reader_0")
```

在大多数情况下，*DataIO*位于*Reader*之后，用于将数据转换为DataInstance格式，该格式后续可用于数据工程和模型训练。部分组件（如*Union*和*Intersection*）可以直接在非DataInstance表上运行。

各参与方可通过设置*get_party_instance*，独立配置所有的Pipeline组件。例如，guest方可通过如下命令配置*DataIO*组件：

```
dataio_0 = DataIO(name="dataio_0")
guest_component_instance = dataio_0.get_party_instance(role='guest', arty_id=9999)
guest_component_instance.component_param(with_label=True, output_format="dense")
```

可使用*add_component*将组件加入至Pipeline中。将*DataIO*组件加入至先前已创建的Pipeline的命令如下：

```
pipeline.add_component(dataio_0, data=Data(data=reader_0.output.data))
```

### 以Keras方式构建FATE NN 模型

在Pipeline中，用户能够以Keras方式构建NN结构。以Homo-NN为例：

首先，导入Keras，定义你的nn结构：

```
from tensorflow.keras import optimizers
from tensorflow.keras.layers import Dense

layer_0 = Dense(units=6, input_shape=(10,), activation="relu")
layer_1 = Dense(units=1, activation="sigmoid")
```

然后，像Keras中使用序列类型一样，将nn层加入至Homo-NN模型中：

```
from pipeline.component.homo_nn import HomoNN

# set parameter
homo_nn_0 = HomoNN(name="homo_nn_0", max_iter=10, batch_size=-1, early_stop={"early_stop": "diff", "eps": 0.0001})
homo_nn_0.add(layer_0)
homo_nn_0.add(layer_1)
```

设置优化器，编译Homo-NN模型：

```
homo_nn_0.compile(optimizer=optimizers.Adam(learning_rate=0.05), metrics=["Hinge", "accuracy", "AUC"], loss="binary_crossentropy")
```

将其加入至Pipeline中：

```
pipeline.add_component(homo_nn, data=Data(train_data=dataio_0.output.data))
```

## 初始化运行任务参数

为了训练或预测，用户需初始化运行环境，如“backend”和“work_mode”，

```
from pipeline.runtime.entity import JobParameters
job_parameters = JobParameters(backend=Backend.EGGROLL, work_mode=WorkMode.STANDALONE
```

## 运行Pipeline

在添加全部组件后，用于在运行设计好的任务前，需要首先编译Pipeline。编译后，Pipeline可在相适应的*Backend*和*WorkMode*下进行拟合（运行训练任务）。

```
pipeline.compile()
pipeline.fit(job_parameters)
```

## 任务查询

FATEPipeline提供了API用于查询组件信息，包括数据、模型和概要。所有查询API均具有与[FlowPy](https://github.com/FederatedAI/FATE/tree/master/python/fate_client/flow_sdk)相匹配的名称，而Pipeline检索查询结果并将结果直接返回给用户。

```
summary = pipeline.get_component("hetero_lr_0").get_summary()
```

## 部署组件

在完成Pipeline拟合后，可在新的数据集上进行预测。在预测前，首先需要部署必要的组件。这一步中，预测Pipeline选择组件用于预测任务。

```
pipeline.deploy_component([dataio_0, hetero_lr_0])
```

## 基于Pipeline预测

首先，初始化一个新的Pipeline，然后指定用于预测的数据源。

```
predict_pipeline = PipeLine()
predict_pipeline.add_component(reader_0)
predict_pipeline.add_component(pipeline,
                               data=Data(predict_input={pipeline.dataio_0.input.data: reader_0.output.data}))
```

然后可在新的Pipeline上执行预测任务。

```
predict_pipeline.predict(job_parameters)
```

此外，由于Pipeline是模块化的，用户可在执行预测前为原Pipeline添加新的组件。

```
predict_pipeline.add_component(evaluation_0, data=Data(data=pipeline.hetero_lr_0.output.data))
predict_pipeline.predict(job_parameters)
```

## 保存和恢复Pipeline

可使用**dump**接口保存Pipeline。

```
pipeline.dump("pipeline_saved.pkl")
```

可使用**load_model_from_file**接口恢复Pipeline。

```
from pipeline.backend.pipeline import PineLine
PipeLine.load_model_from_file("pipeline_saved.pkl")
```

## Pipeline的概要信息

可使用**describe**接口获取Pipeline信息，该接口可打印创建时间、拟合或预测状态以及构造的dsl（如存在）。

```
pipeline.describe()
```

## 上传数据

Pipeline提供了上传本地数据表格的功能。请参考[upload_demo](https://github.com/FederatedAI/FATE/blob/master/python/fate_client/pipeline/demo/pipeline-upload.py)获取快速示例。请注意，可以一次性添加所有上传数据，并且用于执行上传的Pipeline可以是训练Pipeline或预测Pipeline（或演示中的单独Pipeline）。

## Pipeline vs. CLI

在过去的版本中，用户通过命令行接口与FATE交互，通常是手动配置conf和dsl的json文件。手动配置枯燥且易错。FATEPipeline在编译时自动生成任务配置文件，从而允许快速进行任务设计实验。