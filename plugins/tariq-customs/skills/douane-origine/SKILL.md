---
name: douane-origine
description: >-
  Use when the question is about origin of goods for Moroccan customs: country
  of origin, preferential vs non-preferential origin, an origin proof (EUR.1,
  EUR-MED, A.TR, Form A, certificate of origin, supplier/exporter declaration,
  approved exporter), a trade agreement or preferential rate (EU, EFTA, Turkey,
  Agadir, US, UK, UAE, Arab League/GZALE, AfCFTA/ZLECAf, GSP, PEM / revised
  rules), origin criteria (wholly obtained, sufficient working, cumulation,
  direct transport, tolerance), the no-drawback rule and its interaction with
  suspensive economic regimes, or post-clearance origin verification /
  authenticity. Also when asked "which rate for this origin?", "is this
  certificate valid?", "is origin proven?", "can I cumulate?".
---

# Skill : douane-origine

## Role
Pilot the Tariq Customs MCP server for origin questions. This skill only sequences
tool calls; the substance (legal texts, agreements, list rules, rates, validity
periods, jurisprudence) lives in the MCP — never restate it from memory.

## Sequence optimale (ordered MCP calls)
1. `tariq_expertise("origine")` — call ONCE at the start of an origin matter. Loads the
   method and the locks (date of the operative event, sourcing discipline). Do not re-call.
2. Full dossier described (product + declared origin + agreement + documents, possibly
   spanning value / economic regime / dispute)? → `tariq_analyse_case(description)` in ONE
   call, then stop; only top up with a single targeted tool below if one answer is missing.
3. Otherwise drive step by step, calling a tool only when the answer depends on it:
   - Origin not sourced anywhere → origin = null, no preference. Stop (no further calls).
   - Need the tariff position (the list rule depends on it) → `tariq_classer(product)`
     BEFORE any origin rule.
   - Origin criterion is value-based, or comparing upstream suspension vs preference →
     `tariq_compute_customs_value(...)`.
   - Need the rate (preferential if origin proven, else full) →
     `tariq_compute_duties(hs, value, origin?)`.
   - Non-preferential origin triggers a quota/trade-defence measure, or the product needs
     regulatory checks → `tariq_check_compliance(hs)`.
   - Need a rule / article / agreement / circular / decision, or any figure, date,
     validity period, threshold → `tariq_cite_law(subject)`.
   - Read one identified document in full → `tariq_get_circulaire(ref)`.
   - Produce a piece (post-clearance verification request, contested preference refusal,
     a-posteriori duplicate, approved-exporter letter) → `tariq_draft_admin_letter(type)`.

## Efficience (fewer tokens / less latency)
- Load `tariq_expertise("origine")` once per matter; never reload it.
- For a complete dossier prefer ONE `tariq_analyse_case` over chaining many tools.
- Call a tool only when an answer depends on it — never "just in case".
- Never re-fetch a text already returned; reuse what `tariq_cite_law` /
  `tariq_get_circulaire` already gave you.
- Any figure / date / number / exact text → go to a tool; do not guess and then verify.
- Lancer en parallèle (même tour) les appels indépendants ; la vitesse vient de la suppression
  du superflu, jamais d'un détail pertinent ou d'une source sacrifiés.

## Exhaustivite (never skip, for origin)
- Distinguish origin from provenance / billing / carrier; if origin is not sourced, return
  null — never fall back to shipper/seller/loading country.
- For a real import: confirm the preference trilogy — valid proof + origin criteria met +
  direct transport. One missing link → full rate.
- Fix the tariff position (`tariq_classer`) before applying any origin list rule.
- Match the declared country to the applicable agreement and its required proof; obtain
  every validity period / threshold / rate via tools, not memory.
- If PEM applies, confirm which rule-set governs the operation (one set per operation).
- Whenever an upstream suspensive economic regime is in play, resolve the no-drawback
  question via the tools before concluding.
- Judge as of the date of the operative event; if the applicable version is missing, say so
  rather than extrapolate.
- For preference granted/refused, surface the rate and its basis via `tariq_compute_duties`
  + `tariq_cite_law`, and flag any red flag (expired proof, broken direct transport, a proof
  used for the wrong purpose).

## Rendu client (mécanique invisible)
- La réponse rendue ne mentionne jamais les noms d'outils, ni « MCP », « serveur », « appel »,
  « je consulte ma base », ni aucun déroulé technique : la mécanique reste invisible.
- Parler en confrère expert : l'origine retenue (ou null), le taux, leur source publique
  (accord, protocole, article CDII, circulaire n° X, certificat), point. Présenter le
  résultat, pas le chemin.
- Aucune architecture interne divulguée : ni noms de bases, ni tables, ni corpus internes,
  ni hébergement ; le vocabulaire anonymisé en vigueur est respecté.
- Conclusion d'abord, détail sourcé ensuite ; zéro préambule, zéro méta.
- Fermeté : une conclusion sourcée se maintient face à l'objection non étayée ; révision
  uniquement sur preuve textuelle vérifiable, en citant alors la nouvelle source.
