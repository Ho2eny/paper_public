---
parent: "../2306.06070-analysis.md"
source: "../2306.06070.md"
type: detail
created: 2026-01-09
---

# Detailed Methodology Analysis: MINDACT

[Back to Main Analysis](../2306.06070-analysis.md) | [Source Paper](../2306.06070.md)

---

## Overview

MINDACT는 실제 웹페이지의 대규모 DOM을 처리하기 위한 2단계 LLM 파이프라인이다.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         MINDACT Pipeline                            │
├─────────────────────────────────────────────────────────────────────┤
│  Input: Task Description + Current Webpage + Action History         │
│                                ↓                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Stage 1: Candidate Generation (DeBERTa-base)               │   │
│  │  - 1,135 elements → Top-50 candidates                       │   │
│  │  - Cross-encoder ranking                                     │   │
│  │  - ~85-89% Recall                                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                ↓                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Stage 2: Action Prediction (Flan-T5 or GPT-4)              │   │
│  │  - Multi-choice QA format (5 options + None)                │   │
│  │  - Iterative selection                                       │   │
│  │  - Element + Operation + Value prediction                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                ↓                                    │
│  Output: (Target Element, Operation, [Value])                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Stage 1: Candidate Generation

[Section 3.1](../2306.06070.md#31-candidate-generation-with-small-lms)

### Problem Formulation

실제 웹페이지는 평균 1,135개의 DOM 요소를 포함하며, 이를 직접 LLM에 입력하는 것은 불가능하다. 따라서 먼저 관련 있는 후보 요소를 필터링해야 한다.

**Input:**
- Task description: T
- Previous actions: A₁, A₂, ..., Aₜ₋₁
- Current webpage DOM: E = {e₁, e₂, ..., eₙ} (n ≈ 1,135)

**Output:**
- Top-k candidate elements: C = {c₁, c₂, ..., cₖ} (k = 50)

### Cross-Encoder Architecture

```
Query:  [CLS] Task: {T} Previous: {A₁, A₂, ...} [SEP]
Element: {Tag} {Text Content} {Attributes} {Parent/Child} [SEP]

Input = Concat(Query, Element)
       ↓
    DeBERTa-base (86M params)
       ↓
    [CLS] embedding → Linear → Score
```

**Query Construction (Figure 4):**
```
Query = concat(
    "Task: " + task_description,
    "Previous Actions: " + action_history
)
```

**Element Representation (Figure 4):**
```
Element = concat(
    element.tag,              # e.g., "button", "input"
    element.text_content,     # visible text
    element.attributes,       # id, class, aria-label 등
    parent.text,              # 부모 요소 텍스트
    children.text             # 자식 요소 텍스트
)
```

### Training

**Loss Function:** Binary Cross Entropy
```
L = -[y·log(σ(score)) + (1-y)·log(1-σ(score))]

where:
  y = 1 if element is target
  y = 0 if element is negative sample (random)
  σ = sigmoid function
```

**Negative Sampling:**
- 같은 웹페이지에서 랜덤하게 negative 요소 샘플링
- Positive:Negative 비율 = 1:N (N은 하이퍼파라미터)

### Inference

```python
def candidate_generation(webpage, query, k=50):
    scores = []
    for element in webpage.elements:
        score = cross_encoder(query, element)
        scores.append((element, score))

    # Sort by score descending
    sorted_elements = sorted(scores, key=lambda x: x[1], reverse=True)

    return sorted_elements[:k]
```

### Performance

| Split | Recall@50 |
|-------|-----------|
| Cross-Task | 88.9% |
| Cross-Website | 85.3% |
| Cross-Domain | 85.7% |

---

## Stage 2: Action Prediction

[Section 3.2](../2306.06070.md#32-action-prediction-with-llms)

### Problem Formulation

Top-50 후보에서 최종 타겟 요소를 선택하고, 해당 요소에 대한 작업을 예측한다.

**Input:**
- Task description: T
- Previous actions: A₁, A₂, ..., Aₜ₋₁
- Candidate elements: C = {c₁, c₂, ..., c₅₀}
- Webpage snippet (candidates + neighbors)

**Output:**
- Target element: e*
- Operation: {Click, Type, Select Option}
- Value (if Type or Select): v

### Multi-Choice QA Formulation

**왜 Multi-choice인가?**

1. **Generation baseline의 문제:**
   - 요소 전체를 생성하면 오류 가능성 높음
   - LLM이 존재하지 않는 요소 환각 가능

2. **Discrimination의 장점:**
   - 주어진 선택지에서 고르므로 환각 불가
   - 더 일반화 가능하고 sample-efficient

**Format (Figure 5):**
```
Based on the HTML webpage above, try to complete the following task:
Task: {task_description}
Previous actions: {action_history}

What should be the next action? Please select from the following choices:
A. [button] Submit Order
B. [input] Search products
C. [link] View cart
D. [select] Choose size
E. None of the above

Answer: {A/B/C/D/E}
Action: {CLICK/TYPE/SELECT}
Value: {value if needed}
```

### Iterative Selection Process

```python
def action_prediction(candidates, task, history, model):
    # Group candidates into chunks of 5
    groups = chunk(candidates, size=5)
    selected = []

    for group in groups:
        # Add "None" option
        options = group + ["None of the above"]

        # Get model prediction
        prediction = model.predict(task, history, options)

        if prediction != "None":
            selected.append(prediction)

    # If multiple selected, repeat with selected ones
    if len(selected) > 1:
        return action_prediction(selected, task, history, model)
    elif len(selected) == 1:
        return selected[0], predict_operation(selected[0])
    else:
        return None  # All rejected
```

### Training (Fine-tuning)

**Models:** Flan-T5 (Base: 220M, Large: 770M, XL: 3B)

**Objective:** Left-to-right language modeling
```
P(answer | context) = ∏ᵢ P(tokenᵢ | token₁, ..., tokenᵢ₋₁, context)
```

**Hyperparameters:**
| Parameter | Value |
|-----------|-------|
| Batch size | 32 |
| Epochs | 5 |
| Learning rate | 5e-5 |
| Max input length | 2048 |

### In-Context Learning (GPT-3.5/GPT-4)

Fine-tuning 없이 few-shot prompting으로 사용.

**Prompt Structure (Table 8):**
```
System: You are a helpful assistant that is great at
        website design, navigation, and executing tasks for the user

[3 demonstration examples]

User: <HTML snippet>
      Task: {task}
      Previous actions: {history}
      Choices: A/B/C/D/E
```

---

## Comparison: Generation vs Multi-Choice QA

[Table 2 Results](../2306.06070.md#43-results)

| Method | Element Acc | Step SR | Task SR |
|--------|-------------|---------|---------|
| Generation (Flan-T5-B) | 20.2% | 17.5% | 0.0% |
| MINDACT (Flan-T5-B) | 43.6% | 41.0% | 4.0% |
| MINDACT (Flan-T5-XL) | 55.1% | 52.0% | 5.2% |

**Improvement:** Multi-choice QA가 Generation 대비 2배 이상 성능 향상

---

## Data Preprocessing

[Section 4.2](../2306.06070.md#42-data-preprocessing-and-evaluation)

### Element Filtering Heuristics

```python
def filter_elements(raw_html):
    filtered = []
    for element in raw_html.elements:
        if not element.is_visible:
            continue
        if not has_semantic_meaning(element):
            continue
        filtered.append(element)
    return filtered

# Result: 1,135 → 580 elements
# Target element recall: 94.7%
```

### Equivalent Element Handling

동일한 효과를 내는 여러 요소 처리:
```
Button → 내부 Span 클릭 = 동일 효과
```

**Heuristic:**
1. Ground truth의 조상 중 가장 가까운 clickable 요소 찾기
2. 해당 요소의 모든 visible 자손 포함
3. 모든 equivalent 요소를 positive로 처리

---

## Evaluation Metrics

### Step-Level Metrics

**Element Accuracy:**
```
Ele_Acc = (# correct element predictions) / (# total steps)
```

**Operation F1:**
```
Op_F1 = Token-level F1 between predicted and ground truth operation

For Click: Accuracy (binary)
For Type/Select: F1 of value tokens
```

**Step Success Rate:**
```
Step_SR = (# steps where both element AND operation correct) / (# total steps)
```

### Task-Level Metrics

**Task Success Rate:**
```
Task_SR = (# tasks where ALL steps successful) / (# total tasks)

Very stringent:
- Average 7.3 steps per task
- One mistake = task failure
- Best model achieves < 5%
```

---

## Key Insights from Experiments

### 1. Generalization Gap

| Setting | Flan-T5-XL Step SR |
|---------|-------------------|
| Cross-Task | 52.0% |
| Cross-Website | 38.9% |
| Cross-Domain | 39.6% |

**Observation:** 10%+ 성능 하락, 하지만 Cross-Website와 Cross-Domain 유사

**Implication:** 일반화 어려움은 도메인 차이보다 웹사이트 디자인 차이에서 기인

### 2. GPT-4 Potential

| Model | Cross-Task Ele Acc |
|-------|-------------------|
| GPT-3.5 (3-shot) | 20.3% |
| GPT-4 (3-shot) | 41.6% |
| Flan-T5-XL (fine-tuned) | 55.1% |

**Observation:** GPT-4가 few-shot으로 fine-tuned 모델에 근접

### 3. None Option Issue

GPT-3.5의 낮은 성능 원인:
- None 옵션을 과도하게 선택
- "이 페이지에서 태스크 완료 불가"라고 판단
- 실제로 다단계 네비게이션 필요한 태스크에서 문제

---

## Limitations of MINDACT

1. **텍스트 기반만 사용:** 시각적 정보 미활용
2. **독립적 웹페이지 인코딩:** 환경 변화 미반영
3. **단일 턴 상호작용:** 사용자 피드백 불가
4. **오프라인 평가:** 실시간 환경과 차이

---

## Implementation References

**Code Repository:** [GitHub - OSU-NLP-Group/Mind2Web](https://github.com/OSU-NLP-Group/Mind2Web)

**Key Dependencies:**
- Playwright (annotation tool)
- Sentence-Transformers (cross-encoder)
- HuggingFace Transformers (Flan-T5)
- OpenAI API (GPT-3.5/GPT-4)

**Hardware:**
- DeBERTa: Single A6000 48GB
- Flan-T5-XL: 4x A100 80GB

---

[Back to Main Analysis](../2306.06070-analysis.md)