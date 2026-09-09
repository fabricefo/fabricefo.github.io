---
layout: post
title: "AIOps : trois chantiers concrets pour passer à l'action"
description: "Une méthode pratique pour fiabiliser les données, choisir un premier cas d'usage et automatiser progressivement une démarche AIOps."
date: 2026-09-09
image: assets/images/aiops-trois-chantiers-concrets.jpg
categories: [AI]
tags: [AIOps, observabilite, automatisation, SRE, operations-IT]
---

Une démarche AIOps peut vite se transformer en accumulation de tableaux de bord, de règles de corrélation et de démonstrations prometteuses. Le projet fonctionne dans un environnement de test, puis se heurte à la réalité : données incomplètes, alertes mal qualifiées, responsabilités floues et automatisations difficiles à approuver.

Pour éviter cet écueil, mieux vaut avancer par étapes. Trois chantiers permettent de construire une approche utile sans chercher à tout automatiser dès le départ.

## 1️⃣ Fiabiliser les données et le contexte opérationnel

L'AIOps dépend directement de ce qu'on lui fournit. Une plateforme sophistiquée ne peut pas compenser durablement des métriques absentes, des journaux non structurés ou des services impossibles à relier à une équipe.

Le premier chantier consiste donc à rendre les signaux opérationnels exploitables.

### Commencer par un service précis

Choisissez un service suffisamment important pour justifier l'effort, mais assez délimité pour rester maîtrisable. Il peut s'agir d'une API utilisée par les clients, d'un parcours de paiement ou d'une application interne critique.

Pour ce périmètre, identifiez :

- les composants techniques et leurs dépendances ;
- l'équipe propriétaire et les personnes à contacter ;
- les métriques, logs et traces réellement disponibles ;
- les changements susceptibles d'expliquer une anomalie ;
- les indicateurs qui représentent l'expérience utilisateur.

Cette cartographie évite qu'une alerte reste isolée de son contexte.

### Normaliser avant de corréler

Les événements doivent partager quelques informations communes : nom du service, environnement, version, région, niveau de criticité et horodatage fiable. Sans cette base, deux outils peuvent décrire le même incident de manière impossible à rapprocher.

OpenTelemetry peut faciliter la collecte cohérente des métriques, logs et traces, mais un standard technique ne dispense pas de définir une convention interne. Les équipes doivent s'accorder sur les attributs utiles et sur leur signification.

### Contrôler la qualité

Avant d'introduire un modèle, mesurez la complétude et la stabilité des données. Quelques contrôles simples suffisent pour commencer :

- proportion de services correctement identifiés ;
- événements sans propriétaire connu ;
- retards ou ruptures dans la collecte ;
- changements de format non documentés ;
- doublons et volumes d'alertes par source.

Le livrable de ce premier chantier n'est pas un nouveau tableau de bord. C'est un jeu de données opérationnelles suffisamment fiable pour soutenir les étapes suivantes.

## 2️⃣ Choisir un cas d'usage étroit et mesurable

« Réduire les incidents grâce à l'IA » est une ambition, pas un cas d'usage. Pour savoir si l'AIOps apporte quelque chose, il faut cibler une difficulté observable et définir une référence avant le déploiement.

### Partir d'une douleur récurrente

Un bon premier cas d'usage présente trois caractéristiques : il revient souvent, mobilise réellement l'équipe et peut être évalué avec des données existantes.

Par exemple :

- regrouper les alertes dupliquées lors d'un même incident ;
- détecter une dérive de latence avant le franchissement d'un seuil fixe ;
- rapprocher une dégradation d'un changement récent ;
- suggérer le runbook adapté à une catégorie d'incidents connue.

À l'inverse, commencer par la prédiction de toutes les pannes ou par l'analyse automatique de l'ensemble du système d'information crée un périmètre trop large pour apprendre rapidement.

### Définir la situation de départ

Avant le pilote, mesurez quelques indicateurs pendant une période représentative : volume d'alertes, proportion de faux positifs, temps de détection, temps de diagnostic, temps de rétablissement et nombre d'escalades.

Ajoutez au moins un indicateur lié au service rendu. Une réduction du nombre d'alertes n'est pas un progrès si des incidents importants passent désormais inaperçus. Un objectif de niveau de service ou un indicateur d'expérience utilisateur permet de conserver cette perspective.

### Tester avec une boucle de retour

Pendant le pilote, les opérateurs doivent pouvoir qualifier les résultats : corrélation utile ou incorrecte, anomalie pertinente ou bruit, recommandation suivie ou rejetée. Ces retours servent à ajuster les règles, les modèles et les données d'entrée.

Le résultat attendu n'est pas une précision parfaite. Il faut déterminer si l'outil améliore le travail de l'équipe de façon suffisamment régulière pour justifier son coût et sa complexité.

## 3️⃣ Automatiser progressivement et sous contrôle

La tentation est forte de passer directement de la détection à la remédiation automatique. Pourtant, une mauvaise action exécutée rapidement reste une mauvaise action. Google SRE rappelle que l'automatisation agit comme un multiplicateur : elle amplifie aussi bien une procédure fiable qu'une décision mal conçue.

### Utiliser une échelle d'autonomie

Faites évoluer chaque cas d'usage par niveaux :

1. **observer** : le système détecte et documente une situation ;
2. **recommander** : il propose un diagnostic ou un runbook ;
3. **assister** : un opérateur valide l'action avant son exécution ;
4. **automatiser** : l'action est déclenchée dans des conditions précises ;
5. **réévaluer** : le résultat est contrôlé et l'automatisation peut être suspendue.

Cette progression permet de recueillir des preuves avant d'augmenter l'autonomie.

### Choisir des actions réversibles

Les premiers candidats doivent être répétitifs, bien compris et faciles à annuler. Relancer un contrôle, collecter des diagnostics ou ajuster temporairement une capacité présente généralement moins de risque qu'une modification permanente de données.

Chaque automatisation devrait inclure :

- des conditions de déclenchement explicites ;
- une limite de fréquence et de périmètre ;
- un journal d'audit ;
- un mécanisme d'arrêt ;
- une procédure de retour arrière ;
- un propriétaire responsable de sa maintenance.

### Prévoir les cas d'échec

Un runbook automatisé doit préciser ce qui se passe si l'action ne produit pas l'effet attendu. Après un redémarrage, le système doit vérifier que le service a réellement retrouvé un état normal. Dans le cas contraire, il doit arrêter la boucle et escalader vers une personne plutôt que répéter indéfiniment la même commande.

Les actions à fort impact doivent conserver une validation humaine, au moins tant que le système n'a pas démontré sa fiabilité sur une période suffisante.

## 🗓️ Une feuille de route simple sur 90 jours

Ces trois chantiers peuvent être organisés sans lancer un programme massif.

### Jours 1 à 30 : cadrer

Sélectionnez un service et un problème récurrent. Cartographiez les dépendances, vérifiez les données disponibles et mesurez la situation initiale. Nommez un responsable technique et un représentant des opérations.

### Jours 31 à 60 : expérimenter

Activez la détection, la corrélation ou la recommandation sur ce périmètre. Laissez les opérateurs qualifier les résultats. Corrigez d'abord les problèmes de données avant de complexifier les modèles.

### Jours 61 à 90 : sécuriser et décider

Automatisez une action à faible risque si les résultats le permettent. Testez les limites, les droits, la journalisation et le retour arrière. Comparez ensuite les indicateurs au point de départ pour décider d'étendre, de corriger ou d'arrêter l'expérimentation.

## 📊 Les indicateurs à suivre

Un tableau de suivi raisonnable peut tenir en quelques mesures :

- volume d'alertes réellement présentées aux opérateurs ;
- proportion de regroupements ou recommandations jugés utiles ;
- temps moyen de détection et de diagnostic ;
- temps moyen de rétablissement ;
- taux de réussite des actions automatisées ;
- nombre de retours arrière et d'escalades ;
- évolution de l'objectif de niveau de service concerné.

Il faut également suivre le coût : stockage, collecte, licences, exploitation de la plateforme et temps consacré à l'entretien des modèles ou des règles.

## 🎯 Chercher un progrès observable, pas une vitrine technologique

Une démarche AIOps réussie commence rarement par un grand déploiement. Elle commence par des données fiables, un problème précis et une automatisation dont le risque est maîtrisé.

En travaillant successivement sur ces trois chantiers, l'organisation apprend ce qui fonctionne dans son propre contexte. Elle peut alors étendre la démarche avec des preuves, plutôt qu'avec des promesses.
