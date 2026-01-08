---
parent: ../2512.23676-analysis.md
source: ../2512.23676.md
created: 2026-01-09
---

# Methodology - Deep Analysis

## Paper Type Classification

**Type**: Method Paper (Architecture/Framework)

| Aspect | Value |
|--------|-------|
| **Problem Domain** | 언어 에이전트를 위한 환경 구축 |
| **Solution Type** | 하이브리드 아키텍처 (Code + LLM) |
| **Validation** | 데모 시스템 구현 (7개) |
| **Benchmark** | 없음 (정성적 검증) |

---

## Architecture Overview

### Core Concept: State Decomposition

WWM은 세계 상태 $S_t$를 두 개의 직교(orthogonal) 컴포넌트로 분해:

$$S_t = (S^{\phi}_t, S^{\psi}_t)$$

| Component | Name | Characteristics |
|-----------|------|-----------------|
| $S^{\phi}$ | Physics | 결정론적, 코드 정의, 검증 가능 |
| $S^{\psi}$ | Imagination | 확률적, LLM 생성, 창의적 |

**Source**: [Section 2.1](../2512.23676.md#21-separation-of-concerns-physics-vs-imagination)

---

### State Transition Mechanism

**Two-Phase Transition**:

```
Phase 1 (Deterministic):
S^φ_{t+1} = f_code(S^φ_t, a_t)

Phase 2 (Stochastic):
S^ψ_{t+1} ~ π_θ(· | S^φ_{t+1})
```

**Execution Order Guarantee**:
1. 코드가 먼저 Physics 상태 업데이트
2. LLM이 업데이트된 Physics 조건 하에 Imagination 생성

**Why This Order Matters**:
- LLM이 "이미 결정된" Physics를 조건으로 받음
- 구조적 환각(Structural Hallucination) 방지
- Physics 레이어가 "Ground Truth" 역할

**Source**: [Section 2.1](../2512.23676.md#21-separation-of-concerns-physics-vs-imagination)

---

## Key Technical Components

### 1. Typed Interfaces

**Traditional Approach**:
```
Latent State = Opaque Embedding Vector (e.g., 768-dim float)
```

**WWM Approach**:
```typescript
interface Planet {
  name: string;
  biome: "desert" | "ocean" | "forest" | "tundra";
  hazard: "radiation" | "storm" | "creatures" | "none";
  population: number;
}
```

**Benefits**:
- **투명성**: 상태가 무엇인지 명확히 알 수 있음
- **검증 가능**: 스키마 위반 시 즉시 감지
- **디버깅 용이**: JSON으로 상태 덤프 가능

**Implementation**:
- TypeScript interfaces define the contract
- LLM must output valid JSON matching schema
- Runtime validation before state update

**Source**: [Section 2.2](../2512.23676.md#22-typed-interfaces-as-the-common-language)

---

### 2. Deterministic Hashing

**Problem**: 무한한 세계를 DB에 저장할 수 없음

**Solution**: Just-In-Time Generation with Hashing

```
Input: location x (e.g., coordinates)
    ↓
Hash: h(x) → deterministic seed
    ↓
LLM: seed fixes sampling randomness
    ↓
Output: identical content every time
```

**Mathematical Guarantee**:
$$S^{\psi}_t \equiv S^{\psi}_{t+k} \quad \text{if} \quad \text{location}(t) = \text{location}(t+k)$$

**What This Achieves**:
- **Object Permanence**: 떠났다 돌아와도 같은 세계
- **No Storage Cost**: DB 없이 영속성 보장
- **Infinite Scale**: 모든 좌표에 대해 동작

**Source**: [Section 2.3](../2512.23676.md#23-infinite-worlds-via-deterministic-hashing)

---

### 3. Graceful Degradation

**Fidelity Levels**:

| Level | Condition | Behavior |
|-------|-----------|----------|
| High | API 정상, 지연 없음 | 실시간 LLM 생성 |
| Medium | API 지연 또는 비용 제한 | 캐시된 콘텐츠 활용 |
| Base | API 장애 | 템플릿 기반 폴백 |

**Implementation Strategy**:
```
if (llm_available && latency_ok) {
  return await llm.generate(context);
} else if (cache.has(key)) {
  return cache.get(key);
} else {
  return template.fill(context);
}
```

**Why Critical**:
- Production 환경에서 100% 가용성 불가능
- 사용자 경험 연속성 보장
- 비용 제어 가능

**Source**: [Section 2.4](../2512.23676.md#24-graceful-degradation)

---

## Technical Stack

**선택 이유와 역할**:

| Technology | Role in WWM |
|------------|-------------|
| **TypeScript** | Type safety for neuro-symbolic contract |
| **React** | UI component composition |
| **HTTP Streaming** | Real-time text delivery |
| **Serverless** | Infinite scaling without persistent infrastructure |
| **Gemini Flash** | Fast, cost-effective LLM inference |

**Stack Synergy**:
- TypeScript interfaces → Typed latent state
- HTTP streaming → Progressive rendering
- Serverless → Scale with demand, no idle cost

**Source**: [Section 2.5](../2512.23676.md#25-the-technical-stack)

---

## Implementation Pattern (Travel Atlas Example)

### Architecture Flow

```
┌─────────────────────────────────────────────────────────┐
│                    Client (Browser)                      │
├─────────────────────────────────────────────────────────┤
│  User Action: Click coordinate (x, y)                   │
│         ↓                                               │
│  worldPromptService.ts: Initialize query templates      │
│         ↓                                               │
│  proceduralBeaconService.ts: Generate beacon metadata   │
│         ↓                                               │
│  Hash(x, y) → Deterministic seed                        │
│         ↓                                               │
│  LLM API Call with seed + metadata                      │
│         ↓                                               │
│  Render generated travel guide                          │
└─────────────────────────────────────────────────────────┘
```

### Code Structure

```
src/
├── services/
│   ├── worldPromptService.ts    # Query templates
│   ├── proceduralBeaconService.ts # Beacon generation
│   └── geminiService.ts         # LLM API wrapper
├── components/
│   ├── Globe.tsx                # Interactive globe
│   └── TravelGuide.tsx          # Generated content display
└── types/
    └── interfaces.ts            # Typed latent state
```

**Source**: [Section 3.1](../2512.23676.md#31-infinite-travel-atlas)

---

## Validation Approach

### What Was Validated
- **Functionality**: 7개 도메인에서 WWM 패턴 동작 확인
- **Diversity**: 여행, SF, 게임, 시뮬레이션, 백과사전, 소설 등
- **Consistency**: 해싱을 통한 Object Permanence 검증

### What Was NOT Validated
- **정량적 성능**: 응답 시간, 처리량 등 미측정
- **사용자 연구**: 실제 사용자 피드백 없음
- **비용 분석**: LLM 호출 비용 미보고
- **확장성 테스트**: 대규모 동시 접속 시나리오 미검증
- **다른 LLM**: Gemini Flash 외 모델 미테스트

### Limitation Acknowledgment

> "Our main contributions are... We implement a suite of Web World Models on a realistic web technology stack... demonstrating the generality of the abstraction."

논문은 "generality demonstration"을 목표로 명시하며, 정량적 평가는 범위 밖임을 암시.

**Source**: [Section 1 - Introduction](../2512.23676.md#1-introduction)

---

## Comparison with Alternatives

### vs Traditional Web Framework

| Aspect | Traditional Web | WWM |
|--------|-----------------|-----|
| State Storage | Database | Hashing |
| Content | Hand-crafted | LLM-generated |
| Scale | Schema-bound | Unlimited |
| Controllability | Full | High |

### vs Fully Generative World Model

| Aspect | Fully Generative | WWM |
|--------|------------------|-----|
| Control | Low | High |
| Debugging | Hard | Easy |
| Hallucination | Common | Reduced |
| Cost | High | Moderate |
| Creativity | Maximum | Bounded |

**Source**: [Figure 1](../2512.23676.md#abstract), [Section 1](../2512.23676.md#1-introduction)

---

## Reproducibility

### What's Provided
- **Conceptual Architecture**: 명확한 설계 원칙
- **Code Patterns**: 서비스 구조, 인터페이스 예시
- **Demo Descriptions**: 각 데모의 구현 방식 설명

### What's Missing
- **Source Code**: 공개 저장소 없음 (2026년 1월 기준)
- **API Keys**: Gemini API 필요
- **Infrastructure**: Serverless 설정 상세 없음

### Reproduction Difficulty: **Medium**
- 원칙 자체는 명확하여 유사 시스템 구현 가능
- 정확한 재현을 위해서는 추가 정보 필요

---

## Key Implementation Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **LLM Model** | Gemini Flash | Fast, cost-effective |
| **Language** | TypeScript | Type safety, web-native |
| **State Format** | JSON | Human-readable, debuggable |
| **Hashing** | Coordinate-based | Spatial consistency |
| **Fallback** | Template-based | Guaranteed availability |

---

## Navigation

← [Back to Main Analysis](../2512.23676-analysis.md)
