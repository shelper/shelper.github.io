---
title: "Graph-Constrained Micro-Agents: A Model Harness for Reliable Intelligence"
date: 2026-11-18
slug: "graph-constrained-micro-agents"
tags: ["agents", "graphs", "small language models", "event sourcing", "functional programming", "AI architecture"]
categories: ["technology"]
series: ["AI Beyond the Model"]
description: "A proposal for domain-specific agents built around graph-constrained reasoning, structured graph diffs, deterministic execution, and small language models."
---

*This article extends [A Graph as a Model Harness: Let Context Correct Vision](/posts/a-graph-as-a-model-harness/) from visual understanding to agent systems. It is an architectural proposal, not a claim that the proposed system has already been validated experimentally.*

## The question: do agents need to be general?

Much of the excitement around frontier AI is directed toward general-purpose models and agents: systems that can answer almost anything, use almost any tool, and operate across domains. That direction makes sense for broad, ambiguous tasks. But I suspect that a large part of the eventual agent ecosystem will consist of **many small, domain-specific agents** rather than a handful of universal ones.

Consider an inventory agent, a vision-inspection agent, a document-processing agent, or a maintenance diagnostic agent. Each operates in a relatively small world with recognizable entities, relations, constraints, and actions. We should not require each one to carry the full weight of general intelligence.

A larger model could still be valuable as a cross-domain orchestrator, especially when a task is ambiguous or demands unfamiliar knowledge. But even orchestration does not always require a large model: deterministic routing and graph-defined task dependencies can handle many routine cases. Escalate to a general model when the bounded system cannot resolve the request.

This raises a question more interesting than "how small can the model be?":

**Can a well-structured harness make a much smaller model sufficient for a useful class of tasks?**

## The graph is not just memory

An ordinary agent may have an LLM, a collection of tools, a prompt, and some memory. A graph-based agent is often described as an agent that *retrieves* information from a knowledge graph.

I want to go further. The graph should define the agent's **reasoning interface and operating space**, not merely supply facts.

For a domain-specific agent, the graph may encode:

- **Knowledge:** entities, attributes, observations, evidence, and relationships.
- **Permitted actions:** which operations can be proposed on which types of nodes.
- **State and constraints:** invariants, preconditions, and the distinction between hypotheses and confirmed facts.
- **Transitions:** how accepted events change the graph.
- **Task structure:** dependencies, intermediate results, and permitted exploration paths.

This is a stronger contract than retrieval-augmented generation. RAG helps decide *what context a model sees*. A graph-constrained harness can also decide *what relationships it may inspect, what proposals it may emit, and which changes the runtime may commit*.

That does not imply that a graph can control the neural model's private chain of thought. It can, however, constrain its **observable reasoning steps**, context requests, outputs, and effects on the world.

## Graph → linear input → graph diff

A language model consumes a token sequence, not an in-memory graph. So the key abstraction is a **Graph-to-LLM protocol**:

```text
          select relevant subgraph
World Graph ------------------------------+
    ^                                     |
    |                            Graph Linearizer
    |                                     |
    |                              serialized view
    |                                     v
    |                               Small LLM
    |                                     |
    |                        structured proposal / request
    |                                     v
    |                             Proposal Parser
    |                                     |
    |                              candidate graph diff
    |                                     v
    |                      Validator + Policy + Runtime
    |                                     |
    +---- apply accepted events <---------+
              append to Event Log
```

The linearizer should not naively dump all neighboring nodes. It selects a **task-relevant, bounded view**, with stable identifiers, typed edges, provenance, version information, and a clear set of available operations. The model can request another allowed slice of the graph if it needs more information.

The output should likewise be bounded. Instead of generating a whole new graph, the model emits **proposed operations** or a **candidate diff**. The runtime parses them, resolves identities, checks invariants and permissions, and either commits the accepted transition or returns a structured error.

A simple warehouse example makes this concrete.

```text
Context:
  shelf: shelf_12
  product: sku_42
  recorded_quantity: 5
  observation: camera_frame_804
  estimated_quantity: 3
Allowed proposals:
  ConfirmObservation | UpdateQuantity | RequestReview
```

The small model might propose:

```json
{
  "operation": "UpdateQuantity",
  "target": "shelf_12/sku_42",
  "value": 3,
  "evidence": "camera_frame_804",
  "expected_graph_version": 117
}
```

This is **not yet a fact**. The runtime must establish that the referenced entities and evidence exist, that the model is authorized, that the observation meets the policy for updating inventory, and that version 117 is still current. In some workflows an image-based count should only create a review candidate, not directly change the inventory of record.

The important separation is between **Graph State**, **Transformation Proposal**, and **Committed Event**. A model suggestion is never silently promoted into world state.

## Two graphs: world and execution

It helps to keep two different graphs:

1. The **World Graph** records what the system currently believes about the domain, including uncertainty and evidence.
2. The **Execution Graph** records the plan: tasks, dependencies, tool calls, guards, approvals, outputs, and outcomes.

The model may propose an execution subgraph, but the runtime owns validation and scheduling. Independent tasks can run concurrently; tasks with unmet dependencies cannot. External side effects, such as sending a command to a robot or changing a real inventory database, require explicit execution boundaries.

Events connect the graphs. An execution node may emit `ObservationAccepted`, `QuantityUpdateRejected`, or `ToolCallCompleted`; the corresponding projection changes the world view or execution status.

This architecture allows a powerful asymmetry: **probabilistic intelligence proposes; deterministic software governs which proposals become effects.** It limits damage from invalid outputs, although it does not guarantee that a valid proposal is semantically correct.

## Why functional programming and event sourcing matter

The desire behind this design is not merely clean syntax. It is **local reasoning**: understanding a component without mentally tracking hidden state changes across the entire system.

A functional view of graph evolution is:

```text
G_next = reduce(G_current, accepted_event)
```

The reducer is ideally deterministic and side-effect-free. Given the same graph version and accepted event, it produces the same next state. This makes state transitions explicit and testable, without depending on any particular implementation language.

**Event sourcing** complements it by storing a sequence of accepted events, not only the latest mutable state. Snapshots can make reconstruction practical. The event log supports audit, diagnosis, and replay; compensating events and versioned projections may be safer than pretending that every external side effect can simply be undone.

We should also be precise about replay. Replaying **recorded model outputs and tool results** through a deterministic reducer can reproduce a graph projection. Re-running a live LLM or an external tool may produce a different result. Record prompts or serialized graph views, model versions, proposed diffs, validation decisions, tool outcomes, and correlation IDs when reproducibility matters.

Persistent data structures can make successive graph versions cheap to represent through structural sharing. The cost and implementation details still need measurement, especially for large spatial or temporal graphs.

## The micro-agent as a typed component

If each domain agent publishes its graph schema and transition contract, a larger system can treat it like a typed software component:

```text
MicroAgentContract:
  input_graph_schema
  output_proposal_schema
  allowed_context_traversals
  allowed_operations
  preconditions_and_invariants
  evidence_requirements
  escalation_policy
  version
```

An orchestrator does not need to know how a micro-agent reaches a proposal. It needs to know which problems the agent accepts, what results it promises to produce, what evidence accompanies them, and how failure or uncertainty is reported.

This suggests a potential **Micro-Agent Factory**: provide a domain graph schema, operation vocabulary, constraints, example cases, and evaluation set; generate a bounded prompt/serialization layer, validators, and an agent harness around a suitably capable small model.

The result would not be automatically reliable. Domain modeling, schema evolution, permissions, uncertainty handling, and evaluation remain substantial engineering work. But the reusable product could be the **factory and protocol**, not one agent for one task.

## What we should test first

The bold claim—that structure can compensate for model size—deserves an empirical test.

Use the same inventory or shelf-inspection tasks to compare:

- A conventional small-model agent with free-form tool calling.
- The same small model with graph-selected context and a constrained proposal vocabulary.
- A larger general model baseline, with and without the graph harness.

Measure end-to-end task success, invalid proposal rate, incorrect-but-schema-valid changes, policy violations, recovery after conflicting observations, latency, token consumption, and cost. Test both clean cases and adversarial or ambiguous ones. Separate mistakes in perception from mistakes in reasoning and execution.

Also measure the *total* system cost. The graph engine, linearization, validation, and repeated model calls are not free; an overly restrictive traversal policy might increase latency or hide critical evidence. Small-model efficiency should be demonstrated, not assumed.

## A minimal implementation plan

I would start with a single domain and a handful of typed nodes and operations. Implement an immutable graph projection backed by an append-only event log. Create a deterministic subgraph selector and linearizer. Require schema-constrained model output with stable node IDs and an expected graph version. Validate proposals before updating the projection, and expose structured rejection reasons for retry or escalation.

Only after that loop works should we introduce multiple micro-agents, hierarchical orchestration, or specialized compute backends. A clear protocol is more valuable initially than an elaborate framework.

## The bigger idea

There is a temptation to define progress in AI entirely by larger models and more general agents. I think another axis matters: **the quality of the environment in which a model reasons and acts**.

A graph-native harness can make context explicit, keep actions bounded, represent evidence, separate proposals from facts, and turn state changes into testable software transitions. It may let modest models do useful work in tightly defined domains while reserving broader intelligence for the tasks that truly need it.

The architectural hypothesis is simple:

> **The graph defines the world and the allowed transitions. The model proposes local transformations. The runtime decides what becomes real.**

If this works in practice, the future agent ecosystem may look less like one giant agent operating everywhere and more like a network of small, composable, graph-constrained agents with explicit contracts and a shared execution protocol.

That is the experiment I would like to build next.
