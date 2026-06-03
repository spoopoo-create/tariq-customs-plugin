---
name: douane-redaction-administrative
description: >-
  Use when the user wants to write, draft, review, correct or translate an administrative customs document (Maroc / ADII), in French or Arabic.
  Triggers: « rédige une lettre / un courrier », « recours hiérarchique / gracieux », « contestation de PV », « demande d'autorisation / d'apurement / de mainlevée », « mémoire en défense », « réclamation », « note de service / d'analyse / de synthèse », « compte rendu », « rapport », « notification », « bordereau / fiche de transmission », « relis / reformule / corrige mon courrier », « traduis cette lettre en arabe », « مراسلة إدارية », « رسالة إدارية », « طعن », « شكاية » — dans un contexte douanier marocain.
---

# Skill : douane-redaction-administrative (orchestration)

## Role
Pilote les outils du serveur MCP **Tariq Customs** pour produire ou corriger un écrit administratif douanier (FR ou AR). Le fond — textes, articles, taux, montants, références, jurisprudence, méthode rédactionnelle et formules consacrées — vient exclusivement du MCP ; cette skill décide **quel outil appeler, dans quel ordre, et quand s'arrêter**.

## Séquence optimale (appels MCP conditionnels)
Lancer `tariq_expertise("redaction-administrative")` **une seule fois** au début (méthode + verrous de forme). Puis :

1. **Qualifier la demande.**
   - Type d'écrit / destinataire / objet **ambigus** → poser les questions manquantes, **STOP** (aucun outil tant que le contexte n'est pas fixé).
   - **Relecture / correction** d'un texte fourni → sauter aux étapes 4-5 ; aucun appel matière sauf si le texte contient une référence ou un chiffre à vérifier.

2. **Charger la matière de fond.**
   - L'écrit découle d'un **dossier complet** (PV, redressement, sélectivité, compte RED échu, valeur/espèce contestée) → `tariq_analyse_case(description)` **en un seul appel** : il porte la qualification, les chiffres et la suite. Ne PAS enchaîner les outils unitaires si l'analyse globale suffit.
   - Sinon (écrit simple, un seul aspect) → `tariq_expertise(<domaine de fond visé>)` pour le cadrage, sans analyse globale.

3. **Récupérer l'ossature.** `tariq_draft_admin_letter(type)` → structure de la pièce + sources. C'est le pivot ; reprendre cette ossature et l'affiner.

4. **Sourcer / chiffrer — seulement si le contenu de la pièce en dépend** (jamais « au cas où ») :
   - Une **référence** (article, circulaire, dahir) doit figurer dans le corps → `tariq_cite_law(sujet)` ; verbatim/date exacte requis → `tariq_get_circulaire(ref)`.
   - **Montants D&T** à mentionner → `tariq_compute_duties(hs, valeur, origine?)` (sauf si déjà fournis par `analyse_case`).
   - **Valeur** contestée → `tariq_compute_customs_value(...)`. **Espèce** contestée → `tariq_classer(produit)`. **Marchandise contrôlée/prohibée** évoquée → `tariq_check_compliance(hs)`.
   - Aucun de ces enjeux présent → **ne rien appeler**, passer à 5.

5. **Mettre en forme puis livrer** : appliquer les règles de forme servies par l'expertise, passer l'auto-audit servi par l'expertise, livrer. En relecture, **signaler** chaque correction.

## Efficience (réduire tokens / latence)
- `tariq_expertise("redaction-administrative")` **une fois par session** ; ne jamais la recharger.
- Pour un dossier complet, **un seul `tariq_analyse_case`** plutôt que d'enchaîner compute/classer/cite séparément (1 appel au lieu de N).
- N'appeler un outil que si la **réponse écrite en dépend** ; pas d'appel spéculatif.
- Ne **jamais re-fetch** un texte ou un chiffre déjà obtenu dans la session ; réutiliser le résultat.
- En relecture pure, démarrer **sans appel matière** ; n'interroger le MCP que pour vérifier une référence présente dans le texte.

## Exhaustivité (à ne jamais rater pour ce domaine)
- **Type d'écrit + sens de la correspondance** identifiés *avant* de rédiger (ils conditionnent toute la forme servie par l'expertise).
- **Version applicable à la date du fait générateur** : toute référence vérifiée pour la date du dossier, pas par défaut la version courante (via `tariq_cite_law` / `tariq_get_circulaire`).
- **Toute référence ou tout chiffre** écrit dans le corps est **confirmé par un outil MCP** ou fourni par l'utilisateur ; à défaut, marqueur `[référence à confirmer]` / `[à compléter]`, jamais d'invention.
- Si l'écrit touche **valeur, espèce, liquidation, origine ou conformité** → l'outil correspondant a bien été appelé avant d'affirmer la donnée.
- **Auto-audit final systématique** (règles de forme servies par l'expertise) avant livraison ; en relecture, chaque correction est **signalée**, jamais appliquée en silence.
