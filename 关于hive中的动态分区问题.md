# 关于hive中的动态分区问题

[TOC]



## 问题描述

​	在学习数仓时，遇到从几个表中读数据存入到新的表，并涉及到动态分区问题，即在查询语句下新加上一行查询，起初不懂这个动态分区是如何为分区字段赋值的，通过自己实践终于搞懂了，下面详细解析一下

## sql语句

### 建表语句

```sql
CREATE EXTERNAL TABLE dwd_tool_coupon_used_inc
(
    `id`           STRING COMMENT '编号',
    `coupon_id`    STRING COMMENT '优惠券ID',
    `user_id`      STRING COMMENT '用户ID',
    `order_id`     STRING COMMENT '订单ID',
    `date_id`      STRING COMMENT '日期ID',
    `payment_time` STRING COMMENT '使用(支付)时间'
) COMMENT '优惠券使用（支付）事务事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dwd/dwd_tool_coupon_used_inc/'
    TBLPROPERTIES ("orc.compress" = "snappy");
```

### 插入语句

```sql
insert overwrite table dwd_tool_coupon_used_inc partition (dt)
select
    data.id,
    data.coupon_id,
    data.user_id,
    data.order_id,
    date_format(data.used_time,'yyyy-MM-dd') date_id,
    data.used_time,
    date_format(data.used_time,'yyyy-MM-dd')  -- 动态分区添加
from ods_coupon_use_inc
where dt = '2022--06-08'
and type = 'bootstrap-insert';
```

进行动态分区时，在查询的最后加上一行，而不进行动态分区，则将其删除（这就是在学习中引起我困惑的原因）



注意：创建分区表是，不仅是括号中的六个字段(id,coupon_id,user_id,order_id,date_id,payment_time)，还包括分区字段(dt)，**在静态分区时，查询6条语句，分区`dt='xxxx-xx-xx'`为第七条语句，而在动态分区，没有在插入语句中给定分区，故会缺少一个字段，因此要自行加上**

## 总结

这个问题实际上是个非常简单的问题，可是因为在学习时学的一知半解，因此被困扰了好久



最后：在学习过程中要多进行实操，否则感觉自己学会了，但自己一上手就乱了阵脚！！！