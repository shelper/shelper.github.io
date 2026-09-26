---
title: "A Graph as a Model Harness: Let Context Correct Vision"
date: 2026-09-26
tags: ["computer vision", "scene graphs", "structured data", "retail", "model evaluation", "optimization"]
categories: ["technology"]
series: ["AI Beyond the Model"]
description: "A concrete retail example in which structured relationships constrain, rerank, evaluate, and teach visual models."
---

*Part 3 of [AI Beyond the Model](/series/ai-beyond-the-model/). [Part 2 explains the representation layer](/posts/the-representation-layer-physical-ai-needs/).*

Most discussion of scene graphs asks how to turn visual detections into structured state. I am interested in the reverse direction too: **can the structure help correct the detections and associations that produced it?**

This was the focus of a discussion about retail shelf images. The initial models might detect products and price labels, produce top-k product candidates, and propose which label belongs to which product. The outputs are imperfect. A structured representation of shelf regions, product positions, categories, labels, and neighboring relations provides information that isolated crops lack.

I call that representation a **model harness**. It supplies context, constraints, and feedback around a vision model, much as a good execution environment can make a language model more effective. This is not a claim that a graph itself recognizes products. It is a concrete way to ask better questions of the recognition system, rule out inconsistent answers, and measure whether the extra structure helps.

## 1. A shelf is more than a bag of crops

Suppose a detector finds six products and five price labels. Each product classifier proposes several SKUs with scores. A label reader extracts possible text and prices. A separate model proposes candidate product-label links.

If each crop is classified independently, the system ignores facts such as:

- items are grouped into shelves and sections;
- nearby products often share category or brand context;
- labels have geometric relationships to products;
- a barcode match can be stronger evidence than proximity;
- one label may describe a row or facing arrangement rather than exactly one detection;
- candidate associations can conflict with each other.

These are useful constraints, not absolute laws. A new promotion can place an unusual item next to unrelated products. Labels can be missing, shifted, or shared. If the system treats contextual regularities as hard truth, it can turn a single mistake into a confident chain of mistakes.

The graph therefore needs to represent **candidates and evidence**, not just final answers:

```text
Shelf region S
  contains product candidates P1 ... P6
  contains labels L1 ... L5
  P3 has candidate SKUs {A, B, C}
  P3 may connect to labels {L2, L3}
  some relations are confirmed; others are hypotheses
```

The point is to keep enough structure to ask whether a complete shelf interpretation is coherent.

## 2. First experiment: contextual reranking

Start with the simplest intervention. Do not retrain the classifier. For each product, keep its top-k candidates. Estimate broad shelf or region context from the more reliable predictions, then rerank ambiguous products.

For example, if most clearly recognized items in one section belong to the same brand and a blurry crop has two similar-looking candidates, shelf context may favor the matching brand. The reranker might use category agreement, neighbor relationships, label text, geometry, and the original classifier score.

The baseline matters. Measure top-1 and top-k product accuracy before applying context, then evaluate the reranked results on held-out shelf images. Report changes separately for easy and difficult products. It is not enough to show several successful examples.

There is a trap here: majority voting can become circular. If three adjacent products are all misclassified in the same way, their apparent consensus may reinforce the error. I would start context estimation from high-quality anchors such as readable barcodes or independently verified labels, use weaker evidence as a soft prior, and preserve the original model outputs for analysis.

If the correct SKU is absent from top-k, reranking cannot recover it. That observation identifies a distinct perception problem rather than a failure of the graph logic.

## 3. Product-label links as a constrained assignment

Associating products with labels is a richer example. The local model may score each candidate pair separately, but decisions interact. A high-confidence barcode or SKU match can anchor one edge. Shelf boundaries, relative position, and ordering can remove or downweight incompatible alternatives.

One iterative procedure would be:

1. Generate plausible product-label pairs from geometry and shelf regions.
2. Confirm a small set of pairs with independent strong evidence.
3. Revise neighboring candidates using section boundaries and spatial relationships.
4. Solve for a consistent set of assignments, retaining ambiguous cases.
5. Repeat only when new independent evidence appears or an assignment changes.

In a carefully specified shelf geometry, crossing links might signal an invalid association. But a blanket “links must never cross” rule would be unsafe across different camera perspectives or retail arrangements. Its validity belongs in the task's geometry model and evaluation data.

The key shift is from scoring isolated edges to scoring a *configuration*. Informally:

```text
score(configuration) =
    visual product evidence
  + label and barcode evidence
  + geometric compatibility
  + contextual relationship scores
  - penalties for violated constraints
```

Some constraints can be hard if the domain truly guarantees them. Others should be soft, because stores are messy. The implementation might use graph matching, integer programming, factor graphs, or simpler greedy updates. The representation provides the variables and dependencies; it does not force one solver.

The final output should say which links are well supported, which are merely plausible, and which remain unresolved.

## 4. Avoid double counting and self-confirmation

The system must distinguish a *new piece of evidence* from the same prediction repeated in several forms.

For instance, if shelf category is inferred from the product classifier, then feeding that category back as if it were independent ground truth can make the product score seem more certain without adding information. A similar problem occurs if a label association model was trained on annotations generated by the current graph rules.

Provenance helps. Each edge or score should record where it came from: visual model, OCR, barcode, geometry, human correction, or a derived graph rule. A refined confidence score should be calibrated on held-out data; adding weighted scores together does not automatically create a probability.

One way to test whether the graph is genuinely helping is to remove one source at a time. If the apparent improvement disappears when independently read labels are withheld, then label evidence may be doing the work. That is still useful, but the explanation should be accurate.

## 5. The same graph can teach smaller models

The representation is also a training-data factory. With annotated shelf images, it can enumerate plausible product-label pairs and describe each with relative geometry, shelf membership, visual features, text matches, and neighboring context.

That gives a dedicated association model a well-defined input and target. Strongly verified pairs can become positive examples; plausible but incorrect pairs become hard negatives. Human reviewers can focus on ambiguous candidates instead of drawing every relationship from scratch.

This can create a useful cycle:

```text
annotations → structured graph → contextual rules
           → candidate review → better training examples
           → smaller association model → new graph proposals
```

The loop is only as good as its validation. If automatically generated labels are treated as unquestionable truth, the smaller model learns the original rules' errors. Hold out manually reviewed cases and track which examples came from which source.

The same structure helps failure analysis. One query might find cases where visual SKU confidence was high but graph context disagreed. Another might find labels linked across shelf boundaries. Those queries make it easier to sample informative errors and improve the right component.

## 6. One representation, several products

A shelf graph can support more than recognition:

| Use | Example question |
| --- | --- |
| Annotation | Which associations have one clear candidate, and which require review? |
| Search and filtering | Show products with a label mismatch in aisle A. |
| Recognition refinement | Which top-k SKU best fits the label and nearby items? |
| Evaluation | Where did the graph improve or damage the baseline model? |
| Training | Which product-label pairs are reliable positives and hard negatives? |
| Workflow | Which unresolved cases should a worker inspect? |

This is why I see a graph as an infrastructure layer, not a single research feature. It gives multiple consumers a consistent set of objects, candidates, relationships, and evidence. But its worth must be demonstrated through concrete improvements, not assumed from the elegance of the data model.

## 7. A clean evaluation plan

I would begin with a small but real set of shelf images containing image-level or object-level annotations, true product identities, true label text, correct associations, and the unmodified model outputs. Then:

1. Establish independent recognition and association baselines.
2. Apply a minimal contextual reranker.
3. Add one relation family at a time: region, neighbors, label text, geometry.
4. Compare with joint assignment or optimization only if simple methods plateau.
5. Report both aggregate metrics and important slices: poor lighting, missing labels, category transitions, and repeated lookalike products.
6. Inspect regressions, not only gains; an optimization can improve average accuracy while causing unacceptable confident errors.

The dataset split must prevent nearly identical images or the same shelf layout from appearing in training and test in ways that inflate performance. If the model will be used across stores, test across stores.

There is related research on using contextual cues to jointly infer objects and relationships, such as [scene graph generation by iterative message passing](https://openaccess.thecvf.com/content_cvpr_2017/html/Xu_Scene_Graph_Generation_CVPR_2017_paper.html). But the exact retail constraints and deployment conditions here are a separate empirical question.

## Closing thought

The usual pipeline is **vision → graph → application**. A stronger loop is **vision → graph → constraints → revised vision hypothesis**, with provenance and evaluation at every step.

The graph may reduce the effective search space, suggest the next annotation, and supply training examples for a specialist. It may also mislead if its assumptions are wrong or its evidence is counted twice. That is why I would start with real images, a fixed baseline, and explicit failure analysis.

This is a practical instance of a broader theme in the series: intelligence can come from the organization around a model, not only the model's weights. Next, [how a general model might build and coordinate such specialists](/posts/an-ai-that-learns-to-build-its-own-specialists/).

## Series reading guide

1. [From Next-Token Prediction to Persistent World Models](/posts/from-next-token-prediction-to-persistent-world-models/) — inference leads to a persistent world state.
2. [The Representation Layer Physical AI Needs](/posts/the-representation-layer-physical-ai-needs/) — shared semantics, contracts, history, and runtime.
3. [A Graph as a Model Harness](/posts/a-graph-as-a-model-harness/) — structured context corrects and teaches perception.
4. [An AI That Learns to Build Its Own Specialists](/posts/an-ai-that-learns-to-build-its-own-specialists/) — a foundation model develops and coordinates specialized capabilities.
5. [Can AI Learn How to Search?](/posts/can-ai-learn-how-to-search/) — the remaining challenge of discovering useful abstractions and strategies.

### Further reading

- [Scene Graph Generation by Iterative Message Passing](https://openaccess.thecvf.com/content_cvpr_2017/html/Xu_Scene_Graph_Generation_CVPR_2017_paper.html).
- [Hydra: Real-time 3D Scene Graph Construction and Optimization](https://arxiv.org/abs/2201.13360) — a different setting showing structured spatial state as a reusable substrate.
