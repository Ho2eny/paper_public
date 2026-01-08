---
parent: ../2408.04682-analysis.md
source: ../2408.04682.md
---

# Core Contributions - Deep Analysis

## Contribution 1: Implicit State Dependencies

**What**: World state(cellular, WiFi, location service, low battery mode) 간 **implicit dependency chain** 모델링

**Mechanism**:
```
send_message() ──depends on──► cellular_service_on
                              └──depends on──► low_battery_mode_off (optional)

get_location() ──depends on──► location_service_on
               └──depends on──► wifi_on OR cellular_service_on
```

**Key Insight**:
- 도구 signature만으로는 dependency 파악 불가
- Agent가 world state를 **추론**해야 함
- 44%의 도구(15/34)가 stateful dependency 보유

**Why it matters**:
- **실제 환경 복잡성**: 스마트폰/IoT 환경의 implicit dependency 재현
- **Parallel call 문제 노출**: 대형 모델이 dependency 무시하고 병렬 호출
- **새로운 평가 차원**: 기존 벤치마크에서 다루지 않은 영역

**Limitation**: 정의된 dependency가 비교적 단순 (2-3 hop)

**Source**: [Section 2.1](../2408.04682.md#21-stateful-tool-execution)

---

## Contribution 2: LLM User Simulator

**What**: GPT-4o 기반 **on-policy** 대화형 평가를 위한 user simulator

**Design Elements**:

| Element | Purpose | Impact |
|---------|---------|--------|
| **User Goal** | 달성해야 할 태스크 명시 | 기본 지시 |
| **Knowledge Boundary** | 알아야/몰라야 할 것 명시 | Hallucination 12.4%→7.75% |
| **Demonstration** | Few-shot 예시 제공 | Instruction following 6.2%→0.77% |

**Error Rate Progression**:
```
User Goal only          → 12.4% hallucination, 6.2% instruction error
+ Knowledge Boundary    → 7.75% hallucination, 3.88% instruction error
+ Demonstration         → 6.97% hallucination, 0.77% instruction error
```

**Why it matters**:
- **On-policy 평가**: Off-policy trajectory의 distribution shift 문제 해결
- **Realistic Interaction**: Human-like 대화 패턴 재현
- **확장성**: 새로운 시나리오에 쉽게 적용 가능

**Limitation**: 여전히 ~8% 에러율, 복잡한 상황에서 불안정

**Source**: [Section 2.2](../2408.04682.md#22-llm-user-simulator), [Table 2-3](../2408.04682.md#table-2-percentage-of-user-simulation-failures)

---

## Contribution 3: Milestone/Minefield Evaluation

**What**: **DAG 기반 동적 평가**로 다양한 성공 경로 허용

**Milestone System**:
```
                    ┌─► get_contact(Alice)
User Request ───────┤
                    └─► turn_on_cellular()
                              │
                              ▼
                        send_message(Alice, "Hello")
                              │
                              ▼
                          Success
```

- **DAG Structure**: Partial order로 순서 제약만 명시
- **Multiple Paths**: 다양한 유효 경로 허용
- **Similarity Score**: 달성 milestone 비율 (0-1)

**Minefield System**:
- **정의**: 절대 하면 안 되는 행동 (hallucination 탐지)
- **예시**: timestamp 모를 때 `timestamp_diff` 호출 금지
- **효과**: Minefield 매칭 시 전체 점수 0점

**Similarity Score Formula**:
$$S = \frac{|M_{achieved}|}{|M_{total}|} \times (1 - \mathbb{1}_{minefield})$$

**Why it matters**:
- **유연한 평가**: 고정 trajectory 비교의 한계 극복
- **Hallucination 직접 탐지**: Minefield로 명확한 에러 케이스 정의
- **Off-policy 대안**: Trajectory-agnostic 평가 가능

**Limitation**: Milestone 설계에 전문가 annotation 필요

**Source**: [Section 2.3](../2408.04682.md#23-milestones-and-minefields)

---

## Contribution 4: Challenging Category Definition

**What**: SOTA 모델의 한계를 드러내는 **6가지 시나리오 카테고리** 정의

**Categories**:

| Category | Challenge | GPT-4o | Best |
|----------|-----------|--------|------|
| **State Dependency** | Implicit dependency 처리 | 84.0% | GPT-4o |
| **Canonicalization** | 상대 시간/위치 변환 | 76.6% | Claude-3.5-Sonnet 79.7% |
| **Insufficient Information** | "할 수 없다" 인식 | 42.0% | Gemini-1.5-Pro 76.2% |
| **Simple** | 기본 도구 사용 | 94.5% | GPT-4o |
| **Multi-step** | 순차적 도구 사용 | 77.9% | GPT-4o |
| **Sandbox Message** | 에러 메시지 처리 | 73.1% | GPT-4o |

**Key Findings**:
1. **State Dependency 역설**: 대형 모델이 parallel call로 인해 오히려 저조
2. **Canonicalization 공통 난점**: 모든 모델이 상대적 표현 변환에 취약
3. **Insufficient Information 역상관**: 강한 모델일수록 hallucination 경향

**Why it matters**:
- **취약점 진단**: 모델별 개선 포인트 명확화
- **벤치마크 난이도 조절**: 카테고리별 평가로 세밀한 분석
- **연구 방향 제시**: 각 카테고리별 개선 연구 가이드

**Source**: [Section 3](../2408.04682.md#3-test-scenarios), [Table 5](../2408.04682.md#table-5-comparing-the-average-similarity-score)

---

## Contribution Summary Table

| Contribution | Novelty | Impact | Source |
|--------------|---------|--------|--------|
| Implicit State Dependencies | High | 실제 환경 복잡성 반영 | Section 2.1 |
| LLM User Simulator | High | On-policy 대화형 평가 가능 | Section 2.2 |
| Milestone/Minefield Eval | High | Trajectory-agnostic 평가 | Section 2.3 |
| Challenging Categories | Medium | SOTA 모델 한계 진단 | Section 3 |
