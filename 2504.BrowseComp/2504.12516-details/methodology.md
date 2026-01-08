---
parent: ../2504.12516-analysis.md
source: ../2504.12516.md
created: 2026-02-06
---

# Methodology - Deep Analysis

## Paper Type

**Type**: Benchmark

---

## Approach Overview

### High-Level Summary

BrowseComp는 웹 브라우징 에이전트의 핵심 역량을 측정하기 위해 설계된 벤치마크이다. 핵심 방법론은 "inverted question design"으로, 답을 먼저 정한 후 그 답을 찾기 위해 방대한 웹 탐색이 필요하도록 질문을 역으로 구성한다. 이를 통해 "검증은 쉽지만 탐색은 어려운" 문제를 체계적으로 생성한다.

데이터 수집은 전적으로 인간 작업자에 의해 수행되며, 3단계 난이도 검증(모델 테스트, 검색 테스트, 인간 교차 검증)을 통해 품질을 보장한다. 최종 데이터셋은 1,266개 질문으로 구성되며, 각 질문은 짧은 문자열 답변을 가진다.

평가는 AI 모델 기반 자동 grader(Humanity's Last Exam과 동일한 프롬프트)를 사용하여 예측 답변과 참조 답변의 의미적 동치성을 판단한다. 이로써 대규모 자동 평가가 가능하며, 추가적으로 모델의 confidence score를 수집하여 캘리브레이션 분석도 수행한다.

### Key Innovation
기존 QA 벤치마크가 "사람이 쉽게 찾을 수 있는 정보"를 대상으로 하는 반면, BrowseComp는 "사람도 찾기 어려운 정보"를 의도적으로 타겟하여 브라우징 에이전트만의 고유한 가치(끈기, 병렬 탐색, 전략적 검색)를 평가한다.

**Source**: [Section 2](../2504.12516.md#2-data-collection-and-verification)

---

## Technical Details: Benchmark

### Task Definition

| Aspect | Description | Source |
|--------|-------------|--------|
| **Goal** | 웹 브라우징을 통해 hard-to-find 정보를 탐색 | [Section 1](../2504.12516.md#1-introduction) |
| **Input** | 자연어 질문 (제약 조건이 엮인 복합 질의) | [Table 1](../2504.12516.md#1-introduction) |
| **Output** | 짧은 문자열 답변 (1-5 단어) | [Section 2.1](../2504.12516.md#21-browsecomp-criteria) |
| **Success Criteria** | AI grader가 참조 답변과 의미적 동치 판정 | [Section 2.3](../2504.12516.md#23-grading) |

### Dataset Construction

| Aspect | Value | Source |
|--------|-------|--------|
| **Size** | 1,266 questions (원래 1,287에서 21개 제거) | [Section 4.5](../2504.12516.md#45-distribution-of-pass-rates) |
| **Sources** | 인간 작업자 직접 생성 (순수 인간 제작) | [Section 2](../2504.12516.md#2-data-collection-and-verification) |
| **Splits** | 단일 테스트셋 (train/val 분할 없음) | [Section 1](../2504.12516.md#1-introduction) |
| **Annotation** | Inverted question 방식 + 3단계 검증 | [Section 2.1](../2504.12516.md#21-browsecomp-criteria) |
| **Topics** | 10개 카테고리 (Sci&Tech, History, Entertainment 등) | [Section 2.2](../2504.12516.md#22-dataset-diversity) |
| **Answer Format** | 짧은 문자열 (인물명, 작품명, 사건명 등) | [Table 1](../2504.12516.md#1-introduction) |

### Data Collection Pipeline

```
1. 작업자가 Seed 선택 (인물/사건/논문 등)
   ↓
2. 여러 특성을 조합하여 Inverted Question 구성
   ↓
3. 검증 단계 1: GPT-4o, o1, Deep Research로 해결 불가 확인
   ↓
4. 검증 단계 2: 5회 Google 검색으로 첫 페이지에서 답 미발견 확인
   ↓
5. 검증 단계 3: 다른 작업자 10분 내 해결 불가 확인
   ↓
6. 40%+ 해결률 시 수정 요구
   ↓
7. 최종 검토 후 데이터셋 포함
```

### Metrics

| Metric | What it Measures | Pros | Cons | Source |
|--------|------------------|------|------|--------|
| **Accuracy** | AI grader 기반 정답률 | 직관적, 자동화 가능 | 복수 정답 미대응 | [Section 2.3](../2504.12516.md#23-grading) |
| **Calibration Error** | Confidence vs 실제 정확도 괴리 | 신뢰성 측정 | 프롬프트 의존적 | [Section 4.2](../2504.12516.md#42-calibration-analysis) |
| **Pass Rate** | 64회 시도 중 정답 비율 | 문제별 난이도 파악 | 높은 compute 비용 | [Section 4.5](../2504.12516.md#45-distribution-of-pass-rates) |

### Baselines

| Method | Type | Accuracy | Calibration Error | Source |
|--------|------|----------|-------------------|--------|
| GPT-4o | LLM (no tools) | 0.6% | 69% | [Table 3](../2504.12516.md#41-performance-of-openai-models) |
| GPT-4o w/ browsing | LLM + search | 1.9% | 82% | [Table 3](../2504.12516.md#41-performance-of-openai-models) |
| GPT-4.5 | LLM (no tools) | 0.9% | 68% | [Table 3](../2504.12516.md#41-performance-of-openai-models) |
| OpenAI o1 | Reasoner (no tools) | 9.9% | 65% | [Table 3](../2504.12516.md#41-performance-of-openai-models) |
| Deep Research | Agent (browsing-trained) | 51.5% | 91% | [Table 3](../2504.12516.md#41-performance-of-openai-models) |
| Human (2hr limit) | Human baseline | 29.2% | - | [Table 2](../2504.12516.md#3-human-performance-on-browsecomp) |

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| 질문 생성 방식 | Inverted (답 -> 질문) | 높은 난이도 보장, 검증 용이 | 순방향 QA, 자동 생성 | [Section 2.1](../2504.12516.md#21-browsecomp-criteria) |
| 답변 형식 | 짧은 문자열 | 자동 채점 용이, 모호성 최소화 | 장문 답변, 다단계 답변 | [Section 1](../2504.12516.md#1-introduction) |
| 채점 방식 | AI grader (LLM) | 의미적 동치 판단 가능 | 정확 문자열 매치, 인간 채점 | [Section 2.3](../2504.12516.md#23-grading) |
| 난이도 기준 | 모델+인간 교차 검증 | 다각도 난이도 보장 | 모델 기반만, 인간 기반만 | [Section 2.1](../2504.12516.md#21-browsecomp-criteria) |
| 데이터 소스 | 순수 인간 생성 | 높은 품질, 다양성 | 자동 생성, 크라우드소싱 | [Section 2](../2504.12516.md#2-data-collection-and-verification) |
| 토픽 다양성 | 작업자 개인 관심사 기반 | 자연스러운 다양성, 높은 품질 | 카테고리별 할당 | [Section 2.2](../2504.12516.md#22-dataset-diversity) |

---

## Assumptions & Limitations

### Assumptions
1. **단일 정답 가정**: 각 질문에 유일한 정답이 존재한다고 가정하지만, 저자들도 이를 "likely but not guaranteed"로 인정 - [Section 2.1](../2504.12516.md#21-browsecomp-criteria)
2. **웹 콘텐츠 안정성**: 질문의 답이 시간이 지나도 변하지 않는다고 가정하지만, 웹은 지속적으로 변화 - [Section 2.1](../2504.12516.md#21-browsecomp-criteria)
3. **AI grader 정확성**: LLM 기반 채점이 정확하다고 가정하지만, edge case에서 오류 가능 - [Section 2.3](../2504.12516.md#23-grading)

### Limitations
1. **좁은 평가 범위**: 짧은 답변만 평가하므로, 장문 종합, 출처 인용, 모호성 해소 등 실제 브라우징 에이전트의 핵심 역량을 측정하지 못함
2. **복수 정답 문제**: Inverted question의 구조적 한계로 다른 유효 답변이 존재할 가능성. Human agreement rate 86.4%가 이를 반영
3. **OpenAI 모델 편향**: 평가 대상이 OpenAI 모델에 한정되어, 벤치마크의 범용성 주장과 괴리
4. **Data contamination**: Deep Research가 BrowseComp 유사 태스크로 학습되었으며, 향후 모델 학습 시 데이터 오염 우려 (canary string으로 부분 완화)
5. **텍스트 전용**: 이미지, 비디오, 인터랙티브 웹 요소를 통한 정보 탐색은 평가하지 못함

### Scope
- **In Scope**: 텍스트 기반 웹 검색을 통한 사실적 정보 탐색 능력
- **Out of Scope**: 장문 생성, 모호한 질의 처리, 멀티모달 브라우징, 실시간 정보 추적, 웹 인터랙션(폼 작성, 로그인 등)

---

## Complexity Analysis

| Aspect | Complexity | Notes |
|--------|------------|-------|
| **평가 시간 (per question)** | O(minutes) ~ O(hours) | Deep Research: 단일 질문에 수 분 ~ 수십 분 (effort에 따라) |
| **데이터셋 크기** | 1,266 questions | 비교적 소규모이나 난이도가 높아 변별력 충분 |
| **채점 비용** | O(1) per question | AI grader 단일 호출로 채점 가능 |
| **전체 평가 비용** | Significant | 64 샘플 x 1,266 질문 = 81,024 inference (aggregation 분석 시) |

---

## Grading Implementation

### Prediction Prompt (Appendix A)

```
SYSTEM_EXACT_ANSWER = "Your response should be in the following format:
Explanation: {your explanation for your final answer}
Exact Answer: {your succinct, final answer}
Confidence: {your confidence score between 0% and 100% for your answer}"
```

### Grading Prompt (Appendix B)

Humanity's Last Exam과 동일한 프롬프트를 사용하여:
1. 응답에서 최종 답변 추출 (`extracted_final_answer`)
2. 참조 답변과 의미적 비교 (`reasoning`)
3. 정답 여부 판정 (`correct`: yes/no)
4. Confidence score 추출 (`confidence`: 0-100%)

**Source**: [Appendix A](../2504.12516.md#a-additional-instruction-for-model-prediction), [Appendix B](../2504.12516.md#b-grading-prompt)

---

## Source References Summary

| Topic | Section | Link |
|-------|---------|------|
| Task Definition | Introduction | [Section 1](../2504.12516.md#1-introduction) |
| Data Collection | Criteria | [Section 2.1](../2504.12516.md#21-browsecomp-criteria) |
| Dataset Diversity | Topic Distribution | [Section 2.2](../2504.12516.md#22-dataset-diversity) |
| Grading | Evaluation Method | [Section 2.3](../2504.12516.md#23-grading) |
| Human Evaluation | Human Performance | [Section 3](../2504.12516.md#3-human-performance-on-browsecomp) |
| Model Evaluation | Results | [Section 4](../2504.12516.md#4-evaluation-of-models) |
| Grading Prompt | Implementation | [Appendix B](../2504.12516.md#b-grading-prompt) |

---

## Navigation

<- [Back to Main Analysis](../2504.12516-analysis.md)
