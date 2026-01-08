---
parent: ../2601.22149-analysis.md
source: ../2601.22149.md
created: 2026-02-06
---

# Core Contributions - Deep Analysis

## Overview

| # | Contribution | Impact | Source |
|---|--------------|--------|--------|
| 1 | Web World Model | 실제 웹 없이 현실적 상태 전이 시뮬레이션 | [Section 3.2](#contribution-1-web-world-model) |
| 2 | Imagination-driven Online RL | 월드 모델 궤적을 온라인 RL의 1등급 데이터로 활용 | [Section 3.3](#contribution-2-imagination-driven-online-rl) |
| 3 | DynaWeb Framework | 상상 + 실제 궤적 혼합 MBRL 프레임워크 | [Section 3.3](#contribution-3-dynaweb-framework) |
| 4 | Empirical Design Principles | MBRL 설계 변수에 대한 체계적 실험 분석 | [Section 5](#contribution-4-empirical-design-principles) |

---

## Contribution 1: Web World Model

### What

Accessibility tree 기반의 웹 상태 전이를 예측하는 LLM 월드 모델. 현재 웹 페이지 상태와 에이전트 행동이 주어지면, 다음 웹 페이지 상태를 자연어 기반으로 예측한다. 핵심적으로, 전체 다음 상태를 한 번에 예측하는 대신 **상태 변화(state changes, $\Delta$)**를 먼저 추론하고 이를 현재 상태에 적용하는 2단계 방식을 채택.

### Technical Details

- **기반 모델**: GPT-oss-120b를 NNetNav 데이터셋으로 파인튜닝
- **학습 데이터 생성**: 실제 웹 상호작용 궤적 $(o_t, a_t, o_{t+1})$에서 GPT-oss-120b로 상태 변화 설명 $\Delta(o_t, o_{t+1})$ 추출
- **훈련 목표**: 추론 과정($r$)과 상태 변화($\Delta$)를 동시에 예측하는 reasoning 모델로 학습
- **관찰 공간에서 직접 작동**: 잠재 공간이 아닌 accessibility tree 형태로 직접 예측

### Why It Matters

- **학술적**: 웹 환경의 상태 전이를 LLM으로 모델링할 수 있음을 실증적으로 입증. 특히 delta 예측 방식은 웹 페이지의 고유 특성(대부분 불변 + 소수 변화)을 효과적으로 포착.
- **실용적**: 학습된 월드 모델은 재사용 가능한 시뮬레이터로서, 한 번 학습하면 다양한 에이전트 정책 훈련에 반복 활용 가능.

### Comparison to Prior Work

| Aspect | DynaWeb WM | WebDreamer | WMA |
|--------|------------|------------|-----|
| 모델 학습 | 전용 파인튜닝 | 프롬프팅만 | 전용 파인튜닝 |
| 활용 시점 | 훈련 시점 (RL) | 추론 시점 | 추론 시점 |
| 예측 방식 | Delta 기반 | 전체 상태 | 전체 상태 |
| 다단계 롤아웃 | 지원 (5 step) | 1 step | 제한적 |

### Source References

- Definition: [Section 3.2 Web World Model](../2601.22149.md#32-web-world-model)
- Training Loss: [Equation 4](../2601.22149.md#32-web-world-model)
- Evaluation: [Section 5.3](../2601.22149.md#53-essential-role-of-wm-training)

---

## Contribution 2: Imagination-driven Online RL

### What

월드 모델이 생성한 상상 궤적을 온라인 정책 최적화의 직접적인 학습 데이터로 활용하는 패러다임. 기존 연구들이 월드 모델을 추론 시점 보조 도구(WebDreamer)나 오프라인 데이터 합성기(Explorer)로만 활용한 것과 근본적으로 다르다.

### Technical Details

- 에이전트 $\pi_\theta$가 월드 모델 $p_\phi$와 상호작용하여 다단계 궤적 $\hat{\tau}$ 생성
- 각 step에서: 에이전트 행동 $a_t \sim \pi_\theta$ → 월드 모델 예측 $\hat{o}_{t+1} \sim p_\phi$
- 궤적 종료 후 자기 평가로 보상 $\hat{r}(\hat{\tau}, q) \in \{0, 1\}$ 부여
- 이 상상 궤적이 policy gradient 계산의 직접적 입력으로 사용

### Why It Matters

- **학술적**: MBRL의 핵심 아이디어인 "환경 모델을 통한 학습"을 LLM 에이전트 영역으로 확장. Dyna(1991)와 Dreamer(2020)의 사상적 계보를 잇는 연구.
- **실용적**: 실제 웹과의 상호작용 없이도 on-policy 학습의 이점(탐색, 자기 교정)을 확보할 수 있어, 웹 에이전트 RL의 확장성 병목을 해소.

### Source References

- Imagined Rollout Process: [Section 3.3, Equations 5-7](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents)
- vs Prior Work: [Section 2 Related Work](../2601.22149.md#2-related-work)

---

## Contribution 3: DynaWeb Framework

### What

상상 롤아웃과 실제 전문가 궤적을 무작위로 인터리빙하고, GSPO(Group Sequence Policy Optimization)로 정책을 최적화하는 통합 MBRL 프레임워크.

### Technical Details

- **전문가 인터리빙**: 훈련 중 50% 확률로 상상 궤적을 실제 전문가 궤적으로 교체
- **GSPO**: PPO의 토큰 수준 importance ratio를 시퀀스 수준 기하 평균으로 대체
  - 시퀀스 ratio: $s^i(\theta) = \exp\left(\frac{1}{|y^i|}\sum_k \log r_k^i(\theta)\right)$
  - 클리핑된 목적 함수로 안정적 업데이트
- **초기 상태 다양성**: NNetNav SFT 데이터의 초기/중간 상태를 무작위 샘플링하여 다양한 웹 상태에서 시작
- **조기 종료**: 월드 모델이 terminal state 생성 시 롤아웃 즉시 종료

### Why It Matters

- **학술적**: 상상 경험과 실제 경험의 최적 혼합 비율(~40% 실제)이라는 실증적 발견은 MBRL 이론에 대한 실용적 기여
- **실용적**: 기존 SFT 데이터를 그대로 재활용할 수 있어 추가 데이터 수집 불필요. 단순하지만 효과적인 인터리빙 전략은 구현이 용이.

### Source References

- GSPO: [Equations 8-10](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents)
- Expert Interleaving: [Section 3.3](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents)
- Hyperparameters: [Table 3, Appendix A.3](../2601.22149.md#a3-training-recipe-and-hyperparameters)

---

## Contribution 4: Empirical Design Principles

### What

MBRL 웹 에이전트의 핵심 설계 변수 3가지에 대한 체계적 ablation 분석을 통해, 후속 연구를 위한 실증적 가이드라인 제공.

### Technical Details

1. **드림 길이 분석** (Section 5.1): 최대 1~10 step 범위에서 실험. 4-5 step에서 성능 정점, 이후 월드 모델 환각으로 하락.
2. **실제 데이터 비율** (Section 5.2): 0~100% 범위 실험. 0%는 SFT 이하, ~40%에서 최적, 이후 수확 체감.
3. **월드 모델 학습 효과** (Section 5.3): 전용 학습 WM vs 범용 GPT-oss-120b 프롬프팅. WebArena 31.0 vs 20.9, WebVoyager 35.4 vs 28.6으로 학습 필수성 입증.

### Why It Matters

- **학술적**: MBRL에서 "환경 모델의 충실도"와 "롤아웃 깊이"의 트레이드오프를 웹 도메인에서 정량적으로 규명
- **실용적**: 후속 연구자가 DynaWeb 변형을 설계할 때 참고할 수 있는 구체적 수치와 경향성 제공

### Source References

- Dream Length: [Section 5.1, Figure 3a](../2601.22149.md#51-wm-dream-length-matters)
- Data Ratio: [Section 5.2, Figure 3b](../2601.22149.md#52-effect-of-real-interaction-data-percentage)
- WM Training: [Section 5.3, Table 4](../2601.22149.md#53-essential-role-of-wm-training)

---

## Contribution Analysis

### Novelty Assessment

| Contribution | Novelty Level | Justification |
|--------------|---------------|---------------|
| 1. Web World Model | Medium | Delta 예측은 새롭지만 LLM 기반 WM 자체는 선행 연구 존재 |
| 2. Imagination-driven Online RL | High | 웹 에이전트에서 상상 궤적을 온라인 RL에 활용한 최초 |
| 3. DynaWeb Framework | Medium-High | 개별 구성요소는 기존이지만, 통합 프레임워크로서의 효과적 결합이 새로움 |
| 4. Design Principles | Medium | 체계적 ablation 자체는 표준적이지만, 웹 MBRL 특화 분석은 최초 |

### Impact Assessment

| Contribution | Short-term Impact | Long-term Impact |
|--------------|-------------------|------------------|
| 1. Web World Model | 웹 시뮬레이션 품질 기준선 제시 | 범용 웹 시뮬레이터 개발의 기반 |
| 2. Imagination-driven Online RL | 안전한 웹 에이전트 RL 훈련 방법 제공 | MBRL-for-agents 연구 분야 개척 |
| 3. DynaWeb Framework | 즉시 적용 가능한 실용적 프레임워크 | 다양한 에이전트 도메인으로 확장 |
| 4. Design Principles | 후속 연구 설계 가이드 | 이론적 분석으로의 발전 기반 |

### Dependencies

```
Contribution 1 (Web World Model)
    └── enables → Contribution 2 (Imagination-driven Online RL)
                      └── enables → Contribution 3 (DynaWeb Framework)
                                        └── enables → Contribution 4 (Design Principles)
```

---

## Source References Summary

| Contribution | Primary Source | Supporting Sources |
|--------------|----------------|-------------------|
| 1. Web World Model | [Section 3.2](../2601.22149.md#32-web-world-model) | [Section 5.3](../2601.22149.md#53-essential-role-of-wm-training) |
| 2. Imagination-driven RL | [Section 3.3](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents) | [Section 2](../2601.22149.md#2-related-work) |
| 3. DynaWeb Framework | [Section 3.3](../2601.22149.md#33-dynaweb-model-based-rl-of-web-agents) | [Section 4](../2601.22149.md#4-experiments) |
| 4. Design Principles | [Section 5](../2601.22149.md#5-analysis) | [Appendix A.3](../2601.22149.md#a3-training-recipe-and-hyperparameters) |

---

## Navigation

<- [Back to Main Analysis](../2601.22149-analysis.md)
