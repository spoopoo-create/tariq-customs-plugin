---
name: douane-contentieux
description: >-
  Use when the user faces a Moroccan customs dispute, penalty, or recovery question — wording such as
  contentieux douanier, infraction / délit / contravention, fraude / contrebande, amende, transaction /
  « transactionnelle », PV / procès-verbal, saisie / confiscation / consignation, prescription, défense,
  recours / opposition / appel / cassation, nullité du PV, requalification, redressement / AR, recouvrement
  forcé / ATD / contrainte par corps, MRE véhicule saisi, manquant / excédent, non-apurement RED,
  détournement de destination, valeur / espèce / taux préférentiel contesté, devises non déclarées, or cumul
  douane × changes. Sert l'opérateur (combien pour solder) comme l'avocat (stratégie de défense).
---

# Skill : douane-contentieux

## Role
Pilote les outils du serveur MCP **Tariq Customs** pour traiter un dossier de contentieux et de recouvrement douanier marocain. Le fond — textes, qualification, classes, barèmes transactionnels, calculs, prescription, jurisprudence — provient exclusivement des outils MCP. Cette skill décide seulement **quels appels faire, dans quel ordre, et quand s'arrêter**.

## Sequence optimale
1. `tariq_expertise("contentieux")` — **une seule fois**, en ouverture, avant tout raisonnement.
2. Fixer la cible : **opérateur** (combien pour solder) ou **avocat** (stratégie). Cela conditionne la sortie, pas les appels.
3. `tariq_analyse_case(description)` — **appel pivot** dès que les faits + pièces sont réunis : il porte qualification, classe, article, pénalité transactionnelle, circonstances et alerte cumul change en **un seul appel**.
4. Préalable conditionnant la qualification, sinon sauter :
   - espèce contestée → `tariq_classer(produit)` ;
   - valeur contestée → `tariq_compute_customs_value(...)`.
5. `tariq_compute_duties(hs, valeur, origine?)` — quand un montant est demandé (les D&T fondent l'amende, les intérêts, le total opérateur). Sauter sinon.
6. `tariq_check_compliance(hs)` — seulement si la marchandise peut être prohibée/contrôlée et que cela change l'issue. Sinon ne pas appeler.
7. `tariq_cite_law(sujet)` — **avant d'écrire** tout article, délai ou référence, et pour confirmer qu'un article est en vigueur.
8. `tariq_get_circulaire(ref)` — seulement si `tariq_cite_law` renvoie une référence dont le texte intégral est nécessaire (verbatim, datation).
9. `tariq_draft_admin_letter(type)` — seulement si l'utilisateur demande une pièce (contestation PV, recours, demande de transaction, mémoire).
10. Dès que rien d'utile ne dépend d'un outil restant → **s'arrêter** et rédiger.

## Efficience (réduire tokens / latence)
- Charger l'expertise **une seule fois** ; ne jamais ré-appeler `tariq_expertise`.
- Pour un dossier complet, **`tariq_analyse_case` d'abord** : un appel couvre ce que plusieurs outils donneraient séparément. N'ajouter d'autres appels que pour ce qu'il ne couvre pas (chiffrage fin, texte verbatim, pièce).
- N'appeler un outil que si la **réponse en dépend** ; jamais « au cas où ».
- Lancer en parallèle les appels indépendants (ex. `tariq_classer` et `tariq_compute_customs_value`) ; ne séquencer que lorsque l'un alimente l'autre (classement/valeur → `tariq_compute_duties`).
- Ne **jamais re-fetch** un article, une circulaire ou un calcul déjà obtenu dans la session ; réutiliser le résultat.
- Réutiliser le `hs` issu de `tariq_classer` pour `tariq_compute_duties` / `tariq_check_compliance` au lieu de relancer une recherche.

## Exhaustivite (checklist — ne rien rater)
- **Cible fixée** dès le départ : opérateur ou avocat.
- **Date du fait générateur identifiée** : charger via les outils la **version du texte applicable à cette date** (le droit n'est pas rétroactif), pas la version courante par défaut.
- **Préalable contesté établi** : si l'espèce ou la valeur est en jeu, la trancher (classer / valeur) avant de chiffrer.
- **D&T en jeu chiffrés** quand l'utilisateur veut un montant (sortie opérateur).
- **Prescription vérifiée** via les outils : distinguer action douanière et recouvrement ; tenir compte des actes interruptifs / suspensifs datés.
- **PV examiné** quand il existe : régularité confirmée par les outils.
- **Conformité** consultée si import réel d'une marchandise potentiellement contrôlée.
- **Origine préférentielle** vérifiée quand un taux préférentiel est contesté.
- **Cumul change** : dès que devises, rapatriement ou transfert apparaissent, le signaler et router vers `douane-changes-oc`.
- **RED** : si l'infraction touche un régime économique, confier la reconstitution de compte / apurement à `douane-red` ; ne garder ici que l'infraction / pénalité / recouvrement.
- **Tout chiffre, article, délai, référence** vient des outils ; si un point n'est pas couvert, le dire plutôt que combler.
