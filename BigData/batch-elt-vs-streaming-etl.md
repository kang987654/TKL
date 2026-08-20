# 대용량 배치는 데이터 보존을 위해 ELT를 쓰고, 실시간 스트리밍은 즉시 판단을 위해 Streaming ETL을 쓴다

## 🤔 헷갈렸던 점
- 수집/저장 단계에서 미리 가공해서 저장해두면 나중에 분산 연산(Spark)할 때 부담이 적을 텐데, 왜 굳이 Raw 데이터를 그대로 저장한 뒤 Spark로 가공하는(ELT)지 의문이었다.
- 반대로 실시간 스트리밍에서는 왜 저장하기 전에 Spark UDF로 AI 추론을 먼저 돌리는지(ETL처럼 동작하는지) 헷갈렸다.

## ✅ 정리
- **배치 파이프라인에서 ELT(메달리온 아키텍처)를 쓰는 이유**:
  1. **수집 병목 방지**: 수집기(Kafka/적재기)에 복잡한 가공을 넣으면 수집이 지연되어 데이터가 유실된다.
  2. **원천 데이터(Raw) 보존**: AI 피처는 계속 바뀌므로, 원본을 버리지 않고 보존해야 미래 모델 재학습이 가능하다.
  3. **전역 집계/조인의 필요성**: 수집 시점(1건씩 유입)에는 "전체 유저 평균"이나 대규모 테이블 조인이 불가능하다.
  - **메달리온 아키텍처**: **Bronze(날것 저장) ──▶ Silver(Parquet 정제) ──▶ Gold(최종 피처)** 단계로 부하를 분산한다.
- **실시간 스트리밍에서 'In-Flight AI 추론(Streaming ETL)'을 쓰는 이유**:
  - 도난 카드 결제나 금융 사기(FDS)는 **0.1초 만에 차단**해야 하므로, 디스크 저장을 기다릴 시간이 없다.
  - 데이터가 메모리를 흘러가는 그 찰나에 Spark Streaming이 AI 모델을 돌려 즉시 판단하고, 결과를 알림/저장한다.

### 💡 핵심 비교: 배치 ELT vs 실시간 스트리밍
```
[배치 ELT]     데이터 ──> Raw 저장소(Bronze) ──> [새벽 일괄 Spark 가공] ──> Feature(Gold)
[실시간 ETL]   데이터 ──> Kafka ──> [메모리 통과 시 실시간 Spark AI 추론] ──> 즉시 차단 / 알림
```

대용량 데이터의 안정적 보존에는 **ELT(Data Lake)**가 적합하고, 초저지연 즉시 대응에는 **실시간 Stream Processing(Streaming ETL)**이 적합하다.

---

**◀ 이전 글**: [03. CAP 정리에서 CP와 AP의 선택](cap-theorem-tradeoff-between-cp-and-ap.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [05. Parquet의 Predicate Pushdown 원리](parquet-columnar-storage-and-predicate-pushdown.md)
