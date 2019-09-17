# Kairosdb 之 KairosDB Client 日常使用总结
##基本概念
首先介绍一下Kairosdb的几个概念

Metric:  度量，相当于关系型数据库中的 table。

 * A metric contains measurements or data points. Each data point has a time stamp of when the measurement occurred
 * and a value that is either a long or double and optionally contains tags. Tags are labels that can be added to better
 * identify the metric. For example, if the measurement was done on server1 then you might add a tag named "host"
 * with a value of "server1". Note that a metric must have at least one tag.

Tag:  标签。一般存放的是不随时间戳变化的信息。timestamp 加上所有的 tags 可以视为 table 的 primary key。

Data point:  数据点，相当于关系型数据库中的 row。
* A measurement. Contains the time when the measurement occurred and its value.

Timestamp：时间戳，代表数据点产生的时间。


