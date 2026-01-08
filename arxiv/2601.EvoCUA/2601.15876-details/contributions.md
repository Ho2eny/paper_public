```yaml
---
parent: ../2601.15876-analysis.md
source: ../2601.15876.md
---
```

# Core Contributions - Deep Analysis

## Contribution 1: Verifiable Synthesis Engine

**What**: 자연어 지시($g$)와 실행 가능한 검증기($V_g$)를 동시에 자동 생성하는 "Generation-as-Validation" 파이프라인. 세 단계로 구성된다:

1. **Structured Task Space Construction**: 핵심 데스크톱 앱들을 계층적으로 분류하고 사용자 행동을 *원자적 능력(atomic capabilities)*으로 분해. 파라메트릭 합성(코드 기반 문서 생성)과 비파라메트릭 주입(공개 인터넷 데이터)의 하이브리드 전략.
2. **Agentic Dual-Stream Synthesis**: ReAct 기반 VLM 아키텍트가 (역할, 능력, 리소스) 튜플에서 지시문과 검증기 코드를 이중 스트림으로 생성. 폐루프 피드백 메커니즘으로 실행 가능성 보장.
3. **Rigorous Quality Assurance**: 일관성 기반 필터링(참조 에이전트 롤아웃 + 보상 모델 + 수동 점검)과 삼중 오염 제거(의미적, 구성, 평가기).

**Why it matters**: 기존 CUA 학습의 가장 큰 병목인 수작업 데이터 수집을 해소한다. 검증 가능한 보상 $\mathcal{R}_{syn}(s_T; g) = \mathbb{I}[V_g(s_T) = \text{True}]$을 통해 자연어 보상의 모호성을 제거하고, 결정론적 감독 신호를 제공한다. 이를 통해 수만 건의 학습 인스턴스로 확장이 가능해졌다.

**Ablation Evidence**: Table 3에서 Unified Action Space(+4.84%)와 Cold Start(+2.62%)의 기여가 이 엔진이 생성한 데이터의 품질에 직접 의존한다.

**Source**: [Section 3](../2601.15876.md#3-verifiable-synthesis-engine)

---

## Contribution 2: Scalable Interaction Infrastructure

**What**: 대규모 경험 수집을 위한 산업 수준의 샌드박스 오케스트레이션 플랫폼. 핵심 구성:

1. **Architecture Abstractions**: *Tools*(불변 환경 정의)와 *Clusters*(동적 스케일링 단위)의 이중 추상화. 수백 가지 환경 유형 지원.
2. **High-Throughput Orchestration**: 리액터 패턴 기반 비동기 게이트웨이(분당 수십만 요청), 제어 플레인/데이터 플레인 분리, 1분 내 수만 개 샌드박스 부트스트래핑.
3. **High-Fidelity Environment**: QEMU-KVM 하이브리드 가상화(Docker 내 VM), 결정론적 환경 보정:
   - HID 패칭: xkb 커널 레벨에서 키 매핑 충돌 해결
   - 렌더링 일관성: 독점 폰트 주입으로 문서 렌더링 일치
   - 런타임 안정성: 프록시 구성, xsel/qpdf 등 사전 설치

**Why it matters**: On-policy 강화학습의 핵심 요구사항인 "정책 업데이트와 경험 수집 간 지연 최소화"를 만족시킨다. 하루 수십만 건의 샌드박스 세션과 수백만 건의 상호작용 요청을 처리하여, 이론적으로 가능했던 경험 스케일링을 실질적으로 구현했다. 10만+ 동시 샌드박스 유지 능력은 이 분야에서 가장 큰 규모의 인프라 중 하나다.

**Source**: [Section 4](../2601.15876.md#4-scalable-interaction-infrastructure)

---

## Contribution 3: Evolving Paradigm via Learning from Experience

**What**: 정적 모방에서 동적 경험 학습으로의 전환을 실현하는 3단계 진화적 학습 전략:

### Stage 1: Cold Start
- ~1K 고품질 궤적으로 행동 공간($\mathcal{A}$)과 추론 스키마($\mathcal{Z}$)의 구조적 프라이어 확립
- **Semantic Action Mapping**: 물리적 상호작용($\mathcal{A}_{\text{mouse}} \cup \mathcal{A}_{\text{keyboard}}$) + 제어 프리미티브($\mathcal{A}_{\text{control}}$) 통합
- **Reasoning Schema**: Goal Clarification → Observation Consistency → Self-Verification → Reflection → Reasoning-Augmented Termination
- **Hindsight Reasoning Generation**: 정답 경로를 미래 정보로 활용하여 추론 체인 역생성

### Stage 2: Rejection Sampling Fine-Tuning (RFT)
- **Dynamic Compute Budgeting**: 에이전트 능력에 따라 작업별 탐색 예산 적응적 할당. 쉬운 작업 가지치기, 경계 작업에 자원 집중.
- **Step-Level Denoising**: 판단 모델로 성공 궤적 내 중복 스텝 마스킹, 실행 불가 작업은 추론+종료만 보존.
- 수만 건의 고품질 궤적으로 경험 풀 $\mathcal{B}$ 확장.

### Stage 3: Step-Level DPO (Reinforcement Learning)
- **Causal Deviation Discovery**: 실패 궤적과 성공 참조에서 Critical Deviation Step $t^*$ 식별
- **Paradigm I (Action Correction)**: $t^*$에서의 선호도 쌍 — 올바른 vs 잘못된 (추론, 행동)
- **Paradigm II (Reflection & Recovery)**: $t^*+1$에서의 선호도 쌍 — 반성적 추론 vs 맹목적 계속
- DPO 손실로 정책 최적화, RFT→DPO 사이클 반복

**Why it matters**: 이 패러다임의 핵심 가치는 **일반화 가능성**이다:
- Qwen3-VL-32B에서: Action Space(+4.84%) → Cold Start(+2.62%) → RFT(+3.13%) → DPO(+3.21%) → Iterative(+1.90%) = 총 +15.7%
- OpenCUA-72B에서: Cold Start(+2.14%) → RFT(+3.69%) → DPO(+3.02%) → Iterative(+1.82%) = 총 +10.67%
- Pure RFT만으로도 OpenCUA-72B에서 3라운드 1M 데이터로 +8.12% 달성

"무엇을 잘하는지"(RFT)와 "무엇을 잘못하는지"(DPO)를 모두 학습하는 이중 메커니즘이, 모델 크기와 아키텍처에 관계없이 일관된 성능 향상을 제공한다.

**Source**: [Section 5](../2601.15876.md#5-evolving-paradigm-via-learning-from-experience)

---

## Additional Contribution: STEPO Algorithm (Future Work)

**What**: 온라인 RL의 학습-추론 불일치(training-inference discrepancy) 문제를 해결하기 위한 **Step-Level Policy Optimization** 알고리즘. GRPO의 궤적 수준 학습 대신, 궤적 advantage를 각 스텝에 균등 분배하여 스텝 수준 학습을 수행.

**Why it matters**: 아직 예비 단계이나, OpenCUA-32B에서 GRPO 대비 유의미하게 높은 학습 성능을 보여주어 온라인 에이전트 RL의 유망한 방향을 제시.

**Source**: [Section 7](../2601.15876.md#7-future-work-on-online-agentic-rl)
