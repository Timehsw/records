# 开启训练任务
在进行联邦学习的模型训练过程种，需要准备3个配置文件。
1. 为上传数据准备一个上传数据的配置文件；
2. 准备一个DSL配置文件，用来定义你的模型任务过程；
3. 提交运行时的配置文件，为每一个组件设置参数。

注意：哪个带y，json里面就定义哪个为guest

## 第一步：定义上传数据配置文件
为了使FATE能够使用你的数据，你需要去把数据上传上去。因此，一个上传数据的配置文件是需要的。我们命名一个简单的配置文件叫`upload_data.json`,看看其中的内容
```
{
  "file": "examples/data/AA02p2_YM_score_train.csv", # 文件路径
  "head": 1, # head表示你的文件是否包含有表头，1表示有表头，0表示没有
  "partition": 10, # 指定用多少个分区去存储你的数据
  "work_mode": 0, # 指示用什么模型。0表示使用standalone版本，1表示使用cluster版本
  "table_name": "hetero_AA02p2_YM_train", # table_name&namespace指示存储的数据表
  "namespace": "hetero_AA02p2_YM_train_host"
}
```

## 第二步：定义你的模型任务结构
事实上，当构建一个模型任务的时候，其中会涉及到多个组件，比如data_io,feature_engineering,algorithm_model,evaluation as so on . 然后，这些组件的组合是因模型任务而定的。因此，一个便捷的方式去自由组合这些组件就会是个关键功能了。

目前来说，FATE提供了一种DSL语言(domain-specific language)去定义你想要的结构。这些组件由DSL配置文件来组合成一个DAG图的。DSL配置文件的用法就像定义一个json文件一样简单。

DSL配置文件将为每个组件定义输入数据和（或）模型以及输出数据和（或）模型。下游组件将上游组件的输出作为自己的输入，通过这种方式，就可以通过配置文件来构造DAG图了。

### 字段定义

1. `component_name`：组件的件，应该以_num结尾，比如_0、_1 等等,并且应该以_0开始，这么做是为了当多个相同的组件存在的时候，可以区分。
2. `module`：指定使用哪个组件，这个字段应该是FATE模块支持的算法模块之一。FATE-1.0支持以下几个算法模块
    1. DataIO # 将输入数据转换为实例对象以供以后的组件使用
    2. Intersection # 查找数据集不同方的交集，主要用于异构场景建模
    3. FederatedSample # 抽样模块为构建数据集平衡，同时支持联邦和单机模式
    4. FeatureScale # 特征缩放标准化的模块
    5. HeteroFeatureBinning # 对输入数据进行分箱，计算每一列的iv以及根据分箱信息对woe转换
    6. HeteroFeatureSelection # 特征选择模型，支持联邦和单机模式
    7. OneHotEncoder # 特征编码模块，主要用于编码分箱结果
    8. HeteroLR # 逻辑回归模块（纵向联邦学习逻辑回归算法模块）
    9. HeteroLinR  # 线性回归模块
    10. HeteroPoisson # 泊松回归模块
    11. HomoLR # 逻辑回归模块 （横向联邦学习逻辑回归算法模块）
    12. HeteroSecureBoost # 异构的安全Boosting模块
    13. Evaluation # 模型评估模块，支持评估二分类，多分类和回归模型
   
3.`input`：输入有两种类型，数据和模型

+ 数据：这有三种可能的data_input类型
    + data：通常用于data_io,feature_engineering模块和评估中；
    + train_data：用在homo_lr,hetero_lr and secure_boost，如果这个字段被提供了，这个任务将被解析为训练任务
    + eval_data：如果train_data已经被提供了，那么这个字段就是一个可选项了。在这个案例中，这个数据会被作为验证集。如果train_data没有被提供，那么这个任务会被解析成一个预测(predict)或者转换(transform)任务。
    
+ 模型：这有两种可能的model_input类型
    + model：这是由相同类型的组件输入的模型，用于预测\转换阶段。例如hetero_binning_0作为训练的组件运行，hetero_binning_1接收hetero_binning_0模型的输出作为输入，这样就可以用来进行变换或预测。
    + isometric_model：这用来指定来自上游组件的模型输入,仅由FATE-1.0中的HeteroFeatureSelection模块使用.HeteroFeatureSelection可以获取HetereFeatureBinning的模型输出，并使用计算出的iv(information value)作为过滤条件。

补充注意：isometric指的是别的异构的模型，比如就是selection用binning的model就异构的，那么使用isometric_model。如果selection用 selection的model就是正常的同质化的模型，那么使用Model即可。


4. `output`：与input相同，可能会发现两种类型的输出，即数据和模型

+ data：指定输出数据名称
+ model：指定输出模型名称

5. `need_deploy` ：true或者false.该字段用于指定组件是否需要部署以进行在线推断。该字段仅用于在线推断，dsl推论。


**简而言之，配置运行DSL，只需三步**
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-runtime-dsl-conf.png)

### DAG DSL Parser

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/FATE/fate-dag-dsl-parser.png)


## 第三步：为每个特定组件定义提交运行时配置

此配置文件用于配置各方之间所有组件的参数

+ 1. initiator：指定发起者的角色和参与者id
+ 2. role：为所有的角色指出所有的参与者id
+ 3. role_parameters：这些参数与角色不同，角色在此处分别定义。请注意每个参数都是List，其每个元素对应此角色中的一方
+ 4. algorithm_parameters：各方之间的参数在这里是相同的

## 第四步：开始模型任务

**1. 上传数据**
在开始一个任务之前，你需要在所有的数据提供者中加载数据。为此，需要准备load_file配置文件，然后运行以下命令：
```
python ${your_install_path}/fate_flow/fate_flow_client.py -f upload -c upload_data.json
```

以下是一个upload_data.json的例子

```
{
      "file": "examples/data/breast_b.csv",
      "head": 1,
      "partition": 10,
      "work_mode": 0,
      "table_name": "hetero_breast_b",
      "namespace": "hetero_guest_breast"
 }
```

我们使用**hetero_breast_b**&**hetero_guest_breast**作为guest方的table_name 和namespace。要使用默认的运行时配置，请设置host方的table_name和namespace为**hetero_breast_a**&**hetero_host_breast**，然后上传数据，数据的路径为examples/data/breast_a.csv.

要使用其他数据集，请修改你的file path,table_name&namespace.请不要上传具有相同的table_name 和namespace的不同数据集。
注意：这一步对于每一个数据提供节点都是必须的（例如，Guest和Host）

**2. 开始你的模型任务**

在这一步，应准备两个与dsl配置文件和提交运行时配置文件相对应的配置文件。请确保配置文件里面的table_name 和namespace的内容必须和upload_data配置里面的匹配上。

```

   "role_parameters": {
       "guest": {
           "args": {
               "data": {
                   "train_data": [{"name": "hetero_breast_b", "namespace": "hetero_guest_breast"}]
               }
           },
```
上面的例子中，这个输入的train_data必须匹配上upload 配置文件。
然后运行以下命令：

```

python ${your_install_path}/fate_flow/fate_flow_client.py -f submit_job -d hetero_logistic_regression/test_hetero_lr_train_job_dsl.json -c hetero_logistic_regression/test_hetero_lr_train_job_conf.json
```

**3.检查日志文件**
现在你可以在下面的路径中检测日志了`${your_install_path}/logs/{your jobid}`

## 第五步：查看结果
FATE现在提供"FATE-BOARD"，用于显示模型loss损失和评估结果。
用浏览器打开`http://{Your fate-board ip}:{your fate-board port}/index.html#/history.`即可


## FATE-FLOW的使用

**1. 如何获取每个组件的输出数据**

```

cd {your_fate_path}/fate_flow


python fate_flow_client.py -f component_output_data -j $jobid -p $party_id -r $role -cpn $component_name -o $output_dir
```

- jobid:the task jobid you want to get.
- party_id:your mechine's party_id, such as 10000
- role:"guest" or "host" or "arbiter"
- component_name：你想获取的组件名字，例如组件名字"hetero_lr_0"
- output_dir：输出目录

**2. 如何获取每个组件的输出模型**
```

python fate_flow_client.py -f component_output_model -j $jobid -p $party_id -r $role -cpn $component_name
```


**3.如何获取任务的日志**
```
python fate_flow_client.py -f job_log -j $jobid -o $output_dir
```

**4.如何停止一个Job**
```
python fate_flow_client.py -f stop_job -j $jobid
```

**5.如何查询一个Job的当前状态**
```
python fate_flow_client.py -f query_job -j $jobid -p party_id -r role
```

**6.如何获取一个Job的运行时配置**
```
python fate_flow_client.py -f job_config -j $jobid -p party_id -r role -o $output_dir
```

**7.如何下载之前已上传的table**
```
python fate_flow_client.py -f download -n table_namespace -t table_name -w work_mode -o save_file
```


注意：work_mode: 0代表standalone,1代表cluster。这取决于之前upload config里面设置的数
will be 0 for standalone or 1 for cluster, which depend on what you set in upload config



* * *


# 使用预测任务

## 使用训练模型进行预测

为了使用训练好的模型去预测，下面几步是必须的。

## 第一步：训练模型
注意以下几点以进行预测：
1. 你应该为需要在预测阶段部署的模块添加/修改“ need_deploy”字段。除FederatedmSample和Evaluation之外，所有模块均将True设置为其默认值，这通常不会在预测阶段运行。"need_deploy"字段设为True,表示此模块应运行训练过程，并且在训练好的模型需要在预测阶段进行部署。
2. 由于将这些模型设置为"need_deploy"，因此还应该给他们配置一个模型输出，除了Intersect模块除外。只有这样才能让Fate-Flow存储已经训练好的模型，并使其在预测阶段可用。
3. 获取训练模型的model_id以及model_version.有两种方法可以做到这一点。
    a. 在提交一个Job之后，将会输出一些模型信息，其中"model_id"和"model_version"是我们感兴趣的字段。
    b. 除此之外，还可以通过以下命令直接获取这些信息。
    
```

python ${your_fate_install_path}/fate_flow/fate_flow_client.py -f job_config -j ${jobid} -r guest -p ${guest_partyid}  -o ${job_config_output_path}

where
$guest_partyid is the partyid of guest (the party submitted the job)
$job_config_output_path: path to store the job_config
```
之后，一个包含了模型信息的数据会被下载到${job_config_output_path}/model_info.json中，你可以从中找到model_id和model_version.

## 第二步：定义你的预测配置文件
此配置文件用于配置用于预测的参数。

+ 1. initiator：指定发起者的角色和参与者id,它应与训练过程相同。
+ 2. job_parameters：word_mode：cluster or standalone,它也应该与训练阶段一致。model_id\model_version：第一步中提到的模型indicator. job_type:job类型，在这种情况下，它应该是"predict"。
+ 3. role：指出所有角色的所有参与ID，并且与训练过程应该相同。
+ 4. role_parameters：为每一个角色设置参数。在这种情况下，"eval_data"意味着需要去被预测的数据，它应同时被Guest和Host方来填写。

## 第三步：开始你的预测任务
在完成了你的预测配置文件后，运行以下命令：
```

python ${your_fate_install_path}/fate_flow/fate_flow_client.py -f submit_job -c ${predict_config}
```

## 第四步：检查运行状态
运行状态可以从FATE_board中检查得到。其URL为 http://${fate_board_ip}:${fate_board_port}/index.html#/details?job_id=${job_id}&role=guest&party_id=${guest_partyid}

## 第五步：下载预测结果
一旦预测任务完成，前100条预测结果记录可在FATE-board中获得。你同样可以通过下面的命令下载所有的结果
```

python ${your_fate_install_path}/fate_flow/fate_flow_client.py -f component_output_data -j ${job_id} -p ${party_id} -r ${role} -cpn ${component_name} -o ${predict_result_output_dir}

where
${job_id}: predict task's job_id
${party_id}: the partyid of current user.
${role}: the role of current user. Please keep in mind that host users are not supposed to get predict results in heterogeneous algorithm.
${component_name}: the component who has predict results
${predict_result_output_dir}: the directory which use download the predict result to.
```

# 其他配置

## 使用spark

1. 部署spark(yarn or standalone)
2. 在fate_flow 服务启动之前，导出SPARK_HOME环境变量(最好将env添加到service.sh)
3. 调整runtime_conf,调整job_parameters字段

```

"job_parameters": {
    "work_mode": ?,
    "backend": 1,
    "spark_submit_config": {
        "deploy-mode": "client",
        "queue": "default",
        "driver-memory": "1g",
        "num-executors": 2,
        "executor-memory": "1g",
        "executor-cores": 1
    }
}
```