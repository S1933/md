---
type: documentation
source: https://github.com/mattpocock/skills
lang: fr
_width: wide
---

# Documentation des Skills Matt Pocock

Référence complète des skills du repository [mattpocock/skills](https://github.com/mattpocock/skills) — traduite en français.

## Architecture

Les skills sont réparties en **catégories** et en **deux modes d'invocation** :

- **User-invoked** : lancées par l'utilisateur en tapant `/nom-du-skill`
- **Model-invoked** : invocables automatiquement par l'agent quand le contexte correspond

---

## Engineering (User-invoked)

### `/ask-matt`

**Routeur** — demande quel skill utiliser selon ta situation.

- **Flow principal** : idée → grill-with-docs → (prototype) → to-prd → to-issues → implement
- **On-ramps** : `/triage` pour bugs/requêtes
- **Codebase health** : `/improve-codebase-architecture`
- **Crossing sessions** : `/handoff` (nouveau contexte), `/compact` (même contexte)
- **Standalone** : `/grill-me`, `/teach`, `/writing-great-skills`
- **Prérequis** : `/setup-matt-pocock-skills`

### `/grill-with-docs`

Interview d'approfondissement d'un plan ou design. Version avec docs : crée/modifie `CONTEXT.md` et les ADRs pendant la session en utilisant `/domain-modeling`.

Délègue à `/grilling` + `/domain-modeling`.

### `/triage`

Machine à états pour trier les issues sur le tracker :

1. **Catégories** : `bug` | `enhancement`
2. **États** : `needs-triage` → `needs-info` | `ready-for-agent` | `ready-for-human` | `wontfix`

- Affiche les issues non étiquetées, `needs-triage`, `needs-info`
- Triage d'une issue spécifique : contexte → recommandation → reproduction (bugs) → grill → application
- Utilise 5 labels canoniques (configurables via `/setup-matt-pocock-skills`)

### `/improve-codebase-architecture`

Scanne la codebase pour des opportunités d'approfondissement, génère un rapport HTML, puis grille l'utilisateur sur le candidat choisi.

Processus :
1. **Explorer** — lire `CONTEXT.md` + ADRs, utiliser l'agent Explore
2. **Rapport HTML** — visualisations before/after avec Mermaid, cartes par candidat, badge de recommandation
3. **Grilling** — parcourir l'arbre de design avec `/grilling` + `/domain-modeling`

Utilise le vocabulaire de `/codebase-design` (module, interface, depth, seam, adapter, leverage, locality).

### `/setup-matt-pocock-skills`

Configuration initiale du repo pour les engineering skills :

1. **Issue tracker** — GitHub | GitLab | Local markdown | Autre
2. **Triage labels** — mapping des 5 labels canoniques
3. **Domain docs** — single-context ou multi-context (`CONTEXT-MAP.md`)
4. Écrit dans `CLAUDE.md` ou `AGENTS.md` + `docs/agents/*.md`

### `/to-issues`

Découpe un plan/PRD en issues indépendantes (vertical slices / tracer bullets).

- Travaille avec l'issue tracker configuré
- Présente le découpage → quiz utilisateur → publication avec template
- Chaque issue : titre, bloqué par, user stories, critères d'acceptation

### `/to-prd`

Synthétise la conversation en PRD et publie sur l'issue tracker.

- Pas d'interview — synthèse uniquement
- Explore la codebase + vocabulaire du domaine
- Template : Problem Statement, Solution, User Stories, Implementation Decisions, Testing Decisions, Out of Scope

### `/prototype`

Crée un prototype jetable pour répondre à une question.

Deux branches :
- **LOGIC** — appli terminal interactive pour tester un modèle d'état/logique
- **UI** — variations UI radicales sur une route, switchables par paramètre URL

Règles : jetable dès jour 1, une commande pour lancer, pas de persistance, pas de polish, état visible, supprimer ou absorber à la fin.

### `/implement`

Implémente le travail décrit dans un PRD ou des issues.

- Utilise `/tdd` aux seams convenus
- Typecheck régulier, tests unitaires, suite complète à la fin
- `/review` une fois terminé
- Commit sur la branche courante

### `/resolving-merge-conflicts`

Résout les conflits git (merge/rebase).

1. Voir l'état actuel
2. Trouver les sources primaires pour chaque conflit
3. Résoudre chaque hunk — préserver les deux intentions
4. Lancer les vérifications automatisées
5. Finaliser le merge/rebase

---

## Engineering (Model-invoked)

### `/diagnosing-bugs`

Boucle disciplinée pour bugs durs et régressions de performance.

**Phase 1** — Construire une boucle de feedback (la compétence clé)
   - Test échouant, curl, script CLI, headless browser, replay, throwaway harness, fuzz, bisection, differential, HITL bash script
   - La boucle doit être **tight** (rapide, déterministe, red-capable)

**Phase 2** — Reproduire + minimiser
**Phase 3** — 3-5 hypothèses falsifiables, classées, montrées à l'utilisateur
**Phase 4** — Instrumenter (debugger > logs ciblés > jamais "log everything")
**Phase 5** — Fix + test de régression à la bonne seam
**Phase 6** — Cleanup + post-mortem

### `/tdd`

Développement piloté par les tests (red-green-refactor) en vertical slices.

- **Philosophie** : tester le comportement via l'interface publique, pas l'implémentation
- **Anti-pattern** : horizontal slices (tous les tests puis tout le code)
- **Workflow** : Planning → Tracer Bullet → Incrémental (1 test → 1 implémentation) → Refactor
- **Checklist** : behaviour vs implementation, public interface only, survivrait à un refactor, minimal, pas de spéculation

### `/domain-modeling`

Construit et affine le modèle du domaine.

- Challenge des termes contre le glossaire
- Affinage du langage flou
- Scénarios concrets pour stress-test
- Cross-reference avec le code
- Mise à jour inline de `CONTEXT.md`
- ADRs seulement quand : difficile à inverser + surprenant sans contexte + vrai trade-off

### `/codebase-design**

Vocabulaire partagé pour concevoir des **modules profonds** :

- **Module** — interface + implémentation
- **Interface** — tout ce qu'un appelant doit savoir
- **Depth** — quantité de comportement derrière une petite interface
- **Seam** — endroit où l'interface vit
- **Adapter** — chose concrète qui satisfait une interface à une seam
- **Leverage** — ce que les appelants gagnent de la profondeur
- **Locality** — ce que les mainteneurs gagnent

Principes : test de suppression, l'interface est la surface de test, un adaptateur = seam hypothétique/deux = réelle.

---

## Productivity

### User-invoked

#### `/grill-me`

Interview d'approfondissement d'un plan. Délègue à `/grilling`. Version stateless (sans codebase).

#### `/handoff`

Compresse la conversation en document de passation pour un autre agent.

- Sauvegarde dans le répertoire temporaire
- Section "suggested skills"
- Ne duplique pas le contenu des artefacts existants
- Redacte les infos sensibles

#### `/teach`

Enseigne un concept à l'utilisateur sur plusieurs sessions.

- **Workspace** : `MISSION.md`, `RESOURCES.md`, `./reference/*.html`, `./learning-records/*.md`, `./lessons/*.html`, `NOTES.md`
- **Philosophie** : Connaissances → Compétences → Sagesse
- **ZPD** : Zone proximale de développement
- **Leçons** : auto-contenues, belles, rapides, liées à la mission

#### `/writing-great-skills`

Référence pour écrire et éditer des skills de qualité.

- **Prévisibilité** — vertu racine
- **Information hierarchy** — in-skill step > in-skill reference > external reference
- **Leading words** — concepts compacts du pré-entraînement du modèle
- **Failure modes** : premature completion, duplication, sediment, sprawl, no-op

### Model-invoked

#### `/grilling`

Interview sans relâche sur tous les aspects d'un plan.

- Parcourt l'arbre de design une branche à la fois
- Propose une réponse recommandée pour chaque question
- Explore la codebase quand pertinent
- Une question à la fois

---

## Misc

### `/git-guardrails-claude-code`

Installe des hooks Claude Code pour bloquer les commandes git dangereuses.

Bloque : `git push`, `git reset --hard`, `git clean -f`, `git branch -D`, `git checkout .`

1. Demander la portée (projet ou global)
2. Copier le script + `chmod +x`
3. Ajouter le hook dans `settings.json`
4. Personnalisation optionnelle
5. Vérification

### `/migrate-to-shoehorn`

Migre les assertions `as` vers `@total-typescript/shoehorn` dans les tests.

- `as Type` → `fromPartial()`
- `as unknown as Type` → `fromAny()`
- `fromExact()` pour objet forcé complet

Workflow : installer → trouver les fichiers → remplacer → typecheck.

### `/scaffold-exercises`

Crée des structures d'exercices avec sections, problèmes, solutions.

- Sections : `exercises/XX-section-name/`
- Exercices : `XX.YY-exercise-name/`
- Variants : `problem/`, `solution/`, `explainer/`
- Lint : `pnpm ai-hero-cli internal lint`

### `/setup-pre-commit`

Configure Husky + lint-staged + Prettier.

1. Détecter le package manager
2. Installer husky, lint-staged, prettier
3. Initialiser Husky
4. Créer `.husky/pre-commit` (lint-staged → typecheck → test)
5. Créer `.lintstagedrc` + `.prettierrc`
6. Vérifier
7. Commit

---

## Personal

### `/edit-article`

Édite et améliore des articles.

1. Diviser en sections selon les titres. Respecter les dépendances DAG.
2. Pour chaque section : réécrire pour clarté, cohérence, flow. Maximum 240 caractères par paragraphe.

### `/obsidian-vault`

Recherche, création et gestion de notes dans Obsidian.

- Index notes pour agréger les sujets
- Wikilinks `[[Note Title]]`
- Title case pour les noms
- Pas de dossiers — liens et index notes

---

## Déprécié

### `/design-an-interface`

Génère des designs d'interface radicalement différents via sous-agents parallèles, puis compare.

1. Collecter les requirements
2. Lancer 3+ sous-agents avec contraintes différentes
3. Présenter les designs
4. Comparer (simplicité, généralité, efficacité, profondeur)
5. Synthétiser

### `/qa`

Session QA interactive. L'utilisateur décrit des problèmes, l'agent explore la codebase et crée des issues GitHub.

- Écouter, clarifier légèrement
- Explorer la codebase en arrière-plan
- Décomposer en issues si nécessaire
- Créer les issues (template) — sans revue préalable

### `/request-refactor-plan`

Crée un plan de refactor détaillé avec petits commits.

1. Interview détaillée du problème
2. Vérification dans le repo
3. Autres options
4. Scope exact
5. Test coverage
6. Plan en tiny commits
7. Issue GitHub avec template

### `/ubiquitous-language`

Extrait un glossaire de langage ubiquitaire (DDD) de la conversation.

- Scanner la conversation pour concepts
- Identifier problèmes (ambiguïté, synonymes)
- Proposer glossaire canonique
- Écrire `UBIQUITOUS_LANGUAGE.md`
- Dialogue d'exemple
- Règles : opinionated, flag conflits, définitions courtes

---

## En cours

### `/decision-mapping`

Transforme une idée vague en carte de décision avec tickets d'investigation.

- **Bootstrap** : grilling + domain-modeling → écrire la carte (ne pas résoudre)
- **Resume** : charger la carte → résoudre un ticket → mettre à jour
- **Types** : Research | Prototype | Discuss
- **Fog of war** : la carte est délibérément incomplète au-delà de la frontière

### `/review`

Review sur deux axes : Standards (conformité aux conventions) et Spec (conformité au cahier des charges).

- Sous-agents parallèles
- Comparaison HEAD vs point fixe
- Rapport agrégé : axes séparés — pas de re-ranking

### `/writing-beats`

Façonne un article comme un voyage de beats (choix-your-own-adventure).

1. 2-3 beats de départ candidats → l'utilisateur choisit
2. Écrire ce beat uniquement
3. Proposer 2-3 beats suivants
4. Boucler jusqu'à la fin naturelle

### `/writing-fragments`

Grilling qui produit des fragments — pépites d'écriture hétérogènes.

- Capture dès la première phrase
- Fragments séparés par `\n---\n`
- Pas de structure imposée
- Le fichier est édité par l'utilisateur pendant la session

### `/writing-shape`

Prend un fichier markdown brut et le façonne en article via session conversationnelle.

1. Lire la matière première
2. 2-3 ouvertures candidates
3. Croître paragraphe par paragraphe
4. Arguments de format (prose vs liste, callout, tableau, citation)
5. Boucler jusqu'à la fin

---

*Documentation générée à partir du repository [mattpocock/skills](https://github.com/mattpocock/skills) — Skills For Real Engineers par Matt Pocock.*
