---
parent: ../2510.04374-analysis.md
source: ../2510.04374.md
type: detail
section: methodology
---

# Detailed Methodology Analysis

> **Source**: [Sections 2-3](../2510.04374.md#2-task-creation)

## 1. 직업 선정 방법론

### Top-Down Approach
> **Source**: [Section 2.1](../2510.04374.md#21-prioritizing-occupations)

**단계별 프로세스**:
```
[Step 1] GDP 기여 5% 이상 부문 선정
    ↓  (Federal Reserve Bank Q2 2024 데이터)
[Step 2] 부문별 상위 5개 디지털 직업 선정
    ↓  (총 보상 기준 + 디지털 60% 이상)
[Step 3] O*NET 태스크 매핑
    ↓  (직업별 업무활동 커버리지 확인)
[Step 4] 전문가 모집 및 태스크 구축
```

### 디지털 직업 분류
> **Source**: [Section A.7](../2510.04374.md#a7-further-methodological-details-on-selecting-occupations)

**방법**: GPT-4o가 O*NET 태스크를 디지털/비디지털 분류 → 가중 디지털 비율 60% 이상이면 디지털 직업

**가중치 계산**:
- Adjusted Task Score = mean(normalized_relevance, normalized_frequency, normalized_importance)
- Weighted Task Share = Adjusted Task Score / sum(모든 태스크의 Adjusted Task Score)
- Digital Share = sum(디지털 태스크의 Weighted Task Share)

**검증**: Acemoglu & Autor (2011) 프레임워크와 비교
- 디지털 비율 ↑ → 비일상적 인지 업무 점수 ↑
- 디지털 비율 ↑ → 수작업/일상적 업무 점수 ↓

### 9개 부문 구성
> **Source**: [Table 1](../2510.04374.md#table-1-sectors-their-value-added)

| 부문 | GDP 기여(%) | 직업 수 |
|------|-------------|---------|
| Real Estate & Leasing | 13.8% | 5 |
| Government | 11.3% | 5 |
| Manufacturing | 10.0% | 5 |
| Professional Services | 8.1% | 5 |
| Healthcare & Social Assistance | 7.6% | 5 |
| Finance & Insurance | 7.4% | 5 |
| Retail Trade | 6.3% | 4 |
| Wholesale Trade | 5.8% | 5 |
| Information | 5.4% | 5 |

---

## 2. 태스크 구축 파이프라인

### 태스크 구조
> **Source**: [Section 2.3](../2510.04374.md#23-task-creation)

각 GDPval 태스크의 구성:
```
[Request]
├── 자연어 지시문
├── Reference Files (0-38개, 골드 서브셋 최대 17개)
│   ├── 스프레드시트, 문서, 프레젠테이션
│   ├── 이미지, 오디오, 비디오
│   └── CAD 파일, 소셜미디어 포스트 등
└── O*NET 태스크 분류

[Deliverable]
├── 전문가 작업 산출물
├── 품질 평가 (1-5)
├── 난이도 (1-5)
├── 대표성 (1-5)
└── 예상 완료 시간
```

### 태스크 통계
> **Source**: [Table 3-4](../2510.04374.md#table-3-summary-statistics-for-gdpval-gold-subset-tasks)

| 항목 | Gold Subset (평균) | Full Set (평균) |
|------|-------------------|-----------------|
| 전체 품질 (1-5) | 4.47 | 4.55 |
| 난이도 (1-5) | 3.32 | 3.20 |
| 대표성 (1-5) | 4.50 | 4.43 |
| 완료 시간 (시간) | 9.49 | 8.63 |
| 달러 가치 | $398.46 | $391.44 |
| 참조 파일 수 | 1.92 | - |
| 산출물 파일 수 | 1.54 | - |

### 품질 관리 파이프라인
> **Source**: [Section 2.4](../2510.04374.md#24-task-quality-control-pipeline), [Figure 3](../2510.04374.md#figure-3-tasks-undergo-multiple-rounds-of-review)

```
[Round 1] 모델 기반 자동 스크리닝
    ↓  O*NET 관련성, 디지털 수행 가능성, 복잡도, 파일 확인
[Round 2] 제너럴리스트 초기 리뷰
    ↓  프로젝트 요구사항 준수 확인
[Round 3] 직업별 전문가 리뷰
    ↓  직업 대표성, 다른 전문가의 수행 가능성 확인
[Round 4+] 최종 반복 피드백
    ↓  리뷰어와 전문가 간 반복 수정
[평균 5회 리뷰, 최소 3회]
```

---

## 3. 평가 방법론

### 블라인드 Pairwise Comparison
> **Source**: [Section 2.5](../2510.04374.md#25-human-expert-grading-and-automated-grading)

**프로세스**:
1. 전문가에게 요청(request) + 참조파일 제공
2. 2개 이상 산출물을 라벨 없이 제시 (모델명 제거)
3. 전문가가 순위 결정 + 상세 근거 작성
4. 태스크당 평균 1시간 이상 채점

**블라인딩 한계** (footnote 2):
- OpenAI: em dash 사용 경향
- Claude: 1인칭 표현 경향
- Grok: 자기 언급 가능성
- 파일명에서 모델 식별자 제거했으나 스타일 차이 존재

### 모델 샘플링 설정
> **Source**: [Section 3.1](../2510.04374.md#31-headline-results)

| 모델 | 특이 설정 |
|------|-----------|
| GPT-4o, o4-mini, o3, GPT-5 | 웹 검색 + 코드 인터프리터 + 백그라운드 샘플링 |
| Claude Opus 4.1 | UI 기반 (파일 생성/분석 기능 활용) |
| Gemini 2.5 Pro, Grok 4 | 상세 미기재 |

**반복**: 모델당 태스크당 3회 샘플 × 3명 채점 = 9 comparisons/task/model

### 승률(Win Rate) 계산

승률 = 모델 산출물이 인간 산출물보다 우수하다고 판단된 비율

| 점수 | 의미 |
|------|------|
| 1 | 모델 산출물 선호 |
| 0.5 | 동점 (tie) |
| 0 | 인간 산출물 선호 |

---

## 4. 자동 채점기 방법론

### 아키텍처
> **Source**: [Section A.6](../2510.04374.md#a6-automated-grader-details)

- GPT-5-high 기반 pairwise 비교 모델
- 전문가 스타일의 비교 채점 수행 훈련

### 일치율 메트릭
> **Source**: [Section A.6.1](../2510.04374.md#a61-automated-grader-consensus-metrics)

**Human-Automated Grader Agreement**:
$$A_s^{HA} = \mathbb{E}\left[1 - |H - A|\right]$$

**Human Inter-Rater Agreement**:
$$A_s^{HH} = \mathbb{E}\left[1 - |H_1 - H_2|\right]$$

**결과**: $A^{HA}$ = 65.7%, $A^{HH}$ = 70.8% (5.1% 차이)

### 자체 편향
> **Source**: [Section A.6.2](../2510.04374.md#a62-automated-grader-correlation-results)

- GPT-5-high 기반 채점기 → OpenAI 모델 평가 시 자사 선호 편향 가능
- 비OpenAI 모델에서 일치율이 더 높음
- Panickssery et al. (2024): LLM이 자사 생성물을 선호하는 경향 확인

---

## 5. 속도/비용 분석 모델

### 변수 정의
> **Source**: [Section A.2.1](../2510.04374.md#a21-speed-and-cost-analysis-continued)

| 변수 | 의미 | 평균값 (골드 서브셋) |
|------|------|---------------------|
| $H_T$ | 인간 전문가 완료 시간 | 404분 |
| $H_C$ | 인간 전문가 완료 비용 | $361 |
| $R_T$ | 인간 전문가 검토 시간 | 109분 |
| $R_C$ | 인간 전문가 검토 비용 | $86 |
| $M_T$ | 모델 완료 시간 | 모델별 상이 |
| $M_C$ | 모델 완료 비용 | 모델별 상이 |
| $w$ | 모델 승률 | 모델별 상이 |

### 3가지 시나리오

**Naive**: $H_T / M_T$ (순수 시간 비교)

**Try 1x**:
$$\text{Savings} = \frac{H_T}{M_T + R_T + (1-w)H_T}$$

**Try nx** ($n \to \infty$):
$$\text{Savings} = \frac{H_T}{(M_T + R_T) / w}$$

### 분석의 한계
- 인간 산출물의 자체 검토 시간 미포함
- 인간 산출물도 불만족스러울 가능성 미고려
- 치명적 실수(catastrophic mistake)의 비대칭적 비용 미반영
- Claude, Gemini, Grok의 비용 데이터 미확보

---

## 6. Key Design Choices

| 선택 | 결정 | 근거 | Source |
|------|------|------|--------|
| 직업 선정 기준 | GDP 기여 + 디지털 비율 | 경제적 영향력 기반 체계적 접근 | [Section 2.1](../2510.04374.md#21-prioritizing-occupations) |
| 태스크 구축자 | 산업 전문가 (14년 경력) | 현실적 업무 반영 보장 | [Section 2.2](../2510.04374.md#22-expert-recruitment) |
| 평가 메트릭 | 승률 (pairwise comparison) | 포화 없는 연속 평가, 주관성 반영 | [Section 2.5](../2510.04374.md#25-human-expert-grading-and-automated-grading) |
| 베이스라인 | 인간 전문가 산출물 | 실제 업무 수준 기준 | [Section 2.5](../2510.04374.md#25-human-expert-grading-and-automated-grading) |
| 공개 범위 | 220개 골드 서브셋 | 데이터 오염 방지 + 접근성 균형 | [Section 4](../2510.04374.md#4-open-sourcing) |
| 자동 채점기 | GPT-5-high 기반 | 비용 절감, 5% 이내 일치율 | [Section A.6](../2510.04374.md#a6-automated-grader-details) |

---

## Navigation

- [Main Analysis](../2510.04374-analysis.md)
- [TL;DR](./tldr.md)
- [Contributions Detail](./contributions.md)
- [Key Sections Guide](./key-sections.md)
