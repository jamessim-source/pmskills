---
name: prompt-engineer
description: Optimizes prompts for Gemini 3.5 Flash — diagnoses weaknesses in role separation, grounding, instruction ordering, output format, and CoT overhead, then applies Flash-specific techniques for speed and accuracy. Invoke when the user says "improve this prompt", "optimize for Gemini", "make this prompt better", "prompt engineer this", "rewrite this prompt for Flash", or shares a prompt and asks how to make it work on Gemini 3.5 Flash.
argument-hint: "[prompt to optimize]"
---

# Prompt Engineer (Gemini 3.5 Flash)

## Trigger
Activate when the user asks to "improve this prompt", "make this prompt better", "optimize this prompt", "prompt engineer this", or "rewrite this prompt" specifically to run on Gemini 3.5 Flash or when evaluating generic prompts to work on Gemini 3.5 Flash.

## Behavior

### Step 1: Analyze the Current Prompt

Read the user's prompt and diagnose issues across these dimensions:

| Dimension | What to Check |
|-----------|--------------|
| **Role & Persona** | Is there a specific role? Fits in Gemini System Instructions? |
| **Context & Grounding** | Is the LLM grounded to avoid hallucination? Are inputs formatted with delimiters? |
| **Instructions** | Are instruction steps explicit and ordered? Placed correctly at the bottom? |
| **Output Format** | Is the expected structure defined? (Plain text, JSON Schema, or markdown) |
| **Examples**| Are there 1-3 highly structured input/output example pairs? |
| **Constraints** | Are there explicit DO/DON'T rules? |
| **Latency/Cost** | Is CoT over-engineered for simple tasks costing extra latency? |

### Step 2: Apply Techniques for Gemini 3.5 Flash

Apply the relevant techniques below. Match the technique to the problem — keep it lightweight.

**System Instructions Separation**
Gemini 3.5 Flash is highly responsive to the System Instructions role. Separate roles and rules from user input.
- Strong pattern is to place the persona and core guardrails inside System Instructions, and use the User Prompt purely for the dynamic contextual data and the call to action.
- *System:* "You are a senior PM at a B2B SaaS company. You write concise SaaS PRDs."
- *User:* "Draft a PRD for: <feature>{{FEATURE}}</feature>" or "Evaluate this feature description using the system template."

**Structured Output & JSON Schema**
Gemini 3.5 Flash natively supports strict structured JSON output if configured. For standard prompts, specify structural markdown patterns.
- Weak: "Write a summary of this."
- Best: "Provide the output as a valid JSON object matching this schema: ..." or "Provide a markdown list of exactly 3 bullet points, each under 15 words."

**Lightweight Planning / Chain of Thought**
Gemini 3.5 Flash is designed for speed. Extensive inner monologue (using `<thinking>` tags) increases latency and consumes extra tokens.
- Only use CoT/planning for complex multi-step reasoning.
- Instead of `<thinking>` tags, use a lightweight, speed-friendly prompt instruction: "Briefly outline your planning steps in 1-2 sentences, then write your final response."
- Don't enforce CoT for simple tasks (writing a tweet, simple summaries).

**Anchor-Down Grounding (Instruction Location)**
Because long-context focus varies, Gemini models follow instructions best when rules are near the *end* of the prompt (fresh in the active attention window).
- Structure:
  1. System Role/Core constraints (if interface supports system prompts, otherwise at the top)
  2. Input data / references (grounding material wrapped in XML tags)
  3. Actionable instruction and formatting rules (placed at the very bottom)

**Few-Shot Examples**
Fewer, highly accurate examples work best. Provide 1-2 examples to establish format and style.
- Use explicit `<example>` tags.
- Model the exact target complexity level. One high-quality example outweighs long instructional paragraphs.

**Constraints (Explicit DO/DON'T)**
Always frame what TO do positively. For negative constraints, make them hyper-specific:
- "DO: Cite exactly which section the statistics were taken from."
- "DON'T: Mention internal server terminology or technical database fields."

**Grounding & Preventing Hallucination**
Flash is susceptible to guessing when contexts are empty.
- Add grounding cues: "If the text does not contain X, write '[MISSING SECTION]'."

### Step 3: Show the Improvement

Present the improved prompt in a code block. Then label changes specifically with Gemini 3.5 Flash improvements.

**What changed and why (Gemini Flash Optimized):**
- [Technique] → [what problem it fixes for Gemini Flash]
- [Technique] → [what problem it fixes for Gemini Flash]

### Step 4: Offer to Iterate
"Want me to adapt few-shot examples for Flash, define a solid JSON schema for native structured output, configure separate System Instructions, or optimize the instruction layout?"

---

## Full Before/After Examples

### Example 1: Vague Prompt → Gemini Flash Optimized

**Before:**
```
Write a competitive analysis of Notion.
```

Problems: No system role, instructions are at the top without grounding data, layout is loose, does not use XML tags or bottom-anchored instructions.

**After:**
```
[System Instructions]
You are a senior product strategist at a B2B SaaS KM company competing with Notion. Write with clean, professional, and technical density.

[User Prompt]
<context>
Target: Notion AI product features
</context>

Provide a comparative analysis of the target features. Structure the response as follows:

1. CORE OFFERINGS (JSON or concise markdown list of key AI features)
2. OPPORTUNITIES (Top 2 gaps or missing features we should prioritize)
3. RISKS & SOLUTIONS (1 core risk and 1 action plan)

Rule:
- Anchor all analysis to actual features. Do not use generic filler.
- Keep the total response brief and readable under 400 words.
```

**What changed:**
- Role priming separated to System Instructions → Gemini handles persona cleanly without distracting task execution.
- Instruction ordering (Bottom-anchored) → Instructions placed at the final line of the user query for maximum attention.
- XML structure → Delimited Context to differentiate variable data from instructions.

### Example 2: Weak Few-Shot → Solid Few-Shot

**Before:**
```
Rewrite these feature requests as user stories.

Feature requests:
- We need better search
- Users want dark mode
- Add CSV export
```

**After:**
```
[System Instructions]
You are an agile Product Manager. Transform user feature requests into precise user stories.

[User Prompt]
<example>
Input: "Customers want to undo actions"
Output:
Story: As a document editor, I want to undo my last 10 actions so that I can experiment without fear of losing work.
Criteria:
- Cmd+Z undoes the most recent action.
- Undo stack preserves the last 10 actions per session.
- Greyed out when no actions exist.
</example>

Transform the following feature requests into user stories. Keep stories realistic for a B2B task tool.

Input Requests:
- We need better search
- Users want dark mode
- Add CSV export
```

**What changed:**
- Clear `<example>` formatting → Defines expectations and shortens the output style.
- Bottom-anchored call to action → Tells Gemini exactly what to process after reading the example data.

### Example 3: Over-Engineered → Right-Sized for Flash Speed

**Before (too complex / slow):**
```
You are an expert-level product management consultant with 20 years of experience. Please analyze the following customer feedback and provide a comprehensive multi-dimensional assessment including sentiment analysis, theme clustering, priority scoring using the RICE framework, impact mapping, and root cause analysis. First, think step-by-step through each theme inside <thinking> tags to ensure accuracy. Make sure your thinking is extremely detailed.
```

Problems: Over-engineered task, forces extensive CoT reasoning inside `<thinking>` tags which causes slow-down, too many frameworks requested for a fast-response model.

**After (right-sized):**
```
Analyze this customer feedback. Group by theme and list their frequencies.

For the top 3 issues:
- Number of occurrences
- Dynamic quote snippet
- Next prioritized action

Feedback:
[feedback text]
```

**What changed:**
- Removed `<thinking>` block → Flash processes small-batch frequency counts directly, removing 200+ tokens of thinking time.
- Simplified analysis scope → Faster execution without compromising the quality of the raw data evaluation.

---

## Prompt Debugging for Gemini 3.5 Flash

When the improved Gemini prompt still produces bad output, use this taxonomy:

**Step 1: Identify the failure type**

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Hallucinates facts/data | Weak grounding boundary | Wrap sources in XML tags. Add: "Only use context details. Write '[MISSING]' if absent." |
| Ignores negative rules | Negative wording ("DON'T") | Rewrite positively as explicit output limits, or make restrictions final. |
| Output is right but wrong format | Prompt instructions are lost | Move format constraints to the absolute bottom of the user prompt. |
| Slow execution / high latency | Over-engineered instructions or CoT | Remove manual inner thoughts, `<thinking>` blocks, or unnecessary steps. |
| Ignores system-level tone | Inline role cluttering | Move the role declaration to the formal System Instructions setting. |

**Step 2: Test the fix**

Tell the user:
- "Here's the Gemini 3.5 Flash adjustment done"
- "Test it to verify if correctness and speed meet expectations"

---

## Anti-Patterns
- Never place critical rules at the absolute top of the user prompt if followed by massive context chunks. (Attention dilution)
- Never use vague rules like "be extremely detailed and thorough". Focus on exact lists or parameters.
- Never write overly complex formatting briefs when a simple JSON schema or concise template Markdown gets the job done cleanly and quickly.

## Rules
- Always preserve the user's intent, only rewrite the prompt structures to fit Gemini 3.5 Flash architectural advantages.
- Show clear comparative differences in Gemini optimization.
- Ensure the prompt remains highly readable and lightweight.

3. WHAT'S WEAK (3 gaps or friction points)
- For each: the issue, who it affects, opportunity for us

4. IMPLICATIONS
- 2 things we should copy and why
- 2 things we should avoid and why
- 1 opportunity they're missing that we could own

Rules:
- Be specific. "Good UX" is not analysis. Name the interaction and explain why it works.
- If you don't have data, say "[NEED: data on X]" instead of guessing.
- Keep total output under 800 words.
```

**What changed:**
- Role priming → LLM writes from a strategic perspective, not generic
- Structured output → Ensures consistent, complete analysis
- Constraints → Prevents vague filler and controls length
- Scope → "AI features specifically" prevents a surface-level overview of everything

### Example 2: Weak Few-Shot → Strong Few-Shot

**Before:**
```
Rewrite these feature requests as user stories.

Feature requests:
- We need better search
- Users want dark mode
- Add CSV export
```

Problems: No format specified, no quality bar shown, no context about the product.

**After:**
```
You are a PM turning raw feature requests into user stories for an engineering team.

For each request, produce:
- User story (As a [user type], I want [action] so that [outcome])
- Acceptance criteria (2-3 testable conditions)
- One edge case to consider

EXAMPLE:
Request: "Customers want to undo actions"
User story: As a document editor, I want to undo my last 10 actions so that I can experiment without fear of losing work.
Acceptance criteria:
- Cmd+Z undoes the most recent action within 200ms
- Undo stack preserves the last 10 actions per session
- Undo is disabled (greyed out) when no actions exist in the stack
Edge case: What happens if the user undoes a collaborative edit that another user has already built upon?

Now process these requests:
Product context: B2B project management tool for mid-market teams (50-200 people).

Feature requests:
- We need better search
- Users want dark mode
- Add CSV export
```

**What changed:**
- Few-shot example → Shows the exact quality bar and format expected
- Product context → User stories will be specific to the actual product
- Edge case requirement → Forces the LLM to think beyond the happy path
- Structured output → Consistent format across all stories

### Example 3: Over-Engineered → Right-Sized

**Before (too complex):**
```
You are an expert-level product management consultant with 20 years of
experience across consumer, enterprise, and marketplace products. You have
deep expertise in behavioral economics, jobs-to-be-done theory, the Kano
model, and design thinking. You have consulted for Fortune 500 companies
and high-growth startups alike. You approach every problem with a blend of
quantitative rigor and qualitative empathy. You always consider second-order
effects and systemic implications.

Please analyze the following customer feedback and provide a comprehensive
multi-dimensional assessment including but not limited to: sentiment analysis,
theme clustering, priority scoring using the RICE framework, impact mapping,
root cause analysis using the 5 Whys methodology, and strategic recommendations
aligned with OKR best practices.

[50 more lines of instructions...]
```

Problems: Prompt is longer than the output. Role is impossibly broad. Instructions request 8+ frameworks for a simple task. The LLM will produce mediocre output across all dimensions instead of strong output on what actually matters.

**After (right-sized):**
```
Analyze this customer feedback. Group by theme, rank by frequency, and flag the top 3 issues I should act on.

For each top issue:
- How many customers mentioned it
- Representative quote
- Suggested next step

Feedback:
[paste feedback here]
```

**What changed:**
- Removed bloated role (unnecessary for this task)
- Cut 8 frameworks down to the one that matters (theme clustering + prioritization)
- Clear, scannable output format
- The prompt is shorter than the expected output, which is almost always the right ratio for analytical tasks

---

## Prompt Debugging for Gemini 3.5 Flash

When the improved Gemini prompt still produces bad output, use this taxonomy:

**Step 1: Identify the failure type**

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Hallucinates facts/data | Weak grounding boundary | Wrap sources in XML tags. Add: "Only use context details. Write '[MISSING]' if absent." |
| Ignores negative rules | Negative wording ("DON'T") | Rewrite positively as explicit output limits, or make restrictions final. |
| Output is right but wrong format | Prompt instructions are lost | Move format constraints to the absolute bottom of the user prompt. |
| Slow execution / high latency | Over-engineered instructions or CoT | Remove manual inner thoughts, `<thinking>` blocks, or unnecessary steps. |
| Ignores system-level tone | Inline role cluttering | Move the role declaration to the formal System Instructions setting. |

**Step 2: Test the fix**

Tell the user:
- "Here's the Gemini 3.5 Flash adjustment done"
- "Test it to verify if correctness and speed meet expectations"

---

## Anti-Patterns
- Never use complex Claude-specific features like extensive `<thinking>` tags for simple tasks on Gemini 3.5 Flash.
- Never place critical rules at the absolute top of the user prompt if followed by massive context chunks. (Attention dilution)
- Never use vague rules like "be extremely detailed and thorough". Focus on exact lists or parameters.
- Never write overly complex formatting briefs when a simple JSON schema or concise template Markdown gets the job done cleanly and quickly.

## Rules
- Always preserve the user's intent, only rewrite the prompt structures to fit Gemini 3.5 Flash architectural advantages.
- Show clear comparative differences in Gemini optimization.
- Ensure the prompt remains highly readable and lightweight.
