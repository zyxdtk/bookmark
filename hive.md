## Hive 官方文档

http://hive.apache.org/ hive官网

## Hive 使用手册

[LanguageManual UDF](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF) Hive运算符和函数。

[LanguageManual WindowingAndAnalytics](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+WindowingAndAnalytics) **窗口函数和分析函数**。所谓窗口函数就是取（前或后）N个数据，所谓分析函数就是打上诸如第几名的标签。不想看英文可以看这篇 [HIVE 窗口及分析函数 应用场景](http://yugouai.iteye.com/blog/1908121) 。

[LanguageManual SortBy](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SortBy) 关于order/sort/cluster/distribute by等的区别。order是全局有序，sort知识在单个reduce内有序。distribute 是按照columns进行hash，然后保证指定列值相同的数据都进入同一个的reduce。cluster 等价于 distribute by + sort by，就是不仅按照指定的列进行hash，还保证每个reduce内的数据按照指定列是有序的。

[LanguageManual Joins](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Joins)  join的时候会将前面的表缓存，然后流式的取后面表的数据，因此最好把大表放在后面。如果在前面也可以，不过用streamtable关键字可以指定一个表不缓存。joins是左连的，就是从左向右执行，不可以替换。关于join也可以看这个中文的介绍 [Hive JOIN使用详解](http://shiyanjun.cn/archives/588.html) 

## Hive 参数优化

[Hive优化](http://itindex.net/detail/49495-hive-%E4%BC%98%E5%8C%96) 比较简单的一篇文章，讲到了关于数据倾斜的一些经验。

[Configuration Properties](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties) hive配置参数的官方介绍文档。

写ETL的时候一般会用到动态分区，所以一般会添加以下set语句：
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict;

其中HSQL会被解析为mapreduce的job，某些job之间没有直接依赖可以并行，如何让这些job并行？
set hive.exec.parallel=true;

如何定位数据倾斜？如何解决数据倾斜？
一般我碰到的数据倾斜都是类似 *distribute by dt* 这样的语句造成的，其实一般只处理一天的数据（dt=date），那么这个语句就会造成所有数据都集中在一个reduce里面，解决办法就是 distribute by 后面加上其他字段,比如 distribute by dt, ctime

Hive 小文件太多怎么办？
（1）       合并输入的小文件，输入文件太多，会启动太多map，影响效率。可通过设置参数来合并小文件：set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat
（2）      合并输出小文件，以减少输出文件的大小，可通过如下参数设置：
set hive.merge.mapfiles=true; // map only job结束时合并小文件
set hive.merge.mapredfiles=true; // 合并reduce输出的小文件
set hive.merge.smallfiles.avgsize=256000000; //当输出文件平均大小小于该值，启动新job合并文件
set hive.merge.size.per.task=64000000; //合并之后的每个文件大小64M

## Hive 原理

[Hive 工作原理](http://blog.csdn.net/jojo52013145/article/details/19206559) “hive就是一个将sql语句转化为MR工具”，类似编译器，有词法分析、语法分析、语义分析等阶段，最终生成一组MR任务。

[Hive SQL的编译过程](http://tech.meituan.com/hive-sql-to-mapreduce.html) 详细介绍了SQL转化为MapReduce的过程，比《[Hive 工作原理](http://blog.csdn.net/jojo52013145/article/details/19206559)》讲得更加清楚。关键词：antlr、ast

[[一起学Hive\]之十-Hive中Join的原理和机制](http://lxw1234.com/archives/2015/06/313.htm) todo，还没看

## 存储
[Hive：ORC存储格式详解](https://www.iteblog.com/archives/1014.html)

# 写ETL
## 如何命名
换种说法：**如何让人能够快速的从一大堆ETL中找到自己想要的那个ETL？**

这个问题可以拆分下来：
**如何做到看到名字就知道这个ETL大概在做什么？**
**如何快速的在一大堆ETL中筛选出我想要的（或者排除掉我不想要的）？**

答案是表意和命名规范。
例如：
AGGR_[事实表名]_[事实表筛选条件]__[聚合维度]__[聚合表筛选条件]__[多重聚合消失的维度]
aggr体现数据仓库分层思想，ODS层，基础表层，聚合层，主题层等
事实表名 可以用于追溯上游表
后面的几个部分 则是说明数据处理逻辑

当然从命名出发来解决找ETL的问题，是一种自上而下的顶层设计思维，它不可能面面俱到，也会不自由影响效率。特别是公司变大后，ETL数量膨胀，不同团队的命名规范不统一，事情就会变得混沌。这时候可以利用标签、血缘等手段搭建数据地图，提供导航搜索等功能。

## DDL
* 字段类型 数据仓库中的字段类型设置思想与业务不同，为了保证通用可扩展，一般最好用double、decimal、bigint 这样的类型。
* 注释 从可读性的角度，最好是给出准确详细的注释，特别是一些类型字段，给出类型值的含义
* 用什么样的存储类型比较好？ [Hive DDL 官方文档](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-RowFormat,StorageFormat,andSerDe)个人比较推荐ORC格式，[Hive:ORC File Format存储格式详解](http://www.iteblog.com/archives/1014)
* 按天分区字段  ETL的表能保存历史就保存历史，一般是天为单位

## 数据处理逻辑
一般ETL的处理逻辑都比较简单，筛选（where）、连接（join）、聚合（group by）。
如何确定筛选条件？包括业务指定的筛选条件以及异常值过滤。
如何确定异常值？ 这是数据质量检测的范畴，简单的就是看数据量，看数据分布。



## 数据质量检查
数据质量检查还是有套路的，比如数据量、数据分布、值域、数据之间的逻辑关系、上游标数据量等等。
如何自动化数据质量检测过程？
这个问题又可以拆解为：线下的自动化检查？线上的自动化检查？

线下的数据质量检查，使用场景一般是离线分析的ETL，或者ETL上线前的检查，ETL内容变动频繁，很难自动化，应该给予使用者各大自由度。我看到的思路有：
* 单表的数据画像
  * 数据量
  * 空值
  * 数值型字段：最大值、最小值、分位数、平均值、方差等
  * 字符型
    * 长度：最大长度、平均长度
    * 异常字符
* 跨表的数据对比
  * 与上游表对比
  * 与线上表对比
* 自定义sql

线上的数据监测，一种是手动配置阈值条件，一种是自动化识别。
手动配置阈值条件，就是在线下数据监测方法的基础上，人工设置阈值，就是普通监控报警系统的思路。
其实是可以基于历史的数据来预测出取值范围，然后自动识别异常数据的。未来的趋势应该是自动识别为主，手动配置为辅。
计算能力丰富后，能自动化的都会自动化，都是成本v.s.收益的博弈结果。