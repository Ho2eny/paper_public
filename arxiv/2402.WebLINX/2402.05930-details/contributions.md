---
parent: ../2402.05930-analysis.md
source: ../2402.05930.md
created: 2026-01-09
---

# Core Contributions - Deep Analysis

## Overview

| # | Contribution | Impact | Source |
|---|--------------|--------|--------|
| 1 | Conversational Web Navigation Benchmark | 새로운 연구 방향 정의 | [Section 3](#contribution-1-conversational-web-navigation-benchmark) |
| 2 | Multi-dimensional Generalization Evaluation | 실제 배포 가능성 검증 프레임워크 | [Section 3](#contribution-2-multi-dimensional-generalization-evaluation) |
| 3 | Dense Markup Ranking (DMR) | 실시간 에이전트 구현 가능 | [Section 5.1](#contribution-3-dense-markup-ranking-dmr) |
| 4 | Generalization Gap Discovery | 향후 연구 방향 명확화 | [Section 6](#contribution-4-generalization-gap-discovery) |

---

## Contribution 1: Conversational Web Navigation Benchmark

### What
최초의 대규모 **대화형 웹 네비게이션** 벤치마크. 사용자(Instructor)와 에이전트(Navigator)가 채팅 인터페이스를 통해 상호작용하며 실제 웹사이트에서 태스크를 완수하는 시나리오.

### Technical Details
- **규모**: 2,337개 demonstration, 100K+ 상호작용
- **웹사이트**: 155개 실제 웹사이트 (15개 지역)
- **카테고리**: 8개 카테고리, 50개 서브카테고리
- **복잡도**: 평균 43턴/데모, 평균 1,775개 HTML 요소/페이지
- **상태 구성**: DOM tree (d_t), 스크린샷 (i_t), 발화 (u_t), 히스토리 (h_t), viewport (v_t)
- **액션 공간**: 5가지 핵심 (click, load, say, submit, textinput) + 8가지 확장

### Why It Matters
- 학술적: **대화형 웹 에이전트**라는 새로운 문제 정의, 기존 단일 지시문 기반 연구의 한계 극복
- 실용적: 시각 장애인 지원, 음성 제어, 지식 근로자 생산성 향상 등 실제 사용 시나리오 반영

### Comparison to Prior Work

| Aspect | WEBLINX | Mind2Web | MiniWoB++ | RUSS |
|--------|---------|----------|-----------|------|
| Conversational | ✓ | ✗ | ✗ | ✓ |
| Real-world | ✓ | ✓ | ✗ | ✓ |
| General-purpose | ✓ | ✓ | ✗ | ✗ |
| Avg. Turns | 43.0 | 7.3 | 3.6 | 5.4 |
| HTML Elements | 1,775 | 1,135 | 28 | - |

### Source References
- Definition: [Section 3](../2402.05930.md#3-weblinx)
- Data Statistics: [Table 1](../2402.05930.md#table-1-weblinx-is-the-first-benchmark)
- Action Space: [Table 3](../2402.05930.md#table-3-actions-in-weblinx)
- Collection: [Appendix A.5](../2402.05930.md#a5-data-collection)

---

## Contribution 2: Multi-dimensional Generalization Evaluation

### What
단순 IID/OOD를 넘어 **웹사이트/카테고리/지역/접근성** 4가지 차원으로 일반화 능력을 평가하는 5개 split 설계.

### Technical Details
| Split | # Demos | 일반화 차원 | 예시 |
|-------|---------|------------|------|
| TEST-IID | 100 | 동일 분포 | 학습 웹사이트와 동일 |
| TEST-WEB | 211 | 새 웹사이트 (동일 서브카테고리) | 새 레스토랑 예약 사이트 |
| TEST-CAT | 223 | 새 서브카테고리 | 레스토랑 → 병원 예약 |
| TEST-GEO | 290 | 새 지역 | 미국 → 유럽 웹사이트 |
| TEST-VIS | 444 | 시각 정보 없음 | Instructor가 화면 못 봄 |

### Why It Matters
- 학술적: 모델의 **실제 배포 가능성** 정밀 평가 가능, 어떤 차원에서 일반화가 어려운지 식별
- 실용적: 새로운 도메인 확장 시 예상 성능 하락 정도 추정 가능

### Comparison to Prior Work

| Aspect | WEBLINX | Mind2Web | WebArena |
|--------|---------|----------|----------|
| Split 수 | 5 | 3 (Cross-*) | 1 |
| 지역 일반화 | ✓ | ✗ | ✗ |
| 접근성 테스트 | ✓ | ✗ | ✗ |
| 서브카테고리 분리 | ✓ | 부분적 | ✗ |

### Source References
- Split Definition: [Table 2](../2402.05930.md#table-2-demonstration-splits)
- Split Statistics: [Section 3.3](../2402.05930.md#33-evaluation-splits)

---

## Contribution 3: Dense Markup Ranking (DMR)

### What
Dual-encoder 기반 효율적 HTML 요소 후보 선택 방법. Cross-encoder 대비 **5배 속도 향상** (916ms → 186ms).

### Technical Details
- **아키텍처**: MiniLM-L6-v2 dual-encoder
- **학습 목표**: Cosine Similarity Loss
  ```
  L = (1/N) Σ (sim_cos(E(P_DMR(s_t)), E(c_t,i)) - y(c_t,i))²
  ```
- **입력 표현 (OTR)**:
  - HTML 속성/값 포함
  - Viewport 크기 명시
  - XPath + bounding box 좌표
- **Truncation 전략**:
  - DOM tree: 700 토큰
  - 발화: 40 토큰/발화
  - 후보: 65 토큰/후보
  - 총 목표: 2,048 토큰

### Why It Matters
- 학술적: HTML 처리 효율화 방법론 제시, retrieval 기반 요소 선택의 가능성 입증
- 실용적: **실시간 에이전트 구축** 가능, 사용자 경험 향상

### Comparison to Prior Work

| Aspect | DMR (This) | Mind2Web (DeBERTa) |
|--------|------------|-------------------|
| Architecture | Dual-encoder | Cross-encoder |
| Complexity | O(\|s\|² + \|c\|²) | O((\|s\| + \|c\|)²) |
| Speed | 186ms | 916ms |
| ID Recall@10 | 74.27 | 76.86 |
| OOD Recall@10 | 51.87 | 54.78 |

### Source References
- Method: [Section 5.1](../2402.05930.md#51-dense-markup-ranking)
- Technical Details: [Appendix B.4](../2402.05930.md#b4-technical-aspects)
- OTR Format: [Appendix B.1](../2402.05930.md#b1-optimal-text-representation)
- Truncation: [Appendix B.2](../2402.05930.md#b2-strategic-truncation)

---

## Contribution 4: Generalization Gap Discovery

### What
Fine-tuned 모델의 **심각한 일반화 실패** 발견. IID 37.0% → OOD 24-27% (30-35% 성능 하락).

### Technical Details
**핵심 발견**:

| Finding | Evidence |
|---------|----------|
| Fine-tuned > Zero-shot | Sheared-LLaMA 2.7B (25.0) > GPT-4V (10.4) |
| 모델 크기 효과 미미 | LLaMA-13B (25.2) ≈ Sheared-LLaMA 2.7B (25.0) |
| Text-only > Multimodal | LLaMA-2 > Fuyu-8B |
| TEST-CAT 가장 어려움 | 새 서브카테고리로의 전이가 가장 힘듦 |
| Intent는 유지 | Intent Match 높음, Element 선택이 문제 |

**GPT-4V 질적 실패 패턴**:
1. 시간 정보 오해 (C1): 4:15AM 대신 3:30AM 클릭
2. 상황 인식 부족 (C2): 이미 열린 창 다시 열려 함
3. 필드 혼동 (T2): 날짜 필드에 다른 정보 입력
4. 불필요한 거부 (S2): "I'm sorry, but I can't assist"

### Why It Matters
- 학술적: 웹 에이전트 **일반화 연구의 필요성** 명확히 규명, 향후 연구 방향 제시
- 실용적: 새로운 도메인 배포 시 **성능 하락 예상** 가능, 추가 학습 데이터 필요성 인식

### Source References
- Main Results: [Table 4](../2402.05930.md#table-4-aggregated-results)
- OOD Analysis: [Table 5](../2402.05930.md#table-5-results-on-out-of-domain-splits)
- Qualitative: [Section 6.2](../2402.05930.md#62-qualitative-assessment)
- Figure: [Figure 4](../2402.05930.md#figure-4-comparison-of-gpt-4v-and-llama-2-13b)

---

## Contribution Analysis

### Novelty Assessment

| Contribution | Novelty Level | Justification |
|--------------|---------------|---------------|
| 1. Benchmark | High | 최초의 대규모 대화형 실제 웹 벤치마크 |
| 2. Evaluation Splits | High | 다차원 일반화 평가 최초 제안 |
| 3. DMR | Medium | Dual-encoder의 웹 도메인 적용 |
| 4. Findings | High | Fine-tuning vs Zero-shot 트레이드오프 최초 체계적 분석 |

### Impact Assessment

| Contribution | Short-term Impact | Long-term Impact |
|--------------|-------------------|------------------|
| 1. Benchmark | 후속 연구의 평가 기준 | 대화형 웹 에이전트 연구 활성화 |
| 2. Evaluation | 일반화 평가 표준화 | 다른 도메인으로 프레임워크 확산 |
| 3. DMR | 실시간 에이전트 구현 | 더 효율적인 retrieval 방법으로 대체 가능 |
| 4. Findings | 연구 방향 명확화 | 일반화 문제 해결 연구 촉진 |

### Dependencies

```
Contribution 1 (Benchmark)
    │
    └── enables → Contribution 2 (Evaluation Splits)
                      │
                      └── reveals → Contribution 4 (Findings)
    │
    └── motivates → Contribution 3 (DMR)
```

---

## Source References Summary

| Contribution | Primary Source | Supporting Sources |
|--------------|----------------|-------------------|
| 1. Benchmark | [Section 3](../2402.05930.md#3-weblinx) | [Table 1](../2402.05930.md#table-1), [Appendix A](../2402.05930.md#a-dataset-details) |
| 2. Evaluation | [Table 2](../2402.05930.md#table-2) | [Section 4](../2402.05930.md#4-evaluation-framework) |
| 3. DMR | [Section 5.1](../2402.05930.md#51-dense-markup-ranking) | [Appendix B.4](../2402.05930.md#b4-technical-aspects) |
| 4. Findings | [Section 6](../2402.05930.md#6-experimental-results) | [Table 4](../2402.05930.md#table-4), [Table 5](../2402.05930.md#table-5) |

---

## Navigation

← [Back to Main Analysis](../2402.05930-analysis.md)
