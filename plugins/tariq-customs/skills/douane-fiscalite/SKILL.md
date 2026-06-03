---
name: douane-fiscalite
description: >-
  Pilote les outils du serveur Tariq Customs pour la fiscalité douanière à
  l'import au Maroc : liquidation des droits et taxes, taux préférentiels,
  mesures de défense commerciale, franchises et exonérations. Use when l'utilisateur
  demande « combien de droits / taxes à l'import », « calcule la liquidation /
  les D&T », « quel taux de droit d'importation / TVA / TIC », « droit
  d'importation », « TVA à l'importation », « taxe parafiscale », « antidumping »,
  « mesure de défense commerciale », « droit anti-subvention / compensateur »,
  « sauvegarde », « surtaxe », « franchise / exonération douanière », « taux
  préférentiel », « assiette TVA », « consignation antidumping », « franchise
  investissement », « combien je paie pour importer X ». Ne PAS confondre avec la
  VALEUR en douane (skill douane-valeur) ni avec la CLASSIFICATION (skill
  douane-classification).
---

# Fiscalité douanière Maroc — orchestration MCP

## Role

Cette skill ne contient aucun barème, taux, article ni méthode : elle pilote les
outils du serveur **Tariq Customs**, qui fournissent tout le fond (textes, tarif,
calculs, conditions, jurisprudence). Ici, uniquement **quels appels, dans quel
ordre, et quand s'arrêter**.

## Séquence optimale

1. `tariq_expertise("fiscalite")` — **une seule fois**, en tout début, pour charger
   méthode et points de vigilance du domaine.
2. **Dossier complet déjà décrit** (produit + valeur/incoterm + pays, ou litige
   multi-aspect) → `tariq_analyse_case(description)` en **un seul appel**, puis
   compléter seulement les trous qu'il signale. Sinon, suivre 3→7.
3. NGP10 absent → `tariq_classer(produit)`. Présent → passer.
4. Valeur en douane à arrêter (multi-articles, incoterm, ajustements) →
   `tariq_compute_customs_value(...)`. Déjà fournie → passer.
5. Liquidation chiffrée → `tariq_compute_duties(hs, valeur, origine?)` (donne
   droits, taxes, assiette, total). C'est la **source des taux** — ne rien chiffrer
   de tête.
6. Import réel ou question de conformité/autorisation →
   `tariq_check_compliance(hs)`. Pure question de taux → **ne pas appeler**.
7. Besoin de citer un fondement précis → `tariq_cite_law(sujet)` ; texte intégral
   d'un document → `tariq_get_circulaire(ref)` ; formaliser une demande de
   franchise ou une contestation → `tariq_draft_admin_letter(type)`.

## Efficience

- Charger l'expertise **une seule fois** par session ; ne pas la recharger.
- Pour un dossier complet, **préférer `tariq_analyse_case` (1 appel)** plutôt
  qu'enchaîner classer + value + duties + compliance.
- N'appeler un outil que si la réponse en **dépend** ; jamais « au cas où »
  (ex. pas de `check_compliance` ni de `cite_law` sur une simple demande de taux).
- Ne **pas re-fetch** un texte, un NGP10, une valeur ou une liquidation déjà
  obtenus dans la conversation : réutiliser le résultat.
- `compute_duties` rend déjà l'assiette et le total ; ne pas refaire le calcul
  ni rappeler des outils pour des composantes qu'il a déjà fournies.

## Exhaustivité (à ne jamais rater)

- **Couple complet** : pas de liquidation sans NGP10 **et** origine prouvée.
- **Origine non prouvée** → traiter en régime de droit commun et le signaler ;
  ne jamais retenir le pays du fournisseur/chargement comme origine.
- **Préférentiel** : ne l'appliquer que sur origine + preuve valide confirmées
  par les outils, sinon droit commun + réserve.
- **Défense commerciale** : croiser produit **et** origine via `compute_duties` ;
  préciser le statut (provisoire vs définitif) rendu par l'outil.
- **Import réel** → toujours `check_compliance(hs)` (autorisations, contrôles).
- **Date du fait générateur** : demander/retenir la date et faire servir aux
  outils la version applicable à cette date, jamais la version courante par défaut.
- **Franchise/exonération** : ne l'affirmer qu'avec ses conditions confirmées
  par les outils ; sinon droit commun + réserve.
- Tout chiffre ou référence vient d'un outil ; à défaut, marquer **NC**.
