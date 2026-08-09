# VIZ-LENS — Final Roadmap & Task Split

Goal: turn VIZ-LENS from a demo into a reliable, differentiated product.
Positioning: **generative visualization of YOUR code and concepts, verified before you see it, with an enforced learn → quiz → implement loop.** Wedge market: DSA / interview prep.

Suggested split: 4 workstreams, one owner each. Phase 1 ≈ 2 weeks alongside classes. Phase 2 is the flagship.

---

## Phase 0 — Do immediately (Day 1)

**0.1 Fix the iframe sandbox** — *(30 min, whoever touches frontend first)*
- In `visualizer.tsx`, remove `allow-same-origin` from the iframe `sandbox` attribute (the component already uses `srcDoc`, so this is a one-attribute change).
- Why: `allow-scripts allow-same-origin` together = no sandbox at all. Generated code could touch the parent app (and later, the localStorage library).
- While there: generated HTML posts a `START_QUIZ` message but no listener exists in `viz/page.tsx` (pre-existing gap, verified) — add one that reveals/scrolls to the Quiz component.
- Done when: generated viz still runs (canvas, buttons) and cannot read `window.parent` DOM/storage; quiz handoff fires via the new listener.

**0.2 De-IBM the repo** — *(VERIFIED DONE — no work needed)*
- Checked Aug 9: `grep -ri "granite\|watsonx\|ibm"` returns nothing in the codebase — IBM references only ever existed in the hackathon deck, not the repo. Just don't reuse old deck copy in README/pitch updates.

---

## Phase 1 — Core reliability & retention (~2 weeks)

### Workstream A — Verification & Auto-Repair Loop *(Owner 1, ~3 days)* — build first
- Backend pipeline after generation: render HTML in headless Chromium (Puppeteer), capture console errors, assert the 4 mandatory UI sections exist, programmatically click "Next" once.
- On failure: feed the exact error + original HTML back to Gemini for ONE repair attempt, re-test; if still failing, return a clean error to the user (no broken viz ever renders).
- Add a `verified: true/false` flag to the response — only verified output may be cached (Workstream B depends on this).
- Done when: deliberately-broken prompt cases either self-repair or fail gracefully; zero broken visualizations reach the iframe.

### Workstream B — Supabase: Semantic Cache + Library + Share Links *(Owner 2, ~4–5 days)*
1. **Semantic cache:** Supabase + pgvector. Embed the query, exact-match fast path, cosine threshold ~0.90 (start conservative, log near-misses). Store: query, embedding, verified HTML, prompt-version key (bump key whenever the Master Engine prompt changes → auto-invalidates stale entries).
2. **My Library (no-login):** localStorage stores only `{query, topic, date, quizScore, cacheId}` — never full HTML; re-fetch HTML from cache by id. Library page lists past sessions, click to reopen.
3. **Shareable links:** every verified cache entry gets a public URL (`/viz/share/[id]`). Add a Share button. Optional stretch: a public gallery page of the best cached visualizations (instant wow for new visitors, zero API cost).
- Done when: repeat query loads instantly; refresh keeps history; a share link opens on another device with no account.

### Workstream C — Learning Loop Upgrades *(Owner 3, ~3 days)*
1. **Quiz-miss recap:** replace generic recap idea — after quiz, one Gemini call taking the missed questions → misconception profile ("you're confusing the pointer update order — rewatch steps 4–6") with a deep link back into the viz at that step. Render on the quiz results screen.
2. **Context textarea (RAG-lite):** optional "paste your notes / problem statement / professor's slide text" box on input; inject into the generation prompt as grounding context. (Full PDF RAG: deferred — NotebookLM owns that fight.)
3. **Generator/critic split for the Code Judge:** keep one vendor; run the judge with an independent critic prompt (or stronger model tier) that never sees the generator's output — it judges the user's code against the topic spec only.
- Done when: failing a quiz question produces a specific, step-linked explanation; pasted context visibly changes the generated viz.

### Workstream D — Data Lens Trust Fix + UX Polish *(Owner 4, ~3 days)*
1. **Deterministic stats:** compute real aggregates in JS (means, min/max, null counts, correlations, IQR outliers) server-side across the FULL dataset; pass them to Gemini to *narrate*, not invent. LLM never states a number it wasn't handed.
2. **Latency UX:** for uncached generations, show honest progress stages — "Designing layout → Writing animation → Verifying in sandbox" (stage 3 doubles as marketing for the repair loop). Stream if feasible.
3. Freeze further Data Lens feature work after this — it's the most crowded, least differentiated front.
- Done when: every number in Key Insights traces to a computed stat; users see staged progress instead of a bare spinner.

---

## Phase 2 — Flagship differentiator (after Phase 1, ~1–2 weeks, pair up)

### Trace-Driven Visualization for code inputs *(2 people — suggest Owners 1 + 3)*
- For code/algorithm inputs: Gemini generates an *instrumented* version of the algorithm that emits a step log (`{type:"swap", i, j, state}` …). Execute it in the backend sandbox to produce a **real trace**. Gemini separately generates a renderer that animates the trace. Frames come from ground truth, not the model's imagination.
- Why this is the moat: ChatGPT's interactive learning = ~70 pre-built modules; VisuAlgo = 26 fixed algorithms; Python Tutor = real execution, no pedagogy or generative UI. Nobody does arbitrary user code + real execution + generative visuals + the quiz/judge loop. This is also the only honest answer to "how do you know the animation is correct?"
- Start with: sorting, two-pointer, BFS/DFS, DP table fills (your placement-prep bread and butter). Fall back to current one-shot mode for non-code topics.

---

## Killed / merged (so nobody builds them)
- **Dual-vendor model split (Granite):** killed with the hackathon. Principle survives as the generator/critic split in Workstream C.
- **PDF upload RAG:** deferred indefinitely; context textarea covers 80% at 2% of the cost.
- **Standalone Key Insights panel:** already exists in the API; the real upgrade is deterministic stats (D1).
- **Standalone "What You Learned" screen:** merged into the quiz-miss recap (C1).

## Build order (dependencies)
Sandbox fix → Verification loop (A) → Cache (B1) → Library (B2) → Share links (B3), with C and D in parallel after A lands. Phase 2 starts only once Phase 1 ships.
