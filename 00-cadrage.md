# Agent2Test — Cadrage

> Étape 1 sur 7 · proposition soumise à ratification · 25 septembre 2026
> Document compagnon : [`01-architecture.md`](01-architecture.md)

## 1. Objet

Agent2Test choisit, prépare, exécute et journalise les tests essentiels d'une application Angular pour une demande donnée. Il part du plan de tests existant : US, critères d'acceptation numérotés et scénarios.

Playwright est piloté comme une bibliothèque depuis Node. Il n'y a aucun fichier `.spec.ts` et aucun code de test n'est généré : un parcours est une donnée JSON, exécutée par un interpréteur au jeu d'actions fermé. Le système reprend la lignée du banc check-lists et d'Agent2Dev — un bus, des agents qui ne se connaissent que par leur nom, un journal, un observateur, un apprenant — et il est cybernétique de second ordre.

La chaîne de traçabilité suit la règle de la mission, sans Gherkin intermédiaire :

`demande → US → critères numérotés → scénarios → parcours → exécutions → verdicts → journal de tests`

## 2. Périmètre

| Dans le périmètre | Hors périmètre |
|---|---|
| Ingestion de la demande, des US, des critères et des scénarios existants | Rédaction ou correction des US et des critères |
| Cartographie de l'application locale : sources et IHM en marche | Modification du code de l'application (rôle d'Agent2Dev) |
| Sélection argumentée des tests essentiels | Démarrage, arrêt ou déploiement de l'application |
| Mise en place des parcours Playwright et de la session SSO | Détention d'identifiants SSO |
| Exécution, preuves, verdicts, journal de tests | Supervision de production |
| Observabilité et réflexivité d'Agent2Test lui-même | Tout sujet étranger aux tests |

## 3. Ce que fixent les six précisions

| # | Précision | Conséquence dans l'architecture |
|---|---|---|
| 1 | Les US et les scénarios existent | Agent2Test sélectionne et rend exécutable ; il ne rédige rien. Un critère sans scénario est signalé comme trou de couverture, jamais comblé d'office. |
| 2 | L'application est lancée et connectée | Pas d'agent de cycle de vie, contrairement au banc : une sonde, et le refus de lancer une campagne si l'application ne répond pas. |
| 3 | Authentification SSO | Le système ne détient aucun identifiant. L'humain se connecte (point de contrôle CP-0) ; le système conserve la session et détecte son expiration. |
| 4 | Gemini Code Assist via MCP | Montage d'Agent2Dev : Agent2Test est serveur MCP, Gemini rédige, Agent2Test valide. Un port moteur garde le choix du LLM ouvert. |
| 5 | Écriture dans PostgreSQL sous Docker, par agent | Une base, un schéma et un rôle PostgreSQL par agent ; l'agent `donnees` est le seul accès. |
| 6 | Axiome 7E | Sept phases, une seule classe de cycle, récursive, présente à chaque échelle et dans chaque ligne du journal. |

## 4. Hypothèses

Chaque information manquante est remplacée par une hypothèse, avec ce qui changerait si elle était fausse.

| # | Hypothèse | Si elle est fausse |
|---|---|---|
| H1 | Les US et les scénarios sont des fichiers Markdown ou JSON du projet local (dossier configurable), ou la sortie `resultat.json` d'Agent2Dev. Les scénarios sont rédigés en étapes de texte libre. | Seul l'adaptateur d'entrée du module `plan` change ; le modèle canonique reste. Un export Squash TM, Xray ou Excel demande un adaptateur de plus. |
| H2 | « Lancée et connectée » : le front Angular et son API tournent et répondent sur une URL locale. Agent2Test ne les démarre ni ne les arrête ; il les sonde. | Il faudrait réintroduire l'agent `application` du banc. |
| H3 | Le SSO est OIDC ou SAML, avec MFA possible. L'humain se connecte dans une fenêtre ouverte par le système ; la session vit dans un profil de navigateur dédié jusqu'à expiration. Variante : rattachement à un Chrome déjà connecté. | Avec Kerberos ou l'authentification Windows intégrée, le navigateur doit tourner côté Windows, ce qui fixe la topologie (H6). |
| H4 | Gemini Code Assist en mode agent (VS Code, Gemini CLI en repli) est le moteur par défaut, sur le montage d'Agent2Dev. | Le port moteur accepte un autre moteur sans toucher aux agents. |
| H5 | « Par agent » : une instance PostgreSQL (image pgvector) en conteneur, un schéma et un rôle par agent, accès par l'agent `donnees` seul. La base de l'application n'est pas touchée ; un oracle SQL en lecture seule reste possible en option. | Une base par agent multiplierait les instances sans bénéfice. Si les tests doivent écrire dans la base applicative : contrats d'écriture nommés, sur le modèle de `Q-RESTORE` du banc. |
| H6 | Poste Windows 11 + WSL2 Ubuntu 24.04 + Docker. L'application tourne sous Windows (`localhost:4200` et `:8080`). Agent2Test tourne sous WSL2 en réseau `mirrored`, ou directement sous Windows ; PostgreSQL tourne dans Docker Compose. | Seul le profil de déploiement change (étape 6). |
| H7 | Les tests créent des données par l'IHM ; chaque donnée créée porte le préfixe `A2T-<campagne>-`. Pas de restauration globale de la base applicative, puisque l'application tourne. Nettoyage par parcours dédiés, en option. | Une restauration complète imposerait d'arrêter l'application, ce qui contredit H2. |
| H8 | Node 24 LTS recommandé (maintenu jusqu'au 30 avril 2028), Node 22 au minimum (jusqu'au 30 avril 2027) : Node 20, base du banc et d'Agent2Dev, n'est plus maintenu depuis le 30 avril 2026. Deux dépendances : `playwright` (1.63 à date) et `pg`. | — |
| H9 | Tests fonctionnels de bout en bout par l'IHM uniquement. | D'autres familles (performance, accessibilité, sécurité) demanderaient d'autres exécuteurs. |
| H10 | Le livrable « stratégie et plan de tests » couvre deux objets : la stratégie que le kit applique à l'application cible, et le plan de tests d'Agent2Test lui-même. | — |
| H11 | Trois rôles. Le testeur automaticien opère et ratifie CP-1 à CP-3. Le développeur consulte verdicts et preuves, et peut ratifier CP-2. Le référent qualité ratifie CP-4, les changements de règles. La DSI dispose d'une vue de gouvernance en lecture. | Seule la matrice des droits change. |
| H12 | AI Act : Agent2Test est un système d'IA à risque minimal. C'est un outil interne de test logiciel : il n'est pas un composant de sécurité d'une infrastructure critique et ne prend aucune décision sur des personnes. | Un classement à haut risque ajouterait documentation technique et gestion des risques formelles, exigibles à partir du 2 décembre 2027. |
| H13 | Le kit est livré dans un dossier `agent2test/` ou un dépôt dédié. | — |

## 5. Tensions — signalées, non arbitrées

| # | Tension | Exigences en conflit | Proposition | À trancher par |
|---|---|---|---|---|
| T1 | Souveraineté ou Gemini | « Exécution locale / souveraine » contre un moteur hébergé par Google : extraits de sources, carte applicative et US quittent le poste. | Choix réversible par configuration (port moteur) ; moteur local compatible OpenAI (Ollama, vLLM) disponible ; contexte minimisé et masqué. L'envoi des sources de Cockpit CA à Gemini reste à autoriser. | DSI / RSSI |
| T2 | Compose ou SSO | Déploiement Docker Compose contre connexion SSO humaine dans un navigateur visible et accès à `localhost` depuis un conteneur. | Topologie hybride : services à état dans Compose, processus Node sur l'hôte par défaut, profil conteneur sans tête pour la recette, avec session importée. | Vous |
| T3 | Autonomie ou contrôle | Agents semi-autonomes contre contrôle humain (AI Act) et durée des campagnes. | Points de contrôle conditionnels : seul ce qui est nouveau ou modifié est présenté ; CP-4 asynchrone ; niveaux d'autonomie fixés par configuration humaine seulement. | Vous |
| T4 | Apprendre ou comparer | Règles qui évoluent contre verdicts comparables d'une campagne à l'autre. | Référentiel épinglé par campagne ; une règle ratifiée ne vaut qu'à partir de la campagne suivante. | Vous |
| T5 | Essentiel ou exhaustif | Réduire le jeu joué contre risque d'échappement. | Socle toujours joué ; tests écartés listés avec leur motif ; échappements mesurés et réinjectés au second ordre. | Vous et la DSI |
| T6 | Même moteur des deux côtés | Agent2Dev et Agent2Test tous deux sur Gemini : le même modèle peut écrire le code et le test, avec les mêmes angles morts. | Oracles tirés des critères d'acceptation, jamais du code ; parcours ratifiés par un humain ; moteur distinct recommandé quand Agent2Dev a produit le code. | Vous |
| T7 | Sans redondance, mais en lignée | Le banc, Agent2Dev et Agent2Test portent chacun leur bus, leur journal, leur observateur. | Noyau d'Agent2Test conçu comme un paquet extractible, commun à terme ; Agent2Dev n'est pas modifié ici (hors périmètre). | Vous |
| T8 | Multi-LLM ou apprentissage | Une leçon apprise sur Gemini ne vaut pas forcément pour un modèle local. | Leçons et mesures indexées par couple fiche × moteur ; une leçon n'est active que pour les moteurs où elle a été mesurée. | Vous |

## 6. Positionnement AI Act

- **Classement proposé (H12) : risque minimal.** Les obligations propres au haut risque ne s'appliquent pas. Pour l'annexe III, elles sont d'ailleurs reportées au 2 décembre 2027 par le règlement (UE) 2026/1744, dit « omnibus numérique ».
- **Ce qui s'applique** : l'article 4, qui demande de prendre des mesures favorisant la maîtrise de l'IA par les équipes. S'y ajoute l'article 50 (transparence, en vigueur depuis le 2 août 2026), dans la mesure où il vise un outil interne. Les obligations propres aux modèles d'IA à usage général incombent à leur fournisseur, Google pour Gemini.
- **Ce que l'architecture fait en plus, volontairement** : traçabilité, contrôle humain, marquage de l'origine IA, minimisation des données. Correspondance détaillée : `01-architecture.md` §10.
- **Le classement est à confirmer par la conformité.**

## 7. Plan des étapes

| Étape | Livrables | Point de contrôle |
|---|---|---|
| 1 | Cadrage, architecture globale | hypothèses, tensions, décisions, invariants, plan |
| 2 | Cahier des charges : US et critères d'acceptation (`02-cahier-des-charges.md`) ; puis spécifications détaillées par module : contrats de bus, schémas JSON, schémas SQL, outils MCP, langage des parcours, algorithme de sélection, API du cycle 7E | exigences, puis contrats et formats |
| 3 | Backlog modulaire priorisé, roadmap de maturité, stratégie et plan de tests | priorités, paliers, stratégie de test |
| 4 | Prompts système des agents (fiches versionnées), `GEMINI.md` | rôles, consignes, schémas de sortie |
| 5 | Code source exécutable, par lots, chacun jouable en simulation avec ses tests : 5a noyau · 5b données, journal, mémoire · 5c plan, scanner · 5d navigateur, préparateur, verdict, orchestrateur · 5e stratège · 5f observateur, apprenant, réflexif, référentiel · 5g tableau de bord, MCP | un point par lot |
| 6 | Déploiement : Docker Compose, initialisation SQL, scripts bash et `.cmd` en ASCII, configuration Gemini | démarrage sur votre poste |
| 7 | Recette du kit contre la Definition of Done, guide de démarrage | Definition of Done |

## 8. Registre de ratification

| Objet | Où | Statut |
|---|---|---|
| Hypothèses H1 à H13 | §4 | à confirmer |
| Hypothèses H14 à H19, convention d'entrée | `02-cahier-des-charges.md` §5 et annexe A | à confirmer |
| US-1 à US-30 et leurs critères | `02-cahier-des-charges.md` §8 | à ratifier |
| Tensions T1 à T8 | §5 | à arbitrer |
| Décisions D1 à D20 | `01-architecture.md` §12 | à ratifier |
| Invariants I1 à I15, règles modifiables et bornes | `01-architecture.md` §6 | à ratifier |
| Points de contrôle CP-0 à CP-4 | `01-architecture.md` §5 | à ratifier |
| Plan des étapes | §7 | à ratifier |
