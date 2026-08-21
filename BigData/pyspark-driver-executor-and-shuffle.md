# PySpark의 Narrow 연산은 독립 실행이지만 Wide 연산은 물리적 네트워크 이동(Shuffle)을 유발한다

## 🤔 헷갈렸던 점
- Master, Driver, Executor의 역할 구분이 모호했고, 로컬 코드에서는 왜 Master가 잘 안 보이는지 의문이었다.
- Wide 연산의 '데이터 재분배'가 실제로 파티션 간/네트워크 상에서 데이터가 물리적으로 이동하는 것인지 헷갈렸다.

## ✅ 정리
- **역할의 명확한 분리**:
  - **Master (Cluster Manager)**: "인력소장" - 전체 서버 자원(CPU, RAM)을 관리하고 요청 시 Executor 프로세스를 띄워준 뒤 물러남.
  - **Driver Program**: "현장소장" - `SparkSession`이 동작하는 곳으로, 전체 연산 계획(DAG)을 세우고 Task를 배분.
  - **Executor**: "실제 일꾼" - Worker 노드에서 파티션 데이터를 메모리에 올려 Task를 수행.
  - *로컬 모드(`local[*]`)에서는 Master와 Driver가 1개의 프로세스 안에서 함께 동작하여 하나처럼 보인다.*
- **Narrow vs Wide 연산의 본질**:
  - **Narrow Dependency (`filter`, `select`)**:
    - 1개 파티션의 데이터만 보고 독립적으로 처리 가능 (데이터 이동 없음, 초고속).
  - **Wide Dependency (`groupBy`, `join`, `distinct`)**:
    - 동일한 키(Key)를 가진 데이터들을 하나의 노드로 모아야 하므로 **파티션 간 물리적 데이터 이동(Shuffle)**이 발생한다.
    - 데이터가 디스크에 임시 저장되고 네트워크 케이블을 타고 다른 노드로 전송되므로 분산 시스템 성능 저하의 주원인이 된다.

### 💡 실사용 예시: Shuffle 최소화를 위한 코드 작성
```py
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg

spark = SparkSession.builder.master("local[*]").getOrCreate()
df = spark.read.parquet("data.parquet")

# 1. Narrow 연산: 데이터 셔플 없이 각 노드에서 즉시 필터링 (가벼움)
filtered_df = df.filter(col("status") == "ACTIVE")

# 2. Wide 연산: 'category' 키별로 데이터를 네트워크 재분배(Shuffle)하여 집계 (무거움)
aggregated_df = filtered_df.groupBy("category").agg(avg("price"))
```

분산 처리 성능 최적화의 핵심은 **불필요한 Wide 연산(Shuffle)을 최소화하고, Shuffle 전에 필터링(Narrow)으로 데이터 크기를 먼저 줄이는 것**이다.

---

**◀ 이전 글**: [10. Mapper와 Reducer의 태스크 실행과 로컬 셔플 I/O](mapper-reducer-task-execution-and-local-shuffle.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [12. 2-Phase 분산 행렬 곱셈과 MapReduce 체이닝](distributed-matrix-multiplication-two-phase-mapreduce.md)
