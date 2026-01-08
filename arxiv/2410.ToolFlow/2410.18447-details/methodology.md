---
parent: "../2410.18447-analysis.md"
source: "../2410.18447.md"
type: "methodology-analysis"
created: "2026-01-09"
---

# ToolFlow: Detailed Methodology Analysis

[Back to Main Analysis](../2410.18447-analysis.md) | [Original Paper](../2410.18447.md)

---

## Methodology Overview

ToolFlow는 3단계 데이터 합성 파이프라인이다:
1. **Tool Selection**: Graph-based Sampling
2. **Planning**: Dialogue Plan Generation
3. **Synthesis**: Multi-Agent Dialogue Generation

---

## Stage 1: Graph-based Sampling for Tool Selection

### 1.1 Tool Graph Construction ([Section 3.1.1](../2410.18447.md#311-tool-graph-construction))

**그래프 정의**:
- $G = (V, E)$
- $v_i \in V$: 도구 $i$
- $e_{i,j} \in E$: 도구 $i$와 $j$의 연관성

**유사도 유형**:

| 유형 | 정의 | 예시 |
|------|------|------|
| **P-P** | 두 도구의 파라미터 유사 | `book_flight`, `get_weather` 모두 location 필요 |
| **P-R** | 한 도구의 반환값 = 다른 도구의 입력 | `check_calendar` → location → `navigate` |

**임베딩 생성**:
```
Template: "{Name}: {Description}"
예: "Date: Departure date, format as dd/mm/yyyy."
     ↓
Sentence-BERT 인코딩
     ↓
임베딩 벡터 p_i^k 또는 r_i^l
```

**유사도 계산**:
$$\text{sim}(p_i^k, p_j^l) = \cos(\mathbf{p}_i^k, \mathbf{p}_j^l)$$

**에지 생성 조건**:
$$e_{i,j} = \begin{cases} 1 & \text{if } \exists \text{ pair with sim} > \tau \\ 0 & \text{otherwise} \end{cases}$$

- $\tau = 0.82$ (예비 연구 기반 설정)

### 1.2 Graph-based Sampling Algorithm ([Algorithm 1](../2410.18447.md#312-graph-based-sampling))

```python
def graph_based_sampling(G, n):
    """
    G: Tool Graph (V, E)
    n: desired sample size
    Returns: subset of n tools
    """
    V_hat = {random.choice(V)}  # 시작 노드

    while len(V_hat) < n:
        v_i = V_hat[-1]  # 마지막 노드
        neighbors = {v_j for v_j in V if e[i,j] == 1}
        v_j = random.choice(neighbors)

        if v_j not in V_hat:
            V_hat.add(v_j)

    return V_hat
```

**특징**:
- 랜덤 워크 기반 샘플링
- 중복 방지
- 연관된 도구 집합 보장

---

## Stage 2: Dialogue Plan Generation ([Section 3.2](../2410.18447.md#32-dialogue-plan-generation))

### 2.1 계획의 구성 요소

| 요청 유형 | 설명 | 역할 |
|----------|------|------|
| **Tool Call Request** | 도구 사용 필요 요청 | 핵심 태스크 |
| **Chitchat** | 도구 불필요 대화 | 자연스러운 전환 |

### 2.2 계획 생성 프롬프트 (Table 14 기반)

**핵심 요구사항**:
1. 도구 이름 직접 언급 금지 (사용자 자유도 보장)
2. 연속 요청 간 연결성 확보
3. 복잡한 요청 설계 (다중 도구 호출)
4. 목표 턴 수 준수

**계획 예시**:
```
Target turns: 5

1. Tool Call Request: 회의실 예약 요청
2. Tool Call Request: 예약 수정 요청
3. Chitchat: 미팅 과다에 대한 잡담
4. Tool Call Request: 다중 미팅 예약 + 일정 추가 + 이메일 발송
```

### 2.3 계획의 효과

**논리에만 집중**:
- 세부 표현/용어는 합성 단계에서 처리
- 대화 구조의 일관성에 집중

**Chitchat 포함 이유**:
1. 대화 콘텐츠 다양화
2. 주제 간 자연스러운 전환
3. 실제 대화 패턴 반영

---

## Stage 3: Multi-Agent Dialogue Synthesis ([Section 3.3](../2410.18447.md#33-multi-agent-dialogue-synthesis))

### 3.1 Agent 정의

| Agent | 입력 | 출력 | 핵심 로직 |
|-------|------|------|-----------|
| **User** | 계획, 대화 이력 | 사용자 발화 | 현재 요청 완료 확인 → 다음 요청 or 계속 |
| **Assistant** | 사용자 발화, 도구 문서 | 응답 or 도구 호출 | 도구 필요성 판단 → 파라미터 확인 |
| **Tool** | 호출문, 도구 문서 | 반환값 | 반환값 시뮬레이션 |

### 3.2 Dialogue Turn 흐름

```
User: 요청 생성 (계획 기반)
    ↓
Assistant: 도구 필요?
    ├─ No → 직접 응답 (chitchat)
    └─ Yes → 파라미터 완전?
              ├─ No → 사용자에게 질문
              └─ Yes → 도구 호출문 생성
                          ↓
                       Tool: 반환값 시뮬레이션
                          ↓
                       Assistant: 결과 기반 응답
```

### 3.3 종료 조건
1. 계획의 모든 요청 처리 완료
2. 사전 설정된 턴 제한 도달

### 3.4 품질 필터링

| 필터링 규칙 | 제거 대상 |
|-------------|-----------|
| 형식 검증 | 잘못된 도구 호출문 형식 |
| 완성도 검증 | 불완전한 대화 |
| 호출 검증 | 도구 호출 누락 턴 |

---

## Implementation Details ([Section 3.4](../2410.18447.md#34-implementation-details))

### 도구 데이터
- **소스**: ToolBench (16,000+ RESTful APIs)
- **형식 변환**: OpenAI Function Calling JSON
- **변환 모델**: LLaMA-3.1-8B (정보 누락 시 추론 보완)

### 생성 설정
- **생성 모델**: GPT-4 (다양성을 위해 여러 버전 랜덤 선택)
- **데이터 규모**: 8,000 대화

### 도구 JSON 형식 예시 (Figure 2)
```json
{
  "type": "function",
  "function": {
    "name": "getcurrency",
    "description": "Get the current exchange rate...",
    "parameters": {
      "type": "object",
      "properties": {
        "basecurrency": {"type": "string", "description": "..."},
        "targetcurrency": {"type": "string", "description": "..."}
      },
      "required": ["basecurrency", "targetcurrency"]
    },
    "results": {
      "type": "object",
      "properties": {
        "exchangerate": {"type": "number", "description": "..."},
        "last_updated": {"type": "string", "description": "..."}
      }
    }
  }
}
```

---

## Methodology Validation

### 데이터 품질 평가 ([Section 4](../2410.18447.md#4-data-quality-assessment))

**자동 평가 메트릭**:
| 메트릭 | 측정 대상 | 계산 방법 |
|--------|-----------|-----------|
| SS | 일관성 | 연속 턴 간 시맨틱 유사도 |
| EnR | 일관성 | NLI 함의 관계 비율 |
| H | 다양성 | Shannon 엔트로피 |
| D-3 | 다양성 | Distinct-3 점수 |

**GPT-4 평가 차원**:
- NAT (Naturalness): 현실성
- COH (Coherence): 일관성
- HELP (Helpfulness): 유용성
- ACC (Accuracy): 정확성

### Ablation Study 결과 ([Table 2](../2410.18447.md#42-quality-evaluation))

| Setting | SS | EnR | H | D-3 | NAT | COH |
|---------|-----|-----|------|-------|-----|-----|
| Both | **63.36** | **47.3** | **10.36** | **0.4865** | **3.72** | **3.91** |
| -Plan | 62.03 | 32.1 | 10.14 | 0.4364 | 3.00 | 3.72 |
| -Graph | 61.72 | 48.1 | 9.82 | 0.3393 | 3.51 | 3.88 |
| Neither | 58.65 | 35.4 | 9.75 | 0.3078 | 2.93 | 3.66 |

**해석**:
- Graph 제거 → 다양성(H, D-3) 하락
- Plan 제거 → 일관성(SS, EnR, COH) 하락

---

## Reproducibility Considerations

### 필수 자원
- Sentence-BERT 모델 (all-MiniLM-L12-v2 언급)
- ToolBench 도구 데이터셋
- GPT-4 API 접근 (또는 LLaMA-3.1-8B 대체 가능)

### 하이퍼파라미터
| 파라미터 | 값 | 비고 |
|----------|-----|------|
| 유사도 임계값 τ | 0.82 | 예비 연구 기반 |
| 합성 대화 수 | 8,000 | |
| 총 토큰 수 | ~8M | |

### 재현 시 주의사항
1. GPT-4 버전에 따른 결과 변동 가능
2. 도구 JSON 변환 품질이 그래프 품질에 영향
3. 필터링 규칙 세부사항 명시 부족

---

[Back to Main Analysis](../2410.18447-analysis.md)
