[TOC]


















## 一、互联网数据爬取

### 1. 流程的整体概述

​		在获取详情页内容的过程中，采用如下流程图中的步骤进行，首先进入人人车的“概述页”（每页包含可以点进去的40个详情页链接），当爬取该页后，可以解析出约为40个的详情页链接。通过获取的详情页链接进入详情页并爬取后，再解析详情页信息。

<img src="https://i0.hdslb.com/bfs/album/0f7711bd2982b942eecd55c14ad98a86da3f0401.png" alt="image-20220619091127677" style="zoom:80%;" />



### 2. 概述页的爬取

#### 2.1 页面介绍

概述页如图所示，每页可以解析出约为40个的详情页链接。

<img src="https://i0.hdslb.com/bfs/album/57840e9d81ab5f986e65193e9789093c53f9cb2d.png" alt="image-20220619091526540" style="zoom: 33%;" /> 



#### 2.2 url构造

​		经分析，该网页为html静态网页，requests.get(url)即可实现爬取网页html信息的功能。并且该网页无需登录，因此不需要加入cookie信息，对于爬取工作来说较为友好。该网页的具体结构如下：

​                                  https://www.renrenche.com/cn/ershouche/pr-20-30/p2

pr-20-30代表价格区间（价格区间可以自行设置）、p2表示第二页。因此可以通过更改价格区间、页数来访问到不同的页面。,页数范围为1-50，构造的价格区间范围：

```python
#价格区间范围
price_range = ['1-2', '2-3', '3-4', '4-5', '5-6', '6-7', '7-8', '8-9', '9-10', '10-11', '11-12', '12-13', '13-14', '14-15', '15-16', '16-17', '17-18', '18-19', '19-20', '20-21', '21-22', '22-23', '23-24', '24-25', '25-26', '26-27', '27-28', '28-29', '29-30''30-31', '31-32', '32-33', '33-34', '34-35', '35-36', '36-37', '37-38', '38-39', '39-40']

#price表示价格范围，str(i)为页数
url = r"https://www.renrenche.com/cn/ershouche/pr-" + price + "/p" + str(i) + r"/"
```



#### 2.3 多线程获取网页信息

​		在具体的爬取过程中，由于要与服务器进行三次握手连接，所以在访问量较大时比较消耗时间。因此采用多线程技术，开启50个线程并发访问，进行调度。能够有效减少爬取时间。经试验，原来爬取50个网页需要50-60秒，现在仅需10-15秒，提高了近5倍的效率。

<img src="https://i0.hdslb.com/bfs/album/463d8ac988906cc2b7c2eea2fa5ae65708837797.png" alt="image-20220619150826763" style="zoom:80%;" /> 

<img src="https://i0.hdslb.com/bfs/album/85decb13d8cf3f13a60e6d3384c1a2774f25ec7d.png" alt="image-20220619150843292" style="zoom:80%;" />  



#### 2.4 详情页链接解析：

​		通过对于网页中Html信息的分析，其中详情页链接位于 `class_="span6 list-item car-item"`，或`class_="span6 list-item car-item margin-left-0"` 其中的 `href` 元素之中。通过构建：https://www.renrenche.com/href 的方式，进入各个车的详细介绍页面。

![image-20220619093035808](https://i0.hdslb.com/bfs/album/a3ce4c17742d7c57e77a2da403ad6ee339b1517d.png) 



#### 2.5 链接数据处理

​		如图，这种链接是无效的，可以在获得了数据集后，遍历一遍删去。最终，得到了52000行详情页的链接（请见 `data/car_url_new` ）。

<img src="https://i0.hdslb.com/bfs/album/055643ab875747966726d58ae9c15c9f46bad6d6.png" alt="image-20220619094430886" style="zoom:80%;" /> 

<img src="https://i0.hdslb.com/bfs/album/cc7305fde4c3e3baf837950d68b61fbf39f9f857.png" alt="image-20220530153050466" style="zoom:80%;" /> 



### 3. 详情页内容爬取

#### 3.1 页面介绍

​		页面中，有价值的信息主要位于如图两个区域中。

<img src="https://i0.hdslb.com/bfs/album/be253ef2d1f1785b2f0ce9b7a90e06e78811a0c4.png" alt="image-20220619094558140" style="zoom: 33%;" /> 

<img src="https://i0.hdslb.com/bfs/album/cd0ba22bfdc0e7b9d0815f94358937804c0e4c1c.png" alt="image-20220619094612878" style="zoom: 40%;" /> 

#### 3.2 url构造

​		采用上述获得的 `data/car_url_new` 中的数据构造访问链接：https://www.renrenche.com/href 即可访问到各个详情页。

<img src="https://i0.hdslb.com/bfs/album/2a29dcd2bac252d997f7a8c542653d2a0e8e0f6b.png" alt="image-20220619094943236" style="zoom:80%;" />  



#### 3.3 网页内容的获取

​		同上，采用多线程的方法获取详情页Html信息

<img src="https://i0.hdslb.com/bfs/album/2cf7a2c45e9e844a2feeb7bfc3ac081178c135d9.png" alt="image-20220619151030573" style="zoom:80%;" />  



#### 3.4 网页内容解析

​		首先进行的，是较为粗粒度的解析。分别有如下几个方面：车辆名称、二手车售价、新车价格、具体信息、标注信息、检测报告。

<img src="https://i0.hdslb.com/bfs/album/d70882f5dab2c322aa4f996bef9b1af5de4422f4.png" alt="image-20220619095254692" style="zoom: 50%;" /> 

<img src="https://i0.hdslb.com/bfs/album/2e382da19246d84ffb64184658f74630db66671e.png" alt="image-20220619095606664" style="zoom: 50%;" /> 



初步爬到的单条信息如下：包含文本描述的“检测报告”。

需要处理的中文：车辆名称、上牌时间、车辆所在地、外迁查询、自动/手动变速箱、上架时间、检测报告详情

<img src="https://i0.hdslb.com/bfs/album/a7f139b980484decd807e432c6e1f204c1c50ff7.png" alt="image-20220619095657214" style="zoom: 80%;" /> 

数据集（ `data/car_detail` ）样子：

![image-20220617173755691](https://i0.hdslb.com/bfs/album/8897320a0efd639332b041a210b6926e8f89b9ec.png)



## 二、数据预处理

### 1. 数据集的中文文本信息处理

​		在获得的数据集 `data/car_detail` 中，车辆名称、具体信息、标注信息、检测报告都需要进行中文文本信息的处理。

#### 1.1 车辆名称

​		车辆信息如图所示，据调查，在二手车市场上，主流品牌的车辆相对于非主流车辆来说会更加保值一些，首先选取 “ - ” 前的品牌名称，以人人车主要显示的品牌为“主流品牌”，主流品牌为1，非主流品牌为0。

![image-20220619101123245](https://i0.hdslb.com/bfs/album/fc567ec0e0fe14ec48ffbf00d10de42a9165f1b4.png)

<img src="https://i0.hdslb.com/bfs/album/bd759c591440ad325dffa6c054f61ff91012e7ef.png" alt="image-20220619100922040" style="zoom:80%;" /> <img src="https://i0.hdslb.com/bfs/album/2459c8b659807c3e42c6b20fcbce6871a3203215.png" alt="image-20220619101343499" style="zoom:80%;" />



#### 1.2 具体信息

​		具体信息部分可以拆分出多项数据：行驶里程、上牌时间、车牌所在地、外迁查询、变速箱、过户记录、上架时间。

<img src="https://i0.hdslb.com/bfs/album/cdcc8be747cf1987b1c9cff606b5408d0c910c91.png" alt="image-20220619101516706" style="zoom:80%;" /> 

​		

（1）其中，行驶里程、上牌时间、过户记录、上架时间可以采用正则表达式的方法提取出其中的数字。如果没有上架时间项，说明上架时间很近，设置为1。

（2）变速箱只有手动自动两种：手动设置为0，自动设置为1。

（3）车牌所在地：从上牌难度来看，城市不同同样二手车的价格也会有不同。一般来说一线城市上牌最为困难，新一线城市较为困难，其他城市一般，分为三档：2、1、0。

（4）外迁查询：主要有国三、国四、国五三种。其中国三的上路受到限制，无法迁入外地。国四迁入外地受限，国五最佳。设置成一个二维向量，国三[1,1]、国四[0,1]、国五[0,0]。

<img src="https://i0.hdslb.com/bfs/album/89a72e9034b66976b0d4700873df82d593aad426.png" alt="image-20220619102412457" style="zoom:80%;" /> 



#### 1.3 标注信息

标注信息如图所示，分为 超值/急售/0过户/准新车 四项。设置一个1*4的向量来进行表示

<img src="https://i0.hdslb.com/bfs/album/e53c1bcb864ead8558a1fe45568c52277fb9ee82.png" alt="image-20220619102527874" style="zoom:80%;" /> <img src="https://i0.hdslb.com/bfs/album/5b1334872e262635ab8a355293df0bf575075d78.png" alt="image-20220619102656064" style="zoom:80%;" />



#### 1.4 检测报告

在该部分详细描述了车辆的检测情况，具体来看，可以提取两个信息：综合车况、外观检测。

综合车况分为：优秀/良好/较好。分别为3、2、1。

外观检测以瑕疵数目记。

<img src="https://i0.hdslb.com/bfs/album/e0c92d70edc35cda407dea58e6506a8b03190be3.png" alt="image-20220619102810772" style="zoom:80%;" /> 

<img src="https://i0.hdslb.com/bfs/album/c98e578de18c1d12a9e48c196f03dd3e5bf65b24.png" alt="image-20220619102748516" style="zoom:80%;" /> 



### 2. 数据集的所有特征

数据集整理后的表格项如图所示：

| 项目名称   | 数据类型 | 取值情况        | 意义解释                                            |
| ---------- | -------- | --------------- | --------------------------------------------------- |
| Output：   |          |                 |                                                     |
| 二手车价格 | float    | 单位：万元      | 输出项                                              |
|            |          |                 |                                                     |
| Input：    |          |                 |                                                     |
| 品牌       | int      | 主流品牌1/其他0 | 主流品牌的二手车相对保值                            |
| 新车购入价 | float    | 单位：万元      | 当时购入的价格，具有参照意义                        |
| 行驶里程   | float    | 单位：万公里    | 可以体现车辆的使用、损耗情况                        |
| 使用时间   | int      | 单位：月        | 使用时间久的车相对不保值                            |
| 车牌所在地 | int      | 范围：2/1/0     | 一线/新一线/普通城市，反映了经济状况，供给/需求市场 |
| 外迁标准1  | int      | 范围：1/0       | 针对排放标准，国三上路受限，价格受到影响            |
| 外迁标准2  | int      | 范围：1/0       | 针对排放标准，国三国四外迁受限，价格受到影响        |
| 变速箱     | int      | 范围：1/0       | 手动档0，自动档1。反映车况，需求                    |
| 过户次数   | int      | 单位：次        | 过户越多，车况更复杂，相对越不保值                  |
| 上架时间   | int      | 单位：天        | 上架时间长，卖主可能会相对降价等                    |
| 超值       | int      | 范围：1/0       | 超值标记，意味着价格可能偏低                        |
| 急售       | int      | 范围：1/0       | 急售标记，意味着价格可能偏低                        |
| 0过户      | int      | 范围：1/0       | 0过户标记，意味着过户少                             |
| 准新车     | int      | 范围：1/0       | 准新车标记，意味着车更新，价格可能相对高            |
| 综合车况   | int      | 范围：3/2/1     | 人人车车检情况总结，数值高车况好                    |
| 外观检测   | int      | 单位：个        | 外观检查，瑕疵的数目，影响价格                      |

<img src="https://i0.hdslb.com/bfs/album/b3274133ff28fd2c8c639209ded47d3ba1c07cd9.png" alt="image-20220619111243610" style="zoom:80%;" /> 



### 3. 数据集的性质展示

处理后的数据集性质：

#### 3.1 数据行数、列数：

<img src="https://i0.hdslb.com/bfs/album/7eadfffa27732ec8868f3648ec85b7ac911b5d92.png" alt="image-20220619151129523" style="zoom:80%;" />  



#### 3.2 数据统计特征：

<img src="https://i0.hdslb.com/bfs/album/440f406c4d4fdef1d4c593d48c4d7937683d543d.png" alt="image-20220619151219326" style="zoom:80%;" />  

<img src="https://i0.hdslb.com/bfs/album/cbd9c4a042fc8f567dde7fd34ba96bdb0f88483f.png" alt="image-20220619151234000" style="zoom:80%;" />  



#### 3.3 数据性质：

<img src="https://i0.hdslb.com/bfs/album/af87d7ac88e1e143b47b44046054a9474b7908cf.png" alt="image-20220619151252031" style="zoom:80%;" /> 



#### 3.4 值缺失情况：

<img src="https://i0.hdslb.com/bfs/album/c32047093b473788debe9969bcf9f9345f09d64d.png" alt="image-20220619111554461" style="zoom:80%;" /> 



## 三、数据建模、调优、测试

### 1. 数据集划分：

首先，采用7：3的比例划分训练集与测试集：

<img src="https://i0.hdslb.com/bfs/album/f1aa79bdd7fe455e745003749ddd5311d2231fbe.png" alt="image-20220619151311433" style="zoom:80%;" /> 

<img src="https://i0.hdslb.com/bfs/album/692f3fe2ab3b62a4406d7a97451d65d228452060.png" alt="image-20220619151323820" style="zoom:80%;" /> 



### 2. 数据建模训练

#### 2.1 简单的回归算法

首先，采用了几种简单的回归算法：线性回归，岭回归，Lasso回归。效果不太理想。

以线性回归为例，$R2$ 值在0.766左右，均方误差19.72，平均绝对误差：3.28，平均相对误差34.6%。在效果不太理想。

<img src="https://i0.hdslb.com/bfs/album/4e23e1053adba1ce8cd9c0dfca33497dc8bb1ddf.png" alt="image-20220619151406226" style="zoom:80%;" /> 



#### 2.2 SVM高斯核函数

采用SVM支持向量机，并采用核函数：rbf（其他几个核函数经测试效果没有这个好）。

$R2$ 值在0.843，均方误差13.3，平均绝对误差：2.43，平均相对误差19.4%。

<img src="https://i0.hdslb.com/bfs/album/6981da8245da968cc072507b794c7a5cba539d78.png" alt="image-20220619151455609" style="zoom:80%;" />  



#### 2.3 KNN回归算法

采用KNN算法，参数：`n_neighbors=2`

$R2$ 值在0.860，均方误差11.8，平均绝对误差：2.28，平均相对误差19.2%。

<img src="https://i0.hdslb.com/bfs/album/8a6286d9a031ebbc78b0861afef64ce0bd397778.png" alt="image-20220619151530479" style="zoom:80%;" />  



#### 2.4 回归树算法

采用回归树算法，参数：`criterion = "mse",min_samples_leaf = 5`

$R2$ 值在0.901，均方误差8.37，平均绝对误差：1.89，平均相对误差16.1%。

<img src="https://i0.hdslb.com/bfs/album/b0fab5134595036d5dd5422522da8a8cc1a4e98f.png" alt="image-20220619151606107" style="zoom:80%;" /> 



#### 2.5 随机森林回归

采用随机森林的回归算法，默认参数。

$R2$ 值在0.939，均方误差5.16，平均绝对误差：1.45，平均相对误差12.8%。

<img src="https://i0.hdslb.com/bfs/album/889a9071fbab87ff8de0d43242c3809579d80186.png" alt="image-20220619151634340" style="zoom:80%;" /> 



#### 2.6 XGBoost 算法

经过调查得知，基本所有的机器学习比赛的冠军方案都使用了XGBoost算法。

采用默认参数下XGBoost算法：
$R2$ 值在0.915，均方误差7.14，平均绝对误差：1.86，平均相对误差16.0%。

<img src="https://i0.hdslb.com/bfs/album/cba9787cd21909f06f03774fd5e8e0065135b729.png" alt="image-20220619151720612" style="zoom:80%;" /> 



### 3. XGBoost 算法参数调优

经过调查得知，基本所有的机器学习比赛的冠军方案都使用了XGBoost算法。

所以针对XGBoost算法尝试进行参数调优。

| 参数             | 默认值       | 可选值                                                       | 推荐值        | 含义                           |
| ---------------- | ------------ | ------------------------------------------------------------ | ------------- | ------------------------------ |
| n_estimators     | 100          | int                                                          | 100           | 弱评估器的数量                 |
| booster          | ‘gbtree’     | [‘gbtree’, ‘gblinear’ ,  ‘dart’]                             | ‘gbtree’      | 弱学习器的类型                 |
| learning_rate    | 0            | 0~1                                                          | 0.1,0.015...  | 防止过拟合                     |
| gamma            | 0            | 0~1                                                          | 1,0.9,0.7...  | 叶节点分支所需损失减少的最小值 |
| reg_alpha        | 0            | 0~1                                                          | 1,0.1,0.01... | L1正则化权重项                 |
| reg_lambda       | 0            | 0~1                                                          | 1,0.1,0.5...  | L2正则化权重项                 |
| max_depth        | 6            | int                                                          | 9,12,15,17... | 树的最大深度                   |
| min_child_weight | 1            | int                                                          | 1,3,5,7...    | 孩子节点中最小的样本权重和     |
| subsample        | 1            | 0~1                                                          | 1,0.9...      | 弱学习器训练比例，防止过拟合   |
| colsample_bytree | 1            | 0~1                                                          | 1,0.9...      | 特征的随机采样比例             |
| objective        | “reg:linear” | “reg:linear”：线性回归<br/>“reg:logistic”：逻辑回归<br/>“binary:logistic”：二分类输出为概率<br/>“multi:softmax”：多分类问题 | “reg:linear”  | 指定学习任务及学习目标         |
| eval_metric      | ‘rmse’       | ‘rmse’：用于回归任务<br/>‘mlogloss’：用于多分类任务<br/>‘error’：用于二分类任务<br/>‘auc’：用于二分类任务 | ‘rmse’        | 评估方法                       |



==贪心算法==逐个调参寻找局部最优：

<img src="https://i0.hdslb.com/bfs/album/afc1ef220f5cdb92d7145c6a8f4adee6dd0ec6c0.png" alt="image-20220619130047709" style="zoom:80%;" /> 

调参后的结果：

$R2$ 值在0.947，均方误差4.49，平均绝对误差：1.25，平均相对误差11.8%。

<img src="https://i0.hdslb.com/bfs/album/3c00a2b9df2fbbce2f53e89ace3a80bd904b52c2.png" alt="image-20220619151826238" style="zoom:80%;" /> 



## 四、总结

首先，给出各个训练结果的展示：

值在0.843，均方误差13.25，平均绝对误差：2.43，平均相对误差19.4%。

| 算法          | R2    | 均方误差 | 平均绝对误差 | 平均相对误差 |
| ------------- | ----- | -------- | ------------ | ------------ |
| XGBoost调参后 | 0.947 | 4.49     | 1.25         | 11.8%        |
| XGBoost       | 0.915 | 7.14     | 1.86         | 16.0%        |
| 随机森林回归  | 0.939 | 5.16     | 1.45         | 12.8%        |
| 回归树算法    | 0.901 | 8.37     | 1.89         | 16.1%        |
| KNN回归算法   | 0.860 | 11.9     | 2.28         | 19.2%        |
| SVM高斯核函数 | 0.843 | 13.2     | 2.43         | 19.4%        |
| 线性回归      | 0.766 | 19.7     | 3.28         | 34.6%        |
| 岭回归        | 0.766 | 19.7     | 3.28         | 34.6%        |
| Lasso回归     | 0.739 | 22.0     | 3.43         | 35.0%        |

![image-20220619152501688](https://i0.hdslb.com/bfs/album/e97e5bc68380fda7d019ea5e0f14173d1dde8d8b.png) 



最后，在采用 $XGBoost$ 算法并进行参数调优后，获得的 $R2$ ：$0.947$，均方误差：$4.49$，平均绝对误差：$1.25$，平均相对误差：$11.8$%。

![image-20220619152649139](https://i0.hdslb.com/bfs/album/940994129d68a794572a7042f6ee733a339e2e4e.png) 
