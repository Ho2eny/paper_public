---
parent: ../2601.09668-analysis.md
source: ../2601.09668.md
created: 2026-02-06
---

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

**Total Reading Time**: ~60 min (전체) / ~25 min (핵심만)

---

## ⭐⭐⭐ Must Read

### Section 3.2: Reinforcement Learning
- **Why Read**: 이 논문의 핵심 기술 혁신. RLVR → RLHF → PaCoRe의 3단계 파이프라인과 이원 보상 체계가 모델 성능의 근간. 데이터 필터링의 3축(Checkability, Visual Relevance, Difficulty), GenRM의 추론 판단 기반 보상, Behavioral Regularization의 hallucination 억제 기법 등이 모두 포함.
- **Key Points**:
  - PPO+GAE 설정 ($\gamma=1, \lambda=1$, off-policy, truncated IS with $C=8$)
  - RLVR 데이터 필터링: 4회 검증 all-agree + 시각적 관련성 + 24-rollout difficulty tagging
  - GenRM: 직접 스칼라 보상이 아닌 추론 판단 후 점수 산출
  - PaCoRe 훈련: message cache pool → Synthesis Filtration → on-policy PPO 500 iter
- **Time**: ~10 min
- **Link**: [3.2 Reinforcement Learning](../2601.09668.md#32-reinforcement-learning)

### Section 5.2: RL Dynamics, Performance, and Emergence
- **Why Read**: 단순 성능 보고를 넘어 RL 학습의 근본적 역학을 분석. "Missing Trace" 가설과 PaCoRe의 emergence 현상은 이 논문의 가장 독창적인 학술적 기여.
- **Key Points**:
  - 보상 지속 증가(포화 없음) vs 롤아웃 길이 비단조 변화
  - Reasoning: sequential scaling (길이↑) vs Perception: length diminishment (길이↓)
  - Perception의 entropy reduction → Pass@N → Pass@1 전환
  - "Missing Trace": 시각적 인지 과정이 학습 데이터에 부재
  - PaCoRe가 이 gap을 우회: proposal → cross-validation → synthesis
  - System 2 → System 1 압축 비전
- **Time**: ~8 min
- **Link**: [5.2 RL Dynamics](../2601.09668.md#52-rl-dynamics-performance-and-emergence)

### Section 4.4: Comparison with Larger Models
- **Why Read**: 10B 모델이 100B+ 및 프로프리어터리 모델과 비교되는 핵심 결과. SeRe vs PaCoRe 효과의 정량적 비교.
- **Key Points**:
  - PaCoRe로 AIME2025 94.43%(Gemini-2.5-Pro 83.96% 대비 +10.47)
  - 수학/추론에서 가장 큰 격차, 인식/공간에서도 유의미한 향상
  - SimpleVQA, CountQA 등 일부에서는 여전히 대형 모델 우위
- **Time**: ~5 min
- **Link**: [4.4 Comparison with Larger Models](../2601.09668.md#44-comparison-with-larger-models)

---

## ⭐⭐ Important

### Section 2.1: Architecture
- **Why Read**: PE-lang 선택 근거, multi-crop 전략, projector 설계 등 모델 구조 이해에 필수
- **When to Read**: 아키텍처 세부사항이 필요할 때, 유사 모델 설계 참고 시
- **Link**: [2.1 Architecture](../2601.09668.md#21-architecture)

### Section 2.3: Training Recipe
- **Why Read**: 2단계 LR 스케줄, 배치 크기, 토큰 수 등 재현에 필요한 핵심 정보
- **When to Read**: 학습 설정을 참고하거나 비교할 때
- **Link**: [2.3 Training Recipe](../2601.09668.md#23-training-recipe)

### Section 5.1: Ablations and Design Insights
- **Why Read**: PE-lang vs DINOv3, Muon vs AdamW, Deepstack 등 핵심 설계 선택의 실험적 근거
- **When to Read**: 특정 아키텍처/최적화 선택의 trade-off를 이해하고 싶을 때
- **Key Insight**: PE-lang은 OCRBench에서 DINOv3 대비 +12.5%, Muon은 tail-knowledge에서 AdamW 대비 +6.48% 이지만 초기화 불일치로 제외
- **Link**: [5.1 Ablations](../2601.09668.md#51-ablations-and-design-insights)

### Section 3.1: Supervised Finetuning
- **Why Read**: 2단계 SFT 전략(text 9:1 → multimodal 1:1)과 데이터 구축/필터링 기법
- **When to Read**: SFT 파이프라인을 참고하거나 유사 데이터 구축 시
- **Link**: [3.1 Supervised Finetuning](../2601.09668.md#31-supervised-finetuning)

---

## ⭐ Reference

### Section 2.2: Data Construction
- **Content**: 7개 도메인(Knowledge, Education, OCR, Grounding, VQA, GUI)의 상세 데이터 구축 기법
- **When Useful**: 특정 도메인 데이터 구축 방법론을 참고할 때, 데이터 규모 비교 시
- **Link**: [2.2 Data Construction](../2601.09668.md#22-data-construction)

### Section 6: Conclusion and Future Work
- **Content**: 3가지 미래 방향 - Universal RL Scaling, 물리적 World Model, Embodied CoT
- **When Useful**: 연구 방향 설정, 후속 연구 아이디어 탐색 시
- **Link**: [6. Conclusion and Future Work](../2601.09668.md#6-conclusion-and-future-work)

### Appendix B: Serialization Details for Synthesis in PaCoRe
- **Content**: PaCoRe의 직렬화 상세 (16개 롤아웃을 어떻게 합성 컨텍스트로 변환하는지)
- **When Useful**: PaCoRe 재현 시
- **Link**: Appendix B in source

---

## Skip

### Section 4.1: Evaluation Setup
- **Why Skip**: 60+ 벤치마크의 단순 나열. 특정 벤치마크 상세가 필요할 때만 참조
- **Alternative**: Table 1-3의 결과를 직접 보고 필요한 벤치마크만 역추적

### Section 4.2-4.3: Detailed Results (7-10B)
- **Why Skip**: Table 1-2의 수치 설명. 표를 직접 보는 것이 더 효율적
- **Alternative**: Table 1(멀티모달), Table 2(텍스트) 직접 참조

### Appendix C: Evaluation Details
- **Why Skip**: 각 벤치마크의 프롬프트, 설정 등 세부 정보. 재현/디버깅 시에만 필요
- **Alternative**: 필요한 특정 벤치마크만 선택적 참조

---

## Recommended Reading Order

1. [Abstract + Introduction](../2601.09668.md#abstract) (문제/해법 파악, 3 min)
2. [3.2 Reinforcement Learning](../2601.09668.md#32-reinforcement-learning) (핵심 기술, 10 min)
3. [4.4 Comparison with Larger Models](../2601.09668.md#44-comparison-with-larger-models) + Table 3 (핵심 결과, 5 min)
4. [5.2 RL Dynamics](../2601.09668.md#52-rl-dynamics-performance-and-emergence) (학술적 기여, 8 min)
5. [5.1 Ablations](../2601.09668.md#51-ablations-and-design-insights) (설계 근거, 5 min)
6. [2.1 Architecture](../2601.09668.md#21-architecture) (구조 이해, 3 min)
