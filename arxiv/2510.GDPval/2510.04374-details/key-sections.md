---
parent: ../2510.04374-analysis.md
source: ../2510.04374.md
type: detail
section: key-sections
---

# Section Reading Priority Guide

> **Source**: Full paper structure analysis

## Reading Strategy Overview

GDPval은 **벤치마크 논문**으로, 태스크 설계(Section 2)와 헤드라인 실험 결과(Section 3)가 핵심이다. Appendix에 상당한 분량의 추가 분석과 방법론적 세부사항이 포함되어 있으며, 특히 속도/비용 분석(A.2.1)과 직업별/부문별 분석(A.2.2-A.2.5)이 유용하다.

---

## Priority 1: Must Read (핵심 섹션)

### Section 2: Task Creation
> **Source**: [Section 2](../2510.04374.md#2-task-creation)

**읽어야 하는 이유**: 벤치마크의 핵심 설계 철학과 태스크 구축 과정 전체 이해에 필수

**핵심 하위 섹션**:
| 섹션 | 내용 | 중요도 |
|------|------|--------|
| 2.1 | 직업 우선순위 선정 (GDP 기반 top-down) | ★★★★★ |
| 2.2 | 전문가 모집 (14년 경력, <10% 선정률) | ★★★★☆ |
| 2.3 | 태스크 구성 (요청 + 참조파일 + 산출물) | ★★★★★ |
| 2.4 | 품질 관리 파이프라인 (평균 5회 리뷰) | ★★★★☆ |
| 2.5 | 전문가/자동 채점 방법 | ★★★★★ |

**예상 읽기 시간**: 15-20분

---

### Section 3.1: Headline Results
> **Source**: [Section 3.1](../2510.04374.md#31-headline-results)

**읽어야 하는 이유**: 핵심 실험 결과와 모델별 성능 비교

**핵심 내용**:
- Claude Opus 4.1: 47.6% 승률 (최고)
- GPT-5: 39.0% (정확성 최고)
- 모델 산출물이 절반 이상 태스크에서 전문가와 동등 이상
- 시간에 따른 대략 선형적 성능 향상

**예상 읽기 시간**: 10분

---

### Section 3.3: Model Strengths and Weaknesses
> **Source**: [Section 3.3](../2510.04374.md#33-model-strengths-and-weaknesses)

**읽어야 하는 이유**: 모델별 실패 패턴 이해를 통한 실무 적용 인사이트

**핵심 분석**:
| 분석 주제 | 발견 |
|-----------|------|
| 공통 패배 원인 | 지시 따르기 실패 (Claude, Gemini, Grok) |
| GPT-5 특이점 | 포맷 오류가 주요 원인, 지시 따르기는 최소 |
| Claude 강점 | 미학적 품질 (PDF, PPT, XLSX 포맷) |
| GPT-5 강점 | 순수 텍스트 태스크에서 우위 |

**예상 읽기 시간**: 8-10분

---

## Priority 2: Recommended (권장 섹션)

### Section 3.2: Speed and Cost Comparison
> **Source**: [Section 3.2](../2510.04374.md#32-speed-and-cost-comparison)

**읽어야 하는 이유**: AI 모델 도입의 실질적 경제성 분석

**핵심 포인트**:
- Try-n-times 시나리오에서 GPT-5 기준 1.63배 비용 절감
- 검토 시간 포함 시 순수 속도 비율(수백배)이 현실적 수준(1-2배)으로 급감
- 검토 비용과 재수행 확률이 핵심 변수

**예상 읽기 시간**: 10분 (수식 포함 시 A.2.1과 함께 20분)

---

### Section 3.4: Increasing Reasoning Effort and Scaffolding
> **Source**: [Section 3.4](../2510.04374.md#34-increasing-reasoning-effort-and-scaffolding)

**읽어야 하는 이유**: 프롬프트/스캐폴딩으로 성능 개선 가능성 확인

**핵심 결과**:
- 추론 노력 증가 → 예측 가능한 성능 향상
- 프롬프트 튜닝 → 승률 5%p 향상
- 멀티모달 자기 검사 비율: 15% → 97%
- 블랙스퀘어 아티팩트 완전 제거

**예상 읽기 시간**: 8분

---

### Section 5: Limitations
> **Source**: [Section 5](../2510.04374.md#5-limitations)

**읽어야 하는 이유**: 벤치마크의 한계를 이해해야 결과를 올바르게 해석 가능

**핵심 한계**:
- 44개 직업, 직업당 30개 태스크 → 제한적 범위
- 디지털 지식 노동에 국한 → 물리적 노동, 대인 소통 미반영
- 일회성(one-shot) 설정 → 실제 업무의 상호작용성 미반영
- 채점 비용 → 전문가 비교 채점의 확장성 한계

**예상 읽기 시간**: 5분

---

## Priority 3: Optional (선택 섹션)

### Section 1: Introduction
> **Source**: [Section 1](../2510.04374.md#1-introduction)

**스킵 가능 이유**: Abstract에서 대부분 내용 파악 가능
**읽어야 할 경우**: 후행 지표 vs 선행 지표 논증을 상세히 이해하고 싶을 때

---

### Section 4: Open-Sourcing
> **Source**: [Section 4](../2510.04374.md#4-open-sourcing)

**스킵 가능 이유**: 짧은 섹션, 골드 서브셋 공개 및 자동 채점기 안내
**읽어야 할 경우**: 실제 벤치마크를 사용하려 할 때

---

### Section 6: Conclusion
> **Source**: [Section 6](../2510.04374.md#6-conclusion)

**스킵 가능 이유**: Abstract와 대부분 중복, 5가지 기여 요약

---

## Priority 4: Reference Only (참고용)

### Appendix A.2.2-A.2.5: 세부 분석
> **Source**: [Section A.2.2](../2510.04374.md#a22-win-rates-by-sector) ~ [A.2.5](../2510.04374.md#a25-win-rates-by-time-to-complete)

**용도**: 부문별, 직업별, 파일 유형별, 태스크 시간별 승률 세부 분석

### Appendix A.3: 프롬프트 튜닝 상세
> **Source**: [Section A.3](../2510.04374.md#a3-additional-detail-on-prompt-tuning)

**용도**: 에이전트 프롬프트 설계 참고. 포맷 체크 5단계, 멀티모달 검사 방법론

### Appendix A.6: 자동 채점기 상세
> **Source**: [Section A.6](../2510.04374.md#a6-automated-grader-details)

**용도**: 자동 채점기 일치율 메트릭, 한계, 설치 패키지 목록

### Appendix A.7: 직업 선정 방법론 상세
> **Source**: [Section A.7](../2510.04374.md#a7-further-methodological-details-on-selecting-occupations)

**용도**: O*NET 데이터 매핑, 디지털 직업 분류, 누락 데이터 처리 방법

---

## Quick Reading Path

```
[10분 핵심 파악]
Abstract → Figure 5 (승률) → Figure 8 (실패 유형) → Table 2 (속도/비용)

[30분 심층 이해]
Abstract → Section 2 (전체) → Section 3.1 → Section 3.3 → Section 5

[1시간 완전 이해]
Abstract → Sections 1-6 순서대로 → A.2.1 (속도/비용 수식) → A.2.7 (under-context)
```

---

## Figure/Table Priority

| 항목 | 중요도 | 내용 |
|------|--------|------|
| Figure 5 | ★★★★★ | 모델별 승률 (헤드라인 결과) |
| Figure 8 | ★★★★★ | 모델별 실패 유형 클러스터링 |
| Figure 6 | ★★★★☆ | 시간에 따른 성능 향상 추세 |
| Table 2 | ★★★★☆ | 속도/비용 개선 비율 |
| Figure 1 | ★★★★☆ | 태스크 예시 (벤치마크 이해) |
| Figure 7 | ★★★★☆ | 속도/비용 절감 시각화 |
| Figure 9 | ★★★☆☆ | 추론 노력/프롬프트 실험 |
| Figure 4 | ★★★☆☆ | 채점 방법 및 일치율 |
| Table 3-7 | ★★★☆☆ | 태스크 통계, 파일 수, O*NET 커버리지 |
| Figure 10-13 | ★★☆☆☆ | 부문별/직업별/유형별/시간별 승률 |

---

## Navigation

- [Main Analysis](../2510.04374-analysis.md)
- [TL;DR](./tldr.md)
- [Contributions Detail](./contributions.md)
- [Methodology Detail](./methodology.md)
