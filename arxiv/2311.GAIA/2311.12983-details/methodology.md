---
parent: "../2311.12983-analysis.md"
source: "../2311.12983.md"
type: detail
created: 2026-01-09
---

# GAIA: Detailed Methodology Analysis

[Back to Main Analysis](../2311.12983-analysis.md) | [Original Paper](../2311.12983.md)

---

## 1. 벤치마크 설계 방법론

### 1.1 설계 철학: Proof of Work 패러다임

GAIA의 핵심 설계는 암호화폐의 **Proof of Work** 알고리즘에서 영감을 받았다 [Section 1](../2311.12983.md#1-introduction):

```
복잡한 문제 해결 → 쉬운 검증
(Complex to solve → Easy to verify)
```

**적용 방식:**
- 질문 해결: 다단계 추론, 도구 사용, 정보 통합 필요
- 답변 검증: 단일 정답과의 정확한 매칭

### 1.2 4대 설계 원칙

#### 원칙 1: Real-world and Challenging
```yaml
목표: 실제 AI 어시스턴트 사용 사례 반영
구현:
  - 웹 브라우징 (변화하는 실제 웹)
  - 멀티모달 처리 (이미지, 오디오, 비디오)
  - 파일 처리 (PDF, Excel, CSV)
회피:
  - 합성 환경
  - 폐쇄형 API 테스트
```

#### 원칙 2: Easy Interpretability
```yaml
목표: 비전문가도 이해 가능한 과제
구현:
  - 개념적으로 단순한 질문
  - 추적 가능한 추론 과정
  - 제한된 수의 고품질 질문 (466개)
검증:
  - 인간 성공률 92%
```

#### 원칙 3: Non-gameability
```yaml
목표: 데이터 오염 및 암기 저항
구현:
  - 인터넷에 평문으로 없는 답변
  - 정확한 답변 요구 (근사치 불가)
  - 다양한 액션 스페이스 (브루트포스 불가)
  - 추론 과정 추적 가능
보완책:
  - 새 질문 생성 가이드라인 제공
```

#### 원칙 4: Simplicity of Use
```yaml
목표: 간단하고 재현 가능한 평가
구현:
  - Zero-shot 평가
  - 단일 정답 (factoid)
  - 표준화된 시스템 프롬프트
  - 자동화된 스코어링 함수
```

---

## 2. 데이터셋 구축 방법론

### 2.1 질문 생성 프로세스 [Section 3.4](../2311.12983.md#34-building-and-extending-gaia)

```
┌─────────────────────────────────────────────────────────────┐
│                    질문 생성 파이프라인                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [1] 시드 질문 설계 (연구팀)                                  │
│           ↓                                                 │
│  [2] 가이드라인 + 예시 제공                                   │
│           ↓                                                 │
│  [3] 어노테이터 질문 생성 (Surge AI)                          │
│           ↓                                                 │
│  [4] 자체 검증 (질문 생성자가 답변도 작성)                      │
│           ↓                                                 │
│  [5] 독립 검증 (2명의 새 어노테이터)                           │
│           ↓                                                 │
│  [6] 합의 → 채택 / 불일치 → 수정 또는 제거                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 질문 생성 가이드라인 [Appendix D](../2311.12983.md#d-extended-description-of-our-question-design-framework)

**필수 요구사항:**
1. 신뢰할 수 있는 출처 기반 (Wikipedia, arXiv, GitHub 등)
2. 인터넷에 평문으로 없는 답변
3. 숫자 또는 짧은 단어로 된 정답
4. 시간 불변적 답변
5. 명확하고 단일한 정답
6. 실용적이고 흥미로운 질문
7. 합리적 시간 내 인간이 해결 가능
8. robots.txt 준수

### 2.3 어노테이션 형식

**생성 단계 메타데이터 (Table 1):**
```yaml
Question: [질문 텍스트]
File: [첨부 파일 경로 또는 None]
Level: [1, 2, 3]
Steps: [단계별 설명 리스트]
Number of steps: [숫자]
Answer: [정답]
Time to answer: [소요 시간]
Tools: [사용 도구 리스트]
Number of tools: [숫자]
```

**검증 단계 메타데이터 (Table 2):**
```yaml
Question: [질문 텍스트]
File: [첨부 파일 또는 None]
Level: [1, 2, 3]
Verifier response: [검증자 답변]
Answer match: [Yes/No + 설명]
Cause of mismatch: [불일치 원인 또는 None]
```

### 2.4 검증 통계 (Table 3)

| 지표 | 수치 |
|-----|------|
| 전체 생성 질문 | 623개 |
| 전체 검증 어노테이션 | 1,246개 |
| 두 검증자 모두 동의 | 55% |
| 한 검증자만 동의 | 27% |
| 두 검증자 모두 불일치 | 18% |
| 유효 질문 (채택) | 68% |
| Level 1 유효율 | 75% |
| Level 2 유효율 | 68% |
| Level 3 유효율 | 47% |
| 인간 점수 (전체) | 92% |

---

## 3. 난이도 분류 방법론

### 3.1 난이도 정의 [Section 3.3](../2311.12983.md#33-composition-of-gaia)

```
Level 1: 단순 과제
├── 도구: 0-1개
├── 단계: 최대 5단계
└── 예시: 웹 검색 후 단일 정보 추출

Level 2: 중간 과제
├── 도구: 다중
├── 단계: 5-10단계
└── 예시: 여러 출처 정보 조합 및 계산

Level 3: 복잡 과제
├── 도구: 제한 없음
├── 단계: 임의 길이
├── 특징: 복잡한 웹 탐색, 다중 모달리티
└── 예시: NASA 사진 분석 + 우주인 정보 조합 + 시간 계산
```

### 3.2 난이도 검증

Figure 4의 결과가 난이도 정의를 검증:
- Level 1 → Level 2 → Level 3으로 갈수록 모든 모델 성능 감소
- 인간도 난이도에 따라 소폭 성능 감소 (94% → 92% → 87%)

---

## 4. 평가 방법론

### 4.1 시스템 프롬프트 [Figure 2](../2311.12983.md#31-a-convenient-yet-challenging-benchmark-for-general-ai-assistants)

```text
You are a general AI assistant. I will ask you a question.
Report your thoughts, and finish your answer with the
following template: FINAL ANSWER: [YOUR FINAL ANSWER].

YOUR FINAL ANSWER should be a number OR as few words as
possible OR a comma separated list of numbers and/or strings.

If you are asked for a number, don't use comma to write your
number neither use units such as $ or percent sign unless
specified otherwise.

If you are asked for a string, don't use articles, neither
abbreviations (e.g. for cities), and write the digits in
plain text unless specified otherwise.

If you are asked for a comma separated list, apply the above
rules depending of whether the element to be put in the list
is a number or a string.
```

### 4.2 스코어링 함수

**Quasi-exact match:**
```python
def score(prediction, ground_truth):
    # 정규화
    pred_normalized = normalize(prediction)
    gt_normalized = normalize(ground_truth)

    # 타입별 비교
    if is_number(gt_normalized):
        return compare_numbers(pred_normalized, gt_normalized)
    elif is_list(gt_normalized):
        return compare_lists(pred_normalized, gt_normalized)
    else:
        return compare_strings(pred_normalized, gt_normalized)
```

### 4.3 평가 설정

**GPT-4 평가:**
- API 접근 (gpt-4 모델)
- 3회 실행 후 평균

**GPT-4 + Plugins 평가:**
- ChatGPT 웹 인터페이스 사용
- 질문별 수동 플러그인 선택 (oracle)
- Advanced Data Analysis 또는 3개 서드파티 플러그인

**AutoGPT 평가:**
- GPT-4 백엔드
- 자동 도구 선택
- Git hash: ed172dec1947466cc0942abf75bb77b027cd433d

---

## 5. 능력 분류 체계

### 5.1 능력 정의 [Appendix C](../2311.12983.md#c-extended-description-of-gaia)

| 능력 | 설명 | 예시 도구 |
|-----|------|----------|
| Web browsing | 웹 검색 및 탐색 | 브라우저, 검색엔진, YouTube, Street View |
| Multi-modality | 텍스트 외 데이터 이해 | STT, 이미지/비디오 인식, OCR |
| Coding | 코드 실행 | Python, 계산기, 컴파일러 |
| Diverse filetype | 다양한 파일 형식 처리 | PDF, Excel, PowerPoint, CSV |
| N/A | 순수 LLM 능력 | 번역기, 맞춤법 검사기 |

### 5.2 능력 분포 (Figure 3)

웹 브라우징이 가장 많이 요구되며, 대부분의 Level 2, 3 질문은 다중 능력을 요구함.

---

## 6. 재현성 고려사항

### 6.1 도전 과제
- 폐쇄형 API의 시간에 따른 변화
- 플러그인 가용성 변동
- 웹 콘텐츠 변화

### 6.2 완화 전략
- 개발 세트 166개 공개 (정답 포함)
- 테스트 세트 300개 (리더보드용)
- 질문 생성 가이드라인 제공
- 스코어링 함수 공개
