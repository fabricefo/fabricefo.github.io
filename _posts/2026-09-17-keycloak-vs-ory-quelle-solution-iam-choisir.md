---
layout: post
title: "Keycloak vs Ory : quelle solution IAM choisir pour votre architecture ?"
author: fabrice
description: "Comparatif pratique entre Keycloak et Ory : architecture, expérience de connexion, exploitation, fédération d'identité et autorisation."
date: 2026-09-17
image: assets/images/keycloak-vs-ory-choisir-iam.jpg
categories: [Sécurité]
tags: [Keycloak, Ory, IAM, authentification, cybersécurité]
---

Choisir une solution de gestion des identités ne consiste pas seulement à comparer des protocoles. Keycloak et Ory prennent tous deux en charge des standards modernes, mais ils proposent deux manières très différentes de construire un système d'authentification.

Keycloak rassemble l'essentiel dans un serveur intégré. Ory fournit plusieurs services spécialisés que l'équipe assemble selon ses besoins. Ce contraste, mis en évidence dans un [comparatif publié par Cerbos](https://www.cerbos.dev/blog/keycloak-vs-ory), influence autant l'expérience utilisateur que l'exploitation quotidienne.

La bonne question n'est donc pas simplement « lequel possède le plus de fonctionnalités ? ». Il faut surtout déterminer qui construira l'interface de connexion, qui exploitera la plateforme et jusqu'où l'identité doit s'intégrer au produit.

<!-- Image : "Computer Security - Padlock" par perspec_photo88, CC BY-SA 2.0 — https://www.flickr.com/photos/111692634@N04/15327725543 -->

## 🧭 Deux philosophies de l'IAM

[Keycloak](https://www.keycloak.org/) est une plateforme intégrée de gestion des identités et des accès. Un déploiement fournit notamment l'authentification unique, la gestion des utilisateurs, une console d'administration, des pages de connexion personnalisables, la fédération avec LDAP ou Active Directory et la prise en charge d'OpenID Connect, OAuth 2.0 et SAML.

[Ory](https://www.ory.com/docs/oss/getting-started) adopte une architecture modulaire. Son écosystème open source sépare plusieurs responsabilités :

- **Ory Kratos** gère les identités, les justificatifs et les sessions ;
- **Ory Hydra** fournit un serveur OAuth 2.0 et OpenID Connect ;
- **Ory Keto** traite les permissions fines fondées sur les relations ;
- **Ory Oathkeeper** contrôle l'accès à la périphérie du réseau ;
- **Ory Polis** ajoute des fonctions de SSO d'entreprise autour de SAML et OIDC.

Ces deux approches peuvent répondre à des besoins proches, mais elles ne produisent pas la même architecture.

| Critère | Keycloak | Ory |
|---|---|---|
| Structure | Serveur IAM intégré | Services spécialisés et composables |
| Interface de connexion | Pages fournies et personnalisables par thème | Parcours headless, composants et interfaces de référence |
| Administration | Console web complète | Configuration orientée API et outils propres aux composants |
| Fédération d'entreprise | LDAP, Active Directory, SAML et courtage d'identité intégrés | SAML et OIDC via Polis ; assemblage plus modulaire |
| Autorisation | Moteur de politiques intégré | Permissions relationnelles avec Keto |
| Exploitation | Un socle principal à déployer et mettre à niveau | Plusieurs services possibles, plus l'interface utilisateur |

## 🏢 Quand Keycloak est le choix le plus naturel

Keycloak convient particulièrement bien lorsqu'une organisation veut mettre en place rapidement un fournisseur d'identité complet sans développer toute l'expérience de connexion.

Une équipe peut disposer dans un même produit :

- d'un portail d'administration ;
- de pages de connexion et de gestion de compte ;
- de la fédération avec un annuaire d'entreprise ;
- de connexions sociales ou de fournisseurs d'identité externes ;
- de mécanismes d'authentification forte ;
- d'une gestion centralisée des clients, rôles et utilisateurs.

Cette couverture réduit le nombre de composants à sélectionner et à relier. Elle facilite aussi l'administration lorsque les personnes responsables des identités ne développent pas directement l'application.

La contrepartie est une plateforme plus structurante. La personnalisation visuelle passe principalement par le système de thèmes et certaines extensions demandent de comprendre le modèle interne de Keycloak. Il faut également préparer les mises à niveau, tester les flux d'authentification et dimensionner correctement le service.

Keycloak est donc souvent pertinent pour un système d'information hétérogène, des applications internes, un besoin important de fédération ou une équipe qui préfère exploiter un produit IAM central plutôt qu'un ensemble de briques.

## 🧩 Quand Ory devient plus intéressant

Ory est plus adapté lorsque l'identité fait partie de l'expérience du produit. Kratos est conçu pour des parcours sans interface imposée : l'application contrôle l'inscription, la connexion, la récupération du compte et les écrans de vérification.

Ory fournit des SDK, des interfaces de référence et la bibliothèque de composants Ory Elements. L'équipe conserve néanmoins la responsabilité de l'expérience finale. Cette liberté est utile pour un service grand public, une application mobile ou un produit SaaS dont le parcours d'inscription doit rester parfaitement cohérent avec le reste de l'interface.

La modularité permet aussi de ne retenir qu'une partie de l'écosystème. Une organisation disposant déjà de son propre référentiel d'utilisateurs peut, par exemple, utiliser Hydra pour exposer des flux OAuth 2.0 et OpenID Connect sans adopter immédiatement toutes les autres briques.

Cette flexibilité a un coût opérationnel. Chaque service ajouté possède sa configuration, ses données, son cycle de mise à niveau et ses besoins de supervision. L'équipe doit aussi maintenir les écrans et les parcours utilisateur dans la durée, y compris les cas moins visibles comme la récupération de compte, le consentement ou l'authentification multifacteur.

Ory correspond donc mieux à une équipe produit capable d'assumer cette intégration et qui considère le contrôle de l'expérience comme un avantage concurrentiel.

## 🎨 Le véritable point de bascule : l'expérience de connexion

Les protocoles ne suffisent pas à départager les deux solutions. Toutes deux couvrent les besoins courants autour d'OAuth 2.0 et d'OpenID Connect. Le choix devient plus clair lorsqu'on examine la première interaction avec l'utilisateur.

Avec Keycloak, la plateforme fournit une expérience immédiatement exploitable. Il reste possible de modifier les styles, les modèles et certains comportements, mais l'application s'inscrit dans le cadre proposé par l'outil.

Avec Ory, l'équipe construit l'interface et utilise les API ou composants disponibles pour orchestrer le parcours. Elle gagne en liberté, mais elle devient responsable de sa qualité, de son accessibilité et de sa sécurité.

Il faut donc se poser une question simple : **la page de connexion est-elle une fonction d'infrastructure ou une partie stratégique du produit ?**

Si elle doit surtout fonctionner de façon fiable pour plusieurs applications, Keycloak possède un avantage. Si elle doit être conçue comme n'importe quel autre écran du produit, Ory offre davantage de contrôle.

## ⚙️ Comparer le coût d'exploitation, pas seulement le coût de licence

Keycloak et les composants open source d'Ory peuvent être autohébergés. Cela ne signifie pas que leur coût réel soit nul.

Pour Keycloak, l'exploitation se concentre autour d'un service principal et de sa base de données. Il faut surveiller les performances, sauvegarder les données, gérer la haute disponibilité et préparer les mises à niveau.

Pour Ory, la charge dépend du nombre de composants retenus. Une architecture complète peut demander plusieurs déploiements, bases ou schémas, configurations réseau et tableaux de bord. À cela s'ajoute le frontend d'identité développé par l'équipe.

Avant de choisir, il est utile d'estimer :

- le nombre de services à exploiter ;
- la fréquence des évolutions de l'expérience utilisateur ;
- les compétences disponibles en IAM et en frontend ;
- les besoins de support et de correctifs de sécurité ;
- la complexité des migrations ;
- la capacité à tester tous les parcours sensibles.

Une solution légère sur un diagramme peut devenir coûteuse si elle multiplie les responsabilités. À l'inverse, une plateforme plus imposante peut réduire le travail d'intégration.

## 🔐 Authentification et autorisation ne sont pas le même problème

L'authentification détermine qui effectue une requête. L'autorisation décide ce que cette identité peut faire sur une ressource précise.

Keycloak possède des services d'autorisation capables de gérer des ressources, des portées, des rôles et différentes politiques. Cette approche est utile lorsque les règles peuvent être décrites à proximité du fournisseur d'identité.

Ory Keto s'appuie sur un modèle relationnel inspiré de Zanzibar. Il peut exprimer qu'un utilisateur est propriétaire, éditeur ou lecteur d'un document, directement ou par l'intermédiaire d'un groupe ou d'une organisation.

Dans les deux cas, il faut éviter de considérer qu'un rôle placé dans un jeton résout toute l'autorisation applicative. Un rôle comme `editor` ne précise pas nécessairement si une personne peut modifier **ce document**, pour **ce client**, dans **cet état**, au moment de la requête.

Lorsque les règles dépendent fortement des attributs métier, de l'état des ressources ou du contexte de la demande, une couche de décision dédiée peut devenir pertinente. Elle ne remplace pas Keycloak ou Ory : elle utilise l'identité produite par l'IAM pour prendre une décision plus proche de l'application.

## 🧪 Un prototype doit tester les cas difficiles

Une démonstration limitée à une connexion réussie ne permet pas d'évaluer une plateforme IAM. Les difficultés apparaissent surtout dans les parcours secondaires et dans l'exploitation.

Un prototype crédible devrait au minimum vérifier :

1. l'inscription et la connexion ;
2. la récupération d'un compte ;
3. l'authentification multifacteur ;
4. la fédération avec un fournisseur externe ;
5. la révocation d'une session ;
6. la personnalisation de l'interface ;
7. la sauvegarde et la restauration ;
8. une mise à niveau représentative ;
9. une règle d'autorisation liée à une ressource métier ;
10. la qualité des journaux et des traces disponibles.

Il faut également confier le test aux équipes qui exploiteront réellement la solution. Une architecture séduisante pour les développeurs peut être difficile à administrer. Une console très complète peut, à l'inverse, imposer trop de contraintes à une équipe produit.

## ✅ Une grille de décision simple

**Choisissez plutôt Keycloak si :**

- vous voulez une plateforme IAM complète dans un déploiement principal ;
- vous avez besoin d'une console d'administration immédiatement utilisable ;
- LDAP, Active Directory ou la fédération d'identité occupent une place centrale ;
- vous préférez personnaliser des pages existantes plutôt que construire tout le frontend ;
- la cohérence opérationnelle compte davantage que la modularité maximale.

**Choisissez plutôt Ory si :**

- l'inscription et la connexion font partie intégrante de votre produit ;
- vous voulez maîtriser complètement l'interface et les parcours ;
- vous souhaitez adopter uniquement certaines briques IAM ;
- votre architecture et vos équipes sont déjà orientées microservices et API ;
- vous acceptez d'assumer davantage d'intégration et d'exploitation.

## 🎯 Choisir l'architecture que l'équipe saura encore exploiter demain

Keycloak et Ory ne s'opposent pas seulement par leurs fonctionnalités. Keycloak privilégie l'intégration et la couverture fonctionnelle. Ory privilégie la composition et le contrôle de l'expérience.

Le meilleur choix dépend moins de la liste des protocoles que du modèle d'organisation. Une équipe plateforme chargée de plusieurs applications trouvera souvent Keycloak plus direct. Une équipe produit qui veut façonner chaque étape du parcours utilisateur pourra préférer Ory.

Avant de décider, il faut regarder au-delà de la première connexion réussie. L'interface de récupération, les migrations, la supervision, l'autorisation métier et la personne qui assurera l'astreinte dans dix-huit mois sont de meilleurs critères qu'une simple matrice de fonctionnalités.
