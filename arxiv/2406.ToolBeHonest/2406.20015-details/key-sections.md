---
parent: ../2406.20015-analysis.md
source: ../2406.20015.md
---

# Key vs Non-Key Sections - Deep Analysis

## ⭐⭐⭐ Must Read

### Section 3.2: Multi-level Diagnostic Task

- **Why**: 벤치마크의 **핵심 혁신** - 3단계 진단 프레임워크
- **Key Points**:
  - L1 Solvability Detection: Binary classification (EM metric)
  - L2 Solution Planning: Sequential tool selection (PR metric)
  - L3 Missing-tool Analysis: Functionality description (PR + MS)
  - UnsolvableQuery 특수 도구 활용
- **Time**: ~15분
- **Link**: [Section 3.2](../2406.20015.md#32-in-depth-multi-level-diagnostic-task)

### Section 4.3 + Table 2: Main Results

- **Why**: **모델별 성능 비교**의 핵심 레퍼런스
- **Key Points**:
  - Gemini-1.5-Pro 최고 (45.3) but 여전히 절반 이하
  - L1→L3 급락 (특히 LFT 시나리오)
  - Open-weight vs Proprietary 격차
- **Time**: ~15분
- **Link**: [Table 2](../2406.20015.md#table-2-the-toolbh-leaderboard-with-14-llms)

### Section 5.2: Error Analysis + Figure 4

- **Why**: **5가지 에러 유형** 분류로 개선 방향 도출
- **Key Points**:
  - Solvability hallucination이 가장 빈번
  - Proprietary: Wrong tool reasoning 많음
  - Open-weight: Non-existent tools, Solvability hallucination
- **Time**: ~10분
- **Link**: [Section 5.2](../2406.20015.md#52-error-analysis)

---

## ⭐⭐ Important

### Section 3.3: Hallucination-inducing Tasks

- **Why**: **3가지 시나리오** 이해로 태스크 구조 파악
- **Key Points**:
  - MNT: 필수 도구 누락 (Single/Multi-step)
  - PT: OS/Web 환경 잠재 도구 유도
  - LFT: 도구 기능 제한 (Iterative, Optimal Selection)
- **Time**: ~10분
- **Link**: [Section 3.3](../2406.20015.md#33-in-breadth-tool-based-hallucination-inducing-tasks)

### Section 5.3: Solvable vs Unsolvable

- **Why**: **Proprietary vs Open-weight 격차**의 실제 원인
- **Key Points**:
  - Solvable: Open-weight도 91.1% 수준
  - Unsolvable: 39.4%로 격차 폭발
  - 학습 데이터 차이 시사
- **Time**: ~8분
- **Link**: [Section 5.3](../2406.20015.md#53-solvable-tasks-versus-corresponding-unsolvable-tasks)

### Section 5.1: Depth and Breadth Perspective + Figure 3

- **Why**: **Top 3 모델 비교**로 세부 강약점 파악
- **Key Points**:
  - GPT-4-0613: L1 우수, L2/L3 급락
  - Gemini-1.5-Pro: L1→L2→L3 독특한 패턴 (L3 반등)
  - 모든 모델 LFT 시나리오에서 고전
- **Time**: ~8분
- **Link**: [Section 5.1](../2406.20015.md#51-model-performance-a-depth-and-breadth-perspective)

### Section 3.1: Design Philosophy

- **Why**: 벤치마크 **설계 원칙** 이해
- **Key Points**:
  - Unsolvability in real-world tasks
  - Multi-level diagnosis 필요성
  - Hallucination-inducing scenarios 근거
- **Time**: ~5분
- **Link**: [Section 3.1](../2406.20015.md#31-design-philosophy)

---

## ⭐ Reference (필요시 참조)

### Section 2: Related Work

- **When to read**: 기존 tool-use 및 hallucination 벤치마크와의 차별점 이해시
- **Content**: AgentBench, ToolBench, MetaTool, TruthfulQA, HaluEval 등 비교
- **Link**: [Section 2](../2406.20015.md#2-related-work)

### Section 3.4: Data Curation

- **When to read**: 데이터 생성 파이프라인 상세 필요시
- **Content**: Seed sample → Synthesis → Filtering → Construction
- **Link**: [Section 3.4](../2406.20015.md#34-data-curation)

### Section 3.5: General Statistics + Table 1

- **When to read**: 데이터셋 통계 확인 필요시
- **Content**: 평균 쿼리 길이, 도구 수, 샘플 수 등
- **Link**: [Section 3.5](../2406.20015.md#35-general-statistics)

---

## Skip (건너뛰기 가능)

### Appendix A.2: Filtering Criteria

- **Why skip**: 데이터 필터링 상세, 결과 해석에 불필요
- **When needed**: 유사 데이터셋 구축시 참조
- **Link**: [Appendix A.2](../2406.20015.md#a2-filtering-criteria)

### Appendix A.3-A.4: Generation/Evaluation Prompts

- **Why skip**: 프롬프트 템플릿 상세, 핵심 이해에 불필요
- **When needed**: 자체 평가 환경 구축시
- **Link**: [Appendix A.3](../2406.20015.md#a3-generation-prompts)

### Appendix D: Response Length Impact

- **Why skip**: 응답 길이와 성능 상관관계, 부차적 분석
- **When needed**: 모델 출력 최적화 연구시
- **Link**: [Appendix D](../2406.20015.md#d-additional-experiments-and-result-analysis)

### Appendix E: Case Studies

- **Why skip**: 개별 사례 분석, 주요 결론은 본문에 포함
- **When needed**: 특정 에러 패턴 상세 분석시
- **Link**: [Appendix E](../2406.20015.md#e-case-study)

---

## Reading Path Recommendations

### Quick Understanding (25분)
1. Section 3.2 (3단계 프레임워크) → 2. Table 2 (결과) → 3. Figure 4 (에러 분석)

### Deep Dive (50분)
위 + Section 3.3 (시나리오) + Section 5.1 (Top 3 분석) + Section 5.3 (Solvable vs Unsolvable)

### Full Understanding (1.5시간)
위 + Section 3.1 (설계 철학) + Section 2 (Related Work) + Appendix E (Case Studies)
