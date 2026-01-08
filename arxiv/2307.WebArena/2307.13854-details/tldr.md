---
parent: ../2307.13854-analysis.md
source: ../2307.13854.md
type: detail
section: tldr
---

# Extended TL;DR: WebArena

> **Source**: [Abstract & Introduction](../2307.13854.md#abstract)

## One-Liner
WebArena는 자율 에이전트 개발을 위한 최초의 현실적이고 재현 가능한 독립형 웹 환경으로, 4개 도메인의 실제 웹사이트와 812개의 장기 호라이즌 태스크를 제공한다.

## Executive Summary (3-5 문장)

WebArena는 실제 웹사이트의 기능과 데이터를 충실히 재현한 벤치마크 환경이다. [Section 2](../2307.13854.md#2-webarena-websites-as-an-environment-for-autonomous-agents)에서 설명하듯이, e-commerce, 소셜 포럼, 협업 개발, 콘텐츠 관리 시스템의 4개 도메인을 Docker 기반으로 제공한다. 기존 벤치마크들이 단순화된 환경이나 정적 상태에 의존한 반면, WebArena는 **동적 상호작용**, **현실적 환경**, **다양한 인간 태스크**, **기능적 정확성 평가**를 모두 충족하는 최초의 환경이다. GPT-4 기반 최고 에이전트가 14.41%의 성공률을 보인 반면 인간은 78.24%를 달성하여, 현재 LLM 기반 에이전트의 한계를 명확히 보여준다.

## Key Problem Statement
> **Source**: [Section 1 - Introduction](../2307.13854.md#1-introduction)

기존 에이전트 평가 환경의 세 가지 핵심 문제:

1. **단순화 문제**: 실제 환경의 기능을 제한적으로만 구현 (MiniWoB++, WebShop)
2. **정적 리소스 문제**: 캐시된 상태만 접근 가능하여 탐색 다양성 제한 (Mind2Web)
3. **평가 방법 문제**: 텍스트 표면 형태 비교에 의존, 기능적 정확성 무시

## Solution Approach
> **Source**: [Section 2](../2307.13854.md#2-webarena-websites-as-an-environment-for-autonomous-agents)

- **독립형 환경**: Docker 컨테이너 기반, 외부 라이브 웹사이트 의존성 없음
- **현실적 데이터**: 실제 웹사이트에서 추출한 콘텐츠로 진정성 보장
- **도구 통합**: 지도, 계산기, 스크래치패드 등 인간의 문제 해결 방식 모방
- **프로그래매틱 평가**: 데이터베이스 쿼리, API 호출, JavaScript를 통한 기능적 정확성 검증

## Core Numbers
> **Source**: [Abstract](../2307.13854.md#abstract), [Section 3](../2307.13854.md#3-benchmark-suite-of-web-based-tasks), [Section 5](../2307.13854.md#5-results)

| 항목 | 수치 | 의미 |
|------|------|------|
| 태스크 수 | 812개 | 241개 템플릿에서 인스턴스화 |
| 인간 성공률 | **78.24%** | 170개 태스크 샘플, CS 대학원생 5명 |
| GPT-4 성공률 | **14.41%** | CoT 프롬프팅, UA hint 제거 시 |
| 도메인 수 | 4개 | E-commerce, Forum, GitLab, CMS |
| 상품 수 | ~90,000개 | OneStopShop 쇼핑 사이트 |

## What Makes It Different
> **Source**: [Section 6 - Related Work](../2307.13854.md#6-related-work), [Table 4](../2307.13854.md#table-4)

| 특성 | WebArena | 기존 벤치마크 |
|------|----------|---------------|
| 동적 상호작용 | Yes | Mind2Web: No, WebShop: Yes |
| 현실적 환경 | Yes | MiniWoB++: No, WebShop: No |
| 다양한 태스크 | Yes | MiniWoB++: No, AndroidEnv: No |
| 기능적 정확성 | Yes | Mind2Web: No, VirtualHome: No |

## Navigation

- [Main Analysis](../2307.13854-analysis.md)
- [Contributions Detail](./contributions.md)
- [Key Sections Guide](./key-sections.md)
- [Methodology Detail](./methodology.md)
