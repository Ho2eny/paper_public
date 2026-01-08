---
parent: ../2401.13919-analysis.md
source: ../2401.13919.md
created: 2026-01-09
---

# Methodology - Deep Analysis

## Paper Type

**Type**: Method + Benchmark

---

## Approach Overview

### High-Level Summary
WebVoyager는 GPT-4V(ision)를 backbone으로 하여 실제 웹사이트에서 end-to-end로 사용자 태스크를 수행하는 멀티모달 웹 에이전트이다. 핵심 혁신은 Set-of-Mark Prompting을 웹 도메인에 적용하여, 스크린샷 위에 interactive element들을 숫자 라벨이 붙은 bounding box로 표시함으로써 grounding 문제를 해결한 것이다.

에이전트는 Selenium 기반의 실제 웹 브라우징 환경에서 동작하며, ReAct 스타일의 thought-action 패턴을 따른다. 각 스텝에서 현재 상황을 분석하는 thought를 생성한 후, 7가지 action 중 하나를 선택하여 실행한다. 최대 15 steps 동안 반복하거나 ANSWER action으로 태스크를 종료한다.

또한 open-ended 웹 태스크의 평가를 위해 GPT-4V를 활용한 자동 평가 프로토콜을 제안했다. Trajectory 스크린샷을 기반으로 태스크 성공 여부를 판단하며, 인간 평가자와 85.3% 일치율(Kappa 0.70)을 달성했다.

### Key Innovation
- **Set-of-Mark for Web**: Object detection 없이 rule-based JavaScript로 interactive element 추출 및 숫자 라벨 오버레이
- **End-to-End LMM Agent**: 추가 모델 없이 GPT-4V 단독으로 동작
- **Auto Evaluation Protocol**: GPT-4V 기반 scalable 평가

**Source**: [Method Overview](../2401.13919.md#3-webvoyager)

---

## Technical Details

### For Method Papers

#### Problem Formulation
| Aspect | Description | Source |
|--------|-------------|--------|
| **Input** | User instruction $I$ + 웹페이지 observation $o_t$ | [Section 3.2](../2401.13919.md#32-interaction-formulation) |
| **Output** | Action $a_t$ = (thought $s_t$, executable action $\hat{a}_t$) | [Section 3.2](../2401.13919.md#32-interaction-formulation) |
| **Objective** | Task completion (binary success) | [Section 5.1](../2401.13919.md#51-evaluation-methods) |
| **Constraints** | Max 15 steps, single tab only | [Section 3.1](../2401.13919.md#31-browsing-environment) |

#### Architecture / Pipeline

```
[User Instruction I]
       │
       ▼
┌─────────────────────────────────────────────────────┐
│                 WebVoyager Agent                     │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │ Context c_t = (o_1, a_1, ..., o_{t-1}, a_{t-1},│ │
│  │               o_t, I)                           │ │
│  │ (최근 3개 observation만 포함)                   │ │
│  └────────────────────────────────────────────────┘ │
│                       │                              │
│                       ▼                              │
│  ┌────────────────────────────────────────────────┐ │
│  │ GPT-4V: a_t = M(c_t) = (s_t, â_t)             │ │
│  │ - s_t: Natural language thought                │ │
│  │ - â_t: Executable action code                  │ │
│  └────────────────────────────────────────────────┘ │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│            Selenium Browser Environment              │
│  ┌─────────────────────────────────────────────────┐│
│  │ o_{t+1} = E(o_t, a_t)                           ││
│  │ - Action execution                              ││
│  │ - Screenshot capture                            ││
│  │ - Set-of-Mark overlay                          ││
│  └─────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────┘
                      │
                      ▼
              [Loop until ANSWER or max_steps]
```

**Components**:
1. **Observation Space**: Screenshot + Set-of-Mark + Auxiliary text - [Section 3.3](../2401.13919.md#33-observation-space)
2. **Action Space**: 7 actions (Click, Type, Scroll, Wait, Back, Google, Answer) - [Section 3.4](../2401.13919.md#34-action-space)
3. **Context Management**: Recent 3 observations + full action history - [Section 3.2](../2401.13919.md#32-interaction-formulation)

#### Training / Learning
| Aspect | Value | Source |
|--------|-------|--------|
| **Training** | None (zero-shot prompting) | [Section 3](../2401.13919.md#3-webvoyager) |
| **Backbone** | GPT-4V (gpt-4-vision-preview) | [Section 5.2](../2401.13919.md#52-result) |
| **Prompting** | ReAct-style system prompt | [Appendix A](../2401.13919.md#a-system-prompt) |

#### Inference
```python
def webvoyager_inference(instruction, browser):
    context = Context(instruction)

    for step in range(max_steps):
        # Observation: screenshot + Set-of-Mark
        screenshot = browser.capture_screenshot()
        marked_screenshot = add_setofmark(screenshot)
        aux_text = extract_element_info(browser)
        observation = (marked_screenshot, aux_text)

        context.add_observation(observation)

        # Action prediction via GPT-4V
        thought, action = gpt4v.predict(context)
        context.add_action(thought, action)

        # Execution
        if action.type == "ANSWER":
            return action.content

        try:
            browser.execute(action)
        except ActionError as e:
            context.add_error(e)
            continue

    return "Max steps reached"
```

**Source**: [Inference Section](../2401.13919.md#3-webvoyager)

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| Element Marking | Set-of-Mark (검정 bounding box) | 다색상 대비 높은 성공률 | 다색상, 텍스트 라벨 | [Section 3.3](../2401.13919.md#33-observation-space) |
| Context Management | 최근 3 observations | 효율성 + 정보 유지 균형 | 전체 유지, 1개만 | [Section 3.2](../2401.13919.md#32-interaction-formulation) |
| Action Space | 7 actions | 대부분 웹 태스크 커버 | Drag 추가, Hover 추가 | [Section 3.4](../2401.13919.md#34-action-space) |
| "Google" Action | 탈출구 제공 | Navigation stuck 방지 | 없음 | [Section 3.4](../2401.13919.md#34-action-space) |
| Window Size | 1024 x 768 | 텍스트 인식 + 효율성 | 더 높은 해상도 | [Section 3.1](../2401.13919.md#31-browsing-environment) |

---

## Observation Space Details

### Set-of-Mark Prompting Adaptation
[Section 3.3](../2401.13919.md#33-observation-space)

**Original Set-of-Mark** (Yang et al., 2023a):
- 일반 이미지에서 object detection으로 영역 추출
- 각 영역에 숫자 마커 오버레이

**WebVoyager Adaptation**:
- Object detection 불필요 (rule-based extraction)
- GPT-4V-ACT JavaScript 도구로 interactive elements 추출
- 웹 element type 기반 자동 bounding box 생성

```javascript
// 개념적 동작 (실제 코드 아님)
const interactiveElements = document.querySelectorAll(
  'a, button, input, select, textarea, [onclick], [role="button"]'
);
interactiveElements.forEach((el, idx) => {
  addBoundingBox(el, idx);  // 숫자 라벨 + 검은색 border
});
```

### Auxiliary Text Structure
```
Element [1]: type="button", text="Search", aria-label="Search products"
Element [2]: type="input", text="", aria-label="Enter your query"
Element [3]: type="link", text="Sign In", aria-label=""
...
```

---

## Action Space Details

### Action Taxonomy
```
Actions
├── Navigation
│   ├── Click [label]              # 요소 클릭
│   ├── Scroll [target]; [dir]     # 스크롤
│   ├── GoBack                     # 이전 페이지
│   └── Google                     # 검색엔진 이동
├── Input
│   └── Type [label]; [content]    # 텍스트 입력
├── Control
│   └── Wait                       # 로딩 대기
└── Termination
    └── ANSWER; [content]          # 태스크 완료
```

### Action Details

| Action | Format | Example |
|--------|--------|---------|
| Click | `Click [Label]` | `Click [5]` |
| Type | `Type [Label]; [Content]` | `Type [2]; machine learning` |
| Scroll | `Scroll [Target]; [Dir]` | `Scroll [WINDOW]; down` |
| Wait | `Wait` | `Wait` |
| GoBack | `GoBack` | `GoBack` |
| Google | `Google` | `Google` |
| Answer | `ANSWER; [Content]` | `ANSWER; The price is $29.99` |

---

## Assumptions & Limitations

### Assumptions
1. **GPT-4V 능력**: 스크린샷에서 텍스트 인식 및 시각적 이해 가능 - [Section 3](../2401.13919.md#3-webvoyager)
2. **웹 표준 준수**: Interactive elements가 표준 HTML 태그 사용 - [Section 3.3](../2401.13919.md#33-observation-space)
3. **단일 탭 충분**: 대부분 태스크가 단일 탭으로 완수 가능 - [Section 3.1](../2401.13919.md#31-browsing-environment)

### Limitations
1. **Action Space 한계**: Drag, hover, right-click 등 미지원 - [Limitations](../2401.13919.md#limitations)
2. **해상도 제약**: 작은 텍스트 인식 어려움 (1024x768)
3. **Login/CAPTCHA**: 인증 필요 사이트 접근 불가
4. **동적 콘텐츠**: 애니메이션, 비디오 미처리

### Scope
- **In Scope**: 정보 검색, 예약, 쇼핑 등 일반적 웹 태스크
- **Out of Scope**: 로그인 필요 서비스, 드래그 필수 태스크, 실시간 스트리밍

---

## Complexity Analysis

| Aspect | Complexity | Notes |
|--------|------------|-------|
| **Time per Step** | ~5-10 sec | GPT-4V API 호출 + 브라우저 실행 |
| **Max Steps** | 15 | 태스크당 최대 스텝 수 |
| **API Cost** | ~$0.5-1/task | GPT-4V 호출 비용 (추정) |
| **Data** | 643 tasks | 15 웹사이트 |

---

## Evaluation Protocol

### GPT-4V Auto Evaluation
[Section 5.1](../2401.13919.md#51-evaluation-methods)

```
Input:
  - Task instruction
  - Agent's final answer
  - Last k screenshots from trajectory

Prompt: (Figure 8 참조)
  "Given the task, screenshots, and agent response,
   determine if the task was completed successfully."

Output: Success / Failure

Settings:
  - Temperature: 0 (determinism)
  - 3회 실행 → 평균/표준편차 보고
```

### Hyperparameter: k (screenshots)

| k | Agreement | Kappa | Trade-off |
|---|-----------|-------|-----------|
| 1 | 75.3% | 0.51 | 빠름, 낮은 정확도 |
| 2 | 79.7% | 0.59 | 균형 |
| 3 | 81.3% | 0.62 | 균형 |
| Full | 85.3% | 0.70 | 높은 정확도, 비용 |

---

## Source References Summary

| Topic | Section | Link |
|-------|---------|------|
| Method Overview | WebVoyager | [Section 3](../2401.13919.md#3-webvoyager) |
| Environment | Browsing Setup | [Section 3.1](../2401.13919.md#31-browsing-environment) |
| Observation | Set-of-Mark | [Section 3.3](../2401.13919.md#33-observation-space) |
| Actions | Action Space | [Section 3.4](../2401.13919.md#34-action-space) |
| Benchmark | Task Collection | [Section 4](../2401.13919.md#4-benchmark-for-webvoyager) |
| Evaluation | Auto Eval Protocol | [Section 5.1](../2401.13919.md#51-evaluation-methods) |
| Results | Experiments | [Section 5.2](../2401.13919.md#52-result) |
| Error Analysis | Failure Patterns | [Section 5.4](../2401.13919.md#54-error-analysis) |

---

## Navigation

← [Back to Main Analysis](../2401.13919-analysis.md)
