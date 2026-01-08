---
parent: ../2307.13854-analysis.md
source: ../2307.13854.md
type: detail
section: contributions
---

# Core Contributions Deep Analysis

> **Source**: [Introduction](../2307.13854.md#1-introduction), [Sections 2-3](../2307.13854.md#2-webarena-websites-as-an-environment-for-autonomous-agents)

## Primary Contributions

### 1. Realistic & Reproducible Web Environment
> **Source**: [Section 2](../2307.13854.md#2-webarena-websites-as-an-environment-for-autonomous-agents)

**기여 내용**:
- 4개 주요 웹 도메인을 완전히 기능하는 독립형 환경으로 구축
- Docker 컨테이너 기반으로 재현성 보장
- 실제 웹사이트에서 추출한 데이터로 진정성 확보

**기술적 구현**:
- E-commerce: Adobe Magento + ~90,000 상품 (WebShop 데이터)
- Social Forum: Postmill (Reddit 오픈소스) + 95 subreddits, 127,390 posts
- GitLab: 공식 프로젝트 기반 + 300 repositories, 1000+ accounts
- CMS: Adobe Magento Admin Portal

**혁신성**: 기존 벤치마크들이 단순화된 환경(MiniWoB++) 또는 정적 데이터(Mind2Web)에 의존한 반면, WebArena는 실제 운영 가능한 수준의 웹 애플리케이션을 제공

**Impact Score**: ★★★★★ (핵심 기여)

---

### 2. Multi-Tab Observation Space with Accessibility Tree
> **Source**: [Section 2.3](../2307.13854.md#23-observation-space)

**기여 내용**:
- 최초로 멀티탭 웹 태스크 환경 지원
- 세 가지 관찰 모드 제공: HTML DOM, Screenshot, Accessibility Tree

**Accessibility Tree의 장점**:
```
기존 DOM Tree: 복잡하고 장황한 구조
Accessibility Tree:
- 관련 요소만 포함 (더 간결)
- 역할(role), 텍스트 내용, 속성 명시
- 구조화된 정보 유지
```

**혁신성**: 멀티탭 지원으로 도구 사용, 탭 간 비교/참조 등 인간의 실제 브라우징 패턴 모방 가능

**Impact Score**: ★★★★☆

---

### 3. Functional Correctness Evaluation Framework
> **Source**: [Section 3.2](../2307.13854.md#32-evaluation-annotation)

**기여 내용**:
두 가지 평가 함수 프레임워크 제안:

**$r_{info}(\hat{a}, a^*)$ - 정보 탐색 태스크용**:
| 함수 | 용도 | 예시 |
|------|------|------|
| `exact_match` | 정형화된 답변 | 이름, 날짜 |
| `must_include` | 핵심 개념 포함 | 이름 + 이메일 |
| `fuzzy_match` | 의미적 동등성 (GPT-4 평가) | 시간 형식 변환 |

**$r_{prog}(s)$ - 사이트 탐색/콘텐츠 태스크용**:
- **Locator**: 데이터베이스 쿼리, API 호출, JavaScript 요소 선택
- **Validator**: `exact_match`, `must_include` 함수 재사용

**혁신성**: 액션 시퀀스 비교 대신 실제 결과 상태를 검증하여 다양한 정답 경로 허용

**Impact Score**: ★★★★★ (핵심 기여)

---

### 4. Unachievable Task Detection
> **Source**: [Section 3.2 - Unachievable Tasks](../2307.13854.md#32-evaluation-annotation)

**기여 내용**:
- 불가능한 태스크를 의도적으로 포함 (SQuAD 2.0에서 영감)
- 에이전트의 근거 없는 주장 회피 능력 평가
- "N/A" 응답 생성 기대

**예시**: "Tell me the contact number of OneStopShop" - 해당 정보가 웹사이트에 존재하지 않음

**Impact Score**: ★★★☆☆

---

## Secondary Contributions

### 5. Template-based Intent Generation
> **Source**: [Section 3.1](../2307.13854.md#31-intent-collection)

- 241개 템플릿에서 812개 인스턴스 생성
- 변수화된 태스크로 체계적인 다양성 확보
- 동일 템플릿에서도 다른 실행 경로 가능

### 6. User Role Simulation
> **Source**: [Section 2.4 - User Role Simulation](../2307.13854.md#24-action-space), [Appendix A.3](../2307.13854.md#a3-user-roles-simulation)

- 사용자별 역할, 권한, 상호작용 이력 시뮬레이션
- Pre-cached cookie로 자동 로그인
- 공개 에이전트 평가 환경 최초 구현

### 7. Human Performance Baseline
> **Source**: [Section 3.2](../2307.13854.md#32-evaluation-annotation)

- 170개 태스크 샘플, CS 대학원생 5명
- 전체 78.24%, 정보 탐색 74.68%, 기타 81.32%
- 실패 원인 분석: 50%는 의도 오해/불완전 답변

---

## Contribution Hierarchy

```
[Tier 1: Core Contributions]
├── Realistic & Reproducible Environment (★★★★★)
└── Functional Correctness Evaluation (★★★★★)

[Tier 2: Important Contributions]
├── Multi-Tab Observation Space (★★★★☆)
└── 812 Benchmark Tasks (★★★★☆)

[Tier 3: Supporting Contributions]
├── Unachievable Task Detection (★★★☆☆)
├── Template-based Generation (★★★☆☆)
└── User Role Simulation (★★★☆☆)
```

---

## Navigation

- [Main Analysis](../2307.13854-analysis.md)
- [TL;DR](./tldr.md)
- [Key Sections Guide](./key-sections.md)
- [Methodology Detail](./methodology.md)
