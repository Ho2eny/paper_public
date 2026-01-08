---
parent: ../2510.04374-analysis.md
source: ../2510.04374.md
type: detail
section: tldr
---

# Extended TL;DR: GDPval

> **Source**: [Abstract & Introduction](../2510.04374.md#abstract)

## One-Liner
GDPval은 미국 GDP 상위 9개 부문, 44개 직업군의 실제 업무 1,320개를 산업 전문가(평균 14년 경력)가 구축하고, 블라인드 전문가 비교로 AI 모델의 경제적 가치 기반 역량을 측정하는 벤치마크이다.

## Executive Summary (3-5 문장)

AI의 경제적 영향을 측정하는 기존 방법(채택률, GDP 기여도)은 기술 확산에 시간이 걸리므로 후행 지표에 불과하다. GDPval은 AI 모델의 역량을 직접 측정함으로써 경제적 관련성을 채택 이전에 평가할 수 있는 선행 지표를 제안한다. [Section 1](../2510.04374.md#1-introduction)에서 설명하듯이, 전기, 컴퓨터 등 과거 기술 혁신의 경험에 비추어 직접적 역량 측정의 필요성을 논증한다. [Section 3](../2510.04374.md#3-experiments-and-results)의 실험 결과, Claude Opus 4.1이 47.6% 승률로 인간 전문가에 근접하고 있으며, 프론티어 모델 성능은 시간에 따라 대략 선형 증가하고 있다.

## Key Problem Statement
> **Source**: [Section 1 - Introduction](../2510.04374.md#1-introduction)

기존 AI 경제 영향 측정의 세 가지 핵심 한계:

1. **후행 지표 문제**: 채택률, 사용 패턴, GDP 기여도 등은 기술 확산 이후에만 측정 가능
2. **학술 시험 스타일 한계**: MMLU, GPQA 등은 추론 난이도에 집중하며 실제 업무 역량과 괴리
3. **도메인 특화 한계**: SWE-Lancer 등은 소프트웨어 엔지니어링만 커버하여 경제 전반 미반영

## Solution Approach
> **Source**: [Section 2](../2510.04374.md#2-task-creation)

- **Top-down 직업 선정**: GDP 기여 5% 이상 부문 → 부문별 최고 보상 디지털 직업 5개
- **전문가 기반 태스크**: 평균 14년 경력, 엄격한 선발(10% 미만 선정률), 평균 5회 리뷰
- **블라인드 비교 평가**: 전문가가 요청+참조파일만 보고 2개 이상 산출물을 블라인드 순위 결정
- **승률 기반 메트릭**: 포화 없는 연속적 평가, 베이스라인을 점진적으로 강화 가능

## Core Numbers
> **Source**: [Abstract](../2510.04374.md#abstract), [Section 3](../2510.04374.md#3-experiments-and-results), [Table 3](../2510.04374.md#table-3-summary-statistics-for-gdpval-gold-subset-tasks)

| 항목 | 수치 | 의미 |
|------|------|------|
| 전체 태스크 수 | 1,320개 | 44개 직업 x 30개 |
| 골드 서브셋 | 220개 | 오픈소스 공개 |
| Claude Opus 4.1 승률 | **47.6%** | 최고 성능 (인간 대비) |
| GPT-5 승률 | **39.0%** | OpenAI 최고 성능 |
| 부문 수 | 9개 | GDP 5% 이상 기여 |
| 직업군 수 | 44개 | 연간 총 보상 $3T |
| 평균 태스크 시간 | 7시간 | 최대 수주 소요 |
| 평균 태스크 가치 | $398 | 골드 서브셋 기준 |
| 전문가 평균 경력 | 14년 | 엄격한 선발 기준 |

## What Makes It Different
> **Source**: [Section 1](../2510.04374.md#1-introduction)

| 특성 | GDPval | 학술 벤치마크 (MMLU 등) | 도메인 특화 (SWE-Lancer) |
|------|--------|-------------------------|--------------------------|
| 현실적 태스크 | Yes | No (시험 스타일) | Yes (SW만) |
| 대표적 범위 | 44개 직업 | N/A | 1개 직업 |
| 멀티모달 파일 | CAD, PPT, 비디오 등 | 텍스트 중심 | 코드 중심 |
| 주관성 포함 | 구조, 스타일, 미학 | 객관적 정답 | 테스트 통과 |
| 포화 없음 | 승률 기반 | 점수 포화 가능 | 가능 |
| 장기 호라이즌 | 평균 7시간 | 수 분 | 수 시간 |

## Navigation

- [Main Analysis](../2510.04374-analysis.md)
- [Contributions Detail](./contributions.md)
- [Key Sections Guide](./key-sections.md)
- [Methodology Detail](./methodology.md)
