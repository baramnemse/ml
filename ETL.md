# Lazy Evaluation in Apache Spark
Spark는 트랜스포매이션과 엑션으로 연산이 나뉨, 엑션은 실제 답을 구하는 count와 save등의 연산을 뜻함. 스파크는 엑션 실행 전까지 실제 연산을 지연시키고 DAG(Directed Acyclic Graph)로 관리하다 엑션이 수행될때 모든 연산이 실제로 실행됨. 이를 통해 최적화를 수행할 수 있음

https://data-flair.training/blogs/apache-spark-lazy-evaluation/

http://spark.apache.org/docs/latest/rdd-programming-guide.html#transformations

http://spark.apache.org/docs/latest/rdd-programming-guide.html#actions

# Job, Task

![](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/diagrams/job-stage.png)

# DAG vs RDD Lineage

- RDD Lineage logical execution plan

우리가 짠 코드에 해당함, 결과물을 얻기 위한 스텝

- DAG physical execution plan

분석을 어떻게 나누고 실행할지 결정하는 스케줄러

https://www.quora.com/What-is-the-difference-between-RDD-Lineage-Graph-and-Directed-Acyclic-Graph-DAG-in-Spark

# Cluster Resources
노드마다 Yarn/Hadoop이 사용할 수 있는 1Core와 메모리1GB가 필요
```
--executor-cores/spark.executor.cores => executor의 core 개수
--executor-memory/spark.executor.memory => executor의 memory 용량
--num-executor/spark.executor.instances => executor 개수
```
분산환경에서 사용할 수 있는 우버의 자바 프로파일러

https://eng.uber.com/jvm-profiler/
# Spark Memory
메모리 문제는 다음 영역중 하나의 문제, 특히 RDD와 Shuffle메모리가 부족할 경우 디스크 IO를 사용하므로 성능저하가 일어남
- RDD 60% spark.storage.memoryFraction
- Shuffle 20% spark.shuffle.memoryFraction
- Application 20%

# Distribution of Executors, Cores and Memory

https://spoddutur.github.io/spark-notes/distribution_of_executors_cores_and_memory_for_spark_application.html
```shell
spark-submit --class <CLASS_NAME> --num-executors ? --executor-cores ? --executor-memory ?
```
# Broadcasting Join
큰데이터와 작은 데이터를 조인할떄 작은 데이터를 브로드케스팅 배리어블로 지정하고 조인하면 셔플을 획기적으로 줄일수 있음

# Salting
Skewed Data : 특정키가 매우 큰 데이터

매출 데이터에서 특정 점포의 데이터가 전체의 80% 차지하는 경우 다른 executor는 빠르게 처리되지만 다수의 키를 가지고있는 데이터를 처리하는 executor가 병목이됨

이를 처리하기 위해서는 키에 추가 문자를 더해서 키를 분산시키면됨, Key1 Key2 Key3 Key4

# Managing Spark Partitions with Coalesce and Repartition

https://hackernoon.com/managing-spark-partitions-with-coalesce-and-repartition-4050c57ad5c4

다수의 파티션을 사용하는 규모 있는 원본 데이터(10억 rows)에서 일부(2000 rows)를 취해도 원본데이터의 파티션 갯수(10000)가 유지된다.

DATAFRAME.rdd.partitions.size 로 파티션 사이즈 알 수 있음

적은수의 데이터를 다수의 파티션에 담는 구조이므로 이 데이터를 활용하게 되면 대부분 빈 파티션을 억세스하게되므로 비효율

작아진 규모 만큼 repartion필요

비싼 연산인 repartion을 coalesce 대신 사용하는지는 이해안감, 추측은 다수의 파티션에 있는 작은수의 데이터를 repartion하는 비용과 coalesce 비용이 차이가 없어서 그런것으로 추측

# persist(MEMORY_ONLY_SER)
하나의 직렬객체로 캐싱하면서 연산비용이 드나 하나의 객체로 합쳐지므로 GC 비용을 대폭 감소할수 있음

# Data Error Handling
포맷은 맞으나 타입이 안맞을 경우 드랍
```scala
spark.read
.option("mode","PERMISSIVE")
```
포맷이 안맞을 경우 
```scala
spark.read
.option("mode","MALFORMED")
```
# CSV
```scala
val df = spark.read.format("csv").option("header", "true").load("c:/2008.csv")
```
# groupBy VS reduceBy
reduceBy의 셔플량이 작아서 메모리 사용과 속도가 빨라짐

https://databricks.gitbooks.io/databricks-spark-knowledge-base/content/best_practices/prefer_reducebykey_over_groupbykey.html

# Dataset
```scala
case class GameRecord(Year:Integer, City:String, Sport:String, Discipline:String, Athlete:String, Country:String, Gender:String, Event:String, Medal:String)
val ds = spark.read.format("csv").option("inferSchema", "true").option("header", "true").load("c:/summer.csv").as[GameRecord]
```
# Dataframe
debugCodegen 어떻게 쿼리가 실행되는지 자바코드로 볼 수 있다.
```scala
import org.apache.spark.sql.execution.debug._
etlM.select("Sport").except(etlF.select("Sport")).debugCodegen

Generated code:
/* 001 */ public Object generate(Object[] references) {
/* 002 */   return new GeneratedIterator(references);
/* 003 */ }
/* 004 */
/* 005 */ final class GeneratedIterator extends org.apache.spark.sql.execution.BufferedRowIterator {
```
createGlobalTempView(viewName) 다른 스파크 어플리케이션에서도 억세스 가능, 해당 뷰를 만든 세션이 종료되면 삭제됨
```scala
// Register the DataFrame as a global temporary view
df.createGlobalTempView("people")

// Global temporary view is tied to a system preserved database `global_temp`
spark.sql("SELECT * FROM global_temp.people").show()
```
registerTempTable(tableName) 테이블로 메모리에 저장, 다른 노드에서 접근 불가능

saveAsTable()테이블로 디스크에 저장, 모든 노드에서 접근 가능
```scala
bank.registerTempTable("bank")
spark.sql("SELECT * FROM bank")
```
explain(flag) 처리과정을 설명한다
```
var s30=bank.filter($"age" >= 30 && $"age" <=39).groupBy("marital").avg("balance").sort(desc("avg(balance)"))
s30.explain(true)

s30: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [marital: string, avg(balance): double]
== Parsed Logical Plan ==
'Sort ['avg(balance) DESC], true
+- Aggregate [marital#545], [marital#545, avg(cast(balance#547 as bigint)) AS avg(balance)#791]
   +- Filter ((age#543 >= 30) && (age#543 <= 39))
      +- LogicalRDD [age#543, job#544, marital#545, education#546, balance#547]

== Analyzed Logical Plan ==
marital: string, avg(balance): double
Sort [avg(balance)#791 DESC], true
+- Aggregate [marital#545], [marital#545, avg(cast(balance#547 as bigint)) AS avg(balance)#791]
   +- Filter ((age#543 >= 30) && (age#543 <= 39))
      +- LogicalRDD [age#543, job#544, marital#545, education#546, balance#547]

== Optimized Logical Plan ==
Sort [avg(balance)#791 DESC], true
+- Aggregate [marital#545], [marital#545, avg(cast(balance#547 as bigint)) AS avg(balance)#791]
   +- Project [marital#545, balance#547]
      +- Filter ((isnotnull(age#543) && (age#543 >= 30)) && (age#543 <= 39))
         +- LogicalRDD [age#543, job#544, marital#545, education#546, balance#547]

== Physical Plan ==
*Sort [avg(balance)#791 DESC], true, 0
+- Exchange rangepartitioning(avg(balance)#791 DESC, 200)
   +- *HashAggregate(keys=[marital#545], functions=[avg(cast(balance#547 as bigint))], output=[marital#545, avg(balance)#791])
      +- Exchange hashpartitioning(marital#545, 200)
         +- *HashAggregate(keys=[marital#545], functions=[partial_avg(cast(balance#547 as bigint))], output=[marital#545, sum#799, count#800L])
            +- *Project [marital#545, balance#547]
               +- *Filter ((isnotnull(age#543) && (age#543 >= 30)) && (age#543 <= 39))
                  +- Scan ExistingRDD[age#543,job#544,marital#545,education#546,balance#547]
```
map 예외처리
```scala
def plus(num:Integer):Integer = {
    if(num>3)throw new Exception
    num+1
}
List(1,2,3,4).map(x =>{
    try { 
    plus(x)
  } catch {
    case e : Exception =>
    // Log error
    None
  }
})
```
flatmap 인풋을 단일 컬렉션으로 바꿈
```scala
val x = sc.parallelize(List("spark rdd example",  "sample example"), 2)
 
// map operation will return Array of Arrays in following case : check type of res0
val y = x.map(x => x.split(" ")) // split(" ") returns an array of words
y.collect
// res0: Array[Array[String]] = 
//  Array(Array(spark, rdd, example), Array(sample, example))
 
// flatMap operation will return Array of words in following case : Check type of res1
val y = x.flatMap(x => x.split(" "))
y.collect
//res1: Array[String] = 
//  Array(spark, rdd, example, sample, example)
```
예제에 사용한 데이터 셋 http://stat-computing.org/dataexpo/2009/the-data.html

withColumn(java.lang.String colName, Column col) 특정 컬럼을 추가하거나 바꾼다
```
var newDf = df.withColumn("Year",df.col("Year").cast("int")).withColumn("ArrDelay",df.col("ArrDelay").cast("int"))
```
```
root
 |-- Year: integer (nullable = true)
```
groupBy(Column... cols) 집게연산을 할수 있게 컬럼을 묶는다.
```
var aggDf = newDf.groupBy("FlightNum").agg(sum("ArrDelay")) // 총 지연시간
var aggDf = newDf.groupBy("FlightNum").agg(avg("ArrDelay")) // 평균 지연시간
var aggDf = newDf.groupBy("FlightNum").count.show() //비행횟수
var aggDf = newDf.groupBy("FlightNum").agg(max("ArrDelay")) // 최대 
```
persist() 캐싱
```
aggDf.persist(MEMORY_ONLY_SER)
aggDf.unpersist()
```
orderBy 정렬
```
var orderDf = aggDf.orderBy("AVG(ArrDelay)").show() //오름차순
var orderDf = aggDf.sort($"AVG(ArrDelay)".desc).show() //내림차순
var aggDf = newDf.groupBy("FlightNum").count.sort(desc("count")).show() //비행횟수가 많은순으로 
```
distinct 유니크한 Row만 리턴
```
newDf.select("FlightNum").distinct.count //운항하는 항공기의 댓수
```
filter 
```
newDf.groupBy("FlightNum").count.filter($"count">4000).show() //운항횟수가 4천회 이상인 항공기 댓수
```
# Spark SQL
초기화
```scala
import org.apache.spark.sql.SparkSession
val spark = SparkSession.builder().appName("Spark SQL basic example").config("spark.some.config.option", "some-value").getOrCreate()
``` 
HAVING과 ORDER BY를 동시에 쓸수 없음
```sql
SELECT count(value) FROM t GROUP BY key HAVING max(value) > 10;
SELECT count(value) FROM t GROUP BY key HAVING key > 10;

SELECT count(value) FROM t GROUP BY key ORDER BY max(value);

SELECT count(value) FROM t GROUP BY key ORDER BY key;

But these don't:

SELECT count(value) FROM t GROUP BY key HAVING max(value) > 10 ORDER BY max(value);
SELECT count(value) FROM t GROUP BY key HAVING max(value) > 10 ORDER BY key;
```
다음과 같이 SQL과 Dataframe 메소드로는 섞어서 쓸수 있음
```
spark.sql("SELECT marital, AVG(balance) FROM bank WHERE age >=30 AND age <=39 GROUP BY marital HAVING avg(balance) > 30").sort(desc("avg(balance)")).show
```

# 스파크 인터뷰 질문, 개념이해용으로 
https://tekslate.com/spark-interview-questions/

# RDD (Resilient Distributed Datasets)
스파크의 기본 데이터 추상화, read-only, fault-tolerance node fail시 rdd lineage를 통해 데이터 복구

# Spark Flow
1. Driver 에서 어플리케이션 실행
2. 스파크 세션 생성
3. 메니저에서 자원 활당
4. executor 생성
5. transformation 연산으로 부터 DAG 생성
6. action 연산을 만나면 실제로 각 executor 에 일을 분산하여 보냄
7. 최종 결과를 다시 driver 에 전달
8. 자원 반환

# Catalyst

https://databricks.com/session/a-deep-dive-into-spark-sqls-catalyst-optimizer

# Accumerator vs Broadcast Variable
노드간 증가하는 데이터를 공유할때, 노드에서 읽을수는 없음, 비정상 데이터 카운트, driver에서만 읽을수 있음
```scala
var accum = spark.sparkContext.longAccumulator("counter")
accum.add(3)
```
노드간 변하지 않는 데이터를 공유할때, 수정 불가, 원본 데이터에 없는 코드 공유시, 변경되더라도 전파안됨
```scala
scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar: org.apache.spark.broadcast.Broadcast[Array[Int]] = Broadcast(0)

scala> broadcastVar.value
res0: Array[Int] = Array(1, 2, 3)
```
# DStreams
- stateless transformations 이전 배치와 상관 없는 형태
- Stateful transformations 이전 배치의 데이터나 결과를 사용하는 형태

# Realtime Map/Reduce vs Event Driven Microservice

https://www.youtube.com/watch?v=I32hmY4diFY

https://mapr.com/blog/event-driven-microservices-patterns/

https://mapr.com/products/polyglot-persistence/

# mapPartitions, mapPartitionsWithIndex
파티션별로 한번의 초기화, 각 파티션별로 한번의 초기화를 통해 http나 kafka 를 이용 실시간 모니터링 가능
```scala
val mapped =   rdd1.mapPartitionsWithIndex{
                        // 'index' represents the Partition No
                        // 'iterator' to iterate through all elements
                        //                         in the partition
                        (index, iterator) => {
                           println("Called in Partition -> " + index)
                           //초기화 구문
                           val myList = iterator.toList
                           myList.map(x => x + " -> " + index).iterator
                        }
                     }
 ```

# RDD, Dataframe, Dataset

RDD JavaObject GC 위험

Dataframe, Dataset off heap 사용 

![](https://databricks.com/wp-content/uploads/2016/07/sql-vs-dataframes-vs-datasets-type-safety-spectrum.png)

# Large Cluster
https://www.youtube.com/watch?v=5dga0UT4RI8

rpc server

spark.rpc.io.serverThread = 64

Executor Tuning

spark.memory.offHeap.enable = true

Disk IO Tuning
spark.shuffle.file.buffer =1MB
spark.unsafe.sorter.spill.reader.buffer.size=1MB

# Monitoring
SparkLint

Spark History Server

# Spark Optimizing Join

https://www.youtube.com/watch?v=fp53QhSfQcI

# treeReduce vs reduce, treeAggregate vs aggregate

reduce는 연산결과를 driver로 보내기 때문에 밴드위스가 보틀넥이 될수 있음 treeReduce는 driver가 아닌 executor에게 보내므로 병목이 줄어듬
