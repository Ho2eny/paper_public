---
parent: "../2504.18851-analysis.md"
source: "../2504.18851.md"
type: detail
created: 2026-01-09
---

# Core Contributions Analysis: When2Call

> **Navigation**: [Main Analysis](../2504.18851-analysis.md) | [TL;DR](./tldr.md) | [Key Sections](./key-sections.md) | [Methodology](./methodology.md)

## Contribution Overview

| 기여 유형 | 기여 내용 | 영향도 |
|-----------|----------|--------|
| **Benchmark** | When2Call 벤치마크 및 평가 프레임워크 | High |
| **Dataset** | 학습용 데이터셋 (SFT + RPO) | High |
| **Method** | RPO 기반 균형 학습 방법 | Medium-High |
| **Analysis** | 환각 유형별 정량화 및 오류 패턴 분석 | Medium |

---

## 1. When2Call Benchmark

### 핵심 혁신

기존 벤치마크(BFCL, ToolSandbox)가 "도구 호출 여부"만 확인하는 반면, When2Call은 **도구를 호출하지 않을 때 모델이 어떤 행동을 취하는지**까지 평가한다.

```
기존 방식: 도구 제공 → 올바른 호출? (Y/N)
When2Call: 도구 제공 → 호출 vs 추가질문 vs 불가 인정 vs 직접답변
```

### 벤치마크 구성 ([Section 2.3](../2504.18851.md#23-statistics))

| Split | (b) Tool Call | (c) Follow-up | (d) Unable | Total |
|-------|---------------|---------------|------------|-------|
| Test | 1,295 | 1,062 | 1,295 | 3,652 |
| Train | 2,000 | 2,000 | 2,000 | 6,000 |
| RPO | 4,500 | 3,000 | 3,000 | 10,500 |

### 난이도 설계 ([Section 2.4](../2504.18851.md#24-difficulty-of-tool-mismatches))

BFCL Irrelevance의 문제점:
> "The provided tools in BFCL Irrelevance are often largely unrelated to the question, making it easy for the LM to tell that they do not match"

When2Call의 해결:
- **의미적 유사성**: 질문과 도구가 같은 도메인에서 출제
- **미묘한 불일치**: 학생 기록 vs 성적 조회 (Figure 1 예시)
- **파라미터 누락**: 필수 파라미터 하나만 제거하여 follow-up 유도

---

## 2. Training Dataset & Methodology

### 데이터 생성 파이프라인 ([Section 2.2](../2504.18851.md#22-data-generation))

```
BFCL Live / APIGen
       ↓
[Step 1] Tool-requirement Classification (Mixtral 8x22B)
       ↓
[Step 2] Question Modification
       ├── Original → Tool Call 정답
       ├── Parameter 제거 → Follow-up 정답
       └── 관련 질문 생성 → Unable 정답
       ↓
[Step 3] 4가지 선택지 생성
```

### SFT vs RPO 비교 ([Section 3.3](../2504.18851.md#33-training))

| 방법 | When2Call F1 | BFCL AST | 특징 |
|------|-------------|----------|------|
| Baseline SFT | 31.9 | 62.2% | 기존 도구 호출 데이터만 |
| When2Call SFT | 49.4 | 57.5% | 과도하게 보수적 |
| **When2Call RPO** | **52.4** | **62.5%** | 균형 유지 |

RPO의 핵심:
> "RPO training successfully balances these two competing pressures and enhances the training signal by providing negative as well as positive examples" - [Section 5](../2504.18851.md#5-discussion)

---

## 3. Evaluation Framework

### 이중 평가 체계

1. **Log-probability Multiple Choice** ([Section 3](../2504.18851.md#3-multiple-choice-evaluation))
   - 빠르고 재현 가능
   - 오픈소스 모델 대상
   - LM Evaluation Harness 통합

2. **LLM-as-Judge** ([Section 4](../2504.18851.md#4-llm-as-judge-evaluation))
   - 폐쇄형 모델 평가 가능
   - GPT-4-Turbo를 judge로 사용
   - 자유 형식 응답 분류

### 환각 정량화 ([Appendix E.1](../2504.18851.md#e1-calculating-hallucination-rates))

| 환각 유형 | 측정 방법 | 의미 |
|-----------|----------|------|
| Tool Hallucination | 도구 없는 질문에서 (b) 선택 | 존재하지 않는 도구 호출 |
| Answer Hallucination | (a) Direct Answer 선택 빈도 | 도구 없이 추측 답변 |
| Parameter Hallucination | Follow-up 정답에서 (b) 선택 | 누락된 파라미터 추측 |

---

## 4. Analysis & Insights

### 모델별 오류 패턴 ([Section 5](../2504.18851.md#5-discussion))

| 모델 계열 | 주요 오류 패턴 |
|-----------|---------------|
| Llama | 항상 도구 호출 시도 (과적극) |
| Qwen/xLAM | "Unable to answer" 회피 |
| MNM (Baseline) | 균형적이나 낮은 정확도 |

### BFCL과의 관계 ([Figure 2](../2504.18851.md#figure-2))

> "Good performance on BFCL Irrelevance... is not enough to predict good performance on When2Call"

```
BFCL Irr. High + When2Call Low → 도구 거부는 하지만 올바른 대안 선택 실패
BFCL Irr. High + When2Call High → 진정한 도구 호출 의사결정 능력
```

---

## Contribution Impact Summary

1. **벤치마크**: 도구 호출 연구의 새로운 평가 차원 개척
2. **데이터셋**: 공개적으로 사용 가능한 학습 자원 제공
3. **방법론**: RPO를 통한 균형 학습의 효과 입증
4. **분석 도구**: 환각 유형별 진단 스크립트 제공

---

## Source References

- 벤치마크 설계: [Section 2](../2504.18851.md#2-when2call)
- 학습 방법: [Section 3.3](../2504.18851.md#33-training)
- 평가 프레임워크: [Section 3](../2504.18851.md#3-multiple-choice-evaluation), [Section 4](../2504.18851.md#4-llm-as-judge-evaluation)
- 오류 분석: [Section 5](../2504.18851.md#5-discussion), [Appendix E](../2504.18851.md#e-confusion-matrices-and-hallucination-rates)
