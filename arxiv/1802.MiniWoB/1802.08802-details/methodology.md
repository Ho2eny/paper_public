---
parent: ../1802.08802-analysis.md
source: ../1802.08802.md
created: 2026-01-09
---

# Methodology - Deep Analysis

## Paper Type

**Type**: Method

---

## Approach Overview

### High-Level Summary
WGE(Workflow-Guided Exploration)는 웹 인터페이스에서 강화학습 에이전트를 훈련하기 위한 3단계 프레임워크이다. 핵심 아이디어는 demonstration을 직접 모방 학습의 대상으로 사용하지 않고, 탐색 공간을 제약하는 가이드로 활용하는 것이다.

첫째, demonstration에서 "워크플로우 격자"를 추출한다. 워크플로우는 각 시점에서 허용되는 행동의 유형(예: "입력 필드 클릭", "버튼 클릭")만 지정하고, 구체적으로 어떤 요소를 선택할지는 명시하지 않는다. 이를 통해 overfitting을 방지하면서도 탐색을 효과적으로 안내할 수 있다.

둘째, "environment-blind" 워크플로우 정책 $\pi_w$를 학습한다. 이 정책은 환경 상태에 의존하지 않고 demonstration과 시간 스텝에만 의존하여 워크플로우 스텝을 선택한다. REINFORCE로 학습되며, 성공한 워크플로우의 확률을 높인다.

셋째, 워크플로우 탐색이 생성한 성공 에피소드를 replay buffer에 저장하고, 이를 통해 DOMNET 기반 신경망 정책 $\pi_n$을 학습한다. A2C로 on-policy + off-policy 업데이트를 수행하며, 최종적으로 테스트 시에는 $\pi_n$만 사용한다.

### Key Innovation
- **Demonstration 활용 패러다임 전환**: 직접 모방 → 탐색 가이드
- **Environment-blind 정책**: 상태 무관으로 빠른 학습 + overfitting 방지
- **Two-policy 시스템**: $\pi_w$(탐색용) + $\pi_n$(추론용) 역할 분리

**Source**: [Method Overview](../1802.08802.md#1-introduction)

---

## Technical Details

### For Method Papers

#### Problem Formulation
| Aspect | Description | Source |
|--------|-------------|--------|
| **Input** | DOM 트리 상태 $s$ + 구조화된 목표 $g$ | [Section 2](../1802.08802.md#2-setup) |
| **Output** | 행동 시퀀스 (Click/Type) | [Section 2](../1802.08802.md#2-setup) |
| **Objective** | 에피소드 종료 시 +1 reward 최대화 | [Section 2](../1802.08802.md#2-setup) |
| **Constraints** | Sparse reward, Large action space | [Section 1](../1802.08802.md#1-introduction) |

#### Architecture / Pipeline

```
[Demonstrations]
       │
       ▼
┌─────────────────┐
│ Workflow        │
│ Induction       │ → 각 시점에서 가능한 워크플로우 스텝 열거
│ (Section 3)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Workflow        │
│ Policy π_w      │ → environment-blind 워크플로우 선택
│ (Section 4)     │
└────────┬────────┘
         │ (탐색)
         ▼
┌─────────────────┐
│ Replay Buffer   │ → 성공 에피소드 저장
│ (+1 episodes)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Neural Policy   │
│ π_n (DOMNET)    │ → DOM 구조 인식 정책 학습
│ (Section 5)     │
└─────────────────┘
```

**Components**:
1. **Workflow Induction**: Demonstration에서 워크플로우 격자 추출 - [Section 3](../1802.08802.md#3-inducing-workflows-from-demonstrations)
2. **Workflow Policy**: Environment-blind 탐색 정책 - [Section 4](../1802.08802.md#4-workflow-exploration-policy)
3. **Neural Policy**: DOMNET 기반 최종 정책 - [Section 5](../1802.08802.md#5-neural-policy)

#### Training / Learning
| Aspect | Value | Source |
|--------|-------|--------|
| **π_w Loss** | REINFORCE (policy gradient) | [Section 4](../1802.08802.md#4-workflow-exploration-policy) |
| **π_n Loss** | A2C (actor-critic) | [Section 5](../1802.08802.md#5-neural-policy) |
| **Data** | Demonstrations (10개) + 탐색 에피소드 | [Section 6.1](../1802.08802.md#61-task-setups) |

**Workflow Policy 학습**:
```python
# REINFORCE with baseline
for episode in rollouts:
    for t in range(T):
        # 워크플로우 스텝 확률
        log_prob = log(pi_w(z_t | d, t))
        # Advantage
        advantage = G_t - baseline[d, t]
        # Policy gradient
        loss -= log_prob * advantage
```

**Neural Policy 학습**:
```python
# A2C with replay buffer
# Off-policy: 성공 에피소드 모방
for episode in replay_buffer.sample():
    loss = cross_entropy(pi_n(s), a_expert)

# On-policy: 직접 탐색
for episode in pi_n.rollout():
    loss = actor_loss + critic_loss
```

#### Inference
테스트 시에는 워크플로우 정책 $\pi_w$를 사용하지 않고, 학습된 신경망 정책 $\pi_n$만 사용하여 행동 선택.

```python
def inference(env, pi_n):
    s = env.reset()
    while not done:
        a = pi_n.select_action(s)  # DOMNET
        s, r, done = env.step(a)
    return trajectory
```

**Source**: [Inference Section](../1802.08802.md#5-neural-policy)

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| Demonstration 활용 | 탐색 가이드로 사용 | Overfitting 방지 | 직접 BC | [Section 1](../1802.08802.md#1-introduction) |
| 워크플로우 정책 | Environment-blind | 빠른 학습, 일반화 | State-dependent | [Section 4](../1802.08802.md#4-workflow-exploration-policy) |
| 신경망 아키텍처 | DOMNET (DOM 특화) | 웹 구조 활용 | CNN, MLP | [Section 5](../1802.08802.md#5-neural-policy) |
| 학습 알고리즘 | A2C + Replay Buffer | On/Off-policy 혼합 | PPO, DQN | [Section 5](../1802.08802.md#5-neural-policy) |
| 공간 이웃 거리 | 30px | 관련 요소 포착 | 다른 거리 | [Section 5](../1802.08802.md#5-neural-policy) |

---

## Assumptions & Limitations

### Assumptions
1. **구조화된 목표**: 워크플로우 언어가 key-value 형태의 목표에 의존 - [Section 6.3.2](../1802.08802.md#632-natural-language-inputs)
2. **유사 태스크 존재**: 새 태스크와 유사한 demonstration이 있어야 함 - [Section 4](../1802.08802.md#4-workflow-exploration-policy)
3. **DOM 접근 가능**: 웹페이지의 DOM 트리에 접근 가능해야 함 - [Section 2](../1802.08802.md#2-setup)

### Limitations
1. **자연어 목표 제한**: 순수 자연어 지시에는 워크플로우 매칭이 어려움
2. **새로운 웹사이트 일반화**: 완전히 다른 유형의 웹사이트로의 transfer 검증 부족
3. **동적 웹 환경**: 광고, 팝업 등 실제 웹의 동적 요소 미고려
4. **Action space 제한**: Click과 Type만 지원, Scroll, Hover 등 미지원

### Scope
- **In Scope**: MiniWoB++ 스타일의 구조화된 웹 태스크
- **Out of Scope**: 복잡한 멀티스텝 reasoning, 다중 탭/창 조작, 로그인 필요 태스크

---

## Complexity Analysis

| Aspect | Complexity | Notes |
|--------|------------|-------|
| **Workflow Induction** | O(T × \|Z_t\|) per demo | T: 시간 스텝, \|Z_t\|: 가능한 워크플로우 스텝 수 |
| **π_w Training** | O(\|D\| × T × \|Z_t\|) | D: demonstration 수 |
| **π_n Training** | Standard RL | 환경 복잡도에 의존 |
| **Data** | ~10 demonstrations | 기존 대비 100x 감소 |
| **Compute** | 수 시간 (GPU) | 정확한 요구사항 미명시 |

---

## Detailed Method Components

### 1. Workflow Induction (Section 3)

**제약 언어 문법**:
```
constraint  ::=  Click(elementSet) | Type(elementSet, string)
elementSet  ::=  Tag(tag)           # HTML 태그 매칭
             |   Text(string)       # 텍스트 완전 매칭
             |   Like(string)       # 부분 문자열 매칭
             |   Near(elementSet)   # 30px 근접
             |   SameRow(elementSet)  # 수평 정렬
             |   SameCol(elementSet)  # 수직 정렬
```

**워크플로우 격자 구성**:
```
시연: (s̃₁, ã₁, ..., s̃_T, ã_T)
     ↓
각 t에서 Z_t = {z_t : ã_t ∈ z_t(s̃_t)} 열거
     ↓
워크플로우 집합 = Z₁ × Z₂ × ... × Z_T
```

### 2. Workflow Policy (Section 4)

**수학적 정의**:
$$\pi_w(z | d, t) = \frac{\exp(\psi_{z,t,d})}{\sum_{z' \in Z_t} \exp(\psi_{z',t,d})}$$

**특성**:
- Environment-blind: $s_t$에 의존하지 않음
- 파라미터: 각 $(z, t, d)$ 조합에 스칼라 $\psi_{z,t,d}$ 하나
- 학습: REINFORCE with baseline

### 3. DOMNET Architecture (Section 5)

**DOM 임베딩**:

$$v_e^{DOM} = [v_e^{base} \; ; \; v_e^{spatial} \; ; \; v_e^{tree} \; ; \; v_e^{match}]$$

| 임베딩 | 계산 방식 |
|--------|----------|
| $v_e^{base}$ | 태그, 클래스, 텍스트 등 임베딩 concat |
| $v_e^{spatial}$ | 30px 내 요소들의 base 임베딩 합 |
| $v_e^{tree}$ | 깊이 k=3,4,5,6에서 LCA 이웃 max-pooling |
| $v_e^{match}$ | 목표와 일치하는 단어 임베딩 합 |

---

## Source References Summary

| Topic | Section | Link |
|-------|---------|------|
| Method Overview | Introduction | [Section 1](../1802.08802.md#1-introduction) |
| Problem Setup | Setup | [Section 2](../1802.08802.md#2-setup) |
| Workflow Induction | Workflows | [Section 3](../1802.08802.md#3-inducing-workflows-from-demonstrations) |
| Workflow Policy | Policy | [Section 4](../1802.08802.md#4-workflow-exploration-policy) |
| Neural Policy | DOMNET | [Section 5](../1802.08802.md#5-neural-policy) |
| Experiments | Results | [Section 6](../1802.08802.md#6-experiments) |
| Implementation | Appendix | [Appendix A-C](../1802.08802.md#appendix-a-constraint-language-for-workflow-steps) |

---

## Navigation

← [Back to Main Analysis](../1802.08802-analysis.md)
