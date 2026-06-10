---
name: douane-tarif
description: >-
  Lecture du tarif douanier marocain via les outils du serveur Tariq Customs :
  désignation verbatim d'un code NGP, droit d'importation (DI) rattaché,
  arborescence d'un chapitre/position, flags réglementaires portés par le code.
  Use when l'utilisateur cherche « quel DI / quel droit pour le HS XXXX », « le
  taux d'importation de tel produit », « la désignation tarifaire de tel code »,
  « le libellé exact du NGP 10 chiffres », « toutes les positions du chapitre
  NN », « la sous-position de … », « le tarif Maroc », « position / sous-position
  tarifaire », « code 10 chiffres », ou veut vérifier qu'un code NGP existe.
  Distinguer : DÉCIDER du code d'un produit = classement (descente RGI) ;
  CALCULER la dette totale = liquidation. Ici on LIT le tarif.
---

# douane-tarif — pilotage des outils Tariq Customs (lecture du tarif)

## Role
Cette skill pilote les outils du serveur **Tariq Customs** pour lire le tarif (désignation, DI, arborescence, flags réglementaires). Le fond — textes, structure tarifaire, désignations verbatim, quotités, calculs, jurisprudence — provient **exclusivement** du MCP, jamais de la mémoire du modèle.

## Sequence optimale
1. `tariq_expertise("tarif")` — **une seule fois** en ouverture de session : méthode et verrous du domaine. Ne pas la rappeler ensuite.
2. **Lire la donnée tarifaire.** Que l'entrée soit un code, un produit nommé, ou un chapitre/position à déployer → `tariq_classer(<code | produit | chapitre>)` : confirme l'existence du NGP, remonte la désignation verbatim, le DI et la position dans l'arbre. C'est la **source unique** de la donnée tarifaire.
   - Besoin du libellé/texte **mot pour mot** (position, note, fondement) → `tariq_cite_law(<sujet>)`.
   - Texte intégral d'un document par sa référence → `tariq_get_circulaire(<ref>)`.
3. **Aiguillage conditionnel** — n'appeler que si la réponse de l'utilisateur en dépend :
   - La demande glisse vers « combien je paie au total ? » → `tariq_compute_duties(hs, valeur, origine?)`. Si une assiette est requise d'abord → `tariq_compute_customs_value(...)`.
   - Conformité au-delà du flag tarifaire → `tariq_check_compliance(hs)`.
   - Livrable = pièce administrative (ex. demande de décision anticipée, contestation d'un taux notifié) → `tariq_draft_admin_letter(<type>)`.
4. **Stop.** Restituer la donnée renvoyée par l'outil. Si l'outil ne couvre PAS le code ou le millésime demandé → dire « non trouvé / version indisponible » et orienter vers la source officielle ; jamais combler par un chiffre ou une désignation de mémoire.

## Efficience (réduire tokens / latence)
- Charger `tariq_expertise("tarif")` **une seule fois** ; ne pas la relancer à chaque sous-question.
- Pour un dossier **complet** mêlant plusieurs aspects (classement + valeur + liquidation + conformité), préférer **un seul** `tariq_analyse_case(description)` plutôt que d'enchaîner manuellement les outils.
- N'appeler un outil que si la réponse **en dépend** : pas d'appel « au cas où » (p. ex. ne pas lancer `tariq_compute_duties` quand on demande juste un DI).
- Ne **pas re-fetch** un texte ou une désignation déjà obtenu dans la session ; le réutiliser.
- Un **seul** `tariq_classer` suffit le plus souvent pour obtenir code + désignation + arbre + DI ; ne pas multiplier les appels sur le même nœud.
- Lancer en parallèle (même tour) les appels indépendants ; la vitesse vient de la suppression du superflu, jamais d'un détail pertinent ou d'une source sacrifiés.

## Exhaustivité (à ne jamais rater)
- **Version applicable d'abord** : déterminer la date du fait générateur (enregistrement de la DUM) → viser le **millésime du tarif en vigueur à cette date**, pas le tarif courant par réflexe. Si ce millésime n'est pas servi → le déclarer, ne pas extrapoler.
- **Confirmer l'existence du code** avant d'affirmer un DI ; code absent → le dire, jamais substituer un code voisin en silence.
- **Désignation cumulative** : restituer le fil complet (chapitre → position → sous-positions → NGP terminal), pas une ligne isolée.
- **Arborescence = discriminants** : à chaque niveau, les sous-positions se distinguent par UN
  discriminant (matière, teneur, usage, puissance, conditionnement…) ; pour DÉCIDER du code d'un
  produit, appliquer la descente par discriminants (skill douane-classification) — jamais retenir
  une ligne « au plus probable » sur un discriminant non établi.
- **DI = brique, pas dette** : un DI lu n'est pas « le montant à payer ». Question sur le total → passer par `tariq_compute_duties`.
- **Pas de préférentiel implicite** : le tarif affiche le DI plein ; toute réduction d'origine est conditionnée — la traiter via les outils dédiés, jamais par défaut.
- **Conformité** : si le contexte est un import réel, vérifier les flags/contrôles via `tariq_check_compliance` avant de conclure.

## Rendu client (mécanique invisible)
- La réponse rendue ne mentionne jamais noms d'outils, « MCP », « serveur », « appel », « je consulte
  ma base » ni déroulé technique ; aucune architecture interne (bases, tables, corpus, hébergement).
- Parler en confrère expert : la désignation, le DI, leur source publique (NGP, tarif ADII
  millésimé, circulaire n° X), point. Présenter le résultat, pas le chemin ; vocabulaire
  anonymisé respecté.
- Conclusion d'abord, détail sourcé ensuite ; zéro préambule, zéro méta. Fermeté : une donnée
  tarifaire sourcée se maintient face à l'objection non étayée ; révision uniquement sur preuve
  textuelle vérifiable, citée.
