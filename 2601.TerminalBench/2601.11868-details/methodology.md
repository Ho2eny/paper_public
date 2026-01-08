---
parent: ../2601.11868-analysis.md
source: ../2601.11868.md
---

# Methodology - Deep Analysis

## Paper Type: Benchmark

## Approach Overview

Terminal-Bench follows a four-stage methodology: (1) task formulation and collection, (2) rigorous quality verification, (3) multi-agent evaluation, and (4) systematic error analysis.

The benchmark's design is **outcome-driven** rather than process-driven: tests verify the final state of the Docker container, not the specific commands the agent issued. This is a deliberate choice that allows diverse solution strategies and focuses on what matters (did the task get done?) rather than how it was done.

## Task Design

### Formulation

Each task consists of 5 components:

| Component | Description | Purpose |
|-----------|-------------|---------|
| **Instruction** | Natural language task description | Agent's input |
| **Docker Image** | Pre-built container with environment | Reproducibility |
| **Tests** | pytest-based verification of final state | Outcome evaluation |
| **Oracle Solution** | Human-written bash script | Solvability proof |
| **Time Limit** | Maximum execution time | Resource control |

### Collection Process

```
93 contributors -> 229 tasks submitted
    -> Automated checks (GitHub Actions)
    -> Contributor checklist
    -> LLM-backed quality tools
    -> Manual review (Round 1)
    -> Agent testing + adversarial exploit agent
    -> Two additional auditors (Round 2-3)
= 89 tasks in Terminal-Bench 2.0 (38.9% acceptance rate)
```

### Verification Criteria

| Criterion | Definition | How Verified |
|-----------|------------|--------------|
| **Specificity** | Tests pass iff container is in acceptable state | Manual review of instruction-test alignment |
| **Solvability** | Oracle solution passes all tests | Automated CI + manual verification |
| **Integrity** | No cheating shortcuts available | Adversarial exploit agent + manual audit |

## Evaluation Methodology

### Agent-Model Matrix

The paper evaluates a matrix of agents x models:

**Agents** (6):
- Commercial: Claude Code, Codex CLI, Gemini CLI
- Open-source: OpenHands, Mini-SWE-Agent, Terminus 2

**Models** (16 frontier):
- OpenAI: GPT-5.2, GPT-5, GPT-5-Mini, GPT-5-Nano, GPT-OSS-120B, GPT-OSS-20B
- Anthropic: Claude Opus 4.5, Claude Sonnet 4.5, Claude Haiku 4.5, Claude Opus 4.1
- Google: Gemini 3 Pro, Gemini 3 Flash, Gemini 2.5 Pro, Gemini 2.5 Flash
- Others: Grok 4, Grok Code Fast, Llama 4 Maverick, Qwen 3 Coder 480B, Kimi K2, GLM 4.6, MiniMax M2

**Protocol**: Each compatible model-agent combination run >= 5 times per task, 95% confidence intervals reported.

### Scoring

Binary pass/fail per task trial. A task is "resolved" if all tests pass. Resolution rate = (passed trials) / (total trials) for a model-agent pair.

### Terminus 2 as Neutral Baseline

Terminus 2 is deliberately minimal:
- Single tool: headless terminal (Bash only)
- No file editing tools, no web search, no custom tooling
- Purpose: isolate model capability from agent scaffold sophistication

## Error Analysis Methodology

### Trajectory-Level (Section 4.4)

| Aspect | Detail |
|--------|--------|
| **Taxonomy** | Adapted from MAST (Pan et al., 2025) |
| **Classes** | 3: Execution, Coherence, Verification |
| **Subcategories** | 9 failure modes (see Appendix C.1) |
| **Sampling** | 2 failed trials per model per task (Terminus 2 only) |
| **Annotation** | Docent + custom pipeline, 2 human annotators |
| **Inter-annotator** | 93% Cohen's kappa on 20-trial calibration set |
| **LLM Judge** | GPT-5 (high reasoning), 90% agreement with 120 human labels |

### Command-Level (Section 4.5)

| Aspect | Detail |
|--------|--------|
| **Taxonomy** | Novel, 70+ failure types in 8 categories |
| **Scope** | Individual command input-output pairs |
| **Error rates** | 9.2% (Grok 4) to 26.7% (GPT-OSS-120B) |
| **Sampling** | 3,800 failures uniformly sampled across tasks and models |
| **LLM Judge** | GPT-5 (medium reasoning for detection, 92.4% agreement; high reasoning for classification, 82.0% agreement) |

## Key Design Choices

| Choice | Decision | Rationale |
|--------|----------|-----------|
| Outcome-driven testing | Test final container state, not commands | Allows diverse solution strategies |
| Internet access | Allowed during evaluation | Realism (agents install packages, query docs) |
| Docker containerization | All tasks run in containers | Reproducibility and isolation |
| Crowd-sourced tasks | 93 contributors from diverse backgrounds | Diversity of domains and difficulty |
| Minimum 5 trials | Per model-agent-task combination | Statistical reliability |
| Binary scoring | Pass/fail on test suite | Simplicity, objective evaluation |

## Limitations of Methodology

1. **Binary scoring loses partial credit**: An agent 90% done gets same score as one that fails immediately
2. **Internet introduces non-determinism**: Package availability, API changes can affect reproducibility
3. **LLM-as-judge limitations**: 82-92.4% agreement means 8-18% of annotations may be incorrect
4. **Terminus 2 as baseline**: Its simplicity is a strength for model comparison but may disadvantage models that benefit from richer tools

## Source References

- Task Formulation: [Section 2.1](../2601.11868.md#21-task-formulation)
- Dataset Construction: [Section 2.2](../2601.11868.md#22-dataset-construction)
- Verification: [Section 2.3](../2601.11868.md#23-verification)
- Experimental Setup: [Section 3](../2601.11868.md#3-experimental-setup)
- Error Analysis: [Section 4.4-4.5](../2601.11868.md#44-trajectory-level-error-analysis)
- Quality Control Details: [Appendix B](../2601.11868.md#b-quality-control)
