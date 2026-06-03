---
name: douane-red
description: >-
  Use when the user works on Moroccan customs economic/suspensive regimes (RED): warehousing, EIF, AT, ATPA (inward processing), TSD, transit, ET/ETPP/ETPP+ES, drawback, prior export. Use when they mention a RED account, sommier, apurement/imputation, MAC in suite of a regime, échéancier, échu account, déchets régularisation, cautionnement, no-drawback, or describe an import-transform-export / reimport-after-foreign-processing flow. Also use when they ask which suspensive regime applies, want a clearance account reconstructed from DUMs, an échu sommier diagnosed and each régularisation option priced, or RED crossed with origin and duties.
---

# douane-red — orchestration

## Role
Pilot the Tariq Customs MCP tools for Moroccan economic/suspensive customs regimes. This skill only sequences calls. All substance — regime rules, legal texts, deadlines, thresholds, formulas, rates, account methodology — lives in the MCP server. Never answer RED substance from memory; obtain it from a tool.

## Optimal sequence
1. `tariq_expertise("red")` — call ONCE at the very start of a RED matter, before reasoning. It frames method and locks. Do not repeat it in the conversation.
2. Full multi-aspect file (a sommier to clean up, chained regimes, RED + origin + duties, échu accounts to price) → `tariq_analyse_case(description)`. ONE call that orchestrates the dimensions; structure the answer from its output. Prefer this over manually chaining the tools below.
3. Otherwise, call only what the answer depends on:
   - HS/NGP code needed (input or compensating product) → `tariq_classer(product)`. Never derive a code yourself.
   - Customs value for a MAC in suite of a regime → `tariq_compute_customs_value(...)`.
   - Duties liquidation once value + reference date are fixed (e.g. pricing a régularisation option) → `tariq_compute_duties(hs, value, origin?)`.
   - Compliance / licence status of a good (eligibility filter, prohibitions) → `tariq_check_compliance(hs)`.
   - A precise article, deadline, threshold, circular ref, decision, or exact wording → `tariq_cite_law(subject)`.
   - Full text of a referenced document → `tariq_get_circulaire(ref)`.
   - Skeleton of an administrative piece (authorization/extension request, contesting a liquidation d'office) → `tariq_draft_admin_letter(type)`.
4. Stop when the user's question is answered. If a tool returns nothing usable, say so plainly; do not invent.

## Efficiency (tokens / latency)
- Load expertise exactly once. Re-reading it wastes the budget.
- For a complete file, one `tariq_analyse_case` beats N separate calls — fewer round-trips, lower latency.
- Call a tool only when the answer genuinely depends on its result. Never "just in case".
- Do not re-fetch a text or code already returned earlier in the conversation; reuse it.
- Batch independent calls (e.g. classify an input and check its compliance) in one turn rather than serially.

## Exhaustiveness (never skip)
- Open with `tariq_expertise("red")` before any RED reasoning.
- Resolve the applicable regime before pricing anything — never jump to duties on an unqualified flow.
- Anchor every cited rule, deadline, threshold, code, or circular to a tool result, not memory.
- Date sensitivity: identify the file's operative date (opening DUM, apurement/MAC DUM, PV, invoice, origin proof) and obtain the version in force on that date via `tariq_cite_law` before citing.
- A real import with later release → run `tariq_check_compliance` and surface any licence/prohibition constraint.
- Account work (sommier, apurement, échu, reconstitution) → get the deadlines/thresholds/formula from the MCP; show the reference date for any MAC and give every remaining balance an explicit outcome.
- Preferential origin or export under a trade agreement in play → obtain the no-drawback position and the agreement version via the tools; never settle it from memory.
- For an échu account: price each régularisation option from `tariq_compute_duties` / `tariq_compute_customs_value` so the recommendation is grounded.
