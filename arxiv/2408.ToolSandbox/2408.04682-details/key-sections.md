---
parent: ../2408.04682-analysis.md
source: ../2408.04682.md
---

# Key vs Non-Key Sections - Deep Analysis

## ⭐⭐⭐ Must Read

### Section 2: Benchmark Design

- **Why**: 벤치마크의 **3가지 핵심 특징** (Stateful, Conversational, Interactive) 상세 설명
- **Key Points**:
  - 2.1 Stateful Tool Execution: World state, dependency chain
  - 2.2 LLM User Simulator: Knowledge Boundary, Demonstration
  - 2.3 Milestones and Minefields: DAG 기반 평가
- **Time**: ~20분
- **Link**: [Section 2](../2408.04682.md#2-benchmark-design)

### Table 5 + Section 4: Main Results

- **Why**: **모델별 성능 비교**의 핵심 레퍼런스
- **Key Points**:
  - 13개 모델 (7 proprietary + 6 open-source) 비교
  - 카테고리별 성능 분석 (State Dependency, Canonicalization, Insufficient Info)
  - Open vs Proprietary 격차 (20점+)
- **Time**: ~15분
- **Link**: [Table 5](../2408.04682.md#table-5-comparing-the-average-similarity-score)

### Table 1: Benchmark Comparison

- **Why**: 기존 벤치마크와의 **차별점** 명확히 이해
- **Key Points**:
  - Stateful Tools: ToolSandbox만 ✓
  - On-policy: ToolSandbox만 ✓
  - Conversational: ToolSandbox, τ-bench만 ✓
- **Time**: ~5분
- **Link**: [Table 1](../2408.04682.md#table-1-comparison-of-toolsandbox)

---

## ⭐⭐ Important

### Section 3: Test Scenarios

- **Why**: **6가지 시나리오 카테고리** 이해로 평가 구조 파악
- **Key Points**:
  - Simple, Multi-step: 기본 복잡도
  - State Dependency: Implicit dependency
  - Canonicalization: 표현 변환
  - Insufficient Information: Unsolvable 인식
  - Sandbox Message: 에러 처리
- **Time**: ~10분
- **Link**: [Section 3](../2408.04682.md#3-test-scenarios)

### Section 2.2: User Simulator + Table 2-3

- **Why**: User simulator **설계와 에러율** 상세
- **Key Points**:
  - Knowledge Boundary 효과: hallucination 12.4%→7.75%
  - Demonstration 효과: instruction error 6.2%→0.77%
  - 최종 에러율 ~8%
- **Time**: ~10분
- **Link**: [Section 2.2](../2408.04682.md#22-llm-user-simulator)

### Section 4.4: State Dependency Analysis

- **Why**: **Parallel call 문제** 심층 분석
- **Key Points**:
  - 대형 모델(GPT-4, Opus)이 중형보다 낮은 점수
  - Parallel tool call이 dependency 무시
  - Sequential call 강제 시 성능 개선
- **Time**: ~8분
- **Link**: [Section 4.4](../2408.04682.md#44-state-dependency-analysis)

### Figure 1: Example Evaluation Trajectory

- **Why**: 평가 프로세스 **전체 흐름** 시각적 이해
- **Key Points**:
  - Message Bus 구조
  - World State 스냅샷
  - Milestone DAG 매칭
- **Time**: ~5분
- **Link**: [Figure 1](../2408.04682.md#figure-1-an-example-evaluation-trajectory-from-toolsandbox)

---

## ⭐ Reference (필요시 참조)

### Section 2.1: Execution Environment Details

- **When to read**: Execution Context 구현 상세 필요시
- **Content**: Database schema, Tool signatures, State management
- **Link**: [Section 2.1](../2408.04682.md#21-stateful-tool-execution)

### Section 4.2-4.3: Turn Count & Multi-step Analysis

- **When to read**: 효율성 및 복잡도별 성능 분석시
- **Content**: Average turns, Multi-step 카테고리 breakdown
- **Link**: [Section 4.2](../2408.04682.md#42-turn-count-analysis)

### Appendix A: Implementation Details

- **When to read**: 자체 구현 시도시
- **Content**: Execution Context, Tool 정의, API 스펙
- **Link**: [Appendix A](../2408.04682.md#a-implementation-details)

---

## Skip (건너뛰기 가능)

### Appendix B: Prompt Templates

- **Why skip**: User/Agent 프롬프트 템플릿 상세, 핵심 이해에 불필요
- **When needed**: User simulator 재현 시도시
- **Link**: [Appendix B](../2408.04682.md#b-prompt-templates)

### Appendix C: Example Trajectories

- **Why skip**: 개별 사례 분석, 주요 결론은 본문에 포함
- **When needed**: 특정 시나리오 상세 분석시
- **Link**: [Appendix C](../2408.04682.md#c-example-trajectories)

### Appendix D: Full Tool List

- **Why skip**: 34개 도구 전체 목록, 분석에 불필요
- **When needed**: 도메인별 도구 확인시
- **Link**: [Appendix D](../2408.04682.md#d-full-tool-list)

---

## Reading Path Recommendations

### Quick Understanding (25분)
1. Table 1 (벤치마크 비교) → 2. Section 2 요약 (3가지 특징) → 3. Table 5 (결과)

### Deep Dive (50분)
위 + Section 3 (시나리오) + Section 4.4 (State Dependency 분석) + Section 2.2 (User Simulator)

### Full Understanding (1.5시간)
위 + Figure 1 (시각적 이해) + Appendix A (구현) + Appendix C (사례)

---

## Section Dependencies

```
Table 1 (비교)
     │
     ▼
Section 2 (설계) ──────────────────────────────────────┐
     │                                                  │
     ├──► 2.1 Stateful ──► Section 3 (시나리오)        │
     │                          │                       │
     ├──► 2.2 User Sim         │                       │
     │         │                │                       │
     └──► 2.3 Milestone ───────┘                       │
                │                                       │
                ▼                                       │
         Section 4 (결과) ◄─────────────────────────────┘
                │
                ├──► Table 5 (메인 결과)
                ├──► 4.4 (State Dependency)
                └──► 4.5 (Insufficient Info)
```
