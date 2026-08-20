# 분산 처리의 본질은 서버 개수가 아니라 데이터를 쪼개는 소프트웨어 아키텍처다

## 🤔 헷갈렸던 점
- 분산 처리를 하려면 반드시 여러 대의 물리적 서버나 고성능 GPU가 여러 개 있어야만 가능한 줄 알았다.
- 내 컴퓨터(노트북 1대) 환경에서는 분산 처리 실습이나 원리 구현이 불가능하다고 생각했다.
- 빅데이터 분산 처리에 GPU가 필수적인 것으로 오해했다.

## ✅ 정리
- **분산의 본질**: 물리적인 하드웨어 대수가 아니라, **"데이터와 작업을 독립된 조각(Task/Partition)으로 격리하여 병렬 처리하는 소프트웨어 아키텍처"**에 있다.
- **의사 분산(Pseudo-Distributed)**:
  - 1대의 PC에서도 CPU 멀티코어와 독립 프로세스(Process)들을 활용해 가상의 분산 노드(Cluster)를 구축할 수 있다.
  - 로컬 1대에서 도는 코드와 실제 100대 클러스터에서 도는 코드는 **100% 동일**하며, 차이는 오직 설정 파일의 IP 주소뿐이다.
- **CPU vs GPU의 역할**:
  - **빅데이터 분산 엔진(Hadoop, Spark, Kafka)**: 데이터 셔플, I/O, 필터링, 집계 등 **CPU 멀티코어와 RAM/네트워크 기반**으로 동작한다 (GPU 불필요).
  - **딥러닝(AI) 모델 학습**: 대규모 텐서 행렬곱 가속을 위해 **GPU**를 사용한다.

### 💡 실사용 예시: 로컬 CPU 코어를 가상 분산 노드로 활용하기
```py
from pyspark.sql import SparkSession

# local[*] : 내 컴퓨터의 모든 CPU 코어를 가상의 분산 워커(Executor)로 할당
spark = SparkSession.builder \
    .appName("LocalDistributedSimulation") \
    .master("local[*]") \
    .getOrCreate()

# 4개의 파티션(독립된 작업 단위)으로 데이터를 분산 분할
df = spark.range(0, 1000000).repartition(4)
print(f"할당된 분산 파티션 수: {df.rdd.getNumPartitions()}")
```

분산 처리는 하드웨어의 규모가 아니라 **데이터를 쪼개어 독립적으로 다루는 설계 방식**으로 이해하는 것이 더 정확하다.

---

**◀ 이전 글**: [01. Python의 단일 코어 동작과 GIL](python-default-single-core-and-gil.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [03. CAP 정리에서 CP와 AP의 선택](cap-theorem-tradeoff-between-cp-and-ap.md)
