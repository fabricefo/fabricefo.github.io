---
layout: post
title: "GitLab : construire une plateforme développeur interne sans repartir de zéro"
author: fabrice
description: "Comment utiliser les composants CI/CD, les environnements, la sécurité et l'identité fédérée de GitLab pour bâtir une plateforme développeur interne."
date: 2026-09-09
image: assets/images/gitlab-plateforme-developpeur-interne.jpg
categories: [dev]
tags: [GitLab, Internal-Developer-Platform, Platform-Engineering, DevSecOps, CI-CD]
---

À mesure qu'une organisation grandit, ses pipelines GitLab ont tendance à se multiplier. Chaque équipe adapte son fichier `.gitlab-ci.yml`, ajoute ses variables et invente sa propre façon de déployer. Ce fonctionnement paraît souple au début. Il finit souvent par créer des configurations difficiles à maintenir, des contrôles de sécurité inégaux et une dépendance croissante envers quelques spécialistes.

Le problème ne vient pas du YAML lui-même. Il vient de la charge cognitive imposée aux développeurs pour livrer une application. Quand chaque nouveau service exige de comprendre les runners, les registres, les droits cloud, les modules d'infrastructure et les règles de production, la chaîne de livraison devient un produit technique que chacun doit reconstruire.

Une plateforme développeur interne cherche précisément à réduire cette complexité. Après un [premier article consacré aux Internal Developer Platforms](/internal-developer-platform/), regardons comment GitLab peut fournir une partie de ce socle sans prétendre remplacer, à lui seul, tout l'écosystème d'une plateforme.

## 🧭 GitLab est un socle, pas une IDP prête à l'emploi

Une plateforme développeur interne ne se résume pas à un portail ou à une collection de pipelines. Elle propose aux équipes des parcours simples et documentés pour réaliser les opérations courantes : créer un service, lancer les contrôles attendus, obtenir un environnement, déployer et consulter l'état de l'application.

GitLab possède plusieurs briques utiles pour construire ces parcours :

- les [composants CI/CD réutilisables et leur catalogue](https://docs.gitlab.com/ci/components/) ;
- les modèles de projets et de pipelines ;
- le [suivi des environnements et des déploiements](https://docs.gitlab.com/ci/environments/) ;
- les règles d'approbation et la protection des environnements sensibles ;
- les [outils de sécurité intégrables aux pipelines](https://docs.gitlab.com/user/application_security/) ;
- les [jetons d'identité OIDC](https://docs.gitlab.com/ci/cloud_services/google_cloud/) pour accéder à des services cloud sans clé permanente.

Ces fonctions ne définissent toutefois ni les standards de l'entreprise ni l'expérience offerte aux développeurs. L'équipe plateforme doit encore décider quels parcours supporter, quelles valeurs proposer par défaut et quelles exceptions accepter. GitLab fournit les matériaux. La plateforme reste un produit interne à concevoir.

## 🧩 Remplacer le copier-coller par des composants versionnés

Le copier-coller de fichiers CI/CD fonctionne jusqu'au jour où une règle doit évoluer dans des dizaines de dépôts. Chaque copie devient alors une variante à retrouver, comparer et corriger.

Les composants CI/CD de GitLab permettent de publier une unité de configuration réutilisable, avec des paramètres d'entrée et une version. Une équipe peut ainsi consommer un pipeline standard sans posséder toute sa logique interne :

```yaml
include:
  - component: $CI_SERVER_FQDN/platform/ci-components/service-standard@2.1.0
    inputs:
      runtime: node
      deploy_target: staging
```

Dans cet exemple, le composant peut encapsuler la construction de l'image, les tests, les analyses de sécurité et le déploiement. Le dépôt applicatif ne conserve que les choix qui lui appartiennent réellement.

Le versionnage est essentiel. Une référence explicite permet aux équipes de tester une nouvelle version avant de l'adopter. Une référence flottante comme `@latest` est pratique pour expérimenter, mais elle peut modifier un pipeline de production sans changement visible dans le dépôt consommateur.

Un catalogue utile doit donc être géré comme un ensemble de produits logiciels : documentation, tests, journal des changements, politique de compatibilité et procédure de migration. Centraliser du YAML sans ces garanties ne fait que déplacer la dette technique.

## 🛣️ Transformer les standards en chemins préférés

Une plateforme efficace propose des chemins préférés, souvent appelés *golden paths*. Ils regroupent les choix validés par l'organisation pour un type de service donné.

Un chemin standard pour une API pourrait prévoir :

- une structure de dépôt et un pipeline de départ ;
- les étapes de test et de contrôle attendues ;
- une méthode commune pour construire et publier l'image ;
- des environnements de développement, de validation et de production ;
- un mécanisme d'authentification auprès du cloud ;
- des règles de journalisation, de sauvegarde et de supervision.

Le développeur choisit le type de service et renseigne quelques paramètres. La plateforme applique le reste. Cette abstraction réduit les décisions répétitives, mais elle ne doit pas masquer les informations nécessaires au diagnostic. Une équipe doit toujours pouvoir comprendre ce qui a été exécuté, consulter les journaux et sortir du chemin standard lorsqu'un besoin le justifie.

Le meilleur parcours n'est pas celui que la direction rend obligatoire. C'est celui que les équipes choisissent parce qu'il leur évite du travail tout en restant prévisible.

## 🔐 Intégrer la sécurité sans distribuer de secrets permanents

La standardisation des pipelines permet d'appliquer certains contrôles de sécurité par défaut. Selon l'édition GitLab utilisée et les licences disponibles, les composants communs peuvent intégrer l'analyse du code, des dépendances, des conteneurs ou la détection de secrets. Les résultats apparaissent alors dans le même flux de travail que les merge requests et les pipelines.

Tous les résultats ne doivent pas bloquer une livraison de la même manière. Une fuite de secret avérée peut justifier un arrêt immédiat. Un signal moins certain demande parfois une analyse avant de devenir une règle bloquante. Sans seuils compréhensibles ni procédure d'exception, les équipes finissent par contourner le système.

L'accès au cloud mérite la même attention. Stocker une clé de compte de service dans une variable CI/CD crée un secret durable qu'il faut protéger et renouveler. GitLab peut émettre un jeton d'identité OIDC pour un job. Un fournisseur cloud compatible, comme Google Cloud avec Workload Identity Federation, échange ensuite cette identité contre des droits temporaires.

Ce modèle évite de conserver une clé statique dans GitLab. Il ne dispense pas de limiter les autorisations. Les règles de confiance doivent identifier précisément le groupe, le projet, la branche ou l'environnement autorisé. Une fédération d'identité trop large remplace un risque par un autre.

## 🌍 Rendre les déploiements visibles et contrôlables

Les environnements GitLab représentent des cibles comme le développement, la validation ou la production. Lorsqu'un job de déploiement déclare sa cible, GitLab peut conserver l'historique des déploiements et indiquer quelle version du code se trouve dans chaque environnement.

Cette visibilité paraît élémentaire, mais elle répond à des questions fréquentes :

- quelle version tourne en production ?
- le correctif a-t-il déjà atteint l'environnement de validation ?
- quel pipeline a réalisé le dernier déploiement ?
- vers quelle version peut-on revenir en cas d'incident ?

Les environnements protégés ajoutent des restrictions sur les personnes autorisées à déployer. Des validations peuvent être introduites pour les cibles sensibles, selon la configuration et l'offre GitLab retenues. L'objectif n'est pas d'ajouter une file d'attente manuelle à chaque livraison. Il s'agit de réserver les contrôles renforcés aux changements qui présentent un risque réel.

## 🧱 Proposer l'infrastructure comme un service encadré

Les modules Terraform ou OpenTofu sont souvent conçus par une équipe plateforme, puis utilisés directement par les équipes applicatives. Cette approche mutualise le code, mais elle expose encore de nombreux détails : fournisseurs, backend d'état, variables, droits et dépendances entre ressources.

Une couche de libre-service peut simplifier l'interface. Le dépôt applicatif contient, par exemple, une courte déclaration décrivant le service souhaité. Un pipeline contrôlé par l'équipe plateforme valide cette déclaration, appelle le module d'infrastructure approprié et publie le résultat.

Cette abstraction ne doit pas donner l'illusion que l'infrastructure se gère seule. L'équipe plateforme reste responsable des versions de modules, de la protection de l'état, des autorisations et des mises à niveau. L'équipe applicative doit, de son côté, garder accès au plan d'exécution, aux coûts attendus et aux conséquences de ses choix.

GitLab orchestre ce flux, mais le moteur d'infrastructure et son backend restent des décisions d'architecture distinctes. Il vaut mieux conserver cette séparation que présenter la chaîne complète comme une fonctionnalité magique de la plateforme.

## 📊 Piloter la plateforme comme un produit interne

Une équipe plateforme ne devrait pas mesurer son succès au nombre de modèles ou de composants publiés. Ces éléments n'ont de valeur que s'ils améliorent réellement le travail des équipes.

Quelques indicateurs permettent de suivre cette utilité :

- le délai entre la création d'un service et son premier déploiement ;
- la part des nouveaux services utilisant un chemin standard ;
- le nombre de demandes manuelles adressées à l'équipe plateforme ;
- le taux d'échec des pipelines et des déploiements ;
- le temps nécessaire pour adopter une nouvelle version d'un composant ;
- les retours des développeurs sur les étapes les plus pénibles.

Les entretiens avec les équipes restent indispensables. Un tableau de bord peut montrer qu'un composant est peu utilisé, mais pas expliquer si sa documentation est mauvaise, s'il manque une option ou si le parcours ne répond à aucun besoin réel.

La plateforme doit aussi assumer un périmètre. Vouloir couvrir immédiatement tous les langages, tous les clouds et tous les services historiques produit généralement une couche complexe que personne ne maîtrise. Un premier parcours bien entretenu vaut mieux qu'un catalogue étendu mais fragile.

## 🚧 Les pièges à éviter

Construire sur GitLab réduit le nombre d'outils à assembler, sans supprimer les risques classiques du platform engineering.

Le premier piège consiste à confondre standardisation et rigidité. Un chemin préféré doit faciliter la majorité des cas, avec une sortie documentée pour les besoins particuliers.

Le deuxième est de déplacer le goulot d'étranglement. Si chaque évolution d'un composant exige l'intervention de la même petite équipe, la plateforme reproduit sous une autre forme le système de tickets qu'elle devait éliminer.

Le troisième est de sous-estimer la maintenance. Les composants, les images de build, les modules d'infrastructure et les politiques de sécurité évoluent. Ils ont besoin de responsables identifiés et de cycles de mise à jour.

Enfin, toutes les fonctions évoquées ne sont pas disponibles de façon identique dans chaque édition de GitLab. Le périmètre fonctionnel, les licences et les contraintes d'une instance autogérée doivent être vérifiés avant d'arrêter l'architecture.

## 🚀 Par où commencer concrètement ?

Le bon point de départ est un parcours fréquent qui génère déjà des demandes répétitives. Par exemple, la création et le déploiement d'une API standard.

Une démarche progressive peut suivre quatre étapes :

1. observer le parcours actuel et relever les attentes, les délais et les contournements ;
2. extraire un premier composant CI/CD versionné avec une interface réduite ;
3. ajouter les environnements, l'authentification temporaire et les contrôles réellement nécessaires ;
4. tester le parcours avec une équipe volontaire avant de l'étendre.

Cette première version n'a pas besoin d'un portail sophistiqué. Un modèle de projet, un catalogue de composants et une documentation courte peuvent suffire. Le portail devient utile lorsque le nombre de services, de ressources et d'utilisateurs justifie une interface de découverte plus riche.

## 🎯 Faire de GitLab une interface, pas une nouvelle couche de complexité

GitLab peut réunir une grande partie du flux de livraison : code, merge requests, pipelines, contrôles, environnements et historique des déploiements. Cette proximité en fait un socle crédible pour une plateforme développeur interne, surtout lorsque les équipes l'utilisent déjà.

La différence entre une collection de pipelines et une vraie plateforme se trouve pourtant ailleurs. Elle tient à la qualité des interfaces, à la stabilité des chemins proposés et à la capacité de l'équipe plateforme à écouter ses utilisateurs.

Avant d'ajouter un portail ou un nouvel outil, il est donc utile d'examiner ce que GitLab peut déjà standardiser. Si un développeur peut créer et livrer un service sans apprendre toute l'infrastructure sous-jacente, tout en gardant assez de visibilité pour comprendre ce qui se passe, la plateforme commence à remplir son rôle.
