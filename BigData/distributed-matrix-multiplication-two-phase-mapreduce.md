# 분산 행렬 곱셈이 2-Phase인 이유는 1개의 MapReduce Job은 오직 1번의 셔플만 가능하기 때문이다

## 🤔 헷갈렸던 점
- 행렬 곱셈 수학 자체는 알고 있었지만, MapReduce라는 프레임워크 관점에서 왜 굳이 2단계(2-Phase)로 나누어 실행해야 하는지 와닿지 않았다.

## ✅ 정리
- **MapReduce의 구조적 제약**:
  - 1개의 MapReduce Job은 **오직 1개의 기준 Key로만 셔플(그룹화)**을 수행할 수 있다.
  - 행렬 곱셈($C(i, j) = \sum A(i, k) \times B(k, j)$)은 **곱셈 기준($k$)**과 **합산 기준($(i, j)$)**이 서로 다르다.
  - 따라서 **Job 1(곱셈)의 출력을 HDFS 디스크에 저장한 뒤, Job 2(합산)가 이를 읽어서 연달아 실행(Job Chaining)**해야 한다.
- **2-Phase의 구체적 실행 흐름**:
  1. **Phase 1 (k 기준 셔플 & 곱셈)**:
     - Mapper: 공통 인덱스 $k$를 Key로 방출 $\rightarrow$ `Key: k, Value: (A, i, val)` / `Key: k, Value: (B, j, val)`
     - Reducer: $k$별로 모인 $A$와 $B$ 원소를 곱하여 임시 방출 $\rightarrow$ `Key: (i, j), Value: A(i,k)*B(k,j)`
     - 결과: HDFS 임시 폴더(`/temp_phase1`)에 저장.
  2. **Phase 2 ((i, j) 기준 셔플 & 합산)**:
     - Mapper: 임시 파일의 좌표 `(i, j)`를 그대로 Key로 방출.
     - Reducer: 같은 `(i, j)`에 모인 곱셈 결과들을 단순 합산($+$)하여 최종 $C(i, j)$ 완성.

### 💡 핵심 비교: 왜 Spark는 행렬 연산에서 하둡보다 100배 빠른가?
* **Hadoop**: Phase 1 결과를 **HDFS 하드디스크에 썼다가(Write)** Phase 2에서 **다시 디스크에서 읽음(Read)** $\rightarrow$ 디스크 I/O 극심.
* **Spark**: Phase 1 결과를 **워커 노드 RAM에 그대로 올려둔 채** Phase 2를 즉시 실행 $\rightarrow$ 메모리 초고속 처리.

분산 행렬 곱셈의 2-Phase 분할은 **셔플 기준 키의 전환($k \rightarrow (i, j)$)을 위한 설계**이며, **현대 거대 모델(LLM) 텐서 병렬화의 근본 뼈대**이다.

---

**◀ 이전 글**: [11. PySpark의 Driver/Executor와 Shuffle](pyspark-driver-executor-and-shuffle.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [13. 키 없는 조인의 활용과 2차원 격자 분할](grid-partitioning-all-pairs-join-without-keys.md)
