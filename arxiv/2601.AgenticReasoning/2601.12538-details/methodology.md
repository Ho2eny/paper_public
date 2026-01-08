```yaml
---
parent: ../2601.12538-analysis.md
source: ../2601.12538.md
created: 2026-02-06
---
```

# Methodology - Deep Analysis

## Paper Type

**Type**: Survey

---

## Approach Overview

### High-Level Summary
이 논문은 "추론 중심(reasoning-centric)" 관점에서 LLM 기반 에이전트 시스템을 체계적으로 정리하는 서베이이다. 기존의 LLM 추론 서베이(내부 추론에 집중)와 에이전트 서베이(시스템 아키텍처 중심)를 연결하는 교차점에 위치한다.

핵심 방법론은 에이전트 추론을 환경 복잡도에 따른 3개 계층(Foundational → Self-Evolving → Collective)으로 구조화하고, 각 계층을 2개 최적화 모드(In-context vs Post-training)로 분석하는 이중 축 분류 체계이다. 이 프레임워크를 바탕으로 797개 참고문헌을 분류하고, 6개 응용 분야와 벤치마크를 맵핑한다.

차별점은 "추론"을 에이전트 지능의 핵심 조직 원리로 재정의하는 관점이다. 계획, 도구 사용, 탐색, 피드백, 메모리, 협업을 개별 모듈이 아닌, 하나의 추론 프로세스의 서로 다른 측면으로 바라본다.

### Key Innovation
POMDP 기반 수학적 형식화를 통해 에이전트 정책을 내부 추론($\pi_{\text{reason}}$)과 외부 행동($\pi_{\text{act}}$)으로 분해하는 통합 프레임워크. 이를 통해 단일 에이전트(POMDP), 멀티에이전트(Dec-POMDP), 자기진화(메타학습 루프)를 하나의 체계로 통합.

**Source**: [Section 1](../2601.12538.md#1-introduction), [Section 2.2](../2601.12538.md#22-preliminaries)

---

## Technical Details

### Survey Scope

| Aspect | Value | Source |
|--------|-------|--------|
| **Time Range** | ~2025년 (2026-01-18 arXiv 제출) | [Survey Scope](../2601.12538.md#survey-scope) |
| **Sources** | arXiv, 주요 ML/NLP 학회 (NeurIPS, ICML, ICLR, ACL, EMNLP 등) | References 분석 |
| **Paper Count** | 797개 참고문헌 | References 섹션 |
| **Selection Criteria** | 추론을 적응적 행동의 핵심으로 다루는 에이전트 시스템 | [Survey Scope](../2601.12538.md#survey-scope) |

### Taxonomy Design

**Primary Axis — 환경 복잡도에 따른 3계층**:

```
Foundational Agentic Reasoning
├── Planning Reasoning (Sec 3.1)
│   ├── In-context: Workflow, Tree Search, Formalization, Decomposition, External Aid
│   └── Post-training: Reward Design / Optimal Control
├── Tool-Use Optimization (Sec 3.2)
│   ├── In-Context Tool-integration
│   ├── Post-training Tool-integration (SFT → RL)
│   └── Orchestration-based Tool-integration
└── Agentic Search (Sec 3.3)
    ├── In-Context Search
    ├── Post-Training Search (SFT → RL)
    └── Structure-Enhanced Search

Self-Evolving Agentic Reasoning
├── Feedback Mechanisms (Sec 4.1)
│   ├── Reflective Feedback
│   ├── Parametric Adaptation
│   └── Validator-Driven Feedback
├── Agentic Memory (Sec 4.2)
│   ├── Flat Memory (Factual + Experience)
│   ├── Structured Memory (Graph + Multimodal)
│   └── Post-training Memory Control
└── Evolving Capabilities (Sec 4.3)
    ├── Self-evolving Planning
    ├── Self-evolving Tool-use
    └── Self-evolving Search

Collective Multi-Agent Reasoning
├── Role Taxonomy (Sec 5.1)
│   ├── Generic Roles (Leader/Worker/Critic)
│   └── Domain-Specific Roles
├── Collaboration (Sec 5.2)
│   ├── In-context Collaboration
│   └── Post-training Collaboration
├── Multi-Agent Evolution (Sec 5.3)
└── Memory Management (Sec 5.3.3)
```

**Secondary Axis — 최적화 모드**:

| Mode | Description | Examples |
|------|-------------|----------|
| **In-context** | 파라미터 고정, 추론 시간 최적화 | ReAct, ToT, MCTS, Reflexion |
| **Post-training** | RL/SFT로 파라미터 업데이트 | DeepSeek-R1, Search-R1, ToolRL, GRPO |

**Source**: [Survey Structure](../2601.12538.md#survey-structure)

### Coverage Analysis
- [x] 계획 (Planning): 6가지 스타일, 22개 대표 시스템
- [x] 도구 사용 (Tool Use): 3가지 단계, 18개 대표 시스템
- [x] 탐색 (Search): 3가지 아키텍처, 16개 대표 시스템
- [x] 피드백 (Feedback): 3가지 메커니즘, 22개 대표 시스템
- [x] 메모리 (Memory): 3가지 유형, 30개 대표 시스템
- [x] 멀티에이전트 (Multi-Agent): 역할/협업/진화/메모리
- [x] 응용: 수학/코드, 과학, 로보틱스, 의료, 웹/GUI, 자율 연구
- [x] 벤치마크: 메커니즘 수준 + 응용 수준
- [ ] 상업 시스템 (Claude, GPT-4 에이전트 모드 등) - 학술 논문 위주
- [ ] 안전성/정렬 (Safety/Alignment) - Section 8.6에서 간략히만 다룸
- [ ] 비용/효율성 분석 - 메모리/계산 비용 관점의 체계적 비교 부재

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| 분류 축 | 3계층 × 2모드 | 환경 복잡도 + 최적화 시점을 동시에 포착 | 도메인별 분류, 역량별 분류 | [Section 1](../2601.12538.md#1-introduction) |
| 형식화 | POMDP + 정책 분해 | 부분 관측성과 think-act 구조를 자연스럽게 표현 | MDP, 게임 이론 | [Section 2.2](../2601.12538.md#22-preliminaries) |
| 시간 범위 | ~2025년 | 에이전트 AI의 급성장기를 포괄 | 더 좁은 범위(2024-2025) | [Survey Scope](../2601.12538.md#survey-scope) |
| 추론 중심 관점 | 추론을 조직 원리로 | 기존 아키텍처 중심 서베이와 차별화 | 시스템 중심, 응용 중심 | [Section 2.1](../2601.12538.md#21-positioning-our-survey) |

---

## Assumptions & Limitations

### Assumptions
1. **추론이 에이전트의 핵심**: 추론이 인식/계획/결정/검증을 연결하는 조직 원리라는 전제. 일부 순수 RL 에이전트(명시적 추론 없이 정책 최적화)와는 관점 차이 존재 - [Section 1](../2601.12538.md#1-introduction)
2. **3계층의 누적성**: Collective가 Self-Evolving 위에, Self-Evolving이 Foundational 위에 구축된다는 가정. 실제로는 기초 역량 없이도 멀티에이전트 시스템이 작동할 수 있음 - [Section 1](../2601.12538.md#1-introduction)

### Limitations
1. **실험적 검증 부재**: 분류 체계의 유효성을 정량적으로 검증하지 않음. 분류 경계가 모호한 경우(예: 피드백과 메모리의 중첩) 주관적 판단에 의존
2. **상업 시스템 누락**: Claude Code, Cursor, Gemini 에이전트 등 실제 배포된 시스템의 분석이 제한적
3. **정량적 비교 부재**: "어떤 접근이 효과적인가"보다 "어떤 접근이 존재하는가"에 초점

### Scope
- **In Scope**: 추론을 핵심으로 하는 LLM 기반 에이전트 시스템의 방법론, 응용, 벤치마크
- **Out of Scope**: 순수 RL(LLM 미사용), 비에이전트 LLM 추론(정적 CoT만), 하드웨어/인프라, 비용 분석

---

## Complexity Analysis

| Aspect | Value | Notes |
|--------|-------|-------|
| **커버리지** | 797개 참고문헌 | 2025년 에이전트 AI의 사실상 모든 주요 연구 포함 |
| **분류 깊이** | 3단계 hierarchy | Section → Sub-section → Method Style |
| **테이블** | 6개 대표 시스템 비교 테이블 | 각 핵심 영역별 1개 |
| **응용 범위** | 6개 도메인 | 수학/코드, 과학, 로보틱스, 의료, 웹/GUI, 자율 연구 |
| **벤치마크** | 2단계 (메커니즘 + 응용) | ~50개 벤치마크 소개 |

---

## Survey Quality Assessment

### Strengths of the Methodology
1. **일관된 분석 프레임워크**: 모든 영역을 In-context vs Post-training 축으로 분석하여 일관성 유지
2. **대표 시스템 테이블**: 각 영역별 Structure/Format/Tool 등의 차원으로 비교하여 빠른 파악 가능
3. **형식화와 직관의 균형**: POMDP 형식화로 엄밀성 확보하면서, 직관적 설명도 병행

### Weaknesses of the Methodology
1. **선택 기준 불명확**: 797개 참고문헌의 선택 기준(검색 전략, 포함/배제 기준)이 명시되지 않음
2. **정량적 메타분석 부재**: 방법론 간 성능 비교, 트렌드의 정량적 분석 없음
3. **참고문헌 편향**: UIUC 연구 그룹의 논문이 과다 포함(ref 152-180 등 자체 인용)

---

## Source References Summary

| Topic | Section | Link |
|-------|---------|------|
| Framework Definition | Section 1 + 2 | [Link](../2601.12538.md#1-introduction) |
| POMDP Formalization | Section 2.2 | [Link](../2601.12538.md#22-preliminaries) |
| Foundational Methods | Section 3 | [Link](../2601.12538.md#3-foundational-agentic-reasoning) |
| Self-Evolving Methods | Section 4 | [Link](../2601.12538.md#4-self-evolving-agentic-reasoning) |
| Multi-Agent Methods | Section 5 | [Link](../2601.12538.md#5-collective-multi-agent-reasoning) |
| Applications | Section 6 | [Link](../2601.12538.md#6-applications) |
| Benchmarks | Section 7 | [Link](../2601.12538.md#7-benchmarks) |
| Open Problems | Section 8 | [Link](../2601.12538.md#8-open-problems) |

---

## Navigation

← [Back to Main Analysis](../2601.12538-analysis.md)
