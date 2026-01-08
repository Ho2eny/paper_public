---
parent: "../2404.09992-analysis.md"
source: "../2404.09992.md"
type: detail
created: 2026-01-09
---

# Detailed Methodology Analysis

[Back to Main Analysis](../2404.09992-analysis.md)

---

## 1. Environment Design

### 1.1 MDP Formulation

웹 브라우징을 **Partially Observable Markov Decision Process**로 모델링:

$$\langle S, A, P, R \rangle$$

| Component | Definition | Implementation |
|-----------|------------|----------------|
| $S$ (State) | 전체 인터넷 + 브라우저 상태 | 실제로는 표현 불가 |
| $\Omega$ (Observation) | $S$의 부분 관측 | Accessibility tree + Screenshot |
| $A$ (Action) | 가능한 행동 집합 | 12개 표준화된 액션 |
| $P$ (Transition) | 상태 전이 확률 | 인터넷 world knowledge (암묵적) |
| $R$ (Reward) | 보상 함수 | PASS/FAIL per hop |

Source: [Section 3.1](../2404.09992.md#31-environment)

### 1.2 Observation Space $\Omega$

```
Observation at time t:
├── Accessibility Tree
│   ├── Element ID
│   ├── Element Type
│   └── Text Content
├── Screenshot (with element IDs painted on images)
├── Linked Images (downloaded)
└── Action/State History
```

**특징**:
- Accessibility tree로 웹 페이지 구조화 표현
- 이미지에 element ID 오버레이 → 멀티모달 참조 가능
- 히스토리 포함으로 context 제공

### 1.3 Action Space $A$

VisualWebArena [18]의 12개 액션 채택:

| Action Type | Description |
|-------------|-------------|
| click | 요소 클릭 |
| scroll_up/down | 스크롤 |
| type | 키보드 입력 |
| navigate | URL 이동 |
| stop | 태스크 종료 |
| ... | (기타 mouse/keyboard 액션) |

**구현**: Playwright 라이브러리 + X graphics server

---

## 2. Dataset Construction

### 2.1 Data Generation Pipeline

```
Step 1: Template Creation
    - Human annotators create task templates
    - Each template generates 2-10 tasks

Step 2: GPT-4V Augmentation
    - WebQA 스타일 질문 생성
    - Multimodal content 포함 (e.g., "buy a yellow jacket")

Step 3: Minimalist Annotation
    - Annotators as "omniscient readers"
    - Shortest path 기록
    - 모든 crucial nodes 포함

Step 4: Cross-validation
    - Task diversity 검증
    - Answer accuracy 확인
```

Source: [Section 3.2](../2404.09992.md#32-dataset-construction)

### 2.2 Prompt Structure

```
PROMPT = {
    "Instructions": 기본 개념 (tasks, accessibility tree, actions),
    "Rules": 규칙 (e.g., "do nothing after [stop]"),
    "QA Examples": 이해를 위한 예시,
    "Reference Websites": 방문 가능한 웹사이트 목록,
    "Task": 실제 수행할 멀티홉 태스크
}
```

### 2.3 Task Distribution

**Domains** (Figure 2a):
- Shopping, Wikipedia, Travel, Events, Food, etc.

**Intents** (Figure 2b):
- Information seeking, Action execution

**Hop Distribution** (Figure 2c):
- 1-hop: ~500 tasks
- 2-4 hops: ~400 tasks
- 5+ hops: ~150 tasks

---

## 3. Evaluation Methodology

### 3.1 Single-hop Evaluation

#### Method 1: must_include (Strict)

```python
def must_include(response, keywords):
    for keyword in keywords:
        if keyword not in response:
            return "FAIL"
    return "PASS"
```

#### Method 2: fuzzy_match (Flexible)

```python
def fuzzy_match(response, reference):
    prompt = f"Given '{response}', would it be correct to infer '{reference}'? Yes or No"
    answer = gpt_35_turbo(prompt)
    return "PASS" if answer == "Yes" else "FAIL"
```

**활용 시나리오**:
- 색상 인식: "gold" ≈ "yellow" → fuzzy_match로 PASS

Source: [Section 3.5](../2404.09992.md#35-evaluation)

### 3.2 Multihop Evaluation

**Evaluation Queue**:
```
Queue = [hop_1_condition, hop_2_condition, ..., hop_N_condition, END]
Length = N + 1
```

**평가 로직**:
```python
def evaluate_multihop(agent_trajectory, queue):
    current_hop = 0
    for action in agent_trajectory:
        if satisfies(action, queue[current_hop]):
            current_hop += 1
        if current_hop == len(queue) - 1:  # Reached END
            return "TASK_PASS"
    return f"FAILED_AT_HOP_{current_hop}"
```

**조건 유형**:
- 정보 획득: answer string 포함
- 상태 도달: 특정 URL 방문

### 3.3 Metrics Definition

| Metric | Formula | Description |
|--------|---------|-------------|
| Hop Success Rate | $\frac{\text{successful hops}}{\text{total hops}} \times 100$ | 개별 hop 성공률 |
| Task Success Rate | $\frac{\text{completed tasks}}{\text{total tasks}} \times 100$ | 전체 태스크 성공률 |

---

## 4. Memory Augmentation Method

### 4.1 Three Memory Systems

#### Semantic Memory
- **저장**: LLM/LMM weights에 인코딩
- **내용**: 일반 세계 지식, 인터넷 navigation 개념
- **업데이트**: 학습 시 (frozen during inference)

#### Episodic Memory
- **저장**: Autoregressive model context / In-context examples
- **내용**: 현재 태스크의 step-by-step 액션 궤적
- **업데이트**: 실시간 (매 step)

#### Procedural Memory
- **저장**: 프롬프트에 append
- **내용**: 완료된 태스크의 전체 액션 시퀀스 + 결과
- **업데이트**: 태스크 완료 후

Source: [Section 4.4](../2404.09992.md#44-memory-augmented-agents), [Figure 4](../2404.09992.md#figure-4)

### 4.2 Procedural Memory Implementation

```python
def construct_prompt_with_memory(current_task, memory_bank, K):
    # 유사한 K개 태스크 선택
    similar_tasks = memory_bank.get_similar(current_task, K)

    # 각 태스크의 궤적 추출
    trajectories = []
    for task in similar_tasks:
        trajectory = {
            "task_description": task.description,
            "observations": task.web_content_observations,
            "actions": task.action_sequence,
            "outcome": task.result
        }
        trajectories.append(trajectory)

    # 프롬프트 구성
    prompt = f"""
    [Previous Task Examples]
    {format_trajectories(trajectories)}

    [Current Task]
    {current_task.description}
    """
    return prompt
```

### 4.3 Optimal K Selection

**Experimental Findings** (Figure 5):

| Domain | Optimal K | Observation |
|--------|-----------|-------------|
| Shopping | 1-2 | 단순 태스크, 작은 K가 효과적 |
| Wikipedia | 1-2 | 정보 검색, 과다 context 해로움 |
| Travel | 2 | 복합 태스크, 약간 더 많은 예시 필요 |

**Trade-off**:
- K 증가 → Input length $\times K$ 배 증가
- K 증가 → Bias/disturbance 도입
- 최적점: K=1 또는 K=2

### 4.4 LMM Optimization Objective

멀티모달 instruction tuning의 최적화 목표:

$$\mathcal{L} = -\sum_{t} \log P(R_t | I, M, R_{<t})$$

Where:
- $I$: Instruction (task description)
- $M$: Multimodal input (images + text)
- $R_t$: Response token at time $t$
- $R_{<t}$: Previous response tokens

Source: [Appendix C](../2404.09992.md#c-more-related-works)

---

## 5. Experimental Setup

### 5.1 Environment Settings

| Setting | Value |
|---------|-------|
| Viewport | 1280 x 2048 |
| GPU | NVIDIA RTX6000 Ada (48GB) |
| Inference Time | 4-8 hours per epoch |
| Model Size | 7B-9B parameters |

### 5.2 Model Versions

| Model Name | API Version |
|------------|-------------|
| GPT-4 | gpt-4-0125-preview |
| GPT-4o | gpt-4o-2024-11-20 |
| GPT-4V | gpt-4-vision-preview |
| Gemini-Pro | gemini-1.0-pro-001 |
| Gemini-Pro-Vision | gemini-1.0-pro-vision-001 |

Source: [Section A.1](../2404.09992.md#a1-environment--parameter-settings)

---

## 6. Key Findings from Methodology

### 6.1 Search Space Expansion Problem

**Single-hop behavior**:
- 하나의 reference URL
- 실패 시 같은 사이트에서 반복 시도
- 결과: 높은 성공률

**Multihop behavior**:
- 여러 reference URLs
- 실패 시 다른 사이트로 이동
- 결과: 과도한 탐색, 목표 상실

### 6.2 Early Hop Failure Pattern

**반직관적 현상**: 첫 번째 hop의 성공률이 총 hop 수에 의존

| Total Hops | 1st Hop SR (GPT-4V) |
|------------|---------------------|
| 2 | 56.5% |
| 3 | 22.73% |
| 4 | 12.5% |
| 5 | 12.28% |
| 6 | 16.67% |

**원인 분석**:
1. 확장된 search space
2. Zero-shot long-context 추론 능력 부족
3. 복잡한 프롬프트로 인한 주의 분산

Source: [Table 3](../2404.09992.md#table-3), [Section 4.3](../2404.09992.md#43-why-are-multihop-web-tasks-challenging)

[Back to Main Analysis](../2404.09992-analysis.md) | [TL;DR](tldr.md)
