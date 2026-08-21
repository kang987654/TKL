# MapReduce의 셔플은 해시 분배 규칙으로 동작하며, Combiner는 로컬 압축으로 네트워크를 절약한다

## 🤔 헷갈렸던 점
- 셔플(Shuffle)이 단순히 Reducer로 가기 위해 `[1, 1]` 같은 리스트를 모으는 것인지, 어떤 Reducer로 가는지 규칙이 있는지 모호했다.
- 단어 빈도수 세기(WordCount) 같은 합산(Count/Sum) 작업에서 Combiner를 썼을 때 `(I, [1, 1])`이 아니라 `(I, 2)`처럼 나오는 것이 왜 모순이 아닌지 헷갈렸다.

## ✅ 정리
- **셔플(Shuffle)의 본질**:
  - 단순 리스트 생성이 아니라, **"수십 대 서버에 흩어진 수억 개의 `(Key, Value)`를 Key별로 묶어 네트워크 랜선을 통해 담당 Reducer로 배달하는 거대한 물리적 이동 과정"**이다.
- **셔플의 해시 분배 규칙 (Partitioner)**:
  - $\text{Reducer 번호} = \text{hash}(\text{Key}) \pmod{\text{총 Reducer 개수}}$
  - 모든 Mapper가 동일한 해시 공식을 쓰므로, 어떤 노드에서 나온 `"AI"` 단어라도 **무조건 동일한 번호의 Reducer로 배달**된다.
  - Reducer에 도착한 데이터는 `reduce()` 실행 전에 **Key 기준으로 자동 정렬(Sort)**된다.
- **Combiner의 로컬 선집계 원리 (단어 빈도수 합산/Count 기준)**:
  - Combiner는 덧셈(Sum/Count), 최댓값(Max)처럼 결합법칙이 성립하는 집계 연산에서 로컬 리듀서 역할을 한다.
  - **Combiner가 없을 때**: Mapper가 `(I, 1)`과 `(I, 1)`을 각각 네트워크로 전송 $\rightarrow$ Reducer에 `(I, [1, 1])` 도착.
  - **Combiner를 쓸 때**: Mapper가 자기 노드 안에서 먼저 1차 합산하여 **`(I, 2)` 딱 1개만 전송** $\rightarrow$ Reducer에는 각 Mapper의 부분합 `(I, [2, 3])`이 모여 최종 5를 완성!

### 💡 실사용 예시: Combiner를 통한 네트워크 절감
```java
// 단어 빈도수 합산(Count) Job에서 Combiner를 지정하면 네트워크 셔플 트래픽을 90% 이상 절약 가능
job.setMapperClass(TokenizerMapper.class);
job.setCombinerClass(IntSumReducer.class); // ◀ Mapper 노드 로컬에서 1차 합산(Count) 수행
job.setReducerClass(IntSumReducer.class);
```

셔플은 **해시 함수에 기반한 엄격한 네트워크 배달 시스템**이며, **Combiner는 Count/Sum 같은 집계 작업에서 로컬 선집계를 통해 네트워크 대역폭을 지켜내는 핵심 장치**이다.

---

**◀ 이전 글**: [08. Hadoop MapReduce에서 PySpark로의 진화](mapreduce-evolution-from-hadoop-java-to-pyspark.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [10. Mapper와 Reducer의 태스크 실행과 로컬 셔플 I/O](mapper-reducer-task-execution-and-local-shuffle.md)
