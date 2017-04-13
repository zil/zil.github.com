---
layout: post
title: "docker下跑kafka读写千万条记录"
description: ""
category: ""
tags: kafka,mq,docker
---
{% include JB/setup %}

对Kafka耳闻已久,中间翻阅过文档，但一直没有上手跑个demo．  
在github上发现uber公开的2015年几个月的数据，最大文件的数据量在千万级别．  
跑demo测试kafka的机会来了.

### 闲话docker和docker-compose

docker管理容器游刃有余，但涉及到服务有多个容器，它们之间有依赖或需要扩展时，
需要执行多次命令，很费事．

docker-compose就是为解决容器之间依赖和容器扩展问题而被创建的．  
它可以让我们通过描述的方式确定容器间的依赖,并提供命令的方式扩展容器  
它可以让我们只关注服务，不必为容器启动的细节担心．

### 使用docker-compose启动kafka

在这个[docker-compose文件](https://github.com/wurstmeister/kafka-docker/blob/4a42b9e62c3c24c8e04a4cef8993e1713e2ee9b0/docker-compose.yml)基础上，修改了两个地方
{% highlight yml %}
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    build: .
    ports:
      - "9092:9092"
      - "1099:1099"
    environment:
      KAFKA_CREATE_TOPICS: "uber:4:1" # uber topic 有4个分区，１个复制
      HOSTNAME_COMMAND: "route -n | awk '/UG[ \t]/{print $$2}'" # kafka需要和主机通信
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

{% endhighlight %} 
在docker-compose文件所在目录可使用如下命令：  
`docker-compose up -d` 让docker在后台启动kafka和依赖的zookeeper  
`docker-compose logs -f kafka` 查看kafka启动日志  
`docker-compose down` 停止并销毁服务依赖的容器，  
　　再次`docker-compse up -d`时，kafka状态是全新的

### Producer客户端
程序中读取的文件的压缩包路径[uber-raw-data-janjune-15.csv.zip](https://github.com/fivethirtyeight/uber-tlc-foil-response/blob/63bb878b76f47f69b4527d50af57aac26dead983/uber-trip-data/uber-raw-data-janjune-15.csv.zip)
{% highlight java %}
package fun.play.kafka;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.LineNumberReader;
import java.util.List;
import java.util.Properties;
import java.util.concurrent.TimeUnit;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import com.google.common.base.CharMatcher;
import com.google.common.base.Charsets;
import com.google.common.base.Splitter;
import com.google.common.base.Stopwatch;

public class TheProducer 
{
    public static void main( String[] args ) throws IOException
    {
	CharMatcher doubleQuoteTrimmer = CharMatcher.is('"');
	//把每行记录按逗号分割后的值去除首尾的双引号
	Splitter splitter = Splitter.on(',').trimResults(doubleQuoteTrimmer);
	
	Stopwatch stopwatch = Stopwatch.createStarted();
	Producer<String, String> producer = newProducer();
	String line;
	try (InputStream in = TheProducer.class.getResourceAsStream("uber-raw-data-janjune-15.csv")){
	    LineNumberReader lineNumberReader;//行数太多，需流式读入，否则一次读入堆会溢出
	    lineNumberReader = new LineNumberReader(new InputStreamReader(in, Charsets.UTF_8));
			
	    while((line = lineNumberReader.readLine()) != null){
		List<String> columns = splitter.splitToList(line);
		if(columns.size() != 4) continue;
				
		String key = String.join("_", columns.subList(1, 3));
		int lineNumber = lineNumberReader.getLineNumber();
		int partition = lineNumber % 4;//记录依所在行数发送到不同的四个分区
		ProducerRecord<String, String> record =
		    new ProducerRecord<>("uber",partition,key,line);
		producer.send(record);
	    }
	    //处理的条数
	    System.err.println("line: "+lineNumberReader.getLineNumber());
	} catch (Exception e) {
	    e.printStackTrace();
	}

    	System.err.println("写入耗时（毫秒）:" + stopwatch.elapsed(TimeUnit.MILLISECONDS));
	producer.close(3,TimeUnit.SECONDS);
    }

    private static Producer<String, String> newProducer() {
	Properties props = new Properties();
	props.put("bootstrap.servers", "localhost:9092");
	props.put("acks", "all");
	props.put("retries", 0);
	props.put("batch.size", 16384);
	props.put("request.timeout.ms", 1000);
	props.put("max.block.ms", 1000);
	props.put("timeout.ms", 3000);
	props.put("linger.ms", 1);
	props.put("buffer.memory", 33554432);
	props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

	return new KafkaProducer<>(props);
    }
}

{% endhighlight %}
### 多个Consumer
{% highlight java %}
package fun.play.kafka;

import java.util.Arrays;
import java.util.Properties;
import java.util.Set;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.TopicPartition;

import com.google.common.base.Stopwatch;

public class Consumer {
    public static void main(String[] args) throws InterruptedException {
	ExecutorService es = Executors.newCachedThreadPool();
	Stopwatch stopwatch = Stopwatch.createStarted();
	CountDownLatch latch = new CountDownLatch(4);
	for (int i = 0; i < 4; i++) {//四个消费者
	    es.submit(new Runnable() {
		    @Override
		    public void run() {
			try {
			    createConsumerAndPoll();
			    latch.countDown();
			} catch (InterruptedException e) {
			    e.printStackTrace();
			}
		    }
		});
	}
	latch.await();
	System.err.println("elapsed:" + stopwatch.elapsed(TimeUnit.MILLISECONDS));
    }

    private static AtomicInteger consumedTotal = new AtomicInteger();
	
    private static KafkaConsumer<String, String> createConsumerAndPoll() throws InterruptedException {
	Properties cprops = new Properties();
	cprops.put("bootstrap.servers", "localhost:9092");
	cprops.put("group.id", "uber"); //组id
	cprops.put("enable.auto.commit", "true");
	cprops.put("auto.commit.interval.ms", "1000");
	cprops.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
	cprops.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
	KafkaConsumer<String, String> consumer = new KafkaConsumer<>(cprops);
	consumer.subscribe(Arrays.asList("uber"));//只有订阅方法，server才会自动给当前消费者分配分区
	while(true){
	    ConsumerRecords<String, String> records = consumer.poll(200);
	    if (!records.isEmpty()) {
		System.err.printf("%s,%d\n",Thread.currentThread(),records.count());
		consumedTotal.addAndGet(records.count());
	    }
	    if(consumedTotal.get() > 1000_0000)break;//所有消费者消费够１０００万条记录才停止
	}
	Set<TopicPartition> assignment = consumer.assignment();
	for (TopicPartition topicPartition : assignment) {
	    System.err.printf("%s.pos=%d\n",topicPartition,consumer.position(topicPartition));
	}
	return consumer;
    }
}

{% endhighlight %}

### 运行结果
单个kafka节点情况下:

单个Producer写入１４００万数据，耗时：51秒  
４个Consumer消费１０００万数据，耗时：27秒
