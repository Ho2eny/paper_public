---
parent: "../2311.12983-analysis.md"
source: "../2311.12983.md"
type: detail
created: 2026-01-09
---

# GAIA: Section Reading Priority Guide

[Back to Main Analysis](../2311.12983-analysis.md) | [Original Paper](../2311.12983.md)

---

## Reading Priority Matrix

| Priority | Section | Pages | Why Read | Skip If |
|----------|---------|-------|----------|---------|
| **P1** | 1. Introduction | 1-2 | 핵심 동기와 철학 | - |
| **P1** | 3.1 Design | 3-4 | 4가지 설계 원칙 | - |
| **P1** | 4. Results | 5-6 | 주요 실험 결과 | - |
| **P2** | 3.3 Composition | 4-5 | 데이터셋 구성 이해 | 구현 안 할 경우 |
| **P2** | 5. Discussion | 6-7 | 시사점과 미래 방향 | 빠른 이해만 필요시 |
| **P3** | 2. Related Work | 2-3 | 관련 연구 맥락 | 배경 지식 있을 경우 |
| **P3** | 3.4 Building | 5 | 질문 생성 방법론 | 확장 계획 없을 경우 |
| **P3** | 6. Limitations | 7-8 | 한계점 파악 | 깊은 분석 불필요시 |
| **P4** | Appendix A | - | LLM 어시스턴트 확장 | 기본 이해만 필요시 |
| **P4** | Appendix C | - | 상세 데이터 분포 | 통계 분석 불필요시 |
| **P4** | Appendix D | - | 어노테이션 세부사항 | 재현 안 할 경우 |

---

## 읽기 목적별 가이드

### 빠른 이해 (15분)
1. **Abstract** - 전체 요약
2. **Figure 1** - 질문 예시 (Level 1, 2, 3)
3. **Figure 4** - 핵심 결과 (성능 비교)
4. **Section 3.1 첫 2문단** - 설계 철학

### 연구자/개발자 (45분)
위 내용 +
5. **Section 3.2** - 평가 방법론
6. **Section 3.3** - 능력 요구사항과 난이도 정의
7. **Section 4 전체** - 상세 결과 분석
8. **Figure 5** - 능력별 성능 분석

### 벤치마크 확장/재현 (90분)
위 내용 +
9. **Section 3.4** - 질문 생성 프로세스
10. **Section 6** - 한계점과 주의사항
11. **Appendix D** - 상세 가이드라인과 예시
12. **Table 1, 2, 3** - 어노테이션 형식과 통계

---

## Section별 상세 분석

### Section 1: Introduction [Must Read]

**핵심 메시지:**
- 기존 벤치마크의 한계: 빠른 포화, 데이터 오염, 평가 어려움
- 제안: "개념적으로 단순하지만 정확한 실행이 필요한 과제"
- 목표: t-AGI 진척도 측정 가능한 벤치마크

**Key Quotes:**
> "Tasks that are difficult for humans are not necessarily difficult for recent systems"

> "GAIA proposes real-world questions that require a set of fundamental abilities"

**읽기 시간:** 5분

---

### Section 2: Related Work [Optional]

**다루는 내용:**
- LLM 평가의 역사 (GLUE -> SuperGLUE -> MMLU)
- 범용 어시스턴트 평가 (AgentBench, ToolQA, APIBench)
- GAIA의 차별점

**Skip 조건:** 벤치마크 역사에 익숙한 경우

**읽기 시간:** 8분

---

### Section 3.1: Design [Must Read]

**4가지 핵심 원칙:**
1. Real-world and challenging questions
2. Easy interpretability
3. Non-gameability
4. Simplicity of use

**System Prompt 예시 포함** - 평가 재현에 필수

**읽기 시간:** 7분

---

### Section 3.2: Evaluation [Recommended]

**평가 방식:**
- Quasi-exact match (정규화된 매칭)
- 답변 형식: 숫자, 단어, 콤마 구분 리스트
- Zero-shot 평가

**읽기 시간:** 3분

---

### Section 3.3: Composition [Recommended]

**난이도 정의:**
- Level 1: 도구 0-1개, 5단계 이하
- Level 2: 5-10단계, 다중 도구
- Level 3: 임의 길이, 실세계 접근

**능력 분포 (Figure 3):**
- 웹 브라우징이 가장 많이 요구됨
- 멀티모달, 코딩은 상대적으로 적음

**읽기 시간:** 5분

---

### Section 3.4: Building GAIA [Conditional]

**질문 생성 프로세스:**
1. 시드 질문 제공
2. 가이드라인 기반 확장
3. 2명의 독립 검증자 확인
4. 합의 시 채택, 불일치 시 수정/제거

**웹 기반 질문의 도전:**
- 시간에 따른 변화 (Wikipedia 업데이트)
- 봇 접근 제한 (robots.txt)

**읽기 시간:** 7분

---

### Section 4: Results [Must Read]

**핵심 결과:**
- Human: 92% vs GPT-4+plugins: ~15%
- 도구 통합의 효과: 9.1% -> 30.3% (Level 1)
- Level 3는 모든 모델이 0%

**Figure 4, 5 필수 확인**

**읽기 시간:** 8분

---

### Section 5: Discussion [Recommended]

**주요 논점:**
- 재현성 문제 (API 변화, 플러그인 가용성)
- 정적 vs 동적 벤치마크
- 부분 자동화 vs 완전 자동화
- 통합 평가의 미래

**읽기 시간:** 6분

---

### Section 6: Limitations [Conditional]

**주요 한계:**
- 추론 과정 평가 없음
- 질문 생성 비용 높음
- 영어 전용 (언어/문화 다양성 부족)

**읽기 시간:** 4분

---

## Figure Priority

| Priority | Figure | Key Insight |
|----------|--------|-------------|
| **1** | Figure 1 | 난이도별 질문 예시 |
| **1** | Figure 4 | 모델별 성능 비교 |
| **2** | Figure 3 | 능력/난이도 분포 |
| **2** | Figure 5 | 능력별 성능 분석 |
| **3** | Figure 2 | GPT-4 추론 예시 |
| **3** | Figure 6 | 파일 형식 분포 |
| **4** | Figure 7-11 | 상세 분석 예시 |
