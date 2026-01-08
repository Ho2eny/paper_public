---
parent: ../2601.09668-analysis.md
source: ../2601.09668.md
created: 2026-02-06
---

# Methodology - Deep Analysis

## Paper Type
**Method** - 경량 멀티모달 모델의 학습 전략 및 테스트 타임 스케일링

---

## Approach Overview

### High-Level Summary

STEP3-VL-10B의 방법론은 크게 3단계로 구성된다:

**Stage 1 - 사전학습**: PE-lang(1.8B) + Qwen3-8B를 완전 unfrozen으로 1.2T 멀티모달 토큰 학습. 7개 도메인의 데이터로 reasoning과 perception 기초를 동시에 구축한다. 2단계 LR 스케줄로 광역 학습(900B) → 고품질 데이터 집중 학습(300B)을 수행.

**Stage 2 - 후훈련(SFT + RL)**: 2단계 SFT(text-dominant 9:1 → multimodal 1:1, 총 226B 토큰)로 추론 능력을 정렬한 후, PPO+GAE 기반 RL을 1,400+ iterations 수행. RLVR(600 iter, 검증 가능한 태스크)로 추론 강건성을, RLHF(300 iter, 오픈엔드 태스크)로 인간 정렬을 달성.

**Stage 3 - PaCoRe 학습 및 추론**: 병렬 추론 합성 훈련(500 iter)으로 모델이 다수의 지각 가설을 생성·통합하는 능력을 학습. 추론 시 16개 SeRe 롤아웃을 통합하여 테스트 타임 컴퓨트를 확장.

### Key Innovation

PaCoRe는 RL로 scaling이 어려운 인식 태스크의 "Missing Trace" 문제를 우회한다. 인간의 시각적 인지 과정(glance-and-focus)이 학습 데이터에 명시적으로 기술되지 않아 RL이 자발적 지각 추론을 유도하기 어렵다는 통찰에 기반, 다수의 병렬 가설을 생성한 후 순차적 교차 검증으로 통합하는 전략을 채택한다.

---

## Technical Details

### Problem Formulation
- **Input**: 이미지 $I$ + 텍스트 프롬프트 $q$
- **Output**: 응답 궤적 $\tau = (s_0, a_0, \ldots, s_{T-1}, a_{T-1})$
- **Constraints**: 10B 파라미터 예산, 실용적 추론 비용

### Architecture

```
[Image] → PE-lang (1.8B) → Projector (16× downsample) ──┐
                                                          ├→ Qwen3-8B Decoder → Output
[Text]  → Tokenizer ──────────────────────────────────────┘
```

| Component | Specification |
|-----------|--------------|
| Vision Encoder | PE-lang 1.8B (language-optimized) |
| LLM Decoder | Qwen3-8B |
| Projector | 2× stride-2 layers (16× spatial downsample) |
| Multi-crop | 728×728 global + 504×504 local crops |
| Positional Encoding | 1D RoPE + newline tokens |
| Total Parameters | ~10B |

- Source: [2.1 Architecture](../2601.09668.md#21-architecture)

### Training Pipeline

```
Pre-training (1.2T tokens)
  ├─ Phase 1: 900B tokens, LR 5e-5 → 1e-5
  └─ Phase 2: 300B tokens, LR 1e-5 → 6e-6 (고품질 데이터)
       ↓
SFT (226B tokens)
  ├─ Stage 1: text:multimodal = 9:1, 190B tokens
  └─ Stage 2: text:multimodal = 1:1, 36B tokens
       ↓
RLVR (600 iterations)
  └─ 512 prompts × 16 rollouts, max 24K tokens
       ↓
RLHF (300 iterations)
  └─ 512 prompts × 8 rollouts, max 32K tokens
       ↓
PaCoRe Training (500 iterations)
  └─ 64 prompts × 16 rollouts, max 64K tokens
```

- Source: [2.3 Training Recipe](../2601.09668.md#23-training-recipe), [3. Post-Train](../2601.09668.md#3-post-train)

### RL Reward System

| Type | Method | Application |
|------|--------|-------------|
| **Verifiable** | Perception Rewards | Grounding/pointing (IoU, Euclidean) |
| **Verifiable** | Model-based Verification | 일반 시각 태스크 (GPT-OSS-120B) |
| **Non-verifiable** | GenRM | 오픈엔드 생성 (쌍별 선호 + 추론 판단) |
| **Non-verifiable** | Behavioral Regularization | 안전성 (언어 일관성, 인용 검증, 인식론적 교정) |

- Source: [3.2.2 Reward System](../2601.09668.md#322-reward-system)

### PaCoRe: Inference Pipeline

```
Input (I, q)
     ↓
16× Parallel SeRe Rollouts
  ├─ Rollout 1: <think>...</think> answer_1
  ├─ Rollout 2: <think>...</think> answer_2
  ├─ ...
  └─ Rollout 16: <think>...</think> answer_16
     ↓
Synthesis Context (max 131K tokens)
  "Based on these 16 reference responses, provide your comprehensive solution"
     ↓
Final Response (cross-validation + synthesis)
```

- Source: [3.2.4 PaCoRe](../2601.09668.md#324-further-scaling-parallel-coordinated-reasoning)

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives Considered | Source |
|--------|----------|-----------|------------------------|--------|
| Vision Encoder | PE-lang (1.8B) | 언어 정렬로 수렴 속도↑, OCR +12.5% | DINOv3 (modality gap 문제) | [Table 4](../2601.09668.md#51-ablations-and-design-insights) |
| Optimizer | AdamW | 안정적 학습, Muon은 초기화 불일치 | Muon (tail-knowledge +6.48% 이지만 warmup 문제) | [Table 5](../2601.09668.md#51-ablations-and-design-insights) |
| Depth Scaling | No Deepstack | 벤치마크 성능 향상 미미 | Deepstack (수렴 가속이지만 실효 이득 없음) | [Table 6](../2601.09668.md#51-ablations-and-design-insights) |
| RL Algorithm | PPO + GAE ($\gamma=1, \lambda=1$) | 안정적 정책 학습, 표준적 | GRPO, REINFORCE | [Sec 3.2.1](../2601.09668.md#321-optimization-algorithm) |
| Test-time Scaling | PaCoRe (16 rollouts) | Missing Trace 우회, 인식+추론 동시 향상 | Best-of-N, Majority Voting | [Sec 3.2.4](../2601.09668.md#324-further-scaling-parallel-coordinated-reasoning) |
| Training Strategy | Single-stage unfrozen | Vision-language synergy 내재화 | Multi-stage frozen→unfrozen | [Sec 2.3](../2601.09668.md#23-training-recipe) |

---

## Limitations & Assumptions

### Assumptions
1. PE-lang의 언어 정렬이 multimodal 학습에서 modality gap을 충분히 줄인다 - [Sec 2.1](../2601.09668.md#21-architecture)
2. 검증 가능한 보상과 인간 선호 보상의 순차적 적용이 최적이다 (동시 학습 대비)
3. 16개 롤아웃이 PaCoRe에서 충분한 다양성을 제공한다

### Limitations
1. **추론 비용**: PaCoRe 모드는 16× SeRe + 131K 토큰 합성 → 단일 추론 대비 ~17× 비용
2. **데이터 재현성**: StepCrawl, 내부 교육자료, GPT-OSS-120B 기반 필터링 등 비공개 요소 다수
3. **Perception RL Scaling**: "Missing Trace"를 우회(PaCoRe)하지만 근본적으로 해결하지 않음
4. **평가 범위**: 비디오, 멀티턴 대화, 실시간 에이전트 시나리오 미평가
5. **RL 안정성**: 1,400+ iter PPO의 학습 안정성에 대한 상세 분석 부족 (보상 해킹 사례 등)

---

## Source References
- Architecture: [2.1 Architecture](../2601.09668.md#21-architecture)
- Data: [2.2 Data Construction](../2601.09668.md#22-data-construction)
- Pre-training: [2.3 Training Recipe](../2601.09668.md#23-training-recipe)
- Post-training: [3. Post-Train](../2601.09668.md#3-post-train)
- RL Details: [3.2 Reinforcement Learning](../2601.09668.md#32-reinforcement-learning)
- Ablations: [5.1 Ablations](../2601.09668.md#51-ablations-and-design-insights)
- Dynamics: [5.2 RL Dynamics](../2601.09668.md#52-rl-dynamics-performance-and-emergence)
