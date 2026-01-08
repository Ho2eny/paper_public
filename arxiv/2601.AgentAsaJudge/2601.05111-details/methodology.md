---
parent: ../2601.05111-analysis.md
source: ../2601.05111.md
---

# Methodology - Deep Analysis

## Paper Type: Survey

이 논문은 **Agent-as-a-Judge**라는 신생 분야에 대한 첫 번째 포괄적 서베이로, 분류 체계(taxonomy) 제안과 기존 연구의 체계적 정리를 목표로 한다.

---

## Survey Methodology

### Scope Definition

| Dimension | Value |
|-----------|-------|
| **주제** | AI 평가 시스템 (LLM-as-a-Judge → Agent-as-a-Judge) |
| **시간 범위** | 2023-2026 (주로 2024-2025 연구) |
| **논문 수** | 66개 참고문헌 |
| **언어** | 영어 |

### Literature Collection

논문에서 명시적으로 언급하지 않았지만, 참고문헌 분석을 통해 추론한 수집 방법:

1. **Seed Papers**: MT-Bench [[1]](#ref-1), G-Eval [[59]](#ref-59), ChatEval [[10]](#ref-10) 등 대표 연구에서 출발
2. **Forward/Backward Citation**: 인용 관계를 통한 확장
3. **Keyword Search**: "LLM judge", "agent evaluation", "multi-agent assessment" 등
4. **Venue-based**: ACL, EMNLP, NeurIPS, ICLR 등 주요 학회 proceedings

### Classification Framework

**2차원 분류 체계**:

```
                    ┌─────────────────────────────────────────┐
                    │         Methodologies (5개)             │
                    │  - Multi-Agent Collaboration           │
                    │  - Planning                            │
                    │  - Tool Integration                    │
                    │  - Memory & Personalization            │
                    │  - Optimization Paradigms              │
                    └─────────────────────────────────────────┘
                                      ×
                    ┌─────────────────────────────────────────┐
                    │         Applications (8개)              │
                    │  General: Math/Code, Fact-Checking,    │
                    │           Conversation, Multimodal     │
                    │  Professional: Medicine, Law,          │
                    │                Finance, Education      │
                    └─────────────────────────────────────────┘
```

---

## Developmental Taxonomy

### 설계 원칙

저자들은 "agent"라는 용어가 느슨하게 사용되는 현실을 인정하고, **자율성(autonomy)과 적응성(adaptability)** 수준에 따라 세 단계를 정의한다.

### Stage 1: Procedural

**정의**: 사전 정의된 워크플로우 또는 고정된 하위 에이전트 간 구조화된 토론

**특성**:
- 단일 추론을 에이전트 기반 워크플로우로 분해
- 사전 결정된 규칙에 의해 제약
- 새로운 평가 시나리오에 적응 불가

**대표 시스템**:
- ChatEval [[10]](#ref-10): 법정 스타일 토론 메커니즘
- CAFES [[57]](#ref-57): Evidence Gathering → Reasoning → Scoring 순차 구조
- GEMA-Score [[24]](#ref-24): 의료 보고서 평가를 위한 에이전트 협업

**Source**: [Section 2.3](../2601.05111.md#23-agent-as-a-judge)

### Stage 2: Reactive

**정의**: 중간 피드백에 기반한 실행 경로 라우팅 및 도구/하위 에이전트 동적 호출

**특성**:
- 조건부 라우팅을 통한 적응적 의사결정
- 고정된 결정 공간 내에서만 반응
- 기저 루브릭 개선 자율성 부재

**대표 시스템**:
- Evaluation Agent [[28]](#ref-28): 동적 다중 라운드 계획, 자율 종료
- AGENT-X [[45]](#ref-45): Adaptive Router로 관련 베이스 에이전트 동적 선택
- Agentic Reward Modeling [[8]](#ref-8): 도구 기반 검증 신호 통합

**Source**: [Section 2.3](../2601.05111.md#23-agent-as-a-judge)

### Stage 3: Self-Evolving

**정의**: 운영 중 내부 구성요소 개선 - 루브릭 즉석 합성, 학습된 교훈으로 메모리 업데이트

**특성**:
- 높은 자율성
- 자기 수정 능력
- 적응적 평가 시스템의 새 지평
- 안정성 보장이 과제

**대표 시스템**:
- EvalAgents [[53]](#ref-53): Query Generator로 웹 검색을 통해 암묵적 루브릭 발견
- OnlineRubrics [[55]](#ref-55): 강화학습과 루브릭 진화 통합
- RLPA [[11]](#ref-11): 지속적 사용자 페르소나 구축 및 유지

**Source**: [Section 2.3](../2601.05111.md#23-agent-as-a-judge)

---

## Methodology Dimensions Analysis

### Dimension 1: Multi-Agent Collaboration

**목적**: 단일 LLM 편향을 집단 추론으로 완화

| Topology | 접근법 | 특성 | 발전 단계 |
|----------|--------|------|----------|
| **Collective Consensus** | 수평적 토론 | 동등한 에이전트들의 토론 | Procedural → Self-Evolving |
| **Task Decomposition** | 분할 정복 | 하위 작업을 전문 에이전트에 위임 | Procedural → Reactive |

**Key Design Choices**:

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| 토론 vs 분해 | 작업 특성에 따라 선택 | 주관적 평가 → 토론, 다차원 평가 → 분해 | [3.1](../2601.05111.md#31-multi-agent-collaboration) |
| 에이전트 역할 | 명시적 역할 부여 | 맹목적 다수 추종 방지 | [[64]](#ref-64) |
| 동적 전문가 생성 | Self-Evolving 단계 | 정적 앙상블 한계 극복 | [[13]](#ref-13) |

### Dimension 2: Planning

**목적**: 평가 목표를 실행 가능한 하위 작업으로 분해, 평가 궤적 동적 조정

| Sub-category | 역할 | 발전 단계 |
|--------------|------|----------|
| **Workflow Orchestration** | 평가 "방법" 계획 | Procedural, Reactive |
| **Rubric Discovery** | 평가 "대상" 계획 | Self-Evolving |

**Key Design Choices**:

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| 정적 vs 동적 워크플로우 | 복잡도에 따라 선택 | 단순 → 정적, 복잡 → 동적 | [3.2](../2601.05111.md#32-planning) |
| 자율 종료 | 정보 이득 모니터링 | 효율성 최적화 | [[28]](#ref-28) |
| 루브릭 발견 | 웹 검색, 컨텍스트 분석 | 암묵적 평가 기준 발굴 | [[53]](#ref-53), [[54]](#ref-54) |

### Dimension 3: Tool Integration

**목적**: 외부 증거와 명시적 검사로 평가 근거 마련

| Purpose | 도구 유형 | 대표 시스템 |
|---------|----------|-------------|
| **Evidence Collection** | 코드 실행기, 시각 모델, 문서 검색 | Agent-as-a-Judge, Evaluation Agent |
| **Correctness Verification** | 정리 증명기, 검색 엔진, Python 인터프리터 | HERMES, VerifiAgent, Agentic RM |

**Key Design Choices**:

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| 증거 수집 vs 검증 | 작업 유형에 따라 | 생성 작업 → 증거, 추론 작업 → 검증 | [Table 1](../2601.05111.md#table-1) |
| 도구 선택 | 도메인 특화 | 코드 → 실행기, 수학 → 증명기, 사실 → 검색 | [3.3](../2601.05111.md#33-tool-integration) |

### Dimension 4: Memory and Personalization

**목적**: 평가 단계 간 정보 유지, 다단계 추론/일관된 판단/재사용 지원

| Role | 기능 | 발전 단계 |
|------|------|----------|
| **Intermediate State** | 중간 평가 상태 유지 | Reactive 핵심 enabler |
| **Personalized Context** | 사용자 선호/기준/피드백 유지 | Self-Evolving 전제조건 |

**Key Design Choices**:

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| 상태 vs 컨텍스트 | 사용 사례에 따라 | 단일 세션 → 상태, 장기 → 컨텍스트 | [3.4](../2601.05111.md#34-memory-and-personalization) |
| 페르소나 구축 | 선호 데이터에서 추상화 | 일관된 개인화 평가 | [[11]](#ref-11), [[46]](#ref-46) |

### Dimension 5: Optimization Paradigms

**목적**: 파라미터 업데이트 또는 평가 행동 조정으로 평가 품질 향상

| Paradigm | 방법 | 특성 |
|----------|------|------|
| **Training-Time** | SFT, RL | 파라미터 업데이트, 에이전트 행동 학습 |
| **Inference-Time** | 프롬프트, 워크플로우, 에이전트 상호작용 | 고정된 백본, 유연한 제어 |

**Key Design Choices**:

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| 학습 vs 추론 | 리소스와 목표에 따라 | 충분한 데이터/계산 → 학습, 그렇지 않으면 → 추론 | [3.5](../2601.05111.md#35-optimization-paradigms) |
| RL for tool use | 도구 호출 학습 | 프롬프트만으로는 복잡한 도구 사용 어려움 | [[47]](#ref-47), [[12]](#ref-12) |

---

## Survey Quality Assessment

### Strengths

1. **최초성**: Agent-as-a-Judge 분야의 첫 포괄적 서베이
2. **체계적 분류**: 5×8 매트릭스로 방법론과 응용 교차 분석
3. **발전적 관점**: 단계별 taxonomy로 진화 궤적 제시
4. **균형잡힌 coverage**: 일반/전문 도메인 모두 포함

### Limitations

1. **정량적 비교 부재**: 방법론 간 성능 비교 데이터 없음
2. **재현성**: 일부 참고문헌이 프리프린트 상태
3. **실증적 검증 부족**: 제안된 taxonomy의 실제 유용성 미검증
4. **선택 기준 불명확**: 66개 논문 선택 기준 미제시

---

## Summary

이 서베이는 **분류(classification)**와 **종합(synthesis)**을 통해 Agent-as-a-Judge 분야를 체계화한다:

1. **분류**: Procedural/Reactive/Self-Evolving 단계 + 5가지 방법론 차원
2. **종합**: 66개 연구를 8개 응용 도메인에 걸쳐 정리
3. **전망**: 4가지 미래 연구 방향과 "진정한 자율성" 비전 제시

연구 방법론 자체보다는 **분류 체계의 유용성**과 **coverage의 포괄성**이 이 서베이의 주요 기여이다.
