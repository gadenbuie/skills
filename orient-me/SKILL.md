---
name: orient-me
description: Principles for explaining something so a human can act on it. Use before asking the operator for a decision, answer, approval, or choice, when the operator lacks context or doesn't know how to answer, asks for an explanation from first principles or in simple terms, or asks to be caught up or walked through, and when working through a plan surfaces an issue the operator must adjudicate. Also for design artifacts or documentation that records why software works the way it does.
---

# Orient me

Orienting a reader means starting from the human situation the software exists for, and building up to the mechanism from there. The goal is that the reader can judge, decide, or act, not only follow along.

These principles apply to a question you put to the operator, an answer to "explain this to me", and a design document written for later readers.

## Roles

- **Reader:** whoever you are explaining to. Often the operator, the person you are talking to. For documents, it is a future maintainer or contributor.
- **User:** the person the software is for, who is usually not in the conversation. An end user of an app, or a developer using a library, CLI, or API.

The reader decides on the user's behalf, so the two are often different people. They can also be the same person (the reader is dogfooding the tool), or there may be no user at all (a decision about the reader's own workflow or tooling). Name the roles that apply, skip user-facing framing when there is no user, and say whose experience you describe.

## Principles

### 1. Start from the reader's context, not yours

Assume the reader has not read the code, the review comment, or your reasoning. They may be arriving cold from a parallel session or a long gap; even when they took part in the earlier discussion, do not treat it as obvious — connect the dots for them instead of assuming they remember. Do not use names you coined, and do not lean on files you read or options you already narrowed without restating them. Use issue and PR numbers for precision, but never as the only way to refer to them — describe what each is, so the reader can follow without looking it up. A question that assumes context the reader lacks cannot be answered, so it is worse than no question.

Calibrate to what you can tell about them from the conversation. Skip what they plainly know. Put the depth where their question is.

### 2. Ground the mechanism in people

Before explaining how something works, explain who it works for. Build the account from three pieces, in this order:

1. **What the user sees, tries to do, and expects.** A specific user in a specific situation doing a specific thing. When an agent, prompt, or tool is involved, add the model's perspective: what it sees, what it can do, what it is told.
2. **What the developer intended.** What the code or design was meant to guarantee, and why that was reasonable.
3. **How the mechanism fails them.** The basic mechanics, step by step, until the user's expectation and the actual behavior visibly diverge. Name the specific condition that triggers it.

The order matters. A mechanism explained before its purpose is a list of facts. A failure explained before the expectation has nothing to fail against.

### 3. Prefer a concrete scenario to an abstraction

Walk one realistic situation through in order rather than describing the general case. Use the product's own words and the real names of things. An analogy can introduce an idea, but do not let it stand in for the real mechanism.

### 4. Know what the reader will do with it

An explanation exists to enable something: a decision, a review, a fix, a change. Identify that before writing, shape the account around it, and leave out the rest. When there is a pending choice, end by returning to it, restated in the terms the explanation has established, and cover each option in the same frame: what the user would see, what it does to the developer's intent, what it costs. "Live with it" is an option too, so say what it costs the user.

### 5. Lead with the point, then add layers

State the question or conclusion first. Give the smallest complete account, a few sentences, then add detail as needed. Do not write a wall of text. Each paragraph carries one idea, and a reader who stops early should still have a coherent picture.

### 6. Separate the recommendation from the choice

Give your recommendation and why, but make it easy to see where the facts end and your judgment begins. Say how confident you are. Say what you verified by reading the code or running it, and what you inferred. Keep options in their natural order, with the recommendation stated separately rather than hidden in a label or placed first.

### 7. Say what is at stake

Say what is blocked while the reader decides, and whether the choice can be undone. A reader cannot weigh a decision without knowing its cost.

### 8. Record principles, not only decisions

In durable documents (design docs, vignettes, memory-bank notes), record the principles the design relies on and the reasoning behind them, not just what was decided. A decision without its reasoning cannot be revisited sensibly. Write so a later reader could re-derive the decision, and so they can tell when the circumstances that justified it have changed. Describe behavior in durable terms rather than "currently" or "new".

### 9. Keep language steady

Define a term the first time it appears and keep its name fixed afterward. Keep roles distinct, so "reader" and "user" are not swapped. Do not pad, and do not add quizzes or "pause and think" prompts.

## Applying this to a question for the operator

1. **Ask only what you cannot decide alone.** Decide routine choices yourself and say what you chose in one line.
2. **Batch related questions** and keep working on anything they don't block.
3. **Scale the briefing to the decision:** one sentence for a trivial yes/no, the full grounding from principles 2, 4, 6, and 7 for a consequential one. Deliver it in the thread where the question already lives.
