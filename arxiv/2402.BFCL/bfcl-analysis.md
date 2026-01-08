---
title: "Berkeley Function Calling Leaderboard (BFCL) - Analysis"
arxiv_id: "2402.BFCL"
type: "analysis"
paper_type: "benchmark"
analyzed_date: "2026-01-10"
source: "./bfcl.md"
details: "./bfcl-details/"
venue: "ICML 2025"
---

# Berkeley Function Calling Leaderboard (BFCL) - Analysis

## 1. Overview

**1문단: 필요성**
이 논문은 LLM의 **function calling(tool use)** 능력을 평가하기 위한 포괄적인 벤치마크를 제안한다. Function calling은 LLM이 외부 함수, API, 사용자 정의 도구를 호출하여 실시간 정보 검색이나 실제 환경에서의 액션을 수행할 수 있게 하는 핵심 능력이다. 기존 벤치마크들(APIBench, ToolBench 등)은 stateless한 평가에 그치거나, LLM 기반 evaluator에 의존하여 재현성과 객관성에 문제가 있었다. 또한 실제 사용자 쿼리 기반 평가가 부재했다.

**2문단: 설계**
BFCL은 4개 파트로 구성된다: (1) **Single-turn** - 단일 턴 function calling 시나리오(simple, multiple, parallel, parallel multiple, irrelevance), (2) **Crowd-sourced** - 64,517개 실제 사용자 쿼리에서 큐레이션된 2,251개 데이터포인트, (3) **Multi-turn** - 8개 API 도메인에서 1,000개 쿼리를 통한 지속적 컨텍스트 관리 평가, (4) **Agentic** - 웹 검색, 메모리 관리, SQL 쿼리 등 실제 에이전트 태스크 평가. Python, Java, JavaScript, REST API, SQL 등 다양한 프로그래밍 언어를 지원한다.

**3문단: 평가**
핵심 기술적 기여는 **AST(Abstract Syntax Tree) substring matching** 기반 평가 방식이다. 실제 함수 실행 없이도 function call의 정확성을 검증할 수 있어 대규모 평가가 가능하다. 실행 기반 평가와의 강한 상관관계(Figure 3)가 검증되었다. 다양한 LLM을 평가한 결과, GPT-4o-2024-11-20이 66.4%로 최고 성능을 보였으나, multi-turn과 memory 관리 태스크에서는 모든 모델이 저조한 성능을 보여 여전히 개선의 여지가 크다.

**4문단: 의의**
BFCL은 출시 이후 function calling 평가의 **de facto standard**가 되었다. Meta, Anthropic, Cohere, Qwen 등 주요 LLM 제공자들이 모델 평가에 활용하고 있다. 특히 crowd-sourced 데이터셋을 통해 기존 벤치마크에 대한 **data contamination** 탐지가 가능하며, perplexity/char-NLL 비교를 통해 모델의 실제 일반화 능력을 검증할 수 있다.

---

## 2. Core Section

### TL;DR

> LLM의 function calling 능력을 5,551개 데이터포인트로 평가하는 벤치마크로, **AST 기반 결정론적 평가**와 **실제 사용자 쿼리** 포함이 핵심이며, single-turn부터 agentic 시나리오까지 다양한 복잡도를 커버한다.

→ 상세: [tldr.md](./bfcl-details/tldr.md)

### Core Contributions

1. **AST Substring Matching 평가 방식**: 함수 실행 없이 function call 정확성 검증 → 대규모 결정론적 평가 가능
2. **Crowd-sourced 실제 사용자 데이터**: 67,000개+ 실제 쿼리에서 큐레이션 → data contamination 탐지 가능
3. **Multi-turn + Agentic 평가**: 지속적 컨텍스트 관리와 상태 기반 추론 평가 → 에이전트 능력 종합 진단
4. **다중 프로그래밍 언어 지원**: Python, Java, JavaScript, REST, SQL → 실제 개발 환경 반영

→ 상세: [contributions.md](./bfcl-details/contributions.md)

### Key vs Non-Key Sections

| Priority | Sections | Reason |
|----------|----------|--------|
| ⭐⭐⭐ Must Read | Section 3 (Dataset), Section 4.1 (AST Matching) | 벤치마크 구조와 핵심 평가 방식 이해 필수 |
| ⭐⭐⭐ Must Read | Section 5.1 (Accuracy), Table 1 | 모델별 성능 비교 핵심 결과 |
| ⭐⭐ Important | Section 5.4 (Multi-turn Error Analysis) | 실패 원인 분석 통해 개선 방향 도출 |
| ⭐⭐ Important | Section 5.5 (Data Contamination) | 벤치마크 신뢰성 검증 방법 |
| ⭐ Reference | Section 2 (Related Work) | 기존 벤치마크와의 차별점 |
| Skip | Appendix B-E | 데이터 생성 상세 (필요시 참조) |

→ 상세: [key-sections.md](./bfcl-details/key-sections.md)

---

## 3. Paper Type

**Type**: Benchmark

| Aspect | Value |
|--------|-------|
| **Evaluation Target** | LLM Function Calling / Tool Use |
| **Task Count** | 5,551 datapoints (Single-turn + Crowd-sourced + Multi-turn + Agentic) |
| **Domains** | Vehicle Control, Trading, Travel, File System, Messaging, Twitter, Math, SQL, Web Search, Memory |
| **Main Metrics** | AST Matching Accuracy, Execution Accuracy, State/Response-based Evaluation |
| **Languages** | Python, Java, JavaScript, REST API, SQL |

→ 상세 방법론: [methodology.md](./bfcl-details/methodology.md)

---

## 4. Visual Analysis

### Key Figures

#### Figure 1: BFCL Category Visualization

**구성 요소**:
- **Inner Ring**: 4개 주요 섹션 (Single-turn, Crowd-sourced, Multi-turn, Agentic)
- **Middle Ring**: 평가 방법 (AST, Execution, State/Response)
- **Outer Ring**: 세부 카테고리 (Simple, Multiple, Parallel 등)

**핵심 통찰**:
- Single-turn이 가장 많은 비중 차지 (기본 능력 평가)
- Agentic이 가장 도전적 (Memory: 최고 12% 정확도)
- 계층적 복잡도 설계로 모델 능력 단계별 진단 가능

**Source**: [Figure 1](./bfcl.md#figure-1-bfcl-category-visualization)

---

#### Figure 3: AST vs Execution Correlation

**구성 요소**:
- X축: Execution Summary Score
- Y축: AST Summary Score
- 점선: 이상적 상관관계

**핵심 통찰**:
- AST 점수와 Execution 점수 간 **강한 양의 상관관계**
- AST가 실제 실행 없이도 신뢰할 수 있는 평가 지표임을 검증
- 일부 모델에서 미세한 차이 존재 (실행 환경 의존적 케이스)

**Source**: [Figure 3](./bfcl.md#figure-3-ast-vs-execution-evaluation-correlation)

---

### Tables Interpretation

#### Table 1: Main Benchmark Results

| Model | Overall Acc | Single-Turn | Crowd-sourced | Multi-turn | Agentic (Memory) |
|-------|-------------|-------------|---------------|------------|------------------|
| GPT-4o-2024-11-20 (Prompt) | **66.4%** | 79.4% | 84.9% | 41.0% | 6.0% |
| GPT-4o-2024-11-20 (FC) | 65.8% | 77.2% | 81.4% | 37.5% | 0.0% |
| o1-2024-12-17 (Prompt) | 59.1% | 72.7% | 82.9% | 48.5% | 12.0% |
| Qwen2.5-72B-Instruct | 57.0% | 80.2% | 85.3% | 15.5% | 8.0% |

**주요 발견**:
1. **Single-turn vs Multi-turn Gap**: Single-turn에서 80%+ 성능이 Multi-turn에서 15-48%로 급락
2. **Memory 태스크 극심한 어려움**: 최고 12% (o1), 대부분 모델 10% 미만
3. **Prompt vs FC mode**: 복잡한 시나리오에서 Prompt 모드가 더 유연
4. **Open vs Proprietary Gap**: 오픈소스 모델 전반적으로 10-20% 낮은 성능

**실무적 의미**:
- Production 환경에서 multi-turn function calling 사용 시 주의 필요
- Memory 관리 기능은 아직 신뢰하기 어려움
- Claude-3.5-Sonnet FC 모드의 parallel call 불가 이슈 확인

**Source**: [Table 1](./bfcl.md#table-1-evaluating-different-llms-on-bfcl)

---

#### Figure 5: Error Type Distribution

**분류된 에러 유형**:
1. **Failed to Understand Environment State** (가장 빈번) - 환경 상태 오인 또는 hallucination
2. **Failed to Understand User's Request** - 사용자 의도 오해석
3. **Failed to Understand Function Documentation** - 함수 문서 오독

**핵심 통찰**:
- Environment State 이해 실패가 **주요 실패 원인**
- 함수 문서 이해는 상대적으로 양호
- Multi-turn에서 상태 추적 능력이 핵심 병목

**Source**: [Figure 5](./bfcl.md#figure-5-error-types-frequency-distribution)

---

## 5. Critique & Related Works

### Expert Critique

#### Strengths
1. **결정론적 평가**: AST matching으로 LLM-as-judge 의존 제거 → 재현성 확보
2. **실제 사용자 데이터**: Crowd-sourced로 현실 복잡성 반영 + contamination 탐지 가능
3. **De facto Standard 지위**: 주요 LLM 제공자들이 공식 채택 → 비교 용이
4. **포괄적 커버리지**: Single-turn부터 Agentic까지 → 단계별 진단 가능

#### Limitations
1. **Stateless 평가 중심**: ToolSandbox와 달리 implicit state dependency 미평가
2. **User Simulator 미포함**: 실제 대화형 평가 불가 (off-policy 평가)
3. **도메인 편향**: 특정 도메인(Vehicle, Trading 등)에 편중
4. **Memory 평가 한계**: 12% 최고 성능으로 변별력 의문

#### Adoption Status
- [x] Widely used (de facto standard)
- [x] Easy to set up (GitHub 제공)
- [x] Clear leaderboard (https://gorilla.cs.berkeley.edu/leaderboard.html)
- [x] Active maintenance (v4까지 진화)

#### 2026 Perspective
- **Still Valid**: Single-turn AST 평가, Crowd-sourced contamination 탐지
- **Outdated**: Multi-turn 평가 방식 (state dependency 미고려)
- **Missing**: 최신 모델(Claude 4, GPT-5 등) 결과, Tool orchestration 평가

### Related Works

1. **ToolSandbox** (Apple, 2024) - Stateful, Conversational, Interactive 평가 → BFCL보다 state dependency에 강점
2. **ToolBeHonest** (2024) - Hallucination 진단 → Unsolvable task 탐지에 특화
3. **τ-bench** (Princeton, 2024) - Tool-Agent-User 상호작용 → 더 현실적 시나리오
4. **StableToolBench** (2024) - RapidAPI 재현성 문제 해결 → Caching 기반 안정성
5. **API-BLEND** (IBM, 2024) - Training/Benchmarking 통합 코퍼스

---

## Navigation

- **Source**: [원본 논문](./bfcl.md)
- **Details**:
  - [TL;DR 상세](./bfcl-details/tldr.md)
  - [Contributions 상세](./bfcl-details/contributions.md)
  - [Key Sections 상세](./bfcl-details/key-sections.md)
  - [Methodology 상세](./bfcl-details/methodology.md)
