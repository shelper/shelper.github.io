---
title: "Recursive Abstraction: Could Intelligence Be Fractal?"
date: 2026-11-18
slug: "recursive-abstraction-could-intelligence-be-fractal"
tags: ["intelligence", "abstraction", "fractals", "cognitive science", "AI agents", "graphs", "LLMs"]
categories: ["technology"]
series: ["AI Beyond the Model"]
description: "A hypothesis about how intelligence grows through repeated composition, encapsulation, and reuse of a small set of cognitive operations—and why the resulting structure might be fractal-like."
---

*This is an exploratory hypothesis, not an established theory of intelligence. It builds on my earlier thoughts about [graph-constrained micro-agents](/posts/graph-constrained-micro-agents/) and asks a more fundamental question: what makes intelligence scalable?*

## A surprisingly small set of operations

The more I think about problem solving across mathematics, computer science, engineering, and ordinary life, the more I notice a recurring pattern: we repeatedly solve complicated problems by assembling simpler operations, then hiding the resulting complexity behind a new abstraction.

Consider a digital computer. A few kinds of logic gates can be combined into arithmetic circuits, processors, operating systems, and applications. The software engineer who invokes a function does not need to reconstruct every transistor-level operation underneath it.

A similar progression appears in mathematics. A child initially works through arithmetic step by step. Eventually, arithmetic becomes a familiar cognitive unit; then algebra becomes a tool; and later, entire mathematical techniques can be invoked almost as single operations.

This suggests a hypothesis:

> **Intelligence may grow less by acquiring entirely new primitive operations than by recursively composing, encapsulating, and reusing existing ones.**

The claim is not that brains are literally digital logic circuits. Human cognition includes continuous, probabilistic, embodied, and massively parallel processes. Logic gates are an analogy for the possibility that *a relatively limited repertoire of reusable operations* can support open-ended complexity.

Turing completeness offers another suggestive analogy: remarkably small computational systems can express arbitrary computable procedures, given sufficient time and memory. But we must not confuse **computational expressiveness** with **intelligence**, efficiency, learning, or practical problem-solving ability. Universality tells us what could be computed in principle, not how a mind learns useful abstractions.

## The working-memory bottleneck

At any given moment, a person has limited attention and working-memory capacity. We cannot consciously manipulate every detail of a large engineering system at once.

Yet people build bridges, prove theorems, design chips, and coordinate organizations.

One major reason is abstraction. A hard-won procedure becomes a named concept or skill. We can then operate on that concept without explicitly replaying every lower-level step. Cognitive science calls a related phenomenon **chunking**: many elements are reorganized into a smaller number of meaningful units.

The crucial move is not merely forgetting details. It is creating an abstraction that preserves what matters for the next task, while allowing the details to be recovered when necessary.

For example:

```text
Compare two values
    ↓ compose
Find the maximum of a list
    ↓ encapsulate
Select the best candidate
    ↓ compose with other skills
Plan a sequence of decisions
    ↓ encapsulate
Reusable planning strategy
```

At every stage, yesterday's complicated reasoning can become today's simple operation. And those new operations can be combined again.

This is a mechanism by which **bounded local reasoning could produce increasingly sophisticated global behavior**.

## Repeating the same process at every scale

Here is where the idea becomes more speculative—and more interesting.

Suppose intelligent systems tend to use similar operations at different abstraction levels:

1. Combine available operations to solve a problem.
2. Identify a stable, reusable solution.
3. Encapsulate it behind a simpler interface.
4. Compose that interface with other capabilities.
5. Repeat at a higher level.

We can express the intuition informally as:

```text
A[n+1] = Abstract(Compose(A[n]))
```

Here, `A[n]` is a repertoire of operations available at one level; the next level emerges by composing and abstracting over them.

If the *same organizational rules* recur across levels, the resulting system might exhibit **functional self-similarity**: different scales look surprisingly alike when we ignore the details hidden by each abstraction.

This is the motivation for the word **fractal**.

But an important distinction matters. A hierarchy is not automatically a fractal. A mathematically fractal structure exhibits measurable scaling behavior or self-similarity; a stack of software layers does not qualify just because it has many levels.

So the defensible claim today is not “intelligence is a fractal.” It is:

> **Recursive abstraction may create scale-repeating computational motifs in intelligent systems. Under some conditions, those motifs might produce measurable fractal-like structure.**

Whether those conditions hold in real brains or large AI systems is an empirical question.

## Do language models offer a clue?

Transformer models learn internal representations while training on prediction objectives. One striking line of research examines what geometric structure emerges in those representations.

In [*Transformers Represent Belief State Geometry in Their Residual Stream*](https://arxiv.org/abs/2405.15943), researchers studied trained transformers on controlled hidden-state prediction problems and found learned representations with geometry related to the underlying belief states, including fractal structure in certain settings.

That is fascinating because the model was optimized for prediction, not explicitly instructed to draw a fractal or construct the researchers' theoretical geometry.

Still, we should resist drawing conclusions too quickly. These results do **not** demonstrate that arbitrary production-scale LLMs organize knowledge as fractals. Nor do they establish that recursive abstraction causes the reported geometry, or that fractality is necessary for intelligence.

A separate research direction—mechanistic interpretability—attempts to identify meaningful features and computational circuits inside neural networks. Such work reinforces the idea that learned representations can have discoverable structure, but meaningful features and circuits are not themselves proof of fractal organization.

The intriguing question is whether a model trained to predict complex environments naturally develops increasingly reusable representations—and whether the *computations that produce those representations* have comparable motifs across scales.

## Why graphs are an attractive representation

I have previously argued for a [graph-based model harness](/posts/a-graph-as-a-model-harness/) and, more recently, [graph-constrained micro-agents](/posts/graph-constrained-micro-agents/). Recursive abstraction provides another reason to explore that direction.

A graph node need not represent a single object. It may represent a complete subgraph: a skill, a plan, a mathematical operation, an agent, or a learned procedure. Its exposed interface can be simple even when its internal implementation is complex.

```text
High-level node: "Inspect Shelf"
    input: shelf image
    output: verified inspection report
    |
    +-- Detect objects
    +-- Read labels
    +-- Associate products and labels
    +-- Check consistency
    +-- Produce observations
```

At a higher level, “Inspect Shelf” can itself become one node in a store-audit workflow. At an even higher level, store audits become one component in an inventory-management strategy.

The same basic graph operations—select, connect, transform, validate, and compose—can recur at different levels. A graph engine can also support *zooming in* when an abstraction fails: expand the node, inspect its subgraph, and revise the lower-level operation.

This does not prove that intelligence is inherently graph-shaped. Graphs are a useful **explicit representation for experimenting with recursive composition**; the underlying intelligence might be represented differently.

## A possible path for small, growing agents

The hypothesis suggests a different way to scale agent systems.

Instead of continually enlarging a single model, could a system retain a modest base model but steadily accumulate **validated, callable abstractions**?

Imagine an agent that can:

- solve a bounded task using existing operations;
- turn a successful procedure into a typed execution subgraph;
- test it against examples and failure cases;
- publish it as a reusable skill with clear inputs, outputs, and constraints;
- combine that skill with others to solve higher-level tasks.

Its *available capabilities* might grow even when its base model parameters do not.

This is closely related to program synthesis, hierarchical reinforcement learning, skills/options, tool use, and the broader idea of cumulative learning. The novel architectural question for a graph-native harness is how to make abstraction creation, reuse, validation, and selective expansion into first-class operations.

There are substantial obstacles: skills can overfit, interfaces can conceal important assumptions, higher-level errors can amplify lower-level mistakes, and the graph or tool library can become expensive to search. Unlimited accumulation is not automatically useful intelligence.

Nevertheless, the approach offers a concrete experimental direction.

## What would make the hypothesis testable?

I would like to separate three claims, rather than bundle them into one:

**Claim 1 — Recursive abstraction improves capability.** A system with a fixed base model, but an expandable library of verified reusable skills, can solve increasingly complex tasks more efficiently than an otherwise comparable system without skill reuse.

**Claim 2 — Similar computation recurs across abstraction levels.** We can identify recurring computational motifs—such as composition, selection, validation, compression, and expansion—at multiple levels of the learned hierarchy.

**Claim 3 — The structure exhibits scale-invariant regularities.** Quantitative measurements across levels reveal scaling relations that go beyond the simple fact that a hierarchy exists.

The first can be tested using held-out compositional tasks and measurements of success rate, generalization, latency, and total compute cost. The second requires an operational definition of a “computational motif” and a method for comparing motifs at different levels. The third could use appropriate graph-scaling or statistical measurements, with rigorous controls and null models.

A successful result on Claim 1 would be valuable even if Claim 3 turns out to be false. We do not need a literal fractal for recursive abstraction to be a powerful engineering principle.

## The thought I want to keep exploring

I began with an intuition: great problem solvers in different fields often seem to use strikingly similar ways of thinking. Build from a few operations; combine; abstract; reuse; and move up a level.

This could help explain why human technologies evolve through layers of abstraction, why software engineering is so dependent on composability, and why agent systems gain capability when they can construct and reuse higher-level tools.

It might also help explain some kinds of self-organized structure in learned models. That final connection remains speculative.

For now, I want to preserve the idea in its simplest form:

> **Intelligence may be the cumulative process of turning computation into reusable abstraction, then applying the same process to those abstractions again.**

If a small set of rules repeatedly organizes cognition at different scales, a fractal-like pattern may be a consequence. The real scientific challenge is to discover those rules, measure the patterns, and determine what—if anything—they explain about intelligence.

*To be continued.*
