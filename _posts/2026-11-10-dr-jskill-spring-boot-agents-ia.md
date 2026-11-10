---
layout: post
title: "Dr JSkill : un skill d’agent pour construire des applications Spring Boot à la manière de Julien Dubois"
author: fabrice
date: 2026-11-10
image: assets/images/dr-jskill-spring-boot-agents-ia.jpg
categories: ai
tags: [Dr JSkill, Spring Boot, Java, Agents IA, Open Source]
description: "Analyse technique et critique de Dr JSkill, le skill open source de Julien Dubois qui guide un agent de développement dans la création d’applications Spring Boot modernes."
---

Un agent de code sait produire rapidement une application Spring Boot. Le vrai problème commence après cette première génération : choix incohérents, dépendances ajoutées sans nécessité, configuration fragile, secrets mal gérés ou tests annoncés mais jamais réellement exécutés. Un modèle généraliste connaît beaucoup de solutions possibles ; il ne connaît pas spontanément les conventions précises d’une équipe ni les arbitrages qu’un développeur expérimenté souhaite imposer.

[Dr JSkill](https://github.com/jdubois/dr-jskill) répond à ce problème en fournissant un **Agent Skill** open source, sous licence Apache 2.0, consacré à la création d’applications Java et Spring Boot. Il ne s’agit ni d’un framework Java ni d’un générateur autonome comparable, à lui seul, à Spring Initializr. Le dépôt combine un fichier d’instructions principal, des guides techniques, des scripts Node.js et des ressources prêtes à copier. L’ensemble apprend à un assistant compatible comment initialiser, compléter, tester et préparer au déploiement une application conforme aux choix de Julien Dubois.

*Photo : Daniel Sone / National Cancer Institute, [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Stethoscope_and_Laptop_Computer_-_Nci-vol-9713-300.jpg), domaine public.*

## 🩺 Un cadre technique pour les agents de développement

Cette distinction est essentielle : Dr JSkill ne remplace pas l’agent. Il lui donne une méthode, des contraintes et des outils. Le code métier et l’interface restent produits au cours d’une conversation, donc avec la souplesse mais aussi la variabilité propres aux modèles génératifs.

## 🧠 Ce que contient réellement le dépôt

Le cœur du projet est le fichier [`SKILL.md`](https://github.com/jdubois/dr-jskill/blob/main/SKILL.md). Il décrit les prérequis, les commandes, l’architecture attendue et les pratiques à suivre. Autour de lui, le répertoire `references/` détaille notamment Spring Boot 4, PostgreSQL, les tests, Docker, GraalVM, la sécurité, la configuration, Git et quatre options d’interface : Vue.js, React, Angular ou JavaScript sans framework.

Le dépôt contient aussi des scripts JavaScript multiplateformes. Le lanceur `scripts/create-project` délègue à `create-project-latest.mjs`, lequel interroge [Spring Initializr](https://start.spring.io), choisit une version stable de Spring Boot dans la branche majeure privilégiée, puis télécharge et extrait le projet Maven. Si la résolution en ligne échoue, une version de repli est lue dans `versions.json`.

Après l’extraction, Dr JSkill ne se contente pas du squelette standard. Ses utilitaires ajoutent ou ajustent notamment :

- les fichiers `.gitignore`, `.env.sample`, `.editorconfig`, `.gitattributes` et `.dockerignore` ;
- la configuration de développement, les fichiers Docker Compose et plusieurs Dockerfiles ;
- une configuration JDTLS destinée à la navigation sémantique dans le code Java ;
- Maven Enforcer, avec une version minimale de Maven et une interdiction explicite de Lombok ;
- Maven Failsafe afin que les tests d’intégration nommés `*IT` soient exécutés par `mvn verify` ;
- des réglages adaptés à PostgreSQL, Testcontainers et Spring Boot 4 ;
- un listener affichant les URL utiles au démarrage.

Les versions techniques sont centralisées dans [`versions.json`](https://github.com/jdubois/dr-jskill/blob/main/versions.json). L’intégration continue vérifie la syntaxe des scripts et la synchronisation entre ce manifeste, les guides et l’atelier. Le dépôt transforme ainsi des préférences éditoriales en règles vérifiables plutôt qu’en simple prompt.

## ⚙️ Installation et premier usage

Le README demande Java 25, Node.js 24.x avec npm 11.x et Docker pour les scénarios conteneurisés. La méthode la plus directe consiste à cloner le dépôt :

```bash
git clone https://github.com/jdubois/dr-jskill.git
```

Pour GitHub Copilot CLI, la documentation propose de placer le skill dans son répertoire global :

```bash
mkdir -p ~/.github-copilot/skills
git clone https://github.com/jdubois/dr-jskill.git ~/.github-copilot/skills/dr-jskill
```

Le README indique ensuite que le skill est chargé automatiquement et donne cet exemple :

```bash
gh copilot suggest "create an application using the dr-jskill"
```

Dans VS Code, une autre possibilité consiste à référencer le chemin absolu de `SKILL.md` dans `github.copilot.chat.codeGeneration.instructions`. Pour Claude Code, Windsurf ou un autre assistant compatible avec la [spécification Agent Skills](https://agentskills.io/specification), il faut cloner ou copier le dépôt dans un répertoire de skills reconnu par l’outil concerné.

Le script principal peut également être lancé directement depuis la racine du dépôt :

```bash
node scripts/create-project my-app com.myco my-app com.myco.myapp 25 fullstack \
  --output-dir /chemin/absolu/vers/le/workspace
```

Les paramètres positionnels correspondent au nom du projet, au `groupId`, à l’`artifactId`, au package Java, à la version de Java et au type de projet. Trois profils sont prévus : `basic`, `web` et `fullstack`. Le drapeau `--boot-version` force une version de Spring Boot et `--output-dir` évite surtout de générer l’application à l’intérieur du dossier du skill. Le choix entre Vue, React, Angular ou JavaScript sans framework intervient ensuite dans le travail guidé de l’agent ; le script d’initialisation ne fabrique pas à lui seul l’interface complète.

Le script initialise le socle Spring Boot et prépare l’infrastructure, mais l’agent poursuit ensuite le travail. Pour une application Vue complète, il doit échafauder `frontend/`, normaliser ses dépendances, relier Vite au build Maven, puis écrire les composants et le domaine métier. L’atelier officiel illustre ce déroulement avec une application de gestion de tâches.

## 🌱 L’articulation avec Spring Boot

Dr JSkill s’appuie sur Spring plutôt que de le masquer. Spring Initializr reste la source du projet initial ; Maven reste l’outil de build ; les conventions Spring Boot structurent la configuration et l’exécution. Le profil `fullstack` demande notamment Spring Web, Spring Data JPA, Actuator, Validation, DevTools, le pilote PostgreSQL, le support Docker Compose, Testcontainers et la prise en charge native.

Le skill impose cependant des opinions nettes. Les fichiers `.properties` sont préférés au YAML. PostgreSQL est la base retenue. Hibernate `ddl-auto` est le mécanisme de gestion de schéma documenté, sans Flyway ni Liquibase. Maven est obligatoire, Gradle n’est pas pris en charge. Lombok, OpenAPI/springdoc, Jib et les Buildpacks font partie des choix explicitement écartés. Spring Security reste optionnel et ne doit être ajouté que lorsque l’authentification ou l’autorisation le justifie.

Cette ligne directrice réduit l’espace de décision de l’agent. C’est précisément son intérêt : dans un projet généré par IA, moins de variantes signifie moins de combinaisons mal maîtrisées. En contrepartie, une équipe déjà standardisée sur Gradle, Flyway, OpenAPI ou une autre base devra adapter le skill, créer son fork ou choisir une autre approche.

## 🧬 Dr JSkill et JHipster : même héritage, autre mécanique

La comparaison avec [JHipster](https://www.jhipster.tech/) est assumée dans le README. Les deux projets viennent de Julien Dubois et visent des applications Spring Boot structurées, accompagnées d’un front-end et d’un environnement de production crédible. Dr JSkill cherche d’ailleurs à produire des applications proches de ce que JHipster génère, tout en reflétant plus directement les préférences personnelles de son auteur.

La différence fondamentale est la reproductibilité. JHipster est un générateur : pour une configuration donnée et une version donnée, son comportement est largement déterminé par des templates et des modules. Dr JSkill orchestre un agent probabiliste. Deux exécutions du même prompt peuvent donc différer dans le domaine, les contrôleurs, l’interface ou certains détails de configuration.

Cette non-détermination apporte de la souplesse. L’agent peut discuter du besoin, modifier un projet existant et ajuster l’architecture sans attendre l’écriture d’un nouveau blueprint. Elle apporte aussi un coût : exécution plus lente, consommation éventuelle de jetons payants et nécessité de relire davantage. Le dépôt qualifie lui-même Dr JSkill d’expérimentation et non de successeur officiel de JHipster.

JHipster automatise un catalogue de décisions connu. Dr JSkill transmet des préférences à un développeur artificiel capable d’improviser autour d’elles.

## ✅ Les bénéfices techniques les plus convaincants

Le premier bénéfice est la **cohérence**. L’agent reçoit une doctrine explicite sur les dépendances, la structure des packages, la configuration, les tests et le déploiement. Le deuxième est la **maintenabilité du savoir** : les longues explications sont réparties dans des références spécialisées, tandis que les versions restent centralisées.

Le troisième est l’attention portée aux détails souvent oubliés dans les démonstrations d’IA. Le dépôt distingue tests unitaires et tests d’intégration, configure Failsafe, épingle l’image PostgreSQL de Testcontainers et demande d’exécuter `./mvnw verify`. Il prévoit aussi des images JVM, AOT, native GraalVM et CRaC.

Enfin, Dr JSkill est modifiable. Une équipe peut forker le dépôt, remplacer les opinions de Julien Dubois par ses propres conventions et conserver la même architecture : une instruction principale, des références thématiques, des scripts idempotents et des contrôles automatisés.

## ⚠️ Limites et lecture critique

La richesse du dépôt ne garantit pas la qualité d’une application produite. Les scripts peuvent préparer un excellent socle ; ils ne valident pas à eux seuls le modèle de données, les règles métier, l’ergonomie, les autorisations ou les performances réelles. Le README reconnaît que l’agent est non déterministe. La validation finale doit donc porter sur le code obtenu, pas seulement sur la réputation du skill.

Le choix de versions très récentes réduit aussi la tolérance des environnements d’entreprise. Java 25, Spring Boot 4 et Node 24 peuvent exiger une mise à niveau des postes, des images CI et des outils internes. Le mécanisme de version de repli améliore la robustesse de la génération, mais il ne supprime pas le besoin de tester l’ensemble de la chaîne.

Autre limite : `--frontend` ne signifie pas que le script fabrique seul une interface complète. Il conserve les raccords nécessaires dans les Dockerfiles et indique à l’agent qu’un front-end est attendu ; l’échafaudage et l’intégration restent une étape ultérieure. Il faut donc éviter de présenter Dr JSkill comme un bouton unique produisant systématiquement une application finie.

Enfin, ses décisions sont volontairement tranchées. L’absence de Gradle, de migration de schéma dédiée ou d’OpenAPI peut être parfaitement cohérente pour l’auteur, sans l’être pour toutes les organisations. C’est une base opinionated, pas un standard universel.

## 🔐 La sécurité d’un skill tiers

Installer un skill tiers revient à introduire des instructions et des scripts dans la boucle d’un agent qui peut lire, écrire et exécuter des commandes. Même open source, ce composant doit être traité comme une dépendance de développement sensible.

Dr JSkill fournit plusieurs garde-fous utiles : `.env` doit rester ignoré par Git, l’agent ne doit jamais l’afficher, les secrets doivent être externalisés et l’utilisateur doit approuver les opérations Git. Mais ces règles sont des instructions données au modèle ; elles ne constituent pas une sandbox. Un agent ou une configuration d’outils trop permissive peut toujours provoquer une action indésirable.

Avant adoption, il est prudent de lire `SKILL.md`, `AGENTS.md` et les scripts exécutables, d’épingler un commit ou une version interne plutôt que de suivre aveuglément la branche principale, puis de tester la génération dans un dossier isolé sans secrets. Les commandes proposées doivent être examinées, les différences Git relues et les builds exécutés localement et en CI. Pour un usage professionnel, un fork contrôlé permet d’auditer les mises à jour avant diffusion à toute l’équipe.

Cette prudence s’applique à tout skill capable d’orienter un agent de code. L’ouverture du dépôt facilite justement cet audit.

## 🎯 Un laboratoire crédible pour le développement piloté par agent

Dr JSkill est intéressant moins comme « générateur magique » que comme démonstration d’une nouvelle couche d’outillage. Entre la documentation d’architecture et le code applicatif apparaît désormais un artefact exécutable par l’agent : le skill. Il encode des préférences, fournit des scripts, rappelle les validations et accompagne les modifications futures.

Pour une équipe alignée avec ses choix techniques, il peut accélérer la création d’un socle Spring Boot moderne tout en évitant plusieurs erreurs courantes. Pour les autres, sa meilleure valeur est peut-être méthodologique : montrer comment transformer des conventions internes en instructions versionnées, testées et réutilisables. La bonne posture reste la même qu’avec tout code généré par IA : profiter de la vitesse, mais conserver la revue, les tests et la responsabilité technique du résultat.
