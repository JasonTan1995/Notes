## MySQL & Oracle Recursion Menu(递归查找菜单)

### 1. 概述

在项目,菜单总会分级,然后通过子菜单与父菜单这种形式来存储菜单的信息.

![](https://images2017.cnblogs.com/blog/837877/201712/837877-20171219135203365-504537466.png)

### 2. 通过MySQL的函数来完成查询各层菜单

先创建一张Menu表:

```sql
CREATE TABLE `menu` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '菜单id',
  `parent_id` int(11) DEFAULT NULL COMMENT '父节点id',
  `menu_name` varchar(128) DEFAULT NULL COMMENT '菜单名称',
  `menu_url` varchar(128) DEFAULT '' COMMENT '菜单路径',
  `status` tinyint(3) DEFAULT '1' COMMENT '菜单状态 1-有效；0-无效',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=12212 DEFAULT CHARSET=utf8;
```

插入数据:

```sql
INSERT INTO `menu` VALUES ('0', null, '菜单0', ' ', '1');
INSERT INTO `menu` VALUES ('1', '0', '菜单1', '', '1');
INSERT INTO `menu` VALUES ('11', '1', '菜单11', '', '1');
INSERT INTO `menu` VALUES ('12', '1', '菜单12', '', '1');
INSERT INTO `menu` VALUES ('13', '1', '菜单13', '', '1');
INSERT INTO `menu` VALUES ('111', '11', '菜单111', '', '1');
INSERT INTO `menu` VALUES ('121', '12', '菜单121', '', '1');
INSERT INTO `menu` VALUES ('122', '12', '菜单122', '', '1');
INSERT INTO `menu` VALUES ('1221', '122', '菜单1221', '', '1');
INSERT INTO `menu` VALUES ('1222', '122', '菜单1222', '', '1');
INSERT INTO `menu` VALUES ('12211', '1222', '菜单12211', '', '1');
```

查询语句:

```sql
select id from (
              select t1.id,
              if(find_in_set(parent_id, @pids) > 0, @pids := concat(@pids, ',', id), 0) as ischild
              from (
                   select id,parent_id from re_menu t where t.status = 1 order by parent_id, id
                  ) t1,
                  (select @pids := 要查询的菜单节点 id) t2
             ) t3 where ischild != 0
```
其中用到了mysql自身的函数:
1. if（express1,express2,express3）条件语句，if语句类似三目运算符，当exprss1成立时，执行express2,否则执行express3;
2. find_in_set.

这里要详细地讲下find_in_set这个函数:

> 如果字符串str位于由N个子字符串组成的字符串列表strlist中，则返回1到N范围内的值。 字符串列表是由字符串分隔的子字符串组成的字符串。 如果第一个参数是常量字符串而第二个参数是SET类型的列，则FIND_IN_SET（）函数被优化为使用位算术。 如果str不在strlist中，或者strlist是空字符串，则返回0。 如果任一参数为NULL，则返回NULL。 如果第一个参数包含逗号（，）字符，则此函数无法正常工作。

手册中给出的例子:

```sql
mysql> SELECT FIND_IN_SET('b','a,b,c,d');
        -> 2
```



参考文章:

[mysql的函数使用手册](https://dev.mysql.com/doc/refman/5.7/en/functions.html)

[mysql中find_in_set函数的使用](https://www.cnblogs.com/xiaoxi/p/5889486.html)

[mysql递归查找菜单节点的所有子节点](https://www.cnblogs.com/rainydayfmb/p/8028868.html)




