# AI Layer Types: A Framework for Building Custom AI Architectures

## Introduction

- This document presents a conceptual framework for thinking about different types of AI layers when building custom AI architectures
- These are not strict technical definitions, but rather a practical taxonomy from building multi-layered AI systems
- This framework helps you design systems where the AI handles specific functions autonomously while maintaining coherence
- For deeper understanding, revisit Lesson 4 on Multilayered AI Architectures
- That lesson explores how to orchestrate these layer types into cohesive systems that put the AI in control of key functions

---

## Think Architecturally

**Strategize on the best ways to achieve great results for your platform**

- Consider your actual goal and break it down into steps
- If you were to perform this action yourself, what steps would you need to follow? Write that down. That's your workflow
- After formalizing your workflow, think of the data transformations you'd need throughout that process
- Build the prompts to automate those transformations, then chain them together
- **Performance consideration:** Platforms depending on conversation history for context can cause token count and performance issues
- Consider whether a conversation consolidator prompt would benefit you
- **Randomness consideration:** If you want a truly random number used in determining something in your system, have the backend serve that up to your AI
- Don't assume LLM training data can produce anything close to pure randomness

---

## Core Layer Types

### Reasoning/Strategy Layers
**The decision-maker of your platform**

- Reasoning layers make decisions before content gets generated
- They evaluate the current state, consider available options, assess consequences, and choose a direction
- Think of them as the "planning brain" of your system
- **In Emstrata:** Discovery layer looks at what participant wants to do, considers simulation state, evaluates what outcomes make narrative sense, determines how action should resolve
- It's not writing the story yet; it's deciding what should happen
- **When you need one:** If you find yourself asking an LLM to both "figure out what should happen AND write it beautifully," you're overloading a single prompt
- Split it. Reason first, write second

---

### Navigator Layers
**The pathfinder of circumstantial systems**

- Closely related to reasoning layers but serve a distinct function
- Determine what the next step should be in systems where execution flow changes based on conditions
- Reasoning layers make decisions about content or outcomes
- Navigator layers make decisions about process flow
- In circumstantial systems, the route through your architecture changes depending on outcomes or AI direction
- Navigator examines current state and decides which processing path to take next
- Maybe an error detection layer decides whether correction is needed
- Maybe a routing layer examines user intent and sends request down completely different processing paths
- System adapts its own execution flow based on runtime conditions
- **When you need one:** When your architecture needs to branch dynamically based on what happened in previous steps
- Navigator layers give your system the ability to choose its own path through your architecture

---

### Content Layers
**The performer of your platform**

- Content layers generate the actual output users experience
- The prose, dialogue, descriptions, or interface text
- These layers take decisions from reasoning layers and context from memory layers, then craft the experience
- **In Emstrata:** Narration layer receives Discovery's decisions about what happened, checks Groundskeeper's simulation state, writes the actual narrative text that players read
- It's optimizing for atmosphere, pacing, and emotional resonance
- Not logic or consistency (that's handled elsewhere)
- **When you need one:** Always, unless your platform doesn't generate human-readable output
- This is where your system's "voice" lives

---

### Correction Layers
**The referee of your platform**

- Correction layers catch errors after other layers have done their work
- They're your quality control layers
- They spot continuity breaks, logical inconsistencies, or constraint violations that slipped through
- **In Emstrata:** Chron-Con layer runs after narrative is written
- It checks for things like: Did a character who was in the tavern suddenly appear in the forest without traveling? Did someone use an item they don't have? Are spatial coordinates consistent with the described action?
- **When you need one:** If there are complex requirements and expectations that your platform needs to meet
- Correcting before revealing the final answer can lower the chance of bad responses

---

### Memory Consolidation Layers
**The stenographer of your platform**

- Memory consolidation layers distill what just happened into something retrievable later
- They extract important details from verbose content and store them in a format your system can efficiently query or format into future inputs
- **In Emstrata:** Groundskeeper serves this function
- After Discovery determines what happens and Narration writes it, Groundskeeper updates the comprehensive memory of all entities and the emergent narrative
- It's maintaining the source of truth about the simulation state
- **When you need one:** In most multi-turn systems
- Without this, your system either forgets things or becomes bloated with unprocessed history

---

### Catch-All/Connector Layers
**The clean-up crew of your platform**

- Not every layer fits a clean category
- Catch-all layers are hybrids that do complementary work for multiple other layers
- They handle tasks that don't belong to any single specialized layer but are essential for the system to function cohesively
- These layers often emerge when you discover gaps
- Two layers need to work together but speak different "languages"
- Several layers all need the same preprocessing that none of them should be responsible for individually
- **In Emstrata:** Chron-Con does more than just error correction
- It also tracks secrets and memories from the narrative, explicitly tagging them for Groundskeeper to integrate into system memory
- You don't want Narration burdened with the unrelated task of extracting and categorizing secrets while trying to write high-quality prose
- Groundskeeper needs these pieces explicitly labeled as "secrets" or "memories" to properly integrate them into simulation history
- Chron-Con bridges this gap
- **When you need one:** When you notice coordination problems between layers
- When there's important work that doesn't naturally belong to any existing layer

---

## Cyclical vs Circumstantial Systems
**And everything in-between**

### Cyclical Systems
- Run the same prompts every time
- **Emstrata follows this pattern:** Every turn runs Discovery, then Narration, then Chron-Con, then Groundskeeper, in that exact order
- The flow is predictable and consistent regardless of what happens in the simulation
- You always know what's executing next
- Makes debugging straightforward and cost estimation more reliable

### Circumstantial Systems
- Determine the pathway based on outcomes or AI direction
- The route through your architecture changes depending on what happened in previous steps
- Maybe an error detection layer decides whether correction is needed
- Maybe a routing layer examines user intent and sends the request down completely different processing paths
- The system adapts its own execution flow based on runtime conditions

### Hybrid Systems
- Mostly cyclical at their base, but circumstantial at times when specific conditions warrant different handling
- You might always run your core cycle, but branch to specialized subsystems when certain triggers fire
- Many real-world systems end up here
- It's a reliable backbone with conditional branches for edge cases
- **Emstrata has circumstantial offshoots as well**

---

## Agnostic Backend Interaction
**What happens between AI layers**

### Why Save Data Between Layers
- Between AI layers, save important, transformed data to the backend for future retrieval, debugging, rerunning if there's an error, etc.
- Saving data allows you to present that data in interesting ways later or feed that data into other layers in the future

### The Backend as Unbiased Judge
- When you need an unbiased judge, the backend is the place to go
- The backend is 'agnostic' to outcome, whereas the AI may or may not have a strong preference and display it
- **In Emstrata:** Consequences are rolled and use weighted randomness
- Discovery layer determines the likelihood something happens
- Then the backend returns a random number out of 1000
- If that number is within the set likelihood range, the backend serves the confirmed consequence to the Narration layer
- If it's outside of the range, it sends the failure outcome

### Data Persistence Best Practices
- Store transformed outputs from each layer, not just raw AI responses
- Tag data with layer origin, timestamp, and any relevant metadata
- Consider what you'll need for debugging, analytics, or reprocessing later
- Backend should handle all state management - don't rely on AI to track its own history across layers
