---
title: PHPExcel
date: 2017-11-30 10:42:03
tags:
---
# PHPEXcel
----
## 背景
在工作中我们肯定会遇到由于后端管理系统不完善，而导致运营同学无法从管理系统获取想要的数据。亦或程序设计之初没有考虑到完善，导致一些必要的数据没有录入，希望后续补充批量导入。在这个时候我们就需要获取从数据批量导出数据或者导入数据，而无论导入还是导出数据，我们都希望是Excel格式。

## 根据Execel生成sql


1.PHPExcel

准备：我们需要从github获取PHPExcel工具包。地址 `https://github.com/PHPOffice/PHPExcel` 

打开PHPExcel文件夹，看下代码结构。我们需要的主要代码在`Classes`文件下，我们只需要引入`PHPExcel.php`即可。

代码实现
```php
include './code/web_code/PHPExcel/Classes/PHPExcel.php';//引入必要的第三方依赖
$inputFileType = 'Excel2007';//设置Excel的版本
$inputFileName = './targe.xlsx';// 需要导入文件地址


$currentsheet = $objPHPExcel->getSheet(0); //选择Excel页
$maxColumn = $currentsheet->getHighestColumn(); //获取当前页的最大列数
$maxRow = $currentsheet->getHighestRow(); //获取当前页的最大行数

//循环拼接sql
for ($i=2; $i< $maxRow; $i++) { 
    //过滤条件
     if(is_null($currentsheet->getCell('J'.$i)->getValue())||$currentsheet->getCell('J'.$i)->getValue()== '已退款' || $currentsheet->getCell('J'.$i)->getValue() == '待退款' || $currentsheet->getCell('J'.$i)->getValue() == '已经申请退款' || $currentsheet->getCell('J'.$i)->getValue() == '退了' ||$currentsheet->getCell('J'.$i)->getValue() == '没有电话' || $currentsheet->getCell('J'.$i)->getValue() == '0') {

        continue;
     }
     //过滤条件
     if(!is_numeric($currentsheet->getCell('J'.$i)->getValue())||$currentsheet->getCell('J'.$i)->getValue() == '0' || empty($currentsheet->getCell('J'.$i)->getValue()) || $currentsheet->getCell('J'.$i)->getValue() == '') {
        continue;
     }

    $sql_bd_name .= ' WHEN '.$currentsheet->getCell('A'.$i)->getValue().' THEN '.$currentsheet->getCell('J'.$i)->getValue();
    $sql_bd_name .= ','.$currentsheet->getCell('A'.$i)->getValue();
    $res +=1;
}

//输入，当然这里无论我们用fopen或者其他的输入当属都可以，这里我把结果打到了控制台。
echo $sql_bd_name;
echo $res 

```


我们生成的sql是`WHEN`  `THEN`的样式，完整语句应该是 update tablename set seller_jn_activity CASE id
WHEN 1 TEHN 2 
END,
bd_jn_activity CASE id
when 2 THEN 3
END
WHERE in (1,2,3)

发现数据清洗其实是一件很麻烦的事。