# 🌌 Big Data & Distributed AI Architecture

빅데이터 파이프라인의 **데이터 유입 ──▶ 분산 저장 ──▶ 연산 및 분산 알고리즘 ──▶ AI 서빙 및 오케스트레이션** 흐름 순서대로 정리한 학습 노트입니다.  
각 문서는 단일 주제에 대한 통찰과 오개념 교정 과정을 담고 있습니다.

---

## 🗺️ 파이프라인 아키텍처 흐름도

```
[0. 기초 철학]    Python GIL 한계 ──▶ 분산 처리(Scale-Out)의 본질 ──▶ CAP 정리 (CP vs AP)
                          │
                          ▼
[1. 수집 & 경로]   데이터 유입 ──▶ 대용량 배치(ELT) vs 실시간 스트리밍(ETL) 선택
                          │
                          ▼
[2. 분산 저장]    디스크 I/O 최적화 ──▶ Parquet (Predicate Pushdown & Z-Order) ──▶ Hadoop HDFS(저장소)
                          │
                          ▼
[3. 분산 연산]    MapReduce 진화 ──▶ 셔플 & Combiner ──▶ 로컬 셔플/태스크 실행 ──▶ PySpark ──▶ 2-Phase 행렬곱 ──▶ 격자 분할 조인
                          │
                          ▼
[4. AI & 서빙]    오프라인 K-Means 배치 + 온라인 1-NN 서빙 ──▶ In-Mapper Top-K 검색 최적화
                          │
                          ▼
[5. 통합 관리]    파이프라인 지휘 ──▶ Airflow (DAG 하네스) & Docker 인프라 격리
```

---

## 📚 목차

| 순서 | 단계 | 문서 제목 & 링크 | 핵심 키워드 |
| :---: | :--- | :--- | :--- |
| **01** | **기초 배경** | [01. Python의 단일 코어 동작과 GIL](python-default-single-core-and-gil.md) | `CPython`, `GIL`, `Single Core` |
| **02** | **분산 원리** | [02. 분산 처리의 본질과 단일 노드 의사 분산](scale-out-and-pseudo-distributed-on-single-node.md) | `Scale-Out`, `Pseudo-Distributed`, `CPU vs GPU` |
| **03** | **시스템 제약**| [03. CAP 정리에서 CP와 AP의 선택](cap-theorem-tradeoff-between-cp-and-ap.md) | `CAP Theorem`, `Consistency`, `Availability` |
| **04** | **수집 & 흐름**| [04. 대용량 배치 ELT vs 실시간 스트리밍 ETL](batch-elt-vs-streaming-etl.md) | `Medallion`, `Bronze/Silver/Gold`, `In-Flight` |
| **05** | **저장 최적화**| [05. Parquet의 Predicate Pushdown 원리](parquet-columnar-storage-and-predicate-pushdown.md) | `Columnar`, `Footer Metadata`, `I/O Skip` |
| **06** | **저장 심화**  | [06. Parquet 행 정합성과 다차원 정렬/Z-Order](multi-column-sorting-and-parquet-row-alignment.md) | `Row Group Index`, `Partitioning`, `Z-Order` |
| **07** | **생태계 저장**| [07. Hadoop 생태계: HDFS 저장과 MapReduce 연산의 분리](hadoop-ecosystem-hdfs-storage-vs-mapreduce-compute.md) | `Apache Foundation`, `HDFS`, `Decoupling` |
| **08** | **연산 원리**  | [08. Hadoop MapReduce에서 PySpark로의 진화](mapreduce-evolution-from-hadoop-java-to-pyspark.md) | `Map-Shuffle-Reduce`, `Catalyst Optimizer` |
| **09** | **셔플 메커니즘**| [09. MapReduce 셔플의 해시 분배 규칙과 Combiner](mapreduce-shuffle-mechanism-and-combiner.md) | `Hash Partitioner`, `Sort`, `Combiner` |
| **10** | **실행 메커니즘**| [10. Mapper와 Reducer의 태스크 실행과 로컬 셔플 I/O](mapper-reducer-task-execution-and-local-shuffle.md) | `Task Lifecycle`, `Phase Separation`, `Local Disk I/O` |
| **11** | **연산 최적화**| [11. PySpark의 Driver/Executor와 Shuffle](pyspark-driver-executor-and-shuffle.md) | `Driver/Executor`, `Narrow/Wide`, `Shuffle` |
| **12** | **분산 행렬곱**| [12. 2-Phase 분산 행렬 곱셈과 MapReduce 체이닝](distributed-matrix-multiplication-two-phase-mapreduce.md) | `2-Phase`, `Job Chaining`, `Tensor Parallelism` |
| **13** | **격자 분할 조인**| [13. 키 없는 조인의 활용과 2차원 격자 분할](grid-partitioning-all-pairs-join-without-keys.md) | `Non-Equi Join`, `Grid Partitioning`, `All-Pairs` |
| **14** | **AI 배치/서빙**| [14. 오프라인 K-Means 배치와 온라인 1-NN 초고속 서빙](offline-kmeans-clustering-and-online-nearest-centroid-serving.md) | `Offline Clustering`, `1-NN Serving`, `O(K) Latency` |
| **15** | **분산 Top-K** | [15. In-Mapper Top-K와 cleanup() 우선순위 큐](in-mapper-topk-search-optimization-and-cleanup.md) | `In-Mapper Combining`, `cleanup()`, `Min-Heap` |
| **16** | **오케스트레이션**| [16. Airflow DAG 하네스와 Docker](airflow-dag-harness-and-docker.md) | `DAG Harness`, `Retry`, `Docker Container` |
