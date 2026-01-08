---
parent: ../1802.08802-analysis.md
source: ../1802.08802.md
created: 2026-01-09
---

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

| Metric | Value |
|--------|-------|
| **Total Reading Time** | ~50 minutes (전체) |
| **Essential Only** | ~25 minutes (핵심만) |
| **Total Sections** | 7 sections + Appendix |
| **Must Read** | 4 sections |

---

## ⭐⭐⭐ Must Read

### Section 1: Introduction

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
문제 동기, 기존 접근법 한계, WGE의 핵심 아이디어를 파악할 수 있음. 논문 전체를 이해하기 위한 기초.

**Key Points**:
1. Sparse reward 문제: 웹 태스크에서 하나의 실수로 전체 실패
2. BC의 overfitting 문제: warm-starting도 효과 미미
3. WGE의 핵심 아이디어: demonstration → 워크플로우 → 탐색 가이드
4. Environment-blind 특성의 장점

**Reading Tips**:
- Figure 1을 먼저 보고 전체 구조 파악
- "environment-blind"가 왜 중요한지 집중

**Estimated Time**: ~5 minutes

**Link**: [Section 1](../1802.08802.md#1-introduction)

---

### Section 3: Inducing Workflows from Demonstrations

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
WGE의 핵심인 워크플로우 추출 방법. 이 섹션을 이해해야 전체 방법론 이해 가능.

**Key Points**:
1. 워크플로우 스텝 $z_t$의 정의: 상태 → 허용 행동 집합
2. 워크플로우 격자(lattice) 구성 방법
3. 제약 언어 문법: `Click(Tag("img"))`, `Type(Near(Text("Bob")))` 등
4. 노이즈 처리: shortcut 스텝 추가

**Reading Tips**:
- Figure 2의 워크플로우 격자 예시를 상세히 분석
- Table 3의 제약 언어 문법 참조

**Estimated Time**: ~7 minutes

**Link**: [Section 3](../1802.08802.md#3-inducing-workflows-from-demonstrations)

---

### Section 4: Workflow Exploration Policy

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
탐색 정책 $\pi_w$의 수학적 정의와 학습 방법. Environment-blind 특성의 구체적 구현.

**Key Points**:
1. 워크플로우 스텝 선택 확률: $\pi_w(z | d, t) \propto \exp(\psi_{z,t,d})$
2. Environment-blind: 상태 $s_t$에 의존하지 않음
3. REINFORCE로 학습
4. 시연 선택: 목표 유사도 기반

**Reading Tips**:
- 수식보다 직관적 이해에 집중
- "왜 environment-blind가 overfitting을 방지하는가?" 생각

**Estimated Time**: ~5 minutes

**Link**: [Section 4](../1802.08802.md#4-workflow-exploration-policy)

---

### Section 6.2: Main Results

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
방법론의 효과를 정량적으로 확인. SOTA 달성 증거.

**Key Points**:
1. MiniWoB에서 대부분 태스크 SOTA 달성
2. Figure 3: 태스크별 성공률 비교
3. WGE vs BC+RL: 아키텍처 동일 조건에서 WGE 우위
4. 어려운 태스크에서 더 큰 격차

**Reading Tips**:
- Figure 3을 상세히 분석
- click-checkboxes-large (0% vs 84%) 같은 극적 차이에 주목

**Estimated Time**: ~3 minutes

**Link**: [Section 6.2](../1802.08802.md#62-main-results)

---

## ⭐⭐ Important

### Section 5: Neural Policy

**Priority**: ⭐⭐ Important

**Why Read**:
DOMNET 아키텍처의 핵심 설계. 웹 환경 특화 신경망 이해.

**Key Points**:
1. DOM 임베딩: base + spatial + tree + match
2. 공간적 이웃 (30px) vs 트리 이웃 (LCA 깊이)
3. Attention 메커니즘
4. A2C로 on-policy + off-policy 학습

**When to Read**:
- 웹 환경에서 신경망 설계에 관심 있을 때
- DOMNET 재현이 필요할 때

**Link**: [Section 5](../1802.08802.md#5-neural-policy)

---

### Section 6.3: Analysis

**Priority**: ⭐⭐ Important

**Why Read**:
방법론의 다양한 측면 심층 분석. 한계와 강점 파악.

**Key Points**:
1. 6.3.1 MiniWoB++: 새로운 벤치마크 소개
2. 6.3.3 Alaska Airlines: 실제 웹사이트 적용
3. 6.3.4 Sample Efficiency: 100x 효율성 검증

**When to Read**:
- 실제 적용 가능성 평가 시
- MiniWoB++ 벤치마크 사용 시

**Link**: [Section 6.3](../1802.08802.md#63-analysis)

---

## ⭐ Reference

### Section 2: Setup

**Priority**: ⭐ Reference Only

**Content Summary**:
MDP 정의, 상태/행동 공간 정의. 표준적인 문제 설정.

**When Useful**:
- 정확한 문제 정의가 필요할 때
- 재현 시 환경 설정 참조

**Link**: [Section 2](../1802.08802.md#2-setup)

---

### Section 6.1: Task Setups

**Priority**: ⭐ Reference Only

**Content Summary**:
실험 설정, 하이퍼파라미터, 베이스라인 상세.

**When Useful**:
- 재현 시 실험 설정 참조
- 베이스라인 비교 상세 필요 시

**Link**: [Section 6.1](../1802.08802.md#61-task-setups)

---

### Appendix B: Learned Workflows

**Priority**: ⭐ Reference Only

**Content**:
실제 학습된 워크플로우 예시 (login-user, email-inbox, Alaska 등)

**When Useful**:
- 워크플로우가 실제로 어떻게 생겼는지 확인
- 방법론의 작동 방식 직관적 이해

**Link**: [Appendix B](../1802.08802.md#appendix-b-examples-of-learned-workflows)

---

### Appendix C: Neural Architecture

**Priority**: ⭐ Reference Only

**Content**:
DOMNET 상세 구현, attention 메커니즘 상세

**When Useful**:
- DOMNET 재현 시
- 아키텍처 상세 이해 필요 시

**Link**: [Appendix C](../1802.08802.md#appendix-c-details-of-the-neural-model-architecture)

---

## Skip

### Section 7: Discussion

**Priority**: Skip

**Why Skip**:
- 관련 연구 정리 위주
- 핵심 기여와 직접 관련 없음
- 새로운 정보 적음

**Alternative**:
- Related Works 섹션은 WebArena, Mind2Web 등 후속 연구에서 더 잘 정리됨

**Link**: [Section 7](../1802.08802.md#7-discussion) (참고용)

---

### Appendix A: Constraint Language

**Priority**: Skip (unless implementing)

**Why Skip**:
- 재현하지 않으면 불필요
- Section 3에서 충분히 설명됨

**Alternative**:
- Section 3의 예시로 충분히 이해 가능

**Link**: [Appendix A](../1802.08802.md#appendix-a-constraint-language-for-workflow-steps)

---

## Recommended Reading Order

최적의 읽기 순서:

```
1. [Abstract](../1802.08802.md#abstract) - 전체 개요
   ↓
2. [Figure 1 + Algorithm 1](../1802.08802.md#figure-1) - 시스템 구조 파악
   ↓
3. [Section 3](../1802.08802.md#3-inducing-workflows-from-demonstrations) - 워크플로우 추출
   ↓
4. [Section 4](../1802.08802.md#4-workflow-exploration-policy) - 탐색 정책
   ↓
5. [Section 6.2](../1802.08802.md#62-main-results) - 주요 결과
   ↓
6. [Section 5](../1802.08802.md#5-neural-policy) - DOMNET (선택)
   ↓
7. [Section 6.3](../1802.08802.md#63-analysis) - 심층 분석 (선택)
```

### Quick Read Path (~15 min)
1. Abstract 전체 읽기
2. Figure 1 + Algorithm 1 상세히 보기
3. Section 3 워크플로우 격자 예시 (Figure 2) 중심으로
4. Figure 3 + Table 1 결과 확인
5. Section 6.3.4 샘플 효율성

### Deep Read Path (~45 min)
1. 전체 Introduction
2. Section 2 (Setup)
3. 전체 Section 3, 4
4. 전체 Section 5
5. 전체 Section 6
6. Appendix B (예시 확인)

---

## Section Dependency Map

```
Abstract
    │
    ├── Introduction (1)
    │       │
    │       ▼
    │   Setup (2) ←──────────────────────┐
    │       │                            │
    │       ▼                            │
    │   Workflows (3) ─────┐             │
    │       │              ↓             │
    │       ▼              ↓             │
    │   Workflow Policy (4)              │
    │       │              ↓             │
    │       ▼              ↓             │
    │   Neural Policy (5) ←┘             │
    │       │                            │
    │       ▼                            │
    │   Experiments (6) ─────────────────┘
    │       │
    │       ▼
    └── Discussion (7)
```

---

## Source References

| Section | Priority | Time | Link |
|---------|----------|------|------|
| Introduction | ⭐⭐⭐ | ~5 min | [Section 1](../1802.08802.md#1-introduction) |
| Workflows | ⭐⭐⭐ | ~7 min | [Section 3](../1802.08802.md#3-inducing-workflows-from-demonstrations) |
| Workflow Policy | ⭐⭐⭐ | ~5 min | [Section 4](../1802.08802.md#4-workflow-exploration-policy) |
| Main Results | ⭐⭐⭐ | ~3 min | [Section 6.2](../1802.08802.md#62-main-results) |
| Neural Policy | ⭐⭐ | ~5 min | [Section 5](../1802.08802.md#5-neural-policy) |
| Analysis | ⭐⭐ | ~7 min | [Section 6.3](../1802.08802.md#63-analysis) |
| Setup | ⭐ | ~3 min | [Section 2](../1802.08802.md#2-setup) |
| Discussion | Skip | - | [Section 7](../1802.08802.md#7-discussion) |

---

## Navigation

← [Back to Main Analysis](../1802.08802-analysis.md)
