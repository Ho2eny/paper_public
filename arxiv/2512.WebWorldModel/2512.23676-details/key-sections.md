---
parent: ../2512.23676-analysis.md
source: ../2512.23676.md
created: 2026-01-09
---

# Key Sections Guide - Deep Analysis

## Reading Priority Overview

| Priority | Time | Sections | Why |
|----------|------|----------|-----|
| ⭐⭐⭐ Must Read | 30min | Section 2, Section 3.1-3.2 | 핵심 설계 원칙 + 대표 구현 |
| ⭐⭐ Important | 20min | Section 3.3-3.4 | 게임/시뮬레이션 응용 |
| ⭐ Reference | 15min | Section 3.5-3.7, Section 4 | 추가 데모 + 배경 지식 |
| Skip | - | Appendix A | UI 스크린샷 갤러리 |

---

## ⭐⭐⭐ Must Read Sections

### Section 2: Design Principles of Web World Models

**Link**: [Section 2](../2512.23676.md#2-design-principles-of-web-world-models)

**Why Must Read**:
- 논문의 **이론적 핵심** - 모든 데모가 이 원칙에 기반
- 4가지 설계 원칙의 상세 설명
- 수학적 정의와 직관적 설명 포함

**Key Subsections**:

| Subsection | Content | Key Insight |
|------------|---------|-------------|
| [2.1](../2512.23676.md#21-separation-of-concerns-physics-vs-imagination) | Physics vs Imagination | 상태 전이 수식, 역할 분리 |
| [2.2](../2512.23676.md#22-typed-interfaces-as-the-common-language) | Typed Interfaces | JSON 스키마로 환각 방지 |
| [2.3](../2512.23676.md#23-infinite-worlds-via-deterministic-hashing) | Deterministic Hashing | Object Permanence without Storage |
| [2.4](../2512.23676.md#24-graceful-degradation) | Graceful Degradation | Fidelity Slider 개념 |
| [2.5](../2512.23676.md#25-the-technical-stack) | Technical Stack | TypeScript, HTTP, Serverless |

**Reading Tips**:
- Figure 3 (아키텍처 다이어그램)과 함께 읽기
- Figure 4 (해싱 프로세스) 이해 필수
- 수식 2개만 이해하면 전체 파악 가능

---

### Section 3.1: Infinite Travel Atlas

**Link**: [Section 3.1](../2512.23676.md#31-infinite-travel-atlas)

**Why Must Read**:
- **가장 상세하게 설명된 데모** - WWM 패턴의 레퍼런스 구현
- 실제 지리 데이터 기반 → 검증 가능한 출력
- Figure 5의 상세 플로우 다이어그램 포함

**Key Elements**:

| Element | Description |
|---------|-------------|
| **Environment** | TypeScript + CSS + HTML, 클라이언트 사이드 |
| **Physics Layer** | 지리 좌표, 테마 선택 규칙 |
| **Imagination Layer** | 여행 가이드, 일정, 스토리 생성 |
| **Hash Point** | 좌표 → 시드 → 일관된 콘텐츠 |

**Figure References**:
- Figure 5: 전체 상호작용 플로우
- Figure 15-24: UI 스크린샷 (Appendix)

---

### Section 3.2: Galaxy Travel Atlas

**Link**: [Section 3.2](../2512.23676.md#32-galaxy-travel-atlas)

**Why Must Read**:
- **완전 가상 세계** 구현 사례 - 실제 데이터 없이 무한 확장
- Typed Interface 활용의 명확한 예시 (`interface Planet`)
- 에이전트 아키텍처 설명 포함

**Key Elements**:

| Element | Description |
|---------|-------------|
| **Environment** | TypeScript, universe.ts로 절차적 생성 |
| **Physics Layer** | 은하 레이아웃, 행성 속성, 해저드 타입 |
| **Imagination Layer** | 미션 브리프, 내러티브 훅, 캐릭터 |
| **Graceful Degradation** | API 실패 시 템플릿 폴백 |

**Key Quote**:
> "If the model fails or is unreachable, the system gracefully degrades to template-based descriptions, proving that the world's existence is independent of the generative model."

**Figure References**:
- Figure 6: 시스템 아키텍처
- Figure 25-34: UI 스크린샷 (Appendix)

---

## ⭐⭐ Important Sections

### Section 3.3: AI Spire (Card Game)

**Link**: [Section 3.3](../2512.23676.md#33-a-card-game-called-ai-spire)

**Why Important**:
- **게임 도메인** 적용 사례
- "The Wish" 메커니즘 - 사용자 프롬프트 → 카드 생성
- Effect Code 개념 - LLM이 생성해도 코드가 실행

**Key Innovation**:
```
User Prompt: "a fireball that deals burn and freezes enemy"
→ LLM generates: { name, description, effect_codes }
→ Code executes: effect_codes → actual game state changes
```

**Source**: [Section 3.3](../2512.23676.md#33-a-card-game-called-ai-spire)

---

### Section 3.4: AI Alchemy (Sandbox)

**Link**: [Section 3.4](../2512.23676.md#34-a-sandbox-called-ai-alchemy)

**Why Important**:
- **셀룰러 오토마타** + LLM 조합
- 동적 반응 규칙 생성 - 고정 테이블 대신 실시간 생성
- AI Supervisor 개념 - 자율 에이전트가 시뮬레이션 조절

**Key Innovation**:
```
Traditional: Water + Fire = Steam (fixed table)
AI Alchemy: Any Element + Any Element → LLM proposes valid reaction
```

**Emergent Behavior Examples**:
- Life + Fire = Ash
- Ash + Water = Nutrient Mud
- Nutrient Mud + Life = More Life

**Source**: [Section 3.4](../2512.23676.md#34-a-sandbox-called-ai-alchemy)

---

## ⭐ Reference Sections

### Section 3.5: Cosmic Voyager

**Link**: [Section 3.5](../2512.23676.md#35-cosmic-voyager)

**When to Reference**:
- 3D WebGL 환경에 WWM 적용 시
- 교육용 콘텐츠 생성 시스템 관심 시

**Key Points**:
- WebGL 기반 태양계 탐험
- Orbit View / Pilot Mode / Surface Walk 전환
- "Cosmic Guide" 자막 30초마다 갱신

---

### Section 3.6: WWMPedia

**Link**: [Section 3.6](../2512.23676.md#36-wwmpedia)

**When to Reference**:
- 지식 기반 WWM 구현 시
- 검색 + 생성 파이프라인 설계 시

**Key Points**:
- 오픈 웹을 환경으로 사용
- 쿼리 → 검색 → 증거 수집 → 페이지 생성
- Grokipedia와 비교 (Figure 13)

---

### Section 3.7: Bookshelf

**Link**: [Section 3.7](../2512.23676.md#37-bookshelf)

**When to Reference**:
- 장문 생성 시스템 설계 시
- 스타일/톤 제어 인터페이스 관심 시

**Key Points**:
- "Physics = Narrative Mechanics" 재해석
- Interface Style + Literary Tags로 제어
- 페이지 넘김 → 연속 생성

---

### Section 4: Related Work

**Link**: [Section 4](../2512.23676.md#4-related-work)

**When to Reference**:
- 관련 연구 조사 시
- 논문 맥락 파악 시

**Categories Covered**:
1. World Models and Web Architectures
2. Persistent Agent Environments
3. Dynamic Games and Neuro-Symbolic AI
4. Agent Reasoning and Benchmarks

**Key References**:
- Ha & Schmidhuber (2018) - 원조 World Model
- Park et al. (2023) - Generative Agents
- Wang et al. (2023) - Voyager
- Yao et al. (2023) - ReAct

---

## Skip Sections

### Appendix A: Additional Examples

**Link**: [Appendix A](../2512.23676.md#appendix-a-additional-examples)

**Why Skip**:
- 본문의 Figure 확장판 (UI 스크린샷 갤러리)
- 새로운 개념 없음
- 필요시 시각적 참고용으로만 활용

**When to Reference**:
- 특정 데모의 UI/UX 확인 필요 시
- 프레젠테이션 자료 제작 시

---

## Section Dependencies

```
Section 1 (Introduction)
    ↓
Section 2 (Design Principles) ← CRITICAL FOUNDATION
    ↓
Section 3 (Examples)
    ├─ 3.1 Travel Atlas ← REFERENCE IMPLEMENTATION
    ├─ 3.2 Galaxy Atlas ← FICTIONAL VARIANT
    ├─ 3.3 AI Spire ← GAME APPLICATION
    ├─ 3.4 AI Alchemy ← SIMULATION APPLICATION
    ├─ 3.5 Cosmic Voyager
    ├─ 3.6 WWMPedia
    └─ 3.7 Bookshelf
    ↓
Section 4 (Related Work) ← CONTEXT
    ↓
Section 5 (Conclusion)
```

---

## Quick Reading Path

**15분 속독 경로**:
1. Abstract → 전체 요약
2. Figure 1 → Design Space 이해
3. Section 2.1 → Physics/Imagination 분리
4. Figure 3 → 아키텍처 시각화
5. Section 3.1 첫 3문단 → Travel Atlas 개요
6. Conclusion → 마무리

**1시간 정독 경로**:
1. Section 1 전체
2. Section 2 전체 (핵심)
3. Section 3.1-3.2 전체
4. Section 3.3-3.4 요약 부분
5. Section 4 훑어보기
6. Section 5 전체

---

## Navigation

← [Back to Main Analysis](../2512.23676-analysis.md)
