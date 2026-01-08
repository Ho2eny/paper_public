---
parent: ../2406.20015-analysis.md
source: ../2406.20015.md
---

# Core Contributions - Deep Analysis

## Contribution 1: Multi-level Diagnostic Framework

**What**: Hallucination을 **3단계**로 분해하여 진단하는 프레임워크

**Level Breakdown**:

| Level | Task | Question | Metric |
|-------|------|----------|--------|
| L1 | Solvability Detection | "이 태스크가 해결 가능한가?" | Exact Match (EM) |
| L2 | Solution Planning | "어떤 순서로 도구를 사용해야 하는가?" | Progress Rate (PR) |
| L3 | Missing-tool Analysis | "누락된 도구는 무슨 기능이 필요한가?" | PR + Matching Score (MS) |

**Why it matters**:
- **단순 정오답 → 원인 진단**: "틀렸다" 대신 "여기서 이런 이유로 틀렸다"
- **점진적 난이도**: L1→L2→L3로 갈수록 더 세부적인 이해 필요
- **실용적 피드백**: L3에서 누락 도구 기능 설명 → 실제 도구 개발에 활용 가능

**Limitation**: 단계 간 독립성 부족 (L1 틀리면 L2, L3 의미 감소)

**Source**: [Section 3.2](../2406.20015.md#32-in-depth-multi-level-diagnostic-task)

---

## Contribution 2: Hallucination-inducing Scenarios

**What**: Toolset 특성 기반 **3가지 hallucination 유발 시나리오** 정의

**Scenarios**:

### Missing Necessary Tools (MNT)
- **설정**: 필수 도구를 toolset에서 제거
- **Sub-tasks**:
  - Single-step: 단일 도구로 해결 가능한 태스크
  - Multi-step w/o Repetition: 여러 도구, 반복 없음
  - Multi-step w/ Repetition: 여러 도구, 반복 있음
- **Challenge**: "필요한 도구가 없음"을 인식해야 함

### Potential Tools (PT)
- **설정**: OS/Web 환경에서 제공되지 않은 도구 사용 유도
- **Sub-tasks**:
  - OS: `rm`, `ufw` 같은 시스템 명령어 유도
  - Web: JavaScript, SQL 같은 웹 기술 유도
- **Challenge**: 환경 내 "사용 불가" 도구와 "제공된" 도구 구분

### Limited Functionality Tools (LFT)
- **설정**: 도구는 있지만 기능이 제한됨
- **Sub-tasks**:
  - Iterative: 다기능 도구의 일부 기능만 제공
  - Optimal Selection: 여러 유사 도구 중 요구사항 충족 도구 없음
- **Challenge**: 도구 기능의 세부 제약 이해 필요

**Why it matters**:
- **현실 복잡성 반영**: 실제로 toolset은 항상 불완전
- **다양한 실패 모드 커버**: 도구 부재, 환경 혼동, 기능 제한 모두 평가
- **에러 원인 분리**: 어떤 시나리오에서 어떤 에러가 나는지 분석 가능

**Source**: [Section 3.3 In-breadth](../2406.20015.md#33-in-breadth-tool-based-hallucination-inducing-tasks)

---

## Contribution 3: Unsolvable Task Focus

**What**: 기존 벤치마크의 **"solvable 가정"** 탈피, unsolvable long-tail 태스크에 집중

**Key Insight**:
- 기존 벤치마크: 모든 태스크가 해결 가능하다고 가정
- 실제 환경: 사용자 요청 중 상당수가 주어진 도구로 해결 불가능
- ToolBH: 50% solvable + 50% unsolvable로 균형 잡힌 평가

**Solvable vs Unsolvable 성능 격차** (Table 3):

| Model | Solvable | Unsolvable | Gap |
|-------|----------|------------|-----|
| GPT-4o | 89.0% | 37.0% | **-52%** |
| Llama-3-70B | 84.3% | 14.6% | **-70%** |
| Ratio (Open/Proprietary) | 91.1% | 39.4% | 2.3x 격차 |

**Why it matters**:
- **실제 서비스 안전성**: "할 수 없다"고 말하는 능력이 hallucination 방지에 핵심
- **Proprietary vs Open-weight**: Unsolvable에서 격차 폭발 → 학습 데이터 차이 시사
- **새로운 연구 방향**: Unsolvable handling은 미개척 영역

**Source**: [Section 5.3](../2406.20015.md#53-solvable-tasks-versus-corresponding-unsolvable-tasks)

---

## Contribution 4: Progress Rate (PR) Metric

**What**: 순차적 도구 호출의 **부분적 정확도**를 측정하는 새로운 메트릭

**Formula**:
$$PR = \frac{k-1}{|G|}$$

**특징**:
- **첫 번째 오류까지 측정**: k는 첫 불일치 인덱스
- **순서 민감**: 초반 오류 시 이후 전부 0점
- **부분 점수 가능**: 완전 정답 아니어도 진행도 측정

**비교**:
| Metric | 완전 정답 | 부분 정답 | 순서 고려 |
|--------|-----------|-----------|-----------|
| Exact Match | 1점 | 0점 | N/A |
| Set Match | 1점 | 부분 | X |
| **Progress Rate** | 1점 | 부분 | **O** |

**Why it matters**:
- **ReAct 에이전트 평가 적합**: 순차적 reasoning 과정 평가
- **세밀한 진단**: "어디까지 맞았나"로 모델 능력 세분화
- **L2와 L3에서 활용**: Solution planning의 진행도 측정

**Source**: [Section 3.2.2](../2406.20015.md#322-level-2-solution-planning)

---

## Contribution Summary Table

| Contribution | Novelty | Impact | Source |
|--------------|---------|--------|--------|
| Multi-level Diagnosis | High | 원인 진단 가능한 평가 체계 | Section 3.2 |
| Hallucination Scenarios | High | 현실적 unsolvable 상황 재현 | Section 3.3 |
| Unsolvable Focus | High | Long-tail robustness 평가 | Section 5.3 |
| Progress Rate | Medium | 순차적 정확도 세밀 측정 | Section 3.2.2 |
