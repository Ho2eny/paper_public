---
parent: ../2601.22149-analysis.md
source: ../2601.22149.md
created: 2026-02-06
---

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

| Metric | Value |
|--------|-------|
| **Total Reading Time** | ~50 minutes (전체) |
| **Essential Only** | ~25 minutes (핵심만) |
| **Total Sections** | 8 sections (본문 6 + 부록 3) |
| **Must Read** | 3 sections |

---

## ⭐⭐⭐ Must Read

### Section 3: Method (3.2 Web World Model + 3.3 DynaWeb)

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
DynaWeb의 핵심 기술적 기여가 집중된 섹션. 웹 월드 모델의 설계(delta 예측, reasoning trace 학습)와 MBRL 프레임워크의 구체적 메커니즘(상상 롤아웃, 전문가 인터리빙, GSPO)을 이해하지 않으면 논문의 본질을 파악할 수 없다.

**Key Points**:
1. 월드 모델은 전체 다음 상태가 아닌 **상태 변화(delta)**를 예측하도록 설계 -> 웹 페이지의 대부분이 불변이라는 특성 활용
2. 에이전트와 월드 모델의 반복적 상호작용으로 최대 5 step 상상 궤적 생성
3. 실제 전문가 궤적을 50% 비율로 무작위 인터리빙하여 학습 안정성 확보
4. GSPO의 시퀀스 수준 importance ratio가 긴 궤적에서의 gradient 안정화에 핵심

**Reading Tips**:
- Equation 3-4 (월드 모델)와 Equation 8-10 (GSPO)에 집중
- Figure 2의 시스템 다이어그램을 먼저 보면 전체 흐름 파악에 도움
- 3.1 (POMDP 정의)은 표준적이므로 빠르게 스캔 가능

**Estimated Time**: ~12 minutes

**Link**: [Section 3](../2601.22149.md#3-method)

---

### Section 4.2: Main Results (Tables 1-2)

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
DynaWeb의 핵심 주장 - "상상 기반 훈련이 실제로 작동한다"는 것의 실증적 증거. WebArena와 WebVoyager 양쪽에서의 도메인별 성능 분해를 통해, 어떤 유형의 웹 태스크에서 가장 큰 이득이 있고 어디서 한계가 드러나는지 파악 필수.

**Key Points**:
1. WebArena: 31.0% (vs WebRL 26.7%, +16.1%) - 시뮬레이션 환경에서의 우위
2. WebVoyager: 38.7% (vs WebRL 32.6%, +18.7%) - 라이브 웹에서도 유효
3. 도메인별 패턴: Reddit/Amazon 등에서 강점, GitHub/Map에서 상대적 약점
4. 8B 모델이 GPT-4o(14.3%)를 2배 이상 능가

**Reading Tips**:
- 각 도메인별 1위/2위를 비교하며, DynaWeb이 어디서 가장 큰 개선을 보이는지 주목
- GitHub와 ArXiv에서의 상대적 약점이 왜 발생하는지 고찰

**Estimated Time**: ~8 minutes

**Link**: [Section 4.2](../2601.22149.md#42-main-results)

---

### Section 5: Analysis (5.1-5.3)

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
단순 성능 보고를 넘어, MBRL 웹 에이전트의 설계 원칙을 이해하는 데 핵심적인 ablation 분석. 드림 길이, 데이터 비율, 월드 모델 학습이라는 3가지 축에서의 체계적 실험은 후속 연구의 설계 지침이 된다.

**Key Points**:
1. **드림 길이 4-5 step 최적**: 너무 짧으면 태스크 불완전, 너무 길면 환각 누적
2. **실제 데이터 ~40% 최적**: 0%는 SFT 이하, 100%는 수확 체감
3. **전용 WM 학습 필수**: 범용 LLM 프롬프팅 대비 WebArena +10.1%, WebVoyager +6.8%

**Reading Tips**:
- Figure 3의 두 차트를 먼저 보면 핵심 경향을 시각적으로 파악 가능
- Table 4의 DynaWeb WM vs GPT-oss-120b 비교가 가장 극적인 결과

**Estimated Time**: ~5 minutes

**Link**: [Section 5](../2601.22149.md#5-analysis)

---

## ⭐⭐ Important

### Section 3.1: Problem Formulation

**Priority**: ⭐⭐ Important

**Why Read**:
POMDP 프레임워크와 웹 에이전트의 수학적 정의. 나머지 섹션의 notation을 이해하기 위한 기반.

**When to Read**:
POMDP/RL 형식화에 익숙하지 않은 독자, 또는 수식을 정확히 이해하고 싶을 때.

**Key Points**:
1. 관찰 공간: accessibility tree 기반
2. 행동 공간: click, type, goback, scroll, stop
3. 에이전트: chain-of-thought + 행동을 동시 생성하는 LLM 정책

**Link**: [Section 3.1](../2601.22149.md#31-problem-formulation)

---

### Section 4.1: Setups

**Priority**: ⭐⭐ Important

**Why Read**:
실험 설정의 세부사항. 재현이나 비교 연구 시 필수적인 정보.

**When to Read**:
직접 실험을 재현하거나, 기존 방법과의 공정한 비교를 검증하고 싶을 때.

**Key Points**:
1. NNetNav 에이전트를 초기 체크포인트로 사용
2. 전문가 궤적 50% 인터리빙
3. WebVoyager는 접근 불가 사이트 필터링 후 평가

**Link**: [Section 4.1](../2601.22149.md#41-setups)

---

## ⭐ Reference

### Section 1: Introduction

**Priority**: ⭐ Reference Only

**Content Summary**:
웹 에이전트 RL의 동기, 기존 한계, DynaWeb의 핵심 아이디어, 기여 요약. 논문의 전체 맥락 파악용.

**When Useful**:
- 논문을 처음 접할 때 배경 이해
- 발표/리뷰를 위해 동기 부분을 인용할 때

**Link**: [Section 1](../2601.22149.md#1-introduction)

---

### Appendix A: More Details

**Priority**: ⭐ Reference Only

**Content**:
- A.1: WebArena 에이전트 시스템 프롬프트
- A.2: 월드 모델 시스템 프롬프트
- A.3: 전체 하이퍼파라미터 (Table 3)

**When Useful**:
- 재현 구현 시 정확한 프롬프트와 설정값 필요할 때
- 시스템 프롬프트 설계 참고

**Link**: [Appendix](../2601.22149.md#appendix)

---

## Skip

### Section 2: Related Work

**Priority**: Skip

**Why Skip**:
- 웹 에이전트, 월드 모델, RL 에이전트의 표준적 문헌 리뷰
- MBRL이나 웹 에이전트 분야에 이미 익숙한 독자에게 새로운 정보 제한적
- DynaWeb의 차별점은 이미 Introduction과 Method에서 충분히 설명됨

**Alternative**:
분야에 익숙하지 않다면 Introduction의 첫 3문단으로 충분한 맥락 확보 가능

**Link**: [Section 2](../2601.22149.md#2-related-work) (참고용)

---

### Section 6: Conclusion

**Priority**: Skip

**Why Skip**:
- 논문의 요약으로 새로운 정보 없음
- Introduction과 Main Results를 읽었다면 중복

**Link**: [Section 6](../2601.22149.md#6-conclusion) (참고용)

---

## Recommended Reading Order

최적의 읽기 순서:

```
1. Figure 2 (시스템 다이어그램) - 전체 구조 파악
   ↓
2. Section 3.2-3.3 (Method) - 핵심 기술 이해
   ↓
3. Tables 1-2 (Main Results) - 성능 검증
   ↓
4. Section 5 (Analysis) - 설계 원칙 이해 (선택)
   ↓
5. Appendix A (Details) - 재현 필요 시
```

### Quick Read Path (~15 min)

1. Abstract
2. Figure 1 + Figure 2 (핵심 개념)
3. Section 3.2-3.3 (핵심 수식 스캔)
4. Tables 1-2 (결과 수치)

### Deep Read Path (~50 min)

1. 전체 Section 3 (Method)
2. 전체 Section 4 (Experiments)
3. 전체 Section 5 (Analysis)
4. Appendix A (시스템 프롬프트, 하이퍼파라미터)

---

## Section Dependency Map

```
Introduction
    │
    ├── Related Work (선택)
    │
    ▼
Section 3.1 (POMDP) ◀── 수식 기반
    │
    ├── Section 3.2 (Web World Model) ◀── 핵심 #1
    └── Section 3.3 (DynaWeb MBRL) ◀── 핵심 #2
    │
    ▼
Section 4 (Experiments)
    │
    ├── Tables 1-2 (Main Results) ◀── 필수
    └── Section 5 (Analysis) ◀── 설계 원칙
    │
    ▼
Appendix A (재현용)
```

---

## Source References

| Section | Priority | Time | Link |
|---------|----------|------|------|
| Section 3 (Method) | ⭐⭐⭐ | ~12 min | [Link](../2601.22149.md#3-method) |
| Section 4.2 (Results) | ⭐⭐⭐ | ~8 min | [Link](../2601.22149.md#42-main-results) |
| Section 5 (Analysis) | ⭐⭐⭐ | ~5 min | [Link](../2601.22149.md#5-analysis) |
| Section 3.1 (Formulation) | ⭐⭐ | ~3 min | [Link](../2601.22149.md#31-problem-formulation) |
| Section 4.1 (Setups) | ⭐⭐ | ~3 min | [Link](../2601.22149.md#41-setups) |
| Section 1 (Intro) | ⭐ | ~5 min | [Link](../2601.22149.md#1-introduction) |
| Appendix A | ⭐ | ~5 min | [Link](../2601.22149.md#appendix) |
| Section 2 (Related Work) | Skip | - | [Link](../2601.22149.md#2-related-work) |
| Section 6 (Conclusion) | Skip | - | [Link](../2601.22149.md#6-conclusion) |

---

## Navigation

<- [Back to Main Analysis](../2601.22149-analysis.md)
