---
title: "AI Will Evolve Like a Society, Not Just a Model"
date: 2026-11-02
tags: ["artificial intelligence", "agents", "simulation", "software", "collective intelligence"]
categories: ["technology"]
description: "Why AI progress may increasingly depend on collaboration, shared knowledge, executable tools, and simulation—not just stronger language models."
---

A thought has been on my mind: what if we are still looking at AI too much like an individual brain?

We often discuss progress in terms of the next model: its benchmark scores, reasoning ability, context window, or parameter count. These things matter. But I suspect that language models and today's agents are only the beginning. The more consequential evolution may be in how AI systems organize work, share knowledge, use tools, and build on one another's results.

The analogy I find useful is human society.

## The intelligence of an individual and the intelligence of a civilization

Human civilization can do things today that would have seemed impossible ten thousand years ago: manufacture chips, send spacecraft to other planets, and design medicines. Yet we should be cautious about claiming that modern humans are innately much more intelligent than our distant ancestors. We cannot give people from ten thousand years ago a modern IQ test and draw a meaningful direct comparison.

What clearly changed is the **system around the individual**. Language, writing, mathematics, science, engineering, institutions, and communication made it possible to preserve knowledge and build on it across generations. Each person inherits tools and understanding that no individual could recreate in a lifetime.

The same distinction may become essential for AI. A model has capabilities and limits, but a system built around it can also have memory, tools, specialists, verification, and ways to pass experience between tasks. Even if improvements in a single model eventually slow down, the system as a whole could continue to improve through better organization.

## From models to agents to an organization of capabilities

A language model can interpret a request, reason about a problem, and generate a response. An agent adds the ability to act: call tools, observe results, maintain state, and revise its approach.

From there, I can imagine additional layers. A system might create specialized agents for data analysis, engineering design, testing, and deployment. A higher-level process could coordinate them around an objective, manage dependencies, and decide when to seek a human decision. Useful solutions could become reusable skills, programs, datasets, or workflows.

This does not require every agent to be an expert at everything. A general model might identify the problem and orchestrate the work, while a specialized model, a numerical solver, or a conventional algorithm handles a well-defined step. What matters is whether the combined system can reliably accomplish something that its individual components could not.

I explored a related idea in [An AI That Learns to Build Its Own Specialists](/posts/an-ai-that-learns-to-build-its-own-specialists/): a general system could learn when to create a specialist, how to evaluate it, and when to revise or replace it. Here I am thinking about the broader consequence: those capabilities could accumulate into something resembling a learning organization.

## Why the cycle could move faster

AI systems may be able to iterate much faster than human organizations in several ways:

- **Parallel exploration:** many experiments or candidate solutions can be examined at once.
- **Fast feedback:** code, tests, simulations, and benchmarks can sometimes be run automatically.
- **High-bandwidth transfer:** a useful procedure can be shared as code, data, a model, or a structured workflow.
- **Reuse:** once a capability is validated, many agents can potentially call the same tool or use the same skill.
- **Persistence:** an agent need not run continuously for its state, responsibilities, and accumulated knowledge to remain available.

I can imagine parts of this process moving ten times faster than comparable human work, perhaps more in some domains. That is an intuition, not a measured forecast. Tasks requiring physical experiments, manufacturing, long-term observation, or human adoption will still face constraints that cannot simply be removed by faster inference.

Speed also makes knowledge selection more important. If agents continually copy one another's unverified conclusions, errors could spread as quickly as useful discoveries. A learning organization needs independent tests, evidence, provenance, and ways to revise or discard knowledge. Saving an answer is not the same as knowing when it is trustworthy.

## Recursive improvement can happen at the system level

Recursive self-improvement is often imagined as one superintelligent model rewriting itself. A broader and more practical possibility is that the **whole AI system** improves its own ability to work.

One agent might improve an evaluation pipeline. Another could build a faster specialist for a recurring task. Another might organize experiment results so future agents can find them. Together, these changes make later work more effective even if the underlying foundation model has not changed.

The loop could look like this:

> Attempt a task → measure the result → keep and share what works → use the improved capability to perform the next task.

If AI also helps improve the models, tools, and evaluation systems used in this loop, progress could accelerate further. How far and how fast that happens remains uncertain: computation, data, energy, reliable evaluation, and the difficulty of discovering genuine improvements can all become bottlenecks.

## Software becomes a tool for agents

I do not think this evolution means software disappears. It may mean that **software gets used much more extensively**, often by agents rather than directly by people.

A language model is useful for interpreting a goal and choosing an approach. A mature program may be better at executing a precise computation repeatedly. An agent can connect the two: decide what to do, call the right software, inspect the result, and adjust.

That changes what makes software valuable. An interface built around menus and manual interaction is useful to people, but an agent also needs APIs, structured inputs and outputs, automation, reproducibility, and clear failure behavior. Professional software that provides hard-to-recreate capabilities—such as CAD, simulation, numerical optimization, or chip design—could become essential infrastructure for AI systems.

At the same time, simple applications whose primary value is arranging information for human interaction may be easier for agents to assemble on demand. This is not a claim that every graphical application disappears. It is a question of where lasting value resides when a growing share of software interactions are machine-to-machine.

## Information should be easy for machines to reuse

If agents become major producers and consumers of information, the format of that information matters.

For many tasks, plain text, Markdown, or structured data can be searched, compared, versioned, transformed, and combined more easily than a document optimized for visual presentation. A machine evaluating an experiment needs the measurement, units, conditions, provenance, and uncertainty. It usually does not need page breaks or a decorative layout.

The appropriate format still depends on the task. JSON or a database may be better for precise records; Markdown can explain design decisions; images and binary data are appropriate when the information is inherently visual or high-dimensional. A human-facing report can be generated when someone needs one, without requiring every intermediate artifact to be formatted as a report.

Bandwidth is not only the number of bytes transmitted. An agent also has to find and interpret the relevant information. Reducing ambiguity and unnecessary context can make knowledge easier to use.

## Executable knowledge: a 3D model as a recipe

A more powerful idea is to share not only a result but also **the procedure that produces it**.

Consider a 3D scene built with Blender. An agent could generate a Python script that calls Blender's APIs to create the geometry, assign materials, position the camera, and render the result. Another agent could read the script, modify a parameter, and create thousands of variations.

This has advantages:

- The process can be repeated and automated.
- Code and parameters can be compared and versioned.
- Outputs can be checked against explicit constraints.
- A generator can be reused in other workflows.
- The expensive reasoning that created the procedure can be reused across many executions.

A script alone does not guarantee bit-for-bit determinism. Reproducing an output may also require specific software and plugin versions, assets, settings, random seeds, and hardware behavior. Some rendering and numerical operations may still differ slightly across environments. A useful reproducible artifact therefore includes not only the code, but also the inputs and relevant execution environment.

Nor is a script always smaller or better than a finished artifact. A scanned object or a complex texture may depend on large original datasets. But for parameterized work, an executable recipe can be much more flexible than a single static result.

## Simulation as an accelerator

Simulation could be one of the most important tools in this evolving system because it lets agents explore possibilities without building every candidate in the physical world.

Imagine designing a robotic gripper. Agents could generate different geometries, simulate forces and motion, compare the results, and select a small number of promising designs for fabrication. A practical workflow might use a fast, approximate model to filter a large design space, a higher-fidelity solver for a smaller set, and physical tests for final validation.

This can create another learning loop:

> Propose a design → simulate it → test selected candidates in reality → use the results to improve the simulation → explore again.

The important limitation is the **sim-to-real gap**. A simulator is an approximation, and an optimizer may discover solutions that exploit its errors. Faster exploration can therefore produce more convincing failures as well as more useful designs. Knowing where a simulation is reliable, checking its assumptions, and comparing predictions against physical measurements remain essential.

AI can also train a fast surrogate model on earlier simulation results and use it for routine predictions. That may reduce the number of expensive solver runs for a particular task, while new designs, unfamiliar conditions, and final validation may still require higher-fidelity tools or physical experiments.

## From a model to an evolving ecosystem

The picture I find most compelling is not one model replacing every program, expert, or scientific instrument. It is an ecosystem in which models, agents, specialized algorithms, and trusted software work together.

A model provides general reasoning and communication. Agents turn that capability into action. Software supplies precise and reusable operations. Shared representations and memory preserve what the system has learned. Simulation increases the number of ideas that can be explored. Evaluation and real-world feedback help separate progress from convincing mistakes.

Human progress came not only from individual intelligence but from our ability to organize, communicate, preserve knowledge, and improve our tools. AI may follow a similar pattern—with much faster communication and iteration in some parts of the system.

How far that analogy goes is an open question. But it changes the question I ask about the future of AI. Instead of only asking, “How intelligent will the next model be?”, I also want to ask:

**What kind of system can learn to organize its capabilities, verify its discoveries, and pass useful knowledge forward?**
