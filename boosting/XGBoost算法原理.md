# 概述
+ XGBoost是GBDT算法的一种变种，是一种常用的有监督集成学习算法；是一种伸缩性强、便捷的可并行构建模型的Gradient Boosting算法。
 + XGBoost官网：http://xgboost.readthedocs.io
 + XGBoost Github源码位置：https://github.com/dmlc/xgboost
 + XGBoost支持开发语言：Python、R、Java、Scala、C++等

 # 目标函数
 xgboost的目标函数也就是在GBDT的目标函数上加了一个惩罚项,使得xgboost算法的泛华性能也就更好了.
 ![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/ml/xgboost/xgboost-object-function.png)
 
 有一张很经典的图展示了目标函数和惩罚项的关系.
 
 ![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/ml/xgboost/learning-step-function.png)
  
  ## 回顾一下GBDT的目标函数
  ![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/ml/xgboost/GBDT-object.png)
  
  ## 再看XGBoost的目标函数
  
  