---
layout: post
title: "Jev : comprendre le modèle System One de TypeSafe AI (1/2)"
author: fabrice
description: "Présentation de Jev, le modèle System One de TypeSafe AI conçu pour prendre des décisions typées, rapides et exploitables directement par du code."
tags: [Jev, TypeSafe AI, intelligence artificielle, classification, architecture IA]
categories: ai
image: assets/images/jev-system-one-modele-decision-ia.jpg
---

Les grands modèles de langage savent résumer, expliquer, rédiger et raisonner. Mais tous les problèmes d’IA ne demandent pas de générer du texte. Dans beaucoup de systèmes, la question est plus simple : **quelle action faut-il choisir parmi une liste connue ?**

C’est le terrain sur lequel se positionne [Jev](https://docs.typesafe.ai/introduction), le premier modèle dit *System One* de TypeSafe AI. Au lieu de produire une réponse libre, Jev reçoit un état, évalue une ou plusieurs questions fermées et retourne des résultats structurés accompagnés de probabilités.

Cette première partie présente son concept, son fonctionnement et ses limites. La [seconde partie](/jev-cas-usages-premiers-pas/) passera aux cas d’usage et à une première intégration.

*Photo : BalticServers.com, [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:BalticServers_data_center.jpg), licence CC BY-SA 3.0.*

## 🧭 Jev n’est pas un nouveau chatbot

Jev 1.13.0 est un modèle propriétaire proposé par TypeSafe AI. Sa fonction n’est ni de tenir une conversation ni de rédiger une réponse. Il sert à transformer un contexte textuel en **jugements exploitables par un programme**.

Quelques exemples :

- choisir l’équipe qui doit traiter un ticket ;
- estimer le niveau de gravité d’un incident ;
- déterminer si un message exprime une demande de remboursement ;
- sélectionner le modèle ou l’agent adapté à une requête ;
- attribuer une probabilité à une affirmation.

Cette différence est fondamentale. Un LLM génératif répond à une consigne par une séquence de tokens. Jev choisit uniquement parmi les possibilités définies par le développeur. Il ne remplace donc pas un assistant, un moteur de recherche ou un orchestrateur agentique. Il peut, en revanche, devenir une **brique de décision placée devant eux**.

## ⚡ Pourquoi parler de « System One » ?

Le nom fait référence à la distinction popularisée par Daniel Kahneman entre deux modes de pensée :

- le **système 1**, rapide, intuitif et spécialisé dans les jugements immédiats ;
- le **système 2**, plus lent, analytique et adapté au raisonnement délibéré.

TypeSafe reprend cette métaphore pour décrire un modèle conçu pour des décisions ciblées. La documentation recommande d’ailleurs de poser une question qu’une personne compétente pourrait trancher rapidement avec le bon contexte : « ce message exprime-t-il de l’urgence ? » est adapté ; « analyse toute la situation et définis le meilleur plan d’action » ne l’est pas.

Le concept ne signifie pas que Jev reproduit réellement la cognition humaine. C’est avant tout une **orientation d’architecture** : décomposer un problème en petits jugements, puis composer leurs résultats avec du code déterministe.

## 🧩 Le contrat central : un état et des questions typées

Une requête adressée à l’endpoint `POST /v1/systemone` contient trois éléments principaux :

1. `state` : le contenu à évaluer ;
2. `questions` : les décisions demandées au modèle ;
3. `model` : par exemple l’alias `jev-latest`.

L’état peut être une chaîne de caractères, un objet JSON ou un tableau. Il peut représenter un ticket, un échange de support, un document ou l’état courant d’une application. Jev accepte du texte, mais pas des images, de l’audio ou de la vidéo.

Chaque question possède un identifiant, un type et des instructions. Plusieurs questions peuvent être envoyées en une seule requête : elles observent le même état, sont évaluées indépendamment et reviennent sous les identifiants choisis par le développeur.

La fenêtre de contexte annoncée pour Jev 1.13 est de 64 000 tokens au total, avec une limite de 32 000 tokens pour l’état additionné à la plus longue question. Cette capacité ne doit pas devenir une invitation à tout envoyer : la documentation prévient qu’un contexte non pertinent peut dégrader les décisions.

## 🧱 Les trois primitives de Jev

Le modèle expose [trois types de questions](https://docs.typesafe.ai/primitives). Elles définissent à la fois la décision attendue et la forme de la réponse.

### Choice : choisir parmi des options connues

`Choice` convient lorsqu’une entrée doit être affectée à une catégorie sans ordre naturel : équipe de support, type de document, intention utilisateur ou agent destinataire.

Le développeur fournit une carte d’options avec, si nécessaire, une description pour chacune. Jev retourne :

- l’option choisie ;
- la distribution de probabilités sur toutes les options ;
- une valeur de confiance synthétique.

Prévoir une option `other` ou `none_of_the_above` est prudent quand la taxonomie n’est pas exhaustive. Sans elle, le modèle doit sélectionner l’une des catégories proposées, même si aucune ne convient vraiment.

### Score : positionner une entrée sur une échelle

`Score` sert à mesurer un niveau ordonné : gravité, frustration, complexité ou adéquation. Le développeur définit une échelle de deux à dix descriptions, par exemple :

- faible : gêne mineure sans impact métier ;
- moyen : fonctionnalité dégradée avec contournement ;
- élevé : service critique indisponible.

Jev retourne un score qui peut se situer entre deux niveaux, la légende associée, les probabilités et la confiance. L’intérêt est de formaliser une échelle métier plutôt que de demander arbitrairement « une note de 1 à 10 ».

### Noul : estimer un oui ou un non

`Noul` répond à une question binaire avec une valeur comprise entre 0 et 1. Une valeur proche de 1 indique un oui probable, une valeur proche de 0 un non probable, et une valeur proche de 0,5 une forte incertitude.

Cette primitive s’adapte à des affirmations telles que :

- le client demande-t-il explicitement un remboursement ?
- le message contient-il des informations personnelles ?
- le document mentionne-t-il une expérience Kubernetes ?

Contrairement à `Choice` et `Score`, `Noul` ne possède pas de champ de confiance séparé : sa probabilité est directement le signal exploité par le code.

## 📊 Probabilités, confiance et décisions

La réponse contrainte ne supprime pas l’incertitude ; elle la rend visible. Pour un `Choice`, Jev peut par exemple préférer `technical` à `billing` tout en répartissant presque également ses probabilités entre les deux.

Le champ `confidence` résume le degré de concentration de cette distribution. Il ne faut pas le lire comme une garantie universelle de vérité. TypeSafe indique que la confiance est calibrée sur un ensemble de décisions, pas sur chaque réponse prise isolément.

La bonne pratique consiste à faire varier le seuil selon le risque :

- faible confiance : demander une confirmation ou transmettre à un humain ;
- confiance moyenne : autoriser une action réversible ou demander une validation ;
- forte confiance : automatiser uniquement si l’impact d’une erreur reste acceptable.

Les seuils ne doivent pas être copiés depuis un exemple de documentation. Ils doivent être mesurés sur les données réelles du projet.

## 🏗️ Une répartition claire entre le modèle et le code

L’architecture proposée par TypeSafe sépare deux responsabilités :

- **Jev évalue** les éléments ambigus contenus dans le texte ;
- **le code décide** des règles, des seuils, des pondérations et des actions.

Cette séparation est intéressante. Au lieu de cacher toute la logique métier dans un long prompt, on demande plusieurs jugements simples, puis on les combine explicitement :

```text
si catégorie = incident
et gravité > seuil_critique
et confiance suffisante
alors alerter l’astreinte
sinon créer un ticket à vérifier
```

Les règles déterministes restent versionnables, testables et auditables. Le modèle intervient uniquement là où le langage naturel rend une condition difficile à coder.

Jev ne doit donc pas absorber toute la logique d’un workflow. Plus une règle peut être exprimée précisément en code, plus elle doit rester dans le code.

## ✅ Ce que ce modèle peut apporter

Le positionnement de Jev présente plusieurs avantages potentiels :

- des réponses conformes à un schéma connu ;
- aucune extraction fragile d’un choix depuis du texte généré ;
- des probabilités disponibles pour appliquer des seuils ;
- plusieurs jugements réalisables sur le même état ;
- une intégration simple dans des arbres de décision ;
- un coût annoncé très faible pour les tâches de classification.

Le tarif documenté au moment de la rédaction est de **0,042 dollar par million de tokens d’entrée**, sans facturation des tokens de sortie. Ce prix mérite toutefois d’être évalué avec l’ensemble du système : appels réseau, observabilité, reprises, validations humaines et dépendance à une API propriétaire ont eux aussi un coût.

TypeSafe affirme par ailleurs ne pas utiliser les données clients pour entraîner son modèle. Une option *Zero Data Retention* est annoncée pour les offres entreprise. Ces engagements doivent être vérifiés contractuellement avant d’envoyer des données sensibles.

## ⚠️ « Zéro hallucination » ne signifie pas « zéro erreur »

Le discours de TypeSafe insiste sur l’absence d’hallucinations et d’erreurs de type. Il faut interpréter cette promesse précisément.

Jev ne peut pas inventer une quatrième catégorie si le schéma n’en contient que trois. Il ne renvoie pas non plus un paragraphe impossible à analyser à la place d’un booléen. **La sortie est contrainte** : c’est un avantage réel pour l’ingénierie.

Mais le modèle peut toujours :

- choisir la mauvaise option ;
- attribuer une probabilité mal calibrée sur un cas particulier ;
- mal comprendre une formulation ;
- être influencé par du contenu adversarial ;
- produire un jugement inadapté si la taxonomie est mauvaise.

Autrement dit, il réduit les erreurs de forme ; il n’abolit pas les erreurs de jugement. Les performances et la latence publiées par l’entreprise doivent également être considérées comme des affirmations du fournisseur tant qu’elles n’ont pas été reproduites dans son propre environnement.

## 🪚 Les limites documentées de Jev 1.13

TypeSafe publie une page de [« jaggedness »](https://docs.typesafe.ai/model-jaggedness/jev-1.13), ce qui est une bonne pratique : elle décrit les zones dans lesquelles le modèle est moins fiable.

Parmi les limites importantes :

- **lecture littérale** : Jev peut privilégier une formulation explicite sans comprendre toutes les implications ;
- **arithmétique et comptage** : les calculs doivent rester dans le code ;
- **dates** : les comparaisons et déductions temporelles sont fragiles ;
- **indirection** : les références complexes et dépendances entre éléments peuvent échouer ;
- **bruit contextuel** : des données inutiles peuvent détourner l’attention du modèle ;
- **contenu adversarial** : un texte évalué peut tenter d’influencer la décision ;
- **ordre des options** : la position des choix peut introduire un biais ;
- **langues** : l’anglais est la langue principale d’entraînement et offre les résultats les plus prévisibles.

Il faut donc traiter tout contenu externe comme non fiable, réduire l’état au strict nécessaire et tester les variantes de formulation ou d’ordre des catégories. Pour un usage en français, un benchmark local est indispensable.

## 🎯 Où placer Jev dans une architecture IA ?

Jev est pertinent entre une entrée textuelle et un ensemble de chemins connus :

```text
message ou document
        ↓
Jev : classification et scoring
        ↓
seuils et règles dans le code
        ↓
API métier, LLM spécialisé, agent ou humain
```

Il est beaucoup moins adapté lorsque la sortie attendue est une explication, un plan, du code, une synthèse ou une création. Dans ces cas, un modèle génératif reste nécessaire.

Cette complémentarité constitue probablement le point le plus intéressant du concept : utiliser un petit modèle spécialisé pour décider **où** envoyer une demande, puis réserver les modèles génératifs aux tâches qui ont réellement besoin de leurs capacités.

## ✅ Ce qu’il faut retenir

Jev propose une idée simple mais structurante : ne pas demander à un LLM génératif de prendre toutes les décisions du système. Un modèle spécialisé évalue des questions étroites ; les probabilités exposent l’incertitude ; le code conserve les règles et les actions.

La promesse est séduisante pour le routage, la qualification et les garde-fous. Elle ne dispense cependant ni d’une taxonomie bien conçue, ni de tests représentatifs, ni d’un contrôle humain pour les décisions sensibles.

Dans la [partie 2](/jev-cas-usages-premiers-pas/), nous verrons les cas d’usage les plus crédibles, un premier appel API et une méthode pour évaluer Jev avant de l’introduire dans un système de production.
