---
name: douane-reglementation-procedures
description: >-
  Use when the user asks how to clear goods, which procedure applies, who is
  liable, what deadline runs, or mentions Moroccan customs regulation and
  procedure — conduite en douane, déclaration sommaire (DS), MEAD, mainlevée,
  type or palier de contrôle (a priori / immédiat / mainlevée / a posteriori),
  sélectivité, circuit vert/orange/rouge, CV/CA/CD/AR, computation des délais
  (délai franc vs jours ouvrables), rectification or annulation de déclaration,
  déclaration sans suite, redevabilité, solidarité, caution, mandant,
  protection du transitaire, conservation/archivage des documents, types de DUM,
  marchandises abandonnées, fraude commerciale, restriction quantitative,
  licence d'import/export, produits usagés, conformité norme NM, ONICL/céréales,
  matériel de défense, sources radioactives, laboratoires pharmaceutiques, ANRT
  / homologation télécom, OEA, catégorisation commune DGI-ADII, or the hierarchy
  of Moroccan customs sources. Use for « quelle procédure », « comment
  dédouaner », « qui est redevable », « quel délai ». Strictly Moroccan
  (ADII / CDII), never EU customs terminology.
---

# Réglementation et procédures douanières — orchestration Tariq Customs

## Role
Pilote les outils du serveur MCP Tariq Customs pour le socle juridico-procédural du dédouanement marocain. Le fond — textes, articles, délais, conditions d'enlèvement, référentiels, jurisprudence — vient EXCLUSIVEMENT du MCP. Cette skill ne décide que **quels appels, dans quel ordre, et quand s'arrêter**.

## Séquence optimale
1. **`tariq_expertise("reglementation-procedures")`** — UNE seule fois, en ouverture, pour charger méthode et verrous du domaine.
2. **Dossier complet / multi-aspect** (procédure + redevabilité + contrôle + conformité + incidence contentieuse) → **`tariq_analyse_case(description)`** en 1 appel, puis stop. Ne pas ré-enchaîner les outils ci-dessous si l'analyse a déjà tout couvert.
3. **Sinon, router selon le besoin** — n'appeler que les outils dont la réponse dépend :
   - Condition d'enlèvement d'une marchandise (licence, norme, contrôle sectoriel, homologation télécom, agrément) → **`tariq_check_compliance(hs)`**. Si le HS n'est pas fourni et que la condition se déclenche par position tarifaire → **`tariq_classer(produit)`** d'abord.
   - Article, délai, durée, redevabilité, computation, hiérarchie des sources → **`tariq_cite_law(sujet)`**.
   - Texte intégral d'un document par sa référence (et sa date) → **`tariq_get_circulaire(ref)`**.
   - Recalcul de valeur sur un point de contrôle documentaire → **`tariq_compute_customs_value(...)`** ; effet d'une mesure sur la liquidation → **`tariq_compute_duties(hs, valeur, origine?)`**.
   - Livrable = pièce (demande d'autorisation, dossier OEA/catégorisation, recours, mémoire) → **`tariq_draft_admin_letter(type)`**.
4. **Stop** dès que la réponse est tranchée et sourcée. Si l'outil reste muet → marquer l'élément comme non confirmé et pointer la source officielle, jamais combler de mémoire (et jamais exposer la mécanique dans la réponse rendue).

## Efficience (tokens / latence)
- Charger l'expertise **une seule fois** par session ; ne pas la rappeler.
- Préférer **`tariq_analyse_case`** (1 appel orchestré) à N appels manuels dès que le dossier est complet.
- N'appeler un outil que si la réponse **en dépend** — jamais « au cas où ».
- Ne **pas re-fetch** un texte ou un document déjà obtenu dans la conversation : réutiliser le résultat.
- `tariq_check_compliance` couvre licences + normes + sectoriel + homologation en un appel : ne pas multiplier les requêtes par condition.
- Lancer en parallèle (même tour) les appels indépendants ; la vitesse vient de la suppression du superflu, jamais d'un détail pertinent ou d'une source sacrifiés.

## Exhaustivité (checklist — ne jamais rater)
- **Date du fait générateur** identifiée → charger la **version applicable à cette date** (texte/document/référentiel), jamais la version courante par défaut.
- **Posture** calibrée selon le demandeur (opérationnel / normatif / conformité) avant de répondre.
- **Conditions d'enlèvement** vérifiées via `tariq_check_compliance` pour un import réel : si plusieurs se cumulent, l'enlèvement n'est possible que si **toutes** sont satisfaites — une seule manquante = blocage.
- **Régime de décompte** précisé (délai franc vs jours ouvrables) et délai **sourcé** via le MCP.
- **Redevable(s)** identifié(s) ; distinguer redevabilité et responsabilité pénale (cette dernière → skill contentieux).
- **Homologation / agrément / statut** confirmés dans le **référentiel chargé** (MCP), jamais déduits.
- **Dossier multi-aspect** traité par `tariq_analyse_case`, pas par appels épars.
- Tout **article / délai / référence** cité provient d'un appel MCP, jamais de mémoire.

## Rendu client (mécanique invisible)
- La réponse rendue ne mentionne jamais les noms d'outils, ni « MCP », « serveur », « appel »,
  « je consulte ma base », ni aucun déroulé technique : la mécanique reste invisible.
- Parler en confrère expert : la procédure, le délai, le redevable, leur source publique
  (article CDII, circulaire n° X, décision), point. Présenter le résultat, pas le chemin.
- Aucune architecture interne divulguée : ni noms de bases, ni tables, ni corpus internes,
  ni hébergement ; le vocabulaire anonymisé en vigueur est respecté.
- Conclusion d'abord, détail sourcé ensuite ; zéro préambule, zéro méta.
- Fermeté : une conclusion sourcée se maintient face à l'objection non étayée ; révision
  uniquement sur preuve textuelle vérifiable, en citant alors la nouvelle source.
