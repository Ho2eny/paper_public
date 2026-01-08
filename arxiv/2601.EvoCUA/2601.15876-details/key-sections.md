```yaml
---
parent: ../2601.15876-analysis.md
source: ../2601.15876.md
---
```

# Key vs Non-Key Sections - Deep Analysis

## ⭐⭐⭐ Must Read

### Section 5: Evolving Paradigm via Learning from Experience
- **Why**: 논문의 핵심 방법론. Cold Start → RFT → Step-Level DPO의 3단계 진화적 학습이 이 논문의 가장 중요한 기여.
- **Key Points**:
  - Cold Start는 "가볍게"가 핵심 — 1K 궤적으로 패턴 다양성 확보가 대량 데이터보다 효과적
  - RFT의 Dynamic Compute Budgeting으로 경계 작업에 자원 집중
  - Step-Level DPO의 Critical Forking Point 발견과 이중 패러다임(Action Correction + Reflection)
  - Hindsight Reasoning Generation으로 물리적 궤적에 인지적 체인 보강
- **Reading time**: ~15분
- **Link**: [Section 5](../2601.15876.md#5-evolving-paradigm-via-learning-from-experience)

### Section 6.2: Main Results
- **Why**: SOTA 성능의 직접적 근거. 오픈/클로즈드 모델 비교와 스케일 효율성 분석.
- **Key Points**:
  - EvoCUA-32B: 56.7% (오픈 SOTA), 50스텝 제약 하
  - vs OpenCUA-72B: +11.7% 절대 개선
  - vs UI-TARS-2: +3.6% (클로즈드 모델 초과)
  - EvoCUA-8B: 46.1%, 같은 backbone의 Step-GUI-8B(40.2%) 대비 +5.9%
  - 일반 벤치마크(Table 2)에서의 성능 하락 분석
- **Reading time**: ~10분
- **Link**: [Section 6.2](../2601.15876.md#62-main-results)

### Section 3: Verifiable Synthesis Engine
- **Why**: 데이터 병목 해소의 핵심. "Generation-as-Validation" 패러다임이 검증 가능한 합성 데이터를 대규모로 생성하는 방법.
- **Key Points**:
  - 계층적 도메인 분류 → 원자적 능력 분해 → 조합적 작업 생성
  - ReAct 기반 이중 스트림(지시문 + 검증기) 동시 생성
  - 폐루프 피드백으로 실행 가능성 보장
  - 삼중 오염 제거(semantic, configuration, evaluator)
- **Reading time**: ~10분
- **Link**: [Section 3](../2601.15876.md#3-verifiable-synthesis-engine)

---

## ⭐⭐ Important

### Section 6.3: Ablation Study
- **Why**: 각 컴포넌트의 개별 기여도를 정량적으로 분리. 어떤 요소가 얼마나 중요한지 이해하는 데 필수.
- **Key Points**:
  - EvoCUA-32B: Action Space(+4.84%), Cold Start(+2.62%), RFT(+3.13%), DPO(+3.21%), Iterative(+1.90%)
  - OpenCUA-72B: 동일 패러다임이 다른 모델에서도 일관된 효과
  - Pure RFT 3라운드 1M 데이터로 +8.12% — 강한 base model에서는 합성 엔진만으로 대폭 개선 가능
- **Link**: [Section 6.3](../2601.15876.md#63-ablation-study)

### Section 6.4: Scaling Analysis
- **Why**: Pass@k, Max Steps, 데이터 볼륨에 대한 스케일링 법칙 분석. 향후 확장 가능성 판단에 중요.
- **Key Points**:
  - Pass@k: k=16에서 +4.93% 피크, 높은 k에서도 유지 → 성능 천장 자체를 높임
  - Max Steps: 15→50 스텝에서 32B +16.25%, 50 이후 완화
  - Experience: 20K→226K→1M으로 RFT 스케일링, +2.61→+6.79→+8.12
- **Link**: [Section 6.4](../2601.15876.md#64-scaling-analysis)

### Section 6.5: Discussions
- **Why**: 1,000+ 실험의 결정적 인사이트. 후속 연구자에게 가장 실용적인 가이드.
- **Key Points**:
  - 경험의 이중성: 성공은 저잡음/저정보, 실패는 고잡음/고정보
  - 초기화: 가벼운 Cold Start + 강력한 RFT/DPO가 무거운 Cold Start보다 우수
  - On-policy 필수: Off-policy 데이터가 최적화 벡터의 주방향을 교란
  - 종료 비대칭성: 실패 인식은 빠르나 성공 인식은 양성 샘플 밀도에 민감
- **Link**: [Section 6.5](../2601.15876.md#65-discussions)

### Section 7: Future Work (STEPO)
- **Why**: 온라인 에이전트 RL의 학습-추론 불일치 문제 분석과 STEPO 알고리즘 제안. 아직 예비 단계이나 중요한 방향성.
- **Key Points**:
  - GRPO의 궤적 수준 학습은 GUI 태스크에서 training-inference discrepancy 야기
  - STEPO: 궤적 advantage를 스텝별 균등 분배, 스텝 수준 PPO 클리핑
  - OpenCUA-32B에서 GRPO 대비 유의미한 성능 우위
- **Link**: [Section 7](../2601.15876.md#7-future-work-on-online-agentic-rl)

---

## ⭐ Reference

### Section 4: Scalable Interaction Infrastructure
- **Why**: 시스템 엔지니어링 관심이 있다면 참조. QEMU-KVM 하이브리드 가상화, 비동기 오케스트레이션, 결정론적 환경 보정의 상세.
- **When to read**: 유사 인프라를 구축하거나 환경 결정론성 문제를 다룰 때
- **Link**: [Section 4](../2601.15876.md#4-scalable-interaction-infrastructure)

### Section 2: Preliminaries
- **Why**: POMDP 정형화와 최적화 목표의 수학적 기반. CUA의 형식적 정의에 관심 있을 때.
- **When to read**: 수학적 프레임워크를 정확히 이해하고 싶을 때
- **Link**: [Section 2](../2601.15876.md#2-preliminaries)

### Appendix A-D
- **Why**: 행동 공간 상세(A), Hindsight Reasoning Generation 알고리즘(B), DPO 쌍 구성 알고리즘(C), 궤적 시각화(D)
- **When to read**: 구현 레벨의 상세가 필요할 때
- **Link**: [Appendix A](../2601.15876.md#appendix-a-unified-action-space), [Appendix B](../2601.15876.md#appendix-b-cold-start-hindsight-reasoning-generation), [Appendix C](../2601.15876.md#appendix-c-algorithm-for-dpo), [Appendix D](../2601.15876.md#appendix-d-trajectory-analysis-and-visualization)

---

## Skip

### Section 8: Related Work
- **Why**: GUI 에이전트, VLM, 강화학습 분야에 이미 익숙한 독자에게는 새로운 정보가 적다.
- **Exception**: CUA 분야 입문자라면 배경 지식으로 읽을 가치 있음
- **Link**: [Section 8](../2601.15876.md#8-related-work)
