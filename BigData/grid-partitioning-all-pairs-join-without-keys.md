# 조인 키가 없는 전수 비교는 2차원 격자 분할(All-Pairs)로 노드를 배치하여 병렬화한다

## 🤔 헷갈렸던 점
- SQL 조인은 항상 `ON A.user_id = B.user_id`처럼 딱 떨어지는 키가 있었는데, 키가 없는 조인은 도대체 언제 왜 쓰는지 이해되지 않았다.
- 키가 없는 전수 비교 작업을 어떻게 여러 노드로 골고루 분배하는지(AllPairPartition) 메커니즘이 모호했다.

## ✅ 정리
- **키가 없는 조인(Non-Equi Join)의 실제 활용 사례**:
  1. **위치 기반 매칭**: 배달 라이더 GPS($R$)와 주문 식당 GPS($S$) 간의 **거리 수식 계산 (`distance <= 1km`)**. (서로 좌표값이 100% 일치할 수 없음)
  2. **AI 추천 & 벡터 검색**: 유저 취향 임베딩($R$)과 상품 벡터($S$) 간의 **코사인 유사도 계산 (`CosineSim >= 0.9`)**.
  3. **문서 중복/표절 검사**: 두 텍스트의 글자 일치율 / 자카드 유사도 비교.
- **2차원 격자 분할(AllPairPartition)의 미팅방 배정 원리**:
  - $N \times M$ 전수 비교를 위해 노드들을 $P \times P$ (예: $2 \times 2=4$개) 미팅방으로 배치한다.
  - **$R$ 데이터는 자신이 속한 '가로 줄(Row)'의 모든 방으로 복제** 전송.
  - **$S$ 데이터는 자신이 속한 '세로 줄(Column)'의 모든 방으로 복제** 전송.
  - $\rightarrow$ 4개의 방에서 동시에 대화(비교)가 진행되어 **속도가 4배 빨라지며, 모든 $(r, s)$ 쌍이 빠짐없이 딱 1번씩 만나게 된다.**

### 💡 핵심 비교: 키가 있는 조인 vs 키가 없는 조인
```sql
-- 1. 키가 있는 일반 조인 (RPJoin: 해시 우체통 배달)
SELECT * FROM Users U JOIN Orders O ON U.user_id = O.user_id;

-- 2. 키가 없는 조인 (AllPairPartition: 2차원 격자 분할 전수 비교)
SELECT * FROM Riders R JOIN Restaurants S ON DISTANCE(R.gps, S.gps) <= 1.0;
```

키가 없는 조인은 **거리/유사도 같은 수식으로 가까운 짝을 찾기 위한 필수 기법**이며, **격자 분할은 전수 비교를 완벽히 균등하게 병렬화하는 분산 해법**이다.

---

**◀ 이전 글**: [12. 2-Phase 분산 행렬 곱셈과 MapReduce 체이닝](distributed-matrix-multiplication-two-phase-mapreduce.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [14. 오프라인 K-Means 배치와 온라인 1-NN 초고속 서빙](offline-kmeans-clustering-and-online-nearest-centroid-serving.md)
