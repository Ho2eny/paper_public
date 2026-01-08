---
parent: ../2307.13854-analysis.md
source: ../2307.13854.md
type: detail
section: key-sections
---

# Section Reading Priority Guide

> **Source**: Full paper structure analysis

## Reading Strategy Overview

WebArena는 **벤치마크 논문**으로, 환경 설계(Section 2)와 태스크 구성(Section 3)이 핵심이다. 실험 결과(Section 5)는 베이스라인 성능을 보여주는 보조적 역할을 한다.

---

## Priority 1: Must Read (핵심 섹션)

### Section 2: WebArena Environment
> **Source**: [Section 2](../2307.13854.md#2-webarena-websites-as-an-environment-for-autonomous-agents)

**읽어야 하는 이유**: 논문의 핵심 기여인 환경 설계 전체 이해에 필수

**핵심 하위 섹션**:
| 섹션 | 내용 | 중요도 |
|------|------|--------|
| 2.1 | 환경 형식화 $\mathcal{E} = \langle \mathcal{S}, \mathcal{A}, \mathcal{O}, \mathcal{T} \rangle$ | ★★★★★ |
| 2.2 | 웹사이트 선정 (4개 도메인 + 3개 도구) | ★★★★★ |
| 2.3 | 관찰 공간 (멀티탭, Accessibility Tree) | ★★★★☆ |
| 2.4 | 액션 공간 (12가지 액션 타입) | ★★★★☆ |

**예상 읽기 시간**: 15-20분

---

### Section 3.2: Evaluation Annotation
> **Source**: [Section 3.2](../2307.13854.md#32-evaluation-annotation)

**읽어야 하는 이유**: 기능적 정확성 평가의 혁신적 방법론 이해

**핵심 내용**:
- $r_{info}$: exact_match, must_include, fuzzy_match
- $r_{prog}$: locator + validator 시스템
- Unachievable 태스크 설계

**예상 읽기 시간**: 10-15분

---

## Priority 2: Recommended (권장 섹션)

### Section 3.1: Intent Collection
> **Source**: [Section 3.1](../2307.13854.md#31-intent-collection)

**읽어야 하는 이유**: 태스크 설계 철학과 템플릿 기반 생성 방법 이해

**핵심 포인트**:
- Abstract & High-level: 1-2 액션으로 해결 불가능한 태스크
- Creative: 제약 조건 추가로 고유성 확보
- Template: 변수화를 통한 체계적 다양화

**예상 읽기 시간**: 8-10분

---

### Section 5.1: Analysis
> **Source**: [Section 5.1](../2307.13854.md#51-analysis)

**읽어야 하는 이유**: 에이전트 실패 패턴과 한계점 파악

**핵심 분석**:
| 분석 주제 | 발견 |
|-----------|------|
| Early Stopping | GPT-4가 54.9% 실행 가능 태스크를 불가능으로 판단 |
| Template Consistency | GPT-4가 100% 성공한 템플릿은 4개뿐 |
| Observation Bias | 첫 관련 정보에 고착, 검증 부족 |

**예상 읽기 시간**: 10분

---

### Section 6: Related Work
> **Source**: [Section 6](../2307.13854.md#6-related-work)

**읽어야 하는 이유**: 연구 맥락과 차별점 파악

**주요 비교 대상**:
- 정적 환경: Mind2Web, FormWoB, QAWoB
- 단순화 환경: MiniWoB++, WebShop, ALFRED
- 실제 환경: AndroidEnv (재현성 문제)

**예상 읽기 시간**: 5-8분

---

## Priority 3: Optional (선택 섹션)

### Section 1: Introduction
> **Source**: [Section 1](../2307.13854.md#1-introduction)

**스킵 가능 이유**: Abstract에서 대부분 내용 파악 가능
**읽어야 할 경우**: 동기와 문제 정의를 상세히 이해하고 싶을 때

---

### Section 4: Baseline Web Agents
> **Source**: [Section 4](../2307.13854.md#4-baseline-web-agents)

**스킵 가능 이유**: 표준적인 프롬프팅 기법 (CoT, few-shot) 사용
**읽어야 할 경우**: 베이스라인 에이전트 구현을 재현하고 싶을 때

---

### Section 5: Results (Main Table)
> **Source**: [Section 5](../2307.13854.md#5-results)

**스킵 가능 이유**: 숫자만 빠르게 확인하면 됨
**핵심 숫자**: GPT-4 14.41%, Human 78.24%

---

### Section 7: Conclusion
> **Source**: [Section 7](../2307.13854.md#7-conclusion)

**스킵 가능 이유**: Abstract 반복

---

## Priority 4: Reference Only (참고용)

### Appendix A.1-A.3: Implementation Details
> **Source**: [Appendix A.1-A.3](../2307.13854.md#a1-website-implementation)

**용도**: 환경 구축/확장 시 참고

### Appendix A.9: Agent Prompts
> **Source**: [Appendix A.9](../2307.13854.md#a9-the-prompts-of-the-baseline-web-agents)

**용도**: 에이전트 재현 시 참고

### Appendix A.10: Error Analysis
> **Source**: [Appendix A.10](../2307.13854.md#a10-additional-error-analysis)

**용도**: 에이전트 개선 방향 탐색 시 참고

---

## Quick Reading Path

```
[10분 핵심 파악]
Abstract → Figure 1 → Table 4 → Table 2

[30분 심층 이해]
Abstract → Section 2 (전체) → Section 3.2 → Section 5.1

[1시간 완전 이해]
Abstract → Sections 1-7 순서대로 (Appendix 제외)
```

---

## Figure/Table Priority

| 항목 | 중요도 | 내용 |
|------|--------|------|
| Figure 1 | ★★★★★ | WebArena 전체 개요 |
| Figure 2 | ★★★★☆ | 태스크 실행 예시 |
| Table 2 | ★★★★★ | 메인 실험 결과 |
| Table 4 | ★★★★★ | 벤치마크 비교 |
| Figure 3 | ★★★☆☆ | 관찰 공간 시각화 |
| Figure 4 | ★★★☆☆ | 액션 공간 정의 |

---

## Navigation

- [Main Analysis](../2307.13854-analysis.md)
- [TL;DR](./tldr.md)
- [Contributions Detail](./contributions.md)
- [Methodology Detail](./methodology.md)
