---
parent: ../2408.04682-analysis.md
source: ../2408.04682.md
---

# TL;DR - Deep Analysis

## Summary

> **ToolSandbox**는 Tool-use LLM을 **Stateful**, **Conversational**, **Interactive** 환경에서 평가하는 벤치마크다. 44%의 도구가 world state(cellular, wifi, location)에 의존하는 **implicit dependency**를 갖고, **LLM User Simulator**로 on-policy 대화형 평가를 수행하며, **Milestone/Minefield** 기반 DAG 평가로 다양한 성공 경로를 허용한다. 34개 도구, 11개 도메인, 1,032개 시나리오에서 GPT-4o가 73.0%로 최고 성능을 보이며, **State Dependency**와 **Canonicalization**이 SOTA 모델에게도 핵심 도전 과제임을 밝힌다.

## Extended Summary

### What It Does
- **Stateful Tool Evaluation**: Cellular service 꺼진 상태에서 메시지 전송 요청 시, 먼저 service 켜기 필요
- **Interactive Assessment**: GPT-4o 기반 user simulator로 realistic한 multi-turn 대화 수행
- **Dynamic Evaluation**: 고정 trajectory 대신 Milestone DAG로 다양한 성공 경로 허용

### How It Works
1. **Execution Context**: World state 초기화 (Settings, Contacts, Messages DB)
2. **User Simulation**: Knowledge Boundary + Demonstration으로 hallucination 최소화
3. **Agent Interaction**: Message Bus를 통한 User-Agent-Environment 통신
4. **Evaluation**: Milestone 달성 여부 + Minefield 회피 여부로 점수 계산

### Why It Matters
- **현실 복잡성 반영**: 기존 벤치마크의 stateless 가정 탈피
- **On-policy 평가**: Off-policy trajectory 의존성 제거
- **새로운 난이도 카테고리**: State Dependency, Canonicalization, Insufficient Information

## Why This Matters

### For Researchers
- **State Dependency 연구**: Implicit dependency chain 처리 연구의 기반 데이터
- **User Simulation 개선**: Knowledge Boundary 프롬프팅 기법 참조
- **평가 방법론**: Milestone/Minefield DAG 기반 평가 설계 가이드

### For Practitioners
- **Parallel Tool Call 주의**: 대형 모델(GPT-4, Opus)이 parallel call 시 dependency 무시
- **Canonicalization 처리**: 상대적 시간/위치 표현 변환 파이프라인 필요
- **Insufficient Info 대응**: "할 수 없다" 응답 능력 평가 및 guardrail 구축

### For the Field
- **Interactive Evaluation 표준**: Stateful 환경에서의 평가 기준 제시
- **BFCL 보완**: Stateless → Stateful 확장
- **ToolBeHonest 연동**: Insufficient Information 카테고리 중첩

## Key Numbers

| Metric | Value | Significance |
|--------|-------|--------------|
| Best Overall | 73.0% (GPT-4o) | Stateful 환경에서도 최고 |
| Open vs Proprietary Gap | 20점+ | Hermes-2-Pro 31.4% vs GPT-4o 73.0% |
| User Simulator Error | ~8% | Knowledge Boundary로 12.4%→6.97% |
| Stateful Tools | 44% | 15/34 도구가 state 의존 |
| Test Scenarios | 1,032 | 6개 카테고리 × 다양한 복잡도 |

## State Dependency Paradox

가장 흥미로운 발견 중 하나:

| Model | State Dependency Score |
|-------|------------------------|
| GPT-3.5-Turbo | **82.6%** |
| GPT-4 | 77.9% |
| GPT-4o | 84.0% |
| Claude-3-Opus | 74.5% |
| Claude-3-Sonnet | **75.7%** |

**역설**: 대형 모델(GPT-4, Opus)이 중형 모델(GPT-3.5, Sonnet)보다 **낮은** State Dependency 점수
- **원인**: Parallel tool call 기능을 더 적극 사용하면서 dependency 무시
- **시사점**: 기능 확장이 항상 성능 향상으로 이어지지 않음

## Source References

- **Problem Statement**: [Section 1 Introduction](../2408.04682.md#1-introduction)
- **Design Philosophy**: [Section 2 Benchmark Design](../2408.04682.md#2-benchmark-design)
- **Main Results**: [Table 5](../2408.04682.md#table-5-comparing-the-average-similarity-score)
- **User Simulator**: [Section 2.2](../2408.04682.md#22-llm-user-simulator)
