---
title: "Google Elephant-Goldfish Model : concevoir avant de coder avec des agents IA"
date: 2026-11-08
image: assets/images/google-elephant-goldfish-model.jpg
description: "Découverte d’Elephant-Goldfish Model, le plugin open source de Google qui sépare contexte long, conception et revue critique par des sous-agents au contexte neuf."
categories: ai
tags: [IA, Développement, Open Source, SDD, Agents IA]
---

Les agents de code savent produire rapidement une fonctionnalité. Ils savent moins bien remettre en cause une hypothèse qu’ils ont eux-mêmes introduite quelques minutes plus tôt. Plus la conversation s’allonge, plus le modèle accumule du contexte, des décisions provisoires et parfois des erreurs qu’il finit par considérer comme acquises.

C’est précisément le problème que cherche à résoudre **Elephant-Goldfish Model**, ou **EGM**. Présenté par Google dans une [annonce publiée sur le Google Open Source Blog](https://opensource.googleblog.com/2026/10/introducing-elephant-goldfish-model-plugin.html), ce projet propose un workflow de développement piloté par la conception : un agent conserve le contexte du système, tandis que d’autres agents repartent d’un contexte neuf pour éprouver la spécification.

L’objectif n’est pas d’ajouter un générateur de code supplémentaire. EGM essaie surtout de rendre la phase située **avant le code** plus explicite, plus critique et plus durable.

<!-- Image : Jackson Kusack via Wikimedia Commons, CC0 -->

## 🐘 L’Elephant conserve la mémoire du système

Dans la métaphore d’EGM, l’**Elephant** est l’agent qui connaît le projet. Il charge le contexte utile : architecture, parcours utilisateurs, conventions, dépendances, historique et cartographie du code.

Il accompagne le développeur pendant la conception. Son rôle consiste à :

- explorer le besoin et ses contraintes ;
- faire émerger les choix d’architecture ;
- comparer les alternatives ;
- documenter les compromis ;
- produire une spécification exploitable ;
- conserver cette connaissance dans des fichiers versionnés.

Cette mémoire est utile, mais elle a un défaut. L’agent qui a participé à la discussion peut rester attaché à ses premières hypothèses. Il connaît les raccourcis du raisonnement et complète mentalement les informations absentes du document.

Autrement dit, l’Elephant peut comprendre une spécification incomplète parce qu’il se souvient de la conversation qui l’a produite.

## 🐟 Les Goldfish oublient pour mieux relire

Les **Goldfish** sont des agents relecteurs lancés dans des contextes conversationnels neufs. Ils ne disposent pas de l’historique de la discussion avec l’Elephant. Ils reçoivent le document, les fichiers explicitement référencés et une grille de revue.

Cet oubli est volontaire. Il permet de poser une question simple : **la spécification se suffit-elle à elle-même ?**

Si un agent sans mémoire préalable ne comprend pas le fonctionnement attendu, un développeur qui découvrira le projet dans six mois risque de rencontrer le même problème. Le contexte neuf aide également à limiter le biais de confirmation : le relecteur ne cherche pas à défendre les décisions prises pendant la conception.

Il ne s’agit toutefois pas d’une isolation absolue. Selon l’environnement, des fichiers d’instructions tels que `AGENTS.md`, `CLAUDE.md` ou `GEMINI.md` peuvent encore être injectés dans le contexte du sous-agent. EGM exige donc un vrai mécanisme de sous-agents pour la revue et doit s’arrêter si l’outil hôte ne sait pas fournir ce contexte neuf.

Le dépôt officiel décrit aujourd’hui une [revue en cinq étapes](https://github.com/google/elephant-goldfish-model/blob/main/docs/designs/architecture.md) :

1. **Zoom out initial** : vérifier la simplicité, la réutilisation de l’existant et le respect de YAGNI.
2. **Test de compréhension** : reformuler l’architecture et détecter les connaissances implicites.
3. **Revue critique** : rechercher les limites, dépendances cachées, risques de montée en charge et modes de panne.
4. **Nouveau zoom out** : réévaluer le document après les corrections apportées.
5. **Test de préparation à l’implémentation** : contrôler les chemins de fichiers, structures de données, signatures et scénarios de test.

L’annonce initiale de Google résumait cette séquence en trois grands contrôles : compréhension, critique antagoniste et préparation à l’implémentation. Le dépôt a depuis formalisé un pipeline plus détaillé autour de ces mêmes intentions.

## 🧭 Un workflow où le code arrive en dernier

Le cœur du cycle repose sur trois skills complémentaires.

### `egm-brainstorm` : challenger l’idée

La première phase explore le besoin sans modifier le code. L’Elephant interroge le développeur, clarifie l’hypothèse de départ et sollicite plusieurs regards critiques. Le résultat est enregistré dans un fichier `concepts_brief.md`.

Cette étape évite de transformer trop vite une demande encore ambiguë en solution technique.

### `egm-draft` : produire une spécification

L’Elephant transforme ensuite le brief en document de conception Markdown. Le document décrit notamment :

- le problème à résoudre ;
- le comportement attendu ;
- l’architecture proposée ;
- les alternatives écartées ;
- les risques et contraintes ;
- le plan détaillé d’implémentation.

Le livrable devient un véritable artefact du dépôt, généralement sous `docs/designs/`, et non un long message perdu dans l’historique d’un chat.

### `egm-review` : exposer les zones faibles

La revue Goldfish soumet le document à plusieurs agents spécialisés. Les constats sont enregistrés dans des rapports séparés, puis consolidés. Le développeur choisit ensuite de corriger le document, de contester une remarque ou d’accepter explicitement un compromis.

L’implémentation ne commence qu’après cette validation.

Ce découpage matérialise une frontière utile : **les premières phases écrivent des documents et des rapports, pas du code de production**. Le [document d’architecture du projet](https://github.com/google/elephant-goldfish-model/blob/main/docs/designs/architecture.md) précise toutefois qu’il s’agit d’une convention du workflow, pas d’une barrière technique. Pour un usage sensible, il faut donc aussi imposer cette séparation avec les permissions de l’environnement d’exécution.

### Quatre skills complètent le cycle

Le dépôt officiel en contient aujourd’hui sept au total. Aux trois skills précédents s’ajoutent :

- `egm-resolve`, pour trier les constats et appliquer uniquement les corrections approuvées ;
- `egm-review-loop`, pour configurer une boucle de reviewers adaptée à un document ou à une tâche ;
- `egm-bootstrap`, pour construire la documentation d’un projet existant ;
- `egm-drift-report`, pour signaler les écarts entre les documents versionnés et le code.

La syntaxe dépend de l’outil hôte. Antigravity CLI et Claude Code utilisent par exemple `/egm:egm-review`, tandis que Codex expose les commandes EGM dans son sélecteur de skills. Les formes courtes utilisées dans cet article désignent les skills, pas une syntaxe universelle.

## 🗂️ Bootstrapper la documentation d’un projet existant

EGM ne se limite pas aux nouveaux projets. La commande `egm-bootstrap` parcourt un codebase existant et construit une documentation hiérarchique à partir des répertoires les plus profonds.

Le principe est ascendant :

1. documenter les composants de bas niveau ;
2. synthétiser leur rôle au niveau du répertoire parent ;
3. remonter progressivement jusqu’à la vue globale du système ;
4. effectuer une première vérification des README produits.

Ce mécanisme peut fournir à l’Elephant une carte initiale du dépôt. Il ne faut cependant pas confondre cette carte avec l’intention métier originale. Le code révèle ce qui existe aujourd’hui, avec ses contournements et sa dette technique. Il ne permet pas de reconstruire avec certitude les décisions que les auteurs avaient en tête.

La documentation générée doit donc passer par une **revue humaine** avant de devenir une référence. Le [guide de démarrage officiel](https://github.com/google/elephant-goldfish-model/blob/main/docs/getting_started.md) insiste lui-même sur cette validation.

## 🔄 Détecter la dérive entre les documents et le code

Une spécification pertinente le jour de sa fusion peut devenir fausse trois mois plus tard. EGM fournit pour cela `egm-drift-report`, un workflow qui compare les documents versionnés avec l’état courant du code.

Il recherche notamment :

- une fonctionnalité documentée mais absente ;
- un modèle incomplet par rapport à la spécification ;
- une logique simplifiée ou codée en dur ;
- du code qui n’est couvert par aucun document ;
- un chemin, une structure ou une interface qui a changé.

Les divergences sont regroupées dans un rapport daté. Une personne doit ensuite décider quelle partie représente la réalité souhaitée : parfois le code doit être corrigé, parfois la documentation doit être mise à jour.

Le rapport de dérive n’est donc ni une preuve formelle ni un mécanisme d’auto-correction. C’est un outil de maintenance qui rend les écarts visibles avant qu’ils ne deviennent la nouvelle vérité implicite du projet.

## 🧱 Des artefacts plutôt qu’un chat interminable

Un autre choix intéressant d’EGM est son approche **artifact-first**. Les briefs, spécifications et rapports sont écrits dans des fichiers Markdown. Le chat sert principalement à prendre des décisions et à demander des validations.

Cette organisation apporte plusieurs bénéfices :

- les documents peuvent être relus dans une pull request ;
- les décisions restent proches du code ;
- l’historique Git apporte une traçabilité ;
- un nouvel agent peut repartir d’artefacts approuvés ;
- les longues sorties ne polluent pas le contexte principal ;
- les revues peuvent être répétées après une évolution majeure.

Le document devient une interface entre les humains, les agents de conception et les agents d’implémentation.

## 🔐 Ce qu’EGM sécurise — et ce qu’il ne sécurise pas

Le modèle réduit plusieurs risques cognitifs : contexte pollué, hypothèses implicites, validation complaisante et implémentation prématurée. Il ne remplace cependant pas les contrôles classiques du développement logiciel.

Une revue Goldfish ne remplace pas :

- les tests unitaires, d’intégration et de sécurité ;
- la revue de code humaine ;
- la protection des branches ;
- l’analyse des dépendances ;
- la gestion des secrets ;
- les politiques d’accès au dépôt ;
- l’observabilité en production.

Le [README officiel](https://github.com/google/elephant-goldfish-model) indique d’ailleurs explicitement que la revue ne remplace ni les tests ni le jugement humain. Le projet est distribué sous licence Apache 2.0 et n’est pas présenté comme un produit Google officiellement supporté.

## 🛠️ Quand utiliser Elephant-Goldfish Model ?

EGM me paraît particulièrement adapté aux changements qui nécessitent de vraies décisions : nouvelle capacité transverse, évolution d’API, migration de données, refonte d’un flux ou modification d’architecture.

Pour une correction locale et évidente, le workflow complet peut être disproportionné. Le coût de plusieurs revues n’a de sens que si le risque d’une mauvaise hypothèse dépasse celui du temps consacré à la conception.

Le point le plus utile n’est finalement pas la métaphore animalière. C’est la séparation des responsabilités :

- un agent riche en contexte aide à concevoir ;
- des agents sans historique cherchent ce qui manque ;
- des fichiers versionnés portent la mémoire du système ;
- un humain arbitre les compromis et autorise le passage au code.

Cette séparation ne rend pas une conception automatiquement juste. Elle la rend **plus difficile à accepter sans l’avoir réellement comprise**.
