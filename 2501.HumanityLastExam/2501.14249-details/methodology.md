---
parent: ../2501.14249-analysis.md
source: ../2501.14249.md
created: 2026-02-06
---

# Methodology - Deep Analysis

## Paper Type

**Benchmark**

---

## Approach Overview

### High-Level Summary

HLE는 "인류의 마지막 시험"이라는 야심찬 목표 하에, 기존 벤치마크의 포화 문제를 근본적으로 해결하기 위해 설계되었다. 핵심 접근법은 AI가 아닌 인간 전문가의 지식 최전선에서 질문을 구성하고, 다단계 필터링을 통해 현재 LLM이 답할 수 없는 질문만을 엄선하는 것이다.

데이터 수집은 전 세계 전문가 크라우드소싱에 기반하며, $500K 상금풀과 논문 공저자 인센티브로 고품질 제출을 유도한다. 수집된 질문은 LLM 사전 필터링(모델이 답하면 탈락) → 1차 전문가 리뷰(피드백/수정) → 2차 전문가 리뷰(최종 승인)의 3단계를 거치며, 릴리즈 후 커뮤니티 보정까지 포함한다.

평가 방식은 폐쇄형 질문(MC + exact-match)에 자동 채점(O3-MINI judge)을 적용하며, 정확도와 calibration error 두 가지 지표를 동시에 측정한다. 이는 모델의 "무엇을 알고 있는가"뿐 아니라 "무엇을 모르는지 아는가"까지 평가하는 포괄적 설계이다.

### Key Innovation

기존 벤치마크와의 핵심 차별점:
1. **적대적 구축(Adversarial Construction)**: LLM이 틀린 질문만 채택하는 선제적 난이도 보장
2. **글로벌 전문가 스케일**: ~1,000명 전문가로부터 100+ 분야의 질문 수집
3. **검색 방지(Non-searchable) 설계**: 인터넷에서 답을 찾을 수 없도록 원본/비자명 합성 질문 요구

**Source**: [Section 3 - Dataset](../2501.14249.md#3-dataset)

---

## Technical Details

### Task Definition

| Aspect | Description | Source |
|--------|-------------|--------|
| **Goal** | Frontier LLM의 전문가 수준 학술 지식 및 추론 능력 측정 | [Section 1](../2501.14249.md#1-introduction) |
| **Input** | 질문 텍스트 (+ 이미지, 14% 멀티모달) | [Section 3.1](../2501.14249.md#31-collection) |
| **Output** | Exact-match string 또는 MC 선택지 | [Section 3.1](../2501.14249.md#31-collection) |
| **Success Criteria** | 정확한 답 일치 (O3-MINI judge 검증) | [Section 4.1](../2501.14249.md#41-setup) |

### Dataset Construction

| Aspect | Value | Source |
|--------|-------|--------|
| **Size** | 2,500 questions (public + private) | [Section 3](../2501.14249.md#3-dataset) |
| **Sources** | ~1,000 experts, 500+ institutions, 50 countries | [Section 3.1](../2501.14249.md#31-collection) |
| **Splits** | Public set (대부분) + Private held-out set | [Section 3](../2501.14249.md#3-dataset) |
| **Format** | MC 24% (5+ choices) / Exact-match 76% | [Section 3.1](../2501.14249.md#31-collection) |
| **Multi-modal** | 14% include images | [Section 3.1](../2501.14249.md#31-collection) |
| **LLM Pre-filter** | 70,000 attempts → 13,000 forwarded | [Section 3.2](../2501.14249.md#32-review) |

### Quality Assurance Pipeline

```
[Expert Submission]
    │
    ▼
[LLM Difficulty Check] ── fail → 재작성 또는 폐기
    │ pass (모델이 틀림)
    ▼
[Round 1 Review] ── 1-3 전문 리뷰어
    │               피드백/수정/점수(0-5)
    ▼
[Round 2 Review] ── 고품질 리뷰어 + 오거나이저
    │               최종 승인/거절(0-6)
    ▼
[Public Release]
    │
    ▼
[Community Feedback] ── 버그 바운티, 감사, 검색 검토
    │
    ▼
[Final Dataset] + [Private Held-out Set]
```

### Metrics

| Metric | What it Measures | Pros | Cons | Source |
|--------|------------------|------|------|--------|
| Accuracy (%) | 정답 비율 | 직관적, 비교 용이 | 낮은 값에서 변별력 제한 | [Section 4.2](../2501.14249.md#42-quantitative-results) |
| RMS Calibration Error (%) | 모델 확신도와 실제 정확도의 괴리 | Hallucination 정량화 | Confidence 추출의 프롬프트 의존성 | [Section 4.2](../2501.14249.md#42-quantitative-results) |

### Evaluation Setup

| Aspect | Detail | Source |
|--------|--------|--------|
| **System Prompt** | "Explanation → Answer → Confidence" 구조 | [Section C.1.1](../2501.14249.md#c11-evaluation) |
| **Judge** | O3-MINI (structured decoding 활성화) | [Section 4.1](../2501.14249.md#41-setup) |
| **Temperature** | 0.0 (가능한 경우), O1/O3-MINI는 1.0 | [Table 4](../2501.14249.md#c5-model-versions) |

### Baselines

| Model | Type | Accuracy (%) | Cal. Error (%) | Source |
|-------|------|-------------|----------------|--------|
| O3-MINI (HIGH) | Reasoning | **13.4** | 80 | [Table 1](../2501.14249.md#table-1) |
| DeepSeek-R1 | Reasoning | 8.5 | **73** | [Table 1](../2501.14249.md#table-1) |
| O1 | Reasoning | 8.0 | 83 | [Table 1](../2501.14249.md#table-1) |
| Gemini 2.0 Flash Thinking | Reasoning | 6.6 | 82 | [Table 1](../2501.14249.md#table-1) |
| Gemini 1.5 Pro | Non-reasoning | 4.6 | 88 | [Table 1](../2501.14249.md#table-1) |
| Claude 3.5 Sonnet | Non-reasoning | 4.1 | 84 | [Table 1](../2501.14249.md#table-1) |
| Grok 2 | Non-reasoning | 3.0 | 87 | [Table 1](../2501.14249.md#table-1) |
| GPT-4O | Non-reasoning | 2.7 | 89 | [Table 1](../2501.14249.md#table-1) |

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| Question Format | MC 24% + Exact-match 76% | 자동 채점 가능 + 추측 방지 | Open-ended (채점 어려움) | [Section 3.1](../2501.14249.md#31-collection) |
| LLM Pre-filter | 제출 전 6개 모델로 검증 | 난이도 사전 보장 | 제출 후 필터링 (비효율) | [Section 3.2](../2501.14249.md#32-review) |
| Expert Incentive | $500K + 공저자 | 고품질 전문가 유인 | 무보수 (참여율 저하) | [Section 3.1](../2501.14249.md#31-collection) |
| Non-searchable | 원본/비자명 합성 요구 | 검색 기반 해결 방지 | 검색 가능 허용 (memorization 취약) | [Section 3.1](../2501.14249.md#31-collection) |
| Judge Model | O3-MINI (structured decoding) | 정확한 답 비교 | 인간 채점 (비용 높음) | [Section 4.1](../2501.14249.md#41-setup) |
| Public + Private Split | 대부분 공개 + 일부 비공개 | 연구 접근성 + 과적합 검출 | 전체 비공개 (접근성 부족) | [Section 3](../2501.14249.md#3-dataset) |

---

## Assumptions & Limitations

### Assumptions

1. **LLM 난이도 = 인간 전문가 난이도**: LLM이 틀린 질문이 인간에게도 어렵다고 가정 - [Section 3.2](../2501.14249.md#32-review)
2. **전문가 답변의 정확성**: 제출자의 답이 정확하다고 가정 (리뷰로 보완하나 15.4% 불일치 존재) - [Section B.3](../2501.14249.md#b3-expert-disagreement-rate-and-hle-rolling)
3. **자동 채점의 충실성**: O3-MINI judge가 답 동등성을 정확히 판단한다고 가정 - [Section 4.1](../2501.14249.md#41-setup)

### Limitations

1. **전문가 불일치율**: 15.4% (생물/화학/건강 18%)로 상당함. FutureHouse는 화학/생물 답변의 ~30%에 오류 가능성 지적 - [Reference 47](../2501.14249.md#ref-47)
2. **수학 편향**: 41%가 수학으로, 범용 학술 벤치마크로서의 균형 문제
3. **적대적 구축의 한계**: 특정 시점의 LLM에 기반한 필터링이므로, 새 모델에는 효과 감소
4. **영어 전용**: 다국어 전문 지식 평가 불가
5. **폐쇄형 한정**: 개방형 추론, 창의적 문제 해결, 연구 능력 등은 측정하지 않음

### Scope

- **In Scope**: 폐쇄형 학술 질문에 대한 전문가 수준 지식 및 추론
- **Out of Scope**: 개방형 연구, 에이전트 기반 도구 사용, 코드 실행, 다국어, 실시간 정보

---

## Complexity Analysis

| Aspect | Value | Notes |
|--------|-------|-------|
| **Data Collection** | ~6개월+ | 글로벌 전문가 모집, 70K+ LLM 시도 |
| **Review Process** | 2-stage, ~1-3 reviews/question | Round 1 피드백 + Round 2 승인 |
| **Evaluation Cost** | Medium | 8개 frontier 모델 × 2,500 질문 |
| **Maintenance** | Ongoing (HLE-Rolling) | 커뮤니티 피드백 기반 동적 업데이트 |

---

## Source References Summary

| Topic | Section | Link |
|-------|---------|------|
| Dataset Overview | Section 3 | [Link](../2501.14249.md#3-dataset) |
| Data Collection | Section 3.1 | [Link](../2501.14249.md#31-collection) |
| Review Process | Section 3.2 | [Link](../2501.14249.md#32-review) |
| Evaluation Setup | Section 4.1 | [Link](../2501.14249.md#41-setup) |
| Results | Section 4.2 | [Link](../2501.14249.md#42-quantitative-results) |
| Submission Details | Appendix B.1 | [Link](../2501.14249.md#b1-submission-process) |
| Expert Disagreement | Appendix B.3 | [Link](../2501.14249.md#b3-expert-disagreement-rate-and-hle-rolling) |
| Review Instructions | Appendix C.7 | [Link](../2501.14249.md#c7-human-review-instructions) |

---

## Navigation

<- [Back to Main Analysis](../2501.14249-analysis.md)
