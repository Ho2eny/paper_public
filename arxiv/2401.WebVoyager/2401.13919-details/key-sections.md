---
parent: ../2401.13919-analysis.md
source: ../2401.13919.md
created: 2026-01-09
---

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

| Metric | Value |
|--------|-------|
| **Total Reading Time** | ~55 minutes (전체) |
| **Essential Only** | ~25 minutes (핵심만) |
| **Total Sections** | 6 sections + Appendix |
| **Must Read** | 3 sections |

---

## ⭐⭐⭐ Must Read

### Section 3: WebVoyager

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
논문의 핵심 기술 기여. 에이전트 아키텍처, observation/action space, Set-of-Mark 적용 방법의 모든 세부사항 포함.

**Key Points**:
1. Browsing Environment: Selenium 기반 실제 웹 환경
2. Interaction Formulation: context, observation, action의 수학적 정의
3. Observation Space: Set-of-Mark + auxiliary text 설계
4. Action Space: 7가지 action (Click, Type, Scroll, Wait, Back, Google, Answer)

**Reading Tips**:
- Figure 1의 전체 workflow를 먼저 이해
- Figure 2의 Set-of-Mark 스크린샷 예시 확인
- "검은색 단일 색상"이 다중 색상보다 효과적인 이유 파악

**Estimated Time**: ~15 minutes

**Link**: [Section 3](../2401.13919.md#3-webvoyager)

---

### Section 5.1: Evaluation Methods

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
GPT-4V 자동 평가 프로토콜의 상세 설계. 웹 에이전트 평가의 새로운 패러다임 제시.

**Key Points**:
1. Human evaluation 프로토콜 (3명 annotator)
2. GPT-4V evaluator 설계 (k screenshots)
3. Agreement/Kappa 분석 (Full trajectory: 85.3%, Kappa 0.70)
4. 다른 모델(Claude-3-Opus, GPT-4o) 비교

**Reading Tips**:
- Table 2의 k값에 따른 일치율 변화 확인
- Kappa 0.70 = "substantial agreement"의 의미 이해
- Figure 8의 auto evaluation 프롬프트 참조

**Estimated Time**: ~10 minutes

**Link**: [Section 5.1](../2401.13919.md#51-evaluation-methods)

---

### Section 5.4: Error Analysis

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
300개 실패 사례의 체계적 분류. 웹 에이전트 개발 시 해결해야 할 핵심 문제들.

**Key Points**:
1. Navigation Stuck (44.4%): 부정확한 검색어, 스크롤 방향 혼란, 반복적 실수
2. Visual Grounding Issue (24.8%): 희귀 패턴 인식 실패, 근접 요소 혼동
3. Hallucination (21.8%): 부분 정답으로 만족, 잘못된 필드 입력
4. Prompt Misalignment (9.0%): 파싱 불가 출력, 조기 종료

**Reading Tips**:
- Table 4의 에러 분포 숙지
- 각 에러 유형별 해결 방향 생각해보기
- Appendix F의 에러 케이스 예시 확인

**Estimated Time**: ~8 minutes

**Link**: [Section 5.4](../2401.13919.md#54-error-analysis)

---

## ⭐⭐ Important

### Section 4: Benchmark for WebVoyager

**Priority**: ⭐⭐ Important

**Why Read**:
벤치마크 연구자 또는 새로운 웹 에이전트 평가 시 필수 참고.

**Key Points**:
1. Website Selection: 15개 사이트 선정 기준 (일상생활 다양한 측면 커버)
2. Data Construction: Self-instruct 기반 태스크 생성 (643개)
3. Annotation Process: Golden (22.3%) vs Possible (77.7%) 정답 분류
4. 유사도 분석: 99.68% < 0.6 similarity → 높은 다양성

**When to Read**:
- 새로운 웹 에이전트 벤치마크 구축 시
- Self-instruct 방식의 데이터 수집 방법 참고 시

**Link**: [Section 4](../2401.13919.md#4-benchmark-for-webvoyager)

---

### Section 5.2: Result

**Priority**: ⭐⭐ Important

**Why Read**:
모든 실험 결과가 집약된 섹션. Table 1의 웹사이트별 성능 비교가 핵심.

**Key Points**:
1. Table 1: 15개 웹사이트별 상세 성능
2. GPT-4 (All Tools) vs Text-only vs WebVoyager 비교
3. GAIA Level 1/2 결과
4. SeeAct 온라인 테스트 비교

**When to Read**:
- 웹 에이전트 성능 기준선 파악 시
- 특정 웹사이트 도메인의 난이도 이해 시

**Link**: [Section 5.2](../2401.13919.md#52-result)

---

### Section 5.3: Discussions

**Priority**: ⭐⭐ Important

**Why Read**:
실험에서 얻은 insights 정리. 실무 적용 시 참고.

**Key Points**:
1. Direct interaction 필수: GPT-4 All Tools는 Bing 검색 의존 → 한계
2. Text + Vision 모두 필요: 각각 강점이 다른 웹사이트 존재
3. 복잡도 요인: interactive element 수 ↑, trajectory 길이 ↑ → 성공률 ↓

**When to Read**:
- 웹 에이전트 설계 시 trade-off 이해
- 어떤 상황에서 멀티모달이 유리한지 파악

**Link**: [Section 5.3](../2401.13919.md#53-discussions)

---

## ⭐ Reference

### Section 1: Introduction

**Priority**: ⭐ Reference Only

**Content Summary**:
연구 동기와 기존 접근법의 한계 설명. 웹 에이전트 분야 입문자에게 유용.

**When Useful**:
- 웹 에이전트 연구 배경 파악 시
- 발표 자료 준비 시

**Link**: [Section 1](../2401.13919.md#1-introduction)

---

### Section 2: Related Work

**Priority**: ⭐ Reference Only

**Content**:
기존 웹 에이전트 연구 정리. WebGPT, WebAgent, SeeAct 등.

**When Useful**:
- 서베이 목적
- 관련 연구 비교 분석 시

**Link**: [Section 2](../2401.13919.md#2-related-work)

---

### Appendix A: System Prompt

**Priority**: ⭐ Reference Only

**Content**:
WebVoyager의 실제 시스템 프롬프트

**When Useful**:
- 유사 에이전트 구현 시
- 프롬프트 엔지니어링 참고

**Link**: [Figure 7](../2401.13919.md#figure-7)

---

### Appendix B: Auto Evaluation Prompt

**Priority**: ⭐ Reference Only

**Content**:
GPT-4V evaluator 프롬프트

**When Useful**:
- 자동 평가 시스템 구축 시

**Link**: [Figure 8](../2401.13919.md#figure-8)

---

## Skip

### Section 6: Conclusion

**Priority**: Skip

**Why Skip**:
- 본문 내용의 요약
- 새로운 정보 없음
- 한계점은 Limitations에서 더 상세히 다룸

**Alternative**:
- Abstract로 대체 가능

**Link**: [Section 6](../2401.13919.md#6-conclusion) (참고용)

---

### Appendix D: Additional Trajectories

**Priority**: Skip (unless debugging)

**Why Skip**:
- 성공 trajectory 예시 모음
- 방법론 이해에 필수적이지 않음

**Alternative**:
- Figure 1의 예시만으로 충분

**Link**: [Figures 9-24](../2401.13919.md#d-additional-trajectories)

---

## Recommended Reading Order

최적의 읽기 순서:

```
1. [Abstract](../2401.13919.md#abstract) - 전체 개요
   ↓
2. [Figure 1](../2401.13919.md#figure-1) - 시스템 workflow
   ↓
3. [Section 3](../2401.13919.md#3-webvoyager) - 핵심 방법론
   ↓
4. [Section 5.2](../2401.13919.md#52-result) - 주요 결과
   ↓
5. [Section 5.1](../2401.13919.md#51-evaluation-methods) - 자동 평가
   ↓
6. [Section 5.4](../2401.13919.md#54-error-analysis) - 에러 분석
   ↓
7. [Section 4](../2401.13919.md#4-benchmark-for-webvoyager) - 벤치마크 (선택)
```

### Quick Read Path (~15 min)
1. Abstract 전체 읽기
2. Figure 1 → 시스템 workflow
3. Table 1 → 성능 결과
4. Table 4 → 에러 분포

### Deep Read Path (~45 min)
1. Abstract + Section 1
2. Section 3 (전체)
3. Section 5.2
4. Section 5.4
5. Section 4 (벤치마크)
6. Section 5.1, 5.3

---

## Section Dependency Map

```
Abstract
    │
    ├── Introduction (1)
    │       │
    │       ▼
    │   Related Work (2) [Skip 가능]
    │       │
    │       ▼
    │   WebVoyager (3) ◀── 핵심 방법론
    │       │
    │       ├── 3.3 Observation Space
    │       └── 3.4 Action Space
    │       │
    │       ▼
    │   Benchmark (4) ◀── 평가 대상
    │       │
    │       ▼
    │   Experiments (5)
    │       │
    │       ├── 5.1 Evaluation Methods ◀── 평가 방법
    │       ├── 5.2 Results ◀── 성능
    │       ├── 5.3 Discussions ◀── 분석
    │       └── 5.4 Error Analysis ◀── 실패 분석
    │       │
    │       ▼
    └── Conclusion (6) [Skip 가능]
```

---

## Source References

| Section | Priority | Time | Link |
|---------|----------|------|------|
| WebVoyager | ⭐⭐⭐ | ~15 min | [Section 3](../2401.13919.md#3-webvoyager) |
| Evaluation Methods | ⭐⭐⭐ | ~10 min | [Section 5.1](../2401.13919.md#51-evaluation-methods) |
| Error Analysis | ⭐⭐⭐ | ~8 min | [Section 5.4](../2401.13919.md#54-error-analysis) |
| Benchmark | ⭐⭐ | ~10 min | [Section 4](../2401.13919.md#4-benchmark-for-webvoyager) |
| Results | ⭐⭐ | ~10 min | [Section 5.2](../2401.13919.md#52-result) |
| Discussions | ⭐⭐ | ~5 min | [Section 5.3](../2401.13919.md#53-discussions) |
| Introduction | ⭐ | ~5 min | [Section 1](../2401.13919.md#1-introduction) |
| Conclusion | Skip | - | [Section 6](../2401.13919.md#6-conclusion) |

---

## Navigation

← [Back to Main Analysis](../2401.13919-analysis.md)
