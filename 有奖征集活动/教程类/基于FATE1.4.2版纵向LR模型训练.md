## 一、纵向联邦学习背景
纵向：GUEST方和HOST方的数据特征维度有差异，但用户（ID）重合度高，适用于跨行业应用的场景。
## 二、数据准备
双方数据规模如下图所示，ID存在重叠，特征不同，guest方为有标签的一方：按7:3比例分割好训练集和测试集共四份数据heterotrain_g.csv/heterotest_g.csv/heterotrain_h.csv/heterotest_h.csv。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200818180837491.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200818181146887.png)

## 三、上传数据
 ==Guest端：==
* 上传训练集：python fate_flow_client.py -f upload -c examples/upload_try_g.json 
* upload_try_g.json文件如下：

```bash

{
    "file": "/home/app/projects/heterotrain_g.csv",
    "head": 1,
    "partition": 50,
    "work_mode": 1,
    "namespace": "fate_testg",
    "table_name": "datag"
  }
```

file: 文件路径
head: 数据文件是否包含表头
partition: 用于存储数据的分区数
work_mode: 指定的工作模式，0 代表单机版，1 代表集群版
table_name&namespace: 存储数据表的标识符号
* 上传测试集：python fate_flow_client.py -f upload -c examples/upload_try_gg.json 
* upload_try_gg.json文件如下：

```bash

{
    "file": "/home/app/projects/heterotest_g.csv",
    "head": 1,
    "partition": 50,
    "work_mode": 1,
    "namespace": "fate_testgg",
    "table_name": "datagg"
  }
```


==Host端：==
* 上传训练集：python fate_flow_client.py -f upload -c examples/upload_try_h.json 
* upload_try_h.json文件如下：

```bash
{
    "file": "/home/app/projects/heterotrain_h.csv",
    "head": 1,
    "partition": 50,
    "work_mode": 1,
    "namespace": "fate_testh",
    "table_name": "datah"
  }
```
* 上传测试集：python fate_flow_client.py -f upload -c examples/upload_try_hh.json 
* upload_try_hh.json文件如下：

```bash
{
    "file": "/home/app/projects/heterotest_h.csv",
    "head": 1,
    "partition": 50,
    "work_mode": 1,
    "namespace": "fate_testhh",
    "table_name": "datahh"
  }
```
## 三、提交任务
==Guest端：==
* 提交训练任务: 
python fate_flow_client.py -f submit_job -d examples/test_hetero_lr_job_dsl.json -c 
examples/test_hetero_lr_job_conf.json
* test_hetero_lr_job_conf.json文件如下：

```bash

{
    "initiator": {
        "role": "guest",
        "party_id": 9999
    },
    "job_parameters": {
        "work_mode": 1
    },
    "role": {
        "guest": [9999],
        "host": [10000],
        "arbiter": [10000]
    },
    "role_parameters": {
        "guest": {
            "args": {
                "data": {
                    "train_data": [{"name": "datag", "namespace": "fate_testg"}],
                    "eval_data": [
                        {
                          "name": "datagg",
                          "namespace": "fate_testgg"
                        }
                      ]
                }
            },
            "dataio_0":{
                "with_label": [true],
                "label_name": ["y"],
                "label_type": ["int"],
                "output_format": ["dense"]
            }
        },
        "host": {
            "args": {
                "data": {
                    "train_data": [{"name": "datah", "namespace": "fate_testh"}],
                    "eval_data": [
                        {
                          "name": "datahh",
                          "namespace": "fate_testhh"
                        }
                      ]
                }
            },
             "dataio_0":{
                "with_label": [false],
                "output_format": ["dense"]
            }
        }
    },
    "algorithm_parameters": {
        "feature_scale_0": {
            "method": "standard_scale",
            "need_run": true
          },
          "hetero_feature_binning_0": {
            "method": "quantile",
            "compress_thres": 10000,
            "head_size": 10000,
            "error": 0.001,
            "bin_num": 10,
            "cols": -1,
            "adjustment_factor": 0.5,
            "local_only": false,
            "need_run": true,
            "transform_param": {
              "transform_cols": -1,
              "transform_type": "bin_num"
            }
          },
          "hetero_feature_selection_0": {
              "select_cols": -1,
              "filter_methods": [
                "unique_value",
                "iv_value_thres",
                "coefficient_of_variation_value_thres",
                "iv_percentile",
                "outlier_cols"
              ],
              "local_only": false,
              "unique_param": {
                "eps": 1e-6
              },
              "iv_value_param": {
                "value_threshold": 1.0
              },
              "iv_percentile_param": {
                "percentile_threshold": 0.9
              },
              "variance_coe_param": {
                "value_threshold": 0.3
              },
              "outlier_param": {
                "percentile": 0.95,
                "upper_threshold": 10
              },
              "need_run": true
            },
        "hetero_lr_0": {
            "penalty": "L2",
            "optimizer": "rmsprop",
            "eps": 1e-5,
            "alpha": 0.01,
            "max_iter": 100,
            "converge_func": "diff",
            "batch_size": -1,
            "learning_rate": 0.15,
            "init_param": {
	            "init_method": "random_uniform"
            }
        },
        "intersect_0": {
            "intersect_method": "rsa",
            "sync_intersect_ids": true,
            "only_output_key": false
        }
    }
}

```

* test_hetero_lr_job_dsl.json文件如下：

```bash

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
            },
	"need_deploy": true
         },
         "dataio_1": {
            "module": "DataIO",
            "input": {
                "data": {
                    "data": [
                        "args.eval_data"
                    ]
                },
                "model": [
                    "dataio_0.dataio"
                ]
            },
            "output": {
                "data": ["eval_data"],
                "model": ["dataio"]
            }
         },
         "intersection_0": {
            "module": "Intersection",
            "input": {
                "data": {
                    "data": [
                        "dataio_0.train"
                    ]
                }
            },
            "output": {
                "data": ["train"]
            }
        },
        "intersection_1": {
            "module": "Intersection",
            "input": {
                "data": {
                    "data": ["dataio_1.eval_data"]
                }
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
                        "intersection_0.train"
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
                        "intersection_1.eval_data"
                    ]
                },
                "model":["feature_scale_0.feature_scale"]
            },
            "output": {
                "data": ["eval_data"],
                "model": ["feature_scale"]
            }
        },
        "hetero_feature_binning_0": {
            "module": "HeteroFeatureBinning",
            "input": {
                "data": {
                    "data": [
                        "feature_scale_0.train"
                    ]
                }
            },
            "output": {
                "data": ["transform_data"],
                "model": ["binning_model"]
            },
            "need_deploy": false
        },
        "hetero_feature_binning_1": {
            "module": "HeteroFeatureBinning",
            "input": {
                "data": {
                    "data": [
                        "feature_scale_1.eval_data"
                    ]
                },
                "model":["hetero_feature_binning_0.binning_model"]
            },
            "output": {
                "data": ["transform_data_test"],
                "model": ["binning_model"]
            },
            "need_deploy": false
        },
        "hetero_feature_selection_0": {
            "module": "HeteroFeatureSelection",
            "input": {
                "data": {
                    "data": [
                        "hetero_feature_binning_0.transform_data"
                    ]
                },
                "isometric_model": [
                    "hetero_feature_binning_0.binning_model"
                ]
            },
            "output": {
                "data": ["train"],
                "model": ["selected"]
            }
        },
        "hetero_feature_selection_1": {
            "module": "HeteroFeatureSelection",
            "input": {
                "data": {
                    "data": [
                        "hetero_feature_binning_1.transform_data_test"
                    ]
                },
                "model":["hetero_feature_selection_0.selected"],
                "isometric_model": [
                    "hetero_feature_binning_1.binning_model"
                ]
            },
            "output": {
                "data": ["eval_data"],
                "model": ["selected"]
            }
        },
        "hetero_lr_0": {
            "module": "HeteroLR",
            "input": {
                "data": {
                    "train_data": ["hetero_feature_selection_0.train"],
                    "eval_data": ["hetero_feature_selection_1.eval_data"]
                }
            },
            "output": {
                "data": ["hetero_lr_data"],
                "model": ["hetero_lr"]
            }
        },
        "evaluation_0": {
            "module": "Evaluation",
            "input": {
                "data": {
                    "data": ["hetero_lr_0.hetero_lr_data"]
                }
            },
            "output": {
                "data": ["evaluate"]
            }
        }
    }
}

```

**若页面如图所示，则说明提交成功啦~**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081818485575.png#pic_center)

四、查看任务
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200818185038875.png#pic_center)


* 根据Jobid识别任务，可在右侧添加备注（目前1.4.2仅支持英文备注）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200818185130535.png#pic_center)
==到达board界面可在上方查看到三方的流程图如下：==
|角色| 对应流程图|
|:-:| -------------:|
Guest  | ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081909183670.png#pic_center)
Host|![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819092403346.png#pic_center)
Arbiter|![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819092419782.png#pic_center)
* 点击不同的组件后按==view the outputs==可查看具体的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819092500170.png)
- 查看模型详情：
==Guest端：== 
   * ==点击intersection_0:==
   模型输出中可查看当前的交集样本数以及交集率。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819093110478.png#pic_center)
    * ==点击feature_scale_0:==
    模型输出中可查看特征列表，包括各特征进行归一化后的特征值下限、上限、均值以及标准差。
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819093606488.png)
    * ==点击hetero_feature_binning_0:==
查看自身及各 host 方的特征分箱信息，通过切换 partyid 来查看不同 host 方的特征分箱信息，其中来源于 host 方的特征名及分箱详情均显示**匿名**。特征列表中可查看所有分箱特征的信息，包括特征所属方、特征 iv 值、特征单调性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819093922554.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819095143186.png)
下一张分箱详情表中可根据特征查看具体的分箱，包括各分箱的区间、iv 值、woe 值、正负例样本数和比例:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819095333252.png)
可视化图：两张图依次为特征分箱区间的样本分布统计图及 woe 统计图。【相比1.3版样式有变】
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081909553227.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819095606971.png)
 * ==点击hetero_feature_selection_0：==
  可选择查看 guest 方和 host 方的特征选择结果，通过切换 partyid 选择不同的 host 方查看结果，其中各 host 方的特征显示**匿名**。
  如图所示，每一列代表的是一个过滤方法处理后的结果，从左往右为过滤方法执行的先后顺序分别为【用最大最小值的差值来判断是否某一列所有值一致/用设定的IV值判断/变异系数过滤器/按 iv 值排序后，取 iv 值排序最高的百分比阈值。比如，官方默认设定为 0.9，则取最高的 90%的特征。/ 异常值过滤】，深灰色代表不符合所设定的阈值而被过滤的特征，可以通过 data_ouput 来查看最终输出选择后的特征。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819100553811.png)
-------------------------------------
>**以上组件相对应的组件_1【即测试集组件】outputs格式与训练集组件相同，故在此仅介绍了组件_0**

  * ==点击hetero_lr_0：==
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819092552314.png)
* Model output：仅能查看自身的模型信息，包括最大迭代次数、模型是否收敛以及特征权重列表。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819102013878.png)
* Data output：查看模型训练后输出的预测标签结果。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819102027223.png)
==Host端：==
>以下仅列举Host端组件与Guest端组件不同的outputs界面，
>组件界面展示相同的就不再赘述了哈
* ==点击hetero_feature_binning_0：==
* **Host 方与Guest方不同仅可查看自身的特征分箱信息。**
* 特征列表：可查看自身分箱特征的信息，包括特征显示在 guest 方使用的匿名、特征iv 值、特征单调性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819102837335.png)
* 分箱详情表：可根据特征查看具体的分箱，包括各分箱的区间、匿名。
**具体的woe和iv值Host方获取不到**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819104512514.png)
* ==点击hetero_feature_selection_0：==
**与Guest方不同，Host仅可查看自身的特征选择结果**，其中部分过滤器为了保护guest 方的隐私而不显示指标值只显示选择的结果。同样标灰色的表未选择的特征。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819110042531.png)
* ==点击hetero_lr_0：==
Model output：**仅能查看自身的模型信息**，包括最大迭代次数、模型是否收敛以及特征权重列表。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819110448349.png#pic_center)
Data output：**不输出数据**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081911050327.png#pic_center)
==Aribter端：==
* ==点击hetero_lr_0==：
Model output：只可查看 loss 曲线图，除此以外**无法查看 guest 及 host 方相关的其他信息。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819110717470.png)
Data output：**不输出数据**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819110802418.png#pic_center)
==模型评估：==
* 点击evalution组件:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819110904349.png)
==Guest端：==
* 评估指标表可查看模型在各数据集下的 AUC,KS，precision和recall 值、【其中的Quantile可滑动在0-1区间内选择判断正负样本的阈值】混淆矩阵、PSI稳定度指标(population stability index ,PSI)可衡量测试样本及模型开发样本评分的的分布差异，为最常见的模型稳定度评估指标。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819112445781.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819112513777.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819112546792.png)
* 图表区域可选择查看模型在各数据集下的 ROC,K-S，Lift，Gain，PR 和 Accuracy 曲线： 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819112620379.png)
==Host端：==
* **查看不到模型指标:**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200819112712350.png#pic_center)

