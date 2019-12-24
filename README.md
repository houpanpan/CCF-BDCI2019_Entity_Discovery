# CCF-BDCI2019_Entity_Discovery
CCF-BDCI大数据与计算智能大赛-互联网金融新实体发现-9th  
---
# 赛题描述：  
提取出每篇文章中所有的金融新实体，  
例如某篇文章的文本如下（此处只做示例，实际每篇文章文本长度远远大于以下示例）：  
度小满金融在首次发布的《科创板基金》中提到，今年前5月，京东金融的股票迅速升值。  
那么该篇文章对应的金融新实体列表为：度小满金融;科创板基金。  
***由于京东金融是知名的（金融）实体，所以“京东金融”并不算新实体。***  

# 个人理解：  
结合出题方是“国家互联网应急响应中心”来看，这个赛题的目的实际上是找那种类似校园贷的实体（其实就是不知名的金融实体）。  
**整个赛题要把握住的重点是：**   
①这个赛题实际上偏向于“领域词抽取”（主要针对的是金融领域），但实际上却又不失关键词抽取的要素。  
②注意把握赛题要求中“新实体发现”的“新”，题目中说明了：持有金融牌照的银行、证券、保险、基金等机构、知名的互联网企业如腾讯、淘宝、京东等认为是已知实体（即不能让这些已知实体出现在最终预测结果中）。  

---
# 方案总框架：  
使用了深度模型+传统模型  
深度模型：受限于显存的限制，我们只使用了bert_base，并且搭配的下游模型有bilstm+crf（复赛主力模型）（固定参数）以及全连接+crf（初赛主力模型）  
传统模型：只使用了lgb，共100维特征。虽然特征量大，但由于数据量不大，我们造特征所消耗的时间实际上是并不多的。  
  
--- 
# 具体解决方案：  

## 数据清洗：  
因为本赛题的文本数据大都是由爬虫对html抓取的互联网金融文本，故原始数据集的文本非常脏，主要发现的问题如下：  
①文本中有很多超链接  
②表情、特殊符号  
③多个问号（可能是爬取过程中乱码）  
④无意义文本，比如邮箱、电话之类  
我们通过编写脚本对这些文本进行了清洗。  
- ***特别重要的是，文本不能清洗的特别干净，比如比赛初期我将停用词、除了常用标点（就是回车问号等等一系列的）之外的标点、网页标签都去掉了，把大写的句号全都改成小写的句号（还有大写的逗号，大写的顿号等等），还把所有繁体都转成简体了，这样的预处理所带来的是效果不升反降。因为这些改动实际上都可能会破坏文本原有的结构以及信息，且bert对于停用词、特殊符号等本身就不敏感。  

## 数据预处理：
本赛题提供的文本中，有很多文本的长度甚至都超出了2000个字，***而bert读入文本时，都会把文本截断到maxlen长度以下***，所以bert读入超过maxlen长度的文本后，后面的那些文本就直接“抛弃”了。这样就会有一部分数据得不到利用。  
      - 我的解决方法是将长度超过maxlen的文本切成长度都小于maxlen的几个小段，具体做法请参考：roberta预训练中FULL-SENTENCE的数据预处理方法，                 即用几个连续的句子拼接，直到最大长度512。（这个是我误打误撞想到的，好像前5中也有人用了这个方案。其余很多人都是暴力的直接把句子截到512了，           但我觉得那种方法可能不能保证语义上的连续性）。  
      - 我队友的方法是使用了句子滑窗（也很有借鉴意义），就是例如有句子序列ABCDEF……，若ABCD组成的段能正好小于maxlen，接下来就按固定的句子数滑窗，比         如向前滑两句，即句子的开头由A滑到C，从C开始填充512，即看看CD、CDE、CDEF……直到填充满512为止。  

## 语言模型选择：  
 
 



