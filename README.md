# Spam-identification
# 垃圾短信识别
  
    本文对最近自己做过的一个垃圾短信识别的项目进行整理和描述，下面主要从思路和代码使用等问题进行介绍。
## 建模方案
* 特征抽取：Tf-idf、`特征符号【 + 】计数`、`\W款\W计数`
* 特征选择：方差分析（ANOVA）
* 建模方法：初期方案 – LR，后期方案 – `bagging+LR`
* 模型选择：L2正则、5 fold + Accuracy
## 数据集
* 原始数据集：10973条样本
 * 正样本（垃圾短信）：6990
 * 负样本（非垃圾短信）：3983
* 去重、校对、扩增负样本（`随机上采样`）后的数据集：8928条样本
 * 正样本（垃圾短信）：4464
 * 负样本（非垃圾短信）：4464
数据集特点：包含大量的噪声数据，即存在不多错误标记的样本数据。<br>
## 特征提取方案
* 对所有样本生成所有词的Tf-ifd特征
* 使用ANOVA得到每种特征的F得分和p值
* 抛弃F-score<1或者p-value>0.05的词的tf-idf特征
* 剩下的所有tf-idf特征与 “【” + “】” 、\W款\W的计数ont-hot编码后的特征构成新的sample特征集
## 模型构建
### L2正则化参数的确定
* 特征集构建（所有样本数据）
* 获取检验曲线（5-fold+Acc）
### Bagging思想
* `随机重采样`（采样样本数==原始样本数）
* 基于采样后sample，构建stop word文件和特征集
* 基于stop word文件和特征集，建立模型
* 以上过程重复50次，构建50个不同的模型
* 针对测试样本的预测类型，50个模型`投票`表决
* 投票策略:
  * 策略1：少数服从多数<br>
  * 策略2： 对于每一个样本的模型输出概率值进行加和、然后取均值，然后根据计算结果预测label（阈值：0.5）<br>
## 自己对于建模的一点思考
* 数据集中包含大量的噪音数据，通过随机重采样方式构建多个模型，可以减少噪声数据对模型训练的影响，于是采用了bagging思想
* 文本生成的词库过大，远大于样本数，这就增加了过拟合的风险，于是采用了，如下策略：
  * L2正则化
  * 通过ANOVA特征选择手段，获得每个特征词的F值和p值，筛选出F值<=1.0或者p值>0.05的特征词作为stop word不参与Tf-idf特征向量的构建。
    * ANOVA优势：特征对类别内部和类别之间的影响可以量化
    * Tf-idf优势：暗含了词袋模型，量化了词的重要程度

## 程序脚本的使用说明
#### step_1_buildModel_for_trainData.py  \# 用于构建模型 - 模型保存在model文件夹下
   程序说明：
>> 基于bagging的思想，随机重采样获得50个不同的基分类器<br>
   
   用法：
>> python step_1_buildModel_for_trainData.py -t train.txt

#### step2_obtainPredictLabel_for_testData.py \# 给定无标签数据，用于生成样本的预测标签
  程序说明：
>> 基于现有模型，预测无标签样本的分类标签

  用法： 
>> python step2_obtainPredictLabel_for_testData.py -t test.txt -o output.txt

#### ObtainSampleError_main.py
  程序说明：
>> 用于 核实样本数据（手动将数据中标记不正确的样本修正）

#### train_crossVaildation_main.py
  程序说明：
>>该程序用于获得以下内容：<br>
>> * 绘制学习曲线 - 损失函数随样本量m的变化曲线<br>
>> * 绘制验证曲线 - ACC 随 正则化参数的 变化曲线 （选择最优正则参数）<br>
>> * 查看验证集错误分类的样本，（用于发现新特征） - 将训练数据集分为训练集和验证集， 用训练集训练模型， 用验证集测试，输出验证集错误分类的样本<br>
>> * 查找错误分类的样本 ， 并将其保存到文件中 - 用于校正样本数据<br>
>> * 创建初步的模型 model保存到文件中 - 查看模型性能<br>
