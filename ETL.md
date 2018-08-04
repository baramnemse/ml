# Managing Spark Partitions with Coalesce and Repartition

https://hackernoon.com/managing-spark-partitions-with-coalesce-and-repartition-4050c57ad5c4

다수의 파티션을 사용하는 규모 있는 원본 데이터(10억 rows)에서 일부(2000 rows)를 취해도 원본데이터의 파티션 갯수(10000)가 유지된다.

DATAFRAME.rdd.partitions.size 로 파티션 사이즈 알 수 있음

적은수의 데이터를 다수의 파티션에 담는 구조이므로 이 데이터를 활용하게 되면 대부분 빈 파티션을 억세스하게되므로 비효율

작아진 규모 만큼 repartion필요

비싼 연산인 repartion을 coalesce 대신 사용하는지는 이해안감, 추측은 다수의 파티션에 있는 작은수의 데이터를 repartion하는 비용과 coalesce 비용이 차이가 없어서 그런것으로 추측

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
cache() 캐싱
```
aggDf.cache()
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
