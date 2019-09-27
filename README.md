# Kairosdb 之 KairosDB Client 日常使用总结
基于org.kairosdb:client:2.2.0

基本概念

Metric:  度量，相当于关系型数据库中的 table。

 A metric contains measurements or data points. Each data point has a time stamp of when the measurement occurred
 and a value that is either a long or double and optionally contains tags. Tags are labels that can be added to better
 identify the metric. For example, if the measurement was done on server1 then you might add a tag named "host"
 with a value of "server1". Note that a metric must have at least one tag.

Tag:  标签。一般存放的是不随时间戳变化的信息。timestamp 加上所有的 tags 可以视为 table 的 primary key。

Data point:  数据点，相当于关系型数据库中的 row。
A measurement. Contains the time when the measurement occurred and its value.

Timestamp：时间戳，代表数据点产生的时间。

关系：
Metric包含多个Tag，Tag包含多个DataPoint，DataPoint对应一个Timestamp和一个value。
同一个timestamp可以对应两个value，一个为long类型，一个为double类型。

常用类
MetricBuilder
Builder used to create the JSON to push metrics to KairosDB.

QueryBuilder
 Builder used to create the JSON to query KairosDB.
 <br>
 <br>
 The query returns the data points for the given metrics for the specified time range. The time range can
 be specified as absolute or relative. Absolute times are a given point in time. Relative times are relative to now.
 The end time is not required and defaults to now.
 <br>
 <br>
 For example, if you specify a relative start time of 30 minutes, all matching data points for the last 30 minutes
 will be returned. If you specify a relative start time of 30 minutes and a relative end time of 10 minutes, then
 all matching data points that occurred between the last 30 minutes up to and including the last 10 minutes are returned.
 查询必须有开始时间
 开始时间和结束时间是闭区间，就是查询的数据包含开始时间点和结束时间点。
 [开始时间,结束时间]


AggregatorFactory
聚合工厂，里面包含很多聚合方法

方法

Sending Metrics
Sending metrics is done by using the MetricBuilder. You simply add a metric, the tags associated with the metric, and the data points.
```
try(HttpClient client = new HttpClient("http://localhost:8080"))
{
	MetricBuilder builder = MetricBuilder.getInstance();
	builder.addMetric("metric1")
			.addTag("host", "server1")
			.addTag("customer", "Acme")
			.addDataPoint(System.currentTimeMillis(), 10)
			.addDataPoint(System.currentTimeMillis(), 30L);
	client.pushMetrics(builder);
}
```
说明：metric至少有一个tag

1,
`metricBuilder.addMetric("metric1").addTag("a","a").addDataPoint(System.currentTimeMillis(),1);`

会插入一个datapoint,
可通过以下方式查询：

`queryBuilder.setStart(1, TimeUnit.DAYS);`

`queryBuilder.addMetric("metric1").addTag("a","a");`
`queryBuilder.addMetric("metric1");`

2，

`metricBuilder.addMetric("metric1").addTag("a","a").addTag("b","b").addDataPoint(System.currentTimeMillis(),1);`

会插入一个datapoint,
可通过以下方式查询：

```
queryBuilder.setStart(1, TimeUnit.DAYS);

queryBuilder.addMetric("metric1").addTag("a","a").addTag("b","b");
queryBuilder.addMetric("metric1").addTag("a","a");
queryBuilder.addMetric("metric1").addTag("b","b");
queryBuilder.addMetric("metric1");
```

基本的插入数据分为两种类型，long，double

```
Object a=3;
metricBuilder.addMetric("metric1").addTag("c","c").addDataPoint(System.currentTimeMillis(),1);
metricBuilder.addMetric("metric1").addTag("c","c").addDataPoint(System.currentTimeMillis(),"2");
metricBuilder.addMetric("metric1").addTag("c","c").addDataPoint(System.currentTimeMillis(),a);
```

这三种都是插入long类型的，如果同时执行上面三句，最后插入的value为3，不是按先后顺序，是取最大值插入的。（感觉莫名其妙，但是多次尝试确实这样，很无奈）

```
Object a=3.0;
metricBuilder.addMetric("metric1").addTag("c","c").addDataPoint(System.currentTimeMillis(),1.0);
metricBuilder.addMetric("metric1").addTag("c","c").addDataPoint(System.currentTimeMillis(),"2.0");
metricBuilder.addMetric("metric1").addTag("c","c").addDataPoint(System.currentTimeMillis(),a);
```

这三种都是插入double类型的，如果同时执行上面三句，最后插入的value为3.0，不是按先后顺序，是取最大值插入的。（感觉莫名其妙，但是多次尝试确实这样，很无奈）


同一个时间点上如果同时存在long和double两种类型，查询的时候都会查询出来，且参与聚合。

```
metricBuilder.addMetric("metric1").addTag("c","c").addDataPoint(long,1);
metricBuilder.addMetric("metric1").addTag("c","c").addDataPoint(long,2.0);
```

`queryBuilder.addMetric("metric1").addAggregator(AggregatorFactory.createSumAggregator(1,TimeUnit.DAYS));`

这种聚合调用时，1和2.0都参与，结果会是3.0
**So 一个项目中；最好不要同时用long和double两种类型，要不然同一个点会出现两个数据，会很麻烦的。页面查询的时候不能区分，不过，通过代码查询出来的是能看到value的类型。…………最好统一一种形式，都用double，这样如果值错了也是可以直接用double替换的（删除的效率不高，数据量大了特别明显，尽量别用）。**


聚合器Aggregator 这块儿是个大头儿，里面细节挺多的。

![alt](1569568668860.jpg title)






























