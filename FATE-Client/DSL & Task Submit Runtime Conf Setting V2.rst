DSL 配置和运行配置 V2
======================================

为了让任务模型的构建更加灵活，目前 FATE 使用了一套自定的领域特定语言 (DSL) 来描述任务。在 DSL 中，各种模块（例如数据读写 data_io，特征工程 feature-engineering， 回归 regression，分类 classification）可以通向一个有向无环图 （DAG） 组织起来。通过各种方式，用户可以根据自身的需要，灵活地组合各种算法模块。

除此之外，每个模块都有不同的参数需要配置，不同的 party 对于同一个模块的参数也可能有所区别。为了简化这种情况，对于每一个模块，FATE 会将所有 party 的不同参数保存到同一个运行配置文件（Submit Runtime Conf）中，并且所有的 party 都将共用这个配置文件。这个指南将会告诉你如何创建一个 DSL 配置文件。

我们推荐在FATE 1.5.0 以上的版本里使用 V2 版本的 dsl 配置，当然你也可以选择继续使用 [`V1`_] 版本的配置。

.. _V1: dsl_conf_v1_setting_guide.rst

请注意，在 fate-1.5.0 中不支持在线使用 dsl V2 版本，我们会在更高版本中支持。

DSL 配置文件
------------------

DSL 的配置文件采用 json 格式，实际上，整个配置文件就是一个 json 对象 （dict）。在这个 dict 的第一级是 "components"，用来表示这个任务将会使用到的各个模块。

.. code-block:: json
  
  {
    "components" : {
            ...
        }
    }


每个独立的模块定义在 "components" 之下，例如：

.. code-block:: json
  
  "dataio_0": {
        "module": "DataIO",
        "input": {
            "data": {
                "data": [
                    "reader_0.train_data"
                ]
            }
        },
        "output": {
            "data": ["train"],
            "model": ["dataio"]
        }
    }

如示例所示，用户需要将组件名称定义为该模块的 key。

请注意，在 DSL V2 中，所有建模任务配置都应包含一个** Reader **组件，来读取存储服务的数据，
该组件仅具有“output”字段，如下所示：

.. code-block:: json

  "reader_0": {
        "module": "Reader",
        "output": {
            "data": ["train"]
        }
  }

参数说明
^^^^^^^^^^^^^^^^^^^

:module: 用来指定使用的模块。这个参数的内容需要和 python/federatedml/conf/setting_conf 下各个模块的文件名保持一致（不包括 ``.json`` 后缀）。

:input: 分为两种输入类型，分别是 data 和 model。

    - Data: 有四种可能的输入类型

      1. data: 一般被用于 data_io 模块, feature_engineering 模块或者 evaluation 模块
      2. train_data: 一般被用于 HeteroLR、HeteroSBT 等模块。如果出现了 train_data 字段，那么这个任务将会被识别为一个 fit 任务
      3. validate_data: 如果出现了 train_data 字段，则此字段为可选项。如果填写了此字段，它将被当做校验集使用。
      4. test_data: 指定用于预测的数据，如果设置了此字段，那么 **model** 也必填。

    - Model: 有两种可能的输入类型

      - model: 用于同种类型组件的模型输入。例如，hetero_binning_0 会对模型进行 fit，然后 hetero_binning_1 将会使用 hetero_binning_0 的输出用于 predict 或 transform。代码示例：

      .. code-block:: json

          "hetero_feature_binning_1": {
              "module": "HeteroFeatureBinning",
              "input": {
                  "data": {
                      "data": [
                          "dataio_1.validate_data"
                      ]
                  },
                  "model": [
                      "hetero_feature_binning_0.fit_model"
                  ]
              },
              "output": {
                  "data": ["validate_data"],
                "model": ["eval_model"]
              }
          }

      - isometric_model: 用于指定继承上游组件的模型输入。 例如，feature selection 的上游组件是 feature binning，它将会用到 feature binning 的信息来作为 feature importance。代码示例：

        .. code-block:: json

            "hetero_feature_selection_0": {
                "module": "HeteroFeatureSelection",
                "input": {
                    "data": {
                        "data": [
                            "hetero_feature_binning_0.train"
                        ]
                    },
                    "isometric_model": [
                        "hetero_feature_binning_0.output_model"
                    ]
                },
                "output": {
                    "data": ["train"],
                    "model": ["output_model"]
                }
            }

:output: 和 input 一样，有 data 和 model 两种类型。
    
    1. Data: 指定输出的 data 名
    2. Model: 指定输出的 model 名

    具体情况可以参考上面的例子。


运行配置 Submit Runtime Conf
-------------------

除了 DSL 的配置文件之外，用户还需要准备一份运行配置（Submit Runtime Conf）用于设置各个组件的参数。

:dsl_version:
  要使用 dsl V2 版本，需要设置下面这个字段。

  .. code-block:: json

     "dsl_version": 2

:initiator:
  在运行配置的开头，用户需要定义 initiator。例如

  .. code-block:: json

     "initiator": {
        "role": "guest",
        "party_id": 10000
     }


:role:
  所有参与这个任务的 roles 都需要在运行配置中指定。在 role 字段中，每一个元素代表一种角色以及承担这个角色的 party_id。每个角色的 party_id 以列表形式存在，因为一个任务可能涉及到多个 party 担任同一种角色。

  .. code-block:: json

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
     }

:component_parameters:
  在 dsl 中用于指定组件的运行参数。

  它包含两个子字段 ``common`` 和 ``role``:

  * ``common``字段下的参数规范适用于所有 party
  * ``role`` 字段下的参数值仅使用于对应的 party

  .. code-block:: json

     "component_parameters": {
         "common": {
             "component_x": {
                 ...
             },
             ...
         },
         "role": {
             ...
         }
     }

  :role:
    在 ``role`` 字段中，party 名为 key，参数说明为 value。

    代码示例：

    .. code-block:: json

       "role": {
            "guest": {
                "0": {
                    "reader_0": {
                        "table": {
                                    "namespace": "guest",
                                    "name": "table"
                        }
                    },
                    "dataio_0": {
                        "input_format": "dense",
                        "with_label": true
                    }
                }
            },
            "host": {
                "0": {
                    "reader_0": {
                        "table": {
                                    "namespace": "host",
                                    "name": "table"}
                        },
                    "dataio_0": {
                        "input_format": "tag",
                        "with_label": false
                    }
                }
            }
        }

    “0”表示它是某个 party 的第0个 role（索引从0开始）。

    用户可以为每个组件配置不同的参数。

    组件名称必须和 dsl 配置文件中定义的名称相同。

    每个组件的参数都在 `Param <../python/federatedml/param>` 类中定义。
    
    party 可以打包在一起并共享配置，代码示例：

    .. code-block:: json

       "role": {
            "host": {
                "0|2": {
                    "dataio_0": {
                        "input_format": "tag",
                        "with_label": false
                    }
                },
                "1": {
                    "dataio_0": {
                        "input_format": "dense",
                        "with_label": false
                    }
                }
            }
        }

  :common:
    如果所有的 party 的某些参数相同，就可以吧这些参数提取到 ``common`` 中设置。代码示例：

    .. code-block:: json

        "common": {
            "hetero_feature_binning_0": {
                ...
            },
            "hetero_feature_selection_0": {
                ...
            },
            "hetero_lr_0": {
                "penalty": "L2",
                "optimizer": "rmsprop",
                "eps": 1e-5,
                "alpha": 0.01,
                 "max_iter": 10,
                 "converge_func": "diff",
                 "batch_size": 320,
                 "learning_rate": 0.15,
                 "init_param": {
                    "init_method": "random_uniform"
                 },
            "cv_param": {
                "n_splits": 5,
                "shuffle": false,
                "random_seed": 103,
                "need_cv": false,
                }
            }
        }

    和 ``role`` 字段一样，组件名为 key，参数说明为 value。

:job_parameters:
  请注意，要启用 DSL V2，必须将**dsl_version**设置为**2**。

	与 component_parameters 相同，它有两个子字段 ``common`` 和 ``role``：

  * ``common``字段下的参数规范适用于所有 party
  * ``role`` 字段下的参数值仅使用于对应的 party

  .. code-block:: json

     "job_parameters": {
          "common": {
             ...
          },
          "role": {
             ...
          }
     }

.. list-table:: Job 的可配置项
   :widths: 20 20 30 30
   :header-rows: 1

   * - 参数名
     - 默认值
     - 可选值
     - 备注

   * - job_type
     - train
     - train, predict
     - job 类型

   * - work_mode
     - 0
     - 0, 1
     - 0 代表 standalone, 1 代表 cluster

   * - backend
     - 0
     - 0, 1
     - 0 代表 EGGROLL, 1 代表 SPARK

   * - federated_status_collect_type
     - PUSH
     - PUSH, PULL
     - collecting job 的状态类型

   * - timeout
     - 604800
     - positive int
     - job 超时时间

   * - eggroll_run
     -
     - 最常用的是 “eggroll.session.processors.per.node”，详细信息在 `EggRoll configuration  <https://github.com/WeBankFinTech/eggroll/wiki/eggroll.properties:-Eggroll's-Main-Configuration-File>`_。
     - EGGROLL 的计算引擎参数

   * - spark_run
     -
     - num-executors, executor-cores
     - SPARK 的计算引擎参数

   * - rabbitmq_run
     -
     - queue, exchange 等
     - 用于创建队列的参数，在 rabbitmq 中使用

   * - task_parallelism
     - 2
     - 正整数
     - 允许并发的最大任务数

   * - model_id
     - \-
     - \-
     - 用于指定 prediction task 的 model

   * - model_version
     - \-
     - \-
     - 用于指定 prediction task 的 model 版本

.. list-table:: Job 的不可配置项
   :widths: 20 20 30 30
   :header-rows: 1

   * - 参数名
     - 默认值
     - 可选值
     - 备注

   * - computing_engine
     - 根据 ``work_mode`` 和 ``backend`` 自动设置
     - EGGROLL, SPARK, STANDALONE
     - 计算引擎

   * - storage_engine
     - 根据 ``work_mode`` 和 ``backend`` 自动设置
     - EGGROLL, HDFS, STANDALONE
     - 存储引擎

   * - federation_engine
     - 根据 ``work_mode`` 和 ``backend`` 自动设置
     - EGGROLL, RABBITMQ, STANDALONE
     - party 信息传输引擎

   * - federated_mode
     - 根据 ``work_mode`` 和 ``backend`` 自动设置
     - SINGLE, MULTIPLE
     - 联邦模式

.. 注意事项::

   1. 某些 ``computing_engine``, ``storage_engine``, 和 ``federation_engine`` 只能搭配使用。 例如，SPARK ``computing_engine`` 仅支持 HDFS ``storage_engine``。

   2. ``work_mode`` 和 ``backend`` 的组合将会决定使用哪种引擎组合。

   3. 开发人员可以使用其他引擎并设置新的引擎组合。

**EGGROLL** 配置示例:

.. code-block:: json

     "job_parameters": {
        "common": {
           "work_mode": 1,
           "backend": 0,
           "eggroll_run": {
              "eggroll.session.processors.per.node": 2
           }
        }
     }

**SPARK** 配置示例:

.. code-block:: json

     "job_parameters": {
        "common": {
            "work_mode": 1,
            "backend": 1,
            "spark_run": {
               "num-executors": 1,
               "executor-cores": 2
            }
        }
     }

配置文件设置完成并提交任务后，fate-flow 将结合角色列表中的参数列表和算法参数。
如果当中存在未定义字段，将使用默认值。
FATE Flow 会把这些配置文件发送给对应的 party，并开始 federated task。


多个 Host 情况下的配置
------------------------

对于存在多个 Host 的模型，所有 Host 的 party_id 都应该在 role 中列举出来。例如：

.. code-block:: json

   "role": {
      "guest": [
        10000
      ],
      "host": [
        10000, 10001, 10002
      ],
      "arbiter": [
        10000
      ]
   }

每个针对 Host 的参数都应该以列表的方式储存，列表中组件的个数和 Host 的个数应保持一致。

.. code-block:: json

   "component_parameters": {
      "role": {
         "host": {
            "0": {
               "reader_0": {
                  "table":
                   {
                     "name": "hetero_breast_host_0",
                     "namespace": "hetero_breast_host"
                   }
               }
            },
            "1": {
               "reader_0": {
                  "table":
                  {
                     "name": "hetero_breast_host_1",
                     "namespace": "hetero_breast_host"
                  }
               }
            },
            "2": {
               "reader_0": {
                  "table":
                  {
                     "name": "hetero_breast_host_2",
                     "namespace": "hetero_breast_host"
                  }
               }
            }
         }
      }
   }

``common``字段下的参数不需要一个个拷贝到 ``role``字段中
``common``字段下的参数会被复制到所有 party 中。


预测结果配置
------------------------

请注意，在dsl v2中，预测dsl不会在训练后自动生成。
用户需要先部署所需的组件。
请参考`FATE-Flow CLI <../python/fate_flow/doc/Fate_Flow_CLI_v2_Guide.rst#dsl>`__'
下面这个命令可以查看 deploy 命令的详细信息：

.. code-block:: bash

    flow job dsl --cpn-list ...

**Examples**
训练用的 dsl:

.. code-block:: json

    "components": {
        "reader_0": {
            "module": "Reader",
            "output": {
                "data": [
                    "data"
                ]
            }
        },
        "dataio_0": {
            "module": "DataIO",
            "input": {
                "data": {
                    "data": [
                        "reader_0.data"
                    ]
                }
            },
            "output": {
                "data": [
                    "data"
                ],
                "model": [
                    "model"
                ]
            }
        },
        "intersection_0": {
            "module": "Intersection",
            "input": {
                "data": {
                    "data": [
                        "dataio_0.data"
                    ]
                }
            },
            "output": {
                "data":[
                    "data"
                ]
            }
        },
        "hetero_nn_0": {
            "module": "HeteroNN",
            "input": {
                "data": {
                    "train_data": [
                        "intersection_0.data"
                    ]
                }
            },
            "output": {
                "data": [
                    "data"
                ],
                "model": [
                    "model"
                ]
            }
        }
    }

生成预测 dsl 的命令如下：

.. code-block:: bash

    flow job dsl --train-dsl-path $job_dsl --cpn-list "reader_0, dataio_0, intersection_0, hetero_nn_0" --version 2 -o ./

生成的 dsl：

.. code-block::: json

    "components": {
        "reader_0": {
            "module": "Reader",
            "output": {
                "data": [
                    "data"
                ]
            }
        },
        "dataio_0": {
            "module": "DataIO",
            "input": {
                "model": [
                    "pipeline.dataio_0.model"
                ],
                "data": {
                    "data": [
                        "reader_0.data"
                    ]
                }
            },
            "output": {
                "data": [
                    "data"
                ]
            }
        },
        "intersection_0": {
            "module": "Intersection",
            "output": {
                "data": [
                    "data"
                ]
            },
            "input": {
                "data": {
                    "data": [
                        "dataio_0.data"
                    ]
                }
            }
        },
        "hetero_nn_0": {
            "module": "HeteroNN",
            "input": {
                "model": [
                    "pipeline.hetero_nn_0.model"
                ],
                "data": {
                    "test_data": [
                        "intersection_0.data"
                    ]
                }
            },
            "output": {
                "data": [
                    "data"
                ]
            }
        }
    }

另外，你也可以添加其他组件来预测dsl，例如``Evaluation``：

.. code-block:: json

    "evaluation_0": {
        "module": "Evaluation",
        "input": {
            "data": {
                "data": [
                    "hetero_nn_0.data"
                ]
            }
         },
         "output": {
             "data": [
                 "data"
             ]
          }
    }
