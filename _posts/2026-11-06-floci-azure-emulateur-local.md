---
layout: post
title: "Floci for Azure : un émulateur local pour développer et tester sans abonnement cloud"
author: fabrice
description: "Présentation de Floci for Azure, un émulateur local open source qui expose 32 services Azure pour le développement, les tests et la CI."
date: 2026-11-06
image: assets/images/floci-azure-emulateur-local.jpg
categories: [Azure]
tags: [Azure, Floci, émulation, tests, DevOps]
---

Tester une application conçue pour Azure oblige souvent à choisir entre plusieurs options imparfaites : utiliser de vraies ressources cloud, multiplier les émulateurs spécialisés ou remplacer les dépendances par des mocks. La première solution apporte du réalisme, mais demande un abonnement, des droits, du nettoyage et une maîtrise des coûts. Les deux autres accélèrent les boucles locales au prix d’un environnement fragmenté ou trop éloigné du comportement attendu.

[Floci for Azure](https://floci.io/az/), également appelé **floci-az**, propose une autre approche. Ce projet open source sous licence MIT expose sur une machine locale des interfaces compatibles avec de nombreux services Azure. Les SDK officiels, Azure CLI et certains outils d’Infrastructure as Code peuvent ainsi cibler un endpoint local au lieu d’un abonnement cloud.

L’objectif n’est pas de reconstruire tout Azure sur un poste de développement. Floci cherche plutôt à fournir un environnement léger et suffisamment compatible pour le développement, les tests d’intégration et les pipelines CI.

*Photo : Hay Kranen, [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Microsoft_Data_Center_Middenmeer,_the_Netherlands_-_1.jpg), licence CC BY 4.0.*

## ☁️ Qu’est-ce que Floci for Azure ?

Floci for Azure est le membre Azure de la famille d’émulateurs [Floci](https://floci.io/), qui couvre également AWS, Google Cloud et Oracle Cloud Infrastructure. Il démarre un environnement local accessible principalement sur `localhost:4577`.

Le projet annonce actuellement **32 services Azure**. On y trouve les briques de stockage prises en charge par Azurite — Blob, Queue et Table — mais aussi des services comme :

- Azure Functions ;
- Cosmos DB ;
- Azure SQL et plusieurs bases managées ;
- Event Hubs et Service Bus ;
- Key Vault et App Configuration ;
- Event Grid et SignalR ;
- Microsoft Entra ID et une partie de Microsoft Graph ;
- Azure Monitor, Azure Policy et Resource Graph ;
- AKS, Container Apps, Container Instances et Container Registry ;
- API Management, Virtual Network et Virtual Machines.

Cette liste ne signifie pas que chaque fonction du service cloud est reproduite. Floci implémente des API et des protocoles jugés utiles pour les scénarios locaux. Certaines briques sont simulées en mémoire, tandis que d’autres s’appuient sur de vrais moteurs démarrés dans des conteneurs Docker.

## 🧩 Une architecture unifiée, mais pas monolithique

Pour le développeur, la plupart des interactions HTTP passent par le port `4577`. Cette façade unique simplifie la configuration d’un projet qui dépend de plusieurs services Azure : au lieu de lancer et de câbler plusieurs émulateurs, l’application cible un environnement local cohérent.

Sous cette façade, Floci utilise différentes stratégies selon le niveau de fidélité recherché :

- les services sans état et certaines API de gestion sont implémentés directement ;
- Blob, Queue, Table et l’API NoSQL de Cosmos DB disposent d’un stockage local ;
- Azure Functions et plusieurs services complexes utilisent des conteneurs auxiliaires ;
- les bases de données peuvent démarrer de vrais moteurs PostgreSQL, MySQL, MariaDB, SQL Server, Redis ou des moteurs compatibles avec les API Cosmos DB concernées ;
- Event Hubs peut exposer des interfaces AMQP et Kafka en plus de REST.

Cette architecture explique le positionnement du projet. Le cœur peut démarrer rapidement avec une faible empreinte au repos, tandis que les composants plus lourds ne sont mobilisés que lorsqu’un test en a besoin.

## 🆚 Ce que Floci apporte par rapport à Azurite

[Azurite](https://learn.microsoft.com/azure/storage/common/storage-use-azurite) reste l’émulateur officiel de Microsoft pour Azure Storage. Il couvre Blob, Queue et Table et constitue un choix naturel lorsque le besoin se limite à ces services.

Floci reprend les conventions de connexion d’Azurite pour faciliter la migration, puis étend le périmètre à d’autres briques Azure. Blob, Queue et Table utilisent le même compte et la même clé de développement par défaut. Les endpoints passent simplement sur le port `4577`, avec des chemins distincts pour Queue et Table.

Le principal avantage n’est donc pas de remplacer systématiquement Azurite. Il est de pouvoir tester un flux plus large dans un seul environnement. Une application peut, par exemple, déposer un blob, publier un événement, lire un secret et exécuter une fonction sans créer quatre infrastructures cloud ni assembler quatre substituts différents.

Azurite conserve toutefois un atout important : il est développé par Microsoft et se concentre sur la fidélité des services de stockage. Si un projet dépend d’un comportement Azure très précis, il faut comparer les résultats avec l’émulateur officiel et, surtout, avec un environnement Azure réel.

## 🚀 Démarrer avec la CLI Floci

La documentation propose une CLI commune aux différents émulateurs Floci. Sur macOS ou Linux, l’installation peut se faire avec Homebrew :

```bash
brew install floci-io/floci/floci
```

Un script d’installation et une commande PowerShell sont également proposés. Une fois la CLI disponible, l’environnement Azure se lance avec :

```bash
floci az start
eval $(floci az env)
```

La seconde commande exporte notamment `AZURE_STORAGE_CONNECTION_STRING`. Un client Azure Storage peut alors utiliser la même variable que dans un environnement classique.

Il est possible de vérifier l’état, consulter les journaux, diagnostiquer la configuration ou arrêter l’émulateur :

```bash
floci az status
floci az logs
floci az doctor
floci az stop
```

Pour conserver les données entre deux redémarrages, la CLI accepte un répertoire persistant :

```bash
floci az start --persist ./data
```

Floci peut également être exécuté directement avec Docker. Cette voie est utile dans une CI ou lorsque l’équipe ne souhaite pas installer la CLI. Azure Functions nécessite l’accès au socket Docker afin que l’émulateur puisse démarrer les runtimes à la demande ; ce montage n’est pas nécessaire si le projet n’utilise pas Functions.

## 🧪 Un premier test avec Blob Storage

Après l’export de la chaîne de connexion, Azure CLI peut créer un conteneur et charger un fichier contre l’endpoint local :

```bash
az storage container create \
  --name demo \
  --connection-string "$AZURE_STORAGE_CONNECTION_STRING"

echo "Test local avec Floci" > hello-floci.txt

az storage blob upload \
  --container-name demo \
  --name hello-floci.txt \
  --file hello-floci.txt \
  --connection-string "$AZURE_STORAGE_CONNECTION_STRING"
```

Le même principe s’applique aux SDK officiels. Une application Python, Java, .NET ou JavaScript conserve son client Azure habituel et reçoit simplement une configuration locale. Ce point est essentiel : un test d’intégration doit exercer autant que possible le vrai code d’accès au service, pas une implémentation alternative réservée aux tests.

Pour plusieurs services, la commande suivante exporte des endpoints dédiés aux SDK :

```bash
floci az env --format sdk-vars
```

Elle fournit des variables comme `AZURE_STORAGE_BLOB_ENDPOINT`, `AZURE_APP_CONFIGURATION_ENDPOINT` ou `AZURE_KEY_VAULT_ENDPOINT`.

## ⚙️ Intégrer Floci dans une chaîne de tests

Floci devient surtout intéressant lorsque son cycle de vie est automatisé. Un pipeline peut démarrer un environnement vierge, exécuter les tests d’intégration puis supprimer l’instance. Cette isolation limite les interférences entre deux exécutions et réduit la quantité de ressources Azure temporaires à nettoyer.

Une stratégie pragmatique consiste à répartir les tests en trois niveaux :

1. **tests unitaires** avec des dépendances remplacées lorsque le comportement externe n’est pas utile ;
2. **tests d’intégration locaux** avec Floci pour vérifier les SDK, les formats, les protocoles et les enchaînements entre services ;
3. **tests de validation Azure** sur un véritable abonnement pour contrôler les différences de comportement, l’identité, le réseau, les quotas et les fonctions managées.

Floci ne supprime donc pas les environnements cloud de test. Il réduit leur sollicitation et accélère la boucle de développement avant d’arriver à cette étape plus coûteuse.

## 🤖 Un environnement intéressant pour les agents de développement

Le projet met également en avant les agents de codage. Un agent peut construire une fonctionnalité, démarrer Floci, configurer les variables d’environnement et exécuter une vérification sans demander au développeur de lui fournir un abonnement Azure.

Cette autonomie est utile, mais elle doit rester encadrée. L’agent ne doit pas considérer qu’un test local réussi prouve la compatibilité complète avec Azure. Il doit aussi disposer de permissions minimales, utiliser un environnement jetable et restituer les commandes et résultats réellement obtenus.

Floci propose un skill installable pour apprendre à plusieurs assistants de développement à piloter ses émulateurs :

```bash
npx skills add floci-io/floci-cli
```

Comme pour tout contenu tiers destiné à un agent, ce skill doit être lu et vérifié avant installation. Il peut contenir des commandes exécutées avec les droits de l’utilisateur et doit être traité comme du code externe.

## ⚠️ Les limites à intégrer dès le départ

La compatibilité d’API ne garantit pas une équivalence avec le service managé. Les différences peuvent concerner :

- la gestion réelle des identités et des autorisations ;
- les limites, quotas, délais et erreurs transitoires ;
- le comportement réseau et les points de terminaison privés ;
- la réplication, la disponibilité et la cohérence distribuée ;
- les performances et la montée en charge ;
- des fonctions avancées propres à chaque service ;
- les interactions avec le plan de contrôle Azure.

Cosmos DB illustre bien cette frontière. Floci fournit plusieurs interfaces compatibles et s’appuie sur différents moteurs locaux, mais il ne cherche pas à reproduire exactement les RU/s, la distribution multirégion, l’autoscaling ou toutes les garanties de cohérence d’Azure Cosmos DB.

Terraform et OpenTofu demandent par ailleurs une configuration TLS, car le provider Azure découvre les endpoints du cloud en HTTPS. Floci peut activer un proxy TLS et générer un certificat autosigné, mais ce certificat doit être ajouté proprement au magasin de confiance utilisé par l’outil.

Enfin, la présence de clés et de comptes par défaut est adaptée à un environnement local isolé, pas à une exposition réseau. L’émulateur ne doit pas devenir un service partagé ouvert ou une approximation de plateforme de production.

## 🎯 Quand adopter Floci for Azure ?

Floci mérite un essai lorsque l’équipe souhaite :

- raccourcir la boucle de développement d’une application Azure ;
- exécuter des tests d’intégration sur des runners Linux ;
- tester un scénario qui traverse plusieurs services ;
- limiter la création de ressources temporaires et les coûts associés ;
- fournir aux développeurs ou aux agents IA un environnement reproductible ;
- conserver les SDK Azure dans le code de test.

Il sera moins adapté si le projet doit valider une fonction cloud très avancée, une topologie réseau réaliste, des règles IAM complexes ou des garanties distribuées. Dans ces cas, un abonnement Azure de test reste indispensable.

Le bon positionnement est donc celui d’une **couche intermédiaire entre les mocks et le cloud réel**. Les mocks restent excellents pour les tests unitaires rapides. Floci apporte davantage de réalisme pour les intégrations locales. Azure demeure la référence finale pour valider le comportement de production.

Pour une équipe qui dépend de plusieurs services Azure, cette couche peut rendre les tests plus rapides, plus reproductibles et plus accessibles. La meilleure manière de l’évaluer consiste à choisir un parcours concret, à le faire fonctionner localement, puis à comparer explicitement les résultats avec le même scénario exécuté sur Azure.
