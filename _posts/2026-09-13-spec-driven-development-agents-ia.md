---
layout: post
title: "Spec-Driven Development : pourquoi les agents IA ont besoin de contrats, pas de prompts vagues"
date: 2026-09-13 07:00:00 +0200
categories: [Développement]
tags: [IA, développement, SDD, agents IA, qualité logicielle]
image: assets/images/spec-driven-development-agents-ia.jpg
---

Les assistants de code savent produire vite. Très vite. Mais ils ne savent pas spontanément ce que votre produit doit garantir, quelles règles métier sont intouchables, ni quels compromis d’architecture votre équipe a déjà tranchés.

Le **Spec-Driven Development** (SDD) propose de remplacer la consigne jetable par un contrat explicite, vérifiable et suffisamment durable pour guider le développement.

Un [rapport technique consacré au Spec-Driven Development](https://arxiv.org/html/2602.00180v1) décrit cette évolution comme un déplacement de l’autorité : le code n’est plus nécessairement le seul endroit où vit la vérité du système. La spécification peut devenir le point de départ, l’ancre permanente, voire la source principale des artefacts générés.

<!-- Image: https://commons.wikimedia.org/wiki/File:ComputerProgrammer.jpg | CC0 -->

## 🤖 Le problème n’est pas la génération de code

Demander à un agent IA de « créer une API de gestion des utilisateurs » semble clair. En réalité, cette phrase ouvre une longue liste de décisions implicites :

- Quelles opérations faut-il exposer ?
- Qui peut consulter ou modifier un profil ?
- Comment traiter un email déjà utilisé ?
- Quelles données personnelles peuvent apparaître dans les réponses ?
- Quel niveau de journalisation est attendu ?
- Quelles contraintes de performance, de sécurité ou de conformité s’appliquent ?

En l’absence de réponses, l’agent complète les blancs avec des hypothèses plausibles. Il peut produire un code propre, testé et techniquement cohérent, tout en construisant le mauvais comportement.

Le principal piège du développement assisté par IA tient là : **la qualité apparente de l’implémentation peut masquer une mauvaise compréhension de l’intention**. Un modèle puissant donne simplement plus de moyens à l’agent pour concrétiser ses propres suppositions.

## 📜 Une spécification est un contrat, pas un roman

Le mot « spécification » évoque parfois un document massif, figé avant le début du projet. Ce n’est pas l’objectif du SDD.

Une bonne spécification décrit d’abord **ce que le logiciel doit faire**, sans enfermer prématurément l’équipe dans une solution technique. Elle rend explicites :

- le comportement attendu ;
- les entrées et sorties importantes ;
- les règles métier ;
- les critères d’acceptation ;
- les cas aux limites ;
- les erreurs attendues ;
- les exigences non fonctionnelles réellement déterminantes.

Elle doit être assez précise pour éviter plusieurs interprétations raisonnables, mais pas au point de dicter chaque détail d’implémentation.

Prenons une demande simple : « permettre à un utilisateur de réinitialiser son mot de passe ». Une spécification utile précisera notamment que la réponse publique ne doit pas révéler si l’adresse existe, que le lien expire, qu’un jeton ne peut être utilisé qu’une fois, ainsi que la politique à appliquer aux sessions actives.

Ces éléments ne sont pas des détails de code. Ce sont les propriétés du produit que le code devra respecter.

## 🧭 Du prompt ponctuel à une chaîne d’artefacts vérifiables

Le workflow présenté dans le rapport suit quatre étapes : **Specify → Plan → Implement → Validate**.

Chaque phase produit un artefact qui guide la suivante, sans former une cascade rigide : le plan peut révéler une lacune de la spécification et la validation peut conduire à reprendre les phases précédentes.

1. **Specify** définit le comportement attendu.
2. **Plan** transforme cette intention en choix d’architecture, interfaces et contraintes techniques.
3. **Implement** découpe puis réalise le plan par incréments contrôlables.
4. **Validate** vérifie que le résultat correspond à la spécification.

Cette chaîne est essentielle avec des agents IA. Elle évite de concentrer toute l’intention dans une conversation temporaire que personne ne relira ensuite.

La spécification devient une sorte de **super-prompt**. Conservée dans le dépôt, elle peut être relue par un représentant métier, exploitée par un agent, transformée en tests et utilisée lors de la revue de code.

## 🎯 Pourquoi les agents deviennent meilleurs avec une spécification

### Moins d’ambiguïté

Un agent répond mieux lorsqu’il connaît le résultat attendu et les limites à ne pas franchir. Les critères d’acceptation réduisent l’espace des interprétations possibles.

### Une meilleure décomposition

Une fonctionnalité complexe peut être divisée en comportements indépendants, puis en tâches techniques courtes. Chaque incrément reste compatible avec la fenêtre de contexte de l’agent et peut être vérifié avant le suivant.

### Des revues plus objectives

Sans spécification, la revue se résume souvent à « le code semble-t-il correct ? ». Avec un contrat explicite, la question devient : « chaque comportement attendu est-il couvert et chaque contrainte est-elle respectée ? »

### Une reprise de contexte plus fiable

Les conversations avec un assistant disparaissent, se résument ou changent d’outil. Une spécification conservée dans le dépôt permet à un nouvel agent ou à un nouveau développeur de reprendre le travail sans reconstruire toute l’intention.

### Une validation automatisable

Les exigences testables peuvent alimenter des scénarios BDD, des tests de contrat ou des tests d’acceptation. La conformité ne dépend plus uniquement de la mémoire humaine.

## 🧪 À quoi ressemble une exigence exploitable ?

Une exigence comme « l’application doit être rapide » est inutilisable. Elle n’indique ni le parcours concerné, ni le seuil attendu, ni les conditions de mesure.

Une formulation exploitable serait plutôt :

> Pour 95 % des requêtes de consultation du profil, l’API répond en moins de 300 ms, sous une charge de 100 requêtes par seconde, hors latence réseau du client.

Même principe pour une règle métier :

> Étant donné un jeton de réinitialisation déjà utilisé, lorsqu’un utilisateur tente de l’utiliser à nouveau, le système refuse la demande et invite à générer un nouveau lien.

Le second exemple peut devenir directement un scénario d’acceptation. Il indique une situation initiale, une action et un résultat observable.

## ⚠️ Ce que le SDD ne résout pas automatiquement

Une spécification peut être incomplète, contradictoire ou simplement erronée. L’IA peut aussi respecter fidèlement une mauvaise consigne.

Le SDD ne supprime donc pas le jugement humain. Il le déplace vers les moments où il apporte le plus de valeur :

- clarifier le besoin ;
- arbitrer les contraintes ;
- examiner les risques ;
- valider les comportements ;
- décider si un écart vient du code ou de la spécification.

Il ne faut pas non plus confondre précision et accumulation. Une spécification surchargée de choix techniques devient un plan déguisé. Elle peut empêcher l’agent de proposer une solution plus simple sans améliorer la compréhension du besoin.

## 🛠️ Commencer sans transformer tout le projet

L’adoption peut rester légère. Une première expérience peut se limiter à une fonctionnalité bien délimitée, quelques critères d’acceptation observables et une revue explicite des ambiguïtés avant l’écriture du code.

Cette discipline complète bien les outils comme [GitHub Spec Kit](/vibecoding-avec-github-spec-kit/), mais elle ne dépend pas d’un framework précis. L’important est de conserver le contrat avec le projet et de le confronter au résultat réel. Le troisième épisode détaillera le mode opératoire complet.

## ✅ La vitesse utile vient de l’alignement

Le SDD ne cherche pas à ralentir la génération de code. Il vise à réduire les reprises provoquées par une intention mal comprise.

Avec des agents IA, le goulot d’étranglement se déplace : produire une implémentation devient moins coûteux, tandis que formuler une intention non ambiguë et vérifier le résultat deviennent plus importants.

La spécification n’est donc pas une couche documentaire ajoutée après le travail. Elle est l’interface entre l’intention humaine et l’exécution automatisée. Et plus l’agent gagne en autonomie, plus ce contrat doit être explicite.

Dans le [prochain article](/spec-first-spec-anchored-spec-as-source/), nous comparerons les trois approches proposées par le SDD : **spec-first**, **spec-anchored** et **spec-as-source**.
