---
layout: post
title: "Jev en pratique : cas d’usage et premiers pas (2/2)"
author: fabrice
description: "Découvrez les cas d’usage de Jev, son intégration avec des agents IA et un premier appel API pour tester ce moteur de décision typé."
tags: [Jev, TypeSafe AI, agents IA, routage, automatisation]
categories: ai
image: assets/images/jev-cas-usages-premiers-pas.jpg
---

Dans la [première partie](/jev-comprendre-system-one-typesafe-ai/), nous avons vu que Jev n’est pas un chatbot. C’est un modèle *System One* : il évalue un état textuel à travers des questions fermées et retourne des décisions typées avec leurs probabilités.

Reste la question la plus importante : **dans quels cas cette spécialisation apporte-t-elle réellement quelque chose ?** Voici les usages les plus crédibles, une première requête API et une méthode pour mener un POC sans transformer une démonstration séduisante en dépendance prématurée.

*Photo : Mertbiol, [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Aerial_photograph_of_Clapham_Junction_railway_station_2.jpg), licence CC0.*

## 🧭 Cas d’usage 1 : router vers le bon traitement

Le [routage d’intention](https://docs.typesafe.ai/patterns/intent-routing) correspond naturellement au format de Jev. Un message entrant est classé parmi des chemins connus, puis le code sélectionne le traitement approprié :

- une fonction déterministe pour consulter l’état d’une commande ;
- un modèle spécialisé pour répondre à une question produit ;
- un agent disposant d’outils précis pour traiter un incident ;
- un humain lorsque le cas est sensible ou ambigu.

Le gain potentiel ne vient pas uniquement de la vitesse. Il vient de la réduction du contexte et du coût en aval. Au lieu d’envoyer chaque demande au modèle le plus puissant, on mobilise seulement la ressource nécessaire.

Dans une architecture multi-agent, Jev peut ainsi jouer le rôle d’aiguilleur : `finance`, `support`, `sécurité`, `contenu` ou `autre`. Le développeur doit toutefois prévoir une voie de repli. Une liste de choix sans catégorie `other` force le modèle à sélectionner un mauvais chemin quand la demande sort du périmètre.

## 🎫 Cas d’usage 2 : qualifier des tickets en un seul appel

Un ticket de support demande souvent plusieurs jugements : catégorie, gravité, frustration, présence d’étapes de reproduction ou demande de remboursement.

Jev peut recevoir ces questions ensemble :

- un `Choice` pour l’équipe destinataire ;
- un `Score` pour la gravité ;
- un `Score` pour la frustration ;
- un `Noul` pour détecter une demande explicite de remboursement ;
- un `Noul` pour vérifier la présence d’étapes de reproduction.

Chaque résultat est ensuite utilisé uniquement s’il est pertinent. Si la catégorie est `billing`, le score de gravité technique peut être ignoré. Si la catégorie est `bug`, il devient utile pour prioriser le backlog.

Cette approche, que TypeSafe appelle *speculative fan-out*, évite une chaîne d’appels séquentiels. Elle reste lisible car l’arbre de décision se trouve dans le code, pas dans les réponses du modèle.

## 🔎 Cas d’usage 3 : filtrer un pipeline RAG ou une veille

Dans un système RAG, tous les passages récupérés ne méritent pas d’être transmis au modèle de génération. Jev peut attribuer à chacun un score de pertinence ou déterminer s’il satisfait une condition précise.

Le même principe fonctionne pour une veille technologique :

- le sujet correspond-il au cloud, à DevOps ou à l’IA ?
- s’agit-il d’une annonce produit, d’un tutoriel ou d’une analyse ?
- le contenu semble-t-il suffisamment nouveau pour être lu ?
- mérite-t-il une synthèse longue ou seulement un archivage ?

Le filtrage réduit le bruit et réserve les appels coûteux aux documents les plus intéressants. Il ne remplace pas la recherche, l’indexation ou la vérification des faits : il décide seulement quels éléments poursuivent le pipeline.

## 🧾 Cas d’usage 4 : contrôler une citation ou une sortie LLM

La documentation présente un exemple de vérification de citation : le modèle reçoit une affirmation, un extrait source et une question demandant si le passage soutient bien l’affirmation.

Cette brique peut rejoindre une chaîne de contrôle :

1. un LLM rédige une synthèse ;
2. chaque citation est rapprochée de sa source ;
3. Jev estime si la source soutient l’affirmation ;
4. les cas incertains sont envoyés à une revue humaine.

Le résultat n’est pas une preuve automatique. Jev reste un modèle probabiliste et peut mal comprendre une nuance. Mais il peut servir de filtre systématique avant publication, à condition que le seuil et les faux négatifs aient été évalués.

## 🛡️ Cas d’usage 5 : ajouter un garde-fou

Jev peut analyser une entrée ou une sortie de LLM avec plusieurs signaux : présence de données personnelles, demande dangereuse, contenu injurieux ou tentative de manipulation.

Il faut résister à une conclusion trop rapide : un classifieur ne constitue pas, à lui seul, une politique de sécurité. Un garde-fou robuste combine :

- validation du schéma et des entrées ;
- règles déterministes ;
- restrictions d’outils et de permissions ;
- classification probabiliste ;
- confirmation humaine pour les actions sensibles ;
- journalisation et surveillance.

Jev apporte donc un **signal complémentaire**, pas une autorisation d’exécuter. Une probabilité élevée ne doit jamais contourner un contrôle d’accès ou une demande de confirmation obligatoire.

## 🤖 Jev avec Hermes Agent : complément, pas remplacement

Dans un système Hermes Agent, Jev pourrait intervenir avant l’orchestrateur pour qualifier un volume important d’événements : messages, tickets, alertes ou documents de veille.

Un flux possible serait :

```text
événement entrant
      ↓
Jev : thème, urgence, risque, complexité
      ↓
règles de confiance dans le code
      ↓
profil ou agent Hermes spécialisé
      ↓
outil métier ou validation humaine
```

Jev ne remplace ni l’orchestrateur, qui décompose et coordonne le travail, ni les agents, qui recherchent, rédigent ou agissent. Il aide simplement à choisir le bon chemin avant d’engager ces ressources.

À faible volume, cet ajout est probablement inutile. Un orchestrateur peut déjà comprendre quelques messages et les router. Jev devient intéressant lorsque la fréquence, le coût ou la latence rendent un préfiltre spécialisé mesurable : des milliers de tickets, une veille continue ou un grand nombre d’événements machine.

## 🚫 Quand Jev n’est pas le bon outil

Il vaut mieux conserver un LLM génératif, du code classique ou une revue humaine lorsque la tâche exige :

- une réponse rédigée, une explication ou une synthèse ;
- un raisonnement en plusieurs étapes ;
- des calculs, du comptage ou des comparaisons de dates ;
- une compréhension multimodale ;
- une décision réglementaire ou financière sans droit à l’erreur ;
- une taxonomie encore floue ou en évolution permanente ;
- un traitement entièrement local de données confidentielles.

Jev est également peu pertinent si une règle déterministe suffit. Si un champ `priority` vaut déjà `critical`, il n’est pas nécessaire de demander à un modèle de le redécouvrir.

## 🧪 Commencer avec le Playground

Le moyen le plus simple de tester Jev est le [Playground TypeSafe](https://console.typesafe.ai/playground), accessible après création d’un compte. La démarche recommandée est volontairement petite :

1. choisir dix à vingt exemples réels et anonymisés ;
2. coller un exemple dans l’état ;
3. créer une seule question bien définie ;
4. tester les trois primitives si le choix du format n’est pas évident ;
5. vérifier les probabilités, pas seulement la réponse gagnante ;
6. reformuler les critères plutôt que d’ajouter un long prompt ;
7. introduire ensuite des cas ambigus et adversariaux.

L’anglais étant la langue d’entraînement principale de Jev 1.13, il faut comparer explicitement les résultats avec les contenus français du projet. Traduire automatiquement les entrées peut aider dans certains cas, mais ajoute un coût, une latence et un risque de déformation.

## 🔌 Effectuer un premier appel API

Une clé API est créée depuis la console. Elle doit rester dans une variable d’environnement ou un gestionnaire de secrets, jamais dans le code source.

Voici un exemple `curl` qui qualifie un ticket de support avec les trois primitives :

```bash
export TYPESAFE_API_KEY="votre-cle-api"

curl --request POST \
  --url https://api.typesafe.ai/v1/systemone \
  --header "Authorization: Bearer ${TYPESAFE_API_KEY}" \
  --header "Content-Type: application/json" \
  --data '{
    "model": "jev-latest",
    "state": {
      "ticket": "Depuis la mise à jour, notre API renvoie des erreurs 500 en production. Aucun contournement disponible. Pouvez-vous intervenir rapidement ?"
    },
    "questions": {
      "department": {
        "type": "choice",
        "instructions": "Which team should handle `ticket`?",
        "criteria": {
          "technical": "Bugs, outages, integrations and API failures",
          "billing": "Invoices, payments and refunds",
          "sales": "Pricing, plans and new accounts",
          "other": "None of the listed categories"
        }
      },
      "severity": {
        "type": "score",
        "instructions": "How severe is the problem described in `ticket`?",
        "criteria": [
          "Minor inconvenience",
          "Degraded service with a workaround",
          "Critical service unavailable without a workaround"
        ]
      },
      "is_urgent": {
        "type": "noul",
        "instructions": "Does `ticket` explicitly convey operational urgency?",
        "criteria": {
          "true": "A rapid response is explicitly requested or production is impacted",
          "false": "No urgency or production impact is expressed"
        }
      }
    }
  }'
```

La structure suit la [référence API officielle](https://docs.typesafe.ai/api) : un état partagé, un modèle et une carte de questions. Les instructions sont ici rédigées en anglais tout en conservant le ticket en français ; ce choix doit lui aussi être testé, pas supposé meilleur par principe.

La réponse contient les résultats sous `answers`, avec le même identifiant pour chaque question, ainsi que la consommation de tokens. `Choice` retourne notamment le choix, les probabilités et la confiance ; `Score` ajoute le score et sa légende ; `Noul` retourne directement une probabilité de oui.

## 🚦 Transformer une réponse en action sûre

Le code d’application doit rester maître du workflow. Un exemple volontairement simplifié :

```python
def route_ticket(answers):
    department = answers["department"]
    severity = answers["severity"]
    urgency = answers["is_urgent"]["noul"]

    if department["confidence"] < 0.70:
        return "human_review"

    if department["choice"] == "technical":
        if severity["confidence"] < 0.75:
            return "technical_review"
        if severity["score"] > 1.5 and urgency > 0.75:
            return "on_call_confirmation"
        return "technical_backlog"

    if department["choice"] == "billing":
        return "billing_queue"

    return "human_review"
```

Ces seuils sont illustratifs. Ils ne doivent pas être copiés en production. Un seuil pertinent dépend du risque métier, du coût des erreurs et des performances observées sur un jeu de validation.

Un autre point est essentiel : **la confiance ne remplace pas la politique d’autorisation**. Même avec une confiance très élevée, une opération irréversible ou sensible doit conserver ses contrôles déterministes et, si nécessaire, une validation humaine.

## 📐 Construire un POC mesurable

Une évaluation sérieuse peut tenir en quelques étapes :

### 1. Définir une décision étroite

Choisissez un problème fermé : trois catégories de tickets, un score de gravité ou une affirmation binaire. Évitez de démarrer par « analyser toutes les demandes ».

### 2. Constituer un jeu annoté

Rassemblez des exemples représentatifs, anonymisez-les et faites-les annoter par une personne compétente. Conservez aussi les cas ambigus, rares et hors périmètre.

### 3. Séparer développement et validation

Utilisez une partie des exemples pour concevoir les critères, puis une autre pour mesurer le résultat. Modifier les questions après chaque erreur du jeu de test produit une évaluation artificiellement optimiste.

### 4. Mesurer ce qui compte

Selon le cas, observez :

- précision et rappel par catégorie ;
- matrice de confusion ;
- taux de transfert à un humain ;
- erreurs à fort impact ;
- qualité de la calibration ;
- latence aux percentiles élevés ;
- tokens, coût et taux d’échec API.

Le tarif officiel actuel est de **0,042 dollar par million de tokens d’entrée**, sans facturation de sortie. Il reste préférable de mesurer la facture réelle du pipeline plutôt que d’extrapoler uniquement depuis le prix catalogue.

### 5. Tester les angles morts

Ajoutez du bruit, des textes très courts, des demandes mixtes, un ordre différent des choix et des formulations adversariales. Testez séparément le français et l’anglais. Jev 1.13 documente notamment des fragilités sur les dates, l’arithmétique, les références indirectes et le contenu cherchant à influencer le modèle.

### 6. Comparer à une base simple

Un POC doit être comparé à une alternative : règles, classifieur existant ou petit LLM déjà disponible. Si Jev n’améliore ni le coût, ni la latence, ni la qualité opérationnelle, l’intégrer n’apporte rien.

## ✅ Une stratégie d’adoption raisonnable

Jev est plus convaincant lorsqu’il reste à sa place : une couche de jugement rapide entre du texte non structuré et des chemins d’exécution connus.

Pour débuter proprement :

- sélectionner un seul workflow à fort volume ;
- garder la taxonomie courte et exhaustive ;
- exposer une voie `other` et une revue humaine ;
- utiliser les probabilités pour calibrer les seuils ;
- conserver les règles et permissions dans le code ;
- journaliser les décisions sans stocker inutilement de données sensibles ;
- surveiller la dérive après chaque changement de modèle ou de taxonomie.

La bonne question n’est donc pas « Jev est-il meilleur qu’un LLM ? ». Les deux outils ne font pas le même travail. La vraie question est : **avons-nous suffisamment de décisions répétitives, fermées et mesurables pour justifier une couche spécialisée ?**

Si la réponse est oui, Jev mérite un POC. Si le volume est faible ou si le résultat attendu reste ouvert, une architecture plus simple sera souvent préférable.
