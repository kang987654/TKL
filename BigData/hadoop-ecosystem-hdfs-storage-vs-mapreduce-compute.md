# Hadoop은 단일 프로그램이 아니라 분산 저장(HDFS)과 분산 연산(MapReduce)이 분리된 모듈형 생태계다

## 🤔 헷갈렸던 점
- 하둡(Hadoop)을 하나의 거대한 단일 프로그램으로 생각해서 HDFS와 MapReduce의 관계가 모호했다.
- 아파치(Apache)가 영리 소프트웨어 회사인 줄 알았고, 왜 저장과 연산이 분리되어 개발되었는지 의문이었다.

## ✅ 정리
- **아파치 소프트웨어 재단(ASF)**:
  - 영리 회사가 아니라, 오픈소스 기술들을 공공재로 보존하고 관리하는 **세계 최대의 비영리 재단**이다.
- **하둡의 3대 독립 컴포넌트 (관심사의 분리)**:
  1. **HDFS (저장 계층)**: 128MB 블록 단위로 3중 복제하여 하드디스크 장애 시에도 데이터를 지켜내는 **분산 파일 시스템(금고)**.
  2. **MapReduce (연산 계층)**: HDFS에 저장된 데이터를 수천 개 CPU 코어로 쪼개어 계산하는 **분산 연산 엔진(계산기)**.
  3. **YARN (자원 관리)**: 클러스터의 CPU와 RAM 자원을 조율하는 **매니저**.
- **저장과 연산 분리의 결정적 이점 (교체 가능성)**:
  - 저장(HDFS)과 연산(MapReduce)이 분리되어 있었기 때문에, 훗날 느려터진 MapReduce를 버리고 **저장소(HDFS/S3)는 그대로 둔 채 연산 엔진만 Spark로 갈아 끼우는 혁신**이 가능했다.

### 💡 핵심 비교: 과거와 현대 아키텍처
```
[과거 아키텍처]  [저장] HDFS              ──▶ [연산] Hadoop MapReduce (디스크 I/O 느림)
[현대 아키텍처]  [저장] HDFS / S3 / MinIO ──▶ [연산] Apache Spark / Ray (인메모리 100배 고속)
```

하둡은 단순한 프로그램이 아니라 **"안전한 분산 하드디스크(HDFS)"와 "교체 가능한 계산 엔진(MapReduce/Spark)"을 모듈화하여 설계한 분산 생태계**이다.

---

**◀ 이전 글**: [06. Parquet 행 정합성과 다차원 정렬/Z-Order](multi-column-sorting-and-parquet-row-alignment.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [08. Hadoop MapReduce에서 PySpark로의 진화](mapreduce-evolution-from-hadoop-java-to-pyspark.md)
