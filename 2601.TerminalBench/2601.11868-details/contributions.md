---
parent: ../2601.11868-analysis.md
source: ../2601.11868.md
---

# Core Contributions - Deep Analysis

## Contribution 1: Terminal-Bench Framework & Dataset

**What**: A containerized, outcome-driven evaluation framework with 89 expert-verified tasks spanning 10+ categories (software engineering, ML, cybersecurity, scientific computing, system administration, video processing, personal assistant, etc.). Each task includes a Docker image, natural language instruction, test suite, and human-written oracle solution.

**Why it matters**: Prior benchmarks either target narrow domains (SWE-Bench = GitHub issues only) or use synthetic environments. Terminal-Bench evaluates the kind of high-skill work professionals are actually paid to do, in the actual interface they use. The "outcome-driven" design (testing final container state, not agent commands) allows diverse solution strategies.

**Key design decisions**:
- Tasks are containerized for reproducibility, but agents can access the internet for realism
- Crowd-sourced from 93 contributors for diversity, then filtered through rigorous 3-round review
- Time estimates from task authors provide human difficulty calibration
- Tasks range from < 1 hour (expert) to 24 hours (fix-ocaml-gc)

**Source**: [Section 2](../2601.11868.md#2-terminal-bench)

---

## Contribution 2: Comprehensive Multi-Agent Evaluation

**What**: Evaluation of 6 agent scaffolds across 16 frontier models (both proprietary and open-weight), totaling 32,155 trials with at least 5 runs per model-agent-task combination.

**Why it matters**: This is one of the largest systematic evaluations of agentic LLMs to date. Key findings:
- **Model > Agent**: GPT-5.2 vs GPT-5-Nano on Codex CLI shows 52% improvement; model choice matters more than scaffold
- **Proprietary > Open-weight**: Top 13 positions all proprietary; best open-weight (Kimi K2 Thinking) at 36% vs 63% top
- **Performance trajectory**: SOTA nearly doubled in 8 months (Gemini 2.5 Pro era to GPT-5.2)
- **No correlation**: Neither turn count nor token usage correlates with success rate
- **Cost varies 100x**: $1 to $100+ per benchmark run depending on model

**Source**: [Section 3](../2601.11868.md#3-experimental-setup), [Section 4](../2601.11868.md#4-results)

---

## Contribution 3: Dual-Level Error Taxonomy

**What**: Two complementary error analyses:
1. **Trajectory-level**: Adapted from MAST taxonomy into 9 failure modes (Execution, Coherence, Verification) with LLM judge (90% agreement with humans, Cohen's kappa 93%)
2. **Command-level**: Fine-grained taxonomy of 70+ failure types across 8 categories (Invocation & CLI, Filesystem, Environment, Build/Toolchain, Packages, Network, Runtime, Services) with 3,800 annotated failures

**Why it matters**: Provides actionable improvement directions:
- Frontier models (Opus 4.5, GPT-5.2): execution errors dominate -> focus on instruction adherence
- Open-weight models (Qwen Coder): balanced error profile -> need improvements across all failure modes
- Command-level: "not found on PATH" = 24.1%, suggesting environment setup is the #1 bottleneck

**Source**: [Section 4.4](../2601.11868.md#44-trajectory-level-error-analysis), [Section 4.5](../2601.11868.md#45-command-level-error-analysis)

---

## Contribution 4: Harbor Framework + Benchmark Adapters

**What**: Harbor - an open-source framework for building and running agent evaluations at scale. Includes:
- Standardized task format
- Multi-agent support (Claude Code, Codex CLI, Gemini CLI, OpenHands, Mini-SWE-Agent, Terminus 2)
- Scalable container orchestration (32-100 parallel containers via Daytona)
- 26 adapter integrations for existing benchmarks (SWE-Bench, CyBench, AppWorld, MLEBench, etc.)

**Why it matters**: The evaluation infrastructure fragmentation problem is real - each benchmark has its own runner, format, and agent interface. Harbor's adapter layer enables `harbor run -d terminal-bench@2.0` simplicity while supporting cross-benchmark evaluation. The 26 adapters mean researchers can evaluate one agent on 26+ benchmarks through a single interface.

**Source**: [Section 3.4](../2601.11868.md#34-harbor), [Appendix D](../2601.11868.md#d-adapters)

---

## Contribution 5: Terminus 2 Agent Scaffold

**What**: A deliberately simple agent scaffold with a single tool (headless terminal) that completes tasks using only Bash commands. Created as a "neutral testbed" for fair model comparison.

**Why it matters**: Existing agents (Claude Code, Codex CLI) are engineered for specific models and include sophisticated tools beyond basic terminal access. Terminus 2 strips away agent complexity to isolate model capability. The finding that Terminus 2 + Claude Opus 4.5 (58%) nearly matches Codex CLI + GPT-5.2 (63%) with a simpler scaffold is notable.

**Source**: [Section 3.1](../2601.11868.md#31-terminus-2), [Appendix F](../2601.11868.md#f-terminus-2)
