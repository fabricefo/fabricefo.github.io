---
layout: post
title: "Mettre le Spec-Driven Development en pratique avec des agents IA"
date: 2026-09-13 07:10:00 +0200
categories: [Développement]
tags: [SDD, agents IA, workflow, tests logiciels, ingénierie logicielle]
image: /assets/images/workflow-sdd-agents-ia.jpg
---

Le Spec-Driven Development devient utile lorsqu’il modifie concrètement la façon de livrer une fonctionnalité. Écrire une spécification puis la ranger dans un dossier ne suffit pas : chaque étape doit produire un artefact qui guide la suivante et permet de détecter les écarts.

Le workflow proposé dans ce [rapport technique sur le SDD](https://arxiv.org/html/2602.00180v1) tient en quatre verbes : **Specify, Plan, Implement, Validate**. Il sépare l’intention, les choix techniques, la réalisation et la preuve de conformité. Après avoir vu [pourquoi la spécification devient centrale](/2026/09/13/spec-driven-development-agents-ia/) et [comment choisir son niveau de SDD](/2026/09/13/spec-first-spec-anchored-spec-as-source/), passons à l’exécution.

Voici comment l’appliquer avec une équipe humaine et des agents IA.

<!-- Image: https://commons.wikimedia.org/wiki/File:41B-01-004_-_STS-41B_-_Crewmember_runs_through_checklist_in_aft_flight_deck_-_DPLA_-_561be79cc182e48a2557d5f83741b2c3.jpg | Public domain -->

## 1️⃣ Specify : définir ce que le logiciel doit faire

La première phase décrit le comportement attendu, pas la manière de le programmer.

Prenons comme fil rouge une fonctionnalité de réinitialisation de mot de passe. Une demande insuffisante serait :

> Ajouter « mot de passe oublié ».

Une spécification exploitable précise l’objectif, les acteurs, les règles et les résultats observables.

### Objectif

Permettre à un utilisateur de demander un lien sécurisé afin de choisir un nouveau mot de passe sans révéler l’existence de son compte.

### Règles métier

- la réponse publique est identique, que l’adresse existe ou non ;
- un jeton expire après 30 minutes ;
- un jeton ne peut être utilisé qu’une fois ;
- une nouvelle demande invalide les précédents jetons actifs ;
- le nouveau mot de passe respecte la politique de sécurité ;
- le changement invalide toutes les sessions actives du compte ;
- la fréquence des demandes est limitée par compte et par adresse IP ;
- l’opération est journalisée sans enregistrer le jeton en clair.

### Scénario d’acceptation

```gherkin
Étant donné un utilisateur disposant d'un jeton valide
Quand il soumet un nouveau mot de passe conforme
Alors le mot de passe du compte est remplacé
Et le jeton devient immédiatement inutilisable
```

À ce stade, l’agent IA peut être utilisé comme contradicteur. Demandez-lui de relever les ambiguïtés, cas limites et risques, mais pas encore d’écrire l’implémentation.

Une consigne efficace serait :

> Analyse cette spécification comme un reviewer sécurité. Liste uniquement les décisions manquantes et les scénarios non couverts. N’invente pas les réponses.

L’humain reste responsable des arbitrages. L’agent aide à voir ce qui manque.

## 2️⃣ Plan : décider comment respecter le contrat

La phase de planification traduit le « quoi » en « comment ». Elle décrit l’architecture, les interfaces, les données et les contraintes techniques.

Pour notre exemple, le plan peut couvrir :

- les endpoints de demande et de confirmation ;
- le format et le hachage du jeton ;
- la table ou collection de stockage ;
- le service d’envoi d’email ;
- la stratégie d’expiration et d’invalidation ;
- la limitation de débit ;
- les événements de journalisation ;
- les tests unitaires, d’intégration et d’acceptation ;
- les règles de migration et de retour arrière.

Le plan doit aussi rappeler les conventions du dépôt : framework, organisation des modules, gestion des erreurs, observabilité, dépendances autorisées et exigences de sécurité.

C’est un point souvent oublié. Une spécification fonctionnelle parfaite n’empêche pas un agent de choisir une bibliothèque interdite, de contourner une couche d’architecture ou d’introduire un nouveau modèle de données incohérent.

Demandez ensuite à l’agent de proposer un plan **sans modifier le code** :

> À partir de la spécification et des conventions du dépôt, propose un plan par petits incréments. Pour chaque étape, indique les fichiers concernés, les tests attendus et le critère d’acceptation couvert. Signale toute hypothèse.

La revue humaine valide le plan avant l’exécution. Corriger une mauvaise direction à ce stade coûte beaucoup moins cher qu’après plusieurs centaines de lignes générées.

## 3️⃣ Implement : avancer par incréments vérifiables

L’implémentation ne devrait pas être confiée en un seul bloc avec la consigne « fais tout ».

Découpez le plan en tâches qui produisent chacune un résultat testable. Par exemple :

1. créer le modèle de jeton et ses tests ;
2. implémenter la demande avec une réponse non révélatrice ;
3. intégrer l’envoi d’email derrière une interface ;
4. implémenter la consommation atomique du jeton ;
5. ajouter la limitation de débit et la journalisation ;
6. exécuter les scénarios d’acceptation complets.

Pour chaque tâche, l’agent reçoit seulement le contexte utile : la partie concernée de la spécification, le plan validé, les conventions du module et les tests existants.

Cette réduction du bruit améliore la précision et limite les modifications hors périmètre.

### Le contrat d’exécution de l’agent

Avant chaque incrément, imposez quelques règles :

- ne modifier que les fichiers nécessaires ;
- ne pas changer le comportement hors spécification ;
- écrire ou adapter les tests avant de conclure ;
- exécuter les contrôles réellement disponibles ;
- signaler un conflit entre le plan et le dépôt ;
- ne jamais remplacer une vérification réelle par une sortie supposée.

Une fois l’incrément produit, relisez le diff contre deux références : **respecte-t-il la spécification ?** et **respecte-t-il le plan ?**

Un code qui passe les tests mais contourne l’architecture n’est pas terminé. Un code fidèle au plan mais incapable de satisfaire un cas métier ne l’est pas davantage.

## 4️⃣ Validate : prouver que l’intention est respectée

La validation ferme la boucle. Elle combine l’automatisation et le jugement humain.

### Contrôles automatisés

Selon le projet, la chaîne peut inclure :

- tests unitaires des règles métier ;
- tests d’intégration avec la base et le service d’email ;
- scénarios BDD ;
- tests de contrat API ;
- analyse statique et lint ;
- détection de secrets et audit des dépendances ;
- tests de performance ou de sécurité ciblés.

### Contrôles humains

L’équipe examine aussi ce que les tests couvrent mal :

- l’expérience réelle de l’utilisateur ;
- la qualité des messages et des parcours d’erreur ;
- la cohérence avec le produit ;
- les compromis d’exploitation ;
- les risques résiduels ;
- la lisibilité de la spécification mise à jour.

Lorsque le résultat ne correspond pas au contrat, il faut identifier la bonne correction.

Si l’implémentation est fautive, on corrige le code. Si la spécification était incomplète ou erronée, on la révise, puis on adapte les tests et le code. Ce choix explicite empêche de maquiller un écart en changeant silencieusement le document après coup.

## 🔁 Installer des checkpoints plutôt qu’une surveillance continue

Un agent autonome n’a pas besoin d’une validation humaine après chaque ligne. Il a besoin de frontières claires où il doit s’arrêter et présenter une preuve.

Les checkpoints les plus utiles sont :

- après l’analyse de la spécification ;
- avant l’approbation du plan ;
- après chaque incrément fonctionnel ;
- avant toute migration ou changement irréversible ;
- après l’exécution des tests ;
- avant le merge ou le déploiement.

À chaque checkpoint, l’agent doit fournir des éléments vérifiables : diff, résultats de tests, liste des hypothèses, exigences couvertes et écarts connus.

L’autonomie ne signifie pas l’absence de contrôle. Elle signifie que le contrôle est placé aux bons endroits.

## 📁 Une structure minimale dans le dépôt

Il n’est pas nécessaire d’adopter immédiatement une plateforme dédiée. Une structure légère suffit :

```text
specs/
  reset-password/
    spec.md
    plan.md
    acceptance.feature
```

Le fichier `spec.md` décrit les comportements et critères. `plan.md` conserve les décisions techniques. Le scénario d’acceptation rend le contrat exécutable lorsque l’outillage le permet.

Dans une approche spec-anchored, ces fichiers évoluent dans la même pull request que le code. La CI exécute les tests associés et la revue vérifie la cohérence de l’ensemble.

## 📊 Mesurer si le workflow apporte réellement de la valeur

Le SDD ne doit pas devenir une cérémonie. Sur un pilote, suivez quelques indicateurs simples :

- nombre d’allers-retours avant validation du code ;
- défauts liés à une exigence mal comprise ;
- temps passé à reconstruire le contexte ;
- changements hors périmètre produits par l’agent ;
- couverture des critères d’acceptation ;
- dérive entre spécification et comportement réel.

Comparez ces résultats à des fonctionnalités de taille similaire développées sans workflow explicite.

Si la spécification prend beaucoup de temps sans réduire les reprises, elle est peut-être trop détaillée ou mal ciblée. Si les agents livrent plus vite mais que les écarts augmentent, les critères ne sont probablement pas assez vérifiables.

## 🚀 Un pilote en deux semaines

Pour tester la méthode sans transformer toute l’organisation :

1. choisissez une fonctionnalité limitée mais réellement utile ;
2. désignez un responsable de la spécification ;
3. écrivez les comportements et critères d’acceptation ;
4. faites challenger les ambiguïtés par un agent ;
5. validez un plan avant toute modification ;
6. implémentez en petits incréments ;
7. exigez des preuves de test à chaque checkpoint ;
8. mesurez les reprises et les écarts ;
9. décidez ensuite si le périmètre doit rester spec-first ou devenir spec-anchored.

## ✅ La spécification organise l’autonomie

Le workflow **Specify → Plan → Implement → Validate** ne remplace ni l’expertise métier, ni l’architecture, ni la revue. Il leur donne une place explicite dans une chaîne que les agents peuvent suivre.

La véritable promesse du SDD n’est pas de générer davantage de code. C’est de rendre l’autonomie de l’IA compatible avec la confiance : une intention claire, un plan approuvé, des incréments contrôlables et une validation fondée sur des preuves.

Quand ces quatre éléments sont présents, l’agent ne travaille plus à partir d’une conversation floue. Il agit dans un système de contrats.
