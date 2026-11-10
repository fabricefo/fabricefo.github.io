---
layout: post
title: "JHipster de zéro à une application de gestion de projets avec JDL"
author: fabrice
date: 2026-11-10
image: assets/images/jhipster-exemple-gestion-projets.jpg
categories: dev
tags: [JHipster, Java, Spring Boot, Angular, JDL]
description: "Un tutoriel reproductible pour générer une application JHipster, modéliser des projets et des tâches en JDL, lancer le résultat et le vérifier."
---

JHipster est utile lorsqu’une équipe veut démarrer rapidement une application métier sans assembler manuellement Spring Boot, un client web, la sécurité, la persistance, les migrations de base de données et l’outillage de développement. Il ne remplace pas les décisions d’architecture, mais automatise une grande quantité de code et de configuration répétitifs.

Ce tutoriel suit le parcours du guide officiel [Getting Started](https://www.jhipster.tech/getting-started/) : installer le générateur, créer une application avec l’assistant interactif, décrire le domaine en JDL, importer ce modèle, lancer l’application puis contrôler le résultat. L’exemple est un petit outil de gestion de projets et de tâches. Il reste volontairement monolithique afin de se concentrer sur le flux de travail JHipster plutôt que sur l’infrastructure distribuée.

Les commandes ci-dessous constituent une procédure à reproduire sur votre poste. Elles ne sont pas présentées comme ayant été exécutées dans l’environnement de rédaction de cet article.

*Photo : Joonspoon, [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Programmer_writing_code_with_Unit_Tests.jpg), licence CC BY-SA 4.0.*

## 🧭 Ce que nous allons construire

L’application, nommée `projecthub`, gérera deux concepts principaux :

- un **projet**, avec un nom, une description, un statut et des dates ;
- une **tâche**, avec un titre, un statut, une priorité, une échéance et une estimation ;
- une relation un-à-plusieurs : un projet possède plusieurs tâches et chaque tâche appartient obligatoirement à un projet.

JHipster générera le socle Spring Boot, l’interface web, les API REST, les dépôts de persistance, les migrations Liquibase, les formulaires CRUD et les tests associés selon les options retenues. Nous utiliserons Maven, Angular, une authentification JWT, H2 en développement et PostgreSQL comme cible de production. H2 évite de devoir démarrer un conteneur de base de données pour ce premier essai.

Ce choix est pédagogique, pas universel. Une application d’entreprise peut préférer OAuth 2.0/OIDC, React ou Vue, Gradle, PostgreSQL également en développement, voire une architecture microservices. Le bon réflexe consiste à commencer par les besoins, puis à utiliser le générateur pour matérialiser ces décisions.

## 🧰 Installer les prérequis

La page officielle [Installation](https://www.jhipster.tech/installation/) demande Java 21 LTS ou une version supérieure et recommande une distribution Eclipse Temurin. Elle demande aussi une version LTS 64 bits de Node.js. Git et Docker sont recommandés dans le démarrage rapide : Git pour versionner le projet, Docker pour les services externes et les futurs scénarios de déploiement.

Vérifiez d’abord votre poste :

```bash
java -version
node --version
npm --version
git --version
docker --version
```

Docker n’est pas indispensable pour lancer notre profil de développement avec H2. Une erreur sur la dernière commande ne bloque donc pas ce tutoriel, mais elle limitera les essais avec PostgreSQL, Keycloak ou les fichiers Docker Compose générés.

Installez ensuite JHipster globalement, comme indiqué dans le guide :

```bash
npm install -g generator-jhipster
jhipster --version
```

Si votre organisation interdit les paquets npm globaux, utilisez une stratégie locale maîtrisée par votre équipe et consultez l’aide de la version installée. Le tutoriel officiel privilégie néanmoins l’installation globale. Évitez aussi de générer le projet directement dans votre répertoire personnel : JHipster crée notamment un fichier `.yo-rc.json` qui mémorise la configuration du générateur.

Créez un répertoire dédié :

```bash
mkdir projecthub
cd projecthub
git init
```

## 🏗️ Générer le socle avec l’assistant interactif

Lancez le générateur depuis le répertoire vide :

```bash
jhipster
```

L’assistant pose une série de questions conditionnelles. La [documentation de création d’une application](https://www.jhipster.tech/creating-an-app/) précise que certaines questions apparaissent ou disparaissent selon les réponses précédentes. Leur libellé, leur ordre et la liste exacte des technologies évoluent également entre versions. Ne recopiez donc pas aveuglément un numéro de menu trouvé dans un ancien tutoriel : sélectionnez l’option correspondant à l’intention décrite ci-dessous.

| Intention | Choix conseillé pour l’exemple |
|---|---|
| Architecture | Application monolithique |
| Nom de base | `projecthub` |
| Package Java | `com.example.projecthub` |
| Authentification | JWT |
| Type de base | SQL |
| Base de production | PostgreSQL |
| Base de développement | H2 sur disque ou en mémoire |
| Outil de build | Maven |
| Client web | Angular |
| Internationalisation | Oui, avec le français et éventuellement l’anglais |
| Technologies additionnelles | Aucune pour ce premier passage |
| Tests additionnels | Conserver les valeurs par défaut, sauf besoin explicite |

Les questions de cache, de recherche, de WebSocket, de messagerie, de framework de test ou de thème peuvent varier. Pour un premier projet, restez proche des valeurs par défaut. Ajouter Redis, Elasticsearch ou Kafka « au cas où » augmente le nombre de services à exploiter sans apporter de valeur immédiate à notre CRUD.

À la fin, JHipster installe les dépendances et écrit l’application. Conservez `.yo-rc.json` dans Git : il décrit les choix structurants et permet à l’équipe de comprendre comment le socle a été généré. Faites un premier commit avant d’ajouter les entités afin de séparer clairement la génération initiale du modèle métier :

```bash
git add .
git commit -m "Initialise l'application JHipster"
```

## 🧩 Modéliser projets et tâches en JDL

Le [JHipster Domain Language](https://www.jhipster.tech/jdl/intro/) permet de décrire les entités, leurs champs, leurs validations, leurs relations et plusieurs options de génération dans un fichier texte. La documentation recommande cette approche, notamment parce que les relations deviennent plus faciles à maintenir qu’avec une longue succession de questions interactives.

Créez à la racine du projet un fichier `project-management.jdl` :

```jdl
enum ProjectStatus {
  DRAFT,
  ACTIVE,
  ON_HOLD,
  COMPLETED,
  ARCHIVED
}

enum TaskStatus {
  TODO,
  IN_PROGRESS,
  BLOCKED,
  DONE
}

enum Priority {
  LOW,
  MEDIUM,
  HIGH,
  URGENT
}

entity Project {
  name String required minlength(3) maxlength(120)
  description TextBlob
  status ProjectStatus required
  startDate LocalDate
  dueDate LocalDate
}

entity Task {
  title String required minlength(3) maxlength(160)
  description TextBlob
  status TaskStatus required
  priority Priority required
  dueDate LocalDate
  estimatedHours BigDecimal min(0)
  completedAt Instant
}

relationship OneToMany {
  Project{tasks} to Task{project(name) required}
}

dto Project, Task with mapstruct
service Project, Task with serviceClass
paginate Project, Task with pagination
filter Project, Task
```

Ce modèle exploite les types et validations documentés dans [Entities & fields](https://www.jhipster.tech/jdl/entities-fields/) : `String`, `TextBlob`, `LocalDate`, `Instant`, `BigDecimal`, `required`, `minlength`, `maxlength` et `min`. Les contrôles générés protègent l’API et alimentent aussi la validation côté client, mais ils ne remplacent pas les règles métier complexes.

La relation suit la syntaxe officielle des [relationships JDL](https://www.jhipster.tech/jdl/relationships/). `Project{tasks}` crée la collection de tâches sur le projet. `Task{project(name) required}` crée une référence obligatoire vers le projet et utilise son champ `name` comme libellé d’affichage dans les sélecteurs générés.

Enfin, les [options JDL](https://www.jhipster.tech/jdl/options/) demandent des DTO MapStruct, une couche de service, une pagination et le filtrage des API. Cette structure est plus explicite qu’un accès direct du contrôleur au dépôt et constitue une bonne base pour ajouter ultérieurement des règles métier.

## 📥 Importer le modèle dans l’application

Le démarrage rapide officiel utilise la commande suivante :

```bash
jhipster import-jdl project-management.jdl
```

La documentation JDL présente aussi la forme :

```bash
jhipster jdl project-management.jdl
```

Selon la version du générateur, la commande principale ou l’alias exposé peut différer. Vérifiez `jhipster --help` et utilisez la forme reconnue par votre installation. Dans les deux cas, exécutez l’import depuis la racine de l’application existante.

Lors du premier import, acceptez la création des entités. Lors d’un import ultérieur, lisez attentivement les questions concernant l’écrasement ou la régénération. La variante `--force` régénère les entités et peut écraser des modifications locales ; elle ne doit pas devenir un réflexe. La documentation JDL propose aussi `--json-only` pour ne produire que les fichiers de description dans `.jhipster` sans régénérer tout le code.

Après l’import, examinez au minimum :

```text
.jhipster/Project.json
.jhipster/Task.json
src/main/java/com/example/projecthub/domain/
src/main/java/com/example/projecthub/service/
src/main/java/com/example/projecthub/web/rest/
src/main/resources/config/liquibase/
src/main/webapp/app/entities/
```

Les chemins précis du client peuvent changer avec le framework et la version, mais les fichiers `.jhipster` et les artefacts serveur permettent de confirmer que le modèle a bien été interprété. Créez ensuite un deuxième commit :

```bash
git add .
git commit -m "Ajoute le domaine projets et tâches"
```

## ▶️ Lancer l’application

Pour un projet Maven, la documentation indique que le wrapper généré suffit :

```bash
./mvnw
```

Sous l’invite de commandes Windows, utilisez plutôt :

```bat
mvnw
```

Le premier démarrage peut être long, car Maven et npm téléchargent les dépendances et construisent le client. Une fois Spring Boot démarré, ouvrez [http://localhost:8080](http://localhost:8080). Le profil de développement est utilisé par défaut.

Pour travailler avec le rechargement du client, la documentation de [développement JHipster](https://www.jhipster.tech/development/) prévoit aussi un processus front séparé :

```bash
npm start
```

Dans ce mode, gardez le serveur Java actif dans un autre terminal. Les ports du serveur de développement web peuvent varier selon la génération ; fiez-vous à l’URL affichée par la commande plutôt qu’à une valeur copiée d’un autre projet.

## ✅ Vérifier que l’exemple fonctionne

Une vérification utile ne se limite pas à constater que la page d’accueil s’affiche. Contrôlez successivement les couches générées :

1. vérifiez que le journal Spring Boot ne contient pas d’échec Liquibase ni d’erreur de connexion à la base ;
2. connectez-vous avec un compte disposant des droits d’administration créés par l’application générée ;
3. ouvrez le menu des entités et créez un projet nommé `Refonte extranet` avec le statut `ACTIVE` ;
4. créez une tâche `Préparer le backlog`, associez-la à ce projet et choisissez la priorité `HIGH` ;
5. rechargez les listes pour confirmer la persistance des deux enregistrements ;
6. essayez d’enregistrer un nom de projet de deux caractères : la validation `minlength(3)` doit refuser la saisie ;
7. ouvrez les outils réseau du navigateur et vérifiez que les opérations passent par les API REST générées ;
8. exécutez enfin la suite de tests du build :

```bash
./mvnw test
```

Si l’application démarre mais que le menu des entités est absent, vérifiez que l’import JDL a été lancé dans le même répertoire que `.yo-rc.json`. Si la compilation échoue après une régénération, comparez les changements avec Git avant de relancer quoi que ce soit : les conflits viennent souvent d’une personnalisation manuelle dans un fichier que le générateur souhaite réécrire.

## ⚠️ Limites et bonnes pratiques

JHipster accélère fortement le démarrage, mais le code généré reste du code à posséder. Avant une mise en production, il faut revoir la sécurité, les secrets, la politique CORS, les index SQL, la stratégie de sauvegarde, l’observabilité, les dépendances et les paramètres du profil `prod`. H2 convient au développement local, pas comme substitut implicite à PostgreSQL en production.

Le modèle présenté ne vérifie pas qu’une échéance est postérieure à la date de début, ne calcule pas l’avancement d’un projet et ne contrôle pas les transitions de statut. Ces règles traversent plusieurs champs ou entités : elles appartiennent à la couche de service et doivent être couvertes par des tests dédiés.

Enfin, traitez le JDL comme une source d’architecture versionnée. Modifiez d’abord `project-management.jdl`, importez prudemment, inspectez le diff Git, puis adaptez les extensions métier. Cette discipline permet de profiter durablement de la génération sans transformer chaque mise à jour du modèle en opération risquée.

## 🚀 Aller plus loin

À partir de ce socle, vous pouvez ajouter une entité `Team`, rattacher les tâches à l’entité utilisateur intégrée, exposer des critères de filtrage plus riches ou remplacer JWT par un fournisseur OIDC. Pour visualiser le domaine avant l’import, le guide officiel propose [JDL Studio](https://start.jhipster.tech/jdl-studio/), qui permet de dessiner et d’exporter le modèle.

L’intérêt principal du parcours n’est pas seulement d’obtenir deux écrans CRUD. Il fournit une boucle reproductible : décisions explicites dans l’assistant, domaine versionné en JDL, génération contrôlée, vérification fonctionnelle et commits séparés. JHipster devient alors un accélérateur gouverné par l’équipe, plutôt qu’une boîte noire utilisée une seule fois au démarrage du projet.
