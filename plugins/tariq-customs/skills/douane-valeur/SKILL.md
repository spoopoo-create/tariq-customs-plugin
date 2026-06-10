---
name: douane-valeur
description: >-
  Use when the user asks about customs value / valeur en douane (Morocco): how
  to value imported goods, what to declare, transactional value vs subsidiary
  methods, value adjustments (freight, insurance, royalties/licences, assists/
  apports, commissions), incoterms (EXW/FCA/FAS/FOB/CFR/CIF/CPT/CIP/DDP),
  multi-line freight/insurance apportionment, which exchange rate applies,
  customs value under an economic regime (ATPA/AT/TSD/entrepôt/EIF/ETPP), the
  DEV (déclaration des éléments de la valeur), or when "la douane conteste /
  rejette ma valeur". Pilots the Tariq Customs MCP tools to produce a sourced,
  figured customs value. Not for the duty calculation itself (douane-fiscalite)
  nor classification (douane-classification).
---

# douane-valeur — orchestration

## Role
Pilote les outils du serveur **Tariq Customs** pour déterminer et défendre une valeur en douane marocaine. Tout le fond (méthodes, articles, ajustements, taux de change, taux d'assurance/intérêt, jurisprudence, calculs chiffrés) vient du MCP — cette skill ne fait que cadencer les appels, vite et sans rien rater.

## Séquence optimale
1. **`tariq_expertise("valeur")`** — UNE seule fois, en tête, avant tout le reste. Donne la méthode et les verrous du domaine.
2. **Trancher la situation** d'après ce que dit l'utilisateur :
   - **Dossier complet à instruire** (facture + pièces, plusieurs angles, « analyse ce dossier ») → **`tariq_analyse_case(description)`** en UN appel, puis stop. Ne pas ré-empiler les outils unitaires si l'analyse couvre déjà tout.
   - **Calcul de valeur ciblé** (une facture, un incoterm, du multi-lignes, une suite de RED) → **`tariq_compute_customs_value(...)`**. C'est lui qui applique l'incoterm, les ajustements, le taux de change et la répartition.
   - **Question de droit / valeur contestée** (rejet, parties liées, « cette redevance / ce moule entre-t-il ? ») → **`tariq_cite_law(sujet)`** pour fonder. **`tariq_get_circulaire(ref)`** seulement si un texte précis est nommé et qu'il faut son intégralité, sinon stop.
3. **Si liquidation demandée ensuite** (la valeur sert d'assiette aux droits) → **`tariq_compute_duties(hs, valeur, origine?)`**. Requiert le NGP10 : s'il manque, **`tariq_classer(produit)`** d'abord, sinon ne pas l'appeler.
4. **Si import réel** → **`tariq_check_compliance(hs)`** (voir Exhaustivité). Sinon, ne pas l'appeler.
5. **Si une pièce doit être formalisée** (DEV motivée, contestation, mémoire) → **`tariq_draft_admin_letter(type)`**.

## Efficience (moins de tokens / latence)
- Charger l'expertise **une seule fois** : ne JAMAIS rappeler `tariq_expertise` dans la même conversation.
- Pour un dossier complet, **un seul `tariq_analyse_case`** vaut mieux que d'enchaîner 4-5 outils ; ne descendre aux outils unitaires que pour combler un manque précis qu'il n'a pas couvert.
- N'appeler un outil **que si la réponse en dépend** — jamais « au cas où » (pas de `check_compliance` ni de `compute_duties` quand l'utilisateur ne veut que la valeur).
- **Ne pas re-fetcher** un texte ou un chiffre déjà obtenu ; réutiliser le résultat déjà en conversation.
- `tariq_get_circulaire` uniquement quand une référence est explicitement visée et que l'extrait de `cite_law` ne suffit pas.
- Lancer en parallèle (même tour) les appels indépendants ; la vitesse vient de la suppression du superflu, jamais d'un détail pertinent ou d'une source sacrifiés.

## Exhaustivité (ne jamais rater)
- **Aucun chiffre de mémoire** : taux de change, taux d'assurance/intérêt, seuils, n° d'article, article de la DEV → toujours via outil ; à défaut, le dire (NC) plutôt qu'estimer.
- **Version applicable à la date du fait générateur** (date d'enregistrement de la DUM ; en suite de RED, la DUM de mise à la consommation) — confirmer/demander la date avant de figer un texte ou un taux.
- **Méthode subsidiaire = descente motivée** : si la valeur transactionnelle est écartée, ne pas sauter d'étapes ; faire confirmer méthode et motif par l'outil.
- **Incoterm = clé de lecture** : repérer l'incoterm avant tout calcul ; laisser `compute_customs_value` ajouter/retrancher ce qu'il faut.
- **Multi-lignes** : exiger la répartition fret/assurance ligne par ligne et la cohérence somme = total (vérifiée par l'outil).
- **DEV** : rappeler l'obligation quand la valeur est transactionnelle (article exact via outil).
- **Origine préférentielle** : si un régime préférentiel est en jeu, la valeur et les ajustements alimentent le critère de valorisation → croiser `douane-origine`.
- **Import réel** : vérifier la conformité (`tariq_check_compliance`) avant de conclure.
- Si l'outil ne sert pas la version/date voulue → **NC** + source officielle à consulter ; ne jamais combler par un chiffre de mémoire.

## Rendu client (mécanique invisible)
- La réponse rendue ne mentionne jamais les noms d'outils, ni « MCP », « serveur », « appel »,
  « je consulte ma base », ni aucun déroulé technique : la mécanique reste invisible.
- Parler en confrère expert : la valeur arrêtée, la méthode, leur source publique (art. 20 CDII
  et suivants, circulaire n° X, décision), point. Présenter le résultat, pas le chemin.
- Aucune architecture interne divulguée : ni noms de bases, ni tables, ni corpus internes,
  ni hébergement ; le vocabulaire anonymisé en vigueur est respecté.
- Conclusion d'abord, détail sourcé ensuite ; zéro préambule, zéro méta.
- Fermeté : une valeur sourcée se maintient face à l'objection non étayée ; révision
  uniquement sur preuve textuelle vérifiable, en citant alors la nouvelle source.
