---
title: PHP操控mongodb命令小结
date: 2017-02-14 10:08:22
tags:
	- mongodb
---
最近学习了非关系型数据库mongodb，以json方式来存储数据，数据内容更灵活。以下是php操作mongodb常用命令，记录一下，便于以后查询。
<!--more-->
```
<?php
// 参考 http://php.net/manual/zh/mongo.sqltomongo.php

/* 查询区间范围 */
$rangeQuery = array('count' => array('$gt' => 3, '$lt' => 9));
$doc = $jihe->find($rangeQuery);

/*自定义函数搜索集合*/
$rangeQuery = "function() {
    return this.count == '1';
}";
$doc = $jihe->find(array('$where'=> $rangeQuery));

//in操作符
$doc = $jihe->find(array(
	'count'=> array('$in' => array(4, 6))
	));

//输出集合 使用 iterator_to_array() 会让驱动将强制载入所有搜索结果集到内存，所以对超过内存大小的结果集不要这么做！
方法一：
$doc = $jihe->find();
echo json_encode(iterator_to_array($doc));

方法二：
$doc = $jihe->find();
foreach($doc as $id => $value)
{
	 
	echo json_encode($value);
}

//计算文档数量
$doc = $jihe->count();

// 排序 -1降序   1升序
$doc = $jihe->find()->sort(array("count" => -1));

//获取count=3的文档中的title属性
$doc = $jihe->find(array("count" => 3), array("title"=>1));

// 从第5个开始取10个
$doc = $jihe->find()->limit(10)->skip(4);

// 或者条件句
$doc = $jihe->find(array('$or'=>array(
		array("count"=>3), array("title"=>"mongodb1")
	)
	));

// 限制数量1个
$doc = $jihe->find(array('$or'=>array(
		array("count"=>3), array("title"=>"mongodb1")
	)
	))->limit(1);

// 获取符合条件的数据数量
$doc = $jihe->find(array('$or'=>array(
		array("count"=>3), array("title"=>"mongodb1")
	)
	))->count();

// 获取含有count字段的数据数量
$doc = $jihe->find(
		array("count"=>array('$exists'=>true))
	)->count();

// 获取含有count字段的数据
$doc = $jihe->find(
		array("count"=>array('$exists'=>true))
	);

// 更改：count=0的数据，title改为mongdb,此处‘$set’参数表示修改，如不加此参数，会重写原数据，将数据的其他字段抹掉
$doc = $jihe->update(
		array("count"=>0),array('$set'=>array("title"=>"mongodb"))
	);

// 在title=mongodb的那条数据，把count累加10,用‘$inc’方法
$doc = $jihe->update(
		array("title"=>"mongodb"), array('$inc'=>array("count"=>10))
	);

// 删除title=mongodb1的那条数据
$doc = $jihe->remove(
		array("title"=>"mongodb1")
	);
?>
```
