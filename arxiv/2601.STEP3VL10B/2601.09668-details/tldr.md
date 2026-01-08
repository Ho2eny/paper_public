---
parent: ../2601.09668-analysis.md
source: ../2601.09668.md
created: 2026-02-06
---

# TL;DR - Deep Analysis

## Extended Summary

> STEP3-VL-10B은 StepFun이 개발한 10B 파라미터 오픈소스 멀티모달 모델이다. 1.8B Perception Encoder(언어 정렬 버전)와 Qwen3-8B 디코더를 완전히 unfrozen 상태에서 1.2T 멀티모달 토큰으로 통합 학습하여 비전-언어 시너지를 내재화한다. 이후 2단계 SFT → RLVR(600 iter) → RLHF(300 iter) → PaCoRe 훈련(500 iter)의 체계적 후훈련을 거친다. 테스트 시 PaCoRe(Parallel Coordinated Reasoning)를 통해 16개 병렬 추론을 통합함으로써 10B 규모에서 GLM-4.6V(106B), Qwen3-VL(235B), Gemini-2.5-Pro와 경쟁하거나 능가하는 성능을 달성한다.

## Problem Context

Multimodal LLM의 발전은 주로 scale에 의존해왔다. 프로프리어터리 모델(GPT-5.2, Gemini-3-Pro)은 massive scaling으로 높은 성능을 달성하지만 배포 비용이 막대하다. 반면 10B 이하 경량 모델은 "효율적이지만 제한적"이라는 인식이 있었다. STEP3-VL-10B은 이 이분법이 더 이상 유효하지 않음을 증명하려 한다.

- Source: [1. Introduction](../2601.09668.md#1-introduction)

## Solution Approach

두 가지 전략적 축:

1. **통합 사전학습**: PE-lang(1.8B) + Qwen3-8B를 frozen 없이 1.2T 토큰으로 단일 단계 학습. 2단계 학습률 스케줄(900B 광역 + 300B 고품질 데이터)로 reasoning과 perception 동시 강화.

2. **Scaled RL + Test-time Scaling**:
   - RLVR: 검증 가능한 보상(수학, 과학, 인식)으로 600 iter
   - RLHF: 인간 선호 보상(GenRM + 행동 정규화)으로 300 iter
   - PaCoRe: 병렬 추론 합성 훈련 500 iter → 추론 시 16개 롤아웃 통합

- Source: [2. Pre-train](../2601.09668.md#2-pre-train), [3. Post-Train](../2601.09668.md#3-post-train)

## Key Results

| Benchmark | STEP3-VL-10B (PaCoRe) | Best Competitor (100B+) | 격차 |
|-----------|----------------------|------------------------|------|
| AIME2025 | 94.43% | Gemini-2.5-Pro: 83.96% | +10.47 |
| MathVision | 75.95% | Gemini-2.5-Pro: 73.30% | +2.65 |
| MMMU | 80.11% | Seed-1.5-VL: 79.11% | +1.00 |
| HMMT25 | 92.14% | Qwen3-VL: 67.71% | +24.43 |
| MMBench EN | 92.38% | Gemini-2.5-Pro: 93.19% | -0.81 |

- Source: [4.4 Comparison with Larger Models](../2601.09668.md#44-comparison-with-larger-models)

## Why It Matters

**학술적 의의**:
- 모델 크기가 아닌 학습 전략(RL scaling + test-time compute)으로 성능 격차를 줄일 수 있음을 실증
- 멀티모달 RL에서의 길이 역학 이중 패턴 발견은 향후 연구의 중요한 출발점
- "Missing Trace" 가설은 왜 인식 태스크에서 RL scaling이 추론만큼 효과적이지 않은지에 대한 첫 체계적 설명

**실용적 의의**:
- 모델 가중치 공개 → 커뮤니티의 경량 모델 연구 가속화
- 10B 모델로 100B+ 수준 성능 → 추론 비용 대폭 절감 가능 (단, PaCoRe 비용 고려 필요)
- GUI/OCR/Grounding 등 실용적 태스크에서의 강력한 성능 → 에이전트 기반 응용 가능성

## Source References

- Problem: [1. Introduction](../2601.09668.md#1-introduction)
- Architecture: [2.1 Architecture](../2601.09668.md#21-architecture)
- Training: [2.3 Training Recipe](../2601.09668.md#23-training-recipe)
- RL Pipeline: [3.2 Reinforcement Learning](../2601.09668.md#32-reinforcement-learning)
- Results: [4.4 Comparison with Larger Models](../2601.09668.md#44-comparison-with-larger-models)
- Analysis: [5.2 RL Dynamics](../2601.09668.md#52-rl-dynamics-performance-and-emergence)
