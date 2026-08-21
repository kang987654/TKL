# Top-K 분산 검색은 Reducer가 아니라 Mapper의 cleanup()과 우선순위 큐로 로컬 선별해야 네트워크 폭발을 막는다

## 🤔 헷갈렸던 점
- MapReduce에서 정렬과 Top-K 집계는 무조건 Reducer에서만 가능한 줄 알았는데, Mapper 안에서 로컬 Top-K를 뽑는다는 것이 어떻게 가능한지 의문이었다.

## ✅ 정리
- **Reducer에게만 맡겼을 때의 대참사**:
  - 100개 노드가 10억 개의 데이터를 필터링 없이 그대로 네트워크로 쏘면, **네트워크 대역폭이 마비되고 1개의 Reducer가 메모리 폭발(OOM)로 다운**된다.
- **Mapper 안에서 로컬 Top-K를 뽑는 비밀 (`cleanup()`과 Heap)**:
  1. **`setup()`**: Mapper가 시작할 때 **크기 $K$(예: 10)짜리 우선순위 큐(Min-Heap)**를 메모리에 생성.
  2. **`map()`**: 수천만 줄을 읽으면서 큐를 갱신. 10개가 넘어가면 **꼴찌를 즉시 버림** (네트워크 전송 전혀 안 함!).
  3. **`cleanup()`**: 파일 전체를 다 읽고 Mapper가 끝나기 직전에, **살아남은 최종 10개만 Reducer로 딱 방출**.
- **Reducer의 초고속 병합 (글로벌 Top-K)**:
  - 100개 Mapper에서 올라온 후보 **단 1,000개($100 \times 10$)만 받아서** 최종 글로벌 1등~10등을 추려냄 (네트워크 대역폭 99.999% 절약).

### 💡 실사용 예시: Mapper 내부의 cleanup() 활용 구조
```java
public static class TopKMapper extends Mapper<Object, Text, Text, DoubleWritable> {
    private PriorityQueue<Item> localQueue = new PriorityQueue<>(10);

    public void map(Object key, Text value, Context context) {
        localQueue.add(new Item(value, calculateScore(value)));
        if (localQueue.size() > 10) localQueue.poll(); // 꼴찌는 즉시 폐기
    }

    public void cleanup(Context context) {
        for (Item item : localQueue) context.write(item.text, item.score); // 최종 10개만 전송
    }
}
```

Top-K 분산 검색의 핵심은 **Mapper의 `cleanup()`과 인메모리 힙을 활용하여 네트워크 전송량을 10억 개에서 1,000개로 압축하는 것**이다.

---

**◀ 이전 글**: [14. 오프라인 K-Means 배치와 온라인 1-NN 초고속 서빙](offline-kmeans-clustering-and-online-nearest-centroid-serving.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [16. Airflow DAG 하네스와 Docker](airflow-dag-harness-and-docker.md)
