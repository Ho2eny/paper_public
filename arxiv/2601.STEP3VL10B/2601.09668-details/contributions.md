---
parent: ../2601.09668-analysis.md
source: ../2601.09668.md
created: 2026-02-06
---

# Core Contributions - Deep Analysis

## Contribution 1: 통합 Unfrozen 사전학습

### What
PE-lang(1.8B Perception Encoder, 언어 정렬 버전)과 Qwen3-8B 디코더를 완전히 unfrozen 상태에서 1.2T 멀티모달 토큰으로 단일 단계 학습. 7개 도메인(Knowledge, Education, OCR, Grounding/Counting, VQA, GUI, 텍스트)의 대규모 데이터로 reasoning과 perception을 동시에 구축한다.

### Technical Details
- **Vision Encoder**: PE-lang 선택 (DINOv3 대비 OCRBench +12.5% 우위, Table 4)
- **Multi-crop 전략**: 728×728 글로벌 뷰 + 504×504 로컬 크롭
- **Projector**: 16× spatial downsampling (stride-2 레이어 2개)
- **2단계 LR 스케줄**: Phase 1(900B tokens, $5\times10^{-5}$ → $1\times10^{-5}$) + Phase 2(300B tokens, $1\times10^{-5}$ → $6\times10^{-6}$)
- **데이터 규모**: Knowledge(interleaved + pairs), Education(15M), OCR(Image 40M + Code 30M+ + Doc 80M), Grounding(400M), VQA(30M), GUI(23M)

### Why It Matters
- Frozen encoder 패러다임을 벗어나 vision-language synergy를 학습 단계에서 내재화
- PE-lang의 언어 사전 정렬이 modality gap을 줄여 수렴 속도와 최종 성능 모두 개선
- 단일 단계 학습으로 파이프라인 복잡성 감소

### Source
- Architecture: [2.1 Architecture](../2601.09668.md#21-architecture)
- Data: [2.2 Data Construction](../2601.09668.md#22-data-construction)
- Training: [2.3 Training Recipe](../2601.09668.md#23-training-recipe)
- Ablation: [5.1 Ablations](../2601.09668.md#51-ablations-and-design-insights)

---

## Contribution 2: 대규모 멀티모달 RL 파이프라인

### What
검증 가능한 보상(RLVR)과 인간 선호 보상(RLHF)을 결합한 체계적 3단계 RL 파이프라인. PPO+GAE를 기반으로 1,400+ iterations의 대규모 RL 학습을 수행한다.

### Technical Details

**RLVR (600 iterations)**:
- 데이터 필터링 3축: Checkability(GPT-OSS-120B 4회 검증), Visual Relevance(의미적 상관), Difficulty(24 롤아웃 some-accept)
- 512 prompts × 16 rollouts/iteration, max 24K tokens
- Perception reward(IoU/Euclidean) + Model-based verification

**RLHF (300 iterations)**:
- GenRM: 쌍별 선호 + 추론 판단 기반 스칼라 점수
- Behavioral regularization: 언어 일관성 패널티, 인용 검증(환각 방지), 인식론적 교정
- 512 prompts × 8 rollouts/iteration, max 32K tokens

**PPO 설정**:
- GAE($\gamma=1, \lambda=1$), off-policy, 4 minibatches/iteration
- Actor LR $2\times10^{-6}$, Critic LR $5\times10^{-6}$
- Truncated importance sampling (threshold $C=8$)
- 디코더만 업데이트, 인코더 frozen

### Why It Matters
- 멀티모달 도메인에서 1,400+ RL iterations는 최대 규모 중 하나
- 이원 보상 체계(verifiable + non-verifiable)로 추론 정확성과 인간 정렬을 동시에 달성
- Behavioral regularization이 reward hacking과 hallucination을 실효적으로 억제

### Source
- Optimization: [3.2.1 Optimization Algorithm](../2601.09668.md#321-optimization-algorithm)
- Rewards: [3.2.2 Reward System](../2601.09668.md#322-reward-system)
- RLVR: [3.2.3 Scaling Sequential Reasoning](../2601.09668.md#323-scaling-sequential-reasoning)

---

## Contribution 3: PaCoRe (Parallel Coordinated Reasoning)

### What
16개 순차 추론(SeRe) 롤아웃을 병렬로 생성한 후, 이를 컨텍스트로 통합하여 최종 추론을 수행하는 테스트 타임 컴퓨트 확장 전략. 훈련 시에도 PPO on-policy로 500 iterations 최적화.

### Technical Details
- **추론 시**: 16개 SeRe 롤아웃 → 통합 컨텍스트(최대 131,072 tokens) → 최종 응답 생성
- **훈련 데이터 구축**: RLVR의 24 롤아웃 message cache pool → Synthesis Filtration (some-accept 유지)
- **훈련**: PPO on-policy, 500 iterations, 64 prompts × 16 rollouts, max 64K tokens
- **핵심 동작**: 모델이 다수의 지각 가설(proposals)을 생성 → 순차적 교차 검증 → 최종 통합

### Why It Matters
- RL로 scaling이 어려운 인식 태스크("Missing Trace" 문제)에 대한 실효적 우회 솔루션
- Faster R-CNN의 Region Proposal 철학을 LLM 추론에 적용한 독창적 유추
- 수학 추론뿐 아니라 시각적 인식(counting +4.6%, spatial +7.5%)에서도 유의미한 향상

### Source
- Method: [3.2.4 Further Scaling Parallel Coordinated Reasoning](../2601.09668.md#324-further-scaling-parallel-coordinated-reasoning)
- Results: [4.4 Comparison with Larger Models](../2601.09668.md#44-comparison-with-larger-models)
- Analysis: [5.2 RL Dynamics](../2601.09668.md#52-rl-dynamics-performance-and-emergence)

---

## Contribution 4: RL 학습 역학 분석 + "Missing Trace" 가설

### What
RLVR 600 iterations 동안의 학습 역학을 체계적으로 분석하여, 추론 태스크와 인식 태스크에서 RL이 상반된 길이 역학을 보임을 발견. 인식 태스크에서 RL scaling이 제한되는 원인을 "Missing Trace" 가설로 설명.

### Technical Details
- **이중 길이 역학**: Reasoning은 표준 sequential scaling(길이 증가), Perception은 length diminishment(길이 감소)
- **Cancellation Effect**: 두 역학이 상충하여 평균 롤아웃 길이가 초기 증가 후 원래 수준으로 복귀
- **인식의 길이 감소 원인**: RL이 entropy reduction을 유도 → 탐색 토큰 제거 → Pass@N → Pass@1 전환
- **"Missing Trace" 가설**: 인간의 시각적 인지 과정(glance-and-focus, try-error-correct)이 학습 데이터에 명시적으로 기술되지 않음 → RL 최적화 landscape에 "인지 흔적"이 부재 → 자발적 지각 추론 유도 불가

### Why It Matters
- 멀티모달 RL의 근본적 한계에 대한 첫 체계적 분석
- "Missing Trace"는 향후 데이터 구축(인식 과정의 명시적 기술)과 학습 전략 연구의 중요한 방향 제시
- System 2 → System 1 압축(self-distillation)이라는 장기 비전 제시

### Source
- Training Dynamics: [5.2 RL Dynamics, Performance, and Emergence](../2601.09668.md#52-rl-dynamics-performance-and-emergence)
- Future Work: [6. Conclusion and Future Work](../2601.09668.md#6-conclusion-and-future-work)

---

## Contributions Summary

| # | Contribution | Impact | Source |
|---|-------------|--------|--------|
| 1 | 통합 Unfrozen 사전학습 | Vision-language synergy 내재화 | [Sec 2](../2601.09668.md#2-pre-train) |
| 2 | 대규모 멀티모달 RL 파이프라인 | 추론 + 정렬 동시 개선 | [Sec 3.2](../2601.09668.md#32-reinforcement-learning) |
| 3 | PaCoRe | 10B → 100B+ 수준 성능 달성 | [Sec 3.2.4](../2601.09668.md#324-further-scaling-parallel-coordinated-reasoning) |
| 4 | RL 역학 분석 + Missing Trace | 멀티모달 RL 연구 방향 제시 | [Sec 5.2](../2601.09668.md#52-rl-dynamics-performance-and-emergence) |
