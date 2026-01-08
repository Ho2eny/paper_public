---
parent: ../2511.21631-analysis.md
source: ../2511.21631.md
language: "ko"
---

# 방법론 - 심층 분석

## 논문 유형: 방법 논문 (기술 보고서)

Qwen3-VL은 실전 배포 가능한 비전-언어 모델 시스템을 문서화한 **포괄적 기술 보고서**이다. 단일 초점 연구 논문과 달리 다음을 결합한다:
- 아키텍처 혁신 (3개의 새로운 구성요소)
- 학습 방법론 (4단계 사전학습 + 3단계 사후학습)
- 광범위한 평가 (50개 이상의 벤치마크)
- 제거 실험 (체계적 검증)

## 접근법 개요

**핵심 철학**: 체계적이고 경험적으로 검증된 반복을 통한 진화적 개선. 각 아키텍처 변경은 Qwen2.5-VL에서 관찰된 구체적 한계를 해결하며, 제거 실험을 통해 검증된다.

**고수준 파이프라인**:
```
사전학습된 구성요소 (SigLIP-2, Qwen3)
    ↓
4단계 사전학습 (8K → 32K → 256K 컨텍스트)
    ├─ S0: 비전-언어 정렬 (merger만)
    ├─ S1: 멀티모달 사전학습 (모든 파라미터, 8K)
    ├─ S2: 장문맥 사전학습 (모든 파라미터, 32K)
    └─ S3: 초장문맥 적응 (모든 파라미터, 256K)
    ↓
3단계 사후학습 (분기형)
    ├─ Phase 1: 지도 파인튜닝 (SFT)
    │   ├─ Non-Thinking: 표준 형식
    │   └─ Thinking: Long-CoT 형식
    ├─ Phase 2: Strong-to-Weak 증류
    └─ Phase 3: 강화학습 (RL)
        ├─ Reasoning RL (수학, 코드, 논리)
        └─ General RL (VQA, 캡셔닝, OCR)
    ↓
두 가지 모델 변형: Instruct (non-thinking) + Thinking
```

## 주요 설계 선택

| 선택 | 결정 | 근거 | 출처 |
|--------|----------|-----------|--------|
| **Vision Encoder** | SigLIP-2 SO-400M | SigLIP-1 대비 +3.5 MMMU; 적절한 스케일링이 있는 대비 사전학습 | [Section 2](../2511.21631.md#2-model-architecture), [Table 11](../2511.21631.md#table-11) |
| **위치 인코딩** | Interleaved MRoPE | 장시간 비디오 이해를 위한 균형 잡힌 주파수 스펙트럼 | [Section 2.1](../2511.21631.md#21-interleaved-mrope) |
| **다층 정렬** | DeepStack (3 ViT 계층 → 3 LLM 계층) | +2.2 MMMU; 컨텍스트 오버헤드 없이 더 풍부한 시각 정보 | [Section 2.2](../2511.21631.md#22-deepstack), [Table 12](../2511.21631.md#table-12) |
| **비디오 타임스탬프** | 명시적 텍스트 토큰 (`<3.0 seconds>`) | 절대 시간 위치 ID보다 간단하고 유연 | [Section 2.3](../2511.21631.md#23-video-timestamp) |
| **손실 정규화** | 제곱근 토큰당 손실 | 멀티모달 학습 중 텍스트 능력 보존 | [Section 3.1](../2511.21631.md#31-training-recipe) |
| **컨텍스트 확장** | 점진적 8K → 32K → 256K | 계산 효율성; 특수 256K 적응 | [Table 1](../2511.21631.md#L172) |
| **사후학습 전략** | 분기형 (thinking vs non-thinking) | 실전 배포를 위한 지연-품질 트레이드오프 | [Section 4](../2511.21631.md#4-post-training) |
| **RL 알고리즘** | SAPO (Smooth Adaptive Policy Optimization) | 다양한 작업과 스케일에 걸쳐 일관된 개선 | [Section 4.4.1](../2511.21631.md#441-reasoning-reinforcement-learning) |

## 기술 세부사항

### 아키텍처 설계

**3모듈 표준**: ([Section 2](../2511.21631.md#2-model-architecture))

```
┌─────────────────────────────────────────────────────┐
│ Vision Encoder: SigLIP-2 SO-400M                   │
│ - 동적 네이티브 해상도 (2D-RoPE 보간)              │
│ - 공식 사전학습된 체크포인트에서 초기화             │
│ - 동적 해상도로 학습 계속                           │
└─────────────────────────────────────────────────────┘
               ↓ (다층 특징)
┌─────────────────────────────────────────────────────┐
│ Vision-Language Merger: 2계층 MLP                  │
│ - 2×2 시각적 특징 압축 → 1 토큰                    │
│ - DeepStack: 다층용 3개의 특수 merger              │
│ - LLM 숨겨진 차원과 정렬                            │
└─────────────────────────────────────────────────────┘
               ↓ (시각적 토큰)
┌─────────────────────────────────────────────────────┐
│ Large Language Model: Qwen3                        │
│ - Dense: 2B/4B/8B/32B 파라미터                     │
│ - MoE: 30B-A3B (3B 활성), 235B-A22B (22B 활성)    │
│ - 네이티브 256K 토큰 컨텍스트 창                    │
└─────────────────────────────────────────────────────┘
```

**DeepStack 통합 세부사항**: ([Figure 1](../2511.21631.md#L138))

최종 ViT 계층 특징만 사용하는 표준 VLM 아키텍처와 달리, DeepStack은 여러 계층에서 추출한다:

```
ViT Layer 1 (저수준: 엣지, 텍스처)
    ↓ Merger₁ (2계층 MLP)
    ↓ 잔차 추가 → LLM Layer 1

ViT Layer 2 (중간 수준: 객체 부분)
    ↓ Merger₂ (2계층 MLP)
    ↓ 잔차 추가 → LLM Layer 2

ViT Layer 3 (고수준: 의미적 개념)
    ↓ Merger₃ (2계층 MLP)
    ↓ 잔차 추가 → LLM Layer 3

ViT Final Layer (최고 수준)
    ↓ 표준 Merger (2계층 MLP)
    ↓ 시각적 토큰 → LLM Layer 1 (표준 경로)
```

**주요 혁신**: 잔차 추가 (연결이 아님)는 컨텍스트 길이 증가가 없음을 의미한다. 각 LLM 계층은 해당 ViT 계층의 특징을 받아 깊이 정렬된 비전-언어 융합을 생성한다.

### 학습 파이프라인

#### 사전학습: 4단계 점진적 컨텍스트 확장

**Stage 0: 비전-언어 정렬** ([Lines 181-182](../2511.21631.md#L181-L182))

```python
# 의사 코드
frozen_params = [vision_encoder, llm]
trainable_params = [merger]
data = image_caption_pairs + ocr_data  # 67B 토큰
context_length = 8192
epochs = 1

# 목표: 사전학습된 구성요소를 방해하지 않고 효율적인 크로스 모달 브리징
```

**왜 Merger만 먼저?**
- Vision encoder는 이미 강력 (SigLIP-2 사전학습됨)
- LLM은 이미 강력 (Qwen3 사전학습됨)
- Merger만 크로스 모달 정렬을 학습해야 함
- 처음부터 모든 파라미터를 학습하면 컴퓨팅을 낭비하고 사전학습된 지식을 저하시킬 위험

**Stage 1: 멀티모달 사전학습** ([Lines 183-185](../2511.21631.md#L183-L185))

```python
trainable_params = [vision_encoder, merger, llm]  # 모든 구성요소
data_mix = {
    'image_caption': 30%,
    'interleaved_text_image': 20%,
    'visual_grounding': 15%,
    'vqa': 15%,
    'text_only': 10%,
    'video': 5%,
    'stem': 5%
}  # 대략적 비율 (논문에 명시되지 않음)
total_tokens = ~1_trillion
context_length = 8192
loss_fn = square_root_normalized_per_token_loss
```

**중요한 기법: 제곱근 손실 정규화**

표준 토큰당 손실:
```
Loss_sample = (1/n_tokens) × Σ loss_token_i
Total_Loss = Σ Loss_sample × n_tokens  # 그래디언트 스케일링
```

문제: 이미지는 수백 개의 토큰을 생성하고 텍스트는 수십 개를 생성. 긴 멀티모달 샘플이 그래디언트를 지배.

제곱근 정규화:
```
Total_Loss = Σ Loss_sample × √n_tokens
```

효과: 동적 범위 압축. 100개 토큰 샘플은 이제 √100 = 10× 가중치를 기여 (100×가 아님), 10개 토큰 샘플은 √10 ≈ 3.16× 가중치를 기여. 이는 텍스트와 멀티모달 기여를 균형있게 만든다.

**Stage 2: 장문맥 사전학습** ([Lines 187-188](../2511.21631.md#L187-L188))

```python
context_length = 32768  # Stage 1에서 4배 증가
total_tokens = ~1_trillion
data_mix = {
    'text_only': 40%,  # S1의 ~10%에서 증가
    'long_video': 20%,
    'agent_instructions': 15%,
    'other_vl': 25%
}
```

**왜 텍스트 비율을 증가?**
- 장문맥 텍스트 이해는 장문맥 멀티모달 이해의 전제 조건
- 더 많은 텍스트 전용 데이터는 더 긴 컨텍스트에서 LLM의 언어 능력을 강화
- 비디오 및 에이전트 데이터는 자연스럽게 더 긴 컨텍스트 필요 (다회차 상호작용, 시간 시퀀스)

**Stage 3: 초장문맥 적응** ([Lines 189-190](../2511.21631.md#L189-L190))

```python
context_length = 262144  # 256K 토큰 (Stage 2에서 8배 증가)
total_tokens = 100_billion  # S1/S2 예산의 5%만
data_focus = 'long_video + long_document'

# 특수 적응, 전체 재학습이 아님
# Stage 2에서 강력한 32K 기반 가정
```

**주요 통찰**: 256K 컨텍스트는 **특수 능력**이며, 전체 규모 재학습이 필요하지 않다. 100B 토큰 (S1/S2 예산의 5%)만으로 적응에 충분하며, 다음을 시사한다:
1. 대부분의 일반 멀티모달 이해는 8K-32K 컨텍스트에서 학습됨
2. 256K 적응은 주로 위치 외삽 및 주의 패턴을 가르침
3. 처음부터 256K에서 학습하는 것에 비해 계산적으로 효율적

#### 사후학습: 3단계 분기형 파이프라인

**Phase 1: 지도 파인튜닝 (SFT)**

**Non-Thinking 변형**:
```python
# 표준 질문-답변 형식
data = sft_dataset  # 1.2M 샘플
format = {
    'prompt': "Question: {query}\n\nAnswer:",
    'response': "{answer}"
}
context_length = [32K (에폭 1), 256K (에폭 2)]
```

**Thinking 변형**:
```python
# 명시적 사고 과정 (CoT) 형식
data = long_cot_dataset  # ~1:1 VL 대 텍스트 비율
format = {
    'prompt': "Question: {query}\n\nThink step by step:",
    'response': "Let me analyze this...\n\nStep 1: {reasoning_1}\n...\nTherefore, the answer is {answer}"
}

# Long-CoT 데이터 큐레이션 필터:
filters = [
    'difficulty_filter',  # 낮은 통과율 샘플 유지
    'multimodal_necessity_filter',  # 텍스트 전용 모델이 해결 가능한 샘플 폐기
    'quality_control'  # 잘못된 답변, 반복, 언어 혼합 제거
]
```

**중요한 필터: 멀티모달 필요성** ([Lines 337-338](../2511.21631.md#L337-L338))

비전-언어 수학 문제의 경우:
```python
# 의사 코드
for sample in vl_math_dataset:
    text_only_answer = qwen3_30b_nothink(sample.query)  # 이미지 접근 없음
    if text_only_answer == sample.ground_truth:
        dataset.remove(sample)  # 폐기: 비전 없이 해결 가능
```

이는 나머지 샘플이 **진정으로 멀티모달 이해를 필요로 함**을 보장하며, 단순히 영리한 텍스트 추론이 아니다.

**Phase 2: Strong-to-Weak 증류** ([Section 4.3](../2511.21631.md#43-strong-to-weak-distillation))

```python
# Stage 1: Off-policy 증류
teacher_outputs = qwen3_teacher.generate(prompts)
student.train(prompts, teacher_outputs)

# Stage 2: On-policy 증류
student_outputs = student.generate(prompts)
teacher_logits = qwen3_teacher.get_logits(prompts, student_outputs)
student_logits = student.get_logits(prompts, student_outputs)
loss = KL_divergence(student_logits, teacher_logits)
student.optimize(loss)
```

**왜 텍스트 전용 증류?**
- LLM 백본의 추론 능력 개선에 집중
- 교사에서 학생으로의 멀티모달 분포 변화 방지
- 언어 이해 전이에 효과적임이 Qwen3에서 입증됨

**Phase 3: 강화학습**

**Reasoning RL** ([Section 4.4.1](../2511.21631.md#441-reasoning-reinforcement-learning))

```python
tasks = ['math', 'code', 'logic', 'grounding', 'visual_puzzles']
data_size = 30K_queries  # 쉬운 샘플 (>90% 통과율) 필터링 후

for task in tasks:
    # 쿼리당 16개 응답 샘플링
    responses = model.sample(query, n=16, temperature=0.7)

    # 결정론적 보상 (작업별 검증자)
    rewards = verify(responses, ground_truth)

    # SAPO 최적화
    policy_gradient = compute_sapo_gradient(responses, rewards)
    model.update(policy_gradient)

# 사전 정의된 비율로 다중 작업 배치
batch = mix_tasks(tasks, ratios=[0.3, 0.25, 0.2, 0.15, 0.1])
```

**보상 시스템**:
- **수학**: 테스트 케이스에 대해 답변 실행
- **코드**: 단위 테스트 실행
- **논리**: 논리적 일관성 검증
- **그라운딩**: 정답 바운딩 박스와 IoU
- **퍼즐**: 규칙 기반 검증

**왜 SAPO?** ([Reference](../2511.21631.md#ref-21))
- 부드럽고 적응적인 정책 그래디언트 방법
- 다양한 작업에 걸쳐 일관된 개선
- 다른 모델 크기 (2B - 235B) 및 아키텍처 (dense, MoE)에 잘 작동

**General RL** ([Section 4.4.2](../2511.21631.md#442-general-reinforcement-learning))

```python
tasks = ['vqa', 'captioning', 'ocr', 'document_parsing', 'grounding']

# 하이브리드 보상 시스템
def compute_reward(response, reference):
    rule_based_reward = check_format_and_instructions(response)
    model_based_reward = qwen2_5_vl_72b_judge(response, reference)

    # 가중 조합
    return 0.6 * rule_based_reward + 0.4 * model_based_reward

# 목표 오류 수정 데이터셋
error_dataset = filter_prompts_triggering_known_errors([
    'language_mixing',
    'excessive_repetition',
    'formatting_errors',
    'clock_time_recognition',  # 알려진 약점
    'counter_intuitive_counting'
])

# 이러한 특정 오류에 대한 높은 빈도 페널티
model.train_rl(error_dataset, penalty_weight=2.0)
```

**왜 하이브리드 보상?**
- **규칙 기반**: 검증 가능한 작업 (형식, 지시 따르기)에 대한 고정밀, 보상 해킹 방지
- **모델 기반**: 개방형 작업에 유연, 미묘한 기준 처리 (유용성, 스타일)

**목표 오류 수정**: ([Lines 370-373](../2511.21631.md#L370-L373))

General RL만으로는 드문 오류에 샘플 비효율적. 해결책: **알려진 오류를 트리거하는** 프롬프트의 전용 데이터셋을 큐레이션하고, 이러한 행동을 억제하기 위해 높은 빈도 페널티를 적용.

## 출처 참조

- **아키텍처**: [Section 2](../2511.21631.md#2-model-architecture)
- **사전학습 파이프라인**: [Section 3](../2511.21631.md#3-pre-training)
- **사후학습 파이프라인**: [Section 4](../2511.21631.md#4-post-training)
- **학습 설정 표**: [Table 1](../2511.21631.md#L172)
- **Vision Encoder 제거**: [Table 11](../2511.21631.md#table-11)
- **DeepStack 제거**: [Table 12](../2511.21631.md#table-12)

## 계산 효율성 통찰

### 점진적 컨텍스트 확장

256K 컨텍스트에서 직접 학습하면 엄청나게 비용이 많이 든다:
```
Cost_256K_from_scratch ≈ 16× Cost_8K  (이차 주의 복잡도)

점진적 접근:
Cost_8K (S0+S1) + Cost_32K (S2) + Cost_256K (S3)
≈ 1T tokens @ 8K + 1T tokens @ 32K + 100B tokens @ 256K
≈ 1× + 16× + 1.6× (8K 베이스라인 대비)
Total ≈ 18.6× cost of 8K-only training

직접 256K:
2.1T tokens @ 256K ≈ 336× cost of 8K-only training
```

점진적 확장은 시작부터 최대 컨텍스트에서 학습하는 것에 비해 ~18× 컴퓨팅을 절약한다.

### 분기형 사후학습

동일한 사전학습된 베이스에서 두 가지 변형 (thinking + non-thinking)을 학습하는 것이 두 개의 별도 모델을 학습하는 것보다 효율적이다:
```
공유 비용: 4단계 사전학습 (~2.1T 토큰)
별도 비용: 2 × 사후학습 (~1.2M SFT 샘플 + RL)

vs.

완전 분리: 2 × (사전학습 + 사후학습)
```

공유 사전학습 (총 컴퓨팅의 ~90% 소비)은 두 변형에 걸쳐 상각된다.

### 제거 실험 전략

모든 제거 실험은 flagship 235B 모델이 아닌 Qwen3-VL-8B에서 수행됨. 이는 빠른 반복을 가능하게 한다:
```
8B training cost ≈ 1/30 × 235B training cost (파라미터 스케일링)
```

8B 스케일에서 아키텍처 검증 후, 개선을 확인한 후에만 235B로 확장.

## 구현 권장사항

### 이 작업을 복제하는 실무자를 위해

1. **SigLIP-2로 시작**: 대안 대비 +3.5 MMMU는 채택을 정당화
2. **제곱근 손실을 먼저 구현**: 모달리티 균형을 위한 간단하고 영향력 높은 기법
3. **점진적 컨텍스트 확장**: 처음부터 최대 컨텍스트에서 학습하지 말고 점진적으로 구축
4. **멀티모달 필요성 필터링**: VL 추론 데이터셋의 경우 텍스트로 해결 가능한 샘플 필터링
5. **하이브리드 RL 보상**: 규칙 기반 (정밀도)과 모델 기반 (유연성) 보상 결합
6. **목표 오류 수정**: RL을 위한 오류별 데이터셋 큐레이션, General RL에만 의존하지 말 것

### 이 작업을 확장하는 연구자를 위해

1. **DeepStack 일반화**: 더 많은 ViT 계층 주입 또는 더 깊은 LLM 계층 탐구
2. **컨텍스트 길이 효율성**: 256K는 인상적이지만 비용이 많이 듦—희소 주의, 검색 증강 접근법 조사
3. **Thinking 모드 효율성**: CoT가 언제 도움이 되고 해를 끼치는가? 쿼리당 이를 예측할 수 있는가?
4. **데이터 혼합 최적 비율**: 논문은 정확한 비율을 명시하지 않음—체계적 탐색이 결과를 개선할 수 있음
5. **대안적 RL 알고리즘**: SAPO가 잘 작동하지만, 멀티모달 RL에 더 나은 옵션이 있는가?

## 미해결 질문

1. **총 학습 비용**: GPU 시간과 USD로 얼마나 드는가? 복제 예산 책정에 중요
2. **추론 지연**: thinking 모드 오버헤드는? 256K 컨텍스트의 메모리 요구사항은?
3. **데이터 오염**: 50개 이상의 벤치마크에서 사전학습에 유출된 평가 데이터는 얼마나?
4. **실패 모드**: 모델이 치명적으로 실패하는 경우는? 적대적 강건성은?
5. **수확 체감?**: Qwen2.5-VL → Qwen3-VL 개선이 지속 가능한가, 아니면 포화 상태에 도달하는가?
