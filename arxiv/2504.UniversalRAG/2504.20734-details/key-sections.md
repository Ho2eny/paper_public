---
parent: ../2504.20734-analysis.md
source: ../2504.20734.md
created: 2026-02-06
---

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

| Metric | Value |
|--------|-------|
| **Total Reading Time** | ~60 minutes (전체, 28 pages) |
| **Essential Only** | ~20 minutes (핵심만) |
| **Total Sections** | 11 sections + Appendices A-F |
| **Must Read** | 3 sections |

---

## ⭐⭐⭐ Must Read

### Section 2.2: UniversalRAG

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
논문의 핵심 기여인 modality-aware routing과 granularity-aware retrieval의 설계 원리와 수식적 정의가 집중된 섹션. Proposition 1과 라우팅 함수 $\mathcal{R}$의 정의가 프레임워크 전체를 이해하는 열쇠.

**Key Points**:
1. 통합 임베딩의 modality gap 문제 정의와 실험적 증거 (Figures 2, 7)
2. Modality-aware routing: 쿼리 → 모달리티 예측 → 모달리티별 검색의 two-stage 구조
3. Granularity-aware retrieval: 라우팅 공간을 모달리티-세분화도 쌍으로 확장
4. Proposition 1: 라우팅 기반 검색이 통합 임베딩보다 우월한 조건의 형식적 증명

**Reading Tips**:
- Modality gap의 직관부터 이해 → Proposition 1은 이 직관의 형식화
- 라우팅 함수 $\mathcal{R}$의 출력 공간(7개 경로 + cross-modal 조합)을 머릿속에 그리기
- "Challenges in Multi-Corpus Retrieval" 소절이 motivation의 핵심

**Estimated Time**: ~8 minutes

**Link**: [Section 2.2](../2504.20734.md#22-universalrag)

---

### Section 3.2: Experimental Results and Analyses

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
Table 1의 주요 결과와 6가지 분석(cross-modal, modality routing, multigranularity, efficiency, router size, OOD)이 포함된 핵심 결과 섹션. UniversalRAG의 각 설계 요소가 왜 필요한지를 데이터로 보여줌.

**Key Points**:
1. Table 1: 12개 베이스라인 대비 일관된 우위, Oracle에 근접한 성능
2. Table 2: Cross-modal 검색이 uni-modal보다 우월
3. Table 3: 모달리티 라우팅 정확도와 recall 비교 (95.28% vs 25-36%)
4. Table 4: 세분화도 단계 증가에 따른 점진적 성능 향상
5. Figure 5: 대규모 코퍼스에서의 latency 이점
6. Table 5: In-domain vs OOD generalization trade-off

**Reading Tips**:
- Table 1을 꼼꼼히 → 어떤 베이스라인이 어떤 벤치마크에서 강/약한지 패턴 파악
- "Effectiveness of..." 소제목들이 각각 하나의 ablation을 설명하므로 관심 있는 것만 선택 가능

**Estimated Time**: ~10 minutes

**Link**: [Section 3.2](../2504.20734.md#32-experimental-results-and-analyses)

---

### Table 1: Main Results

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
논문의 핵심 정량 결과. 10개 벤치마크 × 16개 메트릭에서 모든 방법을 비교하는 종합 테이블.

**Key Points**:
1. Unimodal RAG들은 해당 모달리티 벤치마크에서만 강점 → "no single modality fits all"
2. 통합 임베딩(UniRAG, GME 등)이 Naive보다 평균이 낮은 경우 있음 → modality gap의 실질적 피해
3. UniversalRAG(Qwen3-VL-2B)가 42.40으로 Oracle(42.45)에 근접
4. Training-free GPT-5 라우터(39.26)도 모든 베이스라인을 능가

**Link**: [Table 1](../2504.20734.md#table-1)

---

## ⭐⭐ Important

### Section 2.3: Router Implementation Strategies

**Priority**: ⭐⭐ Important

**Why Read**:
Training-based와 Training-free 라우터의 구체적 구현 방법. 어떤 환경에서 어떤 라우터를 선택해야 하는지의 실용적 가이드.

**Key Points**:
1. Training-based: Multi-hot label + BCE loss + sigmoid threshold 0.8 방식
2. Training-free: 카테고리 정의 + 대비적 few-shot 예시 프롬프트 (Figure 8)
3. 라벨 자동 생성: 벤치마크 task 특성에서 귀납적으로 매핑

**When to Read**:
라우터 구현에 관심이 있거나, 자체 도메인에 적용하려 할 때.

**Link**: [Section 2.3](../2504.20734.md#23-router-implementation-strategies)

---

### Table 5: In-Domain vs Out-of-Domain

**Priority**: ⭐⭐ Important

**Why Read**:
학습 기반 라우터의 OOD 취약성과 training-free 라우터의 강건성을 보여주는 핵심 결과. 앙상블 전략의 효과도 포함.

**When to Read**:
OOD 일반화가 중요한 배포 환경을 고려할 때.

**Link**: [Table 5](../2504.20734.md#table-5)

---

### Figures 4-6: Detailed Analyses

**Priority**: ⭐⭐ Important

**Why Read**:
- Figure 4: 모달리티별 검색 분포 비교 (통합 임베딩의 텍스트 편향 직접 시각화)
- Figure 5: 코퍼스 크기별 latency 비교 (실용적 효율 이점)
- Figure 6: 라우터 모델 크기별 정확도 (경량 배포 가능성)

**Link**: [Figure 4](../2504.20734.md#figure-4), [Figure 5](../2504.20734.md#figure-5), [Figure 6](../2504.20734.md#figure-6)

---

## ⭐ Reference

### Section 1: Introduction

**Priority**: ⭐ Reference Only

**Content Summary**:
문제 배경, 기존 RAG의 한계, modality gap 소개, UniversalRAG의 핵심 아이디어 개요.

**When Useful**:
- 논문을 처음 접할 때 전체 그림 파악용
- Presentation이나 요약 작성 시 motivation 정리용

**Link**: [Section 1](../2504.20734.md#1-introduction)

---

### Appendix C: Theoretical Analyses

**Priority**: ⭐ Reference Only

**Content**:
- C.1: Proposition 1의 완전한 증명 (modality routing 우위)
- C.2: Proposition 2의 증명 (multigranularity 효과)
- C.3: Proposition 3의 증명 (라우팅의 효율성 분석)

**When Useful**:
- 이론적 기반을 엄밀히 확인할 때
- 후속 이론 연구를 계획할 때

**Link**: [Appendix C](../2504.20734.md#c-theoretical-analyses-of-universalrag)

---

### Appendix D: Additional Experimental Results

**Priority**: ⭐ Reference Only

**Content**:
- Table 8: InternVL3.5-8B, Molmo2-4B에서의 전체 벤치마크 결과
- Table 9: 학습 기반 라우터의 세분화도 효과
- Table 10: OOD 벤치마크별 상세 결과

**When Useful**:
특정 LVLM이나 벤치마크에 대한 상세 수치가 필요할 때.

**Link**: [Appendix D](../2504.20734.md#d-additional-experimental-results)

---

### Appendix A-B: Dataset & Implementation Details

**Priority**: ⭐ Reference Only

**Content**:
- A.1-A.2: 16개 벤치마크의 상세 설명 (데이터 크기, 구성, 전처리)
- A.3: 평가 메트릭 상세
- B: 검색 인코더, 비디오 세그멘테이션, 라우터 훈련 하이퍼파라미터

**When Useful**:
- 재현 구현 시
- 데이터셋 전처리 상세 확인 시

**Link**: [Appendix A](../2504.20734.md#a-additional-details-on-dataset), [Appendix B](../2504.20734.md#b-additional-implementation-details)

---

## Skip

### Section 4: Related Work

**Priority**: Skip

**Why Skip**:
- LVLM, RAG, Retrieval Granularity의 표준적 관련 연구 서베이
- Section 1의 motivation과 중복이 많음
- 핵심 기여에 대한 새로운 인사이트 없음

**Alternative**:
Main analysis의 Related Works 섹션에서 핵심 5편만 확인.

**Link**: [Section 4](../2504.20734.md#4-related-work) (참고용)

---

### Appendix E-F: Modality Gap Visualization & Case Studies

**Priority**: Skip

**Why Skip**:
- Appendix E: Figure 7의 확장으로, Figure 2에서 이미 핵심 인사이트 전달됨
- Appendix F: Tables 11-15의 정성적 사례 연구. 흥미롭지만 정량 결과에 새로운 정보 추가 안 됨
- Tables 11-15는 프레젠테이션용으로만 유용

**Link**: [Appendix E](../2504.20734.md#e-modality-gap-in-unified-embedding-space), [Appendix F](../2504.20734.md#f-qualitative-results) (참고용)

---

### Figures 8-11: Prompt Templates

**Priority**: Skip

**Why Skip**:
- Training-free 라우터의 프롬프트 상세. 구현 참고용일 뿐 개념 이해에 불필요.
- 특히 Figure 9(세분화도 확장 프롬프트), 10(WebQA 필터링), 11(비디오 쿼리 변환)은 데이터 전처리 상세.

**Link**: [Figure 8](../2504.20734.md#figure-8) (참고용)

---

## Recommended Reading Order

최적의 읽기 순서:

```
1. Section 2.2 (../2504.20734.md#22-universalrag) - 핵심 프레임워크
   ↓
2. Table 1 + Figure 3 - 주요 정량 결과
   ↓
3. Section 3.2의 분석 소절들 (../2504.20734.md#32-experimental-results-and-analyses) - 왜 작동하는가
   ↓
4. Section 2.3 (../2504.20734.md#23-router-implementation-strategies) - 라우터 구현 (선택)
   ↓
5. Table 5 + Appendix C (선택) - OOD 일반화, 이론적 기반
```

### Quick Read Path (~15 min)

1. Abstract
2. Section 2.2의 "Modality-Aware Retrieval"과 "Granularity-Aware Retrieval" 소절
3. Table 1의 UniversalRAG 행과 Oracle 행 비교
4. Table 3의 모달리티 정확도 열

### Deep Read Path (~45 min)

1. Section 1 (motivation, Figures 1-2)
2. 전체 Section 2
3. 전체 Section 3.2 (모든 분석)
4. Table 5 (OOD)
5. Appendix C.1 (Proposition 1 증명)

---

## Section Dependency Map

```
Introduction (Figures 1-2: motivation)
    │
    ├── Section 2.1 (Preliminaries) ◀── 선택, LLM/RAG 기초
    │
    ▼
Section 2.2 (UniversalRAG) ◀── 먼저 읽기
    │
    ├── Modality-Aware Retrieval ◀── 가장 중요
    ├── Granularity-Aware Retrieval
    └── Proposition 1
    │
    ├── Section 2.3 (Router Implementation) ◀── 실용적 관심 시
    │
    ▼
Section 3.2 (Results & Analyses) ◀── 두 번째
    │
    ├── Table 1 + Figure 3 ◀── 필수
    ├── Tables 2-4 (Ablations)
    ├── Figures 4-6 (Analyses)
    └── Table 5 (OOD)
    │
    ▼
Section 5 (Conclusion) + Limitations (선택)
```

---

## Source References

| Section | Priority | Time | Link |
|---------|----------|------|------|
| Section 2.2 | ⭐⭐⭐ | ~8 min | [Link](../2504.20734.md#22-universalrag) |
| Section 3.2 | ⭐⭐⭐ | ~10 min | [Link](../2504.20734.md#32-experimental-results-and-analyses) |
| Table 1 | ⭐⭐⭐ | ~5 min | [Link](../2504.20734.md#table-1) |
| Section 2.3 | ⭐⭐ | ~5 min | [Link](../2504.20734.md#23-router-implementation-strategies) |
| Table 5 | ⭐⭐ | ~3 min | [Link](../2504.20734.md#table-5) |
| Appendix C | ⭐ | ~10 min | [Link](../2504.20734.md#c-theoretical-analyses-of-universalrag) |
| Section 4 | Skip | - | [Link](../2504.20734.md#4-related-work) |

---

## Navigation

← [Back to Main Analysis](../2504.20734-analysis.md)
