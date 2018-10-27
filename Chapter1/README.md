# MapReduce简介

- 单机模式下的mapper函数：将数据从文件中采集并成给映射

  - hadoop规定了自己可用的网络序列化优化的基本类型：
    - LongWritable 相当于Java中的Long类型
    - Text相当于Java中的String类型
    - IntWritable 相当于Java中的Integer类型
    - EXAMPLE：**public void map(LongWritable key, Text value, OuputCollector <Text, IntWritable\> output, Reporter reporter)**
  - 提供了OutputCollector实例来写入输出内容

- 单机模式下的reduce函数：

  - 注意reduce函数的输入类型要与map函数的输出类型匹配
  - 本例是遍历所有气温，直到找到最高的为止

- 运行MapReduce作业

```java
JobConf conf = new JobConf(MaxTemperature.class);
conf.setJobName("Max temperature");
FileInputFormat.addInputPath(conf, new Path(args[0]));
FileOutputFormat.setOutputPath(conf,new Path(args[1]));
conf.setMapperClass(MaTemperatureMapper.class);
conf.setReducerClass(MaxTemperatureReducer.class);
conf.setOutputKeyClass(Text.class);
conf.setOutputValueClass(IntWritable.class);
JobClient.runJob(conf);
```

- 注意在在Hadoop集群中使用的时候需要打包成Jar文件
- 在创建JobConf对象后，将指定输入和输出的路径；指定要使用的map和reduce类型；下面两个则是控制map和reduce函数的输出类型；输入类型通过输入格式来控制；

- 新版Hadoop Mapreduce API

- map函数

```java
private static final int MISSING = 9999;
public void map(LongWritable Key, Text value, Context context)
throws IOException, InterruptedException {
String line = value.toString();
String year = line.substring(15,19);
int airTemperature;
if(line.charAt(87) == '+') {
airTemperature = Integer.parserInt(line.substring(88,92));
} else{
airTemperature = Integer.parseInt(line.substrng(87,92));
}
String quality = line.substring(92,93);
if (airTemperature != MISSING && quality.matches(\*[01459]\*)){
context.write(new Text(year), new IntWritable(airTemperature));
}
}
```

- reduce函数：同样需要将原来的OutputController换为context.write函数
- MAPREDUCE配置：

```java
Job job = new Job();
job.setJarByClass(NewMaxTemperature.class);
FileInputFormat.addInputPath(job,new Path(args[0]))
FileOutputFormat.setOutputPath(conf,new Path(args[1]));
job.setOutputKeyClass(Text.class);
job.setOutputValueClass(IntWritable.class);
System.exit(job.waitForCompletion(true)? 0:1)
```

### 分布化

- 为了分布化，需要将数据存储在分布式文件系统中，如HDFS
- 术语说明： 
- MapReduce中作业(job)是客户端执行的单位，包括输入数据，MapReduce程序和配置信息
- Hadoop通过把作业分成若干个小任务(task)来工作，包括map和reduce任务
- 两种类型的结点控制作业执行过程，1个jobtracker和多个tacktracker。jobtracker调度任务在tasktracker上运行；jobtracker在运行任务时，进度报告，有任务失败，jobtracker重新调度
- 分类情况：单一reduce函数；多个reduce函数；不需要reduce函数(用combiner接口)
- **支持多种语言：python，C++，Ruby等**

