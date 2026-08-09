# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.


---

## Project Context: VIZ-LENS

Educational tool: generates interactive HTML5 visualizations of algorithms
(Concept Lens) and CSV dashboards (Data Lens). Next.js 16 (params is a
Promise in client pages — use useParams()) in frontend/,
Express in backend/server.js, Gemini 3 Flash via @google/genai.
Deployed: Vercel (frontend) + Render free tier (backend, ~512MB RAM).
Supabase (Postgres + pgvector) added: tables viz_cache, share_opens,
generation_failures; RLS enabled, no policies; new-format keys
(sb_secret_... = backend-only SUPABASE_SERVICE_KEY).

### My work, in order (Samaira)
1. Phase 0: iframe sandbox fix + add missing START_QUIZ listener
   (de-IBM already verified done — repo is clean)
2. Workstream A (MINIMAL scope): verification & auto-repair loop
3. Workstream B: semantic cache + no-login library + share links
Full specs: docs/SPEC_PACK.md — follow its schemas, API contracts, and
"Cross-workstream contracts" section exactly.

### Hard rules
- Never commit .env or secrets. Supabase secret key stays backend-only.
- Only verified:true HTML may be inserted into viz_cache.
- Requests carrying user context/code never read from or write to shared cache.
- Generated-viz iframe must stay origin-less: no allow-same-origin, ever.
- Model routing: Gemini 3 Flash for viz generation + repair ONLY;
  Gemini Flash-Lite for all structured-JSON tasks (quiz, judge, recap,
  Data Lens narration). Embeddings: single Gemini embedding model, no substitutes.
- All LLM calls route through one entry point (llm.js) taking a task
  profile; no direct model calls elsewhere.
- Data Lens: chart selection is rule-based in JS; LLM only narrates
  computed stats.
- Keep Workstream A minimal: static checks + one Puppeteer smoke test +
  one repair attempt. No gold-plating.
