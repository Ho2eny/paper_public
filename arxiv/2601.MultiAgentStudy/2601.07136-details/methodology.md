---
parent: ../2601.07136-analysis.md
source: ../2601.07136.md
created: 2026-02-06
---

# Methodology - Deep Analysis

## Paper Type
**Empirical Study (Mining Software Repositories)**

이 논문은 survey도 method 제안도 아닌, MSR(Mining Software Repositories) 방법론 기반의 대규모 실증 연구이다. 기존 소프트웨어 공학에서 확립된 저장소 마이닝 기법을 새로운 도메인(MAS)에 적용했다.

---

## Approach Overview

### High-Level Summary

연구는 두 단계로 구성된다. 첫째, GitHub에서 8개 대표적 MAS 프레임워크의 커밋과 이슈 데이터를 체계적으로 수집/전처리한다. 둘째, 두 가지 Research Question(개발 패턴 + 이슈 패턴)에 대해 정량적 분석을 수행한다. 커밋 분류에는 NLP 모델(DistilBERT)을, 이슈 토픽 추출에는 BERTopic을 활용하여, 수작업 분석의 한계를 극복한다.

### Key Innovation

MAS 도메인에 MSR 방법론을 최초 적용한 것 자체가 주요 혁신. 특히 (1) 커밋의 유지보수 유형 자동 분류와 (2) 이슈의 자동 토픽 모델링을 결합하여 다층적 분석을 수행한 점이 차별화된다.

---

## Technical Details

### Data Collection

| Source | Method | Raw Count |
|--------|--------|-----------|
| Issues | GitHub GraphQL API | 10,813 closed issues |
| Commits | Git clone + extraction | 44,041 commits |
| Selection criteria | GitHub stars (16K~119K), architectural diversity | 8 frameworks |

- Source: [III.B.1 Data Collection](../2601.07136.md#b-dataset-construction)

### Preprocessing Pipeline

```
Issues:  10,813 → (PR 연관 필터) → 4,731  (56% 감소)
Commits: 44,041 → (중복 제거)    → 42,267 (4% 감소)
Labels:  8,519  → (PR+Label)    → 3,793  (55% 감소)
```

- **이슈 필터링**: PR이 연결된 이슈만 포함 (코드 변경이 수반된 이슈에 집중)
- **커밋 중복 제거**: Cherry-pick/rebase로 인한 동일 내용 다른 해시 커밋 제거
- Source: [III.B.2 Preprocessing](../2601.07136.md#b-dataset-construction)

### Analysis Methods

#### RQ1: Development Patterns

| Method | Tool | Purpose |
|--------|------|---------|
| 누적 커밋 그래프 | 시계열 분석 | 개발 궤적 비교 |
| Coefficient of Variation | 통계 | 월별 커밋 규칙성 측정 |
| Code Churn 분석 | Git diff | 추가/삭제 라인, 변경 파일 추적 |
| 커밋 유형 분류 | Fine-tuned DistilBERT | Perfective/Corrective/Adaptive 자동 분류 |

**DistilBERT 커밋 분류:**
- 모델: Fine-tuned DistilBERT (Sarwar et al. 2020)
- 학습 데이터: GitHub 커밋 메시지
- 6가지 분류: P, C, A, AP, CP, CA
- Source: [IV.A.2](../2601.07136.md#a-development-and-contribution-patterns-rq1)

#### RQ2: Issue Patterns

| Method | Tool | Purpose |
|--------|------|---------|
| 레이블 정규화 | 수작업 + 자동화 | 프레임워크간 이슈 유형 비교 |
| 해결 시간 분석 | 통계 | 중앙값, IQR, 분포 특성 |
| 토픽 모델링 | BERTopic | 에이전트 이슈의 세부 토픽 추출 |

**BERTopic 분석:**
- 대상: 에이전트 관련 이슈 (1,049개)
- 결과: 13개 토픽 클러스터
- 검증: 대표 이슈에 대한 수작업 정성 검증
- Source: [IV.B.2](../2601.07136.md#b-issue-landscape-rq2)

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| 프레임워크 선정 | GitHub 스타 기반 8개 | 생태계 대표성 확보 | 커밋/이슈 수 기반, 다운로드 수 기반 | [III.B.1](../2601.07136.md#b-dataset-construction) |
| 이슈 필터링 | PR 연결 이슈만 포함 | 코드 변경 수반 이슈에 집중 | 전체 이슈 분석, 코멘트 수 기반 필터 | [III.B.2](../2601.07136.md#b-dataset-construction) |
| 커밋 분류 | DistilBERT 자동 분류 | 대규모 데이터 처리, 일관성 | 수작업 분류, GPT 기반 분류, 키워드 기반 | [IV.A.2](../2601.07136.md#a-development-and-contribution-patterns-rq1) |
| 토픽 추출 | BERTopic | 비지도 학습, 의미적 일관성 | LDA, NMF, GPT 기반 분류 | [IV.B.2](../2601.07136.md#b-issue-landscape-rq2) |

---

## Limitations & Assumptions

### Assumptions
1. GitHub 스타 수가 프레임워크의 인기/중요도를 반영한다
2. PR이 연결된 이슈가 "의미 있는" 이슈를 대표한다
3. 커밋 메시지가 변경의 성격을 정확히 반영한다
4. DistilBERT 분류가 수작업 분류와 유사한 품질을 가진다

### Limitations
1. **오픈소스 편향**: 독점 MAS 시스템(GPT API 기반 에이전트, 기업 내부 프레임워크)을 배제
2. **스냅샷 한계**: 특정 시점의 데이터로, 생태계의 동적 변화를 추적하지 못함
3. **정성적 분석 부재**: 코드 품질, 아키텍처 결정의 합리성, 개발자 만족도 등 미포함
4. **분류 오류**: 자동 커밋 분류의 정확도가 완벽하지 않으며, 검증이 부분적
5. **이슈 레이블 불균형**: 프레임워크별 레이블링 관행이 상이하여 직접 비교에 한계

---

## Source References

- Research Questions: [III.A](../2601.07136.md#a-study-objective-and-research-questions)
- Data Collection: [III.B.1](../2601.07136.md#b-dataset-construction)
- Preprocessing: [III.B.2](../2601.07136.md#b-dataset-construction)
- Commit Classification: [IV.A.2](../2601.07136.md#a-development-and-contribution-patterns-rq1)
- Topic Modeling: [IV.B.2](../2601.07136.md#b-issue-landscape-rq2)
- Threats: [V. Threats to Validity](../2601.07136.md#v-threats-to-validity)
