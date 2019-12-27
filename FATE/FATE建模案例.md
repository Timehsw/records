# 概述
本文介绍使用联邦学习进行完整建模的流程，包括：

- 使用环境搭建
- 上传数据
- 构建完整建模DAG，包括特征分箱，特征选择，Xgb训练，模型评估

注意：关于联邦学习的配置文件说明，可以参考我的上一篇文章。[点此跳转](
https://github.com/Timehsw/records/blob/master/FATE/FATE%E9%85%8D%E7%BD%AE%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3-%E7%BF%BB%E8%AF%91%E5%8A%A0%E6%89%B9%E6%B3%A8%E8%87%AA%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3.md)

# 环境配置
**注意：** <u>本文是参照微众Github上的搭建文件使用docker进行Standalone进行部署，此部分请参考官方文档</u>。[点此跳转](
https://github.com/FederatedAI/FATE/blob/v1.1/standalone-deploy/doc/Fate-V1.1%E5%8D%95%E6%9C%BA%E7%89%88%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97.md)

咱们使用微众的联邦学习框架建模，主要是构建一个DSL配置文件和一个Runtime 配置文件。一般来说，环境搭建在远程服务器，而我们开发在本地，每次修改配置文件再上传很不方便，效率不高。


因此，我们可以使用VScode编辑器的**Remote-SSH插件**。该插件允许我们修改远程文件如果在本地修改一般。效率翻倍。所以我们可以将远程配置文件的目录映射到docker环境，然后使用Remote-ssh本地连接到远程服务器。如此一来，在本地修改了配置文件后，就会实时同步到docker环境，然后在docker环境中执行作业启动即可。

**效果如下图所示**

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-vs-env01.png)

在vscode中安装插件很简单，再次就不撰述了。下面进入正题，开始使用联邦学习建模。

# 第一步：上传数据

这里数据已经划分为了训练集和验证集。使用一个guest和1个host.

上传guest训练数据配置
```
{
  "file": "examples/data/ABC_A_score_train.csv",
  "head": 1,
  "partition": 10,
  "work_mode": 0,
  "table_name": "hetero_ABC_A_train",
  "namespace": "hetero_ABC_A_train_guest"
}

```
上传guest验证数据配置
```
{
  "file": "examples/data/ABC_A_score_valid.csv",
  "head": 1,
  "partition": 10,
  "work_mode": 0,
  "table_name": "hetero_ABC_A_valid",
  "namespace": "hetero_ABC_A_valid_guest"
}
```
...

host上也雷同，修改file和table_name,namespace即可

在docker环境中执行上传文件的命令
```
python /fate/fate_flow/fate_flow_client.py -f upload -c upload_data_guest_train.json
python /fate/fate_flow/fate_flow_client.py -f upload -c upload_data_guest_valid.json
python /fate/fate_flow/fate_flow_client.py -f upload -c upload_data_host1_train.json
python /fate/fate_flow/fate_flow_client.py -f upload -c upload_data_host1_valid.json
```
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/upload-command.png)
执行数据上传的命令，会在fate_board出现一个job
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/upload-data.png)

ok,我们进入这个job,可以看到详细内容，这个已经是上传完了的
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/upload-data2.png)

# 第二步，构建DAG和RunTime Conf

本文中我们是使用纵向联邦建模，即用户大致相同，特征不同。其中Guest包含了Y和一部分特征，host补充了另外的一部分特征。

构建DAG的内容为，进行样本对齐，然后使用特征分箱，接着使用特征选择组件，去除一部分特征。然后接上xgb模型组件，最后使用模型评估组件。

DAG图如下：

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-dag.png)

## DAG配置文件如下，供参考

```

{
    "components" : {
        "dataio_0": {
            "module": "DataIO",
            "input": {
                "data": {
                    "data": [
                        "args.train_data"
                    ]
                }
            },
            "output": {
                "data": ["train"],
                "model": ["dataio"]
            }
         },
        "dataio_1": {
            "module": "DataIO",
            "input": {
                "data": {
                    "data": [
                        "args.eval_data"
                    ]
                },
                "model": [
                    "dataio_0.dataio"
                ]
            },
            "output": {
                "data": ["eval"],
                "model": ["dataio"]
            },
            "need_deploy": false
         },
        "intersection_0": {
             "module": "Intersection",
             "input": {
                 "data": {
                     "data": [
                         "dataio_0.train"
                     ]
                 }
             },
             "output": {
                 "data": ["train"]
             }
         },
        "intersection_1": {
             "module": "Intersection",
             "input": {
                 "data": {
                     "data": [
                         "dataio_1.eval"
                     ]
                 }
             },
             "output": {
                 "data": ["eval"]
             }
         },
         "hetero_feature_binning_0": {
            "module": "HeteroFeatureBinning",
            "input": {
                "data": {
                    "data": [
                        "intersection_0.train"
                    ]
                }
            },
            "output": {
                "data": ["transform_data"],
                "model": ["binning_model"]
            }
        },
        "hetero_feature_binning_1": {
            "module": "HeteroFeatureBinning",
            "input": {
                "data": {
                    "data": [
                        "intersection_1.eval"
                    ]
                },
                "model": [
                    "hetero_feature_binning_0.binning_model"
                ]
            },
            "output": {
                "data": ["transform_data"]
            }
        },
        "hetero_feature_selection_0": {
            "module": "HeteroFeatureSelection",
            "input": {
                "data": {
                    "data": [
                        "intersection_0.train"
                    ]
                },
                "isometric_model": [
                    "hetero_feature_binning_0.binning_model"
                ]
            },
            "output": {
                "data": ["train"],
                "model": ["selected"]
            }
        },
        "hetero_feature_selection_1": {
            "module": "HeteroFeatureSelection",
            "input": {
                "data": {
                    "data": [
                        "intersection_1.eval"
                    ]
                },
                "model": [
                    "hetero_feature_selection_0.selected"
                ]
            },
            "output": {
                "data": ["eval"],
                "model": ["selected"]
            }
        },
        "secureboost_0": {
            "module": "HeteroSecureBoost",
            "input": {
                "data": {
                    "train_data": ["hetero_feature_selection_0.train"],
                    "eval_data":  ["hetero_feature_selection_1.eval"]
                }
            },
            "output": {
                "data": ["train"],
                "model": ["secureboost"]
            }
        },
        "evaluation_0": {
            "module": "Evaluation",
            "input": {
                "data": {
                    "data": ["secureboost_0.train"]
                }
            },
            "output": {
                "data": ["evaluate"]
            }
        }
    }
}


```

## RunTime conf如下，供参考
```

{
  "initiator": {
    "role": "guest",
    "party_id": 10000
  },
  "job_parameters": {
    "work_mode": 0
  },
  "role": {
    "guest": [
      10000
    ],
    "host": [
      10000
    ],
    "arbiter": [
      10000
    ]
  },
  "role_parameters": {
    "guest": {
      "args": {
        "data": {
          "train_data": [
            {
              "name": "hetero_ABC_A_train",
              "namespace": "hetero_ABC_A_train_guest"
            }
          ],
          "eval_data": [
            {
              "name": "hetero_ABC_A_valid",
              "namespace": "hetero_ABC_A_valid_guest"
            }
          ]
        }
      },
      "dataio_0": {
        "with_label": [true],
        "label_name": ["y"],
        "label_type": ["int"],
        "output_format": ["dense"],
        "missing_fill": [true],
        "outlier_replace": [true]
      },
      "evaluation_0": {
          "eval_type": ["binary"],
          "pos_label": [1]
      }
    },
    "host": {
      "args": {
        "data": {
          "train_data":
          [
            {
              "name": "hetero_ABC_B_train",
              "namespace": "hetero_ABC_B_train_host"
            }
          ],
          "eval_data":
          [
            {
              "name": "hetero_ABC_B_valid",
              "namespace": "hetero_ABC_B_valid_host"
            }
          ]
        }
      },
      "dataio_0": {
        "with_label": [false],
        "output_format": ["dense"],
        "outlier_replace": [true]
      }
    }
  },
  "algorithm_parameters": {
    "hetero_feature_binning_0": {
      "method": "quantile",
      "compress_thres": 10000,
      "head_size": 10000,
      "error": 0.001,
      "bin_num": 10,
      "cols": -1,
      "adjustment_factor": 0.5,
      "local_only": false,
      "transform_param": {
        "transform_cols": -1,
        "transform_type": "bin_num"
      }
    },
    "hetero_feature_binning_1": {
      "method": "quantile",
      "compress_thres": 10000,
      "head_size": 10000,
      "error": 0.001,
      "bin_num": 10,
      "cols": -1,
      "adjustment_factor": 0.5,
      "local_only": false,
      "transform_param": {
        "transform_cols": -1,
        "transform_type": "bin_num"
      }
    },
    "hetero_feature_selection_0": {
      "select_cols": -1,
      "filter_methods": ["unique_value", "iv_value_thres",
        "coefficient_of_variation_value_thres", "iv_percentile", "outlier_cols"],
      "local_only": false,
      "unique_param": {
        "eps": 1e-6
      },
      "iv_value_param": {
        "value_threshold": 0.7
      },
      "iv_percentile_param": {
        "percentile_threshold": 0.6
      },
      "variance_coe_param": {
        "value_threshold": 0.2
      },
      "outlier_param": {
        "percentile": 0.95,
        "upper_threshold": 2.0
      }
    },
    "secureboost_0": {
      "task_type": "classification",
      "learning_rate": 0.1, 
      "num_trees": 5,
      "subsample_feature_rate": 1,
      "n_iter_no_change": false,
      "tol": 0.0001,
      "bin_num": 50,
      "objective_param": { 
          "objective": "cross_entropy"
      },
      "encrypt_param": {
          "method": "paillier" 
      },
      "predict_param": {
          "with_proba": true,
          "threshold": 0.5 
      },
      "cv_param": {
          "n_splits": 5,
          "shuffle": false,
          "random_seed": 103,
          "need_cv": false,
          "evaluate_param": {
              "eval_type": "binary"
          }
       },
      "validation_freqs": 1
  },
  "evaluation_0": {
      "eval_type": "binary"
  }
  }
}


```


## 进入环境执行命令

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-runing.png)



# 第三步：查看特征分箱组件的结果
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-feature_binning-guest.png)
guest方提供了Y值和特征，
host方只提供了特征
由于分箱计算woe和iv,是需要Y标签的，所以只能在guest方看到分箱后的结果，其中guest方的特征，在guest上是可以直接看到特征名称的，但是Host方的只能看到0，1，2，3.。。看不到具体的名称，这样也保证了数据的安全

guest方的特征含有y,以及特征x1,x2,x3,x4,x5,x6,x7,x8,x9


![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-feature_binning-output.png)

host方的数据，看不到特征名称，而且看不到分箱区间
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-feature-binning-host.png)

如果我们登录到host上去看看呢，
我们可以看到名称，看到分箱区间，但是看不到具体的IV,和WOE值，
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-feature-binning-host2.png)

# 第四步：查看特征选择组件的输出

## guest
在guest方，数据输出，做完特征选择，只剩x0,x2,x3,x5,x6,x7

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-feature-select-guest-out.png)

如图所示，做特征选择的时候，会基于异常值，iv百分位，变量相关性，以及变量的iv值来做特征选择，最后就只剩下几个了。
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-feature-selection.png)


## host
在host方
过滤条件
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-feature-selection-model.png)

输出数据特征

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/feature-selection-host-out.png)

## XGBoost

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-xgboost-running.png)
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-xgboost-running-job.png)
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-xgboost-model.png)
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-xgboost-model-evaluate.png)

# 总结

所以，在联邦学习中，若是想要做特征选择，那么必须前面接一个特征分箱的组件，因为联邦学习的特征选择规则中，有基于IV的值来做特征选择。

# 注意

多Host的情况下，目前的1.11版本暂不支持分箱和特征选择。