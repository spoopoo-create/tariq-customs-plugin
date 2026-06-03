---
name: douane-classification
description: >-
  Use when the user submits a product, a technical sheet, a commercial invoice or a goods
  description to classify; asks for an HS code, a 10-digit tariff code, a tariff heading or
  sub-heading, a tariff designation; mentions a tariff descent, the General Interpretative
  Rules, section/chapter notes, the nomenclature; wants to split an invoice into tariff codes,
  hesitates between two headings, or wants to verify or contest a supplier code — even without
  saying the word "classification". Use also for bulk dossiers (multi-line, multi-invoice) to
  ventilate into tariff codes, and ahead of a customs declaration when the code must be fixed
  before computing duties.
---

# douane-classification — orchestration

## Role
Pilote les outils du serveur MCP Tariq Customs pour classer une marchandise (code tarifaire à
10 chiffres) et conclure un dossier. Toute la matière — méthode de descente, règles
d'interprétation, notes, arbre tarifaire, taux, conformité, jurisprudence — vit dans le MCP.
Cette skill ne décide rien par elle-même : elle ordonne les appels.

## Sequence optimale
1. `tariq_expertise("classification")` — UNE fois, en tout premier. Fournit méthode + verrous du
   domaine. Ne pas rappeler ensuite.
2. Dossier complet fourni (plusieurs aspects : classement + valeur + origine + conformité, ou litige) ?
   → `tariq_analyse_case(description)` en UN appel, puis sauter à l'étape 6. Sinon continuer.
3. `tariq_classer(produit)` — dès que le produit est décrit ; renvoie le code et le chemin de descente.
   - Si la description est insuffisante pour que l'outil tranche : combler le manque (recherche web
     ciblée d'abord), et seulement si décisif et introuvable, poser UNE question précise. Sinon stop.
4. Justification ou citation nécessaire (note, règle, article, décision) ?
   → `tariq_cite_law(sujet)`. Référence de document identifiée et texte intégral requis ?
   → `tariq_get_circulaire(ref)`. Si aucun sourçage demandé/utile : ne pas appeler.
5. Suite d'un dédouanement, une fois le code fixé (appeler seulement si la réponse en dépend) :
   - Valeur en douane requise → `tariq_compute_customs_value(...)`.
   - Liquidation des droits → `tariq_compute_duties(hs, valeur, origine?)` (origine si connue).
   - Conformité / contrôles à l'import → `tariq_check_compliance(hs)`.
6. Livrable = pièce administrative (demande de décision anticipée, contestation d'un code notifié) ?
   → `tariq_draft_admin_letter(type)`. Sinon, rendre le résultat.

## Efficience (tokens / latence)
- Charger l'expertise une SEULE fois (étape 1) ; ne jamais la recharger dans la même tâche.
- Dossier complet → préférer `tariq_analyse_case` (1 appel) plutôt que d'enchaîner classer +
  value + duties + compliance séparément.
- N'appeler un outil que si la réponse en dépend réellement — jamais « au cas où ». Pas de
  `tariq_compute_duties` / `tariq_check_compliance` si on ne demande qu'un code.
- Ne pas re-`tariq_classer` un produit déjà classé ni re-`tariq_get_circulaire` un texte déjà obtenu :
  réutiliser le résultat en contexte.
- Caractériser le produit à partir des pièces et du web AVANT d'appeler ; éviter les allers-retours
  d'outil pour des données déjà disponibles.

## Exhaustivité (ne jamais rater)
- Étape 1 (`tariq_expertise`) faite avant tout raisonnement de classement.
- Descente menée jusqu'au code terminal à 10 chiffres via `tariq_classer` — pas un code partiel ni
  « sorti de mémoire ».
- Toute note / règle / article / décision invoquée est confirmée par `tariq_cite_law` (ou
  `tariq_get_circulaire`), jamais affirmée de tête.
- Aucun taux ni statut de conformité hors outil : `tariq_compute_duties` / `tariq_check_compliance`.
- Date du fait générateur identifiée ; version applicable à cette date demandée à l'outil (le tarif
  est révisé périodiquement) — pas la version courante par défaut.
- Import réel → `tariq_check_compliance(hs)` avant de conclure ; origine préférentielle annoncée →
  la passer à `tariq_compute_duties(hs, valeur, origine)`.
- Dossier volumineux : toutes les lignes extraites, regroupées par nature, une descente par groupe,
  et bouclage Σ valeurs ventilées = total des factures avant de clore.
- Un produit physique = un seul code : trancher l'ambiguïté via les outils, jamais proposer deux
  codes au choix.
- Conclure (code livré ou refus motivé) — pas d'arrêt à mi-parcours.
