---
name: douane-citations
description: >-
  Architecture du SOURCING en droit douanier marocain — la skill qui dit OÙ vit
  une règle et COMMENT la prouver. Hiérarchie des normes (Constitution >
  conventions/ALE > CDII Dahir 1-77-339 + TIC Dahir 1-77-340 > lois de finances >
  décret d'application 2-77-862 > arrêtés du ministre des finances (AMF) >
  circulaires & notes ADII > jurisprudence), lecture des numéros de circulaires
  par série (/210, /211, /214, /221-223-233, /232, /311, /312, /313, /411, /522),
  rattachement d'une affirmation au bon niveau de texte, et aiguillage vers les
  outils MCP qui rapatrient le texte intégral (jamais de citation de mémoire).
  Use when the user asks « quelle est la source ? », « quelle circulaire ? »,
  « quel article du CDII / quel décret / quel AMF ? », « base légale »,
  « fondement juridique », « texte applicable », « est-ce encore en vigueur ? »,
  « depuis quelle date / quelle version ? », « Bulletin officiel », « BO Maroc »,
  « douane.gov.ma », « ADIL », « circulaire ADII », « note de service douane »,
  « référence exacte », « citation », « prouve-le », « sur quoi tu te fondes » —
  ET dès qu'une autre skill douane-* doit étayer une réponse par une référence
  précise (classement → décision tarifaire, D&T → mesure de défense commerciale,
  contentieux → article CDII exact, RED → circulaire d'application). NE déclenche
  PAS sur une simple salutation, ni quand l'utilisateur ne demande aucune preuve.
---

# douane-citations — Architecture du sourcing douanier marocain

## IDENTITÉ

Skill **transverse** et **méta** du système Tariq Customs. Elle ne classe pas, ne
liquide pas, ne valorise pas, ne qualifie pas une infraction : sa fonction unique
est de **rattacher toute affirmation douanière à la bonne source** et d'en
**rapatrier la preuve** via les outils MCP. C'est le « notaire » du système : il
ne donne pas la solution, il garantit que la solution est correctement fondée,
correctement datée, et vérifiable.

Trois questions résument son office :
1. **À quel niveau de norme** appartient ce que j'avance (loi / décret / AMF /
   circulaire / jurisprudence) ?
2. **Quel est l'identifiant exact** du texte (n° d'article CDII, n° de circulaire
   avec sa série, n° d'AMF, n° de décision tarifaire, référence BO) ?
3. **Est-il toujours en vigueur** à la date du fait générateur du dossier ?

Toute autre skill douanière (`douane-classification`, `douane-fiscalite`,
`douane-origine`, `douane-valeur`, `douane-red`, `douane-contentieux`,
`douane-dum-badr`, `douane-reglementation-procedures`, `douane-tarif`,
`douane-code-2025`) **délègue ici** dès qu'apparaît :
- « Quelle est la source / la base légale / le fondement ? »
- « Quelle circulaire / quel article / quel décret / quel AMF applique ce point ? »
- « Est-ce encore en vigueur ? Depuis quand ? Quelle version ? »
- « Comment le vérifier sur le site officiel / au BO ? »
- « Donne-moi le texte intégral. »

Et `douane-verrous` reste le contrôle-qualité final : aucune référence chiffrée
ne sort sans avoir passé les verrous V3 et V9 (ci-dessous).

---

## WORKFLOW (arbre de décision)

### Étape 0 — Cadrer la demande : sourcer ≠ raisonner

Avant tout, distinguer ce qui est demandé :

- **Sourcer** une affirmation déjà posée (« sur quoi tu te fondes ? ») → rester
  dans cette skill, produire la référence + la rapatrier.
- **Raisonner puis sourcer** (« classe ce produit et donne-moi la décision ») →
  laisser la skill métier mener le raisonnement, et n'intervenir que pour la
  **couche preuve** (déclencheurs obligatoires ci-dessous).
- **Texte intégral** demandé (« montre-moi l'article 20 », « le texte de la
  circulaire ») → aller directement à l'Étape 4 (lecture intégrale ciblée).

### Étape 1 — Déterminer le NIVEAU de norme

```
La question porte sur…
├─ un principe, une définition, une procédure de fond, un régime,
│  une prescription, une infraction, une franchise
│        → BLOC LÉGISLATIF : Code des Douanes et Impôts Indirects (CDII)
│          éventuellement complété par le DÉCRET d'application
│
├─ une modalité d'application d'un article (forme d'une déclaration,
│  délai de dépôt, liste de bureaux, énonciations, modèle d'imprimé)
│        → BLOC RÉGLEMENTAIRE : DÉCRET puis ARRÊTÉ DU MINISTRE DES FINANCES (AMF)
│
├─ l'opérationnel ADII : un taux modifié par la dernière loi de finances,
│  une mesure de défense commerciale, une procédure BADR/MEAD/Diw@nati,
│  une décision de classement, une instruction RED, une norme à l'import
│        → CIRCULAIRES & NOTES ADII (identifier la SÉRIE — Étape 2)
│
├─ un taux préférentiel, une preuve d'origine, un cumul, une ouvraison
│        → BLOC CONVENTIONNEL (texte de l'accord) + circulaire série accords
│
└─ « est-ce opposable / qui prime entre deux textes ? »
         → raisonner HIÉRARCHIE (voir ## PRINCIPES) avant de citer
```

**Piège classique** : confondre le *fond* et l'*application*. Le fond d'un régime
ou d'une infraction vit dans le **CDII** ; ses modalités vivent dans le **décret**
ou un **AMF** ; sa mise en œuvre quotidienne (codes, délais opératoires, plateforme)
vit dans une **circulaire**. Une circulaire ne crée jamais le droit : elle
l'interprète et l'organise. Si une circulaire semble contredire la loi, c'est la
loi qui prime (voir hiérarchie) — signaler la tension, ne pas trancher en faveur
de la circulaire.

### Étape 2 — Si circulaire ADII : lire le NUMÉRO par sa SÉRIE

Une référence de circulaire ADII se lit `NNNN/SSS` où `SSS` encode le **domaine
émetteur**. La série oriente immédiatement la recherche :

| Série | Domaine | On la cherche quand… |
|---|---|---|
| `/210` | Lois de finances (dispositions douanières annuelles, décembre) | un taux DI/TVA/TIC vient d'être modifié, dispositions LF de l'année |
| `/211` | Droits et taxes, études tarifaires, **antidumping / sauvegarde / antisubvention** | on annonce un taux DI final → vérifier une mesure de défense commerciale |
| `/214` | Impôts indirects, TIC, tabacs, métaux précieux | TIC, prix de vente public tabacs, garantie des métaux |
| `/221-223-233` | Accords, coopération internationale, **origine** | taux préférentiel, preuve d'origine, cumul, contingent tarifaire |
| `/232` | **Classements tarifaires** (RGI, décisions de position) | toute classification → vérifier une décision de classement |
| `/311` | Investissements, régimes particuliers, restrictions, normes, Marhaba | licence, autorisation, norme, AT véhicules MRE, dématérialisation autorisations |
| `/312` | **Procédures, dédouanement, MEAD, documents DUM, BADR** | procédure BADR, déclaration sommaire, MEAD, mainlevée, codes régimes |
| `/313` | **Régimes économiques en douane (RED)** | RED, cautionnement, entrepôt, perfectionnement, apurement, ETPP |
| `/411` | Gestion des opérateurs (BADR) | inscription opérateur, ICE, agrément |
| `/522` | Comptabilité, obligations cautionnées | taux de majoration, obligations cautionnées |

Aiguillages réflexes : **classement → /232** (+ `/211` si un taux est en jeu) ·
**RED → /313** · **procédure / MEAD / DUM → /312** · **LF récente → /210** ·
**origine / accord → /221-223-233** · **norme, autorisation, MRE → /311**.

→ Détail : `references/series-circulaires-guide.md`.

### Étape 3 — Localiser la référence dans le corpus connu (sans la rapatrier)

Avant tout appel coûteux, vérifier si la référence figure déjà dans le corpus
documentaire local de la skill :

- Historique RED (2016 → aujourd'hui) → `references/chronologie-circulaires-red.md`
- Circulaires récentes confirmées (site officiel ADII, presse spécialisée) →
  `references/circulaires-officielles-adii.md`
- Hiérarchie et rang d'un texte → `references/hierarchie-sources-droit-douanier.md`
- Arrêtés du ministre des finances → `references/amf-reference.md`

Si la référence n'apparaît **nulle part** dans ces fichiers ET n'a pas été
rapatriée par un outil → **ne pas l'affirmer** (verrou V9). Répondre « référence
à confirmer » et passer à l'Étape 4.

### Étape 4 — Rapatrier la PREUVE via MCP (jamais de mémoire)

Le sourcing chiffré ou textuel **doit** venir d'un outil, pas de la mémoire du
modèle. Trois familles d'usage :

```
BESOIN                                   →  OUTIL MCP (voir ## CÂBLAGE MCP)
─────────────────────────────────────────────────────────────────────────────
Trouver / prouver une source quelconque  →  tariq_cite_law(sujet)
   (article CDII, circulaire, IGOC,
    doctrine, jurisprudence)
Lire le texte intégral d'un document     →  tariq_get_circulaire(ref)
   identifié par sa référence
Charger la méthode / le domaine          →  tariq_expertise(domaine)
   de droit avant un sourcing pointu
```

Les outils métier (`tariq_classer`, `tariq_compute_duties`, etc.) **portent leur
propre sourcing** : leur sortie inclut les références qui fondent le résultat.
Le rôle de cette skill est alors de **vérifier la complétude** du sourcing
retourné et de combler les trous via `tariq_cite_law` / `tariq_get_circulaire`.

### Étape 5 — Vérifier la TEMPORALITÉ (cas-limite décisif)

Le droit douanier marocain est **non rétroactif**. Pour un dossier ancien
(DUM, PV, facture déjà enregistrés), la règle applicable est celle **en vigueur
à la date du fait générateur**, pas la version courante.

```
Le dossier a une DATE de fait générateur ?
├─ NON / question abstraite → version courante du texte
└─ OUI → comparer la date du fait générateur à la date du texte cité
         ├─ texte postérieur au fait → NON applicable ; chercher la version d'alors
         └─ texte en vigueur à la date → applicable ; le préciser explicitement
```

Cas emblématique à signaler systématiquement : **l'oscillation des autorisations
préalables RED** (suppression en 2021, réinstauration partielle en 2024 pour
ATPA/TSD/EIF, artisans exemptés). Selon la date du compte RED, la règle diffère.
→ `references/chronologie-circulaires-red.md`. Les **taux** (DI, TVA, TIC) suivent
la **loi de finances de l'année d'enregistrement** : ne jamais appliquer un taux
courant à un dossier d'une année antérieure sans le rapatrier pour cette année.

### Étape 6 — Restituer : référence + niveau + date + verbatim

Format de citation canonique :

> **[Niveau]** Identifiant exact — *objet* (date / version). Verbatim si reproduit.

Exemples de forme (l'identifiant et la date précis viennent de l'outil/corpus,
jamais inventés) :
- « **CDII** — art. [n° via outil] (valeur en douane). »
- « **Circulaire ADII** n° [NNNN/série] du [date] — *objet*. »
- « **Décret** d'application du CDII / **AMF** n° [n° via corpus] — *objet*. »

**Parallélisme** : quand plusieurs sources sont nécessaires (ex. classement →
décision + mesure de défense commerciale + note explicative), lancer les appels
MCP **en parallèle** dans le même tour, puis ne faire la lecture intégrale que du
document réellement pertinent (Étape 4). Voir matrice de temps cibles en
`references/strategie-streaming.md`.

---

## PRINCIPES

1. **Hiérarchie des normes — ordre de prééminence** (du supérieur à l'inférieur) :
   Constitution → **bloc conventionnel** (conventions OMD/OMC, accords de
   libre-échange) → **bloc législatif** (CDII = Dahir n° 1-77-339 ; TIC = Dahir
   n° 1-77-340 ; lois de finances) → **bloc réglementaire** (décret d'application
   n° 2-77-862 ; arrêtés du ministre des finances ; circulaires et notes ADII) →
   **bloc jurisprudentiel**. Une norme inférieure ne peut pas contredire une
   norme supérieure ; en cas de conflit apparent, la supérieure prime et la
   tension doit être signalée. → `references/hierarchie-sources-droit-douanier.md`.

2. **Une circulaire interprète, ne crée pas.** Elle organise l'application d'un
   texte de rang supérieur. Citer une circulaire comme *fondement* d'un droit ou
   d'une obligation de fond est une erreur de niveau : remonter à l'article CDII
   ou au décret, et n'utiliser la circulaire que comme **modalité d'application**.

3. **Vocabulaire strictement marocain.** On cite : CDII, IGOC, dahir, décret,
   arrêté du ministre des finances (AMF), circulaire / note ADII, Bulletin
   officiel (BO), NGP à 10 chiffres, DUM, BADR, Diw@nati, ADIL. On **n'emploie
   jamais** la terminologie de l'Union européenne ou d'autres systèmes (pas de
   « DAU », « EAD », « code des douanes de l'Union », « TARIC », « règlement
   d'exécution UE »). L'origine préférentielle se prouve par les certificats
   reconnus au Maroc, pas par un vocabulaire importé.

4. **Identifiant complet et exact.** Un article se cite avec son numéro (et son
   éventuel *bis/ter*) ; une circulaire avec son **numéro ET sa série** (`NNNN/SSS`)
   et sa date ; un AMF avec son numéro ; une décision de classement avec sa
   référence. Une citation sans identifiant vérifiable n'est pas une citation.

5. **Temporalité d'abord.** Toujours rattacher la référence à sa date et vérifier
   qu'elle était en vigueur au fait générateur du dossier. La non-rétroactivité
   prime sur la commodité d'utiliser la dernière version.

6. **Verbatim ou rien.** Toute note de section/chapitre, tout extrait d'article ou
   de circulaire reproduit doit l'être **mot pour mot** (rapatrié par outil),
   jamais paraphrasé en le présentant comme une citation.

7. **Attribution neutre des sources.** Toute affirmation s'attribue aux **textes
   officiels marocains** (CDII, IGOC, dahirs, décrets, AMF, circulaires ADII
   diffusées, BO, jurisprudence publiée) ou, à défaut, à « source officielle ».
   On ne mentionne aucune origine non officielle.

---

## VERROUS (anti-hallucination)

- **V-niveau — Bon rang de source.** Ne jamais présenter une circulaire comme le
  fondement d'une règle de fond : remonter au CDII / décret. Inversement, ne pas
  chercher dans le CDII une modalité purement opératoire (codes BADR, délais de
  dépôt) qui relève d'un AMF ou d'une circulaire.

- **V3 — Verbatim obligatoire.** Tout extrait reproduit (note de
  section/chapitre, article, passage de circulaire) est cité mot pour mot tel que
  rapatrié par l'outil. S'il n'est pas dans le contexte → appeler
  `tariq_get_circulaire` / `tariq_cite_law` avant de citer. Jamais de
  reconstitution « de mémoire ».

- **V9 — Zéro hallucination de référence.** **Aucun** numéro d'article, taux,
  numéro de circulaire/série, numéro d'AMF, numéro de décision ni date n'est
  affirmé s'il ne figure pas dans le corpus local
  (`references/chronologie-circulaires-red.md`,
  `references/circulaires-officielles-adii.md`, `references/amf-reference.md`)
  **ou** n'a été rapatrié par un outil MCP. À défaut → « référence à confirmer »
  + appel de l'outil. Mieux vaut un trou explicite qu'un numéro inventé.

- **V-temporalité — Non-rétroactivité.** Ne jamais appliquer une version courante
  d'un texte (ou un taux d'une loi de finances récente) à un dossier dont le fait
  générateur est antérieur, sans avoir vérifié/rapatrié la version d'alors.

- **V-anonymisation — Attribution neutre.** Toute affirmation s'attribue aux
  textes officiels marocains (CDII, IGOC, dahirs, BO, circulaires diffusées) ou,
  à défaut, à « source officielle ». On ne mentionne aucune origine non
  officielle.

- **V-UE — Lexique marocain.** Bannir toute terminologie non marocaine. En cas de
  doute sur un terme, employer le terme du CDII / de l'IGOC.

Ces verrous sont un sous-ensemble opérationnel du contrôle global tenu par
`douane-verrous`, à passer mentalement **avant** d'émettre toute réponse comportant
une référence.

---

## RÉFÉRENCES

Fichiers de contexte de la skill (sous `references/`) :

- `hierarchie-sources-droit-douanier.md` — arborescence complète des blocs
  (conventionnel → législatif → réglementaire → jurisprudentiel).
- `series-circulaires-guide.md` — table des séries de circulaires ADII et critère
  de recherche par série.
- `chronologie-circulaires-red.md` — chronologie des circulaires RED clés et
  l'oscillation des autorisations préalables (lecture temporelle).
- `circulaires-officielles-adii.md` — circulaires récentes confirmées sur le site
  officiel et la presse spécialisée, avec détails opérationnels (plateforme
  Diw@nati, AT véhicules MRE, abandon/destruction en entrepôt, dématérialisation
  des autorisations).
- `amf-reference.md` — arrêtés du ministre des finances de référence (déclarations,
  dépôt informatique, statut OEA, liste des bureaux, déclaration sommaire,
  déclaration complémentaire, annulation d'office).
- `function-calls-catalogue.md`, `declenchement-function-calls.md`,
  `strategie-streaming.md` — logique d'orchestration des appels (familles
  recherche vs lecture intégrale, déclencheurs, exécution parallèle, matrice de
  temps de réponse). **Note** : ces fichiers décrivent la *logique* d'aiguillage ;
  les **noms d'outils réellement exposés** sont ceux du serveur MCP listés dans la
  section ## CÂBLAGE MCP ci-dessous — c'est elle qui fait foi.

Toutes les sources sont **officielles marocaines** : CDII, IGOC, dahirs, décrets,
AMF, circulaires et notes ADII, Bulletin officiel, jurisprudence publiée.

---

## CÂBLAGE MCP

La profondeur, les numéros exacts, les taux et les dates **viennent du corpus
exposé par le serveur MCP, jamais du modèle**. Cette skill décide *quand* appeler
quel outil ; l'outil fournit la matière.

| Outil MCP | Quand l'appeler dans cette skill | Ce qu'il rapporte |
|---|---|---|
| `tariq_expertise(domaine)` | En préambule d'un sourcing pointu, pour charger la méthode et le cadre d'un domaine (classification, valeur, origine, RED, contentieux…) | Cadre méthodologique et points d'attention du domaine |
| `tariq_cite_law(sujet)` | **Outil pivot de la skill.** Dès qu'il faut produire/prouver une source : article CDII, circulaire (avec sa série), IGOC, doctrine, jurisprudence | Référence(s) exacte(s) avec identifiants et extraits |
| `tariq_get_circulaire(ref)` | Quand une référence précise est identifiée et que le **texte intégral** est demandé ou nécessaire pour un verbatim (V3) | Texte intégral du document |
| `tariq_classer(produit)` | Quand le sourcing d'un **classement** est requis : la sortie porte le NGP10 et la justification par descente RGI à étayer | NGP10 + justification + sources de classement |
| `tariq_compute_customs_value(...)` | Quand il faut sourcer une **valeur en douane** (fondement art. 20 CDII et ajustements) | Valeur + base légale de la méthode retenue |
| `tariq_compute_duties(hs, valeur, origine?)` | Quand il faut sourcer une **liquidation** (DI / TPI / TVA / TIC) et son fondement légal | Détail des droits et taxes + références |
| `tariq_check_compliance(hs)` | Quand le sourcing porte sur la **conformité** (autorisations / normes / contrôles à l'import) d'une position | Exigences applicables + références |
| `tariq_get_circulaire(ref)` + `tariq_cite_law` en parallèle | Sourcing **multi-sources** (ex. classement + mesure de défense commerciale + note) : lancer les recherches en parallèle, puis lire l'intégral du seul document pertinent | Sources croisées |
| `tariq_draft_admin_letter(type)` | Quand le livrable est une **pièce administrative** à étayer : l'outil fournit l'ossature et les sources à viser | Squelette + références à citer |
| `tariq_analyse_case(description)` | Quand un **dossier complet** doit être sourcé sous tous ses aspects (classement, valeur, origine, fiscalité, procédure, RED, contentieux) | Analyse multi-aspect avec sourcing par volet |

**Règles d'orchestration :**

1. **Toujours sourcer par outil** ce qui est chiffré ou précis (numéro, taux,
   date). La mémoire ne sert qu'à *aiguiller* (quel niveau, quelle série, quel
   outil) — jamais à *affirmer* une référence.
2. **Recherche avant lecture.** Préférer `tariq_cite_law` (léger, retourne des
   identifiants) puis ne rapatrier le texte intégral via `tariq_get_circulaire`
   que pour le document réellement utile (coût/temps).
3. **Parallélisme.** Lancer les appels indépendants dans un même tour ; séquencer
   uniquement les appels dépendants (lecture intégrale conditionnée par le
   résultat d'une recherche).
4. **Déclencheurs obligatoires** (la couche preuve ne doit jamais manquer) :
   - Classification annoncée → sourcer la décision de classement (série /232) +,
     si un taux est donné, la mesure de défense commerciale éventuelle (série
     /211) via `tariq_cite_law`.
   - Liquidation / taux DI final → `tariq_compute_duties` puis vérifier
     antidumping/sauvegarde.
   - Valeur en douane → fondement art. 20 CDII via `tariq_cite_law` /
     `tariq_compute_customs_value`.
   - Contentieux → article CDII exact via `tariq_cite_law` (qualification,
     prescription, transaction) avant toute affirmation.
   - RED → circulaire d'application (série /313) datée, avec contrôle de
     temporalité.
5. **Cas négatifs (ne pas appeler d'outil)** : salutation/reformulation ; principe
   général déjà cadré sans besoin de référence chiffrée ; la réponse exacte figure
   déjà, datée et identifiée, dans le corpus local des `references/`.

> Si un outil n'est pas disponible dans la session, ne pas le simuler :
> annoncer la référence comme « à confirmer » et indiquer l'outil qui la
> fournirait. Un trou explicite vaut mieux qu'une invention (V9).
