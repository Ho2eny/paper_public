---
parent: "../2404.09992-analysis.md"
source: "../2404.09992.md"
type: detail
created: 2026-01-09
---

# Core Contributions Deep Analysis

[Back to Main Analysis](../2404.09992-analysis.md)

---

## Contribution 1: MMInA Benchmark

### What It Is
1,050개의 멀티홉 멀티모달 태스크를 14개 실제 웹사이트에서 평가하는 벤치마크.

Source: [Section 1 - Contributions](../2404.09992.md#1-introduction)

### Technical Specification

| Aspect | Specification |
|--------|---------------|
| Total Tasks | 1,050 |
| Total Hops | 2,989 |
| Websites | 14 evolving real-world |
| Avg Hops/Task | 2.85 |
| Max Hops | 10 |
| Avg Actions/Task | 12.9 |

### Environment Formulation

웹 브라우징을 **Partially Observable MDP** $\langle S, A, P, R \rangle$로 정의:

- **State Space $S$**: 전체 인터넷 콘텐츠 + 브라우저 상태
- **Observation $o_t \in \Omega$**: Accessibility tree + 스크린샷 + 이미지 + 히스토리
- **Action Space $A$**: 12개 표준화된 액션 (click, scroll, type 등)
- **Reward $R$**: 각 hop별 PASS/FAIL

Source: [Section 3.1](../2404.09992.md#31-environment)

### 기존 벤치마크와의 차별점

| Benchmark | Max Hops | Avg Hops | Website Type | Multimodal |
|-----------|----------|----------|--------------|------------|
| MiniWoB++ | 1 | 1.00 | Static simplified | O |
| WebArena | 2 | 1.06 | Static real-world | X |
| VisualWebArena | 2 | 1.05 | Static real-world | O |
| WebVoyager | 4 | 2.40 | Dynamic real-world | O |
| **MMInA** | **10** | **2.85** | **Evolving real-world** | O |

Source: [Table 1](../2404.09992.md#table-1)

### Innovation Points
1. **Evolving Environment**: 정적 스냅샷이 아닌 실시간 변화하는 웹
2. **Cross-website Navigation**: 단일 사이트가 아닌 다중 사이트 간 이동
3. **Compositional Tasks**: 여러 하위 태스크의 순차적 조합

---

## Contribution 2: Holistic Evaluation Protocol

### Problem with Existing Metrics
기존 Task Success Rate만으로는:
- 멀티홉 태스크에서 거의 0%에 가까운 성공률
- 에이전트의 부분적 성공/진행도 파악 불가
- 실패 원인 분석 불가능

### Proposed Solution: Hop-level Evaluation

**핵심 아이디어**: 각 hop을 독립적으로 평가

```
Task Completion Queue: [hop_1, hop_2, ..., hop_N, END]
```

- 각 hop 완료 조건: 필요 정보 획득 OR 목표 URL 도달
- 순차적 완료 강제: hop_k 완료 후에만 hop_{k+1} 진행 가능

Source: [Section 3.5](../2404.09992.md#35-evaluation)

### Single-hop Evaluation Methods

1. **must_include**: 필수 키워드 포함 여부 (엄격)
2. **fuzzy_match**: GPT-3.5로 의미적 유사성 평가 (유연)

### Why This Matters

| Metric | 1-hop | 2-4 hops | 5+ hops | Overall |
|--------|-------|----------|---------|---------|
| GPT-4V Hop SR | 42.91 | 21.23 | 3.99 | 13.89 |
| GPT-4V Task SR | 42.91 | 3.03 | 0 | 21.77 |

- **Hop SR로 granular insights 확보**: 어느 단계에서 실패하는지 파악
- **Task SR 0%도 의미있는 분석 가능**: 초기 hop에서의 성공률 확인

Source: [Table 2](../2404.09992.md#42-main-results)

---

## Contribution 3: Memory-Augmented Agent

### Motivation: Why Agents Fail

**문제 1: 확장된 탐색 공간**
- 멀티홉 프롬프트에 여러 웹사이트 포함
- 실패 시 올바른 사이트 재시도 대신 다른 사이트로 전환
- 결과: 과도한 탐색, 태스크 목표 상실

**문제 2: Early Hop Failure Pattern**
- 총 hop 수가 많을수록 첫 번째 hop 성공률 감소
- 2-hop 태스크: GPT-4V 1st hop SR 56.5%
- 6-hop 태스크: GPT-4V 1st hop SR 16.67%

Source: [Section 4.3](../2404.09992.md#43-why-are-multihop-web-tasks-challenging), [Table 3](../2404.09992.md#table-3)

### Three Memory Systems

```
                    ┌─────────────────┐
                    │   Web Agent     │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   Semantic    │   │   Episodic    │   │  Procedural   │
│    Memory     │   │    Memory     │   │    Memory     │
├───────────────┤   ├───────────────┤   ├───────────────┤
│ World         │   │ Step-by-step  │   │ Completed     │
│ knowledge     │   │ action        │   │ task          │
│ (LLM weights) │   │ trajectories  │   │ strategies    │
└───────────────┘   └───────────────┘   └───────────────┘
```

Source: [Section 4.4](../2404.09992.md#44-memory-augmented-agents), [Figure 4](../2404.09992.md#figure-4)

### Implementation Details

**Procedural Memory 구현**:
- 마지막 K개 태스크의 액션 궤적을 프롬프트에 추가
- 각 궤적: task description + web content observations

**Optimal K Value**:
- 실험 결과 K=1 또는 K=2가 최적
- K 증가 시 diminishing returns + bias 도입

Source: [Figure 5](../2404.09992.md#figure-5), [Section A.3](../2404.09992.md#a3-supplementary-results)

### Performance Improvement

| Agent | Setup | Hop SR | Task SR |
|-------|-------|--------|---------|
| Gemini-Pro-Vision | Base | 10.66 | 18.40 |
| Gemini-Pro-Vision | +Memory | 14.27 | 20.13 |

- **Model-agnostic**: 어떤 LMM/LLM에도 적용 가능
- **Lightweight**: 프롬프트 augmentation만으로 구현

---

## Contribution Impact Summary

| Contribution | Type | Novelty | Impact |
|--------------|------|---------|--------|
| MMInA Benchmark | Dataset/Benchmark | High | 멀티홉 웹 에이전트 평가의 새 표준 |
| Holistic Evaluation | Metric | Medium | 장거리 추론 분석 가능하게 함 |
| Memory Augmentation | Method | Medium | 경량화된 성능 향상 기법 제공 |

[Back to Main Analysis](../2404.09992-analysis.md) | [Key Sections Guide](key-sections.md)
