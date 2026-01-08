```yaml
---
parent: ../2601.15876-analysis.md
source: ../2601.15876.md
---
```

# Methodology - Deep Analysis

## Paper Type: Method

EvoCUA는 Computer Use Agent(CUA)를 위한 학습 방법론 논문으로, 데이터 생성-인프라-학습을 통합한 end-to-end 시스템을 제안한다.

## Approach Overview

EvoCUA의 방법론은 세 축의 통합 사이클로 구성된다:

```
[Verifiable Synthesis Engine] → 작업 + 검증기 생성
        ↓
[Scalable Interaction Infrastructure] → 대규모 경험 수집
        ↓
[Evolving Learning Paradigm] → 정책 최적화
        ↓
(갱신된 정책으로 다시 경험 수집 → 반복)
```

### Formal Framework: POMDP

CUA를 POMDP 튜플 $(\mathcal{S}, \mathcal{A}, \mathcal{Z}, \mathcal{O}, \mathcal{P}, \mathcal{R}_{syn})$으로 정형화:

| 요소 | 정의 | 특이점 |
|------|------|--------|
| $\mathcal{S}$ | 컴퓨터 시스템 상태 | 직접 관찰 불가, 스크린샷으로만 인지 |
| $\mathcal{O}$ | 시각적 관찰 $I_t \in \mathbb{R}^{H \times W \times 3}$ | 최근 5개 스크린샷만 유지 |
| $\mathcal{A}$ | Mouse + Keyboard + Control | Stateful Interaction(key_down/up 분리) 지원 |
| $\mathcal{Z}$ | 추론 공간 | 행동 전 자연어 추론 생성 필수 |
| $\pi_\theta$ | 정책 $\pi_\theta(z_t, a_t \mid h_t, o_t)$ | 추론→행동 순차적 생성 |
| $\mathcal{R}_{syn}$ | 검증 기반 보상 $\mathbb{I}[V_g(s_T) = \text{True}]$ | 이진, 희소, 종료 시점 |

**최적화 목표**: $J(\theta) = \mathbb{E}_{(g, V_g) \sim \mathcal{T}_{syn}(\cdot | \pi_{\text{old}})} \left[ \mathbb{E}_{\tau \sim \pi_\theta(\cdot | g)} [\mathcal{R}_{syn}(s_T; g)] \right]$

핵심: 작업 분포 $\mathcal{T}_{syn}$이 현재 정책 $\pi_{\text{old}}$에 따라 적응적으로 변화한다.

**Source**: [Section 2](../2601.15876.md#2-preliminaries)

---

## Key Design Choices

| Choice | Decision | Rationale | Source |
|--------|----------|-----------|--------|
| 보상 설계 | 이진 검증 보상 $\mathcal{R}_{syn}$ | 자연어 보상의 모호성 회피, 결정론적 감독 | [§3](../2601.15876.md#3-verifiable-synthesis-engine) |
| 행동 공간 | Stateful Interaction (key_down/up 분리) | shift+click 등 복합 조작 지원 필수, Ablation에서 +4.84% | [§5.1](../2601.15876.md#51-cold-start) |
| 추론 스키마 | 5단계 구조화된 Thought Space | Goal → Observation → Verification → Reflection → Termination | [§5.1](../2601.15876.md#51-cold-start) |
| Cold Start 규모 | ~1K 궤적 (경량) | 무거운 초기화는 후속 RL 정제를 어렵게 함 | [§6.5](../2601.15876.md#65-discussions) |
| 선호도 학습 단위 | Step-Level (vs Trajectory-Level) | 장기 수평 작업에서 상태 비정렬 문제 회피 | [§5.3](../2601.15876.md#53-reinforcement-learning) |
| 가상화 | QEMU-KVM in Docker | 커널 수준 격임 + 오케스트레이션 호환성 | [§4.3](../2601.15876.md#43-high-fidelity-environment-instantiation) |
| 데이터 정책 | Strictly On-Policy | Off-policy가 최적화 벡터의 주방향을 교란 | [§6.5](../2601.15876.md#65-discussions) |
| 경험 활용 | 성공(RFT) + 실패(DPO) 분리 처리 | 성공=저잡음/저정보, 실패=고잡음/고정보로 다른 처리 필요 | [§6.5](../2601.15876.md#65-discussions) |

---

## Technical Details

### 1. Verifiable Synthesis Engine

**파이프라인**: Task Space Construction → Dual-Stream Synthesis → Quality Assurance

**Generation-as-Validation 원리**: 작업 지시 $g$를 생성할 때 반드시 실행 가능한 검증기 $V_g$를 동시 생성. 검증기는 실제 샌드박스에서 즉시 실행되어 폐루프 피드백.

**데이터 오염 방지**:
1. *Semantic*: LLM 기반 필터로 벤치마크 쿼리와 의미적 동등성 제거
2. *Configuration*: 동일 앱 초기화 설정의 작업 가지치기
3. *Evaluator*: 생성된 성공 조건/GT 파일이 기존 평가 스크립트와 겹치지 않도록 검증

**규모**: 수만 건의 검증된 학습 인스턴스 생성

**Source**: [Section 3](../2601.15876.md#3-verifiable-synthesis-engine)

---

### 2. Scalable Interaction Infrastructure

**핵심 사양**:
- 하루 수십만 건 샌드박스 세션
- 하루 수백만 건 상호작용 요청
- 10만+ 동시 샌드박스 유지
- 1분 내 수만 개 샌드박스 부트스트래핑

**결정론적 환경 보정**:
- **HID 패칭**: `/usr/share/x11/xkb/symbols/pc`에서 심볼 충돌 해결 (예: US 레이아웃의 `<` vs `>` shift-state 오류)
- **렌더링 일관성**: `fc-cache`에 독점 폰트 주입
- **런타임 안정성**: 시스템 프록시, xsel, qpdf 사전 설치

**Source**: [Section 4](../2601.15876.md#4-scalable-interaction-infrastructure)

---

### 3. Cold Start

**목표**: 행동 공간과 추론 패턴의 구조적 프라이어 확립

**Reasoning Schema**:
| 단계 | 역할 | 적용 시점 |
|------|------|-----------|
| Goal Clarification ($z_0$) | 모호한 지시 명확화 + 글로벌 계획 | $t = 0$ |
| Observation Consistency ($z_{\text{obs}}$) | 시각적 관찰과 추론의 의미적 일관성 | 중간 스텝 |
| Self-Verification ($z_{\text{check}}$) | 종료 전 보조 상호작용으로 결과 확인 | 종료 직전 |
| Reflection ($z_{\text{reflect}}$) | 실패 궤적에서 오류 분석 + 자기 교정 | 오류 복구 시 |
| Reasoning-Augmented Termination ($z_T$) | 시각적 증거 기반 완료 판단 | 최종 스텝 |

**Hindsight Reasoning Generation**: 정답 경로를 알고 있는 상태에서 역방향으로 추론 체인을 생성. 1인칭 관점 강제, 픽셀 좌표 대신 의미적 UI 요소 기술.

**Source**: [Section 5.1](../2601.15876.md#51-cold-start), [Appendix B](../2601.15876.md#appendix-b-cold-start-hindsight-reasoning-generation)

---

### 4. Rejection Sampling Fine-Tuning (RFT)

**Dynamic Compute Budgeting**:

$$K^* = k_{i^*} \quad \text{where} \quad i^* = \min\{i \mid \text{SR}(k_i) \geq \tau_1\}$$

계층적 예산 스펙트럼 $K = \{k_1, \ldots, k_n\}$과 하강 성공률 임계값 $\Lambda = \{\tau_1, \ldots, \tau_n\}$. 쉬운 작업은 적은 예산으로, 경계 작업에 집중.

**Step-Level Denoising**:
- 판단 모델이 궤적 분석 → 중복 스텝 마스킹
- 실행 불가 작업: 모든 중간 행동 제거, 추론 + `terminate=failure`만 보존

**Experience Scaling 결과** (OpenCUA-72B):
| Round | Data | Gain |
|-------|------|------|
| 1 | 20K | +2.61% |
| 2 | 226K | +6.79% |
| 3 | 1M | +8.12% |

**Source**: [Section 5.2](../2601.15876.md#52-rejection-sampling-fine-tuning)

---

### 5. Step-Level DPO

**Causal Deviation Discovery**: 실패 궤적 $\tau^-$와 성공 참조 $\tau^+$를 비교하여 환경 상태가 기능적으로 동등함에도 행동이 처음 분기하는 지점 $t^*$ 식별.

**선호도 쌍 구성 (Dual-Paradigm)**:

| Paradigm | 시점 | Chosen | Rejected | 목적 |
|----------|------|--------|----------|------|
| **I: Action Correction** | $t^*$ | 참조 정렬 또는 VLM 합성 $(z_w, a_w)$ | 오류 응답 $(z_l, a_l)$ | 올바른 행동 학습 |
| **II: Reflection** | $t^*+1$ | 반성 추론 + 교정 계획 | 맹목적 다음 행동 | 오류 인식 + 복구 학습 |

**DPO Loss**: $\mathcal{J}(\theta) = -\mathbb{E} \left[ \log \sigma \left( \beta \log \frac{\pi_\theta(z_w, a_w | h_t, o_t)}{\pi_{\text{ref}}(z_w, a_w | h_t, o_t)} - \beta \log \frac{\pi_\theta(z_l, a_l | h_t, o_t)}{\pi_{\text{ref}}(z_l, a_l | h_t, o_t)} \right) \right]$

**Source**: [Section 5.3](../2601.15876.md#53-reinforcement-learning), [Appendix C](../2601.15876.md#appendix-c-algorithm-for-dpo)

---

### 6. STEPO (Step-Level Policy Optimization)

**문제**: GRPO의 궤적 수준 학습은 GUI 태스크에서 training-inference discrepancy 유발. 추론 시 최근 스텝만 완전 컨텍스트를 보유하나, 학습 시 최종 궤적으로 훈련하면 중간 스텝의 감독 신호 손실.

**해법**: 궤적 advantage $\hat{A}_i$를 스텝에 균등 배분:

$$\hat{A}_{i,t} = \hat{A}_i / T_i$$

**효과**:
1. 높은 advantage 궤적 → 더 적은 스텝으로 완료하도록 유도 (중복 감소)
2. 낮은 advantage 궤적 → 더 많은 탐색 스텝 유도 (완료율 향상)

**Source**: [Section 7](../2601.15876.md#7-future-work-on-online-agentic-rl)

---

## Training Configuration

| 항목 | 값 |
|------|-----|
| **Base Models** | Qwen3-VL-Thinking (8B, 32B), OpenCUA (7B, 32B, 72B) |
| **Cold Start Data** | ~1K 고품질 궤적 |
| **RFT Data** | 수만 건 → 최대 1M (3라운드) |
| **Context Window** | 최근 5 스크린샷 + 이전 이력은 텍스트 압축 |
| **Max Steps (Eval)** | 50 (EvoCUA) vs 100 (대부분 베이스라인) |
| **Evaluation Benchmark** | OSWorld-Verified (주), ScreenSpot-v2/Pro, OSWorld-G, MMMU, MathVista 등 |
| **Training Loss** | 현재 스텝의 추론+행동에만 계산 |
| **General Data** | 일반 멀티모달(STEM, OCR, 시각 그라운딩) 혼합 — 망각 방지 |

---

## Source References

- Formal Definition: [Section 2](../2601.15876.md#2-preliminaries)
- Synthesis Engine: [Section 3](../2601.15876.md#3-verifiable-synthesis-engine)
- Infrastructure: [Section 4](../2601.15876.md#4-scalable-interaction-infrastructure)
- Learning Paradigm: [Section 5](../2601.15876.md#5-evolving-paradigm-via-learning-from-experience)
- Evaluation: [Section 6](../2601.15876.md#6-evaluation)
- STEPO: [Section 7](../2601.15876.md#7-future-work-on-online-agentic-rl)
- Action Space: [Appendix A](../2601.15876.md#appendix-a-unified-action-space)
- Hindsight Reasoning: [Appendix B](../2601.15876.md#appendix-b-cold-start-hindsight-reasoning-generation)
- DPO Algorithm: [Appendix C](../2601.15876.md#appendix-c-algorithm-for-dpo)
