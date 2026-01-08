---
parent: ../2601.07136-analysis.md
source: ../2601.07136.md
created: 2026-02-06
---

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

**Total Reading Time**: ~25 minutes (전체) / ~15 minutes (핵심만)

---

## Must Read

### Section IV: Results (A & B)

- **Why Read**: 논문의 핵심 가치가 집중된 섹션. 4개 Finding과 8개 Figure/Table이 모든 연구 결과를 담고 있음
- **Key Points**:
  - IV.A.1: 세 가지 개발 프로파일 식별 (sustained, steady, burst-driven) + 코드 변동 분석
  - IV.A.2: 커밋 유형 분류 (Perfective 40.8% 우세) + 프레임워크별 차이
  - IV.B.1: 이슈 보고 패턴 + 해결 시간 분석 (중앙값 1일~10일+)
  - IV.B.2: 이슈 유형 분류 + BERTopic 토픽 분석
- **Time**: ~10 min
- **Link**: [IV. Results](../2601.07136.md#iv-results)

---

## Important

### Section I: Introduction

- **Why Read**: 연구 동기, 2개 RQ의 맥락, 8개 프레임워크 개요(Table I)가 결과 이해의 필수 배경
- **Key Points**:
  - 연구 갭: MAS의 소프트웨어 공학적 실증 연구 부재
  - Table I: 8개 프레임워크의 아키텍처, 목적, GitHub 인기도 비교
  - RQ1(개발 패턴) + RQ2(이슈 패턴)의 정의
- **Time**: ~5 min
- **Link**: [I. Introduction](../2601.07136.md#i-introduction)

### Section III: Methodology

- **Why Read**: 데이터셋 구축 과정과 분석 방법론의 이해가 결과 해석에 필요
- **Key Points**:
  - 데이터 수집: GraphQL API + git clone
  - 전처리: PR 기반 필터링 (10,813 -> 4,731 이슈), 중복 커밋 제거 (44,041 -> 42,267)
  - Table II: 프레임워크별 데이터 구성
- **When to Read**: 결과의 타당성이나 재현성에 관심이 있을 때
- **Time**: ~3 min
- **Link**: [III. Methodology](../2601.07136.md#iii-methodology)

---

## Reference

### Section II: Background and Related Work

- **Content**: MAS 프레임워크 개요(II.A) + 소프트웨어 저장소 마이닝 관련 연구(II.B)
- **When Useful**: MAS 배경지식이 부족하거나, 관련 연구 흐름을 파악하고 싶을 때
- **Note**: AutoGen, CrewAI, LangChain 등을 이미 알고 있다면 건너뛸 수 있음
- **Link**: [II. Background and Related Work](../2601.07136.md#ii-background-and-related-work)

### Section V: Threats to Validity

- **Content**: Internal, External, Construct, Reliability의 4가지 타당성 위협 분석
- **When Useful**: 연구 결과의 한계를 정확히 파악하거나, 후속 연구 설계에 참고할 때
- **Key Limitation**: GitHub 오픈소스만 분석하여 독점 MAS 시스템에 일반화 어려움
- **Link**: [V. Threats to Validity](../2601.07136.md#v-threats-to-validity)

---

## Skip

### Section VI: Conclusions

- **Why Skip**: Introduction과 Results 섹션의 핵심 내용을 요약한 것이므로, 앞선 섹션을 읽었다면 새로운 정보가 거의 없음
- **Alternative**: 메인 분석의 Overview 참조
- **Link**: [VI. Conclusions](../2601.07136.md#vi-conclusions)

---

## Recommended Reading Order

1. **[I. Introduction](../2601.07136.md#i-introduction)** - RQ와 프레임워크 배경 (5분)
2. **[IV.A. Development Patterns](../2601.07136.md#a-development-and-contribution-patterns-rq1)** - 핵심 결과 1: 개발 프로파일 + 커밋 분류 (5분)
3. **[IV.B. Issue Landscape](../2601.07136.md#b-issue-landscape-rq2)** - 핵심 결과 2: 이슈 분석 + 토픽 모델링 (5분)
4. *(선택)* [III. Methodology](../2601.07136.md#iii-methodology) - 방법론 상세 (3분)
5. *(선택)* [V. Threats to Validity](../2601.07136.md#v-threats-to-validity) - 한계점 (2분)
