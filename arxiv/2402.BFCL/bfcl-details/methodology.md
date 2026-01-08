---
parent: ../bfcl-analysis.md
source: ../bfcl.md
---

# Methodology - Deep Analysis

## Paper Type: Benchmark

BFCL은 LLM의 function calling 능력을 평가하기 위한 **대규모 다중 태스크 벤치마크**다.

---

## Approach Overview

### Design Philosophy

```
                    BFCL Design Space
    ┌─────────────────────────────────────────────┐
    │                                             │
    │   Complexity: Simple ───────────► Agentic   │
    │                                             │
    │   Realism: Synthetic ────────► Crowd-sourced│
    │                                             │
    │   Evaluation: LLM Judge ──────► Deterministic│
    │                                             │
    └─────────────────────────────────────────────┘
```

### Four-Part Structure

| Part | Purpose | Size | Complexity |
|------|---------|------|------------|
| **Single-turn** | 기본 function call 능력 | ~2,000 | Low |
| **Crowd-sourced** | 현실 복잡성 + contamination 탐지 | 2,251 | Medium |
| **Multi-turn** | 컨텍스트 관리 + 동적 의사결정 | 1,000 | High |
| **Agentic** | 메모리, 검색, SQL 등 에이전트 태스크 | ~300 | Highest |

---

## Key Design Choices

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| **Evaluation Method** | AST Substring Matching | 실행 없이 결정론적 평가 → 확장성 | [Section 4.1](../bfcl.md#41-ast-substring-matching) |
| **Data Source** | Expert + Crowd-sourced | Synthetic 한계 극복 + 현실 복잡성 | [Section 3.2](../bfcl.md#32-crowd-sourced-dataset) |
| **Languages** | 5개 (Py/Java/JS/REST/SQL) | 실제 개발 환경 반영 | [Appendix H](../bfcl.md#h-bfcl-ast-substring-matching) |
| **Multi-turn Eval** | State + Response based | 상태 변경과 호출 경로 모두 검증 | [Section 4.4](../bfcl.md#44-state--response-based-evaluation) |
| **Agentic Eval** | Exact Match | 엄격한 정답 기준 | [Section 4.5](../bfcl.md#45-exact-match-evaluation) |

---

## Technical Details

### AST Substring Matching Algorithm

```
Input: model_output, ground_truth_functions, valid_value_sets
Output: correct (boolean)

1. Extract function calls from model_output using ast.parse()
2. For each extracted call (func_name, params):
   a. Check func_name == ground_truth.func_name
   b. For each param (name, value):
      i. Check value ∈ valid_value_sets[name]
3. Return True if all checks pass
```

**Valid Value Set Examples**:
```python
# Date parameter
valid_values["date"] = ["20th June", "2023-06-20", "06/20/2023", "Jun.20, 2023"]

# Location parameter
valid_values["location"] = ["New York City", "NYC"]
```

**Normalization Rules**:
- Case insensitive
- Whitespace removed
- Punctuation stripped: `,./-_*^`

### Multi-turn Evaluation Pipeline

```
┌─────────────────────────────────────────────────────────┐
│                    Turn N                               │
├─────────────────────────────────────────────────────────┤
│  User Query → Model Response → Function Calls → State   │
│                                                         │
│  ┌─────────────────┐    ┌──────────────────┐           │
│  │ State-based     │    │ Response-based   │           │
│  │ Evaluation      │    │ Evaluation       │           │
│  │                 │    │                  │           │
│  │ Final State =   │    │ Call Sequence ⊇  │           │
│  │ Ground Truth?   │    │ Min Required?    │           │
│  └────────┬────────┘    └────────┬─────────┘           │
│           │                      │                      │
│           └──────────┬───────────┘                      │
│                      ▼                                  │
│               Both Pass? → Turn Correct                 │
└─────────────────────────────────────────────────────────┘
```

### Agentic Dataset Domains

| Domain | Tools | Example Task |
|--------|-------|--------------|
| **Web Search** | DuckDuckGo, Fetch | "2024 TIME Person of the Year는?" |
| **Memory** | list_keys, retrieve, store | "이전 대화에서 언급한 내 전공은?" |
| **SQL** | SELECT, INSERT, UPDATE, DELETE | "users 테이블에서 age > 30 조회" |

---

## Data Curation Pipeline

### Single-turn

```
GitHub Repos ──► Extract Functions ──► Generate Queries ──► Augment ──► Validate
     │                 │                    │                │           │
     │                 ▼                    ▼                ▼           ▼
     │           Filter trivial      GPT-4 생성        Parallel/Multiple  Unit test
     │           (< 2 params)                         Irrelevance 추가
     ▼
Top 100 starred
Python/Java/JS
```

### Crowd-sourced

```
64,517 User Queries ──► Dedup (ROUGE-L + Embedding) ──► Exclude Public Sets ──► Human Edit ──► 2,251
                              │                              │                      │
                              ▼                              ▼                      ▼
                        Similarity > 0.8              Nexus 등 필터링        최소 편집 원칙
```

### Multi-turn

```
Custom API Codebase ──► Dependency Graph ──► Task Generation ──► Human Labeling ──► Validation
     │                       │                    │                  │                │
     │                       ▼                    ▼                  ▼                ▼
     │                  Function DAG         GPT-4o +            Ground truth      State +
     ▼                                      Persona Hub          trajectories     Response check
8 domains:
Vehicle, Trading,
Travel, File System,
Messaging, Twitter,
Ticket, Math
```

---

## Evaluation Metrics

### Primary Metrics

| Metric | Formula | Use Case |
|--------|---------|----------|
| **AST Accuracy** | Correct / Total | Single-turn |
| **Execution Accuracy** | Correct Execution / Total | Single-turn (검증) |
| **State Match** | Final State == GT State | Multi-turn |
| **Response Match** | Calls ⊇ Min Required | Multi-turn |
| **Exact Match** | Answer == GT Answer | Agentic |

### Contamination Detection

```
Contamination Score = PPL(Single-turn) / PPL(Crowd-sourced)

If score << 1: Model likely contaminated on Single-turn
If score ≈ 1: Good generalization
If score >> 1: Struggles with real-world complexity
```

---

## Comparison with Other Benchmarks

| Benchmark | Stateful | Conversational | Deterministic | Real User Data |
|-----------|:--------:|:--------------:|:-------------:|:--------------:|
| **BFCL** | Partial | Multi-turn | ✓ (AST) | ✓ (Crowd-sourced) |
| ToolSandbox | ✓ | ✓ (User Sim) | ✗ (Milestone) | ✗ |
| ToolBeHonest | ✗ | ✗ | ✓ (Multi-level) | ✗ |
| τ-bench | ✓ | ✓ | ✗ | ✗ |
| ToolBench | ✗ | ✗ | ✗ (LLM eval) | ✗ |

---

## Source References

- **Evaluation Design**: [Section 4](../bfcl.md#4-evaluation-methodology)
- **Data Curation**: [Appendix E](../bfcl.md#e-data-generation-pipeline-implementation-details)
- **AST Rules**: [Appendix H](../bfcl.md#h-bfcl-ast-substring-matching)
- **Error Analysis**: [Section 5.4](../bfcl.md#54-multi-turn-error-analysis)
