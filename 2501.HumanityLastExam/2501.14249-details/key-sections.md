---
parent: ../2501.14249-analysis.md
source: ../2501.14249.md
created: 2026-02-06
---

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

| Metric | Value |
|--------|-------|
| **Total Reading Time** | ~45 minutes (전체, Appendix 제외) |
| **Essential Only** | ~15 minutes (핵심 3개 섹션) |
| **Total Sections** | 5 main + 3 appendices |
| **Must Read** | 3 sections |

---

## ⭐⭐⭐ Must Read

### Section 3: Dataset

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
이 논문의 핵심 기여가 벤치마크 자체이므로, 데이터셋의 설계 원칙과 구축 과정은 논문 이해의 필수 요소이다. 질문 형식, 수집 방법, 검토 프로세스를 통해 HLE의 차별점을 정확히 파악할 수 있다.

**Key Points**:
1. 2,500개 질문, 100+ 분야, ~1,000명 전문가 참여
2. 24% MC / 76% exact-match, 14% 멀티모달
3. LLM 사전 필터링 (7만 시도 → 1.3만 통과)
4. 2단계 전문가 리뷰 + 커뮤니티 보정

**Reading Tips**:
- Section 3.1의 "Submission Format"에서 질문 요구사항을 주의 깊게 읽을 것
- Section 3.2의 "Expert Review"에서 리뷰 프로세스의 2단계 구조를 파악할 것
- Figure 4 (Pipeline)를 먼저 보면 전체 흐름 이해에 도움

**Estimated Time**: ~7 minutes

**Link**: [Section 3 - Dataset](../2501.14249.md#3-dataset)

---

### Section 4.2: Quantitative Results

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
벤치마크 논문에서 가장 핵심적인 섹션. 8개 frontier 모델의 정확도 및 calibration 결과를 통해 HLE의 난이도와 모델의 한계를 직접 확인할 수 있다.

**Key Points**:
1. O3-MINI (HIGH) 최고 13.4%, GPT-4O 최저 2.7%
2. 모든 모델 73-89% RMS calibration error
3. 추론 모델이 비추론 모델 대비 우수하나 토큰 소비 큼
4. 카테고리별 성능 차이 (Math > Bio/Med > Humanities)

**Reading Tips**:
- Table 1을 중심으로 읽되, calibration error 컬럼에 특히 주목
- Figure 5 (Token counts)로 compute-performance 트레이드오프 파악

**Estimated Time**: ~5 minutes

**Link**: [Section 4.2 - Quantitative Results](../2501.14249.md#42-quantitative-results)

---

### Section 1: Introduction

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
벤치마크 포화 문제의 동기와 HLE의 설계 목표를 명확히 이해하는 데 필수적이다. Figure 1과 함께 읽으면 기존 벤치마크 대비 HLE의 포지션을 한눈에 파악할 수 있다.

**Key Points**:
1. MMLU 등 기존 벤치마크 90%+ 포화 문제
2. HLE의 설계 원칙: 정밀, 모호하지 않음, 검색 불가, 전문가 수준
3. 멀티모달, MC + exact-match 혼합 형식
4. 공개 데이터셋 + private test set 유지

**Estimated Time**: ~3 minutes

**Link**: [Section 1 - Introduction](../2501.14249.md#1-introduction)

---

## ⭐⭐ Important

### Section 5: Discussion

**Priority**: ⭐⭐ Important

**Why Read**:
저자들의 HLE 포화 전망과 벤치마크의 의의에 대한 관점을 파악할 수 있다. "2025년 말까지 50% 달성 가능" 예측은 모델 발전 속도에 대한 중요한 참고점이다.

**When to Read**:
메인 결과(Section 4.2)를 읽은 후, 향후 전망에 관심이 있을 때

**Key Points**:
1. 2025년 말 50% 달성 가능성
2. HLE ≠ AGI 측정 (구조화된 학술 문제에 한정)
3. AI 정책과 거버넌스에 대한 시사점

**Link**: [Section 5 - Discussion](../2501.14249.md#5-discussion)

---

### Section B.3: Expert Disagreement Rate and HLE-Rolling

**Priority**: ⭐⭐ Important

**Why Read**:
벤치마크의 신뢰성 평가에 핵심적인 전문가 불일치율(15.4%)에 대한 상세 분석과, 향후 동적 업데이트 계획(HLE-Rolling)을 포함한다.

**When to Read**:
데이터 품질에 의문이 있거나, HLE의 장기적 유효성을 평가하고 싶을 때

**Key Points**:
1. 전체 불일치율 15.4%, 생물/화학/건강 분야 18%
2. 단일 리뷰어 방식 시 25%까지 상승 → 다중 리뷰의 중요성
3. HLE-Rolling: 커뮤니티 피드백 기반 동적 업데이트 예정

**Link**: [Section B.3](../2501.14249.md#b3-expert-disagreement-rate-and-hle-rolling)

---

## ⭐ Reference

### Section 2: Related Work

**Priority**: ⭐ Reference Only

**Content Summary**:
LLM 벤치마크의 발전 역사와 포화 문제, 프론티어 벤치마크 설계 접근법을 리뷰한다.

**When Useful**:
- 관련 벤치마크(MMLU, GPQA, FrontierMath 등)를 비교하고 싶을 때
- HLE의 설계 결정이 어떤 선행 연구에 기반했는지 추적하고 싶을 때

**Link**: [Section 2 - Related Work](../2501.14249.md#2-related-work)

---

### Section 4.1: Setup

**Priority**: ⭐ Reference Only

**Content Summary**:
평가 셋업: 시스템 프롬프트, O3-MINI 기반 자동 채점, 모델 버전 등 실험 설정 세부사항.

**When Useful**:
- HLE 결과를 재현하고 싶을 때
- 자동 채점 방식의 세부사항이 필요할 때

**Link**: [Section 4.1 - Setup](../2501.14249.md#41-setup)

---

### Appendix B: Dataset Details

**Priority**: ⭐ Reference Only

**Content Summary**:
제출 프로세스(B.1), 릴리즈 후 보정(B.2), 전문가 불일치율(B.3), 분야 목록(B.4)의 세부사항.

**When Useful**:
- 데이터 수집 파이프라인의 세부사항이 필요할 때
- 구체적인 분야 목록을 확인하고 싶을 때

**Link**: [Appendix B - Dataset](../2501.14249.md#b-dataset)

---

### Appendix C.1-C.4: Evaluation Details

**Priority**: ⭐ Reference Only

**Content Summary**:
평가 프롬프트(C.1), Text-only 결과(C.2), 카테고리별 결과(C.3), 비추론 모델 토큰 수(C.4).

**When Useful**:
- 카테고리별 세부 성능 분석이 필요할 때 (Table 3)
- 멀티모달 vs text-only 성능 차이를 확인하고 싶을 때

**Link**: [Appendix C.1](../2501.14249.md#c1-prompts)

---

## Skip

### Appendix A: Authors & Affiliations

**Priority**: Skip

**Why Skip**:
1,112명의 저자 목록과 437개 기관의 상세 어필리에이션으로, 논문 이해에 필요하지 않음. 프로젝트의 규모감을 파악하는 데는 Introduction의 언급만으로 충분.

**Link**: [Appendix A](../2501.14249.md#a-authors) (참고용)

---

### Appendix C.5-C.7: Model Versions & Review Instructions

**Priority**: Skip

**Why Skip**:
- C.5 (Model Versions): 구체적 모델 버전 목록으로, 재현 시에만 필요
- C.6 (Benchmark Comparison Sources): Figure 1의 데이터 출처로, 보조적 참고 자료
- C.7 (Review Instructions): 리뷰 루브릭 세부사항으로, HLE에 문제를 제출하지 않는 한 불필요

**Link**: [Appendix C.5](../2501.14249.md#c5-model-versions) (참고용)

---

## Recommended Reading Order

최적의 읽기 순서:

```
1. Section 1 - Introduction (동기 + Figure 1)
   ↓
2. Section 3 - Dataset (핵심 설계 + Figure 3, 4)
   ↓
3. Section 4.2 - Quantitative Results (Table 1 + Figure 5)
   ↓
4. Section 5 - Discussion (선택: 향후 전망)
   ↓
5. Section B.3 - Expert Disagreement (선택: 데이터 품질)
```

### Quick Read Path (~10 min)
1. Abstract
2. Section 3.1의 첫 2문단 + Figure 3
3. Table 1 (Section 4.2)

### Deep Read Path (~35 min)
1. 전체 Section 1
2. 전체 Section 3 (3.1 + 3.2)
3. 전체 Section 4
4. Section 5
5. Section B.3

---

## Section Dependency Map

```
Section 1 (Introduction)
    │
    ├── Section 2 (Related Work) ← 선택
    │
    ▼
Section 3 (Dataset) ◀── 먼저 읽기
    │
    ├── 3.1 Collection ◀── 핵심
    └── 3.2 Review ◀── 핵심
    │
    ▼
Section 4 (Evaluation) ◀── 두 번째
    │
    ├── 4.1 Setup (참고)
    └── 4.2 Results ◀── 필수
    │
    ▼
Section 5 (Discussion) ← 선택
    │
    ▼
Appendix B.3 (Disagreement) ← 선택
```

---

## Source References

| Section | Priority | Time | Link |
|---------|----------|------|------|
| Section 1 (Introduction) | ⭐⭐⭐ | ~3 min | [Link](../2501.14249.md#1-introduction) |
| Section 3 (Dataset) | ⭐⭐⭐ | ~7 min | [Link](../2501.14249.md#3-dataset) |
| Section 4.2 (Results) | ⭐⭐⭐ | ~5 min | [Link](../2501.14249.md#42-quantitative-results) |
| Section 5 (Discussion) | ⭐⭐ | ~3 min | [Link](../2501.14249.md#5-discussion) |
| Section B.3 (Disagreement) | ⭐⭐ | ~5 min | [Link](../2501.14249.md#b3-expert-disagreement-rate-and-hle-rolling) |
| Section 2 (Related Work) | ⭐ | ~5 min | [Link](../2501.14249.md#2-related-work) |
| Appendix B (Dataset Details) | ⭐ | ~5 min | [Link](../2501.14249.md#b-dataset) |
| Appendix A (Authors) | Skip | - | [Link](../2501.14249.md#a-authors) |
| Appendix C.5-C.7 | Skip | - | [Link](../2501.14249.md#c5-model-versions) |

---

## Navigation

<- [Back to Main Analysis](../2501.14249-analysis.md)
