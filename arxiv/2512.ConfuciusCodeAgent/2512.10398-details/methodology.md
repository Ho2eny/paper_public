---
parent: ../2512.10398-analysis.md
source: ../2512.10398.md
created: 2026-02-06
---

# Methodology - Deep Analysis

## Paper Type

**Type**: Method

---

## Approach Overview

### High-Level Summary

Confucius Code Agent(CCA)는 대규모 코드베이스를 위한 코딩 에이전트로, Confucius SDK라는 AX/UX/DX 분리 설계 플랫폼 위에 구축된다. 핵심 접근법은 에이전트의 인지 환경(스캐폴딩)을 체계적으로 최적화하는 것이다.

시스템은 4개 층으로 구성된다:
1. **오케스트레이터 층**: 최소한의 LLM 호출-해석-실행 루프 (Algorithm 1)
2. **메모리 층**: 계층적 워킹 메모리(세션 내) + 영구적 노트(세션 간)
3. **도구 층**: 익스텐션 기반 모듈형 도구 사용, 파싱, 프롬프트 조작
4. **메타 층**: 메타 에이전트가 에이전트 설정을 자동 생성/개선

### Key Innovation
- **AX/UX/DX 분리**: 에이전트/사용자/개발자에게 각각 다른 정보 채널 제공
- **Architect Agent 기반 적응적 압축**: 구조화된 요약으로 의미적 보존
- **hindsight notes**: 실패 사례를 포함한 영구적 cross-session learning
- **Meta-agent build-test-improve**: 수동 프롬프트 엔지니어링을 자동화

**Source**: [Section 1-2](../2512.10398.md#1-introduction)

---

## Technical Details

### For Method Papers

#### Problem Formulation

| Aspect | Description | Source |
|--------|-------------|--------|
| **Input** | GitHub 이슈 설명 + 전체 레포지토리 컨텍스트 | [Section 3.1](../2512.10398.md#31-setup) |
| **Output** | 이슈를 해결하는 코드 패치 (모든 테스트 통과) | [Section 3.1](../2512.10398.md#31-setup) |
| **Objective** | Resolve Rate (Pass@1) 최대화: 에이전트 패치가 모든 테스트를 통과하는 비율 | [Section 3.1](../2512.10398.md#31-setup) |
| **Constraints** | 최대 반복 횟수, 컨텍스트 윈도우, 도구 접근 동일 | [Section 3.1](../2512.10398.md#31-setup) |

#### Architecture / Pipeline

```
[GitHub Issue + Repository]
    ↓
[Confucius Orchestrator Loop]
├── LLM Call (Claude/GPT)
├── Output Parsing
│   ├── Native tool-use → structured actions
│   └── XML tags → same action representation
├── Extension Execution
│   ├── File Search / Edit
│   ├── Bash / CLI
│   └── Code Search
├── Memory Management
│   ├── Hierarchical Working Memory
│   │   └── Tree-structured namespace
│   └── Context Compression (Architect Agent)
│       └── Structured summary (goals, decisions, errors, TODOs)
└── Iteration Control
    ├── Agent-driven termination (no actions)
    └── Max iteration cap

[Post-Session]
├── Note-Taking Agent
│   └── Trajectory → Markdown notes
│       ├── Project-specific notes
│       ├── Shared insights
│       └── Hindsight notes (failures)
└── Meta-Agent (Development Time)
    ├── Build: Spec → Config + Prompts + Extensions
    ├── Test: Run on regression tasks
    └── Improve: Fix failures, re-test
```

**Components**:
1. **Orchestrator**: 최소한의 확장 가능 루프. 네이티브 tool-use와 XML 태그 양방 지원 - [Section 2.3.1](../2512.10398.md#231-the-confucius-orchestrator)
2. **Hierarchical Working Memory**: 파일시스템 기반 트리 구조. search/read/write/edit/delete 원시 연산 - [Section 2.2.1](../2512.10398.md#221-context-management)
3. **Architect Agent**: 컨텍스트 임계치 도달 시 별도 호출로 구조화된 요약 생성 - [Section 2.2.1](../2512.10398.md#221-context-management)
4. **Note-Taking Agent**: 세션 트래젝토리를 비동기적으로 Markdown 노트로 증류 - [Section 2.2.2](../2512.10398.md#222-note-taking)
5. **Extensions**: typed callback (`on_input_messages`, `on_llm_output`) 기반 모듈형 도구 - [Section 2.2.3](../2512.10398.md#223-extensions)
6. **Meta-Agent**: 자연어 스펙에서 에이전트 자동 생성 및 회귀 테스트 기반 개선 - [Section 2.3.2](../2512.10398.md#232-meta-agent-a-build-test-improve-loop)

#### Evaluation Setup

| Aspect | Value | Source |
|--------|-------|--------|
| **Benchmarks** | SWE-Bench-Pro (731 tasks), SWE-Bench-Verified (500 tasks), PyTorch-Bench (8 tasks) | [Section 3.1](../2512.10398.md#31-setup) |
| **Backbone Models** | Claude 4 Sonnet, Claude 4.5 Sonnet, Claude 4.5 Opus, GPT-5.2 | [Section 3.1](../2512.10398.md#31-setup) |
| **Baselines** | SWE-Agent, Live-SWE-Agent, OpenHands, Anthropic*, OpenAI* | [Table 1](../2512.10398.md#32-main-results-on-swe-bench-pro) |
| **Metric** | Resolve Rate (Pass@1), 3회 평균 | [Section 3.1](../2512.10398.md#31-setup) |
| **Ablation Subset** | 100 examples from SWE-Bench-Pro | [Section 3.3](../2512.10398.md#33-meta-agent-learned-tool-use) |
| **Note-Taking Eval** | 151 instances (2-pass evaluation) | [Section 3.5](../2512.10398.md#35-evaluations-on-long-term-memory) |

#### Inference

CCA는 단일 에이전트 모드로 추론한다:
1. 오케스트레이터가 LLM을 반복 호출하며 도구를 실행
2. 컨텍스트가 임계치에 도달하면 Architect Agent가 구조화된 요약 생성
3. 계층적 메모리에 중간 결과를 저장/검색
4. 이전 세션의 노트가 있으면 초기 컨텍스트에 포함
5. 액션이 없거나 최대 반복에 도달하면 종료

**Source**: [Algorithm 1](../2512.10398.md#algorithm-1), [Section 2.3.1](../2512.10398.md#231-the-confucius-orchestrator)

---

## Key Design Choices

| Choice | Decision | Rationale | Alternatives | Source |
|--------|----------|-----------|--------------|--------|
| 에이전트 아키텍처 | 단일 에이전트 + 보조 에이전트(Architect, Note-Taker) | 원래 컨텍스트 유지, 과도 엔지니어링 방지 | 멀티 에이전트 (Claude Code 방식) | [Appendix G](../2512.10398.md#g-case-studies-comparison-with-claude-code) |
| 컨텍스트 관리 | Architect Agent 기반 적응적 압축 | 의미적 보존 + 최근 메시지 원본 유지 | 고정 윈도우 절삭, RAG 기반 검색 | [Section 2.2.1](../2512.10398.md#221-context-management) |
| 메모리 구조 | 파일시스템 기반 트리 구조 | 계층적 조직화, 친숙한 인터페이스 | 벡터 DB, 그래프 메모리 | [Section 2.2.1](../2512.10398.md#221-context-management) |
| 노트 형식 | Markdown + 메타데이터 태그 | 인간/에이전트 모두 읽기 가능, 검색 용이 | JSON, 구조화 DB | [Section 2.2.2](../2512.10398.md#222-note-taking) |
| 도구 사용 | 익스텐션 기반 typed callback | 모듈성, 재사용성, 감사 가능성 | 하드코딩, 프롬프트 기반 | [Section 2.2.3](../2512.10398.md#223-extensions) |
| 에이전트 개발 | 메타 에이전트 자동 최적화 | 수동 프롬프트 엔지니어링의 스케일 한계 | 수동 튜닝, RL 기반 학습 | [Section 2.3.2](../2512.10398.md#232-meta-agent-a-build-test-improve-loop) |
| LLM 인터페이스 | 네이티브 tool-use + XML 태그 이중 지원 | 모델 호환성 극대화 | 단일 인터페이스 | [Section 2.3.1](../2512.10398.md#231-the-confucius-orchestrator) |
| 요약기 모델 | 메인 모델과 동일 | 일관성, 최고 품질 | 별도 경량 모델 (비용 절감) | [Appendix B](../2512.10398.md#b-context-summarization-ablations) |

---

## Assumptions & Limitations

### Assumptions
1. **환경 동일성 가정**: 모든 비교가 동일 실행 환경(SWE-rex)에서 수행된다고 가정. Claude Code와의 비교가 불가능한 이유 - [Section 3.2](../2512.10398.md#32-main-results-on-swe-bench-pro)
2. **벤치마크 대표성 가정**: SWE-Bench-Pro/Verified가 실제 소프트웨어 엔지니어링 태스크를 충분히 대표한다고 가정 - [Section 3.1](../2512.10398.md#31-setup)
3. **메타 에이전트 수렴 가정**: build-test-improve 루프가 유용한 개선점에 수렴한다고 가정. 수렴 조건이나 반복 횟수 미명시 - [Section 2.3.2](../2512.10398.md#232-meta-agent-a-build-test-improve-loop)

### Limitations
1. **SDK 비공개**: 핵심 기여인 Confucius SDK 자체가 비공개. SWE-Bench 평가 코드만 공개되어 재현 불가
2. **노트 테이킹 효과 미미**: +1.4% (151개 중 2개 추가 해결)는 통계적 유의성 불확실. 더 큰 규모의 실험 필요
3. **비용 분석 부재**: Architect Agent 추가 호출, 노트 테이킹 에이전트, 메타 에이전트의 API 비용 미정량화
4. **Claude Code 직접 비교 불가**: 실행 환경 차이로 정량적 비교 불가. Appendix G는 8개 이슈에서의 질적 비교만 제공
5. **메타 에이전트 투명성 부족**: 메타 에이전트가 발견/개선한 구체적 설정이 Listing 1 하나만 공개

### Scope
- **In Scope**: 에이전트 스캐폴딩 설계, 컨텍스트 관리, 장기 메모리, 자동 에이전트 개선, SWE-Bench 평가
- **Out of Scope**: 모델 학습/파인튜닝, RL 기반 에이전트 학습, 멀티모달 코딩, 비용 효율성 분석, 안전성/정렬

---

## Complexity Analysis

| Aspect | Complexity | Notes |
|--------|------------|-------|
| **평균 턴 수** | 64 turns (Run 1), 61 turns (Run 2) | Table 4, Claude 4.5 Sonnet 기준 |
| **평균 토큰 비용** | 104k (Run 1), 93k (Run 2) | 시스템 프롬프트 제외 |
| **컨텍스트 압축** | 임계치 도달 시 1회 Architect Agent 호출 | 별도 LLM 호출로 오버헤드 발생 |
| **멀티 파일 강건성** | 1-2 files: 57.8% → 10+ files: 44.4% | Table 3, 완만한 감소 |
| **Thinking Budget** | 8k: 67.3% → 32k: 68.7% | Table 6, diminishing returns |
| **노트 테이킹 범위** | 151/731 instances에서 의미 있는 노트 생성 | ~20.7%에서만 활성화 |

---

## Source References Summary

| Topic | Section | Link |
|-------|---------|------|
| Design Philosophy | Section 2.1 | [Link](../2512.10398.md#21-design-philosophy-ax-ux-and-dx) |
| Context Management | Section 2.2.1 | [Link](../2512.10398.md#221-context-management) |
| Note-Taking | Section 2.2.2 | [Link](../2512.10398.md#222-note-taking) |
| Extensions | Section 2.2.3 | [Link](../2512.10398.md#223-extensions) |
| Orchestrator | Section 2.3.1 | [Link](../2512.10398.md#231-the-confucius-orchestrator) |
| Meta-Agent | Section 2.3.2 | [Link](../2512.10398.md#232-meta-agent-a-build-test-improve-loop) |
| Main Results | Section 3.2 | [Link](../2512.10398.md#32-main-results-on-swe-bench-pro) |
| Ablations | Section 3.3-3.5 | [Link](../2512.10398.md#33-meta-agent-learned-tool-use) |
| Context Summarization | Appendix B | [Link](../2512.10398.md#b-context-summarization-ablations) |
| Claude Code Comparison | Appendix G | [Link](../2512.10398.md#g-case-studies-comparison-with-claude-code) |

---

## Navigation

← [Back to Main Analysis](../2512.10398-analysis.md)
