이 문서는 Java 95%, Scala 4%, Python 1% 로 사용하고 ETL 부터 Deeplearning에 이르는 기술스텍을 경험하다보니 스칼라나 파이썬의 표현식을 자주 잊는 상황때문에 만든 문서입니다.

# List
List는 immutable 형태만 있으며 mutable 형태 사용을 위해서는 다음과 같이 사용
```scala
import scala.collection.mutable  

val arrayBuffer = mutable.ArrayBuffer<String,Integer>()
```
