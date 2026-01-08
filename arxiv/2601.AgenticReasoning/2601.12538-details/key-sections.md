```yaml
---
parent: ../2601.12538-analysis.md
source: ../2601.12538.md
created: 2026-02-06
---
```

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

| Metric | Value |
|--------|-------|
| **Total Reading Time** | ~4-5시간 (전체 135페이지) |
| **Essential Only** | ~90분 (핵심만) |
| **Total Sections** | 8 main sections + 다수 sub-sections |
| **Must Read** | 3 sections (Sec 1, 2.2, 3, 8) |

---

## ⭐⭐⭐ Must Read

### Section 1 + Definition: Introduction & Definition of Agentic Reasoning

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
논문의 핵심 프레임워크와 3계층 구조가 이 섹션에서 정의된다. 나머지 논문 전체가 이 정의를 바탕으로 구조화되므로, 이 섹션을 이해하지 못하면 전체 논문을 파악할 수 없다.

**Key Points**:
1. Agentic Reasoning의 정의: 계획-행동-학습하는 자율 추론 에이전트로서의 LLM
2. 3계층 구조: Foundational → Self-Evolving → Collective
3. 2가지 최적화 모드: In-context vs Post-training
4. 기존 LLM 추론/에이전트 서베이와의 차별점

**Reading Tips**:
- "Definition of Agentic Reasoning" 박스를 먼저 읽고 전체 구조 파악
- "Contributions" 리스트로 논문이 다루는 범위 확인
- Table of Contents로 전체 논문 구조 미리 파악

**Estimated Time**: ~15분

**Link**: [Section 1](../2601.12538.md#1-introduction)

---

### Section 2.2: Preliminaries (Formalization)

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
POMDP 형식화, 정책 분해 (Eq. 1), GRPO (Eq. 3-4), 멀티에이전트 확장, 자기진화 메타학습 루프가 모두 이 4페이지에 집약되어 있다. 논문의 수학적 기반이며, 이후 모든 분석의 이론적 뼈대.

**Key Points**:
1. $\pi_\theta(z_t, a_t | h_t) = \pi_{\text{reason}}(z_t | h_t) \cdot \pi_{\text{act}}(a_t | h_t, z_t)$ — 핵심 정책 분해
2. In-context reasoning: $z^* = \arg\max \hat{v}_\phi(h_t, z)$ — 추론 시간 탐색
3. GRPO: 그룹 상대 정책 최적화 — 추론 모델 학습의 표준
4. Dec-POMDP: 멀티에이전트 확장, 통신 채널 $\mathcal{C}$
5. Meta-update: $\mathcal{S}_{k+1} = \text{Update}(\mathcal{S}_k, F_k)$ — 자기진화 형식화

**Reading Tips**:
- 수학에 익숙하지 않다면 Eq. 1만 확실히 이해하면 충분
- GRPO는 DeepSeek-R1 논문과 함께 읽으면 이해 용이
- 각 형식화가 이후 어떤 섹션에 대응하는지 매핑하며 읽기

**Estimated Time**: ~20분

**Link**: [Section 2.2](../2601.12538.md#22-preliminaries)

---

### Section 3: Foundational Agentic Reasoning

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
에이전트 추론의 3대 핵심 역량(계획/도구/탐색)에 대한 가장 상세한 분류. Table 2-4의 대표 시스템 비교와 6가지 계획 스타일 분류는 에이전트 시스템 설계의 필수 참고 자료.

**Key Points**:
1. **계획 6가지 스타일**: Workflow, Tree Search, Formalization, Decomposition, External Aid, Reward Design
2. **도구 사용 3단계**: In-context (ReAct) → Post-training (Toolformer, ToolRL) → Orchestration (HuggingGPT)
3. **탐색 3유형**: In-context (ReAct+Search) → Post-training (Search-R1) → Structure-Enhanced (KG 기반)
4. **SFT → RL 전환**: 도구/탐색 모두에서 SFT로 부트스트래핑 후 RL로 마스터리 달성하는 패턴

**Reading Tips**:
- Table 2, 3, 4를 먼저 스캔하여 대표 시스템 파악
- 관심 있는 하위 분야(예: 도구 사용 RL)만 깊이 읽기
- Figure 2, 3, 4로 시각적 개요 먼저 확인

**Estimated Time**: ~30분 (핵심만), ~60분 (전체)

**Link**: [Section 3](../2601.12538.md#3-foundational-agentic-reasoning)

---

### Section 8: Open Problems

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
6가지 오픈 프로블럼은 현재 에이전트 연구의 가장 중요한 미해결 과제. 연구 제안서 작성이나 연구 방향 설정에 직접적으로 활용 가능.

**Key Points**:
1. **개인화**: 사용자를 환경의 일부로 모델링, 비정상적 목적 함수
2. **장기 추론**: 토큰→도구→스킬→메모리 수준의 크레딧 할당
3. **월드 모델**: 내부 시뮬레이션, 비정상 환경에서의 공진화
4. **멀티에이전트 학습**: 그룹 크레딧 할당, 토폴로지 적응
5. **잠재 추론**: 자연어 없는 내부 추론, 해석 가능성 과제
6. **거버넌스**: 모델/에이전트/생태계 3수준의 안전성

**Reading Tips**:
- 6가지 문제 모두 짧게(각 ~1페이지) 서술되어 빠르게 읽기 가능
- 자신의 연구와 가장 관련 있는 1-2개에 집중

**Estimated Time**: ~15분

**Link**: [Section 8](../2601.12538.md#8-open-problems)

---

## ⭐⭐ Important

### Section 4: Self-Evolving Agentic Reasoning

**Priority**: ⭐⭐ Important

**Why Read**:
피드백/메모리/자기진화 메커니즘은 에이전트를 "일회성"에서 "지속적 개선"으로 전환하는 핵심. 특히 RL 기반 메모리 제어(Memory-R1, MemAgent)는 2025년의 가장 활발한 연구 방향.

**When to Read**:
에이전트의 학습/적응 메커니즘에 관심이 있을 때. 메모리 시스템 설계나 자기 개선 에이전트 구축 시 필수.

**Key Points**:
1. 피드백 3가지: Reflective → Parametric → Validator-Driven (Table 5)
2. 메모리 3가지: Flat → Structured → Post-training Control (Table 6)
3. 자기진화 계획/도구/탐색: Section 4.3

**Link**: [Section 4](../2601.12538.md#4-self-evolving-agentic-reasoning)

---

### Section 5: Collective Multi-Agent Reasoning

**Priority**: ⭐⭐ Important

**Why Read**:
멀티에이전트의 역할 분류, 협업 메커니즘, 진화 방법을 체계적으로 정리. MARL/CTDE 기반 학습, ToM(Theory of Mind), 통신 토폴로지 최적화 등 최신 연구 포함.

**When to Read**:
멀티에이전트 시스템 설계나 에이전트 간 협업 메커니즘 연구 시 필수.

**Key Points**:
1. 역할: Generic (Leader/Worker/Critic) + Domain-Specific
2. In-context 협업: Debate, Voting, Communication topology
3. Post-training 협업: M-GRPO, MALT, CoMARL
4. 메모리 관리: G-Memory, Collaborative Memory, LegoMem

**Link**: [Section 5](../2601.12538.md#5-collective-multi-agent-reasoning)

---

## ⭐ Reference

### Section 6: Applications

**Priority**: ⭐ Reference Only

**Content Summary**:
6개 도메인(수학/코드, 과학, 로보틱스, 의료, 웹/GUI, 자율 연구)별 에이전트 시스템 리뷰. 각 도메인에서 기초→자기진화→집단 프레임워크를 적용.

**When Useful**:
- 특정 도메인의 에이전트 연구 현황 파악 시
- 도메인별 대표 시스템과 벤치마크 확인 시
- 자신의 연구가 어느 응용 분야에 해당하는지 포지셔닝 시

**Link**: [Section 6](../2601.12538.md#6-applications)

---

### Section 7: Benchmarks

**Priority**: ⭐ Reference Only

**Content Summary**:
메커니즘 수준(Tool Use, Search, Memory/Planning, Multi-Agent)과 응용 수준(Embodied, Scientific, Research, Medical, Web, Tool-Use) 벤치마크를 체계적으로 분류.

**When Useful**:
- 에이전트 평가 설계 시 적절한 벤치마크 선택
- 특정 역량(예: 도구 사용, 메모리)을 평가하는 벤치마크 탐색
- 새 벤치마크 설계 시 기존 벤치마크의 커버리지 갭 식별

**Link**: [Section 7](../2601.12538.md#7-benchmarks)

---

## Skip

### Section 2.1: Positioning Our Survey

**Priority**: Skip

**Why Skip**:
- 다른 서베이(Huang & Chang 2022, Wang et al. 2024 등)와의 비교에 할애
- 핵심 기술 내용이 아닌 포지셔닝 논의
- Introduction에서 이미 핵심 차별점이 서술됨

**Alternative**:
Introduction의 마지막 단락에서 차별점 요약 확인

**Link**: [Section 2.1](../2601.12538.md#21-positioning-our-survey) (참고용)

---

## Recommended Reading Order

최적의 읽기 순서:

```
1. Section 1 - Introduction + Definition
   ↓ (프레임워크 이해)
2. Section 2.2 - Preliminaries (POMDP 형식화)
   ↓ (수학적 기반)
3. Section 3 - Foundational (계획/도구/탐색)
   ↓ (핵심 역량 파악)
4. Section 8 - Open Problems (미래 방향)
   ↓ (연구 기회 식별)
5. Section 4 or 5 - 관심사에 따라 선택
   ↓
6. Section 6 or 7 - 필요 시 참고
```

### Quick Read Path (~45 min)
1. Abstract + Introduction (Definition 포함) — 15분
2. Section 2.2의 Eq. 1-5 — 10분
3. Section 3의 Table 2-4 + Figure 2-4 — 10분
4. Section 8 전체 — 10분

### Deep Read Path (~3 hours)
1. 전체 Section 1-2 — 30분
2. 전체 Section 3 — 60분
3. Section 4 (피드백/메모리) — 40분
4. Section 5 (멀티에이전트) — 30분
5. Section 8 + 관심 도메인(Section 6) — 20분

---

## Section Dependency Map

```
Introduction (Definition) ◀── 먼저 읽기
    │
    ├── Section 2.1 (Skip)
    │
    ▼
Section 2.2 (POMDP) ◀── 수학적 기반
    │
    ├─────────────────────────────┐
    ▼                             ▼
Section 3 (Foundational) ◀──   Section 5 (Multi-Agent)
    │  계획/도구/탐색              │  역할/협업/진화
    │  ⭐⭐⭐                      │  ⭐⭐
    ▼                             │
Section 4 (Self-Evolving)         │
    │  피드백/메모리/진화          │
    │  ⭐⭐                       │
    │                             │
    ├─────────────────────────────┘
    ▼
Section 6 (Applications) ── Section 7 (Benchmarks)
    │  ⭐ Reference           ⭐ Reference
    ▼
Section 8 (Open Problems) ◀── 마지막 필수 읽기
    ⭐⭐⭐
```

---

## Source References

| Section | Priority | Time | Link |
|---------|----------|------|------|
| Section 1 + Def | ⭐⭐⭐ | ~15 min | [Link](../2601.12538.md#1-introduction) |
| Section 2.2 | ⭐⭐⭐ | ~20 min | [Link](../2601.12538.md#22-preliminaries) |
| Section 3 | ⭐⭐⭐ | ~30-60 min | [Link](../2601.12538.md#3-foundational-agentic-reasoning) |
| Section 8 | ⭐⭐⭐ | ~15 min | [Link](../2601.12538.md#8-open-problems) |
| Section 4 | ⭐⭐ | ~30 min | [Link](../2601.12538.md#4-self-evolving-agentic-reasoning) |
| Section 5 | ⭐⭐ | ~25 min | [Link](../2601.12538.md#5-collective-multi-agent-reasoning) |
| Section 6 | ⭐ | ~20 min | [Link](../2601.12538.md#6-applications) |
| Section 7 | ⭐ | ~15 min | [Link](../2601.12538.md#7-benchmarks) |
| Section 2.1 | Skip | - | [Link](../2601.12538.md#21-positioning-our-survey) |

---

## Navigation

← [Back to Main Analysis](../2601.12538-analysis.md)
