# Managing Spark Partitions with Coalesce and Repartition

https://hackernoon.com/managing-spark-partitions-with-coalesce-and-repartition-4050c57ad5c4

다순의 파티션을 사용하는 규모 있는 원본 데이터(10억 rows)에서 일부(2000 rows)를 취해도 원본데이터의 파티션 갯수(10000)가 유지된다.

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

# Dataframe
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
var aggDf = newDf.groupBy("FlightNum").agg(sum("ArrDelay"))
var aggDf = newDf.groupBy("FlightNum").agg(avg("ArrDelay"))
```
cache() 캐싱
```
aggDf.cache()
```
orderBy 정렬
```
var orderDf = aggDf.orderBy("AVG(ArrDelay)").show() //오름차순
var orderDf = aggDf.sort($"AVG(ArrDelay)".desc).show() //
```
