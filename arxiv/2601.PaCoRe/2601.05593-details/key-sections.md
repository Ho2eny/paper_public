---
parent: ../2601.05593-analysis.md
source: ../2601.05593.md
---

# Key vs Non-Key Sections - Deep Analysis

## ⭐⭐⭐ Must Read

### Section 2.1: Inference Pipeline
- **Why**: PaCoRe의 핵심 설계. 병렬 탐색 → 메시지 압축 → 다라운드 조율이라는 전체 아키텍처를 이해하는 데 필수.
- **Key Points**:
  - 라운드별 궤적 생성: $\omega_r^{(i)} \sim \pi_r(\cdot \mid P(x, M_{r-1}))$
  - 압축 함수 $\mathcal{C}$: 중간 추론 버리고 최종 결론만 보존
  - 유효 TTC = $\sum_r \sum_i |\omega_r^{(i)}|$로 무한 확장 가능
  - 실제 컨텍스트 비용은 $|x| + \sum_i |m_r^{(i)}|$로 제한
- **Link**: [Section 2.1](../2601.05593.md#21-inference-pipeline)

### Section 3.3: Analysis and Ablations
- **Why**: PaCoRe가 "왜" 작동하는지를 설명하는 핵심 분석. 단순 성능 수치를 넘어 메커니즘 이해에 필수.
- **Key Points**:
  - Parallel vs. Sequential scaling: 동일 총 궤적 수에서 병렬이 순차보다 우월 (Figure 4 Left)
  - Message Passing 필수성: 압축 없이는 컨텍스트 한계에 의해 성능 하락 (Figure 4 Right)
  - Emergent Correctness Rate: 모든 입력이 틀려도 올바른 답 생성 가능 (Figure 5 Right)
  - Cross-checking 마커 빈도 상승: 모델이 자발적으로 입력 참조 (Figure 5 Left)
  - Self-Consistency 대비 우위: SC는 @256에서 포화, PaCoRe는 계속 향상 (Table 2)
- **Link**: [Section 3.3](../2601.05593.md#33-analysis-and-ablations)

### Table 1 & Table 2: Main Results
- **Why**: 7개 벤치마크에서의 정량적 성능과 Self-Consistency 대비 효율성 비교.
- **Key Points**:
  - HMMT 2025: 94.5% > GPT-5 93.2% (단, TTC 112배)
  - Self-Consistency @256 (12,376k tokens) = 84.7% vs. PaCoRe medium (869k) = 92.9%
  - PaCoRe가 ~14배 적은 토큰으로 SC를 크게 능가
- **Link**: [Table 1](../2601.05593.md#table-1), [Table 2](../2601.05593.md#table-2)

---

## ⭐⭐ Important

### Section 2.2: Training Procedure
- **Why**: Synthesis 능력을 유도하는 RL 훈련 설계. "어떻게 훈련하는가"를 이해하려면 필수이나, inference pipeline보다 짧고 높은 수준 기술.
- **Key Points**:
  - Outcome-based RL (PPO) + sparse terminal reward
  - 암시적 multi-agent 환경으로서의 훈련
  - Naive aggregation 실패를 강제하는 데이터 필터링
- **Link**: [Section 2.2](../2601.05593.md#22-training-procedure)

### Section 3.1: Training
- **Why**: 2-Stage 커리큘럼, 하이퍼파라미터, 훈련 역학(Figure 3)의 구체적 설정. 재현에 필수.
- **Key Points**:
  - Stage 1: 250 iter, message_acc < 9/24 필터링
  - Stage 2: 450 iter, synthesis_acc 기반 추가 필터링
  - PPO with GAE ($\lambda=1, \gamma=1$), batch 16 × 64, max len 131K
- **Link**: [Section 3.1](../2601.05593.md#31-training)

### Section 4: Related Work
- **Why**: PaCoRe의 위치를 parallel TTC scaling 연구 지형도 내에서 이해. 특히 PDR, AggLM, ParaThinker 등 concurrent work와의 차이점.
- **Link**: [Section 4](../2601.05593.md#4-related-work)

---

## ⭐ Reference

### Section 3.2: Evaluation
- **Why**: 벤치마크 설정, 비교 대상, 캐싱 전략 등 실험 세부사항. Table 1의 숫자를 이해하는 데 보조적.
- **Link**: [Section 3.2](../2601.05593.md#32-evaluation)

### Appendix A: Initial Checkpoint Derivation
- **Why**: RLVR-8B 기반 모델의 SFT + RLVR 파이프라인. PaCoRe 자체보다는 base model 준비 과정.
- **Link**: [Appendix A](../2601.05593.md#a-initial-checkpoint-derivation)

### Appendix C: Training Data Preparation
- **Why**: 수학/코딩 데이터 소스, 합성 데이터, 품질 관리. 데이터 파이프라인 재현 시 참고.
- **Link**: [Appendix C](../2601.05593.md#c-pacore-training-data-preparation)

---

## Skip

### Appendix B: Synthesis Prompt Template
- **Why**: Jinja2 기반 단순 템플릿 (Table 6). 구현 시 복사하면 되는 수준으로, 분석적 가치 낮음.
- **Link**: [Appendix B](../2601.05593.md#b-pacore-synthesis-prompt-template)

### Appendix D: More Ablation Studies
- **Why**: $K_2$ ablation (Table 8) 하나뿐. $K_2=4$가 최적이라는 결론만 기억하면 됨.
- **Link**: [Appendix D](../2601.05593.md#d-more-ablation-studies)

### Section 6: Acknowledgements
- **Why**: 감사 인사. 분석 가치 없음.
- **Link**: [Section 6](../2601.05593.md#6-acknowledgements)
