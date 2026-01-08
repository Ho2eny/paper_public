---
parent: "../2504.18851-analysis.md"
source: "../2504.18851.md"
type: detail
created: 2026-01-09
---

# Methodology Deep Analysis: When2Call

> **Navigation**: [Main Analysis](../2504.18851-analysis.md) | [TL;DR](./tldr.md) | [Contributions](./contributions.md) | [Key Sections](./key-sections.md)

## Methodology Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    When2Call Framework                       │
├─────────────────────────────────────────────────────────────┤
│  [Data Generation]                                          │
│    BFCL Live / APIGen → Filter → Modify → Generate Options  │
├─────────────────────────────────────────────────────────────┤
│  [Evaluation]                                               │
│    Log-prob MC (Open) │ LLM-as-Judge (Closed)               │
├─────────────────────────────────────────────────────────────┤
│  [Training]                                                 │
│    Baseline SFT → When2Call SFT │ When2Call RPO             │
└─────────────────────────────────────────────────────────────┘
```

---

## 1. Data Generation Pipeline

### 1.1 Source Data Selection ([Section 2.2](../2504.18851.md#22-data-generation))

| Source | 용도 | 선택 이유 |
|--------|-----|----------|
| BFCL v2 Live | Benchmark | 인간 생성, permissive license |
| APIGen | Training | 검증된 합성 데이터 |

### 1.2 Step 1: Tool Requirement Classification

**목적**: 도구 호출이 반드시 필요한 질문만 필터링

**분류 카테고리** ([Table 9](../2504.18851.md#table-9)):
- Real-time information (날씨, 주가 등)
- Database access (기록 조회 등)
- Specialized tools (통계 SW, ML 라이브러리 등)
- Calculator (수학 계산)
- Public data (사전 지식으로 답변 가능 - 제외)

**구현**: Mixtral 8x22B를 분류기로 사용

```python
# Pseudo-code for classification
def classify_question(question, tool_spec):
    prompt = TABLE_9_TEMPLATE.format(question=question)
    category = mixtral_8x22b(prompt)
    return category not in ["Public data", "Calculator"]
```

### 1.3 Step 2: Question Modification

각 원본 질문에서 3가지 변형 생성:

#### (b) Tool Call - 원본 유지
- 원본 질문 그대로 사용
- BFCL/APIGen의 정답 도구 호출 상속

#### (c) Follow-up Question 생성 ([Table 10](../2504.18851.md#table-10))

**핵심 기법**: 필수 파라미터 하나 제거

```
원본: "What is the stock price of AAPL?"
     → required_params: [ticker]

변형: "What is the current stock price?"
     → removed: ticker
     → 정답: "Which stock ticker would you like me to look up?"
```

**품질 향상 전략**:
> "By breaking down the problem, we can avoid the generation model needing to parse the tool specification at all"

- 최소 2개 required parameter가 있는 도구만 사용
- 파라미터 제거를 명시적으로 지정

#### (d) Unable to Answer 생성 ([Table 10](../2504.18851.md#table-10))

**핵심 기법**: 같은 도메인의 답변 불가 질문 생성

```
도구: get_student_records(student_id) → 학생 기록 조회
생성: "What are the current grades for student ID 12345?"
     → 성적은 조회 불가능
```

**난이도 설계 ([Section 2.4](../2504.18851.md#24-difficulty-of-tool-mismatches))**:
> "We address this in When2Call firstly by constructing the questions with the target 'unable to answer' to be in the same semantic domain as the tool"

### 1.4 Step 3: Multiple-Choice Option Generation

각 질문에 대해 4가지 선택지 모두 생성:

| 정답 유형 | (a) Direct | (b) Tool Call | (c) Follow-up | (d) Unable |
|-----------|-----------|---------------|---------------|------------|
| Tool Call | 환각 답변 | **정답** | 불필요한 질문 | 과도한 거부 |
| Follow-up | 환각 답변 | 파라미터 환각 | **정답** | 과도한 거부 |
| Unable | 환각 답변 | 도구 환각 | 불필요한 질문 | **정답** |

---

## 2. Evaluation Methodology

### 2.1 Log-probability Multiple Choice ([Section 3](../2504.18851.md#3-multiple-choice-evaluation))

**장점**:
- 재현 가능
- 빠른 평가
- 선택지 순서, 개수 영향 최소화

**구현 (LM Evaluation Harness)**:
```python
# Pseudo-code
def evaluate(model, question, options):
    log_probs = []
    for option in options:
        prompt = format_prompt(question, option)
        log_prob = model.get_log_prob(prompt, option)
        log_probs.append(log_prob)

    # Length normalization
    normalized = [lp / len(opt) for lp, opt in zip(log_probs, options)]
    return argmax(normalized)
```

**메트릭**:
- Accuracy (raw, length-normalized)
- Macro F1
- Tool Hallucination Rate

### 2.2 Model-Specific Prompting ([Section 3.1](../2504.18851.md#31-model-specific-prompt-templates))

각 모델의 선호 형식 존중:

| 모델 | 시스템 프롬프트 | 도구 형식 |
|------|---------------|----------|
| Qwen 2.5 | 공식 문서 템플릿 | `<tools>` XML |
| Llama 3.1 | 최소 프롬프트 | JSON |
| xLAM | 모델 지정 | 모델 지정 |

### 2.3 LLM-as-Judge ([Section 4](../2504.18851.md#4-llm-as-judge-evaluation))

**대상**: 폐쇄형 모델 (GPT-4o, GPT-4-Turbo 등)

**프로세스**:
```
1. 모델에게 자유 형식 응답 생성 요청
2. GPT-4-Turbo가 응답을 4개 카테고리로 분류
3. 분류 결과로 메트릭 계산
```

**주의사항** ([Table 5](../2504.18851.md#table-5)):
> "Multiple-choice sometimes underestimates performance for models that see specific answer phrasings as part of the When2Call training set"

When2Call 학습 모델은 LLM-as-Judge 사용 권장

---

## 3. Training Methodology

### 3.1 Baseline: Tool-calling SFT ([Section 3.3.1](../2504.18851.md#331-supervised-fine-tuning))

**데이터 구성**:
- 공개 데이터셋: Glaive v2, Hermes
- 균형 샘플링: 단일 도구, 다중 도구, multi-turn

**문제점**: "Unable to answer"와 "Follow-up" 예시 부족

### 3.2 When2Call SFT ([Section 3.3.1](../2504.18851.md#331-supervised-fine-tuning))

**데이터 블렌딩**:
```
Ratio: Tool-calling : Non-tool-calling = 2 : 1
```

**하이퍼파라미터**:
| 모델 | Learning Rate | Warm-up |
|------|---------------|---------|
| 4B | 5e-6 | None |
| 8B | 4e-6 | None |

**결과**: When2Call 향상 but BFCL AST 하락 (과보수적)

### 3.3 When2Call RPO ([Section 3.3.2](../2504.18851.md#332-preference-optimization))

**Reward-aware Preference Optimization (RPO)**:

```
Chosen: 정답 선택지
Rejected: 오답 선택지 (uniform sampling)
```

**추가 데이터**: 도구 호출 정확도 유지용
```
Chosen: 올바른 tool call
Rejected:
  - 필수 파라미터 누락
  - 잘못된 인자 값
  - 일부 tool call 누락 (multi-tool)
```

**최종 블렌딩**: When2Call : Tool-call correction = 1:1

**하이퍼파라미터**:
| 모델 | Learning Rate | KL Penalty | Warm-up |
|------|---------------|------------|---------|
| 4B | 9e-7 | 0.05 | 10 steps |
| 8B | 7e-7 | 0.05 | 10 steps |

**핵심 인사이트**:
> "We find that a low KL-penalty value (0.05) gives the best results on tool-calling benchmarks"

---

## 4. Metrics & Analysis

### 4.1 Hallucination Rate Calculation ([Appendix E.1](../2504.18851.md#e1-calculating-hallucination-rates))

| 환각 유형 | 계산 방법 |
|-----------|----------|
| Tool Hallucination | `P(b \| tools=0)` |
| Answer Hallucination | `sum(predict=a) / total` |
| Parameter Hallucination | `P(b \| correct=c)` |

### 4.2 Confusion Matrix Analysis ([Appendix E.2](../2504.18851.md#e2-confusion-matrices-on-when2call))

**모델별 패턴 예시**:

Llama 3.1 70B ([Table 13](../2504.18851.md#table-13)):
```
              Pred: (a)  (b)   (c)   (d)
True (b)          0    1287    5     0
True (c)          4    1009   44     5
True (d)        100    1030  115    50
```
→ 거의 항상 도구 호출 시도

Qwen 2.5 72B ([Table 15](../2504.18851.md#table-15)):
```
              Pred: (a)  (b)   (c)   (d)
True (b)         11   1116  167     1
True (c)         12    365  684     1
True (d)        162    478  599    56
```
→ "Unable to answer" 극도로 회피

---

## 5. Reproducibility

### 공개 자료
- **코드 & 데이터**: [https://github.com/NVIDIA/When2Call](https://github.com/NVIDIA/When2Call)
- **평가 스크립트**: LM Evaluation Harness task
- **Confusion matrix 생성**: 제공 스크립트

### 재현 시 주의사항
1. 모델별 프롬프트 템플릿 사용 필수
2. Length normalization 적용
3. RPO의 낮은 KL penalty (0.05) 중요

---

## Source References

- 데이터 생성: [Section 2.2](../2504.18851.md#22-data-generation), [Appendix C](../2504.18851.md#c-prompts-for-synthetic-data-generation)
- 평가 방법: [Section 3](../2504.18851.md#3-multiple-choice-evaluation), [Section 4](../2504.18851.md#4-llm-as-judge-evaluation)
- 학습 방법: [Section 3.3](../2504.18851.md#33-training)
- 메트릭: [Appendix E](../2504.18851.md#e-confusion-matrices-and-hallucination-rates)
