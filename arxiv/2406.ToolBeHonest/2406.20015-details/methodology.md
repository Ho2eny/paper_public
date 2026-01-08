---
parent: ../2406.20015-analysis.md
source: ../2406.20015.md
---

# Methodology - Deep Analysis

## Paper Type: Benchmark

ToolBH는 Tool-augmented LLM의 **hallucination**을 진단하기 위한 **multi-level diagnostic benchmark**다.

---

## Approach Overview

### Design Philosophy

```
              ToolBH Design Framework
    ┌────────────────────────────────────────────┐
    │                                            │
    │   In-Depth (3 Levels)                      │
    │   L1: Solvability ──► L2: Planning ──► L3: Analysis │
    │                                            │
    │   In-Breadth (3 Scenarios)                 │
    │   MNT ─────────── PT ─────────── LFT       │
    │   (Missing)      (Potential)    (Limited)  │
    │                                            │
    └────────────────────────────────────────────┘
```

### Three-Level Diagnostic Process

| Level | Task | Input | Output | Metric |
|-------|------|-------|--------|--------|
| **L1** | Solvability Detection | Query + Toolset | "solvable" / "unsolvable" | Exact Match (EM) |
| **L2** | Solution Planning | Query + Toolset | Tool sequence | Progress Rate (PR) |
| **L3** | Missing-tool Analysis | Query + Toolset | Tool sequence + Missing tool description | PR + Matching Score (MS) |

---

## Key Design Choices

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| **Unsolvable Focus** | 50% solvable + 50% unsolvable | 기존 벤치마크의 "solvable 가정" 탈피 | [Section 3.1](../2406.20015.md#31-design-philosophy) |
| **Multi-level Evaluation** | 3단계 progressive diagnosis | 단순 정오답 → 원인 진단 | [Section 3.2](../2406.20015.md#32-in-depth-multi-level-diagnostic-task) |
| **UnsolvableQuery Tool** | 특수 도구로 unsolvable sub-goal 표시 | 해결 불가능 지점 명시적 식별 | [Section 3.2.2](../2406.20015.md#322-level-2-solution-planning) |
| **Scenario-based Tasks** | MNT/PT/LFT 3가지 시나리오 | 다양한 hallucination 유발 상황 커버 | [Section 3.3](../2406.20015.md#33-in-breadth-tool-based-hallucination-inducing-tasks) |
| **Embedding-based MS** | all-MiniLM-L6-v2 for similarity | Missing tool 설명의 semantic 유사도 측정 | [Section 3.2.3](../2406.20015.md#323-level-3-missing-tool-analysis) |

---

## Technical Details

### Evaluation Metrics

#### Level 1: Exact Match (EM)
```
EM = 1 if prediction == ground_truth else 0
```
- Binary classification: "solvable" vs "unsolvable"

#### Level 2 & 3: Progress Rate (PR)

$$PR = \frac{k-1}{|G|}$$

```python
def progress_rate(prediction: List[str], ground_truth: List[str]) -> float:
    """
    prediction: 모델이 예측한 도구 시퀀스
    ground_truth: 정답 도구 시퀀스
    """
    for k, (p, g) in enumerate(zip(prediction, ground_truth), 1):
        if p != g:
            return (k - 1) / len(ground_truth)

    # 완전 일치
    if len(prediction) >= len(ground_truth):
        return 1.0
    else:
        return len(prediction) / len(ground_truth)
```

#### Level 3: Matching Score (MS)

$$MS_{sub\text{-}goal} = \begin{cases} 1.0 & \text{if exact match in tool DB} \\ \cos(emb_u, emb_g) & \text{otherwise} \end{cases}$$

```python
def matching_score(predicted_desc: str, ground_truth_desc: str,
                   tool_db: Dict, embed_model) -> float:
    """
    predicted_desc: 모델이 예측한 누락 도구 설명
    ground_truth_desc: 정답 누락 도구 설명
    """
    # Step 1: Tool DB에서 exact match 시도
    if search_tool_db(predicted_desc, tool_db) == ground_truth_desc:
        return 1.0

    # Step 2: Embedding similarity
    emb_u = embed_model.encode(predicted_desc)
    emb_g = embed_model.encode(ground_truth_desc)
    return cosine_similarity(emb_u, emb_g)
```

### Three Scenarios (In-Breadth)

#### Missing Necessary Tools (MNT)

```
원본 태스크: "날씨 확인 후 이메일 전송"
도구: [WeatherAPI, EmailAPI]

MNT 변환: EmailAPI 제거
도구: [WeatherAPI]
정답: UnsolvableQuery (이메일 기능 없음)
```

**Sub-tasks**:
| Sub-task | Description | Avg Tools Used |
|----------|-------------|----------------|
| Single-step | 단일 도구로 해결 | 2.0 |
| Multi-step w/o Rep | 여러 도구, 반복 없음 | 5.8 |
| Multi-step w/ Rep | 여러 도구, 반복 있음 | 6.8 |

#### Potential Tools (PT)

```
OS 시나리오:
"Ubuntu에서 보안 강화를 위해 방화벽 설치"
도구: [PortManager, AccessController, AutoUpdater]
정답: UnsolvableQuery (방화벽 설치 도구 없음)

모델의 hallucination: "rm", "ufw" 같은 OS 명령어 사용 시도
```

#### Limited Functionality Tools (LFT)

```
Iterative 시나리오:
"HanLP로 NER, POS, 의존관계 분석"
도구: [HanLP(NER), HanLP(POS)]  # 의존관계 기능 없음
정답: [HanLP(NER), HanLP(POS), UnsolvableQuery]

Optimal Selection 시나리오:
"소수 언어 번역 + 1GB 이하 모델"
도구: [TranslatorA(영어만), TranslatorB(대형)]
정답: UnsolvableQuery (조건 충족 도구 없음)
```

---

## Data Curation Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                     Data Curation Pipeline                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  (1) Create Seed Samples                                        │
│      │                                                          │
│      ▼                                                          │
│  4 seed samples × 7 subtasks = 28 initial samples               │
│  (user query, toolset, golden solution)                         │
│                                                                 │
│  (2) Synthesize Samples                                         │
│      │                                                          │
│      ▼                                                          │
│  GPT-4o + Gemini-1.0-Pro with prompt templates                  │
│  → 300 samples per subtask                                      │
│                                                                 │
│  (3) Filter                                                     │
│      │                                                          │
│      ▼                                                          │
│  - Similarity check (embedding < 0.8)                           │
│  - Python code validation                                       │
│  - Manual review (ethical bias removal)                         │
│                                                                 │
│  (4) Construct Final Dataset                                    │
│      │                                                          │
│      ▼                                                          │
│  100 samples × 7 subtasks = 700 samples                         │
│  (50 solvable + 50 unsolvable per subtask)                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Evaluation Setup

### Models Evaluated

**Proprietary (7)**:
- GPT-3.5-Turbo, GPT-4-0613, GPT-4-1106, GPT-4-Turbo, GPT-4o
- Gemini-1.0-Pro, Gemini-1.5-Pro

**Open-weight (7)**:
- Llama-2: 7B, 13B, 70B
- Llama-3: 8B, 70B
- Mistral-7B, Mixtral-8x7B

### Inference Configuration
- **Temperature**: 0.0 (reproducibility)
- **Open-weight**: vLLM on 4× NVIDIA RTX 6000 Ada 48GB
- **Proprietary**: Official API

---

## Comparison with Other Benchmarks

| Benchmark | Unsolvable Tasks | Multi-level | Hallucination Focus | Tool-specific |
|-----------|:----------------:|:-----------:|:-------------------:|:-------------:|
| **ToolBH** | ✓ (50%) | ✓ (3 levels) | ✓ | ✓ |
| BFCL | ✗ (Irrelevance only) | ✗ | ✗ | ✓ |
| ToolSandbox | ✓ (Insufficient Info) | ✗ | ✗ | ✓ |
| MetaTool | ✓ (None output) | ✗ | Partial | ✓ |
| HaluEval | ✗ | ✗ | ✓ | ✗ |

---

## Source References

- **Design Philosophy**: [Section 3.1](../2406.20015.md#31-design-philosophy)
- **Multi-level Tasks**: [Section 3.2](../2406.20015.md#32-in-depth-multi-level-diagnostic-task)
- **Scenarios**: [Section 3.3](../2406.20015.md#33-in-breadth-tool-based-hallucination-inducing-tasks)
- **Data Curation**: [Section 3.4](../2406.20015.md#34-data-curation)
- **Evaluation Setup**: [Section 4.2](../2406.20015.md#42-evaluation-settings)
