---
parent: ../2504.20734-analysis.md
source: ../2504.20734.md
created: 2026-02-06
---

# Methodology - Deep Analysis

## Paper Type

**Type**: Method

---

## Approach Overview

### High-Level Summary

UniversalRAG는 이질적 멀티모달 코퍼스(텍스트, 이미지, 비디오, 테이블)에서 쿼리에 최적인 모달리티와 세분화도를 동적으로 선택하여 검색하는 RAG 프레임워크이다. 핵심 관찰은 모든 모달리티를 하나의 임베딩 공간에 통합하면 *modality gap*이 발생하여 쿼리와 같은 모달리티의 항목이 편향적으로 검색된다는 것이다.

이를 해결하기 위해 two-stage 접근을 채택한다. 첫 단계에서 라우터가 쿼리를 분석하여 최적 모달리티-세분화도 쌍을 예측하고, 둘째 단계에서 해당 모달리티에 특화된 검색기로 타겟 코퍼스 내에서 검색한다. 라우팅은 cross-modal(여러 모달리티 동시 선택)도 지원하며, "검색 불필요" 옵션도 포함한다.

기존 통합 임베딩(UniRAG, GME)과의 차별점은 모달리티 간 비교를 완전히 회피한다는 것이다. 각 모달리티별 코퍼스가 독립적 임베딩 공간을 유지하므로 modality gap이 발생하지 않으며, 새로운 모달리티 추가 시 라우팅 옵션만 확장하면 된다.

### Key Innovation

"통합"이 아닌 "분리 후 선택적 라우팅"이라는 패러다임 전환. 기존의 "더 좋은 통합 임베딩"을 추구하는 방향 대신, modality gap을 근본적으로 우회하는 접근.

**Source**: [Method Overview](../2504.20734.md#2-method)

---

## Technical Details

### Problem Formulation

| Aspect | Description | Source |
|--------|-------------|--------|
| **Input** | 텍스트 쿼리 $\mathbf{q}$ (일부 벤치마크에서 이미지 동반) | [Section 2.1](../2504.20734.md#21-preliminaries) |
| **Output** | LVLM 생성 응답 $\mathbf{a}$ | [Section 2.1](../2504.20734.md#21-preliminaries) |
| **Objective** | 쿼리에 최적인 모달리티-세분화도 코퍼스에서 검색 후 정확한 응답 생성 | [Section 2.2](../2504.20734.md#22-universalrag) |
| **Constraints** | 모달리티별 독립 임베딩 공간, 라우터의 정확도에 종속 | [Proposition 1](../2504.20734.md#22-universalrag) |

### Architecture / Pipeline

```
Query q → [Router R] → Modality-Granularity Pairs M_q
                            ↓
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
         [Text Retriever]  [Image Retriever]  [Video Retriever]
         (Qwen3-Embed)     (VLM2Vec-V2)      (VLM2Vec-V2)
              ↓             ↓             ↓
         Text Context   Image Context   Video Context
              └─────────────┼─────────────┘
                            ↓
                      [LVLM Generator]
                      (Qwen3-VL / InternVL / Molmo)
                            ↓
                        Answer a
```

**Components**:
1. **Router $\mathcal{R}$**: 쿼리를 7개 라우팅 경로 중 하나 이상으로 분류 - [Section 2.3](../2504.20734.md#23-router-implementation-strategies)
2. **Modality-Specific Retrievers $\mathcal{T}_m$**: 각 모달리티에 특화된 임베딩 인코더로 검색 - [Section 3.1](../2504.20734.md#31-experimental-setup)
3. **LVLM Generator**: 검색된 컨텍스트와 쿼리를 입력받아 응답 생성 - [Section 2.1](../2504.20734.md#21-preliminaries)

### Routing Paths (7 Options)

| Path | Modality | Granularity | Corpus | Encoder |
|------|----------|-------------|--------|---------|
| None | - | - | - | - |
| Paragraph | Text | Fine | Wikipedia paragraphs (≤100 words) | Qwen3-Embedding-4B |
| Document | Text | Coarse | Wikipedia documents (≤4K tokens) | Qwen3-Embedding-4B |
| Table | Table | - | HybridQA tables | Dense row-level + text encoder |
| Image | Image | - | Benchmark-specific image pools | VLM2Vec-V2 |
| Clip | Video | Fine | Scene-segmented clips (≤3 min) | VLM2Vec-V2 |
| Video | Video | Coarse | Full-length videos | VLM2Vec-V2 |

### Training / Learning

#### Training-based Router

| Aspect | Value | Source |
|--------|-------|--------|
| **Architecture** | LVLM backbone + LoRA + classifier head | [Section 2.3](../2504.20734.md#23-router-implementation-strategies) |
| **LoRA Rank** | $r = 32$ | [Appendix B](../2504.20734.md#b-additional-implementation-details) |
| **Loss Function** | Binary Cross-Entropy (multi-label, multi-hot) | [Section 2.3](../2504.20734.md#23-router-implementation-strategies) |
| **Learning Rate** | 2e-5 | [Appendix B](../2504.20734.md#b-additional-implementation-details) |
| **Epochs** | 5 | [Appendix B](../2504.20734.md#b-additional-implementation-details) |
| **Label Generation** | 벤치마크 task 특성에서 귀납적 매핑 (자동) | [Section 2.3](../2504.20734.md#23-router-implementation-strategies) |
| **Inference Threshold** | Sigmoid > 0.8 | [Appendix B](../2504.20734.md#b-additional-implementation-details) |
| **Base Models** | Qwen3-VL-2B, InternVL3.5-1B, T5Gemma 2 270M | [Section 3.1](../2504.20734.md#31-experimental-setup) |

#### Training-free Router

| Aspect | Value | Source |
|--------|-------|--------|
| **Method** | Few-shot prompting | [Section 2.3](../2504.20734.md#23-router-implementation-strategies) |
| **Prompt Structure** | 카테고리 정의 + 대비적 예시 | [Figure 8](../2504.20734.md#figure-8) |
| **Models** | GPT-5, Qwen3-VL-8B-Instruct | [Section 3.1](../2504.20734.md#31-experimental-setup) |

### Inference

1. 쿼리 $\mathbf{q}$ 입력
2. 라우터 $\mathcal{R}(\mathbf{q})$ → 모달리티-세분화도 쌍 집합 $M_\mathbf{q}$ (또는 $\varnothing$)
3. $M_\mathbf{q} = \varnothing$이면 검색 없이 직접 생성
4. 각 선택된 $(m, g) \in M_\mathbf{q}$에 대해 $\mathcal{T}_m$으로 코퍼스 $\mathcal{C}_{m,g}$에서 검색
5. 검색된 컨텍스트 $\mathbf{c}$를 통합하여 LVLM에 입력
6. $\mathbf{a} = \text{LVLM}(\mathbf{q}, \mathbf{c})$

**Source**: [Section 2.2](../2504.20734.md#22-universalrag)

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| 검색 전략 | 모달리티별 분리 검색 | Modality gap 회피 | 통합 임베딩 (UniRAG, GME) | [Section 2.2](../2504.20734.md#22-universalrag) |
| 라우팅 단위 | 모달리티 × 세분화도 쌍 | 쿼리 복잡도에 맞는 정밀 검색 | 모달리티만 라우팅 | [Section 2.2](../2504.20734.md#22-universalrag) |
| 라벨 생성 | 벤치마크 귀납적 매핑 | Human annotation 불필요 | 수동 어노테이션, LLM 판정 | [Section 2.3](../2504.20734.md#23-router-implementation-strategies) |
| 추론 threshold | 0.8 (sigmoid) | Cross-modal 허용하되 노이즈 방지 | 고정 top-1, 적응형 threshold | [Appendix B](../2504.20734.md#b-additional-implementation-details) |
| 비디오 세분화 | PySceneDetect (scene detection) | 의미 기반 자연 분할 | 고정 길이 분할, 트랜스크립트 기반 | [Appendix B](../2504.20734.md#b-additional-implementation-details) |
| 이미지 검색 앙상블 | Visual 0.8 + Textual 0.2 | 시각적 유사성 중시 | 동일 가중치, 학습 가중치 | [Appendix B](../2504.20734.md#b-additional-implementation-details) |
| 프레임 샘플링 | 32 frames/video (uniform) | 검색 + 생성 모두 적용 | 가변 프레임, 키프레임 추출 | [Appendix B](../2504.20734.md#b-additional-implementation-details) |

---

## Assumptions & Limitations

### Assumptions

1. **모달리티 정보가 쿼리에서 추론 가능**: 쿼리 텍스트만으로 필요한 모달리티를 판별할 수 있다고 가정. 모호한 쿼리에서는 라우팅 실패 가능 (Table 15의 실패 사례) - [Section 2.2](../2504.20734.md#22-universalrag)
2. **코퍼스가 사전 구성됨**: 모달리티별, 세분화도별로 코퍼스가 미리 분리되어 있어야 함. 실시간 코퍼스 변경에 대한 대응 미고려 - [Section 3.1](../2504.20734.md#31-experimental-setup)
3. **모달리티별 검색기의 품질**: 각 모달리티의 검색기가 충분히 성능이 좋다고 가정. 라우팅이 정확해도 검색기가 약하면 최종 성능 저하 - [Section 3.1](../2504.20734.md#31-experimental-setup)

### Limitations

1. **벤치마크 종속 코퍼스**: 각 벤치마크에 맞춤 구성된 소규모 코퍼스 사용. 대규모 통합 코퍼스에서의 검증 부재.
2. **자동 생성 라벨의 노이즈**: 벤치마크 귀납 편향 기반 라벨은 진정한 최적 모달리티와 다를 수 있음.
3. **2단계 세분화도만 실험**: 텍스트(문단/문서), 비디오(클립/전체)에 대해 2단계만. 더 세밀한 단계 검증 부족.
4. **제한된 모달리티**: 텍스트, 이미지, 비디오, 테이블만. 오디오, 3D, 코드 등 미지원.
5. **라우팅 오류의 cascading**: 라우팅이 틀리면 검색과 생성 모두 실패. 오류 복구 메커니즘 없음.

### Scope

- **In Scope**: 텍스트/이미지/비디오/테이블 코퍼스에서의 QA, 정보 검색
- **Out of Scope**: 실시간 대화, 코퍼스 업데이트, 오디오/3D 모달리티, 생성형 검색

---

## Complexity Analysis

| Aspect | Complexity | Notes |
|--------|------------|-------|
| **Routing Time** | $O(1)$ (고정 비용 $C$) | LVLM forward pass 1회 |
| **Retrieval Time** | $O(T(N))$ vs $O(T(kN))$ unified | $k$: 모달리티-세분화도 수, $N$: 코퍼스 크기 |
| **Exact Retrieval Speedup** | $\Theta(k)$ (linear in $k$) | 7개 경로이므로 ~7배 속도 향상 |
| **ANN Retrieval Speedup** | $\geq 1$ (constant factor) | FAISS 등 사용 시 |
| **Space** | $k \times$ index size | 각 코퍼스별 독립 인덱스 필요 |
| **Training Data** | 벤치마크 train split의 30% | 자동 라벨링, 수동 어노테이션 불필요 |
| **Training Compute** | LoRA fine-tuning (270M-2B) × 5 epochs | NVIDIA RTX Pro 6000 96GB |

---

## Source References Summary

| Topic | Section | Link |
|-------|---------|------|
| Method Overview | Section 2 | [Link](../2504.20734.md#2-method) |
| Modality-Aware Routing | Section 2.2 | [Link](../2504.20734.md#22-universalrag) |
| Granularity-Aware Retrieval | Section 2.2 | [Link](../2504.20734.md#22-universalrag) |
| Router Implementation | Section 2.3 | [Link](../2504.20734.md#23-router-implementation-strategies) |
| Experiments | Section 3 | [Link](../2504.20734.md#3-experiment) |
| Theoretical Analysis | Appendix C | [Link](../2504.20734.md#c-theoretical-analyses-of-universalrag) |
| Implementation Details | Appendix B | [Link](../2504.20734.md#b-additional-implementation-details) |

---

## Navigation

← [Back to Main Analysis](../2504.20734-analysis.md)
