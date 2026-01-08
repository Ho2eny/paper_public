---
parent: ../2601.22149-analysis.md
source: ../2601.22149.md
created: 2026-02-06
---

# Methodology - Deep Analysis

## Paper Type

**Type**: Method

---

## Approach Overview

### High-Level Summary

DynaWeb은 고전적 Dyna 아키텍처를 LLM 기반 웹 에이전트에 적용한 모델 기반 강화학습(MBRL) 프레임워크다. 핵심 아이디어는 학습된 웹 월드 모델을 합성 웹 환경으로 활용하여, 에이전트가 실제 웹 상호작용 없이 "상상(dream)" 속에서 다단계 롤아웃을 수행하고, 이를 온라인 정책 최적화에 직접 활용하는 것이다.

기술적으로 DynaWeb은 세 가지 주요 컴포넌트로 구성된다: (1) accessibility tree 기반의 웹 상태 전이를 예측하는 LLM 월드 모델, (2) 월드 모델과 반복 상호작용하여 다단계 상상 궤적을 생성하는 에이전트 정책, (3) 상상 궤적과 실제 전문가 궤적을 혼합하여 시퀀스 수준 정책 최적화를 수행하는 GSPO 학습 루프. 이 세 컴포넌트가 유기적으로 결합되어, 비용 없이 온라인 RL의 이점을 확보한다.

기존 접근과의 차별점은 월드 모델의 역할에 있다. WebDreamer나 WMA가 월드 모델을 추론 시점의 의사결정 보조 도구로 사용한 반면, DynaWeb은 월드 모델을 정책 최적화의 핵심 학습 기제로 격상시켰다. 이는 실제 환경 상호작용을 대체하는 수준의 시뮬레이션 충실도가 필요하며, delta 기반 예측과 추론 과정 학습이 이를 가능케 했다.

### Key Innovation

웹 월드 모델이 생성한 상상 궤적을 on-policy 학습의 1등급 데이터로 활용하는 최초의 연구. 이를 통해 MBRL의 고전적 "learn from imagination" 패러다임이 LLM 웹 에이전트 영역에서도 유효함을 입증.

**Source**: [Method Overview](../2601.22149.md#3-method)

---

## Technical Details

### Problem Formulation

| Aspect | Description | Source |
|--------|-------------|--------|
| **Input** | 자연어 태스크 쿼리 $q$ + 타겟 웹사이트 $w$ + 시스템 지시 $I$ | [Section 3.1](../2601.22149.md#31-problem-formulation) |
| **Output** | 다단계 상호작용 궤적 $\tau = (o_1, h_1, a_1, \ldots, o_T, h_T, a_T)$ | [Section 3.1](../2601.22149.md#31-problem-formulation) |
| **Objective** | 태스크 완료 보상 $\hat{r}(\tau, q) \in [0,1]$ 최대화 | [Section 3.1](../2601.22149.md#31-problem-formulation) |
| **Constraints** | 부분 관찰(POMDP), accessibility tree 기반 관찰 | [Section 3.1](../2601.22149.md#31-problem-formulation) |

### Architecture / Pipeline

```
[Task Query q + Initial State o_1]
          │
          ▼
    ┌─────────────┐
    │ Agent π_θ   │ ←── Llama-3.1-8B (NNetNav 초기화)
    │ (reasoning  │
    │  + action)  │
    └──────┬──────┘
           │ a_t
           ▼
    ┌─────────────┐
    │ Web World   │ ←── GPT-oss-120b (NNetNav 데이터로 파인튜닝)
    │ Model p_φ   │
    │ (delta      │
    │  prediction)│
    └──────┬──────┘
           │ ô_{t+1}
           ▼
    [Repeat for T steps (max 5)]
           │
           ▼
    ┌─────────────┐
    │ Self-Assess │ ←── 태스크 완료 판정
    │ Reward      │
    └──────┬──────┘
           │ r̂ ∈ {0,1}
           ▼
    ┌─────────────────────────┐
    │ Mix with Expert         │ ←── 50% 확률로 실제 전문가 궤적 교체
    │ Trajectories            │
    └──────────┬──────────────┘
               │
               ▼
    ┌─────────────┐
    │ GSPO        │ ←── Sequence-level policy optimization
    │ Optimizer   │
    └─────────────┘
```

**Components**:
1. **Agent Policy ($\pi_\theta$)**: Llama-3.1-8B-Instruct 기반, NNetNav SFT로 초기화. 관찰 + 이력 기반으로 추론 과정(CoT)과 행동을 동시 생성 - [Section 3.1](../2601.22149.md#31-problem-formulation)
2. **Web World Model ($p_\phi$)**: GPT-oss-120b 기반, NNetNav 데이터로 파인튜닝. 현재 상태 + 행동 → (추론, 상태 변화) 예측 - [Section 3.2](../2601.22149.md#32-web-world-model)
3. **GSPO Optimizer**: PPO 스타일 클리핑 + 시퀀스 수준 importance ratio - [Section 3.3](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents)

### Training / Learning

| Aspect | Value | Source |
|--------|-------|--------|
| **Loss Function** | GSPO 클리핑 목적 함수 (Eq. 8) | [Section 3.3](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents) |
| **Optimizer** | GSPO (GRPO-style advantage) | [Section 3.3](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents) |
| **Agent Training Data** | 상상 궤적 (50%) + 실제 전문가 궤적 (50%) | [Section 4.1](../2601.22149.md#41-setups) |
| **WM Training Data** | NNetNav 데이터셋 (상태 전이 + GPT-oss-120b 생성 delta) | [Section 3.2](../2601.22149.md#32-web-world-model) |
| **Learning Rate** | $1 \times 10^{-6}$ | [Appendix A.3](../2601.22149.md#a3-training-recipe-and-hyperparameters) |
| **Epochs** | 10 | [Appendix A.3](../2601.22149.md#a3-training-recipe-and-hyperparameters) |

### Inference

추론 시점에서는 월드 모델을 사용하지 않음. 학습된 에이전트 $\pi_\theta$가 실제 웹 환경(WebArena/WebVoyager)에서 직접 행동을 생성. 추론은 표준적인 autoregressive 디코딩으로, vLLM을 통한 비동기 추론 사용.

**Source**: [Section 4.1](../2601.22149.md#41-setups)

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| **WM 예측 방식** | Delta 기반 (상태 변화만 예측) | 웹 페이지의 대부분 요소가 불변, 전체 예측은 정보 이득 낮음 | 전체 다음 상태 예측, 잠재 공간 예측 | [Section 3.2](../2601.22149.md#32-web-world-model) |
| **WM 기반 모델** | GPT-oss-120b (120B) | 높은 언어 이해/생성 능력으로 정확한 상태 전이 예측 | 소형 LLM, 비-LLM 모델 | [Section 4.1](../2601.22149.md#41-setups) |
| **드림 길이** | 최대 5 step | 4-5 step에서 성능 정점, 이후 환각 누적으로 하락 | 1 step (짧음), 10+ step (길음) | [Section 5.1](../2601.22149.md#51-wm-dream-length-matters) |
| **전문가 혼합 비율** | ~50% | ~40%에서 최적 성능, 학습 안정화 효과 | 0% (순수 상상), 100% (순수 실제) | [Section 5.2](../2601.22149.md#52-effect-of-real-interaction-data-percentage) |
| **RL 알고리즘** | GSPO | 시퀀스 수준 ratio로 긴 궤적의 gradient 안정화 | PPO (토큰 수준), REINFORCE, DAPO | [Section 3.3](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents) |
| **관찰 표현** | Accessibility tree | 구조화된 웹 요소 표현, LLM이 직접 처리 가능 | 스크린샷, HTML, DOM tree | [Section 3.1](../2601.22149.md#31-problem-formulation) |

---

## Assumptions & Limitations

### Assumptions

1. **월드 모델 충실도**: 학습된 월드 모델이 실제 웹 동작을 충분히 정확하게 근사한다고 가정. 5 step 이내에서는 유효하지만 장기적 정확도는 검증되지 않음 - [Section 5.1](../2601.22149.md#51-wm-dream-length-matters)
2. **자기 평가 보상의 신뢰성**: 에이전트의 자기 평가로 얻은 보상 $\hat{r} \in \{0,1\}$이 실제 태스크 완료를 정확하게 반영한다고 가정 - [Section 3.3](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents)
3. **SFT 초기화의 충분성**: NNetNav SFT 에이전트가 의미 있는 탐색을 할 수 있을 정도로 충분히 사전 학습되어 있다고 가정 - [Section 4.1](../2601.22149.md#41-setups)

### Limitations

1. **단기 드림 한계**: 5 step이라는 드림 길이 제한은 평균 10+ step인 실제 웹 태스크의 일부만 커버. 장기 계획이 필요한 태스크(GitHub, ArXiv)에서 성능 저하의 원인.
2. **도메인 편향**: NNetNav 데이터로 학습된 월드 모델은 해당 데이터에 포함된 웹사이트 유형에 편향. 새로운 유형의 웹사이트에 대한 일반화는 미검증.
3. **컴퓨팅 비용**: GPT-oss-120b 기반 월드 모델 + 8xH100 학습은 상당한 리소스 요구. 학계 연구자의 접근성 제한.
4. **보상 모델 부재**: 외부 보상 모델 없이 자기 평가에 의존하므로, 보상 해킹(reward hacking) 가능성 미검토.

### Scope

- **In Scope**: 텍스트 기반(accessibility tree) 웹 에이전트, WebArena/WebVoyager 벤치마크, 8B 모델 백본
- **Out of Scope**: 멀티모달(시각 기반) 에이전트, 모바일/데스크톱 환경, 실시간 동적 웹사이트, 안전성/정렬 문제

---

## Complexity Analysis

| Aspect | Complexity | Notes |
|--------|------------|-------|
| **WM Training** | 1 epoch SFT on NNetNav data | LlamaFactory 라이브러리 사용, 시간 미기재 |
| **Agent Training** | 10 epochs GSPO | 8xH100 단일 노드, 배치 크기 4 |
| **Dream Generation** | $O(n \times T \times C_\text{WM})$ | $n$=8 롤아웃, $T$=5 step, $C_\text{WM}$=120B 모델 추론 비용 |
| **Inference** | 표준 autoregressive | WM 불필요, 에이전트만 사용 |
| **Data** | NNetNav SFT 데이터셋 | 공개 데이터 |
| **Compute** | 8 x H100 GPU (단일 노드) | bfloat16, FSDP offloading |

---

## World Model Details

### Training Pipeline

```
실제 웹 궤적 (NNetNav)
    │
    ├── (o_t, a_t, o_{t+1}) 추출
    │
    ▼
GPT-oss-120b로 delta 생성
    │
    ├── 입력: (I, o_t, a_t, o_{t+1})
    ├── 출력: 추론 과정 r + 상태 변화 Δ(o_t, o_{t+1})
    │
    ▼
월드 모델 SFT
    │
    ├── 목표: p_φ(r, Δ | I, o_t, a_t)
    ├── 1 epoch
    │
    ▼
학습된 WM p_φ
```

### Inference Pipeline (Dream Generation)

```
초기 상태 o_1 (NNetNav 데이터에서 무작위 샘플)
    │
    ▼
┌──► 에이전트: (h_t, a_t) ~ π_θ(·|I, q, o_{1:t}, ...)
│        │
│        ▼
│   WM: Δ 예측 → o_t에 적용 → ô_{t+1}
│        │
│        ├── terminal state? → 종료
│        └── t < 5? ──────────┘
│                              │
└──────────────────────────────┘
    │
    ▼
자기 평가: r̂(τ̂, q) ∈ {0, 1}
```

### Key Insight: Delta Prediction

웹 페이지 전이의 특성상, $o_{t+1}$의 대부분은 $o_t$와 동일하다. 예를 들어 검색 버튼 클릭 시:
- **변하는 것**: 검색 결과 영역, URL
- **변하지 않는 것**: 헤더, 사이드바, 푸터, 네비게이션

따라서 전체 $o_{t+1}$을 예측하면 대부분이 복사(copy)에 불과하여 정보 이득이 낮다. Delta 예측은 변화 부분에만 집중함으로써 학습 효율성을 극대화한다.

---

## GSPO vs Standard PPO

| Aspect | Standard PPO | GSPO |
|--------|-------------|------|
| Importance Ratio | 토큰 수준 $r_k(\theta)$ | 시퀀스 수준 $s(\theta)$ = 기하 평균 |
| Advantage | 토큰별 advantage | 궤적별 advantage |
| Gradient 안정성 | 긴 시퀀스에서 불안정 | 기하 평균으로 정규화 |
| 적합한 태스크 | 짧은 생성 | 긴 다단계 상호작용 |

웹 에이전트 궤적은 수천 토큰에 달할 수 있어, 토큰 수준 PPO는 gradient 폭발/소실 문제가 심각. GSPO의 시퀀스 수준 ratio는 이를 효과적으로 완화.

---

## Source References Summary

| Topic | Section | Link |
|-------|---------|------|
| Problem Formulation | Section 3.1 | [Link](../2601.22149.md#31-problem-formulation) |
| Web World Model | Section 3.2 | [Link](../2601.22149.md#32-web-world-model) |
| DynaWeb MBRL | Section 3.3 | [Link](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents) |
| Experimental Setup | Section 4.1 | [Link](../2601.22149.md#41-setups) |
| Main Results | Section 4.2 | [Link](../2601.22149.md#42-main-results) |
| Design Ablations | Section 5 | [Link](../2601.22149.md#5-analysis) |
| Hyperparameters | Appendix A.3 | [Link](../2601.22149.md#a3-training-recipe-and-hyperparameters) |
| System Prompts | Appendix A.1-A.2 | [Link](../2601.22149.md#appendix) |

---

## Navigation

<- [Back to Main Analysis](../2601.22149-analysis.md)
