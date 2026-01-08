---
parent: ../2408.04682-analysis.md
source: ../2408.04682.md
---

# Methodology - Deep Analysis

## Paper Type: Benchmark

ToolSandbox는 Tool-use LLM을 **stateful, conversational, interactive** 환경에서 평가하는 벤치마크다.

---

## Approach Overview

### Design Philosophy

```
                    ToolSandbox Design Space
    ┌─────────────────────────────────────────────────┐
    │                                                 │
    │   State: Stateless ─────────────► Stateful     │
    │                                    (44% tools) │
    │                                                 │
    │   Eval: Off-policy ─────────────► On-policy   │
    │                                    (User Sim)  │
    │                                                 │
    │   Trajectory: Fixed ────────────► Dynamic      │
    │                                    (Milestone) │
    │                                                 │
    └─────────────────────────────────────────────────┘
```

### Three-Pillar Structure

| Pillar | Feature | Implementation |
|--------|---------|----------------|
| **Stateful** | World state dependency | Execution Context + 15/34 stateful tools |
| **Conversational** | Multi-turn interaction | LLM User Simulator |
| **Interactive** | Dynamic evaluation | Milestone/Minefield DAG |

---

## Key Design Choices

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| **Tool Execution** | Sandboxed Python | 안전한 실행 + 실제 동작 | [Section 2.1](../2408.04682.md#21-stateful-tool-execution) |
| **User Simulation** | GPT-4o + Knowledge Boundary | Hallucination 최소화 | [Section 2.2](../2408.04682.md#22-llm-user-simulator) |
| **Evaluation** | Milestone Similarity | Trajectory-agnostic | [Section 2.3](../2408.04682.md#23-milestones-and-minefields) |
| **Scoring** | 0-1 Similarity | 연속적 성능 측정 | [Section 2.3](../2408.04682.md#23-milestones-and-minefields) |
| **Domains** | 11 (Contact, Message, ...) | 실제 사용 시나리오 | [Section 3](../2408.04682.md#3-test-scenarios) |

---

## Technical Details

### Execution Context Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Execution Context                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │   World State   │  │   Databases     │                   │
│  │                 │  │                 │                   │
│  │  cellular: bool │  │  contacts: []   │                   │
│  │  wifi: bool     │  │  messages: []   │                   │
│  │  location: bool │  │  reminders: []  │                   │
│  │  low_battery:   │  │  settings: {}   │                   │
│  │       bool      │  │                 │                   │
│  └────────┬────────┘  └────────┬────────┘                   │
│           │                    │                             │
│           └────────┬───────────┘                             │
│                    │                                         │
│           ┌────────▼────────┐                               │
│           │  Tool Executor  │                               │
│           │                 │                               │
│           │  44% stateful   │                               │
│           │  56% stateless  │                               │
│           └─────────────────┘                               │
└─────────────────────────────────────────────────────────────┘
```

### State Dependency Graph

```
                        ┌──────────────────┐
                        │ low_battery_mode │
                        │      = False     │
                        └────────┬─────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
            ┌───────▼───────┐         ┌───────▼───────┐
            │   cellular    │         │     wifi      │
            │   = True      │         │   = True      │
            └───────┬───────┘         └───────┬───────┘
                    │                         │
        ┌───────────┴──────┐     ┌───────────┴──────┐
        │                  │     │                  │
┌───────▼──────┐   ┌───────▼─────▼──────┐   ┌──────▼───────┐
│ send_message │   │   get_location     │   │  web_search  │
└──────────────┘   └────────────────────┘   └──────────────┘
```

### User Simulator Pipeline

```
User Goal ──► Knowledge Boundary ──► Demonstration ──► GPT-4o ──► User Response
    │               │                     │                           │
    ▼               ▼                     ▼                           ▼
"Send msg      "You know Alice's     "User: Hi!\n         "Can you send
to Alice"       number, but NOT       Agent: ..."           a message to
                Bob's number"                                 Alice?"
```

**Knowledge Boundary Categories**:
- **Must Know**: 태스크 완료에 필요한 정보 (e.g., recipient name)
- **Must Not Know**: Agent가 물어봐야 하는 정보 (e.g., message content)
- **Cannot Know**: 환경에 존재하지 않는 정보 (e.g., non-existent contact)

### Milestone/Minefield Evaluation

```
Test Scenario
     │
     ├──► Milestones (DAG)          ├──► Minefields (Set)
     │                               │
     │    M1: get_contact(Alice)     │    ✗ timestamp_diff()
     │         │                     │    ✗ send_message(unknown)
     │         ▼                     │
     │    M2: cellular_on()          │
     │         │                     │
     │         ▼                     │
     │    M3: send_message(...)      │
     │                               │
     └───────────┬───────────────────┘
                 │
                 ▼
         Similarity Score
         = achieved/total × (1 - minefield_hit)
```

---

## Evaluation Metrics

### Primary Metrics

| Metric | Formula | Description |
|--------|---------|-------------|
| **Milestone Similarity** | achieved / total | DAG에서 달성한 milestone 비율 |
| **Minefield Penalty** | 0 if hit, 1 otherwise | Hallucination 패널티 |
| **Final Score** | similarity × penalty | 최종 점수 (0-1) |
| **Turn Count** | # of conversation turns | 효율성 측정 |

### Category-specific Analysis

| Category | Focus | Challenge |
|----------|-------|-----------|
| Simple | 단일 도구 사용 | 기본 function calling |
| Multi-step | 순차적 도구 사용 | Planning 능력 |
| State Dependency | Implicit dependency | State 추론 |
| Canonicalization | 표현 변환 | 자연어 → 구조화 |
| Insufficient Info | Unsolvable 인식 | Hallucination 억제 |
| Sandbox Message | 에러 처리 | Recovery 능력 |

---

## Data Curation Pipeline

### Test Scenario Generation

```
Domain Selection    Expert Design    Scenario Synthesis    Validation
      │                  │                  │                  │
      ▼                  ▼                  ▼                  ▼
11 domains         Milestone DAGs     Complexity          Manual
(Contact,          Minefields         variations          review
 Message, ...)     Knowledge          (Easy/Medium/       + Fix
                   Boundaries         Hard)
```

### Tool Design

```
34 Tools
    │
    ├── Stateful (15, 44%)
    │   ├── send_message (cellular dependency)
    │   ├── get_location (location service dependency)
    │   ├── make_call (cellular dependency)
    │   └── ...
    │
    └── Stateless (19, 56%)
        ├── get_contact
        ├── calculate
        ├── convert_unit
        └── ...
```

---

## Comparison with Other Benchmarks

| Benchmark | Stateful | On-policy | Conversational | Milestone | Real Tools |
|-----------|:--------:|:---------:|:--------------:|:---------:|:----------:|
| **ToolSandbox** | ✓ | ✓ (User Sim) | ✓ | ✓ | ✓ |
| BFCL | Partial | ✗ | Multi-turn | ✗ | ✓ |
| ToolBeHonest | ✗ | ✗ | ✗ | ✗ | ✓ |
| τ-bench | ✓ | ✓ | ✓ | ✗ | ✓ |
| ToolBench | ✗ | ✗ | ✗ | ✗ | ✗ |
| API-Bank | ✗ | ✗ | ✓ | ✗ | ✓ |

---

## Experimental Setup

### Models Evaluated

| Type | Models |
|------|--------|
| **Proprietary** | GPT-4o, GPT-4, GPT-3.5-Turbo, Claude-3-Opus/Sonnet/Haiku, Gemini-1.5-Pro |
| **Open-source** | Llama-3-70B, Mixtral-8x22B, Hermes-2-Pro, Gorilla, ToolACE |

### Configuration

- **User Simulator**: GPT-4o with Knowledge Boundary + Demonstration
- **Max Turns**: 20 per scenario
- **Temperature**: 0.0 for reproducibility
- **Parallel Calls**: Allowed (model default)

---

## Source References

- **Benchmark Design**: [Section 2](../2408.04682.md#2-benchmark-design)
- **Test Scenarios**: [Section 3](../2408.04682.md#3-test-scenarios)
- **Main Results**: [Section 4](../2408.04682.md#4-experimental-results)
- **Implementation**: [Appendix A](../2408.04682.md#a-implementation-details)
