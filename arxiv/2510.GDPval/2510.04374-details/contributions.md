---
parent: ../2510.04374-analysis.md
source: ../2510.04374.md
type: detail
section: contributions
---

# Core Contributions Deep Analysis

> **Source**: [Introduction](../2510.04374.md#1-introduction), [Sections 2-4](../2510.04374.md#2-task-creation)

## Primary Contributions

### 1. 경제적 가치 기반 벤치마크 데이터셋
> **Source**: [Section 2](../2510.04374.md#2-task-creation)

**기여 내용**:
- 미국 GDP 상위 9개 부문, 44개 직업군에서 1,320개 태스크 구축
- 연간 총 보상 $3T에 해당하는 직업군 커버
- O*NET 직업 태스크의 체계적 커버리지 보장 (71.4% 스킬, 63.4% 업무활동)

**설계 방법론**:
1. GDP 5% 이상 기여 부문 선정 (Federal Reserve 데이터)
2. 부문별 총 보상 상위 + 디지털 60% 이상 직업 5개 선정
3. 전문가(평균 14년 경력)가 실제 업무 기반 태스크 구축
4. 평균 5회 인간 리뷰 + 모델 기반 자동 스크리닝

**혁신성**: 기존 벤치마크가 학술 시험 스타일(MMLU) 또는 단일 도메인(SWE-Lancer)에 집중한 반면, 경제 전반을 체계적으로 커버하는 최초의 접근

**Impact Score**: ★★★★★ (핵심 기여)

---

### 2. 전문가 블라인드 비교 평가 체계
> **Source**: [Section 2.5](../2510.04374.md#25-human-expert-grading-and-automated-grading)

**기여 내용**:
- 블라인드 pairwise comparison으로 전문가가 2개 이상 산출물 순위 결정
- 태스크당 평균 1시간 이상 채점, 상세 근거(justification) 작성
- 승률 기반 메트릭으로 포화 없는 연속적 평가 가능

**평가 설계의 장점**:
| 특성 | GDPval | 기존 벤치마크 |
|------|--------|---------------|
| 포화 가능성 | 없음 (베이스라인 교체 가능) | 높음 (정답 기반) |
| 주관성 반영 | 구조, 스타일, 미학 포함 | 객관적 정답만 |
| 평가 근거 | 전문가 상세 근거 제공 | 없음 |
| 다양한 정답 | 허용 | 제한적 |

**혁신성**: 단순 정답 비교가 아닌 전문가 수준의 품질 판단을 메트릭화

**Impact Score**: ★★★★★ (핵심 기여)

---

### 3. 속도/비용 분석 프레임워크
> **Source**: [Section 3.2](../2510.04374.md#32-speed-and-cost-comparison), [Section A.2.1](../2510.04374.md#a21-speed-and-cost-analysis-continued)

**기여 내용**:
3가지 시나리오별 시간/비용 절감 비율 정량화:

**Naive ratio**: $H_T / M_T$ (인간 vs 모델 순수 시간 비교)
- GPT-5: 90배 빠름, 474배 저렴

**Try 1 time**: 모델 시도 → 검토 → 실패 시 인간 수행
$$\mathbb{E}[T_{1,i}] = M_{T,i} + R_{T,i} + (1 - w_i)H_{T,i}$$
- GPT-5: 1.12배 빠름, 1.18배 저렴

**Try n times**: 여러 번 시도 후 인간 수행
$$\mathbb{E}[T_{n,i}] = (M_{T,i} + R_{T,i})\frac{1 - (1 - w_i)^n}{w_i} + (1 - w_i)^n H_{T,i}$$
- GPT-5: 1.39배 빠름, 1.63배 저렴

**혁신성**: 단순 성능 비교를 넘어 실제 도입 시나리오별 경제적 가치를 수학적으로 모델링

**Impact Score**: ★★★★☆

---

### 4. 실험적 자동 채점기
> **Source**: [Section 2.5](../2510.04374.md#25-human-expert-grading-and-automated-grading), [Section A.6](../2510.04374.md#a6-automated-grader-details)

**기여 내용**:
- GPT-5-high 기반 pairwise 자동 채점기
- 인간 전문가와 66% 일치율 (인간 간 일치율 71%의 5% 이내)
- evals.openai.com에서 공개 서비스 제공

**한계**:
- 12/220 태스크는 채점 불가 (인터넷 필요, 비Python 코드, 폰트 등)
- OpenAI 모델 평가 시 자사 편향 가능성 (Panickssery et al., 2024)
- 비OpenAI 모델에서 일치율이 더 높음

**Impact Score**: ★★★★☆

---

### 5. 모델 강약점 클러스터링 분석
> **Source**: [Section 3.3](../2510.04374.md#33-model-strengths-and-weaknesses)

**기여 내용**:
- 전문가 판단 근거(justification)를 클러스터링하여 모델별 실패 유형 분류
- 4개 모델(GPT-5 high, Claude Opus 4.1, Gemini 2.5 Pro, Grok 4) 비교

**핵심 발견**:
| 모델 | 주요 패배 원인 | 상대적 강점 |
|------|----------------|-------------|
| GPT-5 | 포맷 오류 | 정확성, 지시 따르기 |
| Claude Opus 4.1 | 지시 따르기 | 미학(PPT, PDF), 파일 처리 |
| Gemini 2.5 Pro | 지시 따르기, 산출물 미제공 | - |
| Grok 4 | 지시 따르기, 참조 데이터 무시 | 정확성 (GPT-5과 유사) |

**Impact Score**: ★★★☆☆

---

## Secondary Contributions

### 6. 추론 노력 및 프롬프트 튜닝 실험
> **Source**: [Section 3.4](../2510.04374.md#34-increasing-reasoning-effort-and-scaffolding)

- 추론 노력 증가 (low → medium → high): 성능 예측 가능하게 향상
- 프롬프트 튜닝: 승률 5%p 향상, 블랙스퀘어 아티팩트 완전 제거
- Best-of-N 샘플링 (N=4, GPT-5 judge): 추가 성능 향상
- 멀티모달 검사 비율: 15% → 97% (프롬프트 효과)

### 7. Under-contextualized 실험
> **Source**: [Section A.2.7](../2510.04374.md#a27-under-contextualized-gdpval)

- 프롬프트를 원본의 42% 길이로 축소
- 모델 성능 하락 확인 → 맥락 파악 능력의 중요성 실증

### 8. 디지털 직업 분류 방법론 검증
> **Source**: [Section A.7.1](../2510.04374.md#a71-validating-the-digital-tasks-measure)

- Acemoglu & Autor (2011) 프레임워크와의 상관관계 검증
- 디지털 태스크 비율 ↑ → 비일상적 인지 업무 ↑, 수작업 ↓

---

## Contribution Hierarchy

```
[Tier 1: Core Contributions]
├── 경제적 가치 기반 벤치마크 데이터셋 (★★★★★)
└── 전문가 블라인드 비교 평가 체계 (★★★★★)

[Tier 2: Important Contributions]
├── 속도/비용 분석 프레임워크 (★★★★☆)
└── 실험적 자동 채점기 (★★★★☆)

[Tier 3: Supporting Contributions]
├── 모델 강약점 클러스터링 (★★★☆☆)
├── 추론 노력/프롬프트 실험 (★★★☆☆)
└── 디지털 직업 분류 검증 (★★★☆☆)
```

---

## Navigation

- [Main Analysis](../2510.04374-analysis.md)
- [TL;DR](./tldr.md)
- [Key Sections Guide](./key-sections.md)
- [Methodology Detail](./methodology.md)
