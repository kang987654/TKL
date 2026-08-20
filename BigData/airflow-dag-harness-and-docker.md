# Airflow는 LangGraph처럼 DAG로 작업 흐름을 제어하는 하네스이고, Docker는 OS까지 통째로 패키징한 가상환경이다

## 🤔 헷갈렸던 점
- 데이터 파이프라인 지휘자인 `Apache Airflow`의 역할이 모호했다.
- Docker가 파이썬 가상환경(`venv`)과 본질적으로 무엇이 다른지 와닿지 않았다.

## ✅ 정리
- **Apache Airflow = 데이터 파이프라인의 LangGraph (하네스 역할)**:
  - `LangGraph`가 AI 에이전트의 동작 순서를 노드와 엣지로 엮듯, `Airflow`는 **데이터 엔지니어링 작업(수집 $\rightarrow$ Spark AI $\rightarrow$ DB 적재)**을 단방향 비순환 그래프(DAG)로 제어한다.
  - 단순 스케줄러(리눅스의 `cron` 등)가 아닌 **작업 간 선후관계 의존성 관리, 실패 시 자동 재시도(Retry), 실패 알림, 웹 UI 모니터링, 과거 데이터 재처리(Backfill)**를 완벽히 지원한다.
- **Docker = OS와 프로그램을 통째로 담은 가상환경 (venv의 확장판)**:
  - `venv`: 파이썬 라이브러리 목록만 격리 (호스트 PC에 Java, C 의존성이 없으면 실행 실패).
  - `Docker`: **Linux OS + Java + Python + Kafka + Spark 프로그램 전체**를 컨테이너 상자에 통째로 패키징하여 어떤 컴퓨터에서든 동일하게 1초 만에 실행.

### 💡 실사용 예시: Airflow DAG에서의 작업 의존성 정의
```py
from airflow import DAG
from airflow.operators.bash import BashOperator

with DAG(dag_id="ai_pipeline_dag", schedule_interval="@daily") as dag:
    task_ingest = BashOperator(task_id="ingest_data", bash_command="python ingest.py")
    task_spark = BashOperator(task_id="spark_ai", bash_command="spark-submit ai_job.py")
    task_export = BashOperator(task_id="export_db", bash_command="python export.py")

    # LangGraph처럼 직관적인 엣지(Edge) 정의: 수집 -> Spark -> DB
    task_ingest >> task_spark >> task_export
```

Airflow는 **복잡한 분산 데이터 작업들의 안전한 실행 흐름을 보장하는 지휘자(Harness)**이며, Docker는 **인프라 종속성을 없애주는 컨테이너 환경**이다.

---

**◀ 이전 글**: [08. PySpark의 Driver/Executor와 Shuffle](pyspark-driver-executor-and-shuffle.md) | [🏠 목차 (README)](README.md)
