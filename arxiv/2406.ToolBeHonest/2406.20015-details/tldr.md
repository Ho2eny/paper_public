---
parent: ../2406.20015-analysis.md
source: ../2406.20015.md
---

# TL;DR - Deep Analysis

## Summary

> **ToolBeHonest (ToolBH)**는 Tool-augmented LLM이 **해결 불가능한 태스크**에서 hallucination을 얼마나 피하는지 평가하는 multi-level 진단 벤치마크다. **3단계 깊이**(Solvability Detection → Solution Planning → Missing-tool Analysis)와 **3가지 시나리오**(MNT/PT/LFT)로 구성되며, 700개 수동 어노테이션 샘플로 평가한 결과 최고 성능 모델도 **45.3점/100점**에 그쳐 심각한 한계를 드러낸다.

## Extended Summary

### What It Does
- **Unsolvable Task 평가**: 주어진 toolset으로 해결 불가능한 상황에서 LLM의 행동 진단
- **3단계 진단 (Depth)**:
  - L1: 태스크가 solvable한지 unsolvable한지 판단
  - L2: 어떤 단계에서 unsolvable해지는지 식별
  - L3: 누락된 도구의 기능을 설명
- **3가지 시나리오 (Breadth)**:
  - MNT: 필수 도구 누락
  - PT: 환경 내 잠재 도구(사용 불가) 존재
  - LFT: 도구 기능 제한

### How It Works
1. **데이터 수집**: Seed sample 생성 → GPT-4o/Gemini로 합성 → 필터링 → 수동 검증
2. **평가 방식**:
   - L1: Exact Match (solvable/unsolvable)
   - L2: Progress Rate (순차 정확도)
   - L3: Progress Rate + Matching Score (설명 유사도)
3. **UnsolvableQuery 도구**: 특수 도구로 해결 불가능한 sub-goal 표시

### Why It Matters
- **Hallucination의 실체**: LLM이 "할 수 없다"고 말하지 못하는 심각한 문제 정량화
- **Solvability가 핵심 병목**: 모든 모델에서 solvability hallucination이 가장 빈번
- **Open vs Proprietary Gap**: Unsolvable에서 격차가 **2.5배** (39.4% 수준)

## Why This Matters

### For Researchers
- **새로운 연구 방향**: Unsolvable task handling은 미개척 영역
- **에러 분류 프레임워크**: 5가지 에러 유형으로 개선 포인트 명확화
- **모델 크기 vs 데이터 품질**: 파라미터 수보다 학습 데이터와 응답 전략이 중요

### For Practitioners
- **Production 배포 주의**: 최고 모델도 45% → 실제 서비스에서 hallucination 빈번
- **Unsolvable 처리 필수**: "할 수 없다"고 적절히 응답하는 로직 필요
- **도구 선택 검증**: 제공되지 않은 도구 사용 시도 모니터링 필요

### For the Field
- **Benchmark Gap 해결**: 기존 벤치마크의 "solvable 가정" 한계 극복
- **Hallucination 세분화**: 일반 hallucination vs tool-use hallucination 구분
- **다단계 진단**: 단순 정오답 넘어 원인 분석 가능

## Key Numbers

| Metric | Value | Significance |
|--------|-------|--------------|
| Best Overall | 45.3 (Gemini-1.5-Pro) | 절반도 못 맞춤 |
| GPT-4o Score | 37.0 | SOTA도 저조 |
| Llama-3-70B | 14.6 | Open-weight 최고지만 proprietary의 32% |
| L3-MS in LFT | 0.0~8.5 | Missing-tool 분석 거의 불가 |
| Test Samples | 1,050 (700 base × 3 levels) | 다층 평가 |

## Source References

- **Problem Statement**: [Section 1 Introduction](../2406.20015.md#1-introduction)
- **Multi-level Framework**: [Section 3.2 In-depth: Multi-level Diagnostic Task](../2406.20015.md#32-in-depth-multi-level-diagnostic-task)
- **Main Results**: [Table 2](../2406.20015.md#table-2-the-toolbh-leaderboard-with-14-llms)
- **Error Analysis**: [Section 5.2 Error Analysis](../2406.20015.md#52-error-analysis)
