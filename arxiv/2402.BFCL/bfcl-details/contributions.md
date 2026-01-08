---
parent: ../bfcl-analysis.md
source: ../bfcl.md
---

# Core Contributions - Deep Analysis

## Contribution 1: AST Substring Matching 평가 방식

**What**: Function call의 정확성을 **실제 함수 실행 없이** Abstract Syntax Tree 기반 substring matching으로 검증하는 방법론

**Technical Details**:
- 모델 출력을 Python callable format으로 제한 (prompt instruction)
- Python `ast` 모듈로 함수명과 파라미터 추출
- 파라미터별 **valid value set** 정의하여 exact match 대신 membership 검증
- 함수명 exact match + 모든 파라미터 값이 possible answers 내 포함 시 정답

**Why it matters**:
- **재현성**: LLM evaluator의 주관성/변동성 제거
- **확장성**: 실행 환경 구축 없이 수천 개 함수 평가 가능
- **검증됨**: Execution 평가와 강한 상관관계 (Figure 3)

**Limitations**:
- Semantic equivalence 미포착 (e.g., `"NYC"` vs `"New York City"`)
- Side effect 검증 불가 (stateful operation)

**Source**: [Section 4.1 AST Substring Matching](../bfcl.md#41-ast-substring-matching)

---

## Contribution 2: Crowd-sourced 실제 사용자 데이터

**What**: 2024년 2-4월 기간 수집된 **64,517개 실제 사용자 쿼리**에서 큐레이션된 2,251개 고품질 데이터포인트

**Data Curation Pipeline**:
1. **중복 제거**: ROUGE-L + embedding similarity
2. **공개 테스트셋 제외**: Nexus 등 기존 벤치마크 쿼리 필터링
3. **최소 편집 원칙**: 의미 보존하며 명확성만 개선
4. **개인정보 마스킹**: placeholder 대체

**Why it matters**:
- **현실 복잡성 반영**: Multi-lingual, redundant info, diverse styles
- **Contamination 탐지**: Single-turn과 perplexity 비교로 과적합 탐지
- 예시: xLAM-7B는 crowd-sourced에서 perplexity 3.67→5.09 급증 (과적합 징후)

**Statistics**:
- 평균 3개 함수 선택지 (최대 37개)
- 평균 4개 파라미터 (최대 28개)
- 15+ 언어 포함 (중국어, 프랑스어, 일본어, 한국어 등)

**Source**: [Section 3.2 Crowd-sourced Dataset](../bfcl.md#32-crowd-sourced-dataset), [Section 5.5 Measuring Data Contamination](../bfcl.md#55-measuring-data-contamination-using-crowd-sourced)

---

## Contribution 3: Multi-turn + Agentic 평가 체계

**What**: 단일 턴 호출을 넘어 **지속적 컨텍스트 관리**와 **에이전트 수준 추론**을 평가하는 프레임워크

**Multi-turn Categories**:
| Category | Description | Challenge |
|----------|-------------|-----------|
| Base | 기본 multi-turn 태스크 | 상태 추적 |
| Missing Parameters | 불완전 쿼리 처리 | Follow-up 생성 |
| Missing Functions | 불가능 태스크 인식 | 적절한 거절 |
| Long Context | 긴 대화 처리 | 토큰 효율성 |

**Agentic Categories**:
| Category | Tools | Challenge |
|----------|-------|-----------|
| Web Search | DuckDuckGo + Fetch | 정보 검색 + 추론 |
| Memory | Key-Value Store | 장단기 기억 관리 |
| SQL | JSON Schema | 구조화된 쿼리 생성 |

**Evaluation Method**:
- **State-based**: 최종 상태가 ground truth와 일치하는지
- **Response-based**: 최소 필요 함수 호출 경로를 따랐는지

**Why it matters**:
- **실제 에이전트 시나리오 반영**: Production 환경과 유사
- **병목 발견**: Memory 태스크에서 최고 12% → 핵심 미해결 문제

**Source**: [Section 3.3 Multi-turn Dataset](../bfcl.md#33-multi-turn-dataset), [Section 3.4 Agentic Dataset](../bfcl.md#34-agentic-dataset)

---

## Contribution 4: 다중 프로그래밍 언어 지원

**What**: Python, Java, JavaScript, REST API, SQL 5개 언어/형식 지원

**Implementation**:
- **Python**: AST 직접 파싱 + 실행 평가
- **Java/JavaScript**: Tree-sitter로 Python 변환 후 평가
- **REST API**: `requests.get()` wrapper 형태
- **SQL**: JSON schema 기반 구조화된 호출

**Type Validation Rules**:
| Language | int → float | float → int |
|----------|-------------|-------------|
| Python | OK (auto-convert) | Invalid |
| Java/JavaScript | Invalid (must use 5.0) | Invalid |

**Why it matters**:
- **실제 개발 환경 반영**: 다양한 언어 사용 상황
- **Cross-language 일관성**: Tree-sitter 변환으로 통일된 평가

**Source**: [Appendix H BFCL AST Substring Matching](../bfcl.md#h-bfcl-ast-substring-matching)

---

## Contribution Summary Table

| Contribution | Novelty | Impact | Source |
|--------------|---------|--------|--------|
| AST Matching | High | Enables scalable deterministic evaluation | Section 4.1 |
| Crowd-sourced | High | Real-world complexity + contamination detection | Section 3.2, 5.5 |
| Multi-turn/Agentic | Medium | Comprehensive capability assessment | Section 3.3, 3.4 |
| Multi-language | Medium | Practical coverage | Appendix H |
