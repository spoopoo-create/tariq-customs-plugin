---
name: douane-citations
description: >-
  Sourcing et preuve en droit douanier marocain — rattacher une affirmation à la
  bonne source officielle et en rapatrier le texte via les outils Tariq Customs.
  Use when the user asks « quelle est la source ? », « quelle circulaire ? »,
  « quel article / quel décret / quel arrêté ? », « base légale », « fondement
  juridique », « texte applicable », « est-ce encore en vigueur ? », « depuis
  quelle date / quelle version ? », « Bulletin officiel », « BO Maroc »,
  « douane.gov.ma », « ADIL », « référence exacte », « citation », « prouve-le »,
  « sur quoi tu te fondes » — ET dès qu'une autre réponse douanière (classement,
  liquidation, valeur, origine, contentieux, RED, procédure) doit être étayée par
  une référence précise. NE déclenche PAS sur une simple salutation, ni quand
  l'utilisateur ne demande aucune preuve.
---

# douane-citations — orchestration

## Role
Pilote les outils du serveur MCP **Tariq Customs** pour SOURCER et PROUVER une
affirmation douanière marocaine. Tout le fond (textes, taux, dates, hiérarchie des
normes, séries de circulaires, jurisprudence) vient du MCP : cette skill décide
seulement *quel outil appeler, dans quel ordre, et quand s'arrêter*.

## Séquence optimale
1. **`tariq_expertise("citations")` — UNE seule fois** en début de session pour
   charger méthode et points d'attention du domaine. Ne pas la rappeler ensuite.
2. **Cadrer la demande** :
   - Pur sourcing d'une affirmation isolée → aller en 3.
   - **Dossier complet à sourcer sous tous ses aspects** →
     `tariq_analyse_case(description)` en **un seul appel** (il porte son propre
     sourcing par volet), puis STOP. Ne pas réenchaîner les outils ci-dessous.
   - Texte intégral d'un document déjà identifié par sa référence → aller en 4.
3. **Trouver / prouver la source** → `tariq_cite_law(sujet)` (outil pivot :
   renvoie identifiants exacts + extraits, niveau de norme et version).
4. **Texte intégral** (verbatim requis, ou référence déjà connue) →
   `tariq_get_circulaire(ref)`. Ne l'appeler **que** sur le document réellement
   pertinent renvoyé en 3.
5. **Si le sourcing est adossé à un calcul ou un statut** (la réponse en dépend) :
   - classement → `tariq_classer(produit)` ;
   - valeur en douane → `tariq_compute_customs_value(...)` ;
   - liquidation DI/TPI/TVA/TIC → `tariq_compute_duties(hs, valeur, origine?)` ;
   - conformité à l'import → `tariq_check_compliance(hs)` ;
   - pièce administrative → `tariq_draft_admin_letter(type)`.
   Ces outils renvoient déjà leurs références : vérifier la complétude, combler un
   trou uniquement via `tariq_cite_law`/`tariq_get_circulaire`, sinon STOP.
6. Outil indisponible → ne pas simuler : annoncer la référence « à confirmer » et
   nommer l'outil qui la fournirait.

## Efficience (tokens / latence)
- `tariq_expertise` : **une fois par session**, jamais re-chargée.
- **Dossier complet → `tariq_analyse_case` en 1 appel** au lieu d'enchaîner N
  outils volet par volet.
- **Recherche avant lecture** : `tariq_cite_law` (léger, rend des identifiants)
  d'abord ; ne rapatrier le texte intégral via `tariq_get_circulaire` que pour le
  seul document utile.
- N'appeler un outil **que si la réponse en dépend** — jamais « au cas où » ; ne
  jamais **re-fetch** un texte déjà obtenu dans la session.
- Lancer en parallèle (même tour) les appels indépendants ; séquencer seulement
  les appels dépendants (lecture intégrale conditionnée par une recherche).

## Exhaustivité (ne jamais rater)
- **Identifiant complet** restitué pour chaque source (tel que renvoyé par
  l'outil), jamais une citation sans référence vérifiable.
- **Niveau de norme** : pour une règle de fond, remonter au texte fondateur ; ne
  pas s'arrêter à un texte d'application. Laisser `tariq_cite_law` trancher le rang.
- **Temporalité / version applicable** : si le dossier a une date de fait
  générateur, sourcer la version **en vigueur à cette date** (droit non
  rétroactif), pas la version courante, et le préciser explicitement.
- **Verbatim** : tout extrait reproduit est celui rapatrié par l'outil, jamais
  reconstitué de mémoire.
- **Couche preuve à ne jamais omettre** : classement → décision de classement
  (+ mesure de défense commerciale si un taux est avancé) ; liquidation/taux →
  fondement via `tariq_compute_duties` puis vérification antidumping/sauvegarde ;
  valeur → fondement légal de la méthode ; contentieux → article exact avant toute
  affirmation ; RED → circulaire d'application datée, temporalité contrôlée ;
  origine préférentielle → preuve d'origine et version de l'accord.
- Référence ni dans le contexte ni rapatriée par un outil → « référence à
  confirmer » + appel de l'outil. Un trou explicite plutôt qu'un numéro inventé.
