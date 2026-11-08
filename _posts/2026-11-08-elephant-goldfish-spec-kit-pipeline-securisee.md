---
title: "Elephant-Goldfish et GitHub Spec Kit : construire une pipeline SDD complète et sécurisée"
date: 2026-11-08
image: assets/images/elephant-goldfish-spec-kit-pipeline.jpg
description: "Une architecture concrète pour combiner Elephant-Goldfish Model et GitHub Spec Kit, du bootstrap documentaire au contrôle de dérive, avec validations humaines et garde-fous de sécurité."
tags: [IA, GitHub, SDD, Sécurité, DevOps]
---

[Elephant-Goldfish Model](https://github.com/google/elephant-goldfish-model) et [GitHub Spec Kit](https://github.com/github/spec-kit) appartiennent tous les deux à la famille du Spec-Driven Development. Ils ne résolvent pourtant pas exactement le même problème.

**Spec Kit** structure le changement à construire : exigences, clarification, plan technique, tâches, implémentation et convergence. **Elephant-Goldfish Model**, ou **EGM**, apporte une mémoire documentaire du système, une conception assistée par un agent riche en contexte et une revue critique par des agents isolés.

Les combiner permet de construire une chaîne plus complète, notamment sur un projet existant. Mais il n’existe pas, à ce jour, de connecteur officiel qui fusionne automatiquement les deux outils. L’intégration repose sur des artefacts Markdown partagés, des conventions de répertoire et des points de contrôle explicites.

L’objectif n’est donc pas d’empiler deux frameworks. Il s’agit de donner à chacun une responsabilité claire.

<!-- Image : Coolingtowerr via Wikimedia Commons, CC0 -->

## 🧩 Répartir les responsabilités

Une pipeline cohérente commence par éviter les doublons.

| Besoin | Outil responsable | Artefact ou résultat |
|---|---|---|
| Cartographier un codebase existant | EGM Bootstrap | README hiérarchiques et contexte initial |
| Valider cette cartographie | Humain | Baseline documentaire approuvée |
| Définir les principes du projet | Spec Kit | Constitution |
| Exprimer le besoin métier | Spec Kit | Spécification de fonctionnalité |
| Lever les ambiguïtés | Spec Kit | Spécification clarifiée |
| Définir la solution technique | Spec Kit | Plan d’implémentation |
| Décomposer le travail | Spec Kit | Liste de tâches |
| Challenger le plan avec un contexte neuf | EGM Goldfish | Rapports de revue |
| Réaliser le changement | Spec Kit et agent de code | Code et tests |
| Vérifier les travaux restants | Spec Kit | Convergence ou nouvelles tâches |
| Détecter la dérive dans la durée | EGM Drift Report | Rapport de divergences |

Cette répartition évite un piège fréquent : demander à EGM et Spec Kit de produire chacun leur propre spécification canonique. Deux sources de vérité finissent presque toujours par diverger.

Dans la chaîne proposée ici, **Spec Kit possède les artefacts du changement**. EGM utilise ces fichiers comme contexte et comme cible de revue, sans recréer une seconde version du besoin.

## 🏗️ La pipeline cible

La chaîne complète peut être représentée ainsi :

```text
EGM Bootstrap
      │
      ▼
Revue humaine de la baseline documentaire
      │
      ▼
Spec Kit : constitution → specify → clarify → plan
      │
      ▼
Revue EGM Goldfish du plan et de son contexte
      │
      ▼
Corrections dans l’artefact qui en est propriétaire
      │
      ▼
Spec Kit : checklist → tasks → analyze
      │
      ▼
Approbation humaine
      │
      ▼
Spec Kit : implement → tests → converge
      │
      ▼
Pull request + contrôles CI + revue humaine
      │
      ▼
EGM Drift Report périodique
```

Chaque flèche représente une transition contrôlée. Une étape ne doit pas modifier silencieusement les artefacts possédés par l’étape précédente.

## 🗺️ Étape 1 — Bootstrapper l’existant avec EGM

Sur un projet brownfield, le premier problème est rarement la rédaction de la prochaine user story. Il faut d’abord comprendre ce qui existe réellement.

La notation abrégée suivante désigne le skill de documentation ascendante d’EGM :

```text
egm-bootstrap <chemin-du-projet>
```

Le workflow parcourt les sous-répertoires, documente les composants locaux puis synthétise progressivement une vue d’ensemble. Le résultat sert de carte au futur Elephant et aux agents de Spec Kit.

Cette génération ne doit pas être exécutée directement sur la branche principale. Une approche plus sûre consiste à :

1. créer une branche ou un worktree temporaire ;
2. limiter l’écriture aux fichiers de documentation autorisés ;
3. examiner le diff produit ;
4. faire relire les README par les propriétaires des composants ;
5. fusionner la baseline documentaire dans une pull request dédiée.

EGM demande également une approbation humaine avant de parcourir plus de quinze répertoires. Cette limite évite de lancer sans contrôle une génération documentaire très large.

Le point critique est la validation humaine. Le code permet d’observer une architecture et des comportements, mais pas de récupérer automatiquement l’intention métier originale. Une anomalie historique peut être décrite avec beaucoup de précision sans devenir pour autant une exigence à préserver.

La baseline issue du bootstrap doit donc être considérée comme **une description candidate de l’existant**, pas comme une spécification métier incontestable.

## 📜 Étape 2 — Installer Spec Kit sans écraser le projet

Pour intégrer Spec Kit dans un dépôt existant, la documentation officielle propose :

```bash
specify init --here --force --integration <key>
```

L’option `--force` autorise l’initialisation dans un répertoire non vide. Elle justifie une précaution simple : exécuter la commande dans une branche dédiée, inspecter le diff et confirmer qu’aucun fichier propre au projet n’a été remplacé de façon indésirable.

Le [guide Spec Kit pour les projets existants](https://github.github.io/spec-kit/how-to/existing-projects.html) recommande ensuite de créer une constitution fondée sur des règles déjà vraies dans le dépôt : compatibilité des API, conventions de tests, exigences de sécurité, architecture existante et critères de qualité.

Il ne faut pas demander à l’agent d’inventer ces règles. Elles doivent être extraites de la baseline, des contrôles CI et des décisions des mainteneurs, puis validées par l’équipe.

## 📝 Étape 3 — Piloter le changement avec Spec Kit

Le chemin complet actuel de Spec Kit enchaîne les opérations suivantes :

```text
constitution → specify → clarify → plan → checklist
→ tasks → analyze → implement → converge
```

Selon l’intégration choisie, les commandes apparaissent sous différentes formes : `/speckit.*`, `/speckit-*`, `$speckit-*` ou `/skill:speckit-*`. Avec Codex, elles prennent notamment la forme `$speckit-specify`. Les skills EGM se sélectionnent dans le sélecteur de skills de Codex. Les noms varient, pas les responsabilités.

### Définir le besoin

`specify` doit décrire **ce qu’il faut obtenir et pourquoi**, sans figer trop tôt la technologie. Sur un projet existant, la demande doit citer les comportements à préserver et les composants concernés.

### Lever les ambiguïtés

`clarify` transforme les zones floues en décisions explicites : compatibilité, permissions, erreurs, limites, migration, reprise et critères d’acceptation.

### Concevoir la solution

`plan` utilise la documentation EGM approuvée comme contexte du codebase. Le plan doit indiquer les composants réutilisés, les interfaces modifiées, les impacts sur les données et la stratégie de test.

### Préparer le travail

`checklist` vérifie la qualité des exigences. `tasks` produit les unités de travail. `analyze` contrôle la cohérence entre la spécification, le plan et les tâches avant que l’implémentation ne rende les corrections plus coûteuses.

Les checklists personnalisées doivent rester des artefacts de revue. Un agent d’implémentation ne doit pas se donner lui-même toutes les validations nécessaires pour déverrouiller son propre travail.

Dans la pipeline proposée ici, `checklist`, `tasks` et `analyze` sont déclenchés après la première revue Goldfish du plan.

## 🐟 Étape 4 — Ajouter une revue Goldfish indépendante

La revue Goldfish principale se place **après `plan` et avant la génération des tâches**. Le plan est assez détaillé pour être éprouvé, mais les tâches n’ont pas encore été dérivées d’une conception qui pourrait être incomplète.

Le plan Spec Kit peut servir de cible principale :

```text
egm-review specs/<numéro>-<feature>/plan.md
```

Cette forme est une notation abrégée : l’invocation réelle dépend de l’outil hôte. Le chemin exact dépend de la structure générée par la version de Spec Kit utilisée. La spécification, la constitution, les ADR et les documents EGM de référence sont fournis comme `GoldfishContext`, ou liés depuis le plan lorsque le runtime sait résoudre ces liens locaux.

Cette revue apporte plusieurs regards :

- un lecteur sans historique vérifie que le plan est compréhensible ;
- un critique recherche les dépendances et scénarios oubliés ;
- un relecteur structurel contrôle la cohérence du document ;
- un contrôleur de préparation vérifie que le plan est assez précis pour guider l’implémentation.

Les corrections doivent être appliquées dans le bon artefact :

- une exigence manquante retourne dans la spécification ;
- une ambiguïté métier retourne dans `clarify` ;
- un défaut de conception retourne dans le plan ;
- après la revue du plan, `checklist`, `tasks` et `analyze` sont exécutés ;
- une tâche absente impose de régénérer ou corriger les tâches ;
- une incohérence globale impose de relancer `analyze`, puis au besoin une revue Goldfish ciblée.

Il faut ensuite relancer la revue concernée. Modifier uniquement le rapport Goldfish reviendrait à masquer le problème sans corriger sa source.

## ✅ Étape 5 — Imposer une décision humaine avant le code

Un rapport Goldfish n’est pas un verdict automatique. Chaque constat peut être :

- accepté et corrigé ;
- contesté avec une justification ;
- transformé en risque explicitement assumé ;
- reporté dans une tâche séparée.

Après `checklist`, `tasks` et `analyze`, la transition vers `implement` doit exiger une approbation humaine identifiable. Dans GitHub, ce contrôle peut être matérialisé par :

- une pull request dédiée aux artefacts de conception ;
- des règles `CODEOWNERS` sur la constitution et les spécifications ;
- une protection de branche ;
- un nombre minimal d’approbations ;
- des conversations obligatoirement résolues ;
- un contrôle CI qui vérifie la présence des artefacts attendus.

L’agent qui a produit le plan ne devrait pas être le seul à autoriser sa mise en œuvre.

## ⚙️ Étape 6 — Implémenter puis converger

Une fois la conception approuvée, Spec Kit peut exécuter les tâches avec `implement`. Pour réduire le rayon d’impact, il est préférable de limiter chaque exécution à une phase ou à une user story plutôt que d’autoriser une modification globale du dépôt.

La CI doit rester l’autorité pour les contrôles reproductibles :

- compilation ;
- tests unitaires et d’intégration ;
- analyse statique ;
- détection de secrets ;
- analyse des dépendances ;
- vérifications d’infrastructure si nécessaire.

La commande `converge` compare ensuite l’implémentation aux artefacts Spec Kit. Si elle détecte un travail manquant, elle ajoute des tâches à `tasks.md`. Le cycle `implement → converge` se répète jusqu’à convergence.

Cette convergence n’est pas équivalente à une validation fonctionnelle complète. Elle vérifie l’alignement avec les artefacts fournis ; elle ne prouve pas que le besoin initial était juste ni que le logiciel est sûr en production.

## 🔄 Étape 7 — Contrôler la dérive après la fusion

La chaîne ne doit pas s’arrêter au merge. EGM propose `egm-drift-report` pour comparer périodiquement les documents de conception et les README avec le code courant.

```text
egm-drift-report <répertoire-des-documents>
```

Le rapport doit rester non destructif. Il signale les différences, puis une personne décide de la correction :

- si le code ne respecte plus une décision toujours valide, le code doit changer ;
- si une évolution approuvée n’a pas été répercutée, le document doit changer ;
- si les deux sont incomplets, une nouvelle spécification Spec Kit doit être ouverte.

Une exécution hebdomadaire peut convenir à un dépôt actif, comme le suggère le [guide EGM](https://github.com/google/elephant-goldfish-model/blob/main/docs/getting_started.md). Le rapport ne doit pas déclencher automatiquement une correction ou un merge.

## 🔐 Les garde-fous indispensables

L’association de deux frameworks d’agents augmente la surface d’action. Une pipeline sécurisée doit donc séparer les étapes de lecture, de documentation et de modification du code.

### Permissions minimales

- Le bootstrap écrit uniquement dans les emplacements documentaires autorisés.
- Les Goldfish lisent les artefacts nécessaires mais n’accèdent pas aux secrets.
- L’agent d’implémentation ne peut écrire que dans le périmètre approuvé du plan.
- La CI utilise des identités distinctes et des jetons à durée courte.
- Aucun agent ne dispose d’un droit direct de fusion sur la branche protégée.

### Environnements isolés

Chaque phase s’exécute dans une branche, un worktree ou un conteneur éphémère. Les dépendances du projet ne doivent pas être installées sur un poste privilégié uniquement pour permettre une revue documentaire.

### Artefacts non fiables par défaut

Un README généré, une spécification et un rapport de revue peuvent contenir une erreur ou une instruction malveillante provenant du dépôt analysé. Il faut traiter ces fichiers comme des entrées non fiables : limiter les outils accessibles, interdire l’exfiltration réseau inutile et valider toute action à effet de bord.

### Traçabilité

Pour chaque exécution importante, conserver :

- la version d’EGM et de Spec Kit ;
- le commit du dépôt analysé ;
- les artefacts d’entrée ;
- les rapports produits ;
- l’identité de l’approbateur ;
- le diff finalement fusionné.

### Maîtrise de la chaîne d’approvisionnement

Les plugins et CLI évoluent. Il faut épingler des versions testées, vérifier les archives ou sommes de contrôle lorsqu’elles sont publiées, examiner les permissions demandées et mettre à jour selon un processus contrôlé. EGM doit être installé comme plugin complet : copier seulement `skills/` omettrait notamment les personas de `agents/` et les standards de `resources/document_standards/`.

## 🚫 Les anti-patterns à éviter

Plusieurs raccourcis annuleraient l’intérêt de cette architecture :

- faire produire puis approuver la spécification par le même contexte d’agent ;
- considérer le bootstrap comme une récupération fiable de l’intention métier ;
- maintenir une spécification EGM et une spécification Spec Kit concurrentes ;
- laisser la revue Goldfish modifier directement le code de production ;
- autoriser `implement` avec des checklists non relues ;
- corriger automatiquement tout drift sans arbitrage ;
- donner au plugin un jeton GitHub capable de pousser et fusionner sur `main` ;
- confondre « aucune divergence détectée » avec « système correct et sécurisé ».

## 🎯 Le résultat : une boucle de spécification durable

La combinaison devient réellement intéressante lorsque chaque outil reste dans son rôle :

1. **EGM documente l’existant et entretient la mémoire technique.**
2. **L’équipe valide ce que cette documentation signifie.**
3. **Spec Kit transforme une intention métier en changement structuré.**
4. **Les Goldfish relisent le plan sans hériter du raisonnement qui l’a produit.**
5. **Un humain autorise le passage à l’implémentation.**
6. **La CI vérifie les propriétés reproductibles.**
7. **Spec Kit contrôle la convergence du changement.**
8. **EGM recherche ensuite la dérive dans le temps.**

Cette pipeline ne supprime ni les erreurs ni les arbitrages. Elle rend en revanche visibles les moments où une hypothèse devient une décision, où une décision autorise du code et où le code commence à s’éloigner de sa documentation.

C’est précisément à ces frontières que les agents IA ont besoin de garde-fous — et que le jugement humain conserve toute sa valeur.
