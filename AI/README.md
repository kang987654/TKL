# 🤖 AI Agent & LLM Reasoning Architecture

대규모 언어 모델(LLM)의 **확률적 본질 ──▶ 인식의 4범주(Known/Unknown) ──▶ 유동적 가설-검증 추론 프레임워크 ──▶ 실측 A/B 테스트** 흐름 순서대로 정리한 학습 노트입니다.  
각 문서는 LLM 및 AI 에이전트의 동작 메커니즘에 대한 통찰과 오개념 교정 과정을 담고 있습니다.

---

## 🗺️ AI 에이전트 추론 흐름도

```
[0. 원리 & 본질]  LLM의 확률적 본질 ──▶ Test-Time Compute & 모델별(GPT/Claude/Gemini) 사후학습 차이
                         │
                         ▼
[1. 인식의 범주]  Known/Unknown 4매트릭스 ──▶ '잘못된 확신(Unknown Unknown)'과 기계적 실측의 필요성
                         │
                         ▼
[2. 추론 프레임]  유동적 가설-검증 추론 ──▶ 확신의 가설 강등 & 실측 위계 (SKILL.md)
                         │
                         ▼
[3. 실증 & 한계]  Antigravity A/B 테스트 ──▶ 스킬의 실효성(안전성 확보 vs 과잉비용)과 적용 한계
```

---

## 📚 목차

| 순서 | 단계 | 문서 제목 & 링크 | 핵심 키워드 |
| :---: | :--- | :--- | :--- |
| **01** | **원리 & 본질** | [01. LLM의 확률적 본질과 플래그십 사고 모델의 진화](llm-probabilistic-nature-and-reasoning-models.md) | `Token Prediction`, `Inference-Time Compute`, `PRM` |
| **02** | **인식의 범주** | [02. LLM 추론과 인식의 4범주 (Known/Unknown 매트릭스)](llm-reasoning-and-known-unknown-matrix.md) | `Known/Unknown`, `Calibration`, `Elicitation` |
| **03** | **추론 프레임** | [03. 유동적 가설-검증 추론 프레임워크](fluid-hypothesis-verification-reasoning.md) | `Hypothesis`, `Verification Hierarchy`, `Forced Doubt` |
| **04** | **실증 & 한계** | [04. 가설-검증 스킬의 실측 A/B 테스트와 한계](hypothesis-verification-skill-ab-test.md) | `A/B Testing`, `Antigravity`, `Safety vs Overhead` |

---

## 📎 참고 자료 (Skills)
* [📄 hypothesis-verification.md](hypothesis-verification.md): 03번 프레임워크의 원본이자 04번 A/B 테스트에 실제 사용된 Antigravity 스킬 명세 전문
