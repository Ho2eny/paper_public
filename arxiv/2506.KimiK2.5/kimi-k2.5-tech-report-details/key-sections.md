---
parent: ../kimi-k2.5-tech-report-analysis.md
source: ../kimi-k2.5-tech-report.md
created: 2026-02-06
---

# Key vs Non-Key Sections - Deep Analysis

## Reading Strategy

| Metric | Value |
|--------|-------|
| **Total Reading Time** | ~90 minutes (전체) |
| **Essential Only** | ~35 minutes (핵심만) |
| **Total Sections** | 6 main + 9 appendix sections |
| **Must Read** | 4 sections |

---

## ⭐⭐⭐ Must Read

### Section 2: Joint Optimization of Text and Vision

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
K2.5의 첫 번째 핵심 기여. Early fusion이 late fusion보다 우수하다는 반직관적 발견과 비전 RL→텍스트 성능 향상이라는 크로스모달 전이가 이 섹션에서 제시된다.

**Key Points**:
1. **Table 1**: Early fusion(10:90) vs Late fusion(50:50) 비교 — 모든 지표에서 early 우위
2. **Zero-Vision SFT**: 텍스트 전용 SFT로 비전 능력 활성화 가능 (Section 2.2)
3. **Cross-modal transfer**: Vision RL 후 MMLU-Pro +1.7%, GPQA-Diamond +2.1% (Table 2)
4. **Joint RL 설계**: 모달리티별이 아닌 능력별 도메인 조직화

**Reading Tips**:
- Table 1의 수치를 직접 비교하며 읽기 — early fusion의 이점이 명확히 드러남
- Section 2.2의 zero-vision SFT 메커니즘에 집중 — 간결하지만 핵심적 발견
- Figure 2의 RL 학습 곡선으로 스케일링 특성 확인

**Estimated Time**: ~10 minutes

**Link**: [Section 2](../kimi-k2.5-tech-report.md#2-joint-optimization-of-text-and-vision)

---

### Section 3: Agent Swarm

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
K2.5의 두 번째 핵심 기여. PARL 패러다임의 전체 설계(아키텍처, 보상 함수, CriticalSteps)가 이 섹션에서 제시된다.

**Key Points**:
1. **분리 아키텍처**: Trainable orchestrator + Frozen sub-agents — 왜 end-to-end가 아닌지 근거
2. **PARL Reward**: 3개 보상 항 각각의 역할과 어닐링 전략
3. **CriticalSteps**: 병렬 실행 시간 측정 지표 — 크리티컬 패스 개념 차용
4. **Prompt Construction**: wide search / deep search로 병렬화를 명시적 지시 없이 유도

**Reading Tips**:
- Figure 3의 아키텍처 다이어그램을 먼저 보고 구조 파악
- PARL reward의 3개 항을 serial collapse와 spurious parallelism 문제와 연결하며 이해
- CriticalSteps 수식이 왜 total steps보다 더 적절한 지표인지 직관적으로 이해

**Estimated Time**: ~12 minutes

**Link**: [Section 3](../kimi-k2.5-tech-report.md#3-agent-swarm)

---

### Section 5.1: Main Results

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
40+ 벤치마크에서의 종합적 성능 비교. 프로프라이어터리 모델(GPT-5.2, Claude Opus 4.5, Gemini 3 Pro)과의 직접 비교가 핵심.

**Key Points**:
1. **Table 4**: 6개 도메인 종합 비교 — K2.5의 강세/약세 영역 한눈에 파악
2. **Agentic SOTA**: BrowseComp, WideSearch, DeepSearchQA 등에서 전체 1위
3. **Vision 경쟁력**: MMMU-Pro 78.5%, OCRBench 92.3%로 프로프라이어터리급
4. **약점**: SimpleQA Verified(36.9%)에서 Gemini(72.1%) 대비 큰 격차
5. **Table 5**: 토큰 효율 비교 — Toggle 효과 확인

**Reading Tips**:
- Table 4를 도메인별로 나누어 읽기 (추론, 코딩, 에이전틱, 이미지, 비디오, 컴퓨터)
- 각 도메인에서 bold(SOTA) 위치를 확인하여 K2.5의 포지셔닝 파악
- 약점 영역(SimpleQA)이 도구 사용 시 어떻게 변하는지 주목 (HLE w/ tools)

**Estimated Time**: ~8 minutes

**Link**: [Section 5.1](../kimi-k2.5-tech-report.md#51-main-results)

---

### Section 5.2: Agent Swarm Results

**Priority**: ⭐⭐⭐ Essential

**Why Read**:
Agent Swarm의 실제 효과를 정량적으로 검증. 성능 향상과 지연 시간 단축 데이터.

**Key Points**:
1. **Table 6**: 단일 에이전트 대비 Agent Swarm 성능 향상 (BrowseComp +17.8%p)
2. **Figure 8**: 3-4.5x 실행 시간 단축 — 태스크 복잡도 증가 시 효과 증대
3. **Proactive context management**: Figure 7에서 Discard-all 대비 Agent Swarm 우위

**Reading Tips**:
- Figure 8의 x축(target Item-F1)에 따른 시간 차이 변화에 주목
- "context sharding"과 "context truncation"의 개념적 차이를 명확히 이해

**Estimated Time**: ~5 minutes

**Link**: [Section 5.2](../kimi-k2.5-tech-report.md#52-agent-swarm-results)

---

## ⭐⭐ Important

### Section 4.2: Model Architecture (MoonViT-3D)

**Priority**: ⭐⭐ Important

**Why Read**:
MoonViT-3D의 네이티브 해상도 비전 인코딩과 3D 비디오 압축 메커니즘. 비전 성능의 기술적 기반.

**Key Points**:
1. NaViT 패킹 전략으로 다양한 해상도 네이티브 처리
2. 4프레임 묶음 → 시공간 볼륨 → 시간 풀링으로 4x 비디오 길이 확장
3. 이미지-비디오 인코더 완전 파라미터 공유

**When to Read**:
비전 아키텍처 세부사항이 필요하거나, MoonViT-3D를 다른 비전 인코더와 비교할 때.

**Link**: [Section 4.2](../kimi-k2.5-tech-report.md#42-model-architecture)

---

### Section 4.4: Post-Training (SFT + RL)

**Priority**: ⭐⭐ Important

**Why Read**:
Policy optimization loss 수식(토큰 레벨 클리핑)과 Toggle 알고리즘의 상세. RL 학습 안정성의 핵심.

**Key Points**:
1. 토큰 레벨 log-ratio 클리핑으로 off-policy drift 방지
2. GRM(Generative Reward Models) — rule-based reward 너머의 nuanced 보상
3. Toggle: budget-limited과 standard scaling 교대 최적화
4. Figure 5: Toggle의 토큰 절감 효과와 도메인 일반화

**When to Read**:
RL 학습 알고리즘 세부사항이 필요하거나, 자체 RL 파이프라인에 적용하려 할 때.

**Link**: [Section 4.4](../kimi-k2.5-tech-report.md#44-post-training)

---

### Section 4.3: Pre-training Pipeline

**Priority**: ⭐⭐ Important

**Why Read**:
ViT Training → Joint Training → Long-context Mid-training 3단계 파이프라인과 데이터 구성.

**Key Points**:
1. Table 3: 단계별 데이터, 시퀀스 길이, 학습 대상 정리
2. CoCa 기반 ViT 학습: SigLIP + captioning 이중 목표
3. YaRN 보간법으로 4K→262K 컨텍스트 확장

**When to Read**:
사전훈련 파이프라인 설계를 참고하거나 재현하려 할 때.

**Link**: [Section 4.3](../kimi-k2.5-tech-report.md#43-pre-training-pipeline)

---

## ⭐ Reference

### Section 4.5: Training Infrastructure (DEP)

**Priority**: ⭐ Reference Only

**Content Summary**:
DEP(Decoupled Encoder Process)의 3단계 실행과 멀티모달 학습 효율 최적화. H800 클러스터 설정.

**When Useful**:
- 대규모 멀티모달 모델 학습 인프라를 설계할 때
- 비전 인코더를 기존 PP 파이프라인에 통합할 때

**Link**: [Section 4.5](../kimi-k2.5-tech-report.md#45-training-infrastructure)

---

### Appendix D: Unified Agentic RL Environment

**Priority**: ⭐ Reference Only

**Content Summary**:
Gym-like RL 환경 인터페이스, 비동기 코루틴 기반 에이전트 태스크 관리, LLM Gateway 프록시.

**When Useful**:
- 에이전틱 RL 학습 환경을 구축할 때
- 100K 동시 에이전트 태스크 관리 방법이 필요할 때

**Link**: [Appendix D](../kimi-k2.5-tech-report.md#d-unified-agentic-reinforcement-learning-environment)

---

### Appendix E: Evaluation Settings

**Priority**: ⭐ Reference Only

**Content Summary**:
모든 벤치마크의 상세 평가 설정, 온도, 시스템 프롬프트, 에이전트 설정.

**When Useful**:
- K2.5의 벤치마크 결과를 재현하거나 비교할 때
- 에이전트 평가 프로토콜을 설계할 때

**Link**: [Appendix E](../kimi-k2.5-tech-report.md#e-evaluation-settings)

---

## Skip

### Section 4.1: Foundation (Kimi K2 Base)

**Priority**: Skip

**Why Skip**:
- Kimi K2의 기본 사양(1.04T 파라미터, 32B 활성화, MoE) 요약만 포함
- 상세는 별도 K2 기술 보고서 참조

**Alternative**:
[Kimi K2 Technical Report](https://arxiv.org/abs/2507.20534)

**Link**: [Section 4.1](../kimi-k2.5-tech-report.md#41-foundation-kimi-k2-base-model) (참고용)

---

### Appendix B-C: Pre-training Data & Infra Details

**Priority**: Skip

**Why Skip**:
- 데이터 카테고리 나열과 GPU 클러스터 설정은 본 논문의 핵심 기여가 아님
- 재현이 불가능한 수준의 인프라 상세

**Alternative**:
- Table 3 (Section 4.3)에 데이터/학습 단계 요약이 충분
- DEP만 Section 4.5.1에서 확인하면 충분

**Link**: [Appendix B](../kimi-k2.5-tech-report.md#b-pre-training), [Appendix C](../kimi-k2.5-tech-report.md#c-infra) (참고용)

---

### Appendix A: Contributions (Author List)

**Priority**: Skip

**Why Skip**:
- 저자 목록만 포함

**Link**: [Appendix A](../kimi-k2.5-tech-report.md#a-contributions) (참고용)

---

## Recommended Reading Order

최적의 읽기 순서:

```
1. Section 2: Joint Optimization ← 핵심 기여 #1 (early fusion, zero-vision SFT, cross-modal transfer)
   ↓
2. Section 3: Agent Swarm ← 핵심 기여 #2 (PARL, CriticalSteps)
   ↓
3. Section 5.1-5.2: Evaluations ← 결과 검증 (40+ benchmarks, Agent Swarm 효과)
   ↓
4. Section 4.4: Post-Training (선택) ← RL 알고리즘 상세 (Toggle, policy optimization)
   ↓
5. Section 4.2-4.3: Architecture & Pipeline (선택) ← 기술적 배경
```

### Quick Read Path (~15 min)
1. Abstract
2. Section 2.1 (Table 1만 집중) + Section 2.2 (zero-vision SFT 핵심만)
3. Section 3 (Figure 3 + PARL reward + CriticalSteps만)
4. Table 4 (도메인별 SOTA 확인)
5. Table 6 + Figure 8 (Agent Swarm 효과)

### Deep Read Path (~50 min)
1. 전체 Section 2
2. 전체 Section 3
3. Section 4.4 (RL 상세)
4. 전체 Section 5
5. Section 4.2 (MoonViT-3D)
6. Appendix D (RL 환경 — 관심 시)

---

## Section Dependency Map

```
Introduction (§1)
    │
    ├── Section 2: Joint Optimization ◀── 먼저 읽기
    │   ├── §2.1: Native Multimodal Pre-training ◀── Table 1 핵심
    │   ├── §2.2: Zero-Vision SFT ◀── 핵심 발견
    │   └── §2.3: Joint Multimodal RL ◀── Cross-modal transfer
    │
    ├── Section 3: Agent Swarm ◀── 두 번째
    │   ├── Architecture & PARL ◀── Figure 3 필수
    │   ├── CriticalSteps ◀── 새 지표
    │   └── Prompt Construction
    │
    ├── Section 4: Method Overview (배경)
    │   ├── §4.2: MoonViT-3D (선택)
    │   ├── §4.3: Pre-training Pipeline (선택)
    │   └── §4.4: Post-Training ◀── RL 상세 (Important)
    │
    └── Section 5: Evaluations ◀── 세 번째
        ├── §5.1: Main Results ◀── Table 4 필수
        └── §5.2: Agent Swarm Results ◀── Table 6, Figure 8 필수
```

---

## Source References

| Section | Priority | Time | Link |
|---------|----------|------|------|
| §2 Joint Optimization | ⭐⭐⭐ | ~10 min | [Link](../kimi-k2.5-tech-report.md#2-joint-optimization-of-text-and-vision) |
| §3 Agent Swarm | ⭐⭐⭐ | ~12 min | [Link](../kimi-k2.5-tech-report.md#3-agent-swarm) |
| §5.1 Main Results | ⭐⭐⭐ | ~8 min | [Link](../kimi-k2.5-tech-report.md#51-main-results) |
| §5.2 Agent Swarm Results | ⭐⭐⭐ | ~5 min | [Link](../kimi-k2.5-tech-report.md#52-agent-swarm-results) |
| §4.2 Architecture | ⭐⭐ | ~5 min | [Link](../kimi-k2.5-tech-report.md#42-model-architecture) |
| §4.4 Post-Training | ⭐⭐ | ~8 min | [Link](../kimi-k2.5-tech-report.md#44-post-training) |
| §4.3 Pre-training | ⭐⭐ | ~5 min | [Link](../kimi-k2.5-tech-report.md#43-pre-training-pipeline) |
| §4.5 Infrastructure | ⭐ | ~5 min | [Link](../kimi-k2.5-tech-report.md#45-training-infrastructure) |
| Appendix D-E | ⭐ | ~10 min | [Link](../kimi-k2.5-tech-report.md#d-unified-agentic-reinforcement-learning-environment) |
| §4.1, Appendix A-C | Skip | - | [Link](../kimi-k2.5-tech-report.md#41-foundation-kimi-k2-base-model) |

---

## Navigation

← [Back to Main Analysis](../kimi-k2.5-tech-report-analysis.md)
