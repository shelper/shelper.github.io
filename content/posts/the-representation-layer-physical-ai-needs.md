---
title: "The Representation Layer Physical AI Needs"
date: 2026-09-26
tags: ["physical ai", "computer vision", "world models", "data architecture", "robotics", "agents"]
categories: ["technology"]
series: ["AI Beyond the Model"]
description: "How layered world representations, capability contracts, composition, and an observable runtime can connect sensors to agents."
---

*Part 2 of [AI Beyond the Model](/series/ai-beyond-the-model/). [Start with Part 1](/posts/from-next-token-prediction-to-persistent-world-models/).*

In the [first article](/posts/from-next-token-prediction-to-persistent-world-models/), I argued that a vision system should maintain a persistent belief about the world rather than answer every question independently from the latest frame. That leaves a harder engineering question: **what information should this system expose, and how should other models, applications, and agents use it?**

An image is rich but expensive to reinterpret. A single classification is cheap but often discards the context needed for a different task. A graph of objects and relationships is useful, but it is only one level of abstraction. My proposal is to build an *evolvable representation layer* spanning observations, entities, physical relationships, history, and semantic state, with explicit contracts for the capabilities that produce and consume each level.

This grew out of August conversations about AI infrastructure, perception stacks, agent harnesses, and robotics. It is a design thesis, not a claim that one universal ontology has already been discovered.

## 1. Stop looking for the one perfect abstraction

A camera can produce several kinds of information:

| Level | Example | What it preserves | What it makes easier |
| --- | --- | --- | --- |
| Sensor record | Image, depth, timestamp, calibration | Raw evidence | Reprocessing and debugging |
| Perception | Detection, OCR result, pose, trajectory | Measurements and uncertainty | Swapping perception algorithms |
| Entity | A particular box or label with a stable identity | Continuity across observations | Tracking through occlusion |
| Relation and event | Box on shelf; box placed on shelf | Structure and change | Temporal queries and constraints |
| Semantic state | Shelf occupancy; package status | Task-relevant meaning | Downstream decisions |
| Domain knowledge | Product catalog, shelf rules, workflow policy | External context | Application reasoning |

Each upward step compresses information. That can reduce token use and make an application more deterministic, but it can also erase evidence that a new model needs. I would therefore expose multiple levels through well-defined interfaces. A researcher may need image crops and candidate detections; an inventory application may need only a shelf occupancy answer with evidence and uncertainty.

This is why I use “sensor” broadly. A document pipeline that emits extracted fields is effectively a smart sensor to an upstream system. So is a tracking service that emits entity observations. The distinction is not whether a component contains a neural model. The distinction is the semantic level and contract of the information it provides.

## 2. State is a belief with provenance

To maintain a useful world representation, an object ID is not enough. A record should say what was observed, when it was observed, how it was associated with an entity, and which conclusions are inferred rather than measured.

For example:

```text
Observation:
  camera C saw a candidate box at capture time t
  crop, geometry, detector version, confidence

Entity belief:
  probably physical box B
  current location: shelf S, with uncertainty
  visibility: occluded
  supporting observations: o17, o23

Event:
  B was likely placed on S between t1 and t2
  inferred from trajectory and interaction evidence
```

The observation's capture time can differ from the time the system commits an update. A delayed label read may correct an association made several seconds earlier. If the system overwrites the old answer without recording the correction, it becomes impossible to understand why it changed.

Useful fields therefore include identity, coordinate frame, capture time, validity interval, source, model or rule version, confidence semantics, and evidence references. Confidence should describe a specific proposition; an uncalibrated detector score is not automatically the probability that a physical entity exists.

The representation must also permit missing information. “Not visible” is not “gone.” “No confident match” is not “new object.” “Unknown” can be the most accurate state.

## 3. Standardize meaning before transport

If a detector, tracker, OCR engine, and graph updater each use incompatible notions of identity, time, or confidence, connecting them with a message bus does not solve the integration problem.

I would define a **capability contract** for each reusable primitive: what its input and output mean, what metadata it supplies, what failures it can report, and which versions are compatible. Detection could be expressed in plain terms as:

```text
Capability: vision.detect.v1
Input: frame reference, coordinate frame, optional ROI
Output: detections with geometry, class candidates, capture time,
        source version, and quality information
Failure: unavailable, invalid input, or incomplete result
```

The contract says nothing about whether the implementation uses a small CNN, a VLM, a traditional algorithm, or a future model. It also says nothing about the transport. A local call, gRPC service, or agent-facing protocol can expose the same semantic capability. Protocols determine *how to invoke it*; the contract determines *what an invocation means*.

The representation itself needs versioning and migration. New sensors and tasks will require new fields and relationships. Schema evolution is therefore not administrative overhead: the choice of representation can change accuracy, latency, debugging cost, and how many reasoning steps an agent needs. **The representation is part of the optimization problem.**

## 4. Modular components still need a runtime

In one discussion, I asked what makes an AI harness more than a collection of plugins. The answer is that replaceable components and an executable workflow solve different problems.

The contracts define the available operations. A runtime selects implementations, connects stages, handles backpressure and failures, and records what happened. A composition grammar describes how a particular Skill combines bounded operations: order, branches, parallel work, checkpoints, retries, escalation, and success criteria.

One possible camera workflow is:

```text
Frame → Detect → Associate → Update entity state
                         ├→ OCR when label is visible
                         ├→ Event detector on trajectories
                         └→ Queryable scene state
```

Some steps run for every frame; others only when an event or uncertainty warrants them. This is more flexible than treating an entire application as one opaque “Skill.” An agent can inspect and change the workflow only if the stages have typed boundaries and measurable outcomes.

I would keep the latency-sensitive frame path near the camera or on the same device. Later stages may consume compact observations rather than full image buffers. That makes an in-process processing graph, a distributed service, or a hybrid deployment possible behind the same contracts. **Frames stay local when possible; useful observations can travel.** This is a design choice to validate under actual load, not a rule that every deployment must follow.

## 5. The event log and the current graph do different jobs

A current scene graph answers “what do we believe now?” A record of observations and changes answers “how did we get here?” Keeping only the graph loses the ability to explain or correct a decision. Rebuilding the graph from raw video for every query is costly.

A practical design could keep:

1. A materialized current state for low-latency queries.
2. An append-only record of observations, association decisions, semantic events, and corrections.
3. Optional snapshots and retained media references for replay or independent review.

The log enables an investigation such as: which tracker version first associated this observation with box B? Did a later OCR reading overturn that match? It also supports regression tests: replay the same evidence through a new algorithm and compare the resulting state.

An event log is not automatically ground truth. It records what the system believed and did, often with uncertainty. Likewise, event sourcing is an implementation pattern, not a requirement to place every video frame on a durable event bus.

Observability belongs at both the component and workflow level. We need to know whether an incorrect business answer originated in detection, association, a stale relation, a query, or a downstream rule. An AI system cannot improve a workflow it cannot measure.

## 6. Why physical AI needs a common substrate

Robots may have different bodies, cameras, and control loops, yet many tasks need to reason about the same kinds of things: objects, places, agents, actions, and changes over time. A device-independent entity and event layer could let a planner ask about the world without depending on the internal format of one camera or robot.

That does not imply identical perception or control for every embodiment. Coordinate transformations, uncertainty, sensor coverage, and action affordances remain device-specific. Nor does a scene graph alone solve sim-to-real transfer. It can, however, supply a shared *comparison surface*: do the simulated and real systems produce compatible observations and state changes under the same task?

Work such as [Kimera](https://arxiv.org/abs/2101.06894) and [Hydra](https://arxiv.org/abs/2201.13360) demonstrates concrete spatial and semantic graph construction for robotics. My interest is the broader contract connecting those representations to applications, learned specialists, and evaluation across different devices.

## 7. How I would start

I would choose two or three real tasks that share objects but ask different questions. For example, one task asks where a box is, another asks whether it moved into a zone, and a third asks whether its identity is uncertain.

Then I would build the smallest useful vertical slice:

- Define observation, entity, relation, and event semantics.
- Connect one detector and one tracker through versioned contracts.
- Maintain current state plus an inspectable change history.
- Expose a handful of queries with evidence and explicit unknowns.
- Replay difficult cases, especially occlusion and late evidence.
- Compare a second implementation behind one contract.

Only after those tasks work would I generalize the schema or introduce agent-directed workflow changes. Otherwise the platform risks becoming an elegant vocabulary without a measurable use case.

## Closing thought

Foundation models will change. Detectors, trackers, and agent frameworks will change too. A representation layer becomes valuable if those components can exchange stable meaning while retaining the evidence needed to challenge and revise conclusions.

That layer is the connective tissue between raw perception and higher-level intelligence. It can support an AI that composes Skills, a human debugging a missed event, or a robot querying the state of its environment. But its usefulness is earned by precise semantics and measurable tasks, not by putting the word “graph” on every data structure.

Next: [how a graph can go beyond storing state and actively improve a vision model's predictions](/posts/a-graph-as-a-model-harness/).

## Series reading guide

1. [From Next-Token Prediction to Persistent World Models](/posts/from-next-token-prediction-to-persistent-world-models/) — inference leads to a persistent world state.
2. [The Representation Layer Physical AI Needs](/posts/the-representation-layer-physical-ai-needs/) — shared semantics, contracts, history, and runtime.
3. [A Graph as a Model Harness](/posts/a-graph-as-a-model-harness/) — structured context corrects and teaches perception.
4. [An AI That Learns to Build Its Own Specialists](/posts/an-ai-that-learns-to-build-its-own-specialists/) — a foundation model develops and coordinates specialized capabilities.
5. [Can AI Learn How to Search?](/posts/can-ai-learn-how-to-search/) — the remaining challenge of discovering useful abstractions and strategies.

### Further reading

- [Kimera: From SLAM to Spatial Perception with 3D Dynamic Scene Graphs](https://arxiv.org/abs/2101.06894).
- [Hydra: A Real-time Spatial Perception System for 3D Scene Graph Construction and Optimization](https://arxiv.org/abs/2201.13360).
- [ROS 2 Intra-process Communications](https://design.ros2.org/articles/intraprocess_communications.html) — one example of the engineering trade-offs in moving sensor data between components.
