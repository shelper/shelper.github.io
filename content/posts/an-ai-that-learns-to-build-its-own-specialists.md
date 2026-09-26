---
title: "An AI That Learns to Build Its Own Specialists"
date: 2026-09-26
tags: ["artificial intelligence", "small models", "knowledge distillation", "edge ai", "agents", "meta-learning"]
categories: ["technology"]
description: "From cloud-to-edge distillation to a foundation model that creates, evaluates, and coordinates specialized models."
---

I have been thinking about a different way to organize AI. Instead of asking one ever-larger model to perform every task directly, what if the large model learned to **create, teach, select, and coordinate specialists**?

This idea emerged in several conversations. In July 2025, I considered a practical feedback loop in which a cloud model teaches a smaller model running on a device. In August 2026, I broadened that into heterogeneous AI infrastructure: large models working with detectors, trackers, 3D algorithms, and other specialized components. Finally, I asked a more ambitious question: could a general model learn to produce its own domain-specific models and workflows as it encounters new environments?

That last question is the heart of this post. Some ingredients already exist. The complete architecture remains a research direction, not a solved route to AGI.

## 1. The first version: cloud teacher, local student

Consider a device that receives customer questions. Sending every question to a powerful cloud model may be expensive, slow, or impractical when connectivity is poor. A small local model could answer common questions quickly. But how should it improve once deployed?

My original proposal was to let the small model operate normally, while sampling some requests for a cloud teacher. For each sampled request, collect the device model's answer and a more capable model's answer, then compare them. Curate the useful cases, train a better student centrally, evaluate it, and deploy an update.

```text
Real requests → local student → local response
        └────→ sampled cloud teacher → comparison and training
                                      ↓
                              evaluated student update
```

This resembles teacher-student distillation and shadow evaluation. Classical knowledge distillation can train a student against a teacher's softened output distribution. In many real products, access to teacher logits is unavailable, so the signal may instead be answers, structured labels, explanations, preferences, or feedback from an independent evaluator. These are distinct training methods, and the quality of teacher-generated data matters as much as its volume.

The platform would need more than a fine-tuning script. It would sample strategically, deduplicate near-identical requests, measure which task regions are underrepresented, track regressions, and decide when a new model is worth deploying. Privacy and permissions for customer data would have to be built into that loop.

The key idea is that **the cloud model can do expensive learning work intermittently so that the device model can do cheap inference repeatedly**.

## 2. From one student to a system of specialists

The next step came from computer vision and edge computing. A real-world system rarely consists of one model. It might need a detector, OCR, a tracker, segmentation, depth estimation, geometric calibration, a language interface, and business rules. Some components are neural models; others are deterministic algorithms. They have different latency, memory, reliability, and hardware requirements.

A large multimodal model is useful for exploring a new problem, interpreting unusual situations, proposing annotations, or designing a workflow. It does not follow that it should be invoked for every camera frame, nor that it is the best tool for precise geometry or a high-frequency tracking loop. A small detector or a conventional algorithm may be more accurate, cheaper, and easier to validate for a narrowly defined task.

That suggested a different infrastructure goal: make capabilities interchangeable behind clear contracts. A task should specify its inputs, outputs, confidence and failure semantics, timing requirements, and evaluation criteria. Then the system can choose a large model, small model, deterministic routine, or combination without forcing downstream applications to care which implementation produced a valid result.

This is what I mean by a **Skill** in this article: a reusable, measurable capability such as “read a shipping label,” “associate this detection with an existing box,” or “estimate whether a box entered a shelf zone.” A Skill is an interface and behavior with evidence of performance. It need not be a single prompt, a single model, or an explicitly coded agent.

## 3. Why might specialization work?

I described different domains as differently shaped regions of a large conceptual space. This “manifold” picture is an intuition, not a proof that every domain has a clean low-dimensional surface. It is useful because it highlights a practical point: a model that must handle every possible input spends capacity on distinctions irrelevant to a particular task.

Suppose a warehouse camera sees the same kinds of shelves, boxes, and labels every day. The local system faces a distribution much narrower than all possible images. A model trained specifically for that distribution may achieve the required accuracy with fewer parameters and less computation. It can also exploit constraints: fixed camera geometry, known shelf dimensions, a limited label format, and a known set of actions.

The trade-off is brittleness. A specialist can fail badly outside its training distribution. The architecture therefore needs a mechanism to detect uncertainty or shift, escalate difficult cases, and refresh the specialist. The large model's value may lie as much in managing that boundary as in solving the common cases.

There is an economic condition too. Creating and maintaining a specialist has a fixed cost. Specialization pays off when repeated use, lower latency, or improved quality exceeds that cost. For a rare and constantly changing task, calling a general model directly may be better.

## 4. The more ambitious idea: a model that builds models

Here is the proposal I find most interesting:

> A general model should learn to identify a domain, define the task boundaries, collect and improve the relevant data, choose an appropriate specialist architecture or algorithm, train or adapt it, test it, deploy it, and monitor when it should be revised.

This is substantially more than “a router sends requests to one of several models.” The router assumes the specialists already exist. I am asking whether the higher-level system can **decide that a new specialist ought to exist and produce a validated version of it**.

The process could look like this:

1. **Observe a repeated need.** A workflow repeatedly calls the large model for the same narrow operation, such as reading a particular label or recognizing an action in one type of camera view.
2. **Define the contract.** Specify the input distribution, desired output, error costs, confidence requirements, runtime budget, and what counts as an abstention.
3. **Acquire examples.** Combine real inputs, trusted labels, teacher suggestions, targeted human review, and carefully validated synthetic or augmented data.
4. **Choose a method.** A rule, geometric algorithm, retrieval index, fine-tuned small model, or a composed pipeline may each be reasonable. “Create a model” should not automatically mean gradient descent.
5. **Train and evaluate.** Measure the candidate on held-out data, difficult edge cases, distribution slices, and device hardware. Compare it with the existing system.
6. **Deploy behind a gate.** Route only appropriate cases to the specialist, retain a fallback for uncertainty, and watch performance after rollout.
7. **Revise or retire.** When conditions change, update the specialist, switch implementations, or return traffic to the general model.

This loop turns the large model into something like a **learning system designer**. It is asked not only to answer a question but to find the smallest reliable computation that can keep answering that class of questions.

The hard question is where judgment comes from. A teacher model can confidently produce wrong labels. A judge model can reward outputs that merely sound plausible. The system therefore needs independent measurements: ground truth where possible, physical constraints, downstream outcomes, human audits for ambiguous samples, and tests on data that was not generated by the same teacher.

## 5. Meta-learning at the system level

“Meta-learning” often refers to algorithms that learn how to adapt quickly across tasks. I am using the phrase more broadly here: the AI system learns **how to build and improve its own task-solving process**.

Given a new problem, it would ask:

- What is the recurring task, and what are its boundaries?
- Which evidence is needed to decide whether it was solved?
- Can a cheap specialist reach the quality target?
- Which examples would most reduce uncertainty?
- Which steps should be executed locally, and which should go to the cloud?
- When has the domain changed enough to require retraining?

These are engineering questions, not mystical properties of a large model. The general model can propose answers, but the platform needs typed interfaces, experiment tracking, data lineage, evaluation gates, deployment controls, and observability. Without that surrounding system, “the AI builds its own Skills” is a slogan rather than an architecture.

A useful analogy is a compiler. A compiler takes a higher-level intention and produces an executable plan under constraints. Here the “source program” is a task specification and examples; the outputs might be a prompt-based workflow, a distilled model, a detector plus tracker, or a deterministic algorithm. Unlike a traditional compiler, the result must be tested against uncertain data and updated as the environment changes. That makes it an **empirical compiler**, not a source-to-source translator.

## 6. Routing is a decision under cost and uncertainty

Once specialists exist, the system needs to decide who should handle each case. “Always use the smallest model first” is too simple. A mistaken local answer can cost more than a cloud call. “Always use the largest model” wastes time and money when the answer is routine.

For each request, the router should estimate both expected quality and cost. A useful conceptual objective is:

> choose the path with the lowest expected latency and compute cost, subject to an acceptable error rate for this kind of task.

Some requests can be resolved deterministically. Others go to a specialist. Uncertain or novel cases can go to the foundation model, a second independent checker, or a human. The route can also depend on the device's network state, remaining compute, and the consequence of being wrong.

A specialist's confidence score is not automatically a calibrated probability. The routing policy must be evaluated against observed outcomes, especially on unusual inputs. Otherwise the system will confidently send precisely the hardest cases to its weakest component.

There is also a feedback loop: escalated cases reveal where the specialist struggles. Those cases become candidates for annotation, diagnosis, and retraining. A well-designed router is therefore both an inference component and a data-collection strategy.

## 7. What an Agent OS would actually provide

I used the phrase **Agent OS** to describe the layer that would make this possible. I do not mean an operating system that magically makes a model autonomous. I mean a set of services that allow models and algorithms to be composed, evaluated, and updated with clear boundaries.

At minimum, it would need:

- **A capability registry** with typed inputs and outputs, versions, execution requirements, and known limits.
- **An orchestration layer** that calls components, enforces contracts, and handles fallback and retries.
- **A data and evidence layer** that records what was observed, what was inferred, by which component, and when.
- **Evaluation and monitoring** with task-level metrics, distribution slices, regressions, and drift detection.
- **A learning pipeline** for curating examples, training or adapting specialists, and comparing candidates.
- **Deployment controls** for progressive rollout, rollback, and model selection across cloud and edge hardware.

For a physical-world system, I would add persistent state: entities, identities, locations, relations, events, and uncertainty. A detector sees a box in one frame; a world-state layer determines whether it is the same box seen earlier and whether it remains present while occluded. Specialists operate on or contribute to that shared state. Applications query the state instead of reimplementing perception for every use case.

That connects this idea to my [earlier post on persistent world models](/posts/from-next-token-prediction-to-persistent-world-models/). The two ideas fit together: the world model provides shared, revisable facts; the specialist-building system decides which computations produce and consume those facts.

## 8. A concrete example: boxes in a delivery truck

Imagine a truck with several cameras and shelves. A large multimodal model can help inspect initial footage, suggest object categories, describe failure cases, and assist in designing the task. But a production system might ultimately use separate components for label reading, box detection, camera geometry, tracking, re-identification, and shelf occupancy.

The higher-level system might discover that label reading works reliably with a small OCR model in a fixed camera view, while re-identification across occlusion needs a more sophisticated model. It can test both claims on annotated footage. It can route ambiguous identities for review and preserve multiple hypotheses rather than inventing a single certain answer.

When enough reviewed cases accumulate, it can train a better specialist for this truck configuration. If lighting or packaging changes, performance monitoring can trigger investigation. The API exposed to an application remains simple: “Where is box X?”, “Which boxes are on shelf Y?”, or “Which answers need verification?”

The large model's most valuable job is not necessarily to watch every frame. It may be to **design the pipeline, teach and audit the components, and intervene when the pipeline encounters something it does not know**.

## 9. How this differs from familiar architectures

This proposal has family resemblance to several existing ideas, but the distinctions matter.

**Knowledge distillation** transfers behavior from teacher to student. It is one way to build a specialist; it does not decide which specialist to build or how to run a whole production system.

**Mixture of experts** routes activations among expert subnetworks inside a model. My proposal is about independently deployable models and algorithms with explicit task contracts, potentially running on different devices and evolving on different schedules. One architecture can include the other.

**Agent frameworks** coordinate tools and model calls. They can provide orchestration, but the difficult part here is the closed learning loop: task discovery, data acquisition, specialist creation, measurable evaluation, and safe replacement.

**AutoML and neural architecture search** automate parts of model selection and optimization. The broader proposal also decides whether a model is needed at all, how its output relates to shared state, and when the resulting capability should be invoked.

**A system that asks the large model to write a Skill** produces an implementation candidate. It has not yet shown that the Skill is correct. Reliability comes from contracts, tests, evidence, and monitored use.

## 10. What would prove the idea is useful?

The ambitious version is easy to describe and hard to establish. I would start with a narrow, repeated workload and compare three systems:

1. A capable general model doing the task directly.
2. A fixed, manually built collection of specialists.
3. A system that proposes, trains or configures, evaluates, and routes specialists under explicit constraints.

The third system should earn its complexity. It should improve at least one measurable outcome—quality, latency, cost, adaptability to a changed environment—without unacceptable regressions in the others. In particular, I would measure:

- how quickly a new task reaches a target quality;
- teacher and human labeling effort;
- edge inference cost and end-to-end latency;
- calibration, abstention, and error under distribution shift;
- how often the system detects a broken specialist before applications suffer;
- whether the improvement survives independent evaluation.

An important test is whether the system learns a *reusable method* for creating specialists, rather than merely memorizing one successful recipe for one dataset.

Even a partial success would matter. A platform that reliably identifies a repetitive cloud task, proposes a smaller alternative, gathers the right examples, and proves the replacement meets a quality threshold would already be valuable. It would not require the system to autonomously invent arbitrary new neural architectures.

## Closing thought

My view is that the interesting unit of future AI may be neither a single giant model nor a fixed set of hand-built microservices. It may be a **learning organization of capabilities**: a general model that can understand new problems, specialists that execute repeated work efficiently, and an infrastructure layer that measures what works and changes the organization over time.

The strongest version of this idea approaches meta-learning: a system that learns how to learn new domains, not just how to answer within domains it already knows. Calling that a path toward AGI is a hypothesis, not a conclusion. The engineering path is clearer: define contracts, collect evidence, create specialists where repetition justifies them, evaluate independently, and make the whole system capable of revising its own choices.

A large model does not have to be the worker for every task. Its most important contribution may be learning **which workers to build, what to teach them, and when to trust them**.

### Further reading

- [Distilling the Knowledge in a Neural Network](https://arxiv.org/abs/1503.02531) — the teacher-student distillation foundation.
- [Orca: Progressive Learning from Complex Explanation Traces of GPT-4](https://arxiv.org/abs/2306.02707) — a study of richer teacher signals and small-model learning.
- [Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer](https://arxiv.org/abs/1701.06538) — expert routing *inside* a model, useful for comparison.
- [DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines](https://arxiv.org/abs/2310.03714) — optimizing modular language-model programs.
