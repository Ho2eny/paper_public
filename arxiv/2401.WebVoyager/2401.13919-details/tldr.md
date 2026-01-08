---
parent: ../2401.13919-analysis.md
source: ../2401.13919.md
created: 2026-01-09
---

# TL;DR - Deep Analysis

## Extended Summary

> GPT-4V 기반 멀티모달 웹 에이전트로, Set-of-Mark 프롬프팅을 통해 스크린샷에서 interactive element를 식별하고, 7가지 행동(Click, Type, Scroll 등)을 수행하여 실제 웹사이트에서 end-to-end 태스크를 자율적으로 완수한다.
>
> **문제**: 기존 웹 에이전트는 HTML/accessibility tree에 의존하여 verbose하고 복잡한 입력 처리 필요 → **해결책**: 렌더링된 스크린샷 + Set-of-Mark bounding box로 시각적 이해 기반 상호작용 → **결과**: 15개 실제 웹사이트에서 59.1% Task Success Rate, GPT-4V 자동 평가로 인간 수준 일치 (Kappa 0.70) → **의의**: LMM 기반 웹 에이전트의 실제 적용 가능성 최초 체계적 입증

---

## Problem Context

### What Problem?
실제 웹사이트에서 사용자 지시를 자율적으로 완수하는 **end-to-end 웹 에이전트** 구축. 기존 접근법들은 HTML이나 accessibility tree 같은 텍스트 기반 표현에 의존하여 복잡하고 verbose한 입력을 처리해야 했으며, 대부분 시뮬레이터 환경이나 정적 스냅샷에서만 평가되어 실제 환경 적용에 한계가 있었다.

### Why Important?
- 웹 자동화는 일상생활(쇼핑, 예약, 정보 검색)부터 업무 생산성까지 광범위한 응용
- 시각 장애인 접근성 향상, 음성 제어 인터페이스 등 실질적 사회적 가치
- GPT-4V 등 LMM의 발전으로 시각 기반 상호작용이 가능해진 시점

### Prior Limitations
1. **Text-based 접근**: HTML/DOM은 verbose하고 구조화되어 있지 않음, accessibility tree도 불완전
2. **Simulated 환경**: MiniWoB++ 같은 단순화된 환경에서만 평가, 실제 웹의 복잡성 미반영
3. **정적 스냅샷**: Mind2Web 등은 사전 수집된 스냅샷으로만 평가, live interaction 불가
4. **Grounding 문제**: 어떤 요소를 클릭할지 정확히 지정하기 어려움

**Source**: [Introduction](../2401.13919.md#1-introduction)

---

## Solution Approach

### Key Idea
렌더링된 웹페이지 스크린샷이 UX 원칙에 따라 설계되어 있으므로, **시각적 분석이 HTML 표현보다 더 효과적**일 수 있다. Set-of-Mark Prompting을 웹 도메인에 적용하여 interactive element들에 숫자 라벨이 붙은 bounding box를 오버레이하고, GPT-4V가 이 라벨로 요소를 지칭하여 grounding 문제를 해결한다.

### How It Works
```
1. Observation: 웹페이지 스크린샷 + Set-of-Mark bounding boxes
   - GPT-4V-ACT JavaScript 도구로 interactive elements 자동 추출
   - 각 요소에 숫자 라벨 오버레이 (검정 테두리)
   - Auxiliary text: element type, 텍스트 내용, aria-label

2. Action Prediction: ReAct 스타일 thought-action 생성
   - Thought: 현재 상황 분석 및 다음 행동 계획
   - Action: Click[5], Type[2]; "query", Scroll[WINDOW]; down 등

3. Execution: Selenium을 통해 실제 브라우저에서 행동 실행

4. Loop: 최대 15 steps 또는 ANSWER action까지 반복
```

### What's New
- **Object detection 불필요**: Rule-based JavaScript로 interactive element 추출 → 효율적이고 일반적
- **단일 LMM으로 end-to-end**: SeeAct와 달리 추가 cross-encoder 모델 불필요
- **실제 live 웹사이트에서 동작**: 시뮬레이터가 아닌 Selenium 기반 실제 브라우저

**Source**: [Method](../2401.13919.md#3-webvoyager)

---

## Key Results

### Main Achievement
- **59.1% Task Success Rate**: 자체 벤치마크에서 GPT-4 (All Tools) 30.8%, Text-only 40.1% 크게 상회
- **GPT-4V 자동 평가**: 인간 평가자와 85.3% 일치율, Kappa 0.70 달성
- **다양한 웹사이트에서 검증**: Amazon, Booking, GitHub 등 15개 실제 웹사이트

### Quantitative Results
- **Booking.com**: Text-only 2.3% → WebVoyager 43.2% (캘린더 UI 이해 필수)
- **Google Flights**: Text-only 7.1% → WebVoyager 59.5%
- **vs GPT-4 All Tools**: 30.8% → 59.1% (28.3%p 향상)
- **GAIA Level 1/2**: GPT-4 (All Tools) 대비 우수
- **SeeAct Online Test**: 30% vs SeeAct 26%

### Qualitative Findings
- 시각적 복잡성이 높은 사이트(Booking, Flights)에서 멀티모달 접근의 가치 극대화
- 텍스트 중심 사이트(GitHub, Allrecipes)에서는 Text-only와 유사한 성능
- Navigation Stuck(44.4%)이 가장 큰 실패 원인 → 탐색 전략 개선 필요

**Source**: [Experiments](../2401.13919.md#5-experiments)

---

## Why It Matters

### Academic Significance
- **LMM 기반 웹 에이전트의 가능성 입증**: 최초로 실제 웹사이트에서 체계적 평가
- **자동 평가 프로토콜 제시**: Open-ended 태스크의 scalable 평가 방법론
- **체계적 에러 분석**: 300개 실패 사례 분류로 향후 연구 방향 명확화

### Practical Value
- **즉시 적용 가능**: GPT-4V API만으로 웹 자동화 구현 가능
- **비전문가 사용 가능**: 자연어 지시만으로 복잡한 웹 태스크 수행
- **접근성 향상**: 시각 장애인 등을 위한 웹 인터페이스 가능성

### Future Impact
- Set-of-Mark 기반 요소 식별 방식이 후속 연구의 표준으로 채택 (SeeClick, UGround 등)
- GPT-4V 자동 평가가 웹 에이전트 연구의 표준 평가 방법으로 확산
- Error analysis에서 발견된 패턴들이 후속 연구의 개선 방향 제시

---

## Source References

| Topic | Section | Link |
|-------|---------|------|
| Problem | Introduction | [Section 1](../2401.13919.md#1-introduction) |
| Method | WebVoyager | [Section 3](../2401.13919.md#3-webvoyager) |
| Observation Space | Set-of-Mark | [Section 3.3](../2401.13919.md#33-observation-space) |
| Action Space | 7 Actions | [Section 3.4](../2401.13919.md#34-action-space) |
| Benchmark | Task Collection | [Section 4](../2401.13919.md#4-benchmark-for-webvoyager) |
| Evaluation | Auto Eval Protocol | [Section 5.1](../2401.13919.md#51-evaluation-methods) |
| Results | Main Experiments | [Section 5.2](../2401.13919.md#52-result) |
| Error Analysis | Failure Patterns | [Section 5.4](../2401.13919.md#54-error-analysis) |

---

## Navigation

← [Back to Main Analysis](../2401.13919-analysis.md)
