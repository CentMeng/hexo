---
title: python使用决策树算法
date: 2016-07-19 23:16:52
categories:
- 大数据技术
tags:
- python
---

<img src="/img/data.jpg" />

步骤：

1. 导入数据
	
	***利用csv导入数据***
2. 分离数据

	***将输入数据，和结果数据分离出来***
	
3. 转化数据

	***将输入数据和结果数据转化成0，1格式的数据
	
	输入数据转化如下
	
	```python
	dummyX[[ 0.  0.  1.  0.  1.  1.  0.  0.  1.  0.]
		   [ 0.  0.  1.  1.  0.  1.  0.  0.  1.  0.]
 		   [ 1.  0.  0.  0.  1.  1.  0.  0.  1.  0.]
   		   [ 0.  1.  0.  0.  1.  0.  0.  1.  1.  0.]
 		   [ 0.  1.  0.  0.  1.  0.  1.  0.  0.  1.]
 		   [ 0.  1.  0.  1.  0.  0.  1.  0.  0.  1.]
 		   [ 1.  0.  0.  1.  0.  0.  1.  0.  0.  1.]
		   [ 0.  0.  1.  0.  1.  0.  0.  1.  1.  0.]
 		   [ 0.  0.  1.  0.  1.  0.  1.  0.  0.  1.]
 		   [ 0.  1.  0.  0.  1.  0.  0.  1.  0.  1.]
 		   [ 0.  0.  1.  1.  0.  0.  0.  1.  0.  1.]
 		   [ 1.  0.  0.  1.  0.  0.  0.  1.  1.  0.]
 		   [ 1.  0.  0.  0.  1.  1.  0.  0.  0.  1.]
 		   [ 0.  1.  0.  1.  0.  0.  0.  1.  1.  0.]]
 	其中每一列对应的数据是:
 	['age=middle_aged', 'age=senior', 'age=youth', 'credit_rating=excellent', 'credit_rating=fair', 'income =high', 'income =low', 'income =medium', 'student=no', 'student=yes']

	```
	
	输出数据转化如下
	
	```python
	dummY[[0]
 		  [0]
 		  [1]
 		  [1]
 		  [1]
 		  [0]
 		  [1]
 		  [0]
 		  [1]
 		  [1]
 		  [1]
 		  [1]
 		  [1]
		  [0]]
	其中每一行都是最终结果，此例是是否购买的结果
	```
	
4. 建模

	```python
	 clf = tree.DecisionTreeClassifier(criterion='entropy') #声明使用决策树ID3算法
 clf = clf.fit(dummyX,dummyY);
	```
5. 生成dot文件
   
   ```python
   with open("allEletronicInformationGainOri.dot", 'w') as f:
    f = tree.export_graphviz(clf,feature_names=vec.get_feature_names(),out_file=f)
   ```
6. 利用graphviz工具将dot文件导入把决策树画出来（其中分支节点的上下关系根据<b>信息熵</b>的大小来评估的）

7. 测试和使用

	```python
#newRowX是0，1输入数据
predictedY = clf.predict(newRowX)
print("新数据结果" + str(predictedY))
	```
###信息熵的算法：(变量的不确定性越大，熵越大)  =负的 每一个发生的概率 乘以 以2为低概率的对数###

<img src="/img/data/xinxishanggongshi.png" />

<img src="/img/data/xinxishang.png" />
	

[代码地址](https://github.com/CentMeng/decisiontree)
 
