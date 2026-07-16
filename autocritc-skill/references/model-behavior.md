# Model Behavior Reference

How different LLM providers interpret the same Autocritic Pro instructions, and why the skill produces correct behavior across all three.

## The core insight

Different models have different base alignments — the result of their RLHF/RLAIF training. Instead of fighting against each model's natural tendency (which would require model-specific skills), Autocritic Pro defines a single routing logic that each model interprets through its own tendency. The same instruction produces different-but-correct behavior on each model.

## Claude (Constitutional AI — Cautious, Bottom-Up)

**Base tendency**: Claude was trained with Constitutional AI, giving it a strong bias toward avoiding harm, hallucinations, and wasted tokens. Its instinct when facing any ambiguity is to stop, map the terrain, and ask clarifying questions before acting. It builds understanding from the details up to the whole.

**How Ambiguity Routing lands**: Claude naturally gravitates toward the HIGH ambiguity path more often than other models. The skill restrains this by defining HIGH strictly as "missing information that makes delivery impossible." For anything less, the instruction to "assume best practice and execute" overrides Claude's default caution on minor details.

**How Invisible Self-Review lands**: Claude already has a strong tendency to review its own work. The skill makes this review more rigorous by introducing the perspective shift technique (act as external reviewer). Without this, Claude's self-review tends to be "politically correct" — gentle, diplomatic, reluctant to flag its own work harshly. The instruction to evaluate "as if reviewing a junior's PR" gives Claude permission to be more brutal.

**How No Fake Testing lands**: Claude is the model most likely to produce verbose self-verification. It commonly says things like "Let me review this code for potential issues..." followed by a gentle critique. The skill redirects this energy into the invisible loop, producing cleaner output.

**How the 4-Option Rule lands**: Claude triggers the 4-Option Rule most naturally among the three models, which is correct — it's the model that benefits most from structured option presentation when genuine architectural ambiguity exists.

## Gemini (Agent-Oriented — Proactive, Top-Down)

**Base tendency**: Gemini was designed with agent work, long context, and multitasking in mind. Its instinct is to receive a task, identify the most likely path, and execute immediately — adjusting on the fly if something goes wrong. It starts from the whole picture and fills in details as needed. It prefers asking forgiveness to asking permission.

**How Ambiguity Routing lands**: Gemini naturally gravitates toward the LOW/MEDIUM path, which is its strength. The HIGH ambiguity path acts as a safety valve — it only activates when the task is genuinely impossible to execute without user input. This preserves Gemini's speed while preventing it from charging into situations where it would produce fundamentally wrong output.

**How Invisible Self-Review lands**: This is where Gemini benefits most. Gemini's proactivity is a double-edged sword — it executes fast but sometimes skips details. The invisible self-review forces a "pause" in its reasoning to check for edge cases, security issues, and logic flaws before delivery. The perspective shift is critical for Gemini because without it, the model's speed bias means it rarely second-guesses itself.

**How No Fake Testing lands**: Gemini has a habit of appending test blocks or execution simulations as "proof." The skill eliminates this waste.

**How the 4-Option Rule lands**: Gemini rarely triggers the 4-Option Rule because most questions have a clear best practice that Gemini is comfortable assuming. This is correct behavior — the rule exists for genuine architectural crossroads, and Gemini's tendency to just pick the best option and go is usually what the user wants.

## GPT-4 (Instruction-Following — Precise, Rule-Based)

**Base tendency**: GPT-4 is the most "robotic" of the three in the sense that it follows instructions precisely as written, without strong inherent tendencies in either direction. It doesn't have Claude's constitutional caution or Gemini's agent proactivity. It does what the prompt says.

**How Ambiguity Routing lands**: GPT-4 executes the HIGH/LOW/MEDIUM branching logic exactly as specified. It doesn't over-ask (like Claude might without the skill) or under-ask (like Gemini might). The routing logic gives GPT-4 a decision framework it wouldn't have developed on its own.

**How Invisible Self-Review lands**: GPT-4 follows the 4-step sequence (generate, shift, correct, deliver) as written. It's the model most likely to execute the perspective shift as a literal instruction rather than an instinctive behavior change.

**How No Fake Testing lands**: GPT-4 is the model most prone to generating fake test blocks. The prohibition directly addresses this habit.

**How the 4-Option Rule lands**: GPT-4 follows the activation criteria precisely. If all three conditions are met, it presents 4 options. If not, it doesn't. No behavioral bias in either direction.

## Behavioral comparison table

| Behavior | Claude | Gemini | GPT-4 |
|:---------|:-------|:-------|:------|
| Default ambiguity classification | Leans HIGH (cautious) | Leans LOW/MEDIUM (proactive) | Follows the rules as written |
| Ambiguity Filter effect | Restrains over-asking on minor details | Acts as safety valve for truly impossible scenarios | Gives GPT-4 a decision framework |
| Self-review without skill | Diplomatic, gentle, sometimes verbose | Fast but may skip edge cases | Often generates fake test blocks |
| Self-review with skill | More rigorous, brutal, concise | Catches details its speed would miss | Follows the shift-correct-deliver sequence |
| 4-Option Rule activation | Triggers naturally for architecture questions | Rarely triggers (correct — most things have a best practice) | Triggers exactly when criteria are met |
| Biggest improvement from skill | Less chatty, fewer unnecessary questions | Fewer edge-case bugs, less fake verification | No more fake test blocks, better decision framework |