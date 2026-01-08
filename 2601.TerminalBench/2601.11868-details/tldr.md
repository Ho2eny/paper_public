---
parent: ../2601.11868-analysis.md
source: ../2601.11868.md
---

# TL;DR - Deep Analysis

## Summary

> Terminal-Bench 2.0 is a rigorously curated benchmark of 89 diverse, hard CLI tasks (covering software engineering, ML, cybersecurity, scientific computing, and more) where AI agents operate in Docker containers using only terminal commands. The best frontier model+agent pairing (Codex CLI + GPT-5.2) resolves only 63% of tasks, while open-weight models max out at 36%. A detailed dual-level error analysis reveals execution failures (especially missing executables and environment setup) as the primary bottleneck.

## Why This Matters

**For AI Researchers**: Terminal-Bench fills a critical gap between narrow coding benchmarks (HumanEval, SWE-Bench) and overly broad general-purpose evaluations. By targeting the terminal - the actual interface through which professional developers, sysadmins, and researchers interact with computers - it measures capability on economically valuable work. The 37%+ failure rate of the best models on expert-level tasks that take a human < 1 hour reveals fundamental limitations in current LLM agents.

**For Agent Developers**: The command-level error taxonomy is immediately actionable. The finding that 24.1% of all command failures are "command not found" suggests that better environment awareness and package management could yield significant performance gains without any model improvement. The trajectory-level analysis showing execution errors dominate over coherence/verification errors points to specific architectural improvements.

**For the Evaluation Community**: The Harbor framework and 26 benchmark adapters represent infrastructure that benefits everyone. The ABC checklist self-assessment (0.896, 2nd place) sets a standard for benchmark rigor.

**For Industry**: Claude Code's $1B revenue milestone cited in the paper underscores the economic stakes. Terminal-Bench quantifies exactly how reliable (or unreliable) these tools are on real professional tasks.

## Key Numbers

| Metric | Value |
|--------|-------|
| Tasks | 89 (from 229 submitted) |
| Contributors | 93 |
| Total trials | 32,155 |
| Best score | 63% (Codex CLI + GPT-5.2) |
| Open-weight best | 36% (Kimi K2 Thinking + Terminus 2) |
| #1 error type | Command not found (24.1%) |
| Reviewer-hours/task | ~3 |
| ABC score | 0.896 (2nd overall) |

## Source References

- Problem Statement: [Section 1](../2601.11868.md#1-introduction)
- Dataset Design: [Section 2](../2601.11868.md#2-terminal-bench)
- Main Results: [Section 4](../2601.11868.md#4-results)
- Error Analysis: [Section 4.4-4.5](../2601.11868.md#44-trajectory-level-error-analysis)
