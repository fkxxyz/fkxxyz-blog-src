---
title: tensorflow v1 函数速查（随时更新）
cover: false
date: 2019-12-12 16:38:26
categories:
- 教程
tags:
- 深度学习
- python
---

初学 tensorflow，函数太多了，记不住，文档也难查，各版本函数还有改动，每次百度也麻烦。

于是打算把遇到的所有函数名和大概功能都记录下来，包括老函数名新函数名，用 markdown 进行分类汇总，以便作为速查。

<!--more-->

## 数据

### 占位符与变量

| 功能   | 函数名         | 改动                     |
| ------ | -------------- | ------------------------ |
| 占位符 | tf.placeholder | tf.compat.v1.placeholder |
| 变量   | tf.Variable    |                          |

### 张量

#### 定值

| 功能     | 函数名      |
| -------- | ----------- |
| 填充0    | tf.zeros    |
| 填充1    | tf.ones     |
| 填充数字 | tf.fill     |
| 常量     | tf.constant |

#### 随机函数

| 功能                         | 函数名              | 改动                       |
| ---------------------------- | ------------------- | -------------------------- |
| 正态分布                     | tf.random_normal    | tf.random.normal           |
| 均匀分布                     | tf.random_uniform   | tf.random.uniform          |
| 伽马分布                     | tf.random_gamma     | tf.random.gamma            |
| 截断正态分布（不允许2σ以外） | tf.truncated_normal | tf.random.truncated_normal |

#### 计算函数

##### 数学运算

| 功能         | 函数名      | 符号 |
| ------------ | ----------- | ---- |
| 加法         | tf.add      | +    |
| 减法         | tf.subtract | -    |
| 矩阵点乘     | tf.multiply | *    |
| 矩阵乘法     | tf.matmul   |      |
| 更多数学函数 | tf.math.*   |      |

##### 卷积函数

卷积函数参数较多，这里列出所有参数

###### tf.nn.conv2d

input = [ batch, in_height, in_weight, in_channel ]

| 参数名     | 功能       |
| ---------- | ---------- |
| batch      | 图像数量   |
| in_height  | 图像高度   |
| in_weight  | 图像宽度   |
| in_channel | 图像通道数 |

filter = [ filter_height, filter_weight, in_channel, out_channels ]

| 参数名        | 功能                            |
| ------------- | ------------------------------- |
| filter_height | 卷积核高度                      |
| filter_weight | 卷积核宽度                      |
| in_channel    | 图像通道数（与input的保持一致） |
| out_channels  | 卷积核数量                      |

strides = [ 1, strides, strides, 1]

每一维的步长（对应 input 的每一维）

padding = "SAME" 或 "VALID"

```
"SAME"    下一步步长边界不够时，则补充边界，用 0 填充周围
"VALID"   下一步步长边界不够时，直接丢弃多出来的边界（会造成边界数据的损失）
```

##### 池化函数

| 功能         | 函数名         | 改动             |
| ------------ | -------------- | ---------------- |
| 按最大值池化 | tf.nn.max_pool | tf.nn.max_pool2d |

##### 重组维度

| 功能     | 函数名     |
| -------- | ---------- |
| 重组维度 | tf.reshape |

##### 降维函数

| 功能                                 | 函数名         |
| ------------------------------------ | -------------- |
| 指定轴方向上的所有元素的累加和       | tf.reduce_mean |
| 指定轴方向上的所有元素的最大值       | tf.reduce_max  |
| 指定轴方向上的所有元素的逻辑和       | tf.reduce_all  |
| 指定轴方向上的所有元素的逻辑或       | tf.reduce_any  |
| 指定轴方向上的所有元素的最大值的索引 | tf.arg_max     |

##### 激活函数

| 功能    | 函数名        |
| ------- | ------------- |
| softmax | tf.nn.softmax |
| sigmod  | tf.nn.sigmod  |
| ReLU    | tf.nn.relu    |

## 会话

| 功能         | 函数                  | 改动                            |
| ------------ | --------------------- | ------------------------------- |
| 创建会话     | tf.Session            | tf.compat.v1.Session            |
| 设置互动会话 | tf.InteractiveSession | tf.compat.v1.InteractiveSession |
|              |                       |                                 |

## 操作

| 功能           | 函数                        | 改动                                                         |
| -------------- | --------------------------- | ------------------------------------------------------------ |
| 初始化全局变量 | tf.initialize_all_variables | tf.global_variables_initializer tf.compat.v1.global_variables_initializer |
|                |                             |                                                              |
|                |                             |                                                              |
|                |                             |                                                              |

## 训练相关

### 优化器

| 功能     | 函数                              | 改动                                        |
| -------- | --------------------------------- | ------------------------------------------- |
| 梯度下降 | tf.train.GradientDescentOptimizer | tf.compat.v1.train.GradientDescentOptimizer |
|          |                                   |                                             |
|          |                                   |                                             |

### 保存读取

| 功能   | 函数           | 改动                     |
| ------ | -------------- | ------------------------ |
| 保存器 | tf.train.Saver | tf.compat.v1.train.Saver |
|        |                |                          |

## 