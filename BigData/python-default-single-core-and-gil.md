# 일반 Python 스크립트는 멀티코어 CPU 환경에서도 기본적으로 단 1개의 코어만 사용한다

## 🤔 헷갈렸던 점
- 파이썬이 알아서 CPU의 여러 코어를 나누어 쓸 것이라고 생각했다.

## ✅ 정리
- **CPython과 GIL (Global Interpreter Lock)**:
  - 표준 파이썬 인터프리터는 메모리 안전성을 위해 한 시점에 **오직 1개의 스레드만 바이트코드를 실행하도록 잠금(Lock)**을 건다.
  - 이로 인해 멀티스레드를 생성해도 CPU 연산은 1개 코어 안에서 번갈아 실행될 뿐, 여러 코어가 동시에 연산하지 못한다.
- **예외 라이브러리들**:
  - `NumPy`, `PyTorch` 등 C/C++ 기반 연산은 파이썬 바깥에서 GIL을 일시 해제하므로 멀티코어를 쓴다.
  - 하지만 순수 파이썬 로직, 문자열 파싱, 대부분의 `Pandas` 연산(`apply`, `groupby`)은 **무조건 단 1개 코어**만 사용한다.
- **해결 방식의 발전**:
  1. `multiprocessing`: 프로세스를 코어 수만큼 띄워 GIL을 우회 (내 PC 1대 자원 한계).
  2. `PySpark / Ray`: 데이터를 파티션 단위로 분할하여 1대 PC의 모든 코어뿐만 아니라 수백 대 서버로 동일하게 확장.

### 💡 실사용 예시: 왜 대용량 데이터에서 Pandas 대신 PySpark를 써야 하는가?
```py
# Pandas: 1개 코어만 100% 점유하며 대용량 처리 시 OOM(Out of Memory) 발생
import pandas as pd
df = pd.read_csv("huge_data.csv")  # 1개 CPU 코어만 고군분투

# PySpark: 모든 CPU 코어를 가동하여 병렬 분산 처리
from pyspark.sql import SparkSession
spark = SparkSession.builder.master("local[*]").getOrCreate()
df_spark = spark.read.csv("huge_data.csv")  # 멀티코어 전체 활용
```

일반 파이썬은 **1개 코어만 쓰는 1인 일꾼**이며, 대용량 데이터를 처리하려면 **멀티코어/다중 서버를 풀가동하는 분산 프레임워크(PySpark)**가 필수적이다.

---

| [🏠 목차 (README)](README.md) | **다음 글 ▶** [02. 분산 처리의 본질과 단일 노드 의사 분산](scale-out-and-pseudo-distributed-on-single-node.md) |
