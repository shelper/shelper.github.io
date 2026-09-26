---
title: "From Next-Token Prediction to Persistent World Models: Notes on the Future of AI"
date: 2026-09-26
tags: ["artificial intelligence", "deep learning", "language models", "computer vision", "world models", "scene graphs"]
categories: ["technology"]
description: "A discussion that began with Transformer inference and diffusion language models, then led to a persistent, queryable world-state architecture for computer vision."
---

On September 2 and 3, I had a conversation that began with a practical question: if so many language models seem to share the same architecture, why can one inference engine run so many of them? It ended somewhere less practical and more interesting: perhaps the next step in AI is not one model that directly answers every question about its input, but a system that builds and maintains a model of the world, then lets other components reason over it.

This post reconstructs that progression. The language-model discussion and the computer-vision discussion were adjacent, but they were not one unified technical proposal at the time. The connection between them is my interpretation after following the questions through. I will distinguish what established systems do, what research is exploring, and what I would like to build.

## 1. Are language models all the same under the hood?

I started by asking whether open-weight and frontier language models are fundamentally similar. If `llama.cpp` supports many model families, does that mean those models all share essentially the same architecture?

At a useful level of abstraction, many leading text generators are decoder-only Transformers. They turn an input sequence into token representations, apply repeated attention and feed-forward transformations, and assign a distribution to the next token. That common structure makes reusable inference machinery possible.

But “Transformer” is a family name, not a binary format or a universal execution plan. Models differ in attention variants, positional encoding, normalization, dense versus mixture-of-experts layers, tokenizers, multimodal components, context handling, and training objectives. An inference engine must implement the actual operators and conventions for each supported architecture. `llama.cpp` supporting many open-weight models does not mean every closed frontier model can simply be loaded into it. GGUF is a model packaging format, not a statement that all model architectures are interchangeable. And an inference system such as `vLLM` answers a somewhat different operational question, emphasizing efficient serving and batching.

This led me away from model branding and toward the actual computation: what happens from the moment I submit a prompt to the moment I see the next token?

## 2. Prefill, decode, and the sequential boundary

An autoregressive model factorizes a response into conditional predictions: `P(x₁, …, xₙ) = ∏ᵢ P(xᵢ | x₁, …, xᵢ₋₁)`.

Inference therefore has two different phases. During **prefill**, the model processes the known prompt and constructs attention state for its tokens. During **decode**, it repeatedly predicts a new token, appends it to the context, and predicts the next one. Time to first token includes prompt processing and other serving overhead; the later token rate reflects the decode loop.

The GPU performs enormous amounts of parallel arithmetic inside each forward pass. That does not remove the dependency between successive autoregressive outputs. Token `x_{i+1}` depends on the actual sampled `x_i`. More parallel hardware cannot simply compute an unknown future token as though it were already chosen.

The KV cache makes this process tractable. In a conventional causal Transformer, the keys and values from earlier positions are retained so that a new token can attend to them without recomputing the entire prefix. The query for the current step is computed from the current hidden state; it is used for this attention calculation rather than stored as a growing history of queries. This distinction matters because cache memory grows with context length, layer count, and the dimensions of the stored K/V tensors. Architectural variants can change these costs, but they do not erase the basic tension between memory, throughput, and latency.

So I asked: if the GPU can evaluate many positions in parallel, why must generation advance exactly one token at a time?

## 3. Producing more than one token per expensive model pass

A first answer is **speculative decoding**. A cheaper draft model proposes several tokens. The larger target model evaluates the proposed block in one pass, accepting an appropriate prefix and handling a rejection according to the verification algorithm. Under the standard exact sampling formulation, this can preserve the target model's output distribution while reducing the number of serial target-model passes. The advantage depends on draft quality, verification overhead, hardware, and workload.

A rough sketch:

```text
Draft model:    proposes     A  B  C  D
Target model:   evaluates    A  B  C  D  together
Sampler:        accepts a valid prefix; corrects at rejection
```

The target model still needs to condition on preceding positions; the trick is that the proposed positions are already available as inputs to a parallel verification pass. This is different from asserting that an ordinary autoregressive model can independently guess four future tokens with no conditioning.

There are related approaches that train models to predict multiple future positions or use extra prediction heads. Those are worth exploring, but “multi-token prediction” is an umbrella term: a model predicting multiple candidates, a separate draft model, and exact speculative sampling are not the same algorithm.

The larger lesson for me was that **the mathematical dependency of a model and the implementation schedule of its inference system are separate design choices**. Sometimes a system can preserve the original distribution while changing how work is grouped. Sometimes it changes the modeling objective itself.

## 4. Diffusion language models: revising a whole block

That distinction brought us to diffusion language models. Instead of defining generation exclusively as left-to-right next-token prediction, masked discrete diffusion models can start with unknown positions and iteratively fill or revise a sequence:

```text
[MASK] [MASK] [MASK] [MASK] [MASK]
  idea  [MASK]  is   [MASK] clear
  idea    now   is     more  clear
```

The cartoon is intentionally simplified. The training corruption process, sampling schedule, confidence policy, and which positions can be revisited all affect the actual model.

LLaDA is an example of a language model trained with a forward masking and reverse prediction process. Dream 7B is another example of diffusion-style language generation. They demonstrate that left-to-right token sampling is not the only viable way to train a capable language model. They do **not** establish that diffusion is universally faster or better than autoregressive generation.

There is a subtle but important limitation: predicting many token positions in one iteration does not mean obtaining a complete answer in one forward pass. A diffusion model may require many refinement iterations, and each iteration may process a substantial block. The real latency question is the total number and cost of sequential refinement steps, as well as how well those steps use parallel hardware. Speed, quality, controllability, long-context behavior, and serving efficiency must be measured, not inferred from the word “parallel.”

Still, the ability to fill a whole block and revise uncertain regions suggests a different computational shape. Left-to-right generation commits early and extends a prefix. Iterative refinement can potentially use information from both sides of a position and reconsider parts of an answer. That is interesting for tasks where global consistency matters.

## 5. Could reasoning be different from writing?

At this point the conversation stopped being only about decoding speed. If a model writes one token after another, are we accidentally making the text stream itself the principal workspace for reasoning?

A language model can, of course, plan while producing autoregressive text, and there is no evidence that replacing its decoder automatically creates better reasoning. But the design space is wider than “think by emitting the final words in order.” A possible system might form a coarse internal plan, test or revise it, and only then render a response:

```text
Problem → candidate internal state → evaluation and revision → language
```

That internal state could be latent representations, structured steps, tool results, search traces, or some mixture. Diffusion generation is one possible mechanism for refinement, not a complete theory of planning. Likewise, an autoregressive model can already be part of an iterative planner that uses external memory and tools.

The question I found productive was therefore not “will diffusion replace Transformers?” Diffusion language models can themselves use Transformers. The better question is: **which parts of intelligence should be learned inside the model, which parts should be stored between interactions, and which parts should be explicit operations over that stored state?**

That question brought me back to computer vision.

## 6. The computer-vision version of the same problem

In a camera system, we often build a separate pipeline for each business question: detect something, track it, classify a scene, infer an event, and immediately issue an application-specific answer. But imagine asking several questions about the same environment:

- Where is a particular box now?
- Which boxes are on a shelf?
- Did a box move from one zone to another?
- Was an item placed inside another item?
- What happened while the object was occluded?
- Which conclusions are uncertain and need verification?

If every question has a separate model and a separate memory of the video, the system repeats work and can give inconsistent answers. My instinct was to represent objects, their relationships, and their changes in a shared graph. Then many applications might become queries over a maintained state.

This is related to scene graphs, dynamic scene graphs, object-centric perception, tracking, mapping, and world models. Those are established research directions. My particular engineering interest is a **persistent, queryable representation layer** between neural perception and downstream applications. It is not merely a graph neural network that consumes a single image, nor does it require a graph database.

A useful outline is:

```text
Sensors → observations → association and state updates
                         ↓
                persistent world state
                         ↓
           queries, reasoning, applications
```

The hard part is the arrow into persistent state. A detection is evidence about the world, not the world itself.

## 7. Observation is not entity

Suppose a camera detects a box at time `t`. That creates an **observation**: a sensor reported a candidate box at a position with some measurements and uncertainty. An **entity** is the system's continuing hypothesis that this is a particular physical box.

The same entity can generate many observations. One observation may be ambiguous between several entities. An entity can persist while it is temporarily invisible. If a box disappears behind another object, deleting its node from the world model immediately would confuse “not observed” with “does not exist.”

This suggests several separate fields or concepts:

- **Visibility:** currently visible, occluded, outside coverage, or lost.
- **Identity confidence:** how strongly an observation matches an existing entity.
- **Estimated state:** position, container, zone, and other properties, possibly predicted.
- **Provenance:** which measurements and inference steps support the belief.
- **Time:** when the observation occurred and when the system incorporated it.

The representation should also admit **unknown**. A camera system that cannot distinguish two possible boxes should be able to keep both candidates, or defer identity, rather than manufacturing certainty. The details of maintaining multiple hypotheses are a further design problem; they should not be silently collapsed into a single best guess.

## 8. Relation is not event

Another distinction emerged as we discussed the graph: a relation describes a state that holds over an interval, while an event describes a change or occurrence.

`Box A is on Shelf 2` is a relation. `Box A was placed on Shelf 2 at 10:32` is an event that might create, confirm, or revise that relation. `Box A is no longer visible` is an observation or visibility change, not necessarily an event saying it was removed from the shelf.

In a static image system, one can define node and edge schemas. Candidate-generating **connectors** propose relevant pairs using geometry, spatial indexes, types, or other rules. **Filters** evaluate whether a candidate relation holds. The result is a scene graph for that image.

Video changes the architecture. Rebuilding a complete independent graph for every frame throws away the continuity we are trying to capture. A tracker can operate at high frequency over detections and trajectories. Typed event detectors can inspect those trajectories and emit meaningful changes. A state-update component applies evidence and events to the persistent graph.

For example:

```text
Observation: box candidate near shelf
Tracking:    trajectory associated with entity B
Event:       B placed on shelf S
Update:      add/confirm relation B --on--> S
Query:       where is B?
```

The update must remain revisable. A later camera angle or label read could reveal that the earlier match was wrong. Good state management preserves the evidence and correction history instead of quietly rewriting the past.

## 9. Current state, history, and query are different products

A world-state system needs at least three views:

1. **Current belief:** the best available estimate of entities, properties, relationships, and uncertainty now.
2. **Temporal history:** observations and meaningful state changes, with enough timing and provenance to inspect or correct earlier inferences.
3. **Query interface:** a stable way for applications to ask about the present, the past, and unresolved alternatives.

One implementation could use a materialized current graph plus a log of semantic updates and occasional snapshots. Event sourcing is a possible pattern, but it does not imply storing only events or reconstructing every query from the beginning of time. High-rate raw trajectories can have different retention and indexing needs from semantic events.

The API boundary is valuable even before the inference system is perfect. A downstream application can ask “where is this box?” and receive a state, supporting evidence, and uncertainty. It should not have to own camera calibration, tracking, re-identification, or occlusion handling. Domain-specific business rules can then consume the answer: an application decides what to do if a box is likely in the wrong zone, while the camera system remains responsible for its best estimate of the physical world.

This separation also makes evaluation clearer. We can measure perception and state quality independently from whether a particular business workflow made the right decision.

## 10. What is a world model here?

“World model” can mean many things: a learned simulator that predicts future sensory observations; a latent dynamics model for planning; a semantic map for a robot; or a structured estimate of the current environment. My proposal is closest to the last of these, while borrowing ideas from the others.

It need not claim to understand all physics or predict everything. A useful first version might represent only the objects, locations, relationships, visibility, and events required by a few concrete camera applications. Its value comes from maintaining identity and state across time and exposing that state consistently.

Nor is the representation a magical replacement for neural networks. Learned models still provide detection, segmentation, recognition, association cues, language understanding, and perhaps event proposals. The explicit state layer gives those outputs continuity, provenance, constraints, and queryability.

The architecture is also not a reason to force every piece of information into nodes and edges. Dense images, embeddings, motion tracks, spatial indexes, probability distributions, and event logs may each belong in different stores. “Graph” describes the object-and-relationship interface, not an obligation to use one physical database or one universal data structure.

## 11. The connection I now see

The September 2 language-model conversation and the September 3 vision conversation converge on a systems question: **where should intermediate understanding live?**

A conventional text generator often keeps much of its working state inside a finite context and emits a serial answer. Diffusion language models explore alternative ways to construct an output, including iterative refinement. Speculative decoding shows that even when the target distribution remains autoregressive, systems design can change the execution schedule.

In video, a camera model that maps each frame directly to an answer also leaves important intermediate understanding implicit. A persistent state layer makes some of it explicit: “this is the same box,” “it is probably behind the shelf,” “this relation began at this time,” and “the evidence does not resolve these two alternatives.”

The analogy has limits. Faster token generation does not prove the need for scene graphs. A structured graph does not solve abstract reasoning, and a diffusion model does not automatically maintain a persistent world. These are different research and engineering problems. Their shared lesson is narrower and more useful: **do not assume one end-to-end model call, one output sequence, or one frame-local prediction is the only sensible unit of intelligence.**

For my work, the practical direction is to build from a few well-defined scenarios and data sets. Start with a small ontology, make identity and time first-class, keep uncertainty visible, and expose queries that actual applications need. Evaluate failures under occlusion, re-identification, delayed evidence, and conflicting observations. Then expand the representation only where the next use case requires it.

## Closing thought

I began by wondering why apparently different language models could run on similar infrastructure. I ended by wondering what an AI system should remember once the model has finished one forward pass.

Those questions are connected by a shift in perspective. A model is a powerful component, but the behavior we care about comes from the whole loop: sensing, proposing, checking, remembering, revising, querying, and acting. The future may include better Transformers, diffusion models, hybrid generators, and new latent reasoning methods. In parallel, it may depend just as much on building systems that can maintain an accountable, revisable picture of the world over time.

That is the part I want to explore next.

### Further reading

- [Fast Inference from Transformers via Speculative Decoding](https://proceedings.mlr.press/v202/leviathan23a.html) — exact speculative sampling and the serial decoding bottleneck.
- [Large Language Diffusion Models (LLaDA)](https://arxiv.org/abs/2502.09992) — masked diffusion language modeling.
- [Dream 7B: Diffusion Large Language Models](https://arxiv.org/abs/2508.15487) — another diffusion language model.
- [Kimera: From SLAM to Spatial Perception with 3D Dynamic Scene Graphs](https://arxiv.org/abs/2101.06894) — a concrete spatial and semantic dynamic graph system.
