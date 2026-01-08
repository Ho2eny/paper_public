---
parent: ../2601.07136-analysis.md
source: ../2601.07136.md
created: 2026-02-06
---

# Core Contributions - Deep Analysis

## Contribution 1: 최초의 MAS 대규모 실증 연구

### What
8개 주요 오픈소스 MAS 프레임워크(AutoGen, CrewAI, Haystack, LangChain, Letta, LlamaIndex, Semantic Kernel, SuperAGI)를 대상으로, 42,267개 고유 커밋과 4,731개 해결된 이슈를 분석한 최초의 대규모 실증 연구. GitHub 스타 16K~119K 범위의 프레임워크를 선정하여 생태계의 대표성을 확보했다.

### Technical Details
- **데이터 수집**: GitHub GraphQL API로 10,813개 closed issue 추출 + git clone으로 44,041개 커밋 수집
- **전처리**: PR 없는 이슈 제외 (10,813 -> 4,731), cherry-pick/rebase 중복 커밋 제거 (44,041 -> 42,267)
- **분석 범위**: 커밋 활동 패턴, 코드 변동(churn), 커밋 유형 분류, 이슈 레이블 분석, 해결 시간, 토픽 모델링

### Why It Matters
기존 MAS 연구는 알고리즘과 아키텍처 혁신에 집중했으며, 소프트웨어 공학적 관점의 실증 분석이 전무했다. 이 연구는 MSR(Mining Software Repositories) 방법론을 MAS 도메인에 최초로 적용함으로써, 개발 관행의 정량적 이해를 가능하게 했다.

### Source
- Dataset: [III.B. Dataset Construction](../2601.07136.md#b-dataset-construction)
- Framework selection: [Table I](../2601.07136.md#i-introduction)

---

## Contribution 2: 세 가지 개발 프로파일 식별

### What
MAS 프레임워크들의 커밋 활동과 코드 변동 패턴을 분석하여, 세 가지 뚜렷한 개발 프로파일을 식별:
1. **Sustained** (지속적 고강도): Haystack - CV 48.6%, 2020년부터 꾸준한 성장과 주기적 리팩토링
2. **Steady** (안정적 균형): Semantic Kernel, LlamaIndex - 중간 수준 변동, 균형 잡힌 활동
3. **Burst-driven** (폭발적 집중): SuperAGI - CV 456.1%, 2023년 중반 폭발적 활동 후 급감

### Technical Details
- **Coefficient of Variation (CV)**: 월별 커밋 수의 변동계수로 개발 일관성 측정
- **Code Churn 분석**: 추가/삭제 라인 수, 변경 파일 수의 시계열 분석
- SuperAGI: 초기 ~300만 라인 추가 후 유지보수 최소화 (rapid prototyping)
- Haystack: ~140K, ~290K 라인 규모의 의도적 삭제 이벤트 (리팩토링)
- LangChain: 반복적 churn 피크와 ~400K 라인 삭제 (아키텍처 재구성)

### Why It Matters
개발 프로파일은 프레임워크의 성숙도, 기여자 참여도, 안정성을 반영한다. 프레임워크 채택 의사결정 시 정량적 기준을 제공하며, 오픈소스 프로젝트의 건강성 평가 프레임워크로 활용 가능하다.

### Source
- Commit patterns: [Figure 2, 3](../2601.07136.md#a-development-and-contribution-patterns-rq1)
- Code churn: [Figure 4, 5](../2601.07136.md#a-development-and-contribution-patterns-rq1)
- Finding 1.1: [IV.A.1](../2601.07136.md#a-development-and-contribution-patterns-rq1)

---

## Contribution 3: 커밋 및 이슈 유형의 정량적 분석

### What
DistilBERT 기반 자동 분류로 42,266개 커밋을 유형별로 분류하고, 이슈 레이블을 정규화하여 MAS 생태계의 개발/유지보수 특성을 정량화:
- **커밋**: Perfective 40.83% > Corrective 27.36% > Adaptive 24.30%
- **이슈**: Bug 22% > Infrastructure 14% > Data Processing 11% > Agent Issues 10%

### Technical Details
- **커밋 분류**: Fine-tuned DistilBERT 모델 사용, 3가지 기본 유형 + 3가지 복합 유형
  - Perfective: 기능 향상 (34.2%~51.5%, 프레임워크별)
  - Corrective: 결함 수정 (18.3%~33.5%)
  - Adaptive: 환경 변화 대응 (17.2%~27%)
- **이슈 토픽 모델링**: BERTopic으로 에이전트 관련 이슈 클러스터링
  - Agent Systems & Intelligence: 58.42%
  - Technical Implementation & Operations: 32.51%

### Why It Matters
MAS 개발이 전통적 유지보수(reactive bug fix)보다 기능 확장(proactive improvement)에 치중한다는 점을 정량적으로 입증. 이는 빠르게 진화하는 AI 분야의 특성을 반영하지만, 동시에 기술 부채 누적 위험을 시사한다.

### Source
- Commit classification: [Table III, IV](../2601.07136.md#a-development-and-contribution-patterns-rq1)
- Issue labels: [Table V](../2601.07136.md#b-issue-landscape-rq2)
- Topic analysis: [Table VI](../2601.07136.md#b-issue-landscape-rq2)

---

## Contribution 4: 2023년 전환점 발견

### What
커밋과 이슈 데이터 모두에서 2023년을 기점으로 활동이 급증하는 inflection point를 발견. 이는 LLM 확산(ChatGPT 등)과 MAS 생태계 성장의 직접적 상관관계를 실증적으로 보여준다.

### Technical Details
- 대부분의 프레임워크가 2023년에 개발을 강화 (Figure 3 sparklines)
- 생태계 수준 누적 삽입/삭제가 2020-2023 사이 가장 가파른 성장 (Figure 5)
- 모든 주요 이슈 카테고리(Bug, Infrastructure, Agent Issues)의 성장 inflection point가 2023년에 집중 (Figure 8)

### Why It Matters
ChatGPT(2022.11) 출시 이후 LLM 기반 에이전트에 대한 관심이 MAS 프레임워크의 실제 개발 활동으로 이어졌음을 데이터로 확인. 생태계의 급속한 성장이 동시에 품질/안정성 과제를 가져왔음을 시사한다.

### Source
- Commit inflection: [Figure 3](../2601.07136.md#a-development-and-contribution-patterns-rq1)
- Ecosystem growth: [Figure 5](../2601.07136.md#a-development-and-contribution-patterns-rq1)
- Issue growth: [Figure 8](../2601.07136.md#b-issue-landscape-rq2)

---

## Contributions Summary

| # | Contribution | Impact | Source |
|---|--------------|--------|--------|
| 1 | 최초의 MAS 대규모 실증 연구 | 연구 분야 개척 | [III](../2601.07136.md#iii-methodology) |
| 2 | 세 가지 개발 프로파일 식별 | 프레임워크 평가 기준 | [IV.A](../2601.07136.md#a-development-and-contribution-patterns-rq1) |
| 3 | 커밋/이슈 유형 정량 분석 | 생태계 특성 규명 | [IV.A-B](../2601.07136.md#iv-results) |
| 4 | 2023년 전환점 발견 | LLM-MAS 상관관계 실증 | [IV](../2601.07136.md#iv-results) |
