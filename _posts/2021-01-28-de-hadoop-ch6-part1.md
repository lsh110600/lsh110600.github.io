---
layout: post
title: '[Hadoop] Hadoop CH 6 정렬 part1:보조 정렬'
subtitle: 'Hadoop CH 6 정렬 part1:보조 정렬'
categories: de
tags: hadoop
comments: true
---
`시작하세요 하둡 프로그래밍`을 기반으로 공부한 내용을 정리합니다. - CH6 part1:부분 정렬

![img](/assets/img/post/hadoop/hadoop_book.png)

## 정렬

맵리듀스는 기본적으로 입력 데이터의 키를 기준으로 정렬되기 때문에 하나의 리듀스 테스크만 실행되게 한다면 쉽게 정렬할 수 있습니다.

하지만 분산 환경의 경우 여러 데이터 노드가 실행되기 때문에 하나의 리듀스 테스크만 실행하는 것은 바람직하지 않습니다.(네트워크 부하가 걸리는 등, 분산 환경의 장점을 살리지 못함)

따라서 다양한 정렬 방법을 사용할 줄 알아야합니다.

### 보조 정렬(Secondary Sort)

5장의 항공 운항 데이터를 보면, 다음과 같이 월의 순서가 제대로 처리돼 있지 않습니다.

```java
2008, 1
2008, 10
2008, 11
2008, 12
2008, 2
2008, 3
```

월의 순서대로 정렬하기 위해 보조 정렬을 사용합니다. 보조 정렬은 다음과 같은 단계로 실행됩니다.

1. 기존 키의 값들을 조합한 복합키(Composite Key)를 정의합니다.

2. 복합키 레코드를 정렬하기 위한 비교기(Comparator) 정의합니다.

3. 그룹핑 키를 파티셔닝하는 파티셔너(Partitioner)를 정의합니다.

4. 그룹핑 키를 정렬하기 위한 비교기(Comparator) 정의합니다.

#### 복합키(Composite Key) 구현

복합키는 기존의 키 값을 조합한 키 집합 클래스입니다.
이전의 5장에는 출력키를 하나의 문자열로 사용했지만, 복합키를 적용해 연 / 월이 각각 변수로 정의됩니다.

- 복합키를 사용하기 위해 WritableComparable 인터페이스를 구현하고, 파라미터는 자기 자신인 DateKey로 설정합니다.

    ```java
    Datekey.java

    public class DateKey implements WritableComparable<DateKey> {

    ...
    ```

WritableComparable 인터페이스의 메소드로 readFields, write, compareTo 메소드를 구현합니다.

- readFields 메소드는 입력 스트림에서 연 / 월을 읽어들입니다.

    ```java
    Datekey.java
    ...

    @Override
    public void readFields(DataInput in) throws IOException {
        year = WritableUtils.readString(in);
        month = in.readInt();
    }

    ...
    ```

- write 메소드는 출력 스트림에 연 / 월을 출력합니다.

    ```java
    Datekey.java
    ...

    @Override
    public void write(DataOutput out) throws IOException {
        WritableUtils.writeString(out, year);
        out.writeInt(month);
    }

    ...
    ```

- compareTo 메소드는 복합키끼리 순서를 비교할 때 사용하기 위한 메소드로, 미리 만들어 둡니다. 

    ```java
    Datekey.java
    ...

    @Override
    public int compareTo(DateKey key) {
        int result = year.compareTo(key.year);
        if (0 == result) {
        result = month.compareTo(key.month);
        }
        return result;
    }
    ...
    ```

- 전체 코드

<details markdown="1">
<summary>접기/펼치기</summary>

```java
Datekey.java
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableUtils;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class DateKey implements WritableComparable<DateKey> {

  private String year;
  private Integer month;

  public DateKey() {
  }

  public DateKey(String year, Integer date) {
    this.year = year;
    this.month = date;
  }

  public String getYear() {
    return year;
  }

  public void setYear(String year) {
    this.year = year;
  }

  public Integer getMonth() {
    return month;
  }

  public void setMonth(Integer month) {
    this.month = month;
  }

  @Override
  public String toString() {
    return (new StringBuilder()).append(year).append(",").append(month)
      .toString();
  }

  @Override
  public void readFields(DataInput in) throws IOException {
    year = WritableUtils.readString(in);
    month = in.readInt();
  }

  @Override
  public void write(DataOutput out) throws IOException {
    WritableUtils.writeString(out, year);
    out.writeInt(month);
  }

  @Override
  public int compareTo(DateKey key) {
    int result = year.compareTo(key.year);
    if (0 == result) {
      result = month.compareTo(key.month);
    }
    return result;
  }
}
```

</details>

#### 복합키 비교기(Comparator) 구현

앞서 만든 compareTo 메소드를 이용해 복합키끼리 순서를 비교하는 compare 메소드를 구현합니다.

- 비교 기준 1순위는 '연도', 2순위는 '월'로 비교하도록 합니다.

    ```java
    DatekeyComparator.java
    ...

    @Override
    public int compare(WritableComparable w1, WritableComparable w2) {
    DateKey k1 = (DateKey) w1;
    DateKey k2 = (DateKey) w2;

    int cmp = k1.getYear().compareTo(k2.getYear());
    if (cmp != 0) {
      return cmp;
    }

    ...
    ```

- 전체 코드

<details markdown="1">
<summary>접기/펼치기</summary>

```java
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

public class DateKeyComparator extends WritableComparator {
  protected DateKeyComparator() {
    super(DateKey.class, true);
  }

  @SuppressWarnings("rawtypes")
  @Override
  public int compare(WritableComparable w1, WritableComparable w2) {
    DateKey k1 = (DateKey) w1;
    DateKey k2 = (DateKey) w2;

    int cmp = k1.getYear().compareTo(k2.getYear());
    if (cmp != 0) {
      return cmp;
    }

    return k1.getMonth() == k2.getMonth() ? 0 : (k1.getMonth() < k2
      .getMonth() ? -1 : 1);
  }
}
```

#### 그룹 키 파티셔너(Partitioner)구현

파티셔너(Partitioner)는 맵 태스크의 출력 데이터를 리듀스 테스크의 입력 데이터로 보낼지 결정합니다.

파티셔닝된 데이터는 맵 태스크의 출력 데이터 키 값에 따라 정렬됩니다.

- 여기서는 연도를 그룹키로 사용하므로, getPartition 메서드는 연도에 대한 해시 코드를 조회해 파티션 번호를 생성합니다.

    ```java
    GroupKeyPartitioner.java
    ...

    @Override
    public int getPartition(DateKey key, IntWritable val, int numPartitions) {
        int hash = key.getYear().hashCode();
        int partition = hash % numPartitions;
        return partition;
    }

    ...
    ```

- 전체 코드

<details markdown="1">
<summary>접기/펼치기</summary>

```java
GroupKeyPartitioner.java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Partitioner;

public class GroupKeyPartitioner extends Partitioner<DateKey, IntWritable> {

  @Override
  public int getPartition(DateKey key, IntWritable val, int numPartitions) {
    int hash = key.getYear().hashCode();
    int partition = hash % numPartitions;
    return partition;
  }
}
```

#### 그룹 키 비교기(Comparator) 구현

위에서 그룹키로 정한 연도를 비교하는 단계입니다.

- `compare` 메서드는 앞의 복합키 비교기와 거의 유사하며, 연도만 비교하면 되기 때문에 월을 비교하는 부분은 빠져있습니다.

    ```java
    GroupKeyComparator.java
    ...

    @Override
    public int compare(WritableComparable w1, WritableComparable w2) {
        DateKey k1 = (DateKey) w1;
        DateKey k2 = (DateKey) w2;

        return k1.getYear().compareTo(k2.getYear());
    }

    ...
    ```

- 전체 코드

<details markdown="1">
<summary>접기/펼치기</summary>

```java
GroupKeyComparator.java
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

public class GroupKeyComparator extends WritableComparator {

  protected GroupKeyComparator() {
    super(DateKey.class, true);
  }

  @SuppressWarnings("rawtypes")
  @Override
  public int compare(WritableComparable w1, WritableComparable w2) {
    DateKey k1 = (DateKey) w1;
    DateKey k2 = (DateKey) w2;

    return k1.getYear().compareTo(k2.getYear());
  }
}
```

#### 매퍼 구현

복합키를 사용한 매퍼를 구현한 코드입니다.

- 매퍼 코드는 5장에서 작성한 코드와 거의 유사하나, 데이터 타입을 DateKey로 바꿔줍니다

    ```java
    DelayCountMapperWithDateKey.java
    ...

    public class DelayCountMapperWithDateKey extends Mapper<LongWritable, Text, DateKey, IntWritable>
    
    ...
    ```

- 출발 지연과 도착 지연을 구분하기 위해 연도 앞에 D(Departure)와 A(Arrive)를 붙여줍니다.

    ```java
    DelayCountMapperWithDateKey.java
    ...

    outputKey.setYear("D," + parser.getYear());
    outputKey.setMonth(parser.getMonth());
            
    outputKey.setYear("A," + parser.getYear());
    outputKey.setMonth(parser.getMonth());

    ...
    ```

- 전체 코드

<details markdown="1">
<summary>접기/펼치기</summary>

```java
DelayCountMapperWithDateKey.java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import wikibooks.hadoop.chapter05.DelayCounters;
import wikibooks.hadoop.common.AirlinePerformanceParser;

import java.io.IOException;

public class DelayCountMapperWithDateKey extends Mapper<LongWritable, Text, DateKey, IntWritable> {
  private final static IntWritable outputValue = new IntWritable(1);
  private DateKey outputKey = new DateKey();

  public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
    AirlinePerformanceParser parser = new AirlinePerformanceParser(value);

    if (parser.isDepartureDelayAvailable()) {
      if (parser.getDepartureDelayTime() > 0) {
        outputKey.setYear("D," + parser.getYear());
        outputKey.setMonth(parser.getMonth());

        context.write(outputKey, outputValue);
      } 
      else if (parser.getDepartureDelayTime() == 0) {
        context.getCounter(DelayCounters.scheduled_departure).increment(1);
      } 
      else if (parser.getDepartureDelayTime() < 0) {
        context.getCounter(DelayCounters.early_departure).increment(1);
      }
    } 
    else {
      context.getCounter(DelayCounters.not_available_departure).increment(1);
    }
    
    if (parser.isArriveDelayAvailable()) {
      if (parser.getArriveDelayTime() > 0) {
        outputKey.setYear("A," + parser.getYear());
        outputKey.setMonth(parser.getMonth());

        context.write(outputKey, outputValue);
      } 
      else if (parser.getArriveDelayTime() == 0) {
        context.getCounter(DelayCounters.scheduled_arrival).increment(1);
      } 
      else if (parser.getArriveDelayTime() < 0) {
        context.getCounter(DelayCounters.early_arrival).increment(1);
      }
    } 
    else {
      context.getCounter(DelayCounters.not_available_arrival).increment(1);
    }
  }
}
```

#### 리듀서 구현

리듀서는 매퍼의 출력 데이터를 받아 월별로 지연 횟수를 합산합니다.

- 리듀서 코드도 5장에서 작성한 코드와 거의 유사하며, 데이터 타입을 DateKey로 바꿔줍니다

    ```java
    DelayCountReducerWithDateKey.java
    ...

    public class DelayCountReducerWithDateKey extends Reducer<DateKey, IntWritable, DateKey, IntWritable>

    ...
    ```

- 월별로 지연 횟수를 합산해야하므로 Iterable 객체를 이용해 순회합니다.

- 이때 백업된 월과 현재 데이터의 월이 일치하지 않으면 백업된 지연 횟수를 출력하고 합산 변수를 0으로 초기화합니다.

    ```java
    DelayCountReducerWithDateKey.java
    ...

    for (IntWritable value : values) {
        if (bMonth != key.getMonth()) {
          result.set(sum);
          outputKey.setYear(key.getYear().substring(2));
          outputKey.setMonth(bMonth);
          mos.write("arrival", outputKey, result);
          sum = 0;
        }
        sum += value.get();
        bMonth = key.getMonth();
    }

    ...
    ```

- Iterable 객체가 모두 순회되고 월 데이터가 일치하면 합산 값을 출력합니다.

    ```java
    DelayCountReducerWithDateKey.java
    ...

    if (key.getMonth() == bMonth) {
        outputKey.setYear(key.getYear().substring(2));
        outputKey.setMonth(key.getMonth());
        result.set(sum);
        mos.write("arrival", outputKey, result);
    }

    ...
    ```

- 전체 코드

<details markdown="1">
<summary>접기/펼치기</summary>

```java
DelayCountMapperWithDateKey.java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.output.MultipleOutputs;

import java.io.IOException;

public class DelayCountReducerWithDateKey extends
  Reducer<DateKey, IntWritable, DateKey, IntWritable> {

  private MultipleOutputs<DateKey, IntWritable> mos;
  private DateKey outputKey = new DateKey();

  private IntWritable result = new IntWritable();

  @Override
  public void setup(Context context) throws IOException, InterruptedException {
    mos = new MultipleOutputs<DateKey, IntWritable>(context);
  }

  public void reduce(DateKey key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
    String[] colums = key.getYear().split(",");

    int sum = 0;
    Integer bMonth = key.getMonth();

    if (colums[0].equals("D")) {
      for (IntWritable value : values) {
        if (bMonth != key.getMonth()) {
          result.set(sum);
          outputKey.setYear(key.getYear().substring(2));
          outputKey.setMonth(bMonth);
          mos.write("departure", outputKey, result);
          sum = 0;
        }
        sum += value.get();
        bMonth = key.getMonth();
      }
      if (key.getMonth() == bMonth) {
        outputKey.setYear(key.getYear().substring(2));
        outputKey.setMonth(key.getMonth());
        result.set(sum);
        mos.write("departure", outputKey, result);
      }
    } 
    else {
      for (IntWritable value : values) {
        if (bMonth != key.getMonth()) {
          result.set(sum);
          outputKey.setYear(key.getYear().substring(2));
          outputKey.setMonth(bMonth);
          mos.write("arrival", outputKey, result);
          sum = 0;
        }
        sum += value.get();
        bMonth = key.getMonth();
      }
      if (key.getMonth() == bMonth) {
        outputKey.setYear(key.getYear().substring(2));
        outputKey.setMonth(key.getMonth());
        result.set(sum);
        mos.write("arrival", outputKey, result);
      }
    }
  }

  @Override
  public void cleanup(Context context) throws IOException,
    InterruptedException {
    mos.close();
  }
}
```

#### 드라이버 구현

앞서 구현한 클래스들을 구동하는 드라이버 클래스를 구현합니다.

- 그룹키 파티셔너, 그룹키 비교기, 복합키 비교기를 잡에 등록합니다.

    ```java
    DelayCountWithDateKey.java
    ...

    job.setPartitionerClass(GroupKeyPartitioner.class);
    job.setGroupingComparatorClass(GroupKeyComparator.class);
    job.setSortComparatorClass(DateKeyComparator.class);

    ...
    ```

- 출력 데이터 포맷에 복합키와 지연 횟수를 설정합니다.

    ```java
    DelayCountWithDateKey.java
    ...

    job.setMapOutputKeyClass(DateKey.class);
    job.setMapOutputValueClass(IntWritable.class);

    job.setOutputKeyClass(DateKey.class);
    job.setOutputValueClass(IntWritable.class);

    ...
    ```

- 전체 코드

<details markdown="1">
<summary>접기/펼치기</summary>

```java
DelayCountWithDateKey.java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.MultipleOutputs;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class DelayCountWithDateKey extends Configured implements Tool {

  public static void main(String[] args) throws Exception {
    int res = ToolRunner.run(new Configuration(), new DelayCountWithDateKey(), args);
    System.out.println("MR-Job Result:" + res);
  }

  public int run(String[] args) throws Exception {
    String[] otherArgs = new GenericOptionsParser(getConf(), args).getRemainingArgs();

    if (otherArgs.length != 2) {
      System.err.println("Usage: DelayCountWithDateKey <in> <out>");
      System.exit(2);
    }
    
    Job job = new Job(getConf(), "DelayCountWithDateKey");

    FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
    FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));

    job.setJarByClass(DelayCountWithDateKey.class);
    job.setPartitionerClass(GroupKeyPartitioner.class);
    job.setGroupingComparatorClass(GroupKeyComparator.class);
    job.setSortComparatorClass(DateKeyComparator.class);

    job.setMapperClass(DelayCountMapperWithDateKey.class);
    job.setReducerClass(DelayCountReducerWithDateKey.class);

    job.setMapOutputKeyClass(DateKey.class);
    job.setMapOutputValueClass(IntWritable.class);

    job.setInputFormatClass(TextInputFormat.class);
    job.setOutputFormatClass(TextOutputFormat.class);

    job.setOutputKeyClass(DateKey.class);
    job.setOutputValueClass(IntWritable.class);

    MultipleOutputs.addNamedOutput(job, "departure",
      TextOutputFormat.class, DateKey.class, IntWritable.class);
    MultipleOutputs.addNamedOutput(job, "arrival", TextOutputFormat.class,
      DateKey.class, IntWritable.class);

    job.waitForCompletion(true);
    return 0;
  }
}
```

### Reference

1. 시작하세요 하둡 프로그래밍
2. [어초 어초님 tistory 블로그](https://eochodevlog.tistory.com/39?category=920528)
