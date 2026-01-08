---
parent: "../2103.16057-analysis.md"
source: "../2103.16057.md"
type: detail
created: 2026-01-09
---

# Detailed Methodology Analysis

[Back to Main Analysis](../2103.16057-analysis.md)

---

## Overall Framework

RUSS는 3개의 모듈로 구성된 파이프라인 아키텍처. [Section 3](../2103.16057.md#3-task-and-model)

```
┌─────────────────────────────────────────────────────────────────┐
│                    RUSS Framework                                │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Stage 1: Semantic Parsing                                   │ │
│  │                                                             │ │
│  │   Natural Language ──▶ Entity Extraction ──▶ BERT-LSTM     │ │
│  │   "Click reset"       "Click TYPE"         @retrieve(...)  │ │
│  │                                             → @click()      │ │
│  └──────────────────────────┬─────────────────────────────────┘ │
│                             │                                    │
│                             ▼                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Stage 2: Grounding                                         │ │
│  │                                                             │ │
│  │   ThingTalk + DOM ──▶ Filter ──▶ SentenceBERT ──▶ element  │ │
│  │   @retrieve(...)      (type/loc)    (similarity)    id=42  │ │
│  └──────────────────────────┬─────────────────────────────────┘ │
│                             │                                    │
│                             ▼                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Stage 3: Execution                                         │ │
│  │                                                             │ │
│  │   Grounded Action ──▶ Puppeteer/Voice API ──▶ Result       │ │
│  │   @click(id=42)       Chrome automation      Task done     │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Stage 1: Semantic Parsing

### 1.1 ThingTalk DSL 설계 [Section 3.1](../2103.16057.md#31-thingtalk)

**설계 원칙**:
1. 오픈 도메인 자연어에 견고
2. 신경망 파싱에 적합한 구조
3. 합성 데이터 생성에 용이

**Agent Actions (6종)**:

| Action | 문법 | 설명 |
|--------|------|------|
| `@goto` | `@goto(url)` | URL 이동 |
| `@enter` | `@enter(element_id, dict_key)` | 텍스트 입력 |
| `@click` | `@click(element_id)` | 클릭 |
| `@read` | `@read(element_id)` | 요소 읽기 |
| `@say` | `@say(message)` | 메시지 출력 |
| `@ask` | `@ask(dict_key)` | 사용자 질문 |

**Grounding Function**:
```
@retrieve(descr, type, loc, above, below, right_of, left_of) → element_id
```

**입력 특징**:
- `descr`: 텍스트 설명 (예: "reset button")
- `type`: HTML 요소 타입 (button, input, etc.)
- `loc`: 절대 위치 (top, bottom, left, right)
- `above/below/...`: 상대 위치 관계

**합성 가능한 문법**:
```
program  ::= [retrieve →]* action
retrieve ::= @retrieve(features)
action   ::= @click(element) | @enter(element, key) | ...
```

### 1.2 Semantic Parser 아키텍처 [Section 3.2](../2103.16057.md#32-semantic-parser-model)

**BERT-LSTM with Pointer-Generator** ([Figure 2](../2103.16057.md#figure-2)):

```
Input: "Click the button on the top of the amazon.com page"
    │
    ▼
┌─────────────────────────────────────────────────┐
│ Entity Extraction                               │
│   URL: amazon.com → [URL]                       │
│   LOC: top → [LOC]                              │
│   TYPE: button → [TYPE]                         │
│   Result: "Click the [TYPE] on the [LOC] of     │
│            the [URL] page"                      │
└─────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────┐
│ BERT Encoder                                    │
│   Pretrained BERT → contextualized embeddings   │
└─────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────┐
│ LSTM Decoder + Pointer-Generator                │
│   Token-by-token generation                     │
│   Copy mechanism for OOV words                  │
│   Output: @retrieve(type=[TYPE], loc=[LOC])     │
│           → @click(element=id)                  │
└─────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────┐
│ Post-processing                                 │
│   Substitute placeholders back                  │
│   Result: @retrieve(type="button", loc="top")   │
│           → @click(element=id)                  │
└─────────────────────────────────────────────────┘
```

**Entity Extraction 기여** [Table 3](../2103.16057.md#table-3):
- 적용 시: 88.2% (dev)
- 미적용 시: 77.6% (dev)
- **+10.6% 개선**

### 1.3 합성 데이터 생성 [Section 4.2](../2103.16057.md#42-synthetic-dataset)

**템플릿 기반 합성**:

```
Template: "At the {loc} of the page, @click the {type} that says {descr}"

Values:
  loc ∈ {top, bottom, left, right, center, ...}
  type ∈ {button, link, checkbox, ...}
  descr ← scraped from real DOMs

Example outputs:
  "At the top of the page, click the button that says Submit"
  "At the bottom of the page, click the link that says Learn more"
```

**규모**:
- 1.5M 훈련 샘플
- ~840 고유 템플릿 조합
- 어휘: 9,305 단어

---

## Stage 2: Grounding Model

### 2.1 DOM 표현 [Section 3.3](../2103.16057.md#33-grounding-model)

**웹페이지 표현** ([Figure 3](../2103.16057.md#figure-3)):

각 DOM 요소의 특징:
- `inner_text`: 요소 내부 텍스트
- `id`, `tag`, `class`: HTML 속성
- `hidden`: 가시성 여부
- `height`, `width`: 크기
- `top`, `left`, `bottom`, `right`: 좌표
- `children`: 자식 요소 목록

### 2.2 Grounding 파이프라인

```
Input: @retrieve(descr="reset", type="button", loc="top")
       + DOM with 689 elements (average)
    │
    ▼
┌─────────────────────────────────────────────────┐
│ Step 1: Type Filtering                          │
│   Filter by HTML tag matching "type" feature    │
│   button → <button>, <input type="button">, ... │
│   Candidates: 689 → ~50                         │
└─────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────┐
│ Step 2: Location Filtering                      │
│   Filter by absolute position (loc="top")       │
│   top → elements in upper 30% of viewport       │
│   Candidates: ~50 → ~20                         │
└─────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────┐
│ Step 3: Relative Position (if specified)        │
│   above/below/left_of/right_of another element  │
│   Check spatial relationship & proximity        │
│   Candidates: ~20 → ~10                         │
└─────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────┐
│ Step 4: Text Matching (SentenceBERT)            │
│   E(descr="reset") vs E(candidate.inner_text)   │
│   cosine_similarity → ranking                   │
│   Return: element with highest score            │
└─────────────────────────────────────────────────┘
    │
    ▼
Output: element_id = 42
```

### 2.3 성능 분석 [Table 5](../2103.16057.md#table-5)

| 추론 유형 | RUSS | PhraseNode | 격차 |
|----------|------|------------|------|
| Type | 67.8% | 61.5% | +6.3% |
| Input | 75.6% | 60.4% | +15.2% |
| Relational | 70.0% | 53.5% | +16.5% |
| Spatial | 36.7% | 30.3% | +6.4% |

**분석**:
- Input target (입력 필드)에서 가장 큰 개선
- Spatial은 여전히 낮음 (이미지 요소 인식 한계)

---

## Stage 3: Runtime Execution

### 3.1 웹 자동화 [Section 3.4](../2103.16057.md#34-the-run-time)

**Puppeteer 기반 실행**:
```javascript
// 개념적 코드
async function executeAction(action) {
  switch (action.type) {
    case '@goto':
      await page.goto(action.url);
      break;
    case '@click':
      await page.click(`[data-id="${action.element_id}"]`);
      break;
    case '@enter':
      const value = userDict[action.dict_key];
      await page.type(`[data-id="${action.element_id}"]`, value);
      break;
    case '@read':
      const text = await page.$eval(`[data-id="${action.element_id}"]`,
        el => el.innerText);
      await say(text);
      break;
  }
}
```

### 3.2 사용자 상호작용

**Google Voice API**:
- `@say(message)`: TTS로 메시지 출력
- `@read(element)`: 요소 텍스트 읽기
- `@ask(dict_key)`: 음성으로 질문 → STT로 응답 수집

**User Dictionary**:
```python
user_dict = {
  "name": "John Doe",
  "email": "john@example.com",
  "gift_card_code": "ABC123"
}
```

---

## Key Design Decisions

### 1. 왜 2단계 파이프라인인가?

| 접근법 | 장점 | 단점 |
|--------|------|------|
| End-to-End | 단순한 학습 | 긴 지시 처리 어려움, 해석 불가 |
| 2단계 (RUSS) | 모듈 독립 개선, 디버깅 용이 | 오류 전파, 복잡한 파이프라인 |

**RUSS 선택 근거**: End-to-End 대비 +17.2% 성능 향상 [Table 4](../2103.16057.md#table-4)

### 2. 왜 합성 데이터인가?

| 데이터 유형 | 장점 | 단점 |
|------------|------|------|
| 실제 데이터 | 분포 정확 | 수집 비용 높음, 규모 제한 |
| 합성 데이터 | 무한 확장, 저비용 | 분포 차이 |

**RUSS 선택 근거**:
- 1.5M 합성 샘플로 87% 파싱 정확도 달성
- Entity extraction으로 분포 차이 완화

### 3. 왜 SentenceBERT인가?

**요구사항**: 텍스트 설명과 DOM 요소 텍스트 유사도 계산

| 방법 | 장점 | 단점 |
|------|------|------|
| TF-IDF | 빠름 | 의미 유사성 미반영 |
| Word2Vec 평균 | 빠름 | 문맥 무시 |
| SentenceBERT | 의미적 유사성 | 계산 비용 |

**RUSS 선택 근거**: 후보 필터링 후 적은 요소에만 적용하여 비용 절감

---

## Comparison with Alternatives

### vs End-to-End Neural (PhraseNode)

```
PhraseNode:
  "Click reset button" ──neural──▶ element_id
                       one-step
  Accuracy: 46.5%

RUSS:
  "Click reset button" ──parse──▶ @retrieve(descr="reset", type="button")
                                        │
                                  ──ground──▶ element_id
                       two-step
  Accuracy: 63.6%
```

### vs Template-based Systems (CoScripter)

```
CoScripter:
  - 사용자 시연 필요
  - 제한된 자연어 파싱
  - 웹사이트별 스크립트

RUSS:
  - 자연어 지시만으로 동작
  - 오픈 도메인 파싱
  - 범용 그라운딩
```

---

## Implementation Details

### 훈련 설정

| 항목 | 값 |
|------|-----|
| 훈련 시간 | ~7시간 (NVIDIA V100) |
| 합성 데이터 | 1.5M 샘플 |
| Entity extraction | URL, LOC, TYPE |
| 추론 시간 | <1분/태스크 |

### 한계 및 개선 방향

1. **이미지 기반 요소**: 시각적 그라운딩 필요
2. **드롭다운 선택**: ThingTalk 확장 필요
3. **CSS 클래스 내 텍스트**: DOM 파싱 개선 필요
4. **오류 복구**: 런타임 에러 핸들링 강화

---

[Back to Main Analysis](../2103.16057-analysis.md) | [TL;DR](tldr.md) | [Contributions](contributions.md)
