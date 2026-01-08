---
parent: ../2307.13854-analysis.md
source: ../2307.13854.md
type: detail
section: methodology
---

# Detailed Methodology Analysis

> **Source**: [Sections 2-4](../2307.13854.md#2-webarena-websites-as-an-environment-for-autonomous-agents)

## 1. Environment Formalization

### Mathematical Framework
> **Source**: [Section 2.1](../2307.13854.md#21-controlling-agents-through-high-level-natural-language)

환경은 튜플로 정의됨:
$$\mathcal{E} = \langle \mathcal{S}, \mathcal{A}, \mathcal{O}, \mathcal{T} \rangle$$

| 기호 | 의미 | 설명 |
|------|------|------|
| $\mathcal{S}$ | State Space | 웹사이트의 모든 가능한 상태 |
| $\mathcal{A}$ | Action Space | 12가지 액션 타입 (Section 2.4) |
| $\mathcal{O}$ | Observation Space | URL, 탭 정보, 페이지 콘텐츠 |
| $\mathcal{T}$ | Transition Function | $\mathcal{T}: \mathcal{S} \times \mathcal{A} \rightarrow \mathcal{S}$ (결정론적) |

### Agent Decision Process

주어진 자연어 의도 $i$에서, 에이전트는 다음을 기반으로 액션 $a_t$를 결정:
- 현재 관찰 $o_t \in \mathcal{O}$
- 액션 이력 $a_1^{t-1}$
- 관찰 이력 $o_1^{t-1}$

### Reward Function

$$r(a_1^T, s_1^T)$$

- $a_1^T$: 시작부터 종료 시점 $T$까지의 액션 시퀀스
- $s_1^T$: 모든 중간 상태
- 상태 전이가 의도의 기대와 일치하는지 평가

---

## 2. Website Implementation

### Domain Selection Process
> **Source**: [Section 2.2](../2307.13854.md#22-website-selection)

**방법론**: 저자들의 실제 브라우저 히스토리 ~200개 분석 후 카테고리 분류

```
[브라우저 히스토리 분석]
     ↓
[카테고리 추상화]
     ↓
[상위 4개 도메인 선정]
     ↓
[오픈소스 프레임워크로 구현]
```

### Implementation Stack

| 도메인 | 오픈소스 기반 | 데이터 규모 |
|--------|---------------|-------------|
| E-commerce (OneStopShop) | Adobe Magento | ~90,000 products, 300+ categories |
| Social Forum | Postmill (Reddit) | 95 subreddits, 127,390 posts, 661,781 users |
| Collaborative Dev | GitLab | 300 repos, 1000+ accounts |
| CMS | Adobe Magento Admin | 샘플 데이터 |

### Utility Tools

| 도구 | 용도 | 구현 |
|------|------|------|
| Map | POI 탐색, 네비게이션 | OpenStreetMap (북동부 미국) |
| Calculator | 계산 | 자체 구현 |
| Scratchpad | 노트 | 자체 구현 |
| Wikipedia | 지식 검색 | Kiwix (2023년 5월 기준) |

### Docker-based Delivery
> **Source**: [Appendix A.2](../2307.13854.md#a2-environment-delivery-and-reset)

```
[Docker Image]
├── Website Code
├── Database
├── Pre-populated Data
└── Software Dependencies

[Reset Mechanism]
컨테이너 중지 → 삭제 → 원본 이미지에서 재시작
(수 초 ~ 1분)
```

---

## 3. Observation Space Design
> **Source**: [Section 2.3](../2307.13854.md#23-observation-space)

### Multi-Tab Support

**혁신점**: 최초의 멀티탭 웹 에이전트 환경

**이점**:
- 도구 사용 지원
- 탭 간 정보 비교/참조
- 인간의 실제 브라우징 패턴 모방

### Observation Modes

| 모드 | 표현 | 장점 | 단점 |
|------|------|------|------|
| Raw HTML | DOM Tree | 완전한 정보 | 장황함, 노이즈 많음 |
| Screenshot | RGB Array | 시각적 정보 | LLM 처리 어려움 |
| Accessibility Tree | 구조화된 요소 | 간결, 관련 정보 집중 | 일부 정보 손실 |

### Accessibility Tree 구조

```
[1582] button 'Add to Cart'
  ├── role: button
  ├── text: "Add to Cart"
  └── properties: {focusable: true, ...}
```

### Viewport Limitation Option

컨텍스트 길이 제한/이미지 크기 요구사항에 대응 가능

---

## 4. Action Space Design
> **Source**: [Section 2.4](../2307.13854.md#24-action-space)

### Action Categories

**Element Operations (6개)**:
| 액션 | 설명 | 예시 |
|------|------|------|
| `noop` | 아무것도 안 함 | - |
| `click(elem)` | 요소 클릭 | `click [1582]` |
| `hover(elem)` | 요소 호버 | `hover [123]` |
| `type(elem, text)` | 텍스트 입력 | `type [456] [hello]` |
| `press(key_comb)` | 키 조합 | `press [Ctrl+Enter]` |
| `scroll(dir)` | 스크롤 | `scroll [down]` |

**Tab Management (3개)**:
| 액션 | 설명 |
|------|------|
| `tab_focus(index)` | i번째 탭으로 전환 |
| `new_tab` | 새 탭 열기 |
| `tab_close` | 현재 탭 닫기 |

**URL Navigation (3개)**:
| 액션 | 설명 |
|------|------|
| `goto(URL)` | 특정 URL로 이동 |
| `go_back` | 이전 페이지 |
| `go_forward` | 다음 페이지 |

### Element Reference Methods

두 가지 방식 지원:
1. **좌표 기반**: $(x, y)$ 화면 좌표
2. **ID 기반**: DOM/Accessibility Tree 순회 시 생성된 고유 ID

**ID 기반의 장점**:
- $n$-way classification 문제로 변환
- 모호성 제거
- 다양한 입력 모달리티 에이전트 지원

---

## 5. Evaluation Framework
> **Source**: [Section 3.2](../2307.13854.md#32-evaluation-annotation)

### Information Seeking Evaluation: $r_{info}(\hat{a}, a^*)$

```python
# Pseudo-code
def exact_match(prediction, reference):
    return prediction == reference

def must_include(prediction, reference):
    return reference in prediction

def fuzzy_match(prediction, reference):
    # GPT-4를 사용한 의미적 동등성 평가
    prompt = f"Is '{prediction}' semantically equivalent to '{reference}'?"
    return gpt4_judge(prompt)
```

**fuzzy_match 정확도** (Appendix A.8):
- 날짜 형식: 100% (900 예시)
- 시간 형식: 100% (900 예시)

### Programmatic Evaluation: $r_{prog}(s)$

**2단계 프로세스**:

```
[Step 1: Locator]
├── Database Query
├── Website API Call
└── JavaScript Element Selection

[Step 2: Validator]
├── exact_match(located_content, expected)
└── must_include(located_content, expected)
```

**예시** (Post 검증):
```javascript
// 1. Locator
url = locate_latest_post_url(s)
body = document.querySelector('.submission__inner').outerText

// 2. Validator
must_include(url, '/f/nyc')
must_include(body, 'a car in NYC')
```

---

## 6. Baseline Agent Implementation
> **Source**: [Section 4](../2307.13854.md#4-baseline-web-agents)

### Prompting Strategies

| 전략 | 설명 |
|------|------|
| Direct | 관찰 → 바로 액션 예측 |
| CoT | 관찰 → 추론 단계 → 액션 예측 |

### Configuration
> **Source**: [Appendix A.6](../2307.13854.md#a6-experiment-configurations)

| 파라미터 | 값 |
|----------|-----|
| Temperature | 1.0 (탐색 장려) |
| top_p | 0.9 |
| Max steps | 30 |
| 동일 액션 반복 제한 | 3회 |
| 연속 무효 액션 제한 | 3회 |

### System Prompt 핵심 요소

1. 에이전트 역할 정의
2. 관찰 정보 설명
3. 액션 카테고리 및 형식
4. Homepage URL 및 계정 정보 위치
5. 성공 규칙 5가지

---

## 7. Human Performance Baseline
> **Source**: [Section 3.2](../2307.13854.md#32-evaluation-annotation)

### Methodology

- **참가자**: CS 대학원생 5명
- **태스크**: 170개 템플릿에서 1개씩 샘플
- **측정**: 성공률, 평균 시간, 궤적 기록

### Results

| 메트릭 | 값 |
|--------|-----|
| 평균 시간 | 110초 |
| 정보 탐색 성공률 | 74.68% |
| 기타 태스크 성공률 | 81.32% |
| 전체 성공률 | **78.24%** |

### Failure Analysis

- **50%**: 의도 오해, 불완전 답변, 불완전 실행
- **50%**: 심각한 실패 (완전히 잘못된 방향)

---

## Navigation

- [Main Analysis](../2307.13854-analysis.md)
- [TL;DR](./tldr.md)
- [Contributions Detail](./contributions.md)
- [Key Sections Guide](./key-sections.md)
