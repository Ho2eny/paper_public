---
parent: "../2401.13919-analysis.md"
source: "../2401.13919.md"
type: detail
section: contributions
---

# Core Contributions Deep Analysis: WebVoyager

[Back to Main Analysis](../2401.13919-analysis.md) | [Source Paper](../2401.13919.md)

---

## Contribution 1: End-to-End Multimodal Web Agent

### 개요
[Section 3: WebVoyager](../2401.13919.md#3-webvoyager)

WebVoyager는 GPT-4V(ision)를 backbone으로 하여 실제 웹사이트에서 end-to-end로 사용자 태스크를 수행하는 최초의 완전 자율 멀티모달 웹 에이전트이다.

### 기술적 상세

#### Observation Space
- **Primary Input**: 웹페이지 스크린샷 (1024x768 pixels)
- **Set-of-Mark Prompting**: GPT-4V-ACT 도구로 interactive elements에 숫자 라벨이 붙은 bounding box 오버레이
- **Auxiliary Text**: element type, 텍스트 콘텐츠, aria-label 속성
- **Context Clipping**: 최근 3개 observation만 유지, 전체 thought-action history 보존

```
c_t = (o_1, a_1, ..., o_{t-1}, a_{t-1}, o_t, I)
a_t = M(c_t)
o_{t+1} = E(o_t, a_t)
```

#### Action Space (7 actions)
| Action | Format | 설명 |
|--------|--------|------|
| Click | `Click [Numerical_Label]` | 요소 클릭 |
| Type | `Type [Numerical_Label]; [Content]` | 텍스트 입력 + Enter |
| Scroll | `Scroll [Label or WINDOW]; [up/down]` | 스크롤 |
| Wait | `Wait` | 페이지 로딩 대기 |
| GoBack | `GoBack` | 이전 페이지 |
| Google | `Google` | 검색엔진으로 이동 |
| Answer | `ANSWER; [Content]` | 태스크 완료 응답 |

### 차별점
- **SeeAct 대비**: 추가 cross-encoder 모델 불필요, 순수 LMM만으로 동작
- **WebArena 대비**: 실제 live 웹사이트 사용 (hosted simulator 아님)
- **기존 text-only 에이전트 대비**: HTML/accessibility tree 파싱 부담 없음

### 한계
- Drag 등 복잡한 interaction 미지원
- 비디오 등 특수 파일 포맷 미지원
- Login/CAPTCHA 필요 사이트 접근 불가

---

## Contribution 2: Real-World Web Task Benchmark

### 개요
[Section 4: Benchmark for WebVoyager](../2401.13919.md#4-benchmark-for-webvoyager)

15개 실제 웹사이트에서 643개의 다양한 태스크를 수집한 벤치마크 제공.

### 웹사이트 선정 기준
일상생활의 다양한 측면을 커버하도록 선정:

| 카테고리 | 웹사이트 |
|----------|----------|
| E-commerce | Amazon |
| Travel | Booking, Google Flights, Google Map |
| Education | Coursera, ArXiv, Cambridge Dictionary |
| Entertainment | Allrecipes, ESPN, BBC News |
| Tech | Apple, GitHub, Huggingface |
| Search/Compute | Google Search, Wolfram Alpha |

### 데이터 수집 방법 (Self-Instruct)
[Figure 3](../2401.13919.md#figure-3)

1. **Seed Tasks**: Mind2Web에서 일부 태스크 수동 샘플링 및 재작성
2. **GPT-4 Generation**: 시드 태스크를 in-context example로 사용하여 ~100개 신규 태스크 생성
3. **Human Verification**: 각 태스크를 수동 검증하여 품질 보장
4. **Iterative Expansion**: 검증된 태스크를 시드로 추가하여 반복

### 데이터 품질 검증
- **유사도 분석**: all-mpnet-base-v2로 206,403 쌍 분석
- 유사도 > 0.8: 49쌍 (수동 확인 후 acceptable)
- 유사도 0.7-0.8: 140쌍
- 유사도 < 0.6: 99.68%

### 정답 Annotation
| 유형 | 비율 | 설명 |
|------|------|------|
| Golden | 22.3% | 정확한 정답 존재, 단기간 안정적 |
| Possible | 77.7% | Open-ended, 다중 정답, 실시간 정보 |

---

## Contribution 3: GPT-4V-based Automatic Evaluation Protocol

### 개요
[Section 5.1: Evaluation Methods](../2401.13919.md#51-evaluation-methods)

Open-ended 웹 태스크의 스케일러블한 평가를 위한 GPT-4V 자동 평가 프로토콜 제안.

### 평가 방식
1. 태스크 instruction 제공
2. 에이전트의 최종 응답 제공
3. Navigation trajectory의 마지막 k개 스크린샷 제공
4. GPT-4V가 성공/실패 이진 판단

### 인간 평가자와의 일치도
[Table 2](../2401.13919.md#table-2)

| k (스크린샷 수) | Success Rate | Agreement | Kappa |
|-----------------|--------------|-----------|-------|
| k=1 | 47.7% | 75.3% | 0.51 |
| k=2 | 55.3% | 79.7% | 0.59 |
| k=3 | 54.3% | 81.3% | 0.62 |
| Full | 58.3% | **85.3%** | **0.70** |

- 인간 평가자 간 Fleiss' Kappa: 0.7 (substantial agreement)
- GPT-4V Full trajectory Kappa 0.70 = 인간 수준

### 다른 평가 모델 비교
[Table 3](../2401.13919.md#table-3)

- **Claude-3-Opus**: Kappa 0.6 (GPT-4V보다 낮음)
- **GPT-4o**: Kappa 0.72 (GPT-4V와 유사, 약간 lenient)
- 각 모델은 자신의 결과에 bias 경향

### 의의
- 웹 에이전트 평가의 **scalability 문제** 해결
- Open-ended 태스크에 대한 **자동화된 평가** 가능
- 향후 웹 에이전트 연구의 **표준 평가 프로토콜** 제시

---

## Contribution 4: Comprehensive Error Analysis

### 개요
[Section 5.4: Error Analysis](../2401.13919.md#54-error-analysis)

300개 태스크 샘플링을 통한 실패 원인 심층 분석.

### 에러 유형 분포
[Table 4](../2401.13919.md#table-4)

| 에러 유형 | 비율 | 주요 원인 |
|-----------|------|-----------|
| Navigation Stuck | 44.4% | 부정확한 검색어, 잘못된 스크롤 영역, 반복적 실수 |
| Visual Grounding Issue | 24.8% | 희귀 패턴 인식 실패, 근접 요소 혼동, 캘린더 숫자/라벨 혼동 |
| Hallucination | 21.8% | 부분 정답으로 만족, 잘못된 text box에 입력 |
| Prompt Misalignment | 9.0% | 파싱 불가 출력, ANSWER로 조기 종료 |

### 향후 연구 방향 시사점
1. **Navigation Stuck 해결**: 더 나은 exploration/backtracking 전략 필요
2. **Visual Grounding 개선**: 고해상도 입력, 텍스트 보조 정보 활용
3. **Hallucination 방지**: 태스크 요구사항 체크리스트, self-verification
4. **Prompt Engineering**: 더 robust한 action 포맷 설계

---

## Summary: Contribution Novelty & Impact

| Contribution | Novelty | Impact |
|--------------|---------|--------|
| Multimodal Web Agent | Set-of-Mark의 웹 도메인 적용 | LMM 기반 웹 에이전트 연구 촉진 |
| Real-World Benchmark | 실제 live 웹사이트 태스크 수집 | 현실적인 웹 에이전트 평가 가능 |
| Auto Evaluation | GPT-4V 기반 trajectory 평가 | 스케일러블한 연구 환경 제공 |
| Error Analysis | 체계적 실패 유형 분류 | 향후 연구 방향 명확화 |

---

[Back to Main Analysis](../2401.13919-analysis.md)
