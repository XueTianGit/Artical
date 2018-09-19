title: 屌屌的Web树--保存分类
date: 2016/5/19 8:45:55  
categories: Database
---


# 屌屌的Web树--保存分类#
> 在实际中，我们可能要把分类信息保存到数据库中，这时当然需要设计一个表来保存这些分类信息啦。


## 自关联表来保存 ##
关联表在理论上可以保存无限分类，而在实际上却是不可以的


## 实际开发中使用的分类树 ##

![](http://7xrbxa.com1.z0.glb.clouddn.com/Database-%E5%88%86%E7%B1%BB%E6%A0%91.png)

这种树状节点的特点：

	1.每一个节点都有一个左右值。
	2.如果右值-左值=1，则代表当前节点为叶子节点。
	3.如果右值-左值>1，则代表当前节点有孩子节点，值在左右值之间的所有节点，即为当前结点的所有孩子节点。

数据库表设计：

	create table category
	(
		id varchar(40) primary key,
		name varchar(100),
		lft int,
		rgt int
	);
	insert into category values('1','商品',1,18);
	insert into category values('2','平板电视',2,7);
	insert into category values('3','冰箱',8,11);
	insert into category values('4','笔记本',12,17);
	insert into category values('5','长虹',3,4);
	insert into category values('6','索尼',5,6);
	insert into category values('7','西门子',9,10);
	insert into category values('8','thinkpad',13,14);
	insert into category values('9','dell',15,16);


问题：为了在页面中显示树状结构，需要得到所有结点，以及每个结点在树中的层次：

解决思路：

	1、要得到结点的层次，就是看节点有几个父亲，例如长虹有2个父亲，则它所在层次就为2。
	
	2、如何知道每一个节点有几个父亲呢？这个表有个特点，父亲和孩子都在同一个表中，为得到父亲所有的孩子，可以把这张表想像成两张表，一张用于保存父亲，一张表保存孩子，如下所示：
	select * from category parent,category child;
	3、父亲下面的孩子有个特点，它的左值>父亲的左值，并且<父亲的右值，如下所示
	select * from category parent,category child where child.lft>=parent.lft and child.rgt<=parent.rgt;
	以上语句会得到父亲下面所有的孩子。
	
	4、对父亲所有孩子的姓名进行归组，然后使用count统计函数，这时就会知道合并了几个孩子，合并了几个孩子姓名，这个孩子就有几个父亲，从而知道它所在的层次
	select child.name,count(child.name) depth from category parent,category child where child.lft>=parent.lft and child.rgt<=parent.rgt group by child.name;
	
	5、最后根据左值排序即可
	select child.name,count(child.name) depth from category parent,category child where child.lft>=parent.lft and child.rgt<=parent.rgt group by child.name order by child.lft;