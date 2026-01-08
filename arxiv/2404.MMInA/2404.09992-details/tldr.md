---
parent: "../2404.09992-analysis.md"
source: "../2404.09992.md"
type: detail
created: 2026-01-09
---

# Extended TL;DR: MMInA Benchmark

[Back to Main Analysis](../2404.09992-analysis.md)

## One-Sentence Summary

MMInA는 **실제 웹사이트 14개**에서 **멀티홉(multi-hop) 멀티모달 태스크 1,050개**를 수행하는 인터넷 에이전트의 장거리 추론 능력을 평가하는 벤치마크이다.

---

## Core Problem Statement

> "Real-world web tasks are inherently compositional, requiring multihop actions across multiple websites." - [Section 1](../2404.09992.md#1-introduction)

기존 웹 에이전트 벤치마크들의 한계:
1. **단일 웹사이트 중심**: WebArena, Mind2Web 등은 대부분 1-2 hop 태스크만 평가
2. **정적 환경**: 시뮬레이션된 웹사이트로 현실 반영 부족
3. **텍스트 의존**: 시각 정보가 필수적인 실제 웹 환경 미반영

---

## What MMInA Offers

### 1. Evolving Real-World Websites
- **14개 실제 웹사이트**: Wikipedia, Amazon, YouTube, Trip.com, Momondo 등
- **동적 콘텐츠**: 시간에 따라 업데이트되는 실제 웹 환경
- Source: [Section 3.1](../2404.09992.md#31-environment), [Table A2](../2404.09992.md#b21-website-links)

### 2. Multihop Task Composition
- **평균 2.85 hops**, 최대 10 hops
- **평균 12.9 액션** per task
- 쇼핑, 여행, 정보 검색 등 다양한 도메인
- Source: [Figure 2](../2404.09992.md#figure-2), [Section 3.4](../2404.09992.md#34-multihop-cross-website-browsing)

### 3. Multimodal Requirement
- 모든 태스크가 이미지+텍스트 처리 필요
- 예: "파란색 면 셔츠 구매" - 시각적 색상 인식 필수
- Source: [Section 3.3](../2404.09992.md#33-multimodal-web-content)

---

## Key Findings

### Performance Gap
| Agent | Task Success Rate |
|-------|-------------------|
| Human | 96.25% |
| GPT-4V | 21.77% |
| DeepSeek-R1 | 23.07% (1-hop only) |

Source: [Table 2](../2404.09992.md#42-main-results)

### Critical Insight: Early Hop Failure

> "Agents are more likely to fail on the early hops when solving tasks with more hops" - [Abstract](../2404.09992.md#abstract)

- 6-hop 태스크에서 1st hop 성공률: GPT-4V 16.67%, Gemini 31.03%
- 2-hop 태스크에서 1st hop 성공률: GPT-4V 56.5%, Gemini 69.28%
- Source: [Table 3](../2404.09992.md#43-why-are-multihop-web-tasks-challenging)

---

## Solution: Memory Augmentation

세 가지 메모리 시스템 제안:
1. **Semantic Memory**: 일반 세계 지식 (LLM weights)
2. **Episodic Memory**: 단계별 액션 궤적 (context)
3. **Procedural Memory**: 완료된 태스크의 전략 (K=1-2 optimal)

Source: [Section 4.4](../2404.09992.md#44-memory-augmented-agents), [Figure 4](../2404.09992.md#figure-4)

---

## Why This Matters (2026 Perspective)

1. **Agentic AI 평가 기준**: 웹 에이전트의 실용성 측정 표준
2. **장거리 추론의 병목**: 멀티홉 태스크에서 조기 실패 패턴 식별
3. **메모리 메커니즘 중요성**: 과거 경험 활용이 성능 향상 핵심

---

## Quick Reference

- **GitHub**: [github.com/shulin16/MMInA](https://github.com/shulin16/MMInA)
- **Tasks**: 1,050개 human-written
- **Websites**: 14개 evolving real-world
- **Max Hops**: 10
- **Human vs Best AI Gap**: ~74%p

[Back to Main Analysis](../2404.09992-analysis.md) | [Contributions Detail](contributions.md)
