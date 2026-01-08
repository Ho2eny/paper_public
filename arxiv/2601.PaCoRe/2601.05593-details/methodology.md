---
parent: ../2601.05593-analysis.md
source: ../2601.05593.md
---

# Methodology - Deep Analysis

## Paper Type: Method

## Approach Overview

PaCoRe는 **추론(inference) 파이프라인**과 **훈련(training) 절차** 두 축으로 구성된다.

**Inference**: 다라운드 병렬 조율 추론. 각 라운드에서 $K_r$개의 병렬 궤적을 생성하고, 궤적의 최종 결론만 추출(compaction)하여 다음 라운드에 전달. 마지막 라운드에서 단일 궤적으로 최종 답 생성.

**Training**: 단일 PaCoRe 라운드를 에피소딕 RL 환경으로 모델링. 모델(policy)은 문제 $x$와 메시지 세트 $M$을 받아 추론 궤적을 생성하고, 정답 여부에 따른 sparse reward를 받음. PPO로 최적화.

## Inference Pipeline Detail

### Round 구조

```
Round 1: Problem x → K₁ parallel trajectories → Compact to M₁
Round 2: (x, M₁) → K₂ parallel trajectories → Compact to M₂
...
Final:   (x, M_R̂) → 1 trajectory → Final answer y
```

### 단계별 분석

**1. Synthesis and Parallel Exploration**

입력 구성: prompting function $P(x, M_{r-1})$로 문제와 이전 라운드 메시지를 직렬화.

궤적 생성: 동일 모델 $\pi_r$을 $K_r$번 독립적으로 호출:

$$\omega_r^{(i)} \sim \pi_r(\cdot \mid P(x, M_{r-1})), \quad i = 1, \ldots, K_r$$

핵심 통찰: 각 호출의 컨텍스트 비용 = $|x| + \sum_j |m_{r-1}^{(j)}|$ (상수 수준), 유효 TTC = $\sum_i |\omega_r^{(i)}|$ (선형 확장).

**2. Message Compaction**

$$M_r = \mathcal{C}(\Omega_r)$$

구현: trajectory-wise 추출 $m_r^{(i)} = C(\omega_r^{(i)})$ --- 각 궤적에서 최종 결론 세그먼트만 보존, 중간 추론(CoT) 버림.

설계 근거: Reasoning model은 구조적으로 `<thinking>...</thinking><answer>...</answer>` 형태. Answer 부분만 추출하면 자연스러운 압축.

**3. Iterative Coordination**

$R$번 반복 후 최종 라운드에서 $K_R = 1$로 단일 답 생성:

$$y = m_R^{(1)}$$

### Inference Configuration

$\vec{K} = [K_1, \ldots, K_{\hat{R}}]$로 각 라운드의 병렬 폭 지정:

| Setting | Configuration | Effective TTC (HMMT) |
|---------|---------------|---------------------|
| Low     | $\vec{K} = [4]$    | ~243k tokens |
| Medium  | $\vec{K} = [16]$   | ~869k tokens |
| High    | $\vec{K} = [32, 4]$ | ~1,796k tokens |

High 설정 분석: 1라운드에서 32개 궤적 → 압축 → 2라운드에서 4개 궤적 → 압축 → 최종 1개.

## Training Pipeline Detail

### RL Environment

- **State**: formatted input $P(x, M)$
- **Action**: 추론 궤적 $\omega$ (토큰 시퀀스)
- **Reward**: sparse terminal $R(\omega) \in [0, 1]$ (정답 여부)
- **Policy**: $\pi_\theta$

### 2-Stage 커리큘럼

**Stage 1 (250 iterations)**:
- 목표: naive aggregation이 실패하는 케이스에서 훈련
- 필터: $0 < \text{mean}(\text{message\_acc}) < 9/24$ (수학), $< 15/24$ (코딩)
- 직관: "다수결이 안 통하는 어려운 문제만 학습"

**Stage 2 (450 iterations)**:
- 목표: Stage 1 체크포인트가 아직 풀지 못하는 문제에 집중
- 필터: $0 < \text{synthesis\_acc} < 1$ (완벽히 풀거나 전혀 못 푸는 문제 제외)
- 직관: "배울 여지가 있는 문제에 집중"

### 핵심 하이퍼파라미터

| Parameter | Value |
|-----------|-------|
| Base model | Qwen3-8B-Base → RLVR-8B |
| Algorithm | Strict on-policy PPO + GAE |
| $\lambda$, $\gamma$ | 1.0, 1.0 |
| Batch size | 16 instances × 64 responses |
| Max sequence length | 131,072 |
| Temperature / Top-p | 1.0 / 1.0 |
| Message pool size | 24 per problem |
| $\|M\|$ sampling | $U(16, 24)$ |
| Total iterations | 700 (250 + 450) |

## Key Design Choices

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| Compaction 방식 | 결론만 추출 (CoT 버림) | Reasoning model의 구조적 특성 활용; 학습 불필요 | [Section 2.1](../2601.05593.md#21-inference-pipeline) |
| 동일 모델 가중치 | 모든 라운드에서 같은 $\pi$ | 운영 단순화; 라운드별 전문화 불필요 | [Section 2.1](../2601.05593.md#21-inference-pipeline) |
| Sparse reward | 최종 정답 여부만 보상 | Process reward보다 단순; 대규모 RL에 적합 | [Section 2.2](../2601.05593.md#22-training-procedure) |
| Message accuracy 필터 | 쉬운 문제 제외 | Majority voting으로 풀리는 문제에선 synthesis 학습 불가 | [Section 3.1](../2601.05593.md#31-training) |
| $\|M\|$ 랜덤 샘플링 | $U(8, 16)$이 최적 | 고정 크기보다 다양한 병렬 설정에 대한 로버스트니스 향상 | [Table 5](../2601.05593.md#table-5) |
| Cached message pool | 오프라인 생성 후 랜덤 샘플링 | 훈련 효율 (매번 생성 불필요) | [Section 3.1](../2601.05593.md#31-training) |
| $K_2 = 4$ (multi-round) | 2라운드 폭 4 최적 | $K_2 = 2$나 $8$보다 두 벤치마크 모두에서 우수 | [Table 8](../2601.05593.md#table-8) |

## Computational Cost Analysis

| Model | HMMT Score | TTC per Problem | Cost Ratio vs GPT-5 |
|-------|-----------|-----------------|---------------------|
| GPT-5 | 93.2% | 16k | 1× |
| PaCoRe Low | 88.1% | 243k | 15× |
| PaCoRe Medium | 92.9% | 869k | 54× |
| PaCoRe High | 94.5% | 1,796k | 112× |

PaCoRe는 GPT-5보다 정확도는 높지만 TTC가 2 자릿수 이상 많다. 이는 "compute-for-accuracy" trade-off가 명확함을 의미.

## Source References

- Inference Pipeline: [Section 2.1](../2601.05593.md#21-inference-pipeline)
- Training Procedure: [Section 2.2](../2601.05593.md#22-training-procedure)
- Training Recipe: [Section 3.1](../2601.05593.md#31-training)
- Base Model Derivation: [Appendix A](../2601.05593.md#a-initial-checkpoint-derivation)
- Prompt Template: [Appendix B](../2601.05593.md#b-pacore-synthesis-prompt-template)
- Data Preparation: [Appendix C](../2601.05593.md#c-pacore-training-data-preparation)
