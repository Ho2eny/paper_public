---
parent: ../2601.11868-analysis.md
source: ../2601.11868.md
---

# Key vs Non-Key Sections - Deep Analysis

## Must Read

### Section 2: Terminal-Bench (Task Formulation, Construction, Verification, Composition)
- **Why**: Defines the benchmark's core design philosophy. The outcome-driven formulation, crowd-sourcing + rigorous audit pipeline, and task diversity are what differentiate Terminal-Bench from other benchmarks.
- **Key Points**:
  - Tasks test final container state, not agent commands (Section 2.1)
  - 229 submitted -> 89 selected through multi-round review (Section 2.2)
  - Three verification criteria: Specificity, Solvability, Integrity (Section 2.3)
  - Tasks span 10+ categories with expert times from < 1h to 24h (Section 2.4)
- **Link**: [Section 2](../2601.11868.md#2-terminal-bench)

### Section 4: Results (Main Results + Error Analysis)
- **Why**: Contains the paper's primary empirical findings and the most actionable insights.
- **Key Points**:
  - Best: 63% (Codex CLI + GPT-5.2), open-weight best: 36% (Section 4)
  - Model selection > agent scaffold for performance (Section 4)
  - Performance nearly doubled in 8 months (Section 4.2)
  - 93.3% of human-hard tasks are also empirically hard; correlation r=0.436 (Section 4.3)
  - Execution errors dominate frontier models; balanced errors in open-weight (Section 4.4)
  - "Command not found" = 24.1% of all command failures (Section 4.5)
- **Link**: [Section 4](../2601.11868.md#4-results)

---

## Important

### Section 3: Experimental Setup
- **Why**: Understanding the evaluation methodology (agents, models, infrastructure) is necessary to interpret results correctly.
- **Key Points**:
  - Terminus 2 as neutral scaffold with single Bash tool (Section 3.1)
  - 6 agents evaluated: 3 commercial + 3 open-source (Section 3.2)
  - 16 models: GPT-5.2, Claude Opus 4.5, Gemini 3 Pro, Grok 4, etc. (Section 3.3)
  - Harbor framework for scalable evaluation with Daytona containers (Section 3.4)
- **Link**: [Section 3](../2601.11868.md#3-experimental-setup)

### Section 2.3: Verification (Detailed)
- **Why**: The quality control pipeline is a key differentiator. Understanding its depth helps assess result reliability.
- **Key Points**:
  - Automated solvability checks, contributor checklists, LLM-backed quality tools
  - Adversarial exploit agent to detect cheating vectors
  - Two additional manual auditors for final inclusion decision
  - ~3 reviewer-hours per task total
- **Link**: [Section 2.3](../2601.11868.md#23-verification)

### Appendix B: Quality Control (B.1-B.5)
- **Why**: Detailed breakdown of deterministic checks, LLM-backed tools, debug tools, and adversarial testing. Essential reading for anyone building benchmarks.
- **Link**: [Appendix B](../2601.11868.md#b-quality-control)

---

## Reference

### Section 5: Limitations
- **Why**: Honest assessment of data contamination risks, internet access variability, and remaining quality concerns.
- **Link**: [Section 5](../2601.11868.md#5-limitations)

### Section 6: Related Work
- **Why**: Positions Terminal-Bench against SWE-Bench, WebArena, AppWorld, OSWorld, MLGym-Bench, and shell-specific benchmarks.
- **Link**: [Section 6](../2601.11868.md#6-related-work)

### Appendix C: Trace Failure Description and Examples
- **Why**: Concrete examples of each failure mode with actual agent transcripts. Invaluable for understanding the error taxonomy.
- **Link**: [Appendix C](../2601.11868.md#c-trace-failure-description-and-examples)

### Appendix D: Adapters
- **Why**: Lists all 26 integrated benchmarks and the adapter architecture.
- **Link**: [Appendix D](../2601.11868.md#d-adapters)

---

## Deep Dive (Specialized Interest)

### Appendix E.2: Command Failure Mode Taxonomy
- **Why**: The full 70+ category taxonomy is a standalone reference for anyone studying agent failures in CLI environments.
- **Link**: [Appendix E.2](../2601.11868.md#e2-command-failure-mode-taxonomy)

### Appendix F: Terminus 2
- **Why**: Architecture details of the neutral agent scaffold. Useful for anyone building custom agents.
- **Link**: [Appendix F](../2601.11868.md#f-terminus-2)

---

## Skip

### Appendix E.1: Per-Model Sankey Diagrams (Figures 15-33)
- **Why**: 19 individual Sankey diagrams showing command failures per model. Highly repetitive - the aggregate Figure 9 captures the key patterns. Only useful if you're analyzing a specific model in depth.
- **Link**: [Appendix E.1](../2601.11868.md#e1-command-failures-across-models)

### Appendix H: List of Tasks
- **Why**: Task listing without substantive analysis. Better to explore tasks directly via the Terminal-Bench repository.
- **Link**: [Appendix H](../2601.11868.md#h-list-of-tasks-in-terminal-bench-20)
