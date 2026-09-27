# Agent2Test — Cahier des charges

> Étape 2, première partie : les exigences · version 0.1 soumise à ratification · 25 septembre 2026
> Amont : [`00-cadrage.md`](00-cadrage.md) (hypothèses H, tensions T) · [`01-architecture.md`](01-architecture.md) (décisions D, invariants I, points de contrôle CP)

## 1. Objet du document

Ce cahier des charges dit ce qu'Agent2Test doit faire et à quoi on le reconnaît. Il ne dit pas comment : les contrats, les schémas et les algorithmes relèvent des spécifications détaillées, qui en découlent.

Il suit le format de la mission : une règle vérifiable, des hypothèses, des décisions, puis des US « En tant que / je veux / afin de ». Chaque critère d'acceptation `CA-n.m` y est observable et binaire. Le document s'arrête aux critères : les tests qui les vérifient relèvent de la stratégie et du plan de tests (étape 3).

Les US sont rangées selon les sept phases de l'axiome. Leur rédaction suit la convention d'entrée de l'annexe A : le module `plan` pourra importer ce document tel quel, et la recette du kit partira de son propre cahier des charges.

## 2. Règle

> Pour une demande donnée, Agent2Test désigne les tests essentiels parmi les scénarios existants, les rend exécutables, les exécute sur l'application en marche sous contrôle humain, et produit un journal de tests opposable. Il ne modifie jamais l'application et n'adopte jamais seul une règle nouvelle.

## 3. Contexte

- **Application cible** : front Angular 20, API Spring Boot (Java 21), base PostgreSQL, authentification SSO. Elle tourne déjà, sur le poste ou en recette (précision 2). Ses composants d'IHM relèvent du design system maison Indigo, appelé à être remplacé à moyen terme : les cibles par rôle et nom accessible survivront à ce remplacement, des sélecteurs propres à Indigo non.
- **Amont** : pour une demande, Agent2Dev produit une analyse (fichiers impactés) et un cahier des charges (US, critères `CA-n.m`). Les scénarios existent (précision 1).
- **Lignée** : le banc check-lists (bus, journal, verdict, relevés Playwright) et Agent2Dev (agent générique piloté par fiches, observant, apprenant, serveur MCP pour Gemini).
- **Utilisateurs** : testeurs automaticiens, développeurs, DSI.

## 4. Décisions tranchées

| # | Décision | Source |
|---|---|---|
| 1 | Les US et les scénarios existent ; le kit ne les rédige pas. | précision 1 |
| 2 | L'application est lancée et connectée ; le kit ne la démarre ni ne l'arrête. | précision 2 |
| 3 | L'authentification passe par un SSO ; le kit ne détient aucun identifiant. | précision 3 |
| 4 | Le moteur par défaut est Gemini Code Assist, relié par MCP comme pour Agent2Dev. | précision 4 |
| 5 | Les données du kit vont dans PostgreSQL sous Docker, par agent. | précision 5 |
| 6 | L'axiome 7E structure le système à toutes les échelles. | précision 6 |
| 7 | Playwright est piloté comme une bibliothèque depuis Node, sans `.spec.ts`. | demande initiale |
| 8 | La chaîne suit la règle de la mission : demande → US → critères numérotés → tests, sans Gherkin. | règle de mission |

Les décisions d'architecture D1 à D20 restent dans `01-architecture.md` §12 ; ce document ne les répète pas.

## 5. Hypothèses

Les hypothèses H1 à H13 du cadrage s'appliquent ; un critère qui en dépend la cite. Ce document en ajoute six. Ce sont des valeurs par défaut, que la configuration humaine peut changer :

| # | Hypothèse | Utilisée par |
|---|---|---|
| H14 | Budget d'exécution du palier P2 : 45 minutes par campagne. P1 et P3 n'y sont pas soumis. | CA-8.5 |
| H15 | Budget du relevé dynamique : 30 écrans par campagne. | CA-5.4 |
| H16 | Conservation des preuves (captures, traces) : 30 jours. Journal et grand livre sont conservés sans limite. | CA-29.3 |
| H17 | Langue de l'IHM, des messages et du journal de tests : français. Identifiants techniques sans accents. | CA-26.6 |
| H18 | Méta-critères d'O2, repris d'Agent2Dev : fenêtre de 10 décisions, exposition minimale de 3, marge de retrait de 15 points de taux de validation. Une suspension allant dans le sens sûr, un seuil bas est acceptable. | CA-20.1, CA-20.3 |
| H19 | Réparations : 2 au plus par parcours et par campagne. | CA-12.3 |

## 6. Acteurs

| Acteur | Rôle | Points de contrôle |
|---|---|---|
| Testeur automaticien | ouvre les campagnes ; ratifie sélections, mises en place et verdicts | CP-0, CP-1, CP-2, CP-3 |
| Développeur | exploite les défauts présumés et leurs preuves ; peut ratifier une mise en place | CP-2 |
| Référent qualité | ratifie ou refuse les changements de règles | CP-4 |
| DSI | gouvernance en lecture, conformité, choix du moteur | — |
| RSSI et conformité | autorisent le moteur externe (T1), confirment le classement AI Act (H12) | — |
| Opérateur | toute personne qui établit la session SSO | CP-0 |
| Moteur (Gemini ou autre) | rédige sélections, parcours et réparations ; ne décide jamais | aucun |

## 7. Objectifs mesurables

| Objectif | Indicateur | Cible |
|---|---|---|
| Tester l'essentiel | part des critères de la demande dotés d'un scénario qui sont couverts par la sélection | 100 % (I15) |
| Tester juste | part des échecs dont la cause présumée est confirmée en CP-3 | mesurée dès la première campagne ; cible fixée après 10 campagnes |
| Rejouer à l'identique | part des rejeux d'un parcours ratifié, application inchangée, qui donnent le même verdict | 100 %, cause `ENVIRONNEMENT` exclue |
| Mettre en place sans réécrire | part des parcours générés ratifiés en CP-2 sans correction | mesurée dès la première campagne ; cible fixée après 10 campagnes |
| Garder la main | nombre de règles devenues actives sans ratification en CP-4 | 0 |
| Prouver | nombre de verdicts sans preuve associée | 0 |

## 8. Exigences

### E1 — Éléments : le plan et la demande

#### US-1 — Importer le plan de tests existant

En tant que testeur automaticien, je veux importer les US, leurs critères et les scénarios existants, afin de disposer d'un plan canonique traçable.

- CA-1.1 : L'import accepte la sortie `cahier` d'Agent2Dev et tout fichier conforme à la convention de l'annexe A (H1).
- CA-1.2 : Chaque US, critère et scénario reçoit un identifiant canonique `<espace>/<code>` ; deux éléments de même identifiant bloquent l'import, et le message cite leurs deux emplacements.
- CA-1.3 : Un scénario qui ne référence aucun critère existant est importé avec la marque « orphelin ».
- CA-1.4 : Un critère qu'aucun scénario ne couvre porte la marque « trou de couverture » ; aucun scénario n'est créé pour lui.
- CA-1.5 : Un scénario dont un prérequis sort du périmètre (batch, redémarrage, écriture directe en base) porte la marque « non automatisable », avec son motif.
- CA-1.6 : Réimporter des sources inchangées ne crée aucune version ; une source modifiée crée une version nouvelle du plan, et la précédente reste consultable.

#### US-2 — Ouvrir une campagne à partir d'une demande

En tant que testeur automaticien, je veux soumettre une demande, afin d'ouvrir une campagne de tests ciblée sur ce qu'elle change.

- CA-2.1 : Une demande comprend un texte et, en option, la sortie `analyse` d'Agent2Dev, une liste de fichiers modifiés ou une référence git.
- CA-2.2 : Un critère cité par la demande et absent du plan est signalé avant la sélection.
- CA-2.3 : Le plan et le référentiel sont figés à l'ouverture de la campagne : une modification ultérieure des sources ne change ni sa sélection ni ses verdicts.
- CA-2.4 : Une demande étrangère aux tests (modifier le code, déployer, administrer) est refusée avec le motif « hors périmètre » (I2).

### E2 — Espace : l'application, sa session, sa mémoire

#### US-3 — Cartographier les sources Angular

En tant que testeur automaticien, je veux une carte des écrans tirée des sources du projet, afin de relier une demande aux écrans qu'elle touche.

- CA-3.1 : La carte liste chaque route, y compris celles chargées à la demande (`loadComponent`, `loadChildren`), avec son composant et ses gardes.
- CA-3.2 : Les gabarits sont analysés par le compilateur Angular du projet ; la syntaxe de contrôle (`@if`, `@for`, `@switch`) n'y produit aucune erreur.
- CA-3.3 : Chaque élément interactif porte un rôle présumé, un nom accessible (libellé, `aria-label`, texte ou clé de traduction résolue) et, s'il existe, son identifiant de test.
- CA-3.4 : Chaque élément interactif sans nom accessible figure dans la liste des fragilités de ciblage.
- CA-3.5 : Un second scan sans fichier modifié ne réanalyse aucun fichier.
- CA-3.6 : Le scan ne crée, ne modifie ni ne supprime aucun fichier du projet, et refuse toute lecture hors de sa racine (I3).

#### US-4 — Relier l'API aux écrans

En tant que testeur automaticien, je veux que chaque point d'API soit relié aux écrans qui l'appellent, afin qu'une demande portant sur le back sélectionne aussi des tests d'IHM.

- CA-4.1 : Les points d'API des contrôleurs Spring sont listés avec leur verbe HTTP et leur chemin.
- CA-4.2 : Chaque appel HTTP d'un service Angular est apparié à un point d'API, avec la confiance « exacte », « par motif » ou « non apparié ».
- CA-4.3 : Pour un fichier modifié, `impact` renvoie les écrans atteints et, pour chacun, le chemin qui le justifie : fichier → point d'API → service → composant → route.

#### US-5 — Relever les écrans en marche

En tant que testeur automaticien, je veux un relevé des écrans impactés tels que l'application les affiche, afin de préparer des cibles fidèles au rendu réel.

- CA-5.1 : Le relevé ne joue que des navigations : aucun clic, aucune saisie, aucune soumission.
- CA-5.2 : Chaque écran relevé produit un instantané ARIA et une capture dont les zones sensibles sont masquées.
- CA-5.3 : Tout écart entre deux relevés du même écran est signalé.
- CA-5.4 : Le relevé s'arrête au budget d'écrans (H15) et liste les écrans non relevés.

#### US-6 — Établir et surveiller la session SSO

En tant qu'opérateur, je veux m'authentifier moi-même et laisser le système réutiliser ma session, afin que les tests traversent le SSO sans que le système détienne mes identifiants.

- CA-6.1 : Session absente ou expirée : le système ouvre une fenêtre de connexion visible et ouvre CP-0, sans rien y saisir lui-même.
- CA-6.2 : CP-0 ne se ferme que lorsque le système a atteint seul une page protégée avec la session obtenue.
- CA-6.3 : Aucun identifiant ni jeton de session n'apparaît dans la configuration, le journal, une preuve ou un contexte transmis au moteur (I11).
- CA-6.4 : Les traces et relevés réseau conservés sont expurgés des en-têtes `Authorization`, `Cookie` et `Set-Cookie`.
- CA-6.5 : Le fichier de session est hors dépôt et lisible par son seul propriétaire (mode 0600).
- CA-6.6 : Une redirection vers l'origine du fournisseur SSO pendant un scénario donne la cause `SESSION_EXPIREE` ; après CP-0, le scénario est rejoué depuis son début et aucun échec ne lui est imputé.
- CA-6.7 : Les stratégies `profil`, `cdp` et `etat` se choisissent par configuration, sans changement de code.

#### US-7 — S'appuyer sur la mémoire du projet

En tant que testeur automaticien, je veux que les agents s'appuient sur le plan, la carte et les campagnes passées, afin que leurs propositions tiennent compte de l'historique.

- CA-7.1 : La recherche fonctionne sans modèle de plongement, par le plein texte seul ; avec un modèle local, plein texte et vecteurs sont fusionnés.
- CA-7.2 : Tout fragment injecté dans une consigne appartient aux versions épinglées de la campagne.
- CA-7.3 : Les identifiants des fragments injectés sont journalisés avec l'appel au moteur.
- CA-7.4 : Avant injection, les secrets, adresses électroniques et noms de personnes déclarés sont masqués, et un fragment trop long est tronqué avec la marque « tronqué ».

### E3 — Engendrent : sélection, mise en place, exécution

#### US-8 — Sélectionner les tests essentiels

En tant que testeur automaticien, je veux une sélection motivée et priorisée des scénarios à jouer, afin de tester l'essentiel de la demande sans tout rejouer.

- CA-8.1 : Le palier P1 contient tous les scénarios qui couvrent un critère cité par la demande ; la règle est vérifiée après la revue du moteur (I15).
- CA-8.2 : Le palier P3 contient tous les scénarios déclarés « socle » (I15).
- CA-8.3 : Le palier P2 est ordonné par score décroissant, et chaque score s'affiche décomposé facteur par facteur.
- CA-8.4 : Chaque scénario retenu et chaque scénario écarté porte un motif d'une phrase.
- CA-8.5 : Le palier P2 tient dans le budget de campagne (H14).
- CA-8.6 : La sélection liste les trous de couverture de la demande.
- CA-8.7 : Moteur indisponible : la sélection déterministe est présentée seule, avec la marque « sans revue du moteur ».
- CA-8.8 : Aucune préparation ne démarre avant la décision en CP-1 ; une correction en CP-1 exige un commentaire.

#### US-9 — Amorcer le profil d'environnement

En tant que testeur automaticien, je veux que le système propose le profil d'exécution à partir de la configuration du projet, afin de mettre en place les tests sans rédiger cette configuration.

- CA-9.1 : Le profil proposé contient l'URL de base, les origines autorisées (dont celle du fournisseur SSO), la stratégie de session, l'attribut d'identifiant de test, les fichiers de traduction et les zones à masquer par écran.
- CA-9.2 : Chaque valeur proposée cite le fichier du projet dont elle vient, ou porte la marque « à renseigner ».
- CA-9.3 : Aucune campagne ne s'exécute tant que le profil n'est pas ratifié en CP-2.
- CA-9.4 : Le même kit cible le développement ou la recette par simple changement de profil.

#### US-10 — Générer les parcours

En tant que testeur automaticien, je veux que chaque scénario retenu devienne un parcours exécutable, afin de le jouer sans écrire de code de test.

- CA-10.1 : Un parcours est une donnée conforme au schéma des parcours ; une action hors du jeu fermé est rejetée avant toute exécution (I4).
- CA-10.2 : Une sortie non conforme revient au moteur avec la liste de ses écarts, 3 fois au plus ; au-delà, CP-2 présente l'échec.
- CA-10.3 : Chaque critère couvert par le parcours est porté par au moins une étape `verifier`.
- CA-10.4 : Chaque cible est résolue dans la carte de l'écran attendu, ou porte la marque « non résolue ».
- CA-10.5 : Les stratégies de ciblage suivent l'ordre fixé — rôle et nom accessible, libellé, identifiant de test, texte, texte indicatif, CSS — et toute cible CSS porte un motif.
- CA-10.6 : Toute valeur saisie par une étape de mutation porte le préfixe `A2T-<campagne>-` (H7).
- CA-10.7 : Chaque parcours porte son origine : moteur, fiche et version, empreinte du prompt, ratificateur (I14).

#### US-11 — Valider un parcours par une exécution probatoire

En tant que testeur automaticien, je veux voir un parcours nouveau ou modifié s'exécuter avant de le ratifier, afin de ne valider que ce qui fonctionne.

- CA-11.1 : L'exécution probatoire produit une trace Playwright consultable et une capture par étape `verifier`.
- CA-11.2 : CP-2 présente côte à côte le parcours, son résultat probatoire et ses preuves.
- CA-11.3 : Un parcours ratifié et inchangé ne repasse pas par CP-2.
- CA-11.4 : Toute modification d'un parcours ratifié crée une version nouvelle, soumise à CP-2.

#### US-12 — Réparer un parcours

En tant que testeur automaticien, je veux qu'un parcours en défaut soit réparé à partir de son diagnostic, afin de ne pas réécrire un test pour un écran qui a changé.

- CA-12.1 : Une réparation part du diagnostic de l'échec et d'un relevé neuf de l'écran concerné.
- CA-12.2 : L'opérateur peut désigner l'élément voulu dans le navigateur ; la cible est alors construite à partir de sa désignation.
- CA-12.3 : Au-delà du budget de réparations (H19), le parcours est présenté en CP-3 comme non réparé.
- CA-12.4 : Une réparation crée une version nouvelle du parcours, soumise à CP-2.

#### US-13 — Exécuter une campagne

En tant que testeur automaticien, je veux que la sélection ratifiée s'exécute seule jusqu'au point de contrôle suivant, afin de ne pas surveiller chaque étape.

- CA-13.1 : Avant le premier scénario, une sonde vérifie que l'application répond ; sinon, la campagne est refusée avec ce motif.
- CA-13.2 : Un parcours n'attend que des états ; aucune étape n'attend une durée fixe.
- CA-13.3 : Une navigation vers une origine non autorisée est bloquée avant chargement ; l'étape échoue et le journal indique l'origine visée (I12).
- CA-13.4 : Une étape de suppression dont la cible ne porte pas le préfixe de campagne n'est pas jouée ; elle ouvre un point de contrôle explicite (I13).
- CA-13.5 : Une étape `verifier`, ou une étape dont l'action n'a pas eu lieu, est retentée deux fois au plus ; une étape de mutation exécutée n'est jamais rejouée. Un succès au réessai classe l'étape `INSTABILITE`, jamais « réussie ».
- CA-13.6 : Chaque étape produit une ligne de journal ; chaque étape `verifier` et chaque échec produisent une capture ; chaque scénario produit une trace ; les erreurs de console et les réponses HTTP de statut 400 ou plus sont relevées.
- CA-13.7 : Une seule campagne s'exécute à la fois sur une application cible ; les autres attendent en file.
- CA-13.8 : L'arrêt d'urgence, depuis le tableau de bord ou le MCP, suspend la campagne au plus tard à la fin de l'étape en cours.
- CA-13.9 : Après un redémarrage du processus, la campagne reprend au début du scénario interrompu, sans perte de journal.

#### US-14 — Piloter par points de contrôle

En tant que testeur automaticien, je veux que le système s'arrête aux points de contrôle définis et nulle part ailleurs, afin de garder la main sans tout surveiller.

- CA-14.1 : CP-0 à CP-4 s'ouvrent aux moments fixés par l'architecture (§5.2) ; aucune phase ne consomme une sortie non ratifiée.
- CA-14.2 : Chaque décision enregistre son auteur, son canal (tableau de bord ou MCP), sa date et son commentaire.
- CA-14.3 : Le niveau d'autonomie de chaque point est lu dans une configuration humaine qu'aucune action du système ne modifie (I8).
- CA-14.4 : Une décision CP-4 reçue par le canal MCP est refusée (I10).

### E4 — État : verdicts et observabilité

#### US-15 — Juger et nommer la cause

En tant que développeur, je veux un verdict par critère, avec sa cause présumée et ses preuves, afin de savoir d'emblée si l'échec vient de l'application, du parcours ou de l'environnement.

- CA-15.1 : Chaque étape reçoit un statut parmi « réussie », « échouée », « bloquée », « non jouée » ; chaque critère, parmi « conforme », « non conforme », « non vérifié ».
- CA-15.2 : Chaque échec reçoit une cause parmi `DEFAUT_APPLICATIF`, `DEFAUT_PARCOURS`, `SESSION_EXPIREE`, `ENVIRONNEMENT`, `INSTABILITE`, `INDETERMINE`, et cite les signaux qui la fondent.
- CA-15.3 : Un critère dont une preuve manque est « non vérifié », jamais « conforme » (I5).
- CA-15.4 : Chaque verdict porte l'empreinte SHA-256 de ses preuves.
- CA-15.5 : Une contestation en CP-3 est consignée à côté du verdict ; le verdict initial reste inchangé (I6).
- CA-15.6 : En simulation, chaque défaut injecté — cible obsolète, session expirée, défaut applicatif, instabilité, application injoignable — reçoit la cause attendue.

#### US-16 — Observer le fonctionnement (premier ordre)

En tant que testeur automaticien, je veux suivre en direct le fonctionnement du système, afin de voir où il peine.

- CA-16.1 : Chaque appel entre agents produit une trace portant corrélation, parent, échelle, phase et vecteur des sept durées.
- CA-16.2 : Pour chaque campagne, le journal contient à chaque échelle les sept phases, dans l'ordre E1 → E7 ; tout écart est signalé comme violation de l'invariant I1.
- CA-16.3 : Les indicateurs de l'architecture (§6.2) se consultent par agent, phase et échelle, et se filtrent par campagne.
- CA-16.4 : Une dérive d'indicateur au-delà de son seuil lève une alerte visible dans le tableau de bord.
- CA-16.5 : L'arbre des traces d'une campagne se déplie de la campagne jusqu'à l'étape.
- CA-16.6 : Quand l'export OTLP est activé, un collecteur OpenTelemetry reçoit les traces de la campagne.

### E5 — Expression : le journal de tests

#### US-17 — Tenir un journal inaltérable

En tant que DSI, je veux un journal qu'on ne peut pas réécrire sans que cela se voie, afin que les verdicts soient opposables.

- CA-17.1 : Toute modification ou suppression d'une ligne du journal par un agent est refusée par la base.
- CA-17.2 : `verifierChaine` détecte une altération faite en base par un compte privilégié et nomme la première ligne rompue.
- CA-17.3 : Chaque ligne porte l'échelle et la phase 7E qui l'ont produite.

#### US-18 — Produire le journal de tests

En tant que testeur automaticien, je veux un journal de tests par campagne, lisible par les équipes et exploitable par les outils, afin de rendre compte et d'alimenter l'intégration continue.

- CA-18.1 : Le journal de tests contient les rubriques de l'architecture (§4.7).
- CA-18.2 : Il est produit en Markdown, JSON, JUnit XML et HTML, à partir de la même source.
- CA-18.3 : Le JUnit XML est lu par le rapport de tests de GitLab CI : un cas par scénario, un échec par scénario échoué, avec sa cause.
- CA-18.4 : La matrice critères × verdicts couvre tous les critères de la demande, trous de couverture compris.
- CA-18.5 : Tout contenu produit par le moteur y est marqué comme tel, avec moteur, fiche et version (I14).
- CA-18.6 : L'empreinte de fin de chaîne qui y figure est celle du journal à la clôture de la campagne.

### E6 — Évolutif : apprentissage et réflexivité

#### US-19 — Apprendre des décisions humaines

En tant que référent qualité, je veux que les corrections et rejets commentés deviennent des leçons candidates, afin que les mêmes erreurs ne reviennent pas.

- CA-19.1 : Chaque correction ou rejet commenté en CP-1, CP-2 ou CP-3 produit une leçon candidate, liée à sa fiche, à son moteur et à sa décision source.
- CA-19.2 : Aucune leçon n'entre dans une fiche avant sa ratification en CP-4.
- CA-19.3 : Une leçon ratifiée ne s'applique qu'au moteur sur lequel elle a été apprise ; l'étendre à un autre moteur est une ratification distincte (T8).

#### US-20 — Observer les critères, les observateurs et l'apprentissage (second ordre)

En tant que référent qualité, je veux savoir si les critères automatiques, les alertes et les leçons du système sont justes, afin de corriger la manière dont il juge, et pas seulement ce qu'il juge.

- CA-20.1 : Pour chaque contrôle automatique, l'accord avec les décisions humaines est mesuré par fiche, version et moteur, dès l'exposition minimale atteinte (H18).
- CA-20.2 : La précision des alertes O1 est mesurée à partir de leur qualification par l'opérateur : pertinente ou non.
- CA-20.3 : Une leçon active dont l'effet dégrade le taux de validation au-delà de la marge (H18) est suspendue automatiquement, avec son motif au grand livre (I9).
- CA-20.4 : Un écart entre la répartition des causes présumées et les reclassements humains en CP-3 lève une alerte au référent (garde anti-Goodhart).
- CA-20.5 : Les propositions du `reflexif` et leurs effets mesurés sont visibles dans la vue gouvernance.
- CA-20.6 : Aucun module du premier ordre n'appelle un module du second ordre, ce que vérifie le graphe des appels du bus.

#### US-21 — Recalibrer sous gouvernance

En tant que référent qualité, je veux ratifier ou refuser chaque recalibrage proposé, preuves à l'appui, afin que le système n'évolue que sous contrôle.

- CA-21.1 : Chaque proposition nomme la règle, l'échelle et la phase touchées, la valeur actuelle, la valeur proposée, les preuves et l'effet attendu.
- CA-21.2 : Aucune proposition ne sort des bornes constitutionnelles (architecture §6.6) ; une valeur hors bornes saisie en CP-4 est refusée.
- CA-21.3 : Une règle ratifiée s'applique à partir de la campagne suivante ; une campagne en cours garde ses versions épinglées (T4).
- CA-21.4 : Chaque ratification, refus, suspension et retour arrière est inscrit au grand livre, chaîné par empreintes.

#### US-22 — Versionner les prompts

En tant que référent qualité, je veux que chaque prompt envoyé au moteur soit versionné et retrouvable, afin de reproduire et d'expliquer toute sortie du moteur.

- CA-22.1 : Chaque fiche porte une version ; toute modification, leçon ratifiée comprise, crée une version composée nouvelle.
- CA-22.2 : Pour toute sortie du moteur, le prompt envoyé se reconstitue à partir de la fiche, de sa version composée et des fragments journalisés, et son empreinte égale celle du journal.
- CA-22.3 : Une campagne utilise, du début à la fin, les versions épinglées à son ouverture.

#### US-23 — Protéger le noyau

En tant que DSI, je veux que le noyau constitutionnel échappe à toute modification par le système, afin que l'auto-modification reste bornée.

- CA-23.1 : Au démarrage, une empreinte de la constitution différente de celle inscrite au grand livre empêche le démarrage, avec un message explicite.
- CA-23.2 : Aucune action d'agent n'écrit la constitution, le code, les schémas du journal ou du grand livre, ni la configuration des points de contrôle.
- CA-23.3 : Les règles modifiables se limitent à la liste fermée de l'architecture (§6.6).

### E7 — Environnement : interfaces, moteurs, exploitation

#### US-24 — Conduire une campagne depuis Gemini Code Assist

En tant que testeur automaticien, je veux conduire une campagne depuis Gemini Code Assist, afin de rester dans mon IDE.

- CA-24.1 : Les outils `a2t_*` de l'architecture (§4.4) sont exposés en HTTP local et en stdio.
- CA-24.2 : Une requête HTTP sans jeton valide, ou venant d'une origine non autorisée, est refusée.
- CA-24.3 : Le contexte renvoyé par `a2t_tache_suivante` est borné et masqué ; aucun secret ni identifiant n'y figure.
- CA-24.4 : Une sortie soumise par `a2t_soumettre` est validée ; en cas d'écart, la réponse liste les écarts.
- CA-24.5 : Le kit fournit la configuration Gemini (`settings.json`) et un `GEMINI.md` qui interdit à Gemini de trancher un point de contrôle à la place de l'opérateur.

#### US-25 — Choisir le moteur

En tant que DSI, je veux choisir le moteur — Gemini, modèle local ou simulation — par configuration, afin de ne dépendre d'aucun fournisseur.

- CA-25.1 : Passer d'un moteur à l'autre ne demande aucune modification de code.
- CA-25.2 : Chaque appel au moteur est journalisé avec fiche et version composée, moteur, empreinte du prompt, fragments injectés et usage.
- CA-25.3 : Avec le profil souverain, une campagne complète aboutit sans autre connexion sortante que vers l'application cible et son fournisseur SSO.

#### US-26 — Disposer d'un tableau de bord par rôle

En tant que testeur automaticien, développeur ou membre de la DSI, je veux une vue adaptée à mon rôle, afin d'y trouver ce qui me concerne.

- CA-26.1 : Les vues testeur, développeur et gouvernance de l'architecture (§4.5) sont disponibles ; les droits de décision suivent la répartition des rôles (H11).
- CA-26.2 : Les mises à jour s'affichent en direct, sans rechargement de page.
- CA-26.3 : Le tableau de bord n'écoute que sur `127.0.0.1`.
- CA-26.4 : Un audit RGAA 4.1 de l'IHM ne relève aucune non-conformité de niveau A ou AA.
- CA-26.5 : L'IHM affiche la notice d'utilisation et les limites du système.
- CA-26.6 : Les textes de l'IHM, les messages et le journal de tests sont en français (H17).

#### US-27 — Isoler les données par agent

En tant que DSI, je veux que chaque agent n'écrive que dans son propre schéma, afin de borner l'effet d'une erreur ou d'une compromission.

- CA-27.1 : Une base PostgreSQL en conteneur (image pgvector) porte un schéma et un rôle par agent propriétaire (H5).
- CA-27.2 : Une écriture d'un agent dans le schéma d'un autre est refusée par la base.
- CA-27.3 : Seuls des contrats nommés s'exécutent ; une requête SQL libre est refusée (I13).
- CA-27.4 : La base de l'application n'est jamais accédée, sauf par l'oracle en lecture seule, activé explicitement.

#### US-28 — Installer et lancer le kit

En tant que testeur automaticien, je veux installer et lancer le kit sur mon poste avec les scripts fournis, afin de démarrer sans assistance.

- CA-28.1 : `docker compose up` démarre PostgreSQL sur le port 5433, et Ollama avec le profil souverain (H6).
- CA-28.2 : Un script bash et un script `.cmd` en ASCII pur installent et lancent le processus Node.
- CA-28.3 : Le kit fonctionne sous Node 22 et 24 ; sous une version antérieure, il refuse de démarrer avec un message explicite.
- CA-28.4 : Le kit n'a que deux dépendances d'exécution, `playwright` et `pg`, épinglées par un fichier de verrouillage ; toutes les dépendances et images sont sous licence open source.
- CA-28.5 : Le profil conteneur exécute une campagne sans tête avec une session importée.

#### US-29 — Rendre compte de la conformité (AI Act)

En tant que DSI, je veux disposer des éléments de conformité du système, afin de l'inscrire au registre des systèmes d'IA.

- CA-29.1 : Une fiche de registre exportable décrit la finalité, les moteurs, les données transmises, les responsables et le classement proposé (H12).
- CA-29.2 : Le volume et la nature des données transmises au moteur sont mesurés par campagne.
- CA-29.3 : Les preuves sont purgées au-delà de la durée de conservation (H16) ; le journal garde leurs empreintes.

### Recette

#### US-30 — Recevoir le kit de test (Definition of Done)

En tant que DSI, je veux constater sur le poste cible que le kit fonctionne de bout en bout, afin de prononcer sa recette.

- CA-30.1 : Sur l'application cible, une campagne va de la demande au journal de tests sans intervention hors des points de contrôle.
- CA-30.2 : Le rejeu d'un parcours ratifié, application inchangée, donne le même verdict.
- CA-30.3 : En simulation, chaque défaut injecté reçoit la cause attendue (CA-15.6).
- CA-30.4 : Une altération du journal faite en base est détectée (CA-17.2).
- CA-30.5 : Les tests automatisés d'Agent2Test passent.
- CA-30.6 : Le guide de démarrage suffit, seul, à installer et lancer le kit sur un poste Windows 11 + WSL2.

## 9. Données

| Donnée | Origine | Lieu | Conservation | Sensibilité |
|---|---|---|---|---|
| Plan : US, critères, scénarios | projet local, Agent2Dev | schéma `plan` | durée du système, versionné | interne |
| Carte applicative | sources, IHM en marche | schéma `scan` | versionnée à chaque scan | reflète le code : interne |
| Sélections, parcours | `stratege`, `preparateur` | schémas `strategie`, `preparation` | durée du système, versionnés | interne |
| Preuves | `navigateur` | fichiers, empreintes en base | 30 jours (H16) | peuvent contenir des données personnelles : masquage |
| Session SSO | opérateur | fichier local en mode 0600 | jusqu'à expiration | secret (I11) |
| Journal, grand livre | tous les agents | schémas `journal`, `referentiel` | durée du système | opposable |
| Contextes envoyés au moteur | agents cognitifs | empreintes et fragments au journal | durée du système | selon l'autorisation du moteur (T1) |

## 10. Interfaces externes

| Système | Sens | Moyen | US |
|---|---|---|---|
| Application cible | sortant | navigateur Playwright, HTTP(S) | US-5, US-13 |
| Fournisseur SSO | sortant | redirections dans le navigateur | US-6 |
| Sources du projet (Angular, Spring) | lecture | système de fichiers, git | US-3, US-4 |
| Agent2Dev | entrant | fichiers `analyse` et `cahier` | US-1, US-2 |
| Gemini Code Assist | entrant et sortant | MCP : HTTP local, stdio | US-24 |
| Moteur local (Ollama, vLLM) | sortant | HTTP compatible OpenAI | US-25 |
| PostgreSQL (conteneur) | sortant | protocole PostgreSQL, un rôle par agent | US-27 |
| GitLab CI | sortant | fichier JUnit XML | US-18 |

## 11. Contraintes

- **C1** — Playwright piloté comme une bibliothèque depuis Node ; ni `.spec.ts`, ni exécuteur `@playwright/test`.
- **C2** — Node 22 au minimum, 24 recommandé ; deux dépendances d'exécution (H8).
- **C3** — PostgreSQL sous Docker, un schéma par agent (précision 5).
- **C4** — Gemini Code Assist par MCP ; tout autre moteur par le même port (précision 4).
- **C5** — Application déjà lancée et connectée, authentification SSO (précisions 2 et 3).
- **C6** — Exécution locale ; environnements de développement et de recette.
- **C7** — Composants open source ; seul le moteur externe fait exception, et il reste remplaçable.
- **C8** — Axiome 7E à toutes les échelles ; cybernétique de second ordre, premier et second ordres séparés.
- **C9** — Une responsabilité, un seul lieu (architecture §3).
- **C10** — AI Act : classement à confirmer (H12) ; traçabilité et contrôle humain en tout état de cause.
- **C11** — Hors périmètre : tout ce qui ne concerne pas les tests (cadrage §2).

## 12. Risques

| # | Risque | Effet | Parade | Réf. |
|---|---|---|---|---|
| R1 | Le SSO impose une MFA à chaque session, ou repose sur Kerberos | CP-0 fréquents, ou navigateur à faire tourner côté Windows | stratégies `profil` et `cdp` ; topologie à confirmer | H3, H6, US-6 |
| R2 | Composants Indigo sans rôles ARIA standard | cibles fragiles, recours au CSS | relevé ARIA du rendu réel ; fragilités signalées aux développeurs ; cibles par rôle et nom | US-3, US-5, US-10 |
| R3 | Données de recette partagées et mouvantes | faux échecs | préfixe de campagne, nettoyage en option, une campagne à la fois | H7, US-13 |
| R4 | Envoi des sources à Gemini non autorisé | moteur local, moins performant | port moteur, leçons par moteur, sélection déterministe en repli | T1, US-25, CA-8.7 |
| R5 | Scénarios en texte libre ambigus | parcours faux ou incomplets | exécution probatoire, CP-2, leçons | US-10, US-11 |
| R6 | Traitements asynchrones (Kafka) et rendu Angular différé | assertions trop précoces | attentes par état dans un délai borné, jamais de durée fixe | CA-13.2 |
| R7 | Serveur MCP bloqué à « Connecting » dans VS Code | Gemini inutilisable depuis l'IDE | transport HTTP par défaut, Gemini CLI en repli | US-24 |
| R8 | Dérive d'apprentissage (Goodhart) | réussite gonflée, sélection appauvrie | I5, I15, garde anti-Goodhart | US-20 |
| R9 | Volume des preuves | disque saturé | conservation bornée, purge | H16, CA-29.3 |
| R10 | Même moteur pour écrire le code et ses tests | angles morts partagés | oracles tirés des critères, CP-2, moteur distinct recommandé | T6 |

## 13. À ratifier

- les US-1 à US-30 et leurs critères ;
- les hypothèses H14 à H19 ;
- la convention d'entrée de l'annexe A, qui tranche H1 si vous l'adoptez ;
- les objectifs mesurables (§7).

## 14. Glossaire

| Terme | Définition |
|---|---|
| Campagne | exécution, pour une demande, de la sélection ratifiée |
| Carte applicative | écrans, routes, éléments et points d'API relevés par le `scanner` |
| Cause présumée | classement d'un échec parmi six causes fermées, confirmé ou contesté en CP-3 |
| Constitution | fichier des invariants I1 à I15, que le système ne peut pas modifier |
| Critère d'acceptation | énoncé observable et binaire rattaché à une US, codé `CA-n.m` |
| Échappement | défaut trouvé après coup par un test écarté, ou déclaré par un humain |
| Épinglage | gel, pour une campagne, des versions de fiches et de règles |
| Fiche | prompt versionné d'un agent cognitif : rôle, consignes, schéma, contrôles |
| Grand livre | mémoire réflexive, chaînée par empreintes, des évolutions de règles |
| Leçon | consigne tirée d'une décision humaine, active après ratification |
| Moteur | LLM relié par le port moteur : Gemini, modèle local ou simulation |
| O1, O2 | premier ordre (observabilité), second ordre (réflexivité) |
| Palier | rang de sélection : P1 confirmation, P2 non-régression ciblée, P3 socle |
| Parcours | traduction exécutable d'un scénario, sous forme de donnée JSON |
| Point de contrôle | arrêt pour décision humaine, CP-0 à CP-4 |
| Preuve | capture, instantané ARIA, trace ou relevé réseau attaché à un verdict |
| Scénario | suite d'étapes en texte libre, qui couvre un ou plusieurs critères |
| Trou de couverture | critère qu'aucun scénario ne couvre |
| 7E | Éléments, Espace, Engendrent, État, Expression, Évolutif, Environnement |

---

## Annexe A — Convention d'entrée du plan de tests

Proposition pour trancher H1. Le présent document la suit.

### A.1 Identifiants

- US `US-<n>`, critère `CA-<n>.<m>` (n : numéro de l'US), scénario `S-<n>.<k>`. C'est la forme de la sortie `cahier` d'Agent2Dev.
- Identifiant canonique : `<espace>/<code>`. L'espace est déclaré en tête de la source. À défaut, c'est le nom du fichier sans extension, ou l'identifiant de la demande pour une sortie d'Agent2Dev.
- Une référence sans espace désigne l'espace de la source courante.

### A.2 Fichier Markdown

```markdown
---
espace: contrats
---

# Plan de tests — Contrats

## US-12 — Créer un contrat

En tant que gestionnaire de contrats, je veux créer un contrat, afin de le rattacher à un portefeuille.

- CA-12.1 : L'enregistrement affiche le message « Contrat créé ».
- CA-12.2 : Le contrat créé figure dans la liste des contrats.

### S-12.1 — Création nominale

Couvre : CA-12.1, CA-12.2
Criticité : haute
Socle : non

1. Ouvrir la liste des contrats.
2. Cliquer sur « Nouveau contrat ».
3. Saisir un libellé.
4. Enregistrer.
5. Constater le message et la présence du contrat dans la liste.
```

Règles de lecture :

- Un titre de n'importe quel niveau qui commence par `US-<n> —` ouvre une US ; le paragraphe qui commence par « En tant que » ou « En tant qu' » la décrit.
- Une ligne de liste qui commence par `CA-<n>.<m> :` est un critère de l'US courante.
- Un titre qui commence par `S-<n>.<k> —` ouvre un scénario.
  - La ligne `Couvre :` est obligatoire ; elle accepte des références qualifiées, par exemple `DEM-042/CA-1.1`.
  - Les lignes `Criticité :` (haute, moyenne ou basse ; moyenne par défaut), `Socle :` (oui ou non ; non par défaut) et `Prérequis :` sont facultatives.
- La liste numérotée qui suit donne les étapes, en texte libre.
- L'import ignore le contenu des blocs de code, comme l'exemple ci-dessus, et toute autre ligne.

### A.3 Sortie d'Agent2Dev

L'espace est l'identifiant de la demande.

| Champ d'Agent2Dev | Élément canonique |
|---|---|
| `cahier.us[]` : `code`, `titre`, `enTantQue`, `jeVeux`, `afinDe` | US |
| `cahier.us[].criteres[]` : `code`, `texte` | critère |
| `cahier.hypotheses`, `cahier.decisions` | contexte de la demande, versé en mémoire |
| `analyse.fichiersImpactes[].chemin` | changements de la demande, entrée d'`impact` |
| `analyse.risques` | contexte de la sélection |

## Annexe B — Traçabilité des exigences

| US | Modules | Décisions, invariants | Points de contrôle |
|---|---|---|---|
| US-1 | `plan` | H1 | — |
| US-2 | `plan`, `orchestrateur` | I2 | — |
| US-3 | `scanner` | D15, I3 | — |
| US-4 | `scanner` | D15 | — |
| US-5 | `scanner`, `navigateur` | D15, I12 | — |
| US-6 | `navigateur` | D7, I11 | CP-0 |
| US-7 | `memoire` | D5, I12 | — |
| US-8 | `stratege` | D8, I15 | CP-1 |
| US-9 | `preparateur`, `scanner` | D7, I12 | CP-2 |
| US-10 | `preparateur` | D1, I4, I13, I14 | CP-2 |
| US-11 | `preparateur`, `navigateur` | D2 | CP-2 |
| US-12 | `preparateur`, `navigateur` | D2 | CP-2, CP-3 |
| US-13 | `orchestrateur`, `navigateur` | D2, D20, I12, I13 | CP-0 |
| US-14 | `orchestrateur` | I8, I10 | CP-0 à CP-4 |
| US-15 | `verdict` | I5, I6 | CP-3 |
| US-16 | `observateur`, `journal` | D9, I1 | — |
| US-17 | `journal`, `donnees` | D16, I7 | — |
| US-18 | `journal` | I14 | — |
| US-19 | `apprenant` | D10, I9 | CP-4 |
| US-20 | `reflexif` | D9, D10, I5, I15 | CP-4 |
| US-21 | `reflexif`, `referentiel` | D10, D18, I9, I10 | CP-4 |
| US-22 | `referentiel`, port moteur | D17, D18 | — |
| US-23 | `referentiel`, noyau | D11, I1 à I15 | — |
| US-24 | `mcp` | D12, D13, I10 | CP-0 à CP-3 |
| US-25 | port moteur | D12 | — |
| US-26 | `tableau-de-bord` | — | CP-0 à CP-4 |
| US-27 | `donnees` | D6, I13 | — |
| US-28 | déploiement | D14, D19 | — |
| US-29 | `journal`, `tableau-de-bord` | I14 | — |
| US-30 | tous | — | — |
