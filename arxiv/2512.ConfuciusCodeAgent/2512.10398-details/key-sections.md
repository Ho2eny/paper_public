---
parent: ../2512.10398-analysis.md
source: ../2512.10398.md
created: 2026-02-06
---

# Key vs Non-Key Sections - Deep Analysis

## ⭐⭐⭐ Must Read

### Section 2: Method (전체)
- **Why**: 논문의 핵심 기여가 모두 이 섹션에 집중. AX/UX/DX 설계 철학(2.1), 컨텍스트 관리(2.2.1), 노트 테이킹(2.2.2), 익스텐션(2.2.3), 오케스트레이터(2.3.1), 메타 에이전트(2.3.2)
- **Key Points**:
  - AX vs UX 구체적 예시 (코드 블록): 사용자는 풍부한 diff를, 에이전트는 압축된 결과만 받음
  - 계층적 메모리의 트리 구조 예시: 실제 SWE-Bench-Pro 인스턴스에서의 메모리 계층
  - Architect Agent의 구조화된 요약: 목표/결정/에러/TODO 카테고리 보존
  - hindsight notes: 실패 사례를 에러 메시지, 스택 트레이스, 컴포넌트별로 인덱싱
  - 익스텐션의 typed callback 패턴: `on_input_messages`, `on_llm_output`
  - Algorithm 1: 최소한의 오케스트레이터 루프
  - 메타 에이전트의 build-test-improve 자동화
- **Link**: [Section 2](../2512.10398.md#2-method)

### Section 3.2: Main Results on SWE-Bench-Pro
- **Why**: Table 1의 핵심 결과와 "스캐폴딩 > 모델" 주장의 근거
- **Key Points**:
  - 4개 백본 모델 × 4개 스캐폴딩에서 일관된 CCA 우위
  - 교차 결과: Sonnet+CCA > Opus+Anthropic
  - Claude Code와의 비교 불가능성 설명 (SWE-rex 호환성 문제)
- **Link**: [Section 3.2](../2512.10398.md#32-main-results-on-swe-bench-pro)

### Section 3.3-3.5: Ablation Studies
- **Why**: 각 메커니즘의 개별 기여를 분리하는 핵심 실험
- **Key Points**:
  - 3.3: 메타 에이전트 도구 사용 ablation (+7%), Listing 1의 file-edit 매처 예시
  - 3.4.1: 컨텍스트 관리 ablation (+6.6%), 멀티 파일 편집 강건성 (Table 3)
  - 3.5: 노트 테이킹 효과 (턴 -3, 토큰 -11k, 해결률 +1.4%)
- **Link**: [Section 3.3](../2512.10398.md#33-meta-agent-learned-tool-use), [Section 3.4](../2512.10398.md#34-evaluations-on-long-context-reasoning), [Section 3.5](../2512.10398.md#35-evaluations-on-long-term-memory)

---

## ⭐⭐ Important

### Section 1: Introduction
- **Why**: C1(장기 컨텍스트 추론)과 C2(장기 메모리) 챌린지 프레임워크가 전체 논문의 구조를 결정
- **Key Points**:
  - 연구용 vs 상용 에이전트의 격차 분석
  - AX/UX/DX가 왜 별도 축이어야 하는지의 동기
  - 4가지 메커니즘의 C1/C2 매핑
- **Link**: [Section 1](../2512.10398.md#1-introduction)

### Section 3.6: SWE-Bench-Verified Results
- **Why**: 오픈소스 스캐폴딩(OpenHands)과의 직접 비교
- **Key Points**:
  - Claude 4 Sonnet으로 74.6% (OpenHands 72.8% 초과)
  - mini-SWE-Agent + Claude 4.5 Sonnet (70.6%)보다 약한 모델로도 우수
- **Link**: [Section 3.6](../2512.10398.md#36-comparison-with-open-sourced-scaffolds-on-swe-bench-verified)

### Appendix B: Context Summarization Ablations
- **Why**: 요약 품질이 성능에 미치는 영향의 구체적 증거
- **Key Points**:
  - Haiku 3.5 vs Sonnet 4 요약기: 22/50 vs 26/50
  - 구체적 요약 비교: Haiku는 키워드 손실, Sonnet은 작업 phase 보존
  - 동일 형식/길이에서 요약 "품질"만이 차이
- **Link**: [Appendix B](../2512.10398.md#b-context-summarization-ablations)

### Appendix G: Claude Code 비교 사례
- **Why**: 단일 에이전트(CCA) vs 멀티 에이전트(CC)의 실질적 행동 차이 분석
- **Key Points**:
  - 3개 PyTorch 이슈에서의 비교: CUDA 체크포인트, 메모리 할당, 정밀도 회귀
  - CCA: 최소 개입, CC: 포괄적이지만 과도 엔지니어링
  - 서브에이전트의 맥락 부재가 과도 분석/복잡한 솔루션으로 이어짐
  - "잘 정의된 디버깅에서는 위임보다 직접 탐색이 나을 수 있다"
- **Link**: [Appendix G](../2512.10398.md#g-case-studies-comparison-with-claude-code)

---

## ⭐ Reference

### Appendix E: Example Execution Trace
- **Why**: CCA의 실제 실행 흐름을 이해하는 데 유용
- **Key Points**: SWE-Bench-Pro 인스턴스에서의 전체 실행 트레이스 (탐색 → 분석 → 수정 → 검증)
- **Link**: [Appendix E](../2512.10398.md#e-example-execution-trace)

### Appendix F: Note-Taking Examples
- **Why**: hindsight notes의 실제 형태와 품질을 확인
- **Key Points**: 프로젝트별/공유 노트 계층, 와일드카드 이스케이핑 사례, 빈 문자열 엣지 케이스
- **Link**: [Appendix F](../2512.10398.md#f-example-notes-for-long-term-memory)

### Section 4: Related Work
- **Why**: 코딩 에이전트 생태계의 빠른 개관
- **Key Points**: SWE-Agent, Live-SWE-Agent, Satori-SWE, Agentless, OpenHands의 위치 정리
- **Link**: [Section 4](../2512.10398.md#4-related-work-on-coding-agents)

### Appendix C: Meta-Agent Details
- **Why**: 메타 에이전트의 구현 세부사항과 개발 사이클
- **Key Points**: 자연어 스펙 → 구조화된 설정 폼 → 자동 생성 → 회귀 테스트 → 개선 루프
- **Link**: [Appendix C](../2512.10398.md#c-meta-agent)

---

## Skip

### Appendix A: Additional Related Work
- **Why**: 표준적인 관련 연구 확장. 벤치마킹, 대규모 SE, LLM 학습 등 일반적 내용
- **Link**: [Appendix A](../2512.10398.md#a-additional-related-work)

### Appendix D: Thinking Budget Scaling
- **Why**: 3개 설정(8k/16k/32k)에서의 미미한 차이(67.3→68.7). Diminishing returns 관찰 외 새로운 통찰 없음
- **Link**: [Appendix D](../2512.10398.md#d-thinking-budget-scaling)

### Appendix H: Future Work
- **Why**: RL 통합 가능성에 대한 전망만 제시. 구체적 실험이나 결과 없음
- **Link**: [Appendix H](../2512.10398.md#h-future-work)

---

## Reading Order Recommendation

1. **Section 1** → 문제 프레임워크 이해 (10분)
2. **Section 2.1** → AX/UX/DX 설계 철학 (5분)
3. **Section 2.2.1-2.2.2** → 컨텍스트 관리 + 노트 테이킹 (15분)
4. **Section 3.2** → 메인 결과 (5분)
5. **Section 3.3-3.5** → Ablation 결과 (10분)
6. **Appendix G** → Claude Code 비교 (15분, 선택적)
7. **Appendix B** → 요약 품질 ablation (10분, 선택적)

**총 필수 읽기 시간**: ~45분
**전체 읽기 시간**: ~70분

---

## Navigation

← [Back to Main Analysis](../2512.10398-analysis.md)
