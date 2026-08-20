# Parquet의 Predicate Pushdown은 메타데이터(Footer)를 읽어 SSD 블록 자체를 건너뛰는 기술이다

## 🤔 헷갈렸던 점
- 저장장치(SSD) 자체는 지능이 없어서 값을 비교하지 못하는데, 어떻게 "데이터 블록을 건너뛴다"는 것인지 하드웨어적 흐름이 이해되지 않았다.
- 데이터 범위가 조건에 딱 떨어지지 않고 걸쳐 있을 때는 어떻게 처리되는지 의문이었다.
- AI 전처리에서 DataFrame을 NumPy로 변환해 쓰듯, Parquet도 NumPy와 같은 메모리 객체인 줄 알았다.

## ✅ 정리
- **하드웨어 데이터 흐름**:
  - `SSD` ──(PCIe & DMA)──▶ `RAM` ──▶ `CPU 캐시/레지스터`
  - DMA 컨트롤러가 SSD의 특정 바이트 블록을 RAM으로 직접 복사한 뒤, CPU가 데이터를 읽는다.
- **Predicate Pushdown의 실제 메커니즘**:
  1. Parquet 파일 맨 끝에 있는 **몇 KB짜리 메타데이터(Footer)**를 먼저 RAM/CPU로 읽는다.
  2. Footer에 기록된 각 Row Group(데이터 블록)의 **최솟값(Min)/최댓값(Max)**을 확인한다.
  3. 조건(예: `age > 30`)에 해당 값이 없는 블록은 **SSD에서 RAM으로 읽어오는 I/O 명령 자체를 생략(Skip)**한다.
- **범위가 걸쳐 있을 때 (Overlap)**:
  - 블록 범위가 `[10, 45]`라면 건너뛸 수 없으므로 RAM으로 올려 CPU가 행 단위로 필터링한다. (따라서 **자주 검색하는 컬럼으로 사전 정렬(Sort)하여 저장**하는 것이 핵심 최적화다.)
- **Parquet vs NumPy**:
  - **Parquet**: 디스크(SSD/S3)에 바이너리로 압축 저장되는 **초고효율 파일 포맷**.
  - **NumPy**: 메모리(RAM)에 상주하며 AI 모델이 연산하는 **행렬 객체**.

### 💡 실사용 예시: Parquet 저장 및 자동 메타데이터 생성
```py
# PySpark에서 Parquet 저장 시 Footer(Min/Max 메타데이터)가 100% 자동 생성됨
df.write.mode("overwrite").parquet("user_data.parquet")

# 쿼리 시 Spark가 Footer를 보고 불필요한 디스크 블록을 자동으로 스킵
filtered_df = spark.read.parquet("user_data.parquet").filter("age > 30")
```

Parquet은 디스크 I/O 병목을 해결하기 위해 **파일 레벨의 통계 메타데이터를 활용하여 하드웨어 읽기 작업 자체를 원천 차단하는 저장 기술**이다.

---

**◀ 이전 글**: [04. 대용량 배치 ELT vs 실시간 스트리밍 ETL](batch-elt-vs-streaming-etl.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [06. Parquet 행 정합성과 다차원 정렬/Z-Order](multi-column-sorting-and-parquet-row-alignment.md)
