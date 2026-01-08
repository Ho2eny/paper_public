---
parent: "../2504.18851-analysis.md"
source: "../2504.18851.md"
type: detail
created: 2026-01-09
---

# Section Reading Priority Guide: When2Call

> **Navigation**: [Main Analysis](../2504.18851-analysis.md) | [TL;DR](./tldr.md) | [Contributions](./contributions.md) | [Methodology](./methodology.md)

## Reading Priority Matrix

| Priority | Section | Reading Time | 대상 독자 |
|----------|---------|-------------|-----------|
| **P0 (Must)** | Abstract, Section 1, Section 2.1 | 10분 | 모든 독자 |
| **P1 (Should)** | Section 3.2, 3.4, Table 3 | 15분 | 연구자, 실무자 |
| **P2 (Could)** | Section 5, 6 | 10분 | 심화 이해 필요 시 |
| **P3 (Optional)** | Section 2.2-2.4, Appendix | 20분 | 구현자, 후속 연구자 |

---

## P0: Must Read (핵심)

### Abstract
- 논문 전체 요약
- 벤치마크, 학습 방법, 결과 개요

### Section 1: Introduction ([Link](../2504.18851.md#1-introduction))
**핵심 인사이트:**
- 도구 호출의 실제 배포 문제 정의
- Figure 1: 학생 기록 vs 성적 조회 예시
- Table 1: 기존 벤치마크와의 비교

**읽어야 할 이유:**
> 논문의 motivation과 기존 연구의 gap을 명확히 이해

### Section 2.1: Tool-calling as Multiple-Choice ([Link](../2504.18851.md#21-tool-calling-as-multiple-choice))
**핵심 인사이트:**
- 4가지 선택지 정의 (a), (b), (c), (d)
- Direct answer가 항상 환각인 이유
- Multiple-choice 형식의 장점

---

## P1: Should Read (중요)

### Section 3.2: Results for Community Models ([Link](../2504.18851.md#32-results-for-community-models))
**핵심 인사이트:**
- Table 3: 주요 모델 성능 비교
- 모델 크기와 성능의 비례하지 않음
- Tool hallucination rate 분석

**Key Quote:**
> "Performance is far from the ceiling on When2Call and does not necessarily improve with model size"

### Section 3.4: Results from SFT & RPO ([Link](../2504.18851.md#34-results-from-sft--rpo))
**핵심 인사이트:**
- SFT vs RPO 성능 비교
- RPO가 BFCL 성능 유지하면서 When2Call 향상
- 4B와 8B 모델 결과 차이

**실무적 가치:** 모델 학습 시 어떤 방법을 선택해야 하는지 지침 제공

### Table 3 ([Link](../2504.18851.md#table-3))
논문의 핵심 결과 테이블. 반드시 상세히 분석해야 함.

---

## P2: Could Read (심화)

### Section 5: Discussion ([Link](../2504.18851.md#5-discussion))
**핵심 인사이트:**
- "Improving when-to-call accuracy is not trivial"
- BFCL Irrelevance와 When2Call의 차이
- Follow-up question 성능 분석
- 모델별 오류 패턴

**Figure 2 분석:**
BFCL Irrelevance와 When2Call 성능의 상관관계

### Section 6: Related Work ([Link](../2504.18851.md#6-related-work))
**핵심 인사이트:**
- 도구 호출 벤치마크 발전 역사
- BFCL, ToolSandbox, ToolBeHonest 비교
- 공개 학습 데이터셋 현황

---

## P3: Optional (구현/후속연구)

### Section 2.2: Data Generation ([Link](../2504.18851.md#22-data-generation))
- Mixtral 8x22B 기반 합성 데이터 생성
- 2단계 파이프라인 상세

**읽어야 할 경우:** 유사한 벤치마크 생성 시

### Section 2.3-2.4: Statistics & Difficulty ([Link](../2504.18851.md#23-statistics))
- Table 2: 데이터셋 통계
- 난이도 설계 철학

**읽어야 할 경우:** 데이터셋 특성 심층 분석 시

### Section 3.1: Model-Specific Prompt Templates ([Link](../2504.18851.md#31-model-specific-prompt-templates))
- 모델별 시스템 프롬프트 처리
- LM Evaluation Harness 통합

**읽어야 할 경우:** 평가 코드 재현 시

### Section 4: LLM-as-Judge Evaluation ([Link](../2504.18851.md#4-llm-as-judge-evaluation))
- GPT-4-Turbo 기반 평가
- Table 4, 5: 폐쇄형 모델 결과

**읽어야 할 경우:** 폐쇄형 모델 평가 필요 시

### Appendix A, C, E
- A: 합성 데이터 품질 체크리스트
- C: 프롬프트 템플릿 전문
- E: Confusion matrix 상세

**읽어야 할 경우:** 재현 실험 또는 확장 연구 시

---

## Skip Recommendations

### 첫 번째 읽기에서 스킵 가능
- Appendix B: 시스템 프롬프트 예시
- Appendix D: 전체 모델 결과 (Table 3의 확장)
- Limitations 섹션 (주요 내용 TL;DR에 포함)

### 특정 목적이 아니면 스킵
- Section 3.3.1-3.3.2: 학습 하이퍼파라미터 상세 (재현 시에만)
- Table 6-11: 프롬프트 및 품질 체크 상세

---

## Quick Reading Path by Goal

### Goal 1: 벤치마크 이해 (15분)
```
Abstract → Section 1 → Section 2.1 → Table 1, 3
```

### Goal 2: 모델 성능 비교 (20분)
```
Section 3.2 → Table 3 → Section 4.2 → Table 4
```

### Goal 3: 학습 방법 적용 (25분)
```
Section 3.3 → Section 3.4 → Section 5 (RPO 부분)
```

### Goal 4: 벤치마크 재현/확장 (40분+)
```
Section 2.2-2.4 → Section 3.1 → Appendix A, C, E
```

---

## Source References

모든 섹션 링크는 [원본 논문](../2504.18851.md)을 참조합니다.
