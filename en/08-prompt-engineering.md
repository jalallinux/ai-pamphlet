> [Back to index](README.md) · Previous: [07 — CLAUDE.md and AGENTS.md](07-claude-md-agents-md.md) · Next: [09 — Essential tools](09-essential-tools.md)

# Chapter 08 — Prompt Engineering

## The framework

Anthropic orders the techniques **by impact**. Start at the top; if it works, don't go further down.

```
1. Be clear and direct           ← biggest effect, lowest cost
2. Give examples (multishot)
3. Let it think (chain of thought)
4. Use XML tags
5. Assign a role (system prompt)
6. Prefill the response
7. Chain prompts
8. Optimize for long-running work
```

---

## Part 1 — Anatomy of a good prompt

| Part | Question it answers | Example |
|---|---|---|
| **Role** | Who are you? | "You are a security engineer" |
| **Task** | What should you do? | "Review this function for SQL injection" |
| **Context** | What should you know? | the code, the DB schema, constraints |
| **Constraints** | What must you not do? | "Don't change the function signature" |
| **Format** | What shape is the output? | "A table: file, line, severity, fix" |
| **Examples** | What does correct look like? | one real input/output pair |
| **Success** | When are you done? | "When `npm test` passes" |

---

## Part 2 — Be clear and direct

**The golden test:** show your prompt to a new colleague with zero context. If they're confused, the model is too.

| ❌ | ✅ |
|---|---|
| "Make this code better" | "Refactor this function so every network error retries 3 times with exponential backoff. Do not change the function signature." |
| "Write tests" | "Write Jest tests for `calculateTax` in `src/tax.ts`. Cover: zero rate, negative amount, very large amount, null input." |
| "Find the bug" | "Users report staying logged in after a password reset. Trace the flow from `POST /auth/reset` through session creation and find the cause." |

**Rules:**
- Say what to do, not what not to do
- Give numbers: "3 cases", not "several cases"
- Describe the output you want

---

## Part 3 — Give examples (multishot)

The most effective technique after clarity. Three to five diverse examples.

```xml
<examples>
<example>
<input>getUserById</input>
<output>get_user_by_id</output>
</example>
<example>
<input>parseHTTPResponse</input>
<output>parse_http_response</output>
</example>
<example>
<input>id</input>
<output>id</output>
</example>
</examples>

Convert this to snake_case: <input>fetchAPIToken2</input>
```

Examples should cover the **edge cases**, not just the happy path.

---

## Part 4 — Let it think (chain of thought)

For analysis, math, debugging, and multi-step decisions.

```xml
Debug this failing test.

Think through it in <thinking> tags first:
1. What does the test assert?
2. What does the code actually return?
3. Where do they diverge?

Then give your answer in <answer> tags.
```

In Claude Code, keywords increase the thinking budget:

```
think < think hard < think harder < ultrathink
```

```
ultrathink about the best architecture for this rate limiter
```

⚠️ More thinking = more tokens = more cost. Don't use it for easy work.

---

## Part 5 — XML tags

Claude is specifically trained on XML tags. They're the best tool for separating sections.

```xml
<context>
This is a Next.js 15 app using the App Router and Drizzle ORM.
</context>

<code>
export async function getUser(id: string) { ... }
</code>

<task>
Add caching with a 60-second TTL.
</task>

<constraints>
- Do not change the function signature
- Use the existing Redis client from src/lib/redis.ts
- Cache misses must not throw
</constraints>
```

There are no standard tag names — just be **consistent**.

---

## Part 6 — Assign a role

```
You are a senior database engineer specializing in PostgreSQL
performance. You care about query plans and index selectivity,
not ORM syntax.
```

What it changes: depth of expertise, vocabulary, and what the model considers important.

In Claude Code, you do this via a [subagent](05-subagents.md).

---

## Part 7 — Chain prompts

Not decomposing complex work means mediocre output at everything.

```
Prompt 1: "Read the codebase and explain the current auth architecture"
   ↓ (review the output)
Prompt 2: "Based on that, give me a plan for adding OAuth"
   ↓ (correct the plan)
Prompt 3: "Implement step 1 of the plan"
```

In Claude Code: **Plan Mode is exactly this**, automatically.

---

## Part 8 — Code-specific patterns

### TDD pattern

```
1. Write tests for this spec. Do not write implementation yet.
2. Run them and confirm they fail for the right reason.
3. Now implement until they pass. Do not modify the tests.
```

The strongest pattern in AI-assisted coding. Tests give an objective success criterion and stop the model from pretending it works.

### Explore → Plan → Code → Commit

```
1. Read src/auth/ and tell me how sessions work. Don't write code.
2. Plan how to add refresh tokens. Show me the plan.
3. Implement step 1 of the plan.
4. Run the tests.
5. Commit.
```

### Visual feedback loop

```
Take a screenshot of localhost:3000 with Playwright.
Compare it to mockup.png.
Fix the differences. Screenshot again. Repeat until they match.
```

An objective target produces markedly better output.

---

## Part 9 — Anti-patterns

| Anti-pattern | Why it's bad | Instead |
|---|---|---|
| "Fix everything" | Vague goal = vague output | One specific job |
| Infinite context | Attention gets diluted | `/clear` between tasks |
| No success criterion | The model decides when it's done | A test or objective condition |
| "Are you sure?" with no reason | The model may flip a correct answer | Show the specific error |
| Accepting output uncritically | Bugs reach production | Always review |

---

## Part 10 — Pocket checklist

```
□ Would a new colleague understand this?
□ Is there one goal, not five?
□ Did I provide the necessary context?
□ Are the constraints explicit?
□ Is the output format stated?
□ Is "done" objectively defined?
□ Would an example help?
□ Should I split this into several prompts?
```

---

## Sources

- Prompt engineering overview — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
- Be clear and direct — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- Multishot prompting — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting
- Chain of thought — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought
- XML tags — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags
- Chain prompts — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/chain-prompts
- Prompt library — https://docs.claude.com/en/resources/prompt-library
- Interactive tutorial — https://github.com/anthropics/prompt-eng-interactive-tutorial
- Claude Code best practices — https://www.anthropic.com/engineering/claude-code-best-practices
