---
parent: ../2504.20734-analysis.md
source: ../2504.20734.md
created: 2026-02-06
---

# Core Contributions - Deep Analysis

## Overview

| # | Contribution | Impact | Source |
|---|--------------|--------|--------|
| 1 | Modality-Aware Routing | Modality gap 회피, 정확도 95.28% | [Section 2.2](#contribution-1-modality-aware-routing) |
| 2 | Granularity-Aware Retrieval | 쿼리별 최적 검색 단위 선택 | [Section 2.2](#contribution-2-granularity-aware-retrieval) |
| 3 | Dual Router Architecture | In-domain + OOD 상보적 강점 | [Section 2.3](#contribution-3-dual-router-architecture) |
| 4 | Comprehensive Multimodal Evaluation | 10 벤치마크, 3 LVLM 검증 | [Section 3](#contribution-4-comprehensive-multimodal-evaluation) |

---

## Contribution 1: Modality-Aware Routing

### What

통합 임베딩 공간에서 발생하는 *modality gap* 문제를 회피하기 위해, 쿼리를 먼저 최적 모달리티별 코퍼스로 동적 라우팅하는 two-stage 검색 메커니즘. Stage 1에서 라우터가 적합한 모달리티를 예측하고, Stage 2에서 해당 모달리티 특화 검색기로 검색한다.

### Technical Details

- **Modality gap 분석**: 통합 임베딩 공간에서 텍스트 쿼리가 모달리티 유사성에 의해 텍스트 문서로 편향 검색됨 (Figures 2, 7)
- **이론적 증명 (Proposition 1)**: 모달리티 편향 $\alpha$가 의미 유사도의 분산보다 충분히 크면, 통합 검색보다 모달리티별 검색이 필요 모달리티에서 검색할 확률이 높음
- **수학적 조건**: $\alpha > 2\sigma\sqrt{\log(|R||S|)/r}$이면 라우팅 기반 검색이 우월 (대규모 코퍼스 $|R|=|S|=10^{12}$에서 $\alpha > 0.17$ 수준)
- **검색 분리**: 텍스트(Qwen3-Embedding-4B), 이미지/비디오(VLM2Vec-V2), 테이블(dense row-level embedding) 각각 독립 인코더 사용

### Why It Matters

- **학술적**: Modality gap을 형식적으로 정의하고 라우팅의 이론적 우위를 증명한 첫 연구
- **실용적**: 기존 모달리티별 검색기를 그대로 활용 → 새 모달리티 추가 시 라우팅 옵션만 확장

### Comparison to Prior Work

| Aspect | UniversalRAG | UniRAG/GME (통합 임베딩) | MultiRAG (전체 검색) |
|--------|-------------|------------------------|---------------------|
| Modality gap | 회피 (분리 검색) | 존재 (편향 검색) | 회피하나 노이즈 유입 |
| 모달리티 정확도 | 95.28% | 25-36% | 100% (모두 검색) |
| 검색 효율 | 선택적 (sub-linear) | 전체 (linear) | 전체 × 모달리티 수 |
| Recall@5 | 44.82 | 4.12-22.16 | - |

### Source References

- Definition: [Section 2.2](../2504.20734.md#22-universalrag) (Modality-Aware Retrieval)
- Theory: [Appendix C.1](../2504.20734.md#c1-effectiveness-of-modality-routing)
- Evidence: [Table 3](../2504.20734.md#table-3), [Figure 4](../2504.20734.md#figure-4)

---

## Contribution 2: Granularity-Aware Retrieval

### What

모달리티뿐 아니라, 동일 모달리티 내에서도 쿼리 복잡도에 맞는 검색 세분화도를 동적으로 선택. 텍스트는 Paragraph/Document, 비디오는 Clip/Video로 구분하여 라우팅 공간을 7개 경로(None, Paragraph, Document, Table, Image, Clip, Video)로 확장.

### Technical Details

- **라우팅 함수 확장**: $\mathcal{R}: \mathcal{Q} \to \{\varnothing\} \cup \mathcal{P}(\bigcup_{m \in M} \{m\} \times G_m)$
- **세분화도 효과 (Table 4)**: 1단계(모달리티만) → 2단계(+세분화도) → 4단계(더 세밀)로 갈수록 일관된 성능 향상
  - HotpotQA EM: 23.20 → 24.35 → 24.70 (GPT-5 라우터)
  - LVBench Acc: 31.92 → 32.30 → 32.85
- **이론적 정당화 (Proposition 2)**: 쿼리 $\mathbf{q}_1$에 fine-grained가, $\mathbf{q}_2$에 coarse-grained가 최적이면, 동적 선택이 고정 세분화도보다 엄밀히 우월
- **실제 세분화도 구성**:
  - 텍스트: 문단(≤100 words) / 문서(≤4K tokens, LongRAG 방식)
  - 비디오: 클립(PySceneDetect, 평균 ≤3분) / 전체 비디오
  - 이미지: 단일 세분화도 (본질적으로 원자적)
  - 테이블: 단일 세분화도

### Why It Matters

- 검색 세분화도와 생성 품질의 관계를 멀티모달 RAG 컨텍스트에서 최초로 체계적으로 분석
- 과도하게 세밀한 검색(필요 정보 분산)과 과도하게 거친 검색(무관 정보 포함)의 trade-off를 동적으로 해결

### Source References

- Definition: [Section 2.2](../2504.20734.md#22-universalrag) (Granularity-Aware Retrieval)
- Theory: [Appendix C.2](../2504.20734.md#c2-effectiveness-of-multigranularity)
- Evaluation: [Table 4](../2504.20734.md#table-4), [Table 9](../2504.20734.md#table-9)

---

## Contribution 3: Dual Router Architecture

### What

Training-based와 Training-free 라우터를 모두 구현하여 각각의 장단점을 분석하고, 앙상블 전략으로 두 접근의 상보적 강점을 결합.

### Technical Details

- **Training-based**: LVLM에 LoRA(rank=32)로 multi-label classifier head 학습. 자동 생성된 라벨(벤치마크 귀납 편향)로 5 epoch, lr=2e-5, BCE loss. 추론 시 sigmoid > 0.8 threshold.
  - Qwen3-VL-2B-Instruct: 95.81% in-domain accuracy
  - InternVL3.5-1B: 93.16%
  - T5Gemma 2 270M: comparable
- **Training-free**: GPT-5/Qwen3-VL-8B에 few-shot prompt (Figure 8)로 라우팅. 카테고리 정의 + 대비적 예시 포함.
  - GPT-5: 72.33% in-domain → 77.38% out-of-domain
- **앙상블**:
  - Confidence-based: 학습 라우터 confidence > threshold → 학습 라우터, 아니면 training-free
  - Majority Voting: 3개 라우터(2 trained + 1 free) 중 다수결
  - 결과: In-domain 98.33%, OOD 80.71% (majority voting)

### Why It Matters

- In-domain 정확도(trained) vs OOD 일반화(training-free)의 trade-off를 명시적으로 분석한 첫 연구
- 실무 배포 시 환경에 따른 라우터 선택 가이드라인 제공

### Source References

- Definition: [Section 2.3](../2504.20734.md#23-router-implementation-strategies)
- OOD Analysis: [Table 5](../2504.20734.md#table-5), [Table 10](../2504.20734.md#table-10)
- Router Size: [Figure 6](../2504.20734.md#figure-6)

---

## Contribution 4: Comprehensive Multimodal Evaluation

### What

텍스트(NQ, HotpotQA), 테이블(HybridQA), 이미지(MRAG, WebQA, InfoSeek), 비디오(LVBench, VideoRAG)를 아우르는 10개 in-domain + 6개 OOD 벤치마크에서 3개 LVLM과 12개 베이스라인으로 체계적 평가.

### Technical Details

- **베이스라인 카테고리**: (1) Naive (검색 없음), (2) Unimodal RAG 6종, (3) Unified Embedding Multimodal RAG 4종 (UniRAG, GME, PE_core, VLM2Vec-V2), (4) Multi-corpus Multimodal RAG (MultiRAG)
- **LVLM**: Qwen3-VL-8B, InternVL3.5-8B, Molmo2-4B
- **메트릭**: Acc, EM, F1, ROUGE-L, BERTScore (벤치마크별 적합 메트릭)
- **OOD 벤치마크**: TruthfulQA, TriviaQA, SQuAD, 2WikiMultiHopQA, Visual-RAG, CinePile

### Why It Matters

- 멀티모달 RAG 분야에서 가장 포괄적인 벤치마크 평가
- 베이스라인 다양성이 높아 공정한 비교 가능
- 3개 LVLM에서의 일관된 결과가 프레임워크의 범용성을 입증

### Source References

- Datasets: [Section 3.1](../2504.20734.md#31-experimental-setup), [Appendix A](../2504.20734.md#a-additional-details-on-dataset)
- Main Results: [Table 1](../2504.20734.md#table-1), [Figure 3](../2504.20734.md#figure-3)
- Additional LVLM: [Table 8](../2504.20734.md#table-8)

---

## Contribution Analysis

### Novelty Assessment

| Contribution | Novelty Level | Justification |
|--------------|---------------|---------------|
| 1. Modality-Aware Routing | **High** | Modality gap의 이론적 분석 + 라우팅 기반 해결이 새로움 |
| 2. Granularity-Aware Retrieval | **Medium** | Dense X Retrieval 등 선행 연구 존재, 멀티모달 확장이 기여 |
| 3. Dual Router Architecture | **Medium** | 개별 접근은 기존, 체계적 비교와 앙상블이 기여 |
| 4. Comprehensive Evaluation | **Medium** | 평가 자체는 novelty가 아니나, 이 규모의 멀티모달 RAG 벤치마킹은 최초 |

### Impact Assessment

| Contribution | Short-term Impact | Long-term Impact |
|--------------|-------------------|------------------|
| 1. Modality-Aware Routing | 멀티모달 RAG 설계 패러다임 전환 | 범용 지식 검색 시스템의 핵심 구성 요소 |
| 2. Granularity-Aware Retrieval | 벤치마크별 성능 개선 | 동적 코퍼스 조직화 연구 촉진 |
| 3. Dual Router Architecture | 배포 환경별 라우터 선택 가이드 | 자기 적응형 라우터 연구의 기반 |
| 4. Comprehensive Evaluation | 후속 연구의 표준 벤치마크 | 멀티모달 RAG 연구의 공통 평가 프레임워크 |

### Dependencies

```
Contribution 1 (Modality-Aware Routing)
    └── extends → Contribution 2 (Granularity-Aware Retrieval)
                      └── implements → Contribution 3 (Dual Router Architecture)
                                          └── validates → Contribution 4 (Comprehensive Evaluation)
```

---

## Source References Summary

| Contribution | Primary Source | Supporting Sources |
|--------------|----------------|-------------------|
| 1 | [Section 2.2](../2504.20734.md#22-universalrag) | [Appendix C.1](../2504.20734.md#c1-effectiveness-of-modality-routing), [Table 3](../2504.20734.md#table-3) |
| 2 | [Section 2.2](../2504.20734.md#22-universalrag) | [Appendix C.2](../2504.20734.md#c2-effectiveness-of-multigranularity), [Table 4](../2504.20734.md#table-4) |
| 3 | [Section 2.3](../2504.20734.md#23-router-implementation-strategies) | [Table 5](../2504.20734.md#table-5), [Figure 6](../2504.20734.md#figure-6) |
| 4 | [Section 3](../2504.20734.md#3-experiment) | [Appendix A](../2504.20734.md#a-additional-details-on-dataset), [Table 8](../2504.20734.md#table-8) |

---

## Navigation

← [Back to Main Analysis](../2504.20734-analysis.md)
