---
layout: post
title: "Spec-first, spec-anchored ou spec-as-source : quel niveau de SDD choisir ?"
date: 2026-09-13 07:05:00 +0200
categories: [Développement]
tags: [SDD, architecture logicielle, IA, documentation, qualité logicielle]
image: /assets/images/spec-first-spec-anchored-spec-as-source.jpg
---

Adopter le Spec-Driven Development ne signifie pas nécessairement générer toute une application depuis un document formel. Entre le prompt improvisé et la spécification érigée en source unique, plusieurs niveaux de discipline sont possibles.

Le [rapport technique sur le Spec-Driven Development](https://arxiv.org/html/2602.00180v1) distingue trois approches : **spec-first**, **spec-anchored** et **spec-as-source**. Elles ne constituent pas une simple échelle de maturité où le niveau le plus avancé serait toujours préférable. Chacune répond à un contexte, un coût de maintenance et un besoin de contrôle différents.

<!-- Image: https://commons.wikimedia.org/wiki/File:Escalators_at_Auber_RER_station_in_Paris,_recently_renovated,_with_clean_and_modern_underground_architecture.jpg | CC0 -->

## 🗺️ Trois niveaux d’autorité de la spécification

Pour choisir, il faut identifier **l’artefact qui fait autorité lorsque le comportement évolue**.

| Approche | Rôle de la spécification | Source de vérité après livraison | Effort continu | Contexte privilégié |
|---|---|---|---|---|
| Spec-first | Guide l’implémentation initiale | Le code | Faible | Prototype, petite fonctionnalité, expérimentation |
| Spec-anchored | Évolue avec le code et reste vérifiable | Spécification et code alignés | Modéré | Produit durable, équipe, API, domaine métier |
| Spec-as-source | Produit les artefacts exécutables | La spécification | Élevé au départ, industrialisé ensuite | Domaine générable et fortement standardisé |

En allant vers la droite, la spécification gagne en autorité. En contrepartie, les règles de synchronisation, l’outillage et la discipline doivent être plus solides.

## 🥉 Spec-first : clarifier avant de construire

Dans une approche **spec-first**, la spécification est écrite avant le code. Elle donne une cible claire au développeur ou à l’agent IA, mais elle peut ensuite cesser d’être maintenue.

Le code redevient alors l’artefact principal après l’implémentation.

Cette formule est adaptée lorsque :

- la fonctionnalité est courte ou isolée ;
- le coût d’une documentation vivante serait disproportionné ;
- le produit est encore exploratoire ;
- une seule personne assure l’essentiel du développement ;
- l’objectif est surtout d’empêcher l’agent de deviner le besoin initial.

Elle apporte déjà un gain majeur par rapport au développement ad hoc. Les critères d’acceptation forcent l’équipe à trancher les ambiguïtés avant de produire du code.

Sa limite apparaît dans le temps. Si le comportement change sans mise à jour de la spécification, celle-ci devient un document historique. Elle reste utile pour comprendre l’intention d’origine, mais ne peut plus servir de référence fiable.

**Exemple :** une équipe teste pendant deux semaines un nouveau parcours d’inscription. Elle spécifie les comportements attendus, fait générer un prototype, puis conserve uniquement le code si l’expérience est concluante.

## 🥈 Spec-anchored : garder le contrat vivant

Dans l’approche **spec-anchored**, spécification et code évoluent ensemble. Une modification de comportement implique de mettre à jour le contrat et l’implémentation.

L’alignement peut être contrôlé par :

- des scénarios BDD exécutables ;
- des tests d’acceptation ;
- des tests de contrat pour les API ;
- des vérifications dans la CI ;
- une revue exigeant le changement conjoint de la spec et du code.

Cette approche convient à la majorité des systèmes de production. Elle offre une documentation suffisamment fiable pour les équipes, les parties prenantes et les agents IA, sans imposer que tout le code soit généré.

Elle est particulièrement pertinente lorsque :

- le système vivra plusieurs années ;
- plusieurs développeurs ou agents interviennent ;
- des équipes consomment des contrats d’API ;
- le domaine métier contient des règles sensibles ;
- les régressions ont un coût important.

Le principal risque n’est pas technique, mais organisationnel : si la mise à jour de la spécification reste facultative, l’ancre finit par dériver. Une spec-anchored sans contrôle automatisé ni règle de revue peut rapidement redevenir une simple documentation.

**Exemple :** une API publique est décrite par OpenAPI. Toute modification du contrat est examinée, testée et publiée en même temps que l’implémentation. Les agents de développement utilisent cette description à chaque intervention.

## 🥇 Spec-as-source : modifier le modèle, régénérer le système

Avec **spec-as-source**, la spécification devient l’artefact que les humains modifient directement. Le code est généré et ne doit pas être corrigé manuellement, car toute régénération effacerait la modification.

L’idée existe déjà dans plusieurs domaines :

- clients et serveurs générés depuis OpenAPI ;
- schémas de données transformés en modèles ;
- infrastructure générée depuis un modèle déclaratif ;
- interfaces produites depuis un design system ;
- plateformes low-code fondées sur un modèle de domaine.

L’approche devient attractive lorsque le domaine peut être décrit avec un langage suffisamment expressif et que la génération est déterministe, testable et reproductible.

Elle demande cependant des garanties fortes :

- la spécification doit couvrir l’essentiel du comportement utile ;
- le générateur doit être versionné ;
- les résultats doivent être reproductibles ;
- les zones d’extension manuelle doivent être clairement séparées ;
- la validation doit rester indépendante de la génération.

Le danger serait de transformer un langage de spécification en langage de programmation moins pratique, ou de créer un générateur si complexe qu’il devient lui-même le véritable produit à maintenir.

**Exemple :** une plateforme interne décrit des entités, permissions, règles de validation et workflows dans un modèle. Elle génère les API, les migrations, une interface d’administration et les tests de contrat.

## ⚖️ Plus de rigueur ne signifie pas automatiquement plus de valeur

Le coût d’une spécification vivante se justifie lorsque le système dure, lorsque plusieurs acteurs doivent partager la même compréhension ou lorsque l’erreur coûte cher.

À l’inverse, maintenir un modèle formel complet pour un script jetable serait du gaspillage.

Le choix dépend surtout de quatre variables :

### La durée de vie

Plus le produit est durable, plus une spécification maintenue apporte de valeur. Elle réduit la perte de contexte et facilite les évolutions.

### Le nombre d’intervenants

Une personne peut garder beaucoup d’hypothèses en tête. Une équipe, plusieurs fournisseurs ou plusieurs agents IA ont besoin d’un contrat commun.

### Le coût de l’erreur

Paiement, sécurité, santé, conformité ou infrastructure critique justifient une validation plus forte qu’une expérimentation interne réversible.

### La capacité de génération

Spec-as-source n’est réaliste que si les artefacts peuvent être générés sans multiplier les exceptions manuelles. Un domaine instable ou très créatif s’y prête moins bien.

## 🌳 Un arbre de décision simple

Vous pouvez orienter le choix avec quatre questions :

1. **La fonctionnalité sera-t-elle maintenue sur la durée ?** Si non, spec-first suffit souvent.

2. **Plusieurs personnes, équipes ou agents vont-ils la modifier ?** Si oui, privilégiez spec-anchored.

3. **Une dérive entre intention et implémentation serait-elle coûteuse ?** Si oui, rendez les exigences exécutables et contrôlées dans la CI.

4. **Le domaine est-il assez standardisé pour générer le code de façon reproductible ?** Si oui, évaluez spec-as-source sur un périmètre limité.

Pour un produit classique en production, ma recommandation par défaut est **spec-anchored**. C’est le meilleur compromis entre confiance, souplesse et effort de maintenance.

## 🔄 Une même organisation peut combiner les trois

Le choix n’a pas besoin d’être uniforme dans tout le système.

Une équipe peut utiliser :

- spec-first pour un prototype d’interface ;
- spec-anchored pour les règles métier et les contrats d’API ;
- spec-as-source pour les clients SDK et certains modèles de données.

Cette approche par périmètre évite deux extrêmes : laisser les agents improviser partout ou imposer une formalisation maximale à chaque ligne de code.

Elle permet aussi de progresser graduellement. On peut commencer par écrire des critères d’acceptation, rendre ensuite les scénarios exécutables, puis automatiser la génération uniquement là où le retour sur investissement est démontré.

## ✅ Choisir le contrat dont l’équipe prendra soin

Une spécification ambitieuse mais abandonnée vaut moins qu’un contrat simple réellement maintenu.

Le bon niveau de SDD est donc celui que l’équipe peut faire vivre : assez autoritaire pour limiter la dérive, assez léger pour ne pas être contourné, et assez vérifiable pour guider aussi bien les humains que les agents IA.

Dans le [troisième article](/2026/09/13/workflow-sdd-agents-ia/), nous passerons de ce choix à l’exécution avec un workflow concret en quatre phases : **Specify, Plan, Implement, Validate**.
