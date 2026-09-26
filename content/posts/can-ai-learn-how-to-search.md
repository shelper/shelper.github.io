---
title: "Can AI Learn How to Search?"
date: 2026-09-26
tags: ["artificial intelligence", "meta-learning", "search", "inductive bias", "external memory", "agi"]
categories: ["technology"]
series: ["AI Beyond the Model"]
description: "Beyond selecting tools and training specialists: can an AI discover useful abstractions, search strategies, and ways to learn?"
---

*Part 5 of [AI Beyond the Model](/series/ai-beyond-the-model/). [Part 4 introduces the specialist-building system](/posts/an-ai-that-learns-to-build-its-own-specialists/).*

In the [previous article](/posts/an-ai-that-learns-to-build-its-own-specialists/), I imagined a general model that discovers repetitive tasks, creates specialized components, and coordinates them. That still leaves one of the deepest questions from my August discussion unanswered: **who designed the space in which the system searches for solutions?**

Choosing among existing models is one kind of intelligence. Training a new model for a specified task is another. Discovering a useful representation, an effective search rule, or a new way to decompose the task is more ambitious. That is the question I want to explore here.

This is an essay about research possibilities, not a claim that current systems have independently invented a general method for intelligence.

## 1. Abstraction makes some problems easier and others bigger

Imagine a system that can choose a detector. Give it the ability to choose a detector and tracker, and the number of combinations grows. Let it choose data preprocessing, memory, search algorithms, evaluation rules, and its own task decomposition, and the design space can become enormous.

The number of possibilities does not have to grow by a fixed geometric factor, but combinatorial growth is a real engineering concern. More freedom is only useful if the system has guidance about where to look. Otherwise an agent that can call more tools may simply spend more computation on unproductive paths.

Humans manage this through **inductive biases**: assumptions about which representations and methods are likely to fit a class of problems. We decide that a warehouse should be represented as objects and locations, that a camera stream needs identity over time, or that a game benefits from tree search. Those choices narrow the search space before optimization begins.

My question was whether AI can eventually discover and revise these biases for itself. Could it decide not just *which move to search for*, but *which search procedure to use*? Could it discover something analogous to Monte Carlo tree search (MCTS) in a new problem family without an engineer supplying that design?

There is a hierarchy of difficulty:

1. Search within a human-defined space.
2. Learn a policy that guides a human-defined search algorithm.
3. Invent useful reusable abstractions or heuristics within an evaluator-defined space.
4. Discover that the current representation, evaluator, or search procedure itself is wrong, and replace it.

The fourth claim is much stronger than the first three. Results at one level should not be presented as proof of the next.

## 2. What current examples do and do not show

[AlphaZero](https://arxiv.org/abs/1712.01815) learned strong game play from self-play and combined its neural network with search. Its success is remarkable, but its rules, action space, objective, and search machinery were supplied by researchers. It did not independently discover MCTS from an unrestricted universe of possible algorithms.

[DreamCoder](https://people.csail.mit.edu/asolar/papers/EllisWNSMHCST21.pdf) is interesting for a different reason: it learns a library of reusable program components and a search policy from a collection of tasks. That is a concrete example of improving the *language of solutions*, not merely fitting parameters inside a fixed program.

[FunSearch](https://www.nature.com/articles/s41586-023-06924-6) and [AlphaEvolve](https://arxiv.org/abs/2506.13131) demonstrate that language models can propose programs or algorithms and improve them through evaluators. These are meaningful steps toward algorithm discovery. Their evaluators, computational budgets, candidate formats, and task definitions still provide strong human-designed structure.

Taken together, they suggest an increasingly capable form of **model plus search plus memory plus evaluation**. They do not yet settle whether an AI can reliably identify the right abstraction and evaluation criterion for an unfamiliar real-world problem.

There may be no single universal search rule. The more valuable ability could be selecting and *adapting* a family of search processes according to feedback. For one task, a learned heuristic and beam search might work; for another, constraint propagation, simulation, or active data collection may be better.

## 3. Can a Transformer learn the meta-level?

I asked whether the Transformer architecture could in principle support that kind of self-directed method discovery, or whether a different paradigm is necessary.

I do not know of a proven architectural barrier. Transformers can represent algorithms and interact with external tools. But “can represent” is not “can learn efficiently,” and “can propose” is not “can verify.” The amount of computation required for open-ended algorithm search could be far beyond what is economical for most products.

This is why I favor a systems view. A model can generate hypotheses; an external runtime can execute them; a verifier can measure outcomes; memory can retain useful discoveries; and specialist models can handle repeated subproblems. The system's intelligence lies partly in how these pieces constrain and inform one another.

Even with this structure, the meta-level remains hard. A system optimizing a benchmark may discover an exploit of the evaluator instead of a useful method. If the evaluation criterion misses real-world failure modes, repeated self-improvement can make the system *better at the wrong thing*.

An agent harness should therefore expose not only tools, but also task definitions, provenance, budgets, outcome checks, replay, and independent tests. More autonomous search increases the value of reliable measurement.

## 4. What should live in the model's weights?

A related question I raised was whether a large model needs to carry so many facts in its parameters. If facts can live in retrieval systems, structured memory, and world state, perhaps the model could devote more of its capacity to transferable reasoning or learning procedures.

Some research interprets Transformer feed-forward layers as behaving in part like key-value memories ([Geva et al.](https://aclanthology.org/2021.emnlp-main.446/)). That is a useful clue, but it does **not** establish a clean division in which “FFN stores facts” and “attention does reasoning.” Knowledge and computation are distributed across layers and mechanisms. Simply shrinking feed-forward networks while retaining attention would not be a demonstrated way to preserve reasoning.

The more defensible systems question is: **which information should be parametric, and which should be external and revisable?**

| Information | Plausible home | Reason |
| --- | --- | --- |
| General patterns and procedures | Model weights, learned policies | Reuse across many tasks |
| Changing facts and documents | Retrieval or versioned stores | Update and inspect independently |
| Current physical state | Structured world model | Preserve identity, time, uncertainty |
| Recent attempts and failures | Episodic log | Replay and improve a strategy |
| Validated reusable tools | Capability registry | Invoke and test consistently |

[Retrieval-augmented generation](https://arxiv.org/abs/2005.11401) demonstrates one way to combine parametric and non-parametric information for knowledge-intensive tasks. It does not imply that all factual knowledge can be removed from the model or that retrieval alone supplies reasoning.

For a physical AI system, the distinction is especially clear. A model might know *how to reason about occlusion*, while a world-state store knows *which box is currently occluded behind which shelf*. The latter is specific, time-sensitive, and correctable. It does not belong only in static weights.

## 5. A model of learning, not merely a model of facts

The most ambitious version of my idea is a foundation model that “knows how to learn.” That phrase needs a concrete interpretation. Given an unfamiliar domain, the system should be able to:

1. Identify what can be observed and what remains latent.
2. Propose useful entities, variables, and relationships.
3. Define measurable tasks and discover where labels are missing.
4. Choose candidate solution methods and estimate their cost.
5. Run experiments that distinguish the candidates.
6. Package successful methods as reusable capabilities.
7. Detect when the assumptions stop holding and revise them.

That is more demanding than training a classifier. It asks the system to reason about representations, evidence, experiments, and its own limits. A domain-specific model can fit a local distribution efficiently; the meta-learner has to decide **which local structure is worth learning and how to tell whether it learned the right thing**.

This connects to my earlier “different domains have different shapes” intuition. I sometimes picture each domain as a differently folded region of a high-dimensional space. That is a metaphor, not a mathematical assertion that every business domain forms a tidy manifold. Its practical meaning is simpler: task structure matters, and a universal model should be able to exploit local structure rather than solve every instance from scratch.

## 6. How would we test genuine progress?

I would evaluate this idea across *families of domains*, not by showing that one agent optimized a single known benchmark. Give the system a collection of related tasks during development and then a held-out domain with unfamiliar data and altered rules.

Compare at least three approaches:

- a fixed large-model workflow with tools and prompts;
- a human-designed system with domain specialists;
- a system allowed to propose representations, specialists, and search procedures, subject to the same information and compute budget.

Measure time and cost to reach a target quality, quality on unseen cases, calibration of uncertainty, and how much human intervention was required. Track whether learned abstractions transfer to the next domain. If it only performs better after seeing many examples of essentially the same task, that is specialization, not evidence of a general learning method.

One especially informative test would change the *kind* of error. For example, a vision system first faces poor lighting, then identity swaps under occlusion, then a new shelf geometry. Does it keep applying the same data-augmentation recipe, or does it recognize that the third problem requires a different representation or search strategy?

The stronger claim is that it can learn when its own previous inductive bias is misleading. That requires honest uncertainty and evaluation outside the loop it optimizes.

## 7. What this suggests about AGI

I see learning-to-learn, abstraction discovery, and search-strategy selection as plausible ingredients of more general intelligence. They are not a definition of AGI and do not provide a reliable timetable.

There is still value in the narrower engineering milestones. An AI system that notices a recurring problem, invents a useful representation, proposes a cheaper specialist, validates it independently, and remembers when to use it would already be a significant advance. We can measure that without claiming it has solved all domains.

The direction I find compelling is not “make one model large enough to contain every answer.” It is to build a system that can **change how it looks for answers**: alter its representation, create tools, test hypotheses, preserve discoveries, and revise its methods when the environment pushes back.

## Closing thought

The earlier posts in this series dealt with model inference, world state, structured context, and specialist creation. Those components make a powerful architecture, but humans still supply most of its abstractions and measures of success.

The next question is whether AI can increasingly take responsibility for those choices without losing the ability to test them. That is a much harder problem than routing among small models. It may also be the most consequential one.

## Series reading guide

1. [From Next-Token Prediction to Persistent World Models](/posts/from-next-token-prediction-to-persistent-world-models/) — inference leads to a persistent world state.
2. [The Representation Layer Physical AI Needs](/posts/the-representation-layer-physical-ai-needs/) — shared semantics, contracts, history, and runtime.
3. [A Graph as a Model Harness](/posts/a-graph-as-a-model-harness/) — structured context corrects and teaches perception.
4. [An AI That Learns to Build Its Own Specialists](/posts/an-ai-that-learns-to-build-its-own-specialists/) — a foundation model develops and coordinates specialized capabilities.
5. [Can AI Learn How to Search?](/posts/can-ai-learn-how-to-search/) — the remaining challenge of discovering useful abstractions and strategies.

### Further reading

- [AlphaZero: Mastering Chess and Shogi by Self-Play with a General Reinforcement Learning Algorithm](https://arxiv.org/abs/1712.01815).
- [DreamCoder: Bootstrapping Inductive Program Synthesis with Wake-Sleep Library Learning](https://people.csail.mit.edu/asolar/papers/EllisWNSMHCST21.pdf).
- [Mathematical discoveries from program search with large language models (FunSearch)](https://www.nature.com/articles/s41586-023-06924-6).
- [AlphaEvolve: A coding agent for scientific and algorithmic discovery](https://arxiv.org/abs/2506.13131).
- [Transformer Feed-Forward Layers Are Key-Value Memories](https://aclanthology.org/2021.emnlp-main.446/).
- [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401).
