---
parent: "../2404.09992-analysis.md"
source: "../2404.09992.md"
type: detail
created: 2026-01-09
---

# Section Reading Priority Guide

[Back to Main Analysis](../2404.09992-analysis.md)

---

## Reading Priority Matrix

| Priority | Section | Time | Why Read |
|----------|---------|------|----------|
| P0 | Abstract + Figure 1 | 3 min | 핵심 아이디어와 벤치마크 구조 파악 |
| P0 | Table 2 | 5 min | 모든 에이전트의 성능 비교 |
| P1 | Section 4.3 | 5 min | 멀티홉 실패 원인 분석 |
| P1 | Section 4.4 | 5 min | 메모리 증강 해결책 |
| P2 | Section 3.1-3.5 | 10 min | 벤치마크 구성 상세 |
| P2 | Table 1 | 3 min | 다른 벤치마크와 비교 |
| P3 | Section 2 | 3 min | 관련 연구 개요 |
| P3 | Appendix C | 10 min | 확장된 관련 연구 |
| Skip | Appendix A, B | - | 세부 구현/데이터 통계 |

---

## Section-by-Section Guide

### Abstract + Figure 1 (P0: Must Read)

**핵심 내용**:
- 벤치마크의 3가지 특성: evolving websites, multihop, holistic evaluation
- 1,050 tasks, 14 websites, 2.85 avg hops
- Human 96.3% vs GPT-4V 21.8%

**Figure 1 해석**:
- 예시 태스크의 멀티홉 구조 시각화
- Wikipedia -> Flight -> Hotel -> Event 순차 이동

Source: [Abstract](../2404.09992.md#abstract), [Figure 1](../2404.09992.md#figure-1)

---

### Section 1: Introduction (P1: Recommended)

**읽어야 할 부분**:
- 첫 3 문단: 문제 정의 (single-hop 한계, multimodal 부재)
- 마지막 bullet points: 3가지 핵심 기여

**건너뛸 수 있는 부분**:
- 중간의 배경 설명 (일반적인 embodied agent 소개)

---

### Section 2: Related Works (P3: Optional)

**요약**:
- Agent Benchmarks: API-Bank, BOLAA, OpenAGI, GAIA 등
- Web Agents: WebShop, Mind2Web, WebArena, VisualWebArena

**Appendix C와의 관계**:
- Section 2는 핵심만, Appendix C에 상세 분석
- GUI Agents (Web + Mobile) 추가 다룸

---

### Section 3: MMInA Benchmark (P2: Important)

#### 3.1 Environment
**MDP 정식화**: $\langle S, A, P, R \rangle$
- State, Action, Transition, Reward 정의
- Accessibility tree + screenshot 기반 observation

#### 3.2 Dataset Construction
- 1,050 tasks, 2,989 hops
- GPT-4V 기반 질문 생성 + 수동 검증
- "Minimalist" 주석 프로토콜

#### 3.3 Multimodal Web Content
- 이미지+텍스트 필수 처리
- VWA와의 차별점: 모든 태스크가 멀티모달 필수

#### 3.4 Multihop Cross-website Browsing
- 14개 도메인 (쇼핑, 여행, 정보검색 등)
- hop 완료 시 자동으로 다음 사이트로 이동

#### 3.5 Evaluation
**핵심 읽기 포인트**:
- must_include vs fuzzy_match
- N-hop 평가 queue 메커니즘

---

### Section 4: Experiments (P0-P1: Critical)

#### 4.1 Baselines (P2)
3가지 에이전트 유형:
1. LLM/LMM as backbone (GPT-4, Gemini, DeepSeek-R1)
2. Heuristic-based (WebShop, CogAgent)
3. Human baseline (96.25% success)

#### 4.2 Main Results (P0)
**Table 2 핵심 인사이트**:
- Multimodal > Text-only
- 긴 context window 중요
- 모든 AI << Human (74%+ gap)

#### 4.3 Why Multihop is Challenging (P1)
**반드시 읽어야 함**:
- Search space 확장 문제
- Early hop failure pattern (Table 3)
- Input length와 성능의 비선형 관계

#### 4.4 Memory-augmented Agents (P1)
**핵심 읽기 포인트**:
- 3가지 메모리 타입 (Semantic, Episodic, Procedural)
- K=1-2가 최적인 이유
- Model-agnostic 특성

---

### Section 5: Conclusion (P2)

**3줄 요약**:
1. 1,050 multimodal multihop tasks on 14 real websites
2. Novel holistic evaluation (hop-level + task-level)
3. Memory augmentation improves performance

---

### Section 6-7: Limitations & Ethics (P3: Optional)

**Limitations**:
- 웹 보안으로 이미지 추출 제한
- 일부 오프라인 웹사이트 사용

**Ethics**:
- Base model bias 주의 필요

---

## Visual Elements Priority

| Priority | Element | Key Insight |
|----------|---------|-------------|
| P0 | Figure 1 | 멀티홉 태스크 구조 |
| P0 | Table 2 | 모든 모델 성능 비교 |
| P0 | Table 3 | Early hop failure pattern |
| P1 | Figure 2 | 데이터 분포 (도메인, 인텐트, hop 수) |
| P1 | Figure 4 | 메모리 시스템 아키텍처 |
| P2 | Figure 3 | Hop 수 vs 액션 수 관계 |
| P2 | Figure 5 | K값에 따른 성능 변화 |
| P3 | Table 1 | 벤치마크 비교 |
| P3 | Table A1-A3 | 상세 실험 결과 |

---

## Quick Reading Path (15분)

1. **Abstract** (2분): 핵심 주장 파악
2. **Figure 1** (1분): 태스크 구조 이해
3. **Table 2** (3분): 성능 결과 확인
4. **Section 4.3** (4분): 실패 원인 분석
5. **Section 4.4** (3분): 해결책 이해
6. **Section 5** (2분): 결론 확인

---

## Deep Reading Path (45분)

1. Quick Reading Path (15분)
2. **Section 3 전체** (15분): 벤치마크 구성 상세
3. **Table 1 + Table 3** (5분): 비교 및 상세 분석
4. **Appendix C** (10분): 관련 연구 확장

[Back to Main Analysis](../2404.09992-analysis.md) | [Methodology Detail](methodology.md)
