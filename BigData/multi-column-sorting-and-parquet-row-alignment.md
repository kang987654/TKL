# Parquet은 인덱스로 행을 맞추며, 다차원 검색은 파티셔닝과 Z-Order로 해결한다

## 🤔 헷갈렸던 점
- Parquet이 컬럼별로 분리 저장된다면, 서로 다른 컬럼의 데이터가 몇 번째 행(Row)에 속하는지 어떻게 일치시키는지 의문이었다.
- 날짜, 지역, 나이처럼 서로 독립적인 복합 컬럼들은 단순 정렬(`ORDER BY`)을 적용해도 첫 번째 컬럼 외에는 정렬이 흩어지는데 어떻게 최적화하는지 궁금했다.
- Parquet 파일이 바이너리 포맷이면 사람이 CSV처럼 열어볼 수 없는지 헷갈렸다.

## ✅ 정리
- **행(Row)의 일치성 원리**:
  - Parquet은 10만 행 단위의 **Row Group** 내부에서 모든 컬럼의 데이터 순서(Index/Offset)를 1:1로 정확히 일치시킨다.
  - Col A의 $i$번째 원소와 Col B의 $i$번째 원소를 합치면 별도의 키 없이도 완벽히 $i$번째 행이 복원된다.
- **다차원 데이터의 검색 최적화 기법**:
  1. **계층적 분할 (디렉토리 파티셔닝 + 파일 내 정렬)**:
     - 가장 빈번히 검색하는 `날짜`는 폴더 구조로 분할 (`/year=2026/month=08/`) $\rightarrow$ 폴더 단위 스킵(Partition Pruning).
     - 파일 내부 데이터는 `지역`이나 `유저ID`로 정렬 $\rightarrow$ Row Group 단위 스킵.
  2. **Z-Order (공간 채움 곡선)**:
     - 다차원 컬럼의 비트를 지그재그(Z자)로 엮어 1차원으로 매핑함으로써, 여러 컬럼 중 어떤 것으로 검색해도 70~80%의 블록을 건너뛸 수 있게 한다.
- **사람의 가독성 (Human Readability)**:
  - 메모장으로 열면 바이너리가 깨져 보이지만, **VS Code의 "Parquet Viewer" 확장 프로그램이나 `pd.read_parquet()`**을 사용하면 CSV와 똑같이 표/스프레드시트 형태로 쉽게 조회할 수 있다.

### 💡 실사용 예시: 디렉토리 파티셔닝 저장
```py
# 가장 자주 조회되는 컬럼을 기준으로 디렉토리를 분할하여 저장
df.write.mode("overwrite") \
    .partitionBy("year", "category") \
    .parquet("partitioned_data")
```

Parquet은 **내부 인덱스 정합성**을 통해 컬럼형 저장을 유지하며, **파티셔닝과 Z-Order**를 통해 다차원 데이터의 디스크 스캔 낭비를 막는다.

---

**◀ 이전 글**: [05. Parquet의 Predicate Pushdown 원리](parquet-columnar-storage-and-predicate-pushdown.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [07. Hadoop 생태계: HDFS 저장과 MapReduce 연산의 분리](hadoop-ecosystem-hdfs-storage-vs-mapreduce-compute.md)
