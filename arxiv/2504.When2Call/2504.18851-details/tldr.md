---
parent: "../2504.18851-analysis.md"
source: "../2504.18851.md"
type: detail
created: 2026-01-09
---

# Extended TL;DR: When2Call

> **Navigation**: [Main Analysis](../2504.18851-analysis.md) | [Contributions](./contributions.md) | [Key Sections](./key-sections.md) | [Methodology](./methodology.md)

## One-Sentence Summary

When2Call은 LLM이 도구를 호출해야 할 때, 추가 질문을 해야 할 때, 그리고 답변할 수 없음을 인정해야 할 때를 평가하는 새로운 벤치마크로, 기존 벤치마크들이 간과한 "도구를 호출하지 말아야 할 때"의 의사결정을 다중 선택 형식으로 정량화한다.

## Extended Summary

### 문제 정의 (Problem Statement)

현재 도구 호출(tool-calling) 벤치마크들은 **"올바른 도구를 올바른 파라미터로 호출했는가"**에만 집중하고 있다. 그러나 실제 배포 환경에서는 다음과 같은 상황이 빈번하게 발생한다:

1. **도구 환각(Tool Hallucination)**: 시스템 프롬프트에 없는 도구를 호출하려 함
2. **파라미터 환각(Parameter Hallucination)**: 사용자가 제공하지 않은 정보를 추측하여 도구 호출
3. **답변 환각(Answer Hallucination)**: 도구 없이는 알 수 없는 정보를 직접 답변

> "If a model is not deployed with a tool that answers the question... the LM should say that it cannot answer the question, not hallucinate a previously seen weather tool" - [Section 1](../2504.18851.md#1-introduction)

### 해결 방안 (Solution)

When2Call은 4가지 선택지를 가진 **다중 선택 형식(Multiple-Choice Format)**으로 문제를 재정의:

| 선택지 | 설명 | 올바른 경우 |
|--------|------|------------|
| (a) Direct Answer | 도구 없이 직접 답변 | 항상 오답 (환각) |
| (b) Tool Call | 도구 호출 | 적절한 도구가 있을 때 |
| (c) Follow-up Question | 추가 정보 요청 | 필수 파라미터 누락 시 |
| (d) Unable to Answer | 답변 불가 인정 | 적절한 도구 없을 때 |

### 핵심 기여 (Key Contributions)

1. **벤치마크 설계**: 3,652개 테스트 샘플, BFCL Live에서 파생
2. **학습 데이터셋**: 6,000개 학습 샘플 (APIGen 기반)
3. **RPO 학습 방법**: 선호도 최적화를 통한 균형 잡힌 학습
4. **LLM-as-Judge 평가**: 폐쇄형 모델 평가 지원

### 주요 발견 (Key Findings)

| 모델 | When2Call F1 | Tool Hall% | BFCL AST |
|------|-------------|------------|----------|
| GPT-4o | 61.3 | 26% | 79.8% |
| Llama 3.1 70B | 37.8 | 57% | 68.3% |
| Qwen 2.5 72B | 32.8 | 23% | 69.3% |
| MNM 8B (RPO) | **52.4** | **1.2%** | 62.5% |

> "Most community models are unwilling to admit they cannot answer the question" - [Section 3.2](../2504.18851.md#32-results-for-community-models)

### 실무적 시사점 (Practical Implications)

1. **단순 SFT의 한계**: 부정 예시만 추가하면 모델이 과도하게 보수적으로 변함
2. **RPO의 효과**: 도구 호출 능력 유지하면서 환각 감소 (1.2% tool hallucination)
3. **모델별 오류 패턴**: Confusion matrix를 통해 개별 모델의 취약점 파악 가능

### 한계 및 향후 과제

- **합성 데이터 품질**: 92% 질문 품질, 94% 답변 품질 (수동 검증)
- **영어 전용**: 다국어 지원 필요
- **폐쇄형 모델 제한**: GPT 계열만 평가

---

## Source References

- 문제 정의: [Section 1 Introduction](../2504.18851.md#1-introduction)
- 벤치마크 설계: [Section 2 When2Call](../2504.18851.md#2-when2call)
- 실험 결과: [Section 3.2](../2504.18851.md#32-results-for-community-models), [Table 3](../2504.18851.md#table-3)
- 학습 방법: [Section 3.3](../2504.18851.md#33-training)
- 한계점: [Limitations](../2504.18851.md#limitations)
