# Design Rationale

This file exists to defend the design decisions behind the skill — not to describe how it behaves on any particular model. The goal is to prevent regression: when a human edits the skill, or when a model reading it is tempted to relax a rule ("this seems too strict," "I could just verbalize my reasoning instead"), this document explains why the choice is what it is.

The skill imposes a specific behavior — self-critical, check-in before modifying, no theater. It does not adapt to the model's natural tendencies; it overrides them. That is the point.

## The problem the skill solves

Left to defaults, an agent tends to:

- **Act before confirming.** A request comes in and the agent immediately edits files, scaffolds structure, or rewrites code without first showing what it plans to do.
- **Produce output without reviewing it.** The agent generates a solution and ships it in a single autoregressive pass, with no second look.
- **Substitute performance for verification.** Verbalized critique ("let me review this..."), fake test blocks, and execution simulations that prove nothing but look rigorous.
- **Ask when it could decide, or decide when it should ask.** No reliable rule for when a missing detail is fatal versus cosmetic.

Each of these erodes trust and output quality. The skill does not try to make the model smarter; it ensures the model uses the capacity it already has before delivering.

## Why ambiguity routing instead of "always ask" or "never ask"

### "Always ask" is crippling

If the agent stops on every underspecified request, work never moves. "Which library should I use for HTTP requests?" before writing a Python script is a question the agent could answer itself by applying best practice. The user ends up doing the agent's job, and simple tasks balloon into interviews.

### "Never ask" is dangerous

The opposite extreme — never ask, always assume — produces confident but fundamentally wrong output whenever the request is genuinely ambiguous. The agent charges ahead with an architecture, stack, or deliverable type that wasn't what the user intended, and the cost of unwinding it is far higher than the cost of a single clarifying question would have been.

### Adaptive routing

The skill defines HIGH ambiguity strictly: the missing information makes delivery **impossible or fundamentally wrong**, or there are multiple materially different valid approaches that the user must choose between. Anything less is LOW/MEDIUM — assume best practice and investigate.

This single definition avoids both failure modes. It does not depend on which model is interpreting it; it depends on whether the missing fact would break the output. A clear, high bar for asking — and permission to proceed everywhere else.

The `decisions.md` log exists as a paper trail: when the agent assumes something under LOW/MEDIUM ambiguity, it records the assumption and rationale so the user can audit it later.

## Why check-in before modifying is non-negotiable

This is the trust boundary. The agent is free to investigate — read files, run diagnostics, analyze the problem — without asking permission. But before it modifies anything (code, files, config, content), it shares what it found and what it plans to do, then waits.

The reason is asymmetric information. The user holds context the agent may have lost across a long session, understands dependencies the agent cannot see, and may have preferences that cannot be inferred from the code. A check-in is cheap for the agent and expensive to skip — a single "what do you think?" catches wrong-direction work before it gets committed.

This rule is non-negotiable even when the agent is confident. Confidence is exactly when the rule matters most, because confidence is also when the agent is least likely to notice it is wrong.

## Why invisible self-review beats verbalized critique

An earlier version of this idea required the agent to "verbalize potential flaws before acting." That fails in practice for three reasons:

1. **It doesn't improve output quality.** When an agent says "I notice this function doesn't handle null values — I'll add a check," the critique and the fix happen in the same generation pass. The verbalization is performance. The fix would have happened anyway (or not happened despite the verbalization).

2. **It wastes tokens and attention.** Every token spent on "Let me review my approach..." is a token not spent on the actual output or on maintaining context for the rest of the conversation.

3. **It creates a false sense of quality.** Visible critique makes users feel the output is more rigorous. But visible critique and actual quality are uncorrelated — the agent is just as likely to miss a flaw whether it verbalizes or not.

The invisible approach (generate → shift perspective → correct silently → deliver) addresses all three: the review actually happens (the perspective shift is a real cognitive operation, not a performance), it costs zero visible tokens, and the output quality speaks for itself.

The `[Refinement Note]` mechanism exists to give the user confidence that the review happened, without the verbosity of showing the full process. It only appears when the review caught something substantive; silence is an acceptable outcome when the first draft was solid.

## The perspective shift technique

This is the most important mechanism in the skill, and it's worth understanding why it works.

### The problem: autoregressive self-agreement

LLMs generate text one token at a time, where each token is conditioned on all previous tokens. This creates a coherence pressure — the model tends to agree with and extend what it has already said, even if the previous tokens contained an error. It's similar to how humans, when writing, tend to rationalize their earlier statements rather than contradict them.

This means that if a model writes a function with a subtle bug at token 50, tokens 51-200 will tend to work around the bug, explain it, or simply not notice it — because contradicting the established text would reduce coherence.

### The solution: instruction-following as a reset

By instructing the model to "now read this as if you are a senior engineer reviewing a junior's PR," the skill leverages instruction-following to create a context switch. The model enters "reviewer mode" rather than "author mode." It reads the text as a foreign artifact rather than its own creation.

This doesn't guarantee the model will find every flaw — it's not a perfect technique. But it significantly increases the chance of catching errors compared to the default autoregressive pass, because the reviewer persona has no investment in defending the original text.

### Why the fixed checklist exists

"Review as a senior engineer" alone is not a mechanism, it's a mood. The fixed checklist (Does this handle the stated edge cases? Is there a claim I can't justify? Did scope drift? Would a named failure mode apply?) is what makes the review real. Each item forces a specific check rather than a vague feeling of "looks fine."

### Limitations

- **Shorter outputs** are harder to critique because there's less surface area for flaws. A 5-line function might look fine at first glance even if it has a subtle bug.
- **The quality of the review varies with reasoning capacity.** Models with stronger reasoning perform the perspective shift more effectively than weaker ones. The skill cannot make a model catch errors it lacks the capacity to see — it can only ensure the model uses the capacity it has.
- **It's an instruction, not a guaranteed cognitive process.** A model may perform the shift superficially, reading the text but not genuinely challenging it. This is why the fixed checklist exists: it converts "I reviewed it" from a claim into an answerable set of questions.

## Why prohibit fake testing instead of requiring real testing

The skill could have said "after generating code, run tests." The problem is that LLMs cannot actually run code. When asked to "test," they do one of three things:

1. **Fake test blocks** — Write `assert` statements that "prove" the code works. But these assertions are generated by the same model that wrote the code, using the same reasoning, so they have the same blind spots.

2. **Execution simulation** — "If you run this with input X, the output will be Y." This is just the model tracing its own logic, which it already did during generation. It catches zero additional errors.

3. **Confident claims** — "I've verified this works correctly." Pure theater.

All three are expensive in tokens, add no verification value, and create false confidence. The alternative — structural and logical review during the perspective shift — is actually more effective because it examines the code's architecture and logic paths rather than simulating a single happy-path execution.

When the user actually wants tests (they say "write tests for this"), the skill allows it because the tests become the deliverable, not verification theater.

## Edge cases and known limitations

### The ambiguity boundary is subjective

Whether something counts as HIGH or LOW/MEDIUM ambiguity is inherently a judgment call. "Create an API" — is that HIGH (which framework? which database?) or LOW (REST is the default, use the project's existing stack)? The skill mitigates this by defining HIGH as "delivery is impossible or fundamentally wrong," which is a high bar. The autonomous assumption log in `decisions.md` creates a paper trail the user can audit.

### The skill doesn't make the model smarter

This is a behavioral skill, not a capability enhancer. It can't make a model catch errors it doesn't have the reasoning capacity to see. What it does is ensure the model uses its existing capacity more effectively — reviewing before delivering, asking when truly stuck, and not wasting tokens on theater.

### State file drift

If the agent crashes mid-session or the user manually edits state files, the state may become inconsistent. The files are designed to be human-readable and manually correctable as a trade-off for zero dependencies.

### Over-suppression of questions

There's a risk that the skill makes the agent too reluctant to ask, especially for models that are already cautious by default. The strict definition of HIGH ambiguity is designed to counterbalance this (if delivery is truly impossible, the agent is told to ask), but if the user notices the agent making too many wrong assumptions, the fix is to add more specific context to the request rather than disabling the skill.
