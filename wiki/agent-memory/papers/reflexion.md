---
title: "Reflexion: Language Agents with Verbal Reinforcement Learning"
tags: [agent-memory, papers, arxiv, reflexion, verbal-reinforcement, episodic-memory, self-reflection]
created: 2026-04-04
last_updated: 2026-04-04
---

## Overview

**Paper:** Reflexion: Language Agents with Verbal Reinforcement Learning
**Authors:** Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopalan, Karthik Narasimhan (Northeastern / Princeton)
**arXiv:** [2303.11366](https://arxiv.org/abs/2303.11366) — March 2023 (NeurIPS 2023)

## The Core Idea

Traditional reinforcement learning updates agent policy by computing gradients and adjusting model weights — an expensive, data-hungry process. Reflexion takes a radically different approach: **reinforce language agents through verbal reflection, not weight updates**.

When an agent fails a task, it verbally analyzes its failure and stores that analysis as **episodic memory**. On the next attempt, it retrieves this self-reflection and uses it to guide better behavior. No gradients. No fine-tuning. Learning through language.

The key insight: LLMs are already excellent at analyzing what went wrong and articulating why. Instead of wasting this analytical capacity, Reflexion harnesses it as a learning signal.

## Architecture

### Three Core Components

**1. Actor**
The main agent that produces action trajectories. Can be any LLM-based agent (ReAct-style, chain-of-thought, etc.). The actor attempts the task, potentially fails, and generates a trajectory of (action, observation) pairs.

**2. Evaluator**
A component that scores the actor's trajectory. Feedback can be:
- **Binary:** pass/fail (e.g., unit tests for coding tasks)
- **Scalar:** numeric reward signal
- **Free-form language:** detailed critique from another LLM

**3. Self-Reflection Module**
When the evaluator indicates failure, the self-reflection module generates a verbal analysis:
- What happened
- Why it went wrong
- What to do differently next time

This reflection is stored in an **episodic memory buffer**.

### The Learning Loop

```
Attempt task → Evaluate outcome → If fail: generate reflection
                                           ↓
                                    Store in episodic memory
                                           ↓
                               Next attempt: retrieve reflections
                                           ↓
                                    Try again with context
```

The agent accumulates up to N reflections in its memory buffer (the paper uses N=3). Each subsequent attempt starts with all prior reflections in context — the agent "remembers" what it learned from previous failures.

## Key Results

- **HumanEval (coding):** Reflexion achieves **91% pass@1** — surpassing GPT-4's previous SOTA of 80%
- **AlfWorld (sequential decision-making):** Reflexion with hallucination detection reaches **97% success** vs 78% baseline
- **HotpotQA (reasoning):** Consistent improvements across diverse question types

The coding result is particularly striking: the agent learned to write correct code not by training on more code, but by verbally reflecting on why its code failed tests and storing those insights.

## Memory as Self-Improvement

Reflexion uses episodic memory in a specific, powerful way: **experiences become improvement signals**.

Traditional episodic memory asks "what happened?" Reflexion's memory asks "what can I learn from what happened?" The stored memories are not raw experiences but **synthesized lessons** — distilled insights about failure modes and better strategies.

This is closely related to [[concepts/reflection-and-abstraction]]: the self-reflection step transforms a raw experience (failure trajectory) into a higher-level insight (principle for improvement). The reflection is stored and retrieved just like any other memory, but it's already abstracted to the lesson level.

## Types of Feedback

Reflexion handles multiple feedback types:
- **External, binary:** Passes unit tests or not
- **External, scalar:** Game score, reward signal
- **External, language:** Detailed natural language critique
- **Internal, language:** Self-generated critique via another LLM call

The flexibility to use any feedback signal is key: Reflexion is applicable to any task where you can evaluate success/failure, even approximately.

## Limitations

- Memory buffer is bounded (N=3 reflections in the paper) — prevents unbounded growth but may lose older lessons
- Requires multiple attempts per task — not suitable for one-shot, high-stakes applications
- Quality of reflection depends on the LLM's self-analysis ability — weaker models may reflect poorly
- Doesn't transfer learning across tasks — reflections from task A don't help with task B

## Comparison to RL

| Feature | Traditional RL | Reflexion |
|---------|---------------|-----------|
| Learning mechanism | Weight updates via gradients | In-context episodic memory |
| Sample efficiency | Low (requires many samples) | High (useful in just 1-3 trials) |
| Stability | Potentially unstable | Stable (no weight changes) |
| Cross-task transfer | Possible (via shared weights) | No (memories are task-specific) |
| Catastrophic forgetting | Yes | No (weights don't change) |
| Cost | Expensive (training) | Cheap (inference only) |

## Key Takeaways

- **Verbal reinforcement:** agents learn from failure through language reflection, not gradient updates
- **Episodic memory buffer:** stores linguistic self-analysis from past attempts; retrieved on future attempts
- **Memory as lessons, not logs:** reflections are already abstracted to "what to do differently" — not raw event logs
- **SOTA on HumanEval:** 91% pass@1, surpassing GPT-4's 80% — one of the clearest demonstrations of memory-as-improvement
- **Flexible feedback:** works with binary, scalar, or language feedback signals

## See Also

- [[papers/generative-agents]]
- [[papers/coala]]
- [[concepts/reflection-and-abstraction]]
- [[concepts/episodic-vs-semantic]]
- [[research/key-papers]]
