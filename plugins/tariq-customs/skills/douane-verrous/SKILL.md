---
name: douane-verrous
description: >-
  Use when any Moroccan customs answer is being PRODUCED, reviewed, or audited — whether or
  not an explicit check is requested. Use to vet a colleague's reply, a draft, or a pipeline
  output before delivery; to control a customs declaration line-up or a value breakdown before
  it ships to a client; and whenever the user says "audit this", "check the rigour", "is this
  sourced?", "zero hallucination", "quality-check", or "re-read this customs answer". Use also
  as the last filter before emitting any answer that carries a rate, an article number, a delay,
  a regime code, a circular reference, or a transaction percentage — to confirm each came from a
  tool and nothing was invented, mis-dated, or mis-attributed.
---

# douane-verrous — orchestration

## Role
Pilote les outils du serveur MCP Tariq Customs comme contrôle-qualité transverse : avant de livrer
toute réponse douane Maroc, garantir que chaque chiffre et chaque référence provient bien d'un
outil, à la bonne date. La matière — méthode, garde-fous, textes, taux, calculs, jurisprudence —
vit dans le MCP. Cette skill ne tranche rien sur le fond : elle ordonne les appels et vérifie que
la couche preuve ne manque jamais.

## Sequence optimale
1. `tariq_expertise("verrous")` — UNE fois, en tout premier. Fournit la méthode de contrôle et les
   garde-fous du domaine. Ne pas rappeler ensuite.
2. Production à auditer = dossier complet (plusieurs aspects : classement + valeur + origine +
   conformité, ou contentieux multi-aspect) ?
   → `tariq_analyse_case(description)` en UN appel, qui porte la qualification fine et le sourcing,
   puis sauter à l'étape 6. Sinon continuer, ciblé par élément à vérifier.
3. Un code tarifaire est affirmé sans descente, ou doit l'être ?
   → `tariq_classer(produit)`. Pas de descente par l'outil = code non livrable. Sinon ne pas appeler.
4. Un élément chiffré reste à fonder — n'appeler que celui dont la réponse dépend :
   - valeur en douane affirmée/contestée → `tariq_compute_customs_value(...)` ;
   - taux ou liquidation affichés → `tariq_compute_duties(hs, valeur, origine?)` (origine si connue) ;
   - conformité / contrôles à l'import en jeu → `tariq_check_compliance(hs)`.
5. Numéro d'article, délai, référence de circulaire/décision à confirmer ?
   → `tariq_cite_law(sujet)` (léger, retourne des identifiants). Citation verbatim ou version datée
   d'un texte précis requise ? → `tariq_get_circulaire(ref)`. Si aucun sourçage en jeu : ne pas appeler.
6. Livrable = pièce administrative à contrôler ?
   → `tariq_draft_admin_letter(type)` (ossature + sources à viser). Sinon, rendre le verdict :
   la production passe, ou corriger / re-sourcer par outil / marquer l'élément non confirmé.

## Efficience (tokens / latence)
- Charger l'expertise une SEULE fois (étape 1) ; ne jamais la recharger dans la même tâche.
- Dossier complet → préférer `tariq_analyse_case` (1 appel) plutôt que d'enchaîner classer + value +
  duties + compliance + cite séparément.
- N'appeler un outil que si la conclusion en dépend — jamais « au cas où ». Pour un audit qui ne porte
  que sur des libellés et la forme, aucun appel chiffré n'est requis.
- Recherche avant lecture : `tariq_cite_law` d'abord (identifiants) ; ne rapatrier le texte intégral
  via `tariq_get_circulaire` que pour le document réellement utile.
- Lancer les appels indépendants dans un même tour ; séquencer seulement les dépendants (lecture
  intégrale conditionnée par une recherche préalable).
- Ne pas re-`tariq_get_circulaire` un texte déjà obtenu ni re-`tariq_classer` un produit déjà descendu :
  réutiliser le résultat en contexte.

## Exhaustivité (ne jamais rater)
- Étape 1 (`tariq_expertise`) faite avant tout contrôle.
- Tout code affirmé est confirmé par descente via `tariq_classer` — pas un code partiel ni « de tête ».
- Aucun taux, liquidation ou statut de conformité hors outil : `tariq_compute_duties` /
  `tariq_compute_customs_value` / `tariq_check_compliance`, sinon l'élément reste non confirmé.
- Tout numéro d'article, délai et référence de circulaire/décision est confirmé par `tariq_cite_law`
  (ou `tariq_get_circulaire`), jamais affirmé de mémoire.
- Date du fait générateur identifiée ; version applicable à cette date demandée à l'outil (les textes
  sont révisés périodiquement) — jamais la version courante par défaut pour un dossier ancien.
- Import réel → `tariq_check_compliance(hs)` avant de conclure ; origine préférentielle annoncée →
  la passer à `tariq_compute_duties(hs, valeur, origine)`.
- Déclencheurs à balayer avant de clore (chacun via l'outil idoine s'il s'active) : mesure de défense
  commerciale sur la position, origine préférentielle et statut no-drawback, délai de régime proche
  d'échéance, prescription, norme/licence à l'import, franchise, suspension de l'année.
- Dossier multi-lignes : toutes les lignes attribuées (code, origine, quantité, valeur), un produit =
  un seul code, et bouclage Σ valeurs ventilées = total des factures avant livraison.
- Conclure par un verdict ferme — livrer, ou corriger / re-sourcer par outil / marquer l'élément non
  confirmé ; jamais combler par déduction, jamais s'arrêter à mi-parcours.
