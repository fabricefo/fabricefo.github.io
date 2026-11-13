---
layout: post
title: "LiteLLM : unifier et gouverner l’accès aux modèles d’IA"
author: fabrice
date: 2026-11-13
image: assets/images/litellm.jpg
categories: ai
tags: [LiteLLM, AI Gateway, LLM, Python, Open Source]
description: "Présentation de LiteLLM et guide pratique pour utiliser son SDK Python ou son proxy afin d’unifier les fournisseurs, le routage, les budgets et l’observabilité."
---

LiteLLM répond à un problème devenu courant : une application commence avec un fournisseur de modèles, puis doit intégrer Azure OpenAI, Anthropic, Bedrock, Gemini ou un modèle auto-hébergé. Sans couche d’abstraction, chaque ajout multiplie les SDK, les formats, les clés, les mécanismes de retry et les tableaux de bord.

Le projet propose à la fois une bibliothèque Python et un proxy compatible avec les principales conventions de l’API OpenAI. Il peut rester une simple dépendance applicative ou devenir le point d’entrée mutualisé d’une plateforme IA interne.

*Photo : Markus Spiske, [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Code_on_computer_monitor_%28Unsplash%29.jpg), domaine public CC0.*

## 🧭 LiteLLM, une couche d’unification plutôt qu’un nouveau modèle

LiteLLM est un projet open source qui normalise l’accès à plus de cent fournisseurs de modèles — OpenAI, Anthropic, Azure OpenAI, Amazon Bedrock, Google Vertex AI, Gemini, Mistral, Groq, Ollama, vLLM et bien d’autres — derrière une interface inspirée de l’API OpenAI. Son objectif n’est pas de remplacer les modèles ni les plateformes cloud, mais d’absorber leurs différences de protocoles, de paramètres, d’authentification, d’erreurs et de formats de réponse.

Il faut distinguer LiteLLM du concept générique d’**AI Gateway**. Une passerelle IA est un rôle d’architecture : point d’entrée central, contrôle d’accès, routage, quotas, journalisation et gouvernance. LiteLLM est une implémentation concrète de ce rôle, avec deux produits complémentaires : une bibliothèque Python embarquée et un serveur Proxy, désormais présenté dans la documentation comme le **LiteLLM AI Gateway**. Le [dépôt GitHub officiel](https://github.com/BerriAI/litellm) reste la source du code, tandis que la [documentation LiteLLM](https://docs.litellm.ai/) décrit les fonctionnalités et les syntaxes courantes.

La promesse centrale est l’interface compatible OpenAI. Une application peut envoyer un appel `chat.completions`, `responses`, `embeddings`, image, audio ou reranking vers un modèle pris en charge, puis recevoir une réponse normalisée. Cette compatibilité est large, mais pas magique : tous les fournisseurs n’implémentent pas les mêmes capacités, paramètres ou sémantiques. La [matrice des endpoints supportés](https://docs.litellm.ai/docs/supported_endpoints) et la page de chaque fournisseur restent indispensables avant de considérer deux modèles comme interchangeables.

## 🧩 SDK Python ou Proxy/Gateway : deux usages différents

Le [SDK Python](https://docs.litellm.ai/docs/python_sdk) s’importe directement dans une application. Il convient lorsqu’un seul service Python doit appeler plusieurs fournisseurs, normaliser les réponses et ajouter localement retries, fallbacks, callbacks ou équilibrage. Il n’y a pas de serveur séparé à exploiter : les identifiants des fournisseurs restent dans le processus applicatif.

Le Proxy/Gateway est un service HTTP autonome. Plusieurs applications, langages, équipes ou agents l’appellent avec une URL de base et une clé LiteLLM. Les clés réelles d’OpenAI, Anthropic ou des clouds restent dans la passerelle. C’est le bon choix lorsqu’il faut centraliser les **clés virtuelles**, les autorisations par modèle, les budgets, les limites RPM/TPM, les journaux, les guardrails et le routage. La [documentation du Gateway](https://docs.litellm.ai/docs/simple_proxy) résume cette séparation.

En pratique :

- choisissez le SDK pour un composant Python autonome et une gouvernance locale ;
- choisissez le Gateway pour une plateforme mutualisée, multi-équipe ou polyglotte ;
- combinez les deux si du code Python utilise les fonctions LiteLLM tout en envoyant ses appels vers un Gateway central.

Attention à la concurrence : la documentation précise que les réglages globaux du SDK, tels que `litellm.api_key`, `litellm.callbacks` ou le cache, ne sont pas protégés pour un usage multithread. Pour une forte concurrence, privilégiez les fonctions asynchrones comme `acompletion()` ou déportez les appels vers le Gateway.

## ⚡ Installation et premiers appels

Pour le SDK complet :

Avec `uv` :

```bash
uv add litellm
```

Ou avec `pip` :

```bash
python -m pip install litellm
```

Une distribution `litellm-core` plus légère existe pour l’usage SDK sans CLI ni tableau de bord. N’installez pas `litellm` et `litellm-core` dans le même environnement : ils exposent les mêmes imports Python.

Un appel direct à Anthropic suit le format `fournisseur/modèle` :

```python
import os
from litellm import completion

os.environ["ANTHROPIC_API_KEY"] = "<secret>"

response = completion(
    model="anthropic/claude-sonnet-4-5",
    messages=[{"role": "user", "content": "Résume ce texte en trois points."}],
)

print(response.choices[0].message.content)
```

Le même code peut viser OpenAI, Bedrock ou un endpoint compatible en changeant le nom de modèle, les credentials et, si nécessaire, `api_base`. Consultez toujours le [catalogue officiel des fournisseurs](https://docs.litellm.ai/docs/providers/) : les préfixes, variables d’environnement et paramètres pris en charge diffèrent.

Pour un Gateway local complet avec PostgreSQL et interface d’administration, le [quickstart Docker officiel](https://docs.litellm.ai/docs/proxy/docker_quick_start) propose un script :

```bash
curl -fsSL https://raw.githubusercontent.com/BerriAI/litellm/main/scripts/quickstart.sh | sh
```

Ce raccourci est pratique pour une évaluation, mais une équipe prudente téléchargera et relira le script avant exécution. Pour un démarrage explicite sans base de données, créez plutôt un fichier `litellm_config.yaml` :

```yaml
model_list:
  - model_name: assistant-general
    litellm_params:
      model: openai/gpt-5.2
      api_key: os.environ/OPENAI_API_KEY

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
```

Puis lancez l’image officielle :

```bash
docker run --rm \
  -v "$(pwd)/litellm_config.yaml:/app/config.yaml:ro" \
  -e OPENAI_API_KEY \
  -e LITELLM_MASTER_KEY \
  -p 4000:4000 \
  docker.litellm.ai/berriai/litellm:latest \
  --config /app/config.yaml
```

Pour un test, `latest` simplifie la commande ; en production, épinglez une version exacte ou un digest. Un client OpenAI standard peut ensuite changer uniquement son `base_url` :

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:4000",
    api_key="sk-<cle-litellm>",
)

response = client.chat.completions.create(
    model="assistant-general",
    messages=[{"role": "user", "content": "Bonjour en cinq mots."}],
)
```

## 🗂️ Comprendre `config.yaml`

La [configuration YAML officielle](https://docs.litellm.ai/docs/proxy/configuration/overview) s’organise principalement autour de cinq sections :

- `model_list` décrit les déploiements et leurs alias publics ;
- `router_settings` règle l’équilibrage, les retries et les fallbacks ;
- `litellm_settings` configure le comportement du SDK interne, les callbacks et certaines politiques ;
- `general_settings` configure le serveur, l’authentification et les fonctions du Proxy ;
- `environment_variables` injecte certaines variables, même si un secret store externe reste préférable.

`model_name` est le nom vu par les clients. `litellm_params.model` est la cible réelle transmise au fournisseur. Plusieurs entrées partageant le même `model_name` forment un groupe de déploiements entre lesquels le Router peut répartir la charge.

```yaml
model_list:
  - model_name: assistant-prod
    litellm_params:
      model: azure/assistant-prod-eu
      api_base: os.environ/AZURE_API_BASE_EU
      api_key: os.environ/AZURE_API_KEY_EU
      api_version: os.environ/AZURE_API_VERSION
      rpm: 300
      tpm: 300000

  - model_name: assistant-prod
    litellm_params:
      model: openai/gpt-5.2
      api_key: os.environ/OPENAI_API_KEY
      rpm: 200
      tpm: 200000

  - model_name: assistant-secours
    litellm_params:
      model: anthropic/claude-sonnet-4-5
      api_key: os.environ/ANTHROPIC_API_KEY

router_settings:
  routing_strategy: simple-shuffle
  num_retries: 2
  timeout: 30
  fallbacks:
    - assistant-prod:
        - assistant-secours

litellm_settings:
  drop_params: true
  callbacks:
    - prometheus
    - otel

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  trusted_proxy_ranges:
    - 10.0.0.0/8
```

Cet exemple illustre une structure, pas une recette universelle. Les noms de modèles évoluent, certains paramètres changent selon les versions, et la priorité entre YAML et base de données a notamment changé à partir de LiteLLM 1.103.0. Vérifiez la [référence de configuration](https://docs.litellm.ai/docs/proxy/configuration/overview) correspondant à la version réellement déployée.

## 🔐 Authentification et gestion des clés

Deux niveaux de credentials coexistent. Le client s’authentifie auprès de LiteLLM avec une **clé virtuelle** ; LiteLLM s’authentifie ensuite auprès du fournisseur avec la clé, le rôle IAM ou l’identité cloud configurée pour le déploiement. Cette séparation évite de distribuer les secrets fournisseurs aux développeurs et permet de révoquer un service sans toucher aux autres.

La `LITELLM_MASTER_KEY` est une clé d’administration, pas une clé applicative. Générez une valeur longue et aléatoire — les exemples officiels utilisent le préfixe conventionnel `sk-` —, conservez-la dans un gestionnaire de secrets et réservez-la aux opérations privilégiées. Les applications doivent recevoir des clés virtuelles limitées à leurs modèles, équipes, budgets et débits. La [documentation d’authentification](https://docs.litellm.ai/docs/proxy/authentication) couvre aussi SSO, JWT et authentification personnalisée.

Exemple de création d’une clé plafonnée :

```bash
curl -X POST http://localhost:4000/key/generate \
  -H "Authorization: Bearer sk-<master-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "service-resume-prod",
    "models": ["assistant-prod"],
    "max_budget": 50,
    "budget_duration": "30d",
    "rpm_limit": 60,
    "tpm_limit": 100000
  }'
```

Ne placez jamais les secrets en clair dans Git. Le motif `os.environ/NOM_VARIABLE` permet de les référencer dans le YAML. LiteLLM sait également centraliser des blocs de credentials réutilisables, mais les intégrations à certains secret managers sont soumises à l’édition ou à la licence utilisée. Si des credentials sont stockés en base, fixez `LITELLM_SALT_KEY` une fois : la [documentation de déploiement](https://docs.litellm.ai/docs/proxy/deploy) avertit de ne pas la changer ensuite.

## 🚦 Routage, fallbacks, budgets et limites

Le Router sépare trois mécanismes. L’**équilibrage** choisit un déploiement dans un même groupe. Les **retries** retentent dans ce groupe. Les **fallbacks** quittent le groupe défaillant pour un autre modèle déclaré. La stratégie `simple-shuffle` est la valeur recommandée par la [documentation du Router](https://docs.litellm.ai/docs/routing) pour son faible surcoût ; avec `rpm` et `tpm`, elle pondère les choix selon les capacités déclarées.

Un fallback n’est pas neutre. Changer de fournisseur ou de famille de modèles peut modifier la qualité, le coût, la latence, la fenêtre de contexte, le comportement des outils et la résidence des données. Testez chaque chaîne avec les mêmes prompts, sorties structurées et appels d’outils que la production. Séparez aussi les fallbacks d’indisponibilité des fallbacks liés à une fenêtre de contexte trop courte.

Les budgets et limites peuvent s’appliquer aux clés, utilisateurs, équipes, modèles, tags ou agents. Les RPM limitent les requêtes par minute, les TPM les tokens, et `max_parallel_requests` la concurrence. Point critique : selon la [documentation Budgets et Rate Limits](https://docs.litellm.ai/docs/proxy/users), les budgets reposent sur les dépenses lues en base. Sans PostgreSQL, un `max_budget` global ne constitue pas un plafond fiable et peut laisser passer les appels. Pour du multi-instance, Redis est nécessaire afin de partager les compteurs et l’état de routage.

## 📈 Observabilité exploitable

LiteLLM peut enregistrer le modèle demandé, le déploiement choisi, les tokens, le coût, la latence, la clé, l’utilisateur, l’équipe et les tags. Les callbacks se branchent à OpenTelemetry, Prometheus, Langfuse, Datadog et d’autres outils listés dans la [documentation Logging](https://docs.litellm.ai/docs/proxy/logging).

Pour Prometheus :

```yaml
litellm_settings:
  callbacks:
    - prometheus
```

L’endpoint `/metrics` expose ensuite les métriques et requiert, sur les versions actuelles, une authentification LiteLLM par défaut. Avec plusieurs workers, configurez `PROMETHEUS_MULTIPROC_DIR`. Depuis la version 1.101.0, un port Prometheus dédié peut isoler le scraping du trafic d’inférence ; ce listener séparé ne bénéficie toutefois pas de l’authentification par clé virtuelle et doit rester sur un réseau privé. Pour OpenTelemetry, la configuration courante utilise `callbacks: ["otel"]` et les variables `OTEL_EXPORTER`, `OTEL_ENDPOINT` et éventuellement `OTEL_HEADERS`.

Ne confondez pas observabilité et collecte indiscriminée. Prompts, réponses, en-têtes et métadonnées peuvent contenir des secrets ou des données personnelles. Définissez ce qui est journalisé, masqué, exporté et conservé avant d’activer un callback externe.

## 🏗️ Deux déploiements pratiques

Pour un environnement de développement, un conteneur, un YAML monté en lecture seule et une master key suffisent pour valider la compatibilité OpenAI, le routage et les modèles. Cette topologie sans base ne fournit pas les fonctions complètes de clés virtuelles, de budgets et de suivi des dépenses.

Pour une plateforme de production, la [topologie recommandée](https://docs.litellm.ai/docs/proxy/deploy) comprend au moins deux réplicas LiteLLM derrière un load balancer, PostgreSQL pour les clés et dépenses, Redis pour les compteurs partagés et l’état distribué, plus un job de migration séparé. Les déploiements Kubernetes peuvent utiliser les charts Helm officiels ; AWS et GCP disposent aussi de modules Terraform officiels. Ajoutez des probes sur `/health/readiness`, des limites CPU/mémoire, un PodDisruptionBudget, TLS, des sauvegardes PostgreSQL et des tests de restauration.

Évitez `:latest` en production. Épinglez l’image, vérifiez sa signature, testez les migrations et déployez d’abord en préproduction. Les réglages et exemples de la documentation suivent rapidement les releases : une configuration copiée aujourd’hui n’est pas un contrat de compatibilité pour une ancienne image.

## 🤖 Agents et contextes MCP

LiteLLM peut servir de plan de contrôle aux agents sans devenir leur moteur de raisonnement. Les agents continuent d’exécuter leur boucle, leurs outils et leur mémoire ; le Gateway centralise leurs appels modèles, leurs coûts et leurs autorisations.

La fonction expérimentale [`litellm.agent()`](https://docs.litellm.ai/docs/harness) peut piloter plusieurs harnesses, dont Claude Code, Codex, OpenCode, Deep Agents et une boucle d’outils LiteLLM. Le préfixe `litellm_proxy/` envoie leurs appels vers un Gateway :

```python
import litellm
from litellm import Harness, sandbox

result = litellm.agent(
    Harness.CODEX,
    "Analyse le dépôt et corrige le test instable.",
    sandbox=sandbox.local("./repo"),
    model="litellm_proxy/coder",
)

print(result.text)
print(result.cost)
```

Cette API est annoncée en bêta : épinglez la version et attendez-vous à des changements. L’intérêt opérationnel est que les credentials fournisseurs — et même la clé virtuelle durable — ne doivent pas entrer dans le sandbox de l’agent.

Le [MCP Gateway](https://docs.litellm.ai/docs/mcp/) agrège des serveurs MCP derrière un endpoint fixe, avec permissions par clé, équipe ou organisation. Il prend en charge Streamable HTTP, SSE et, si explicitement activé, `stdio`. Dans un contexte agentique, séparez toujours l’authentification **client vers LiteLLM** de l’authentification **LiteLLM vers le serveur MCP**. La documentation recommande `x-litellm-api-key` pour la clé LiteLLM afin de laisser `Authorization` disponible pour un jeton OAuth ou un credential amont.

MCP augmente fortement la surface d’attaque : un outil peut lire un dépôt, interroger une base ou déclencher une action externe. Appliquez le moindre privilège, limitez les serveurs visibles par clé, exigez une approbation pour les actions sensibles et ne considérez jamais le texte produit par un modèle comme une autorisation.

## 🛡️ Limites et checklist de production

Avant mise en service, appliquez les [guides de sécurité LiteLLM](https://docs.litellm.ai/docs/proxy/security/overview) et la checklist de production :

- épinglez une release stable ou un digest et corrigez rapidement les vulnérabilités ;
- gardez le Gateway sur un réseau privé et exposez uniquement les routes nécessaires ;
- utilisez TLS, vérifiez les certificats et configurez les CA internes ;
- attribuez une clé virtuelle par workload, avec modèles et quotas minimaux ;
- activez SSO ou JWT lorsque l’identité d’entreprise doit remplacer les clés longues ;
- désactivez le login d’administration par variables d’environnement après le bootstrap avec `disable_env_credential_login: true` ;
- configurez `trusted_proxy_ranges` pour éviter de faire confiance à des en-têtes `X-Forwarded-For` forgés ;
- protégez PostgreSQL, Redis, la master key, la salt key et les credentials fournisseurs ;
- contrôlez les logs, callbacks, guardrails et plugins comme du code de production ;
- testez explicitement le comportement en cas de panne fournisseur, Redis, base de données et système d’observabilité.

Enfin, l’abstraction ne supprime pas les différences entre modèles. Elle les déplace dans la configuration, les tests et les politiques. LiteLLM est particulièrement utile quand une organisation veut standardiser l’accès et reprendre le contrôle opérationnel de ses usages IA. Il devient risqué si sa compatibilité OpenAI est interprétée comme une équivalence fonctionnelle totale, si les budgets sont déployés sans base, ou si un Gateway central critique n’est ni redondé, ni surveillé, ni durci.