# PySpark의 DataFrame 한 줄은 과거 Hadoop MapReduce 100줄의 Java 코드를 내부에서 자동 생성한 것이다

## 🤔 헷갈렸던 점
- 1세대 Hadoop MapReduce와 현대의 PySpark가 어떻게 연결되는지, 왜 PySpark 코드에서는 `map()`과 `reduce()` 함수를 직접 짜지 않는지 의문이었다.
- Java 진영에는 PySpark 같은 현대적인 분산 도구가 없는지 궁금했다.

## ✅ 정리
- **Hadoop MapReduce(Java) $\rightarrow$ PySpark의 진화**:
  - 과거 Java MapReduce에서는 개발자가 `TokenizerMapper`와 `IntSumReducer` 클래스를 100줄에 걸쳐 직접 작성해야 했다.
  - 현대의 PySpark에서는 `df.groupBy("word").count()` 한 줄만 쓰면, **Spark의 Catalyst 최적화 엔진이 내부적으로 Map, Shuffle, Reduce 태스크를 알아서 조립하여 분산 실행**한다.
- **Java 생태계의 Spark**:
  - Spark 엔진 자체가 원래 Scala/Java로 개발되었기 때문에 Java에서도 PySpark와 동일하게 `SparkSession`, `Dataset<Row>` API를 지원한다.
  - 다만 AI/머신러닝(PyTorch, Hugging Face) 생태계와의 결합 편의성 때문에 데이터/AI 분야에서는 **PySpark(Python)**가 압도적으로 많이 활용된다.

### 💡 실사용 예시: Java MapReduce 100줄 vs PySpark 1줄
```py
# 과거 Java로 100줄을 짜야 했던 WordCount가 PySpark에서는 단 한 줄로 끝남
# 내부적으로는 동일하게 Map -> Shuffle -> Reduce 과정이 완벽히 수행됨
word_counts = text_df.groupBy("word").count()
```

현대의 분산 처리 프레임워크는 **과거의 복잡한 MapReduce 원리를 엔진 내부에 추상화하여, 개발자가 SQL/DataFrame 수준에서 직관적으로 대용량 연산을 다룰 수 있게 해 준 것**이다.

---

**◀ 이전 글**: [07. Hadoop 생태계: HDFS 저장과 MapReduce 연산의 분리](hadoop-ecosystem-hdfs-storage-vs-mapreduce-compute.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [09. MapReduce 셔플의 해시 분배 규칙과 Combiner](mapreduce-shuffle-mechanism-and-combiner.md)
