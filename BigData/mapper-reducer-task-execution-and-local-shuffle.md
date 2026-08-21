# Mapper와 Reducer는 고정된 하드웨어가 아닌 순차 실행 태스크이며, 로컬 셔플은 디스크와 IPC(Inter-Process Communication)로 이동한다

## 🤔 헷갈렸던 점
- 셔플(Shuffle) 시 네트워크 랜선을 통한 이동은 다중 서버 환경일 때인데, 단일 CPU/로컬 PC 환경에서는 데이터가 물리적으로 어떻게 이동하는지 의문이었다.
- CPU 1번이 Mapper 전용, CPU 2번이 Reducer 전용처럼 특정 하드웨어 코어가 고정 배정되는 구조인지, 아니면 단순 함수 단위인지 모호했다.

## ✅ 정리
- **단일 PC(로컬)에서 셔플 데이터의 이동 경로**:
  - 물리 랜선 대신 **로컬 디스크(SSD/HDD) 임시 파일 I/O와 OS 메모리 버스(IPC / localhost 루프백)**를 통해 이동한다.
  - Mapper 프로세스가 로컬 임시 디렉토리(`/tmp/hadoop/...`)에 파티션별 파일을 쓰면, Reducer 프로세스가 해당 로컬 파일을 직접 읽어온다.
- **Mapper와 Reducer의 실체 (Task & Phase 분리)**:
  - **코드 관점**: 개발자가 정의한 `map()` 함수와 `reduce()` 함수.
  - **실행(OS) 관점**: 특정 CPU 코어가 고정되는 것이 아니라, **동일한 CPU 코어들이 시간 순서에 따라 역할을 바꾸어 수행하는 독립 작업(Task/프로세스)**이다.
    - **1단계 (Map Phase)**: CPU 코어 1~4가 모두 **Mapper Task**를 맡아 데이터 조각을 읽고 `map()` 실행 $\rightarrow$ 로컬 디스크에 임시 저장.
    - **2단계 (Reduce Phase)**: Map 단계가 모두 끝나면, **똑같은 CPU 코어 1~4가 이번에는 Reducer Task로 전환**되어 디스크에서 데이터를 읽어 `reduce()` 실행.

### 💡 핵심 비교: 시간 순서에 따른 CPU 코어의 Task 할당
```
[시간 T1: Map 단계]    Core 1 (Mapper 1) | Core 2 (Mapper 2) | Core 3 (Mapper 3) | Core 4 (Mapper 4)
                                    │ (로컬 디스크 I/O 및 셔플 정렬)
                                    ▼
[시간 T2: Reduce 단계] Core 1 (Reducer 1) | Core 2 (Reducer 2) | Core 3 (대기/기타) | Core 4 (대기/기타)
```

Mapper와 Reducer는 하드웨어 고정 장치가 아니라, **시간에 따라 동일한 CPU 코어들을 점유하며 순차적으로 실행되는 작업 단위(Task)**이다.

---

**◀ 이전 글**: [09. MapReduce 셔플의 해시 분배 규칙과 Combiner](mapreduce-shuffle-mechanism-and-combiner.md) | [🏠 목차 (README)](README.md) | **다음 글 ▶**: [11. PySpark의 Driver/Executor와 Shuffle](pyspark-driver-executor-and-shuffle.md)
