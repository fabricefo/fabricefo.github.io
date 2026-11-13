---
layout: post
title: "AI Gateway : gouverner les LLM, les agents et les outils MCP"
author: fabrice
date: 2026-11-13
image: assets/images/ai-gateway.jpg
categories: ai
tags: [AI Gateway, LLM, Agents IA, MCP, Gouvernance]
description: "Comprendre le rôle d’un AI Gateway entre applications, agents, serveurs MCP et fournisseurs de LLM, avec ses usages, contrôles et limites."
---

Les premières intégrations d’IA générative commencent souvent par un appel direct depuis une application vers l’API d’un fournisseur de modèles. Cette approche suffit pour un prototype. Elle devient fragile dès que plusieurs équipes, modèles, agents et outils partagent les mêmes services : les clés se dispersent, les quotas sont gérés différemment, les coûts deviennent difficiles à attribuer et chaque changement de fournisseur impose des modifications dans les applications.

Le pattern **AI Gateway** introduit un point de passage contrôlé entre les consommateurs d’IA et les services qu’ils utilisent. Il applique aux flux d’IA les principes déjà éprouvés des API gateways — authentification, routage, limitation, journalisation et politiques — en tenant compte de contraintes propres aux modèles : tokens, prompts, streaming, coûts variables, sécurité du contenu et appels d’outils.

*Photo : Simon Waldherr, [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:CERN_Computer_Center_07.jpg), licence CC BY-SA 4.0.*

## 🧭 Une architecture, pas un produit

Un AI Gateway est une couche d’intermédiation placée devant des modèles, des agents ou des serveurs d’outils. Les applications utilisent une adresse stable et une identité d’entreprise ; la gateway sélectionne le backend, injecte les credentials nécessaires, applique les politiques et produit la télémétrie.

Il ne faut donc pas réduire ce pattern à un produit particulier. Azure API Management le décrit comme un ensemble de capacités permettant de sécuriser, mettre à l’échelle, superviser et gouverner modèles, agents et outils. [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/) met en avant le contrôle du trafic, le cache et l’observabilité. [Kong AI Gateway](https://developer.konghq.com/ai-gateway/) couvre les flux LLM, MCP et agent-à-agent dans un même plan de données. Une implémentation interne peut aussi s’appuyer sur Envoy, NGINX ou un service applicatif ; LiteLLM constitue une option possible pour normaliser et router des appels LLM, mais le pattern est plus large que n’importe lequel de ces composants.

La séparation classique comprend deux plans :

- le **plan de contrôle**, où sont configurés les modèles autorisés, routes, identités, budgets, politiques et destinations de logs ;
- le **plan de données**, traversé par les requêtes et réponses réelles.

Cette séparation limite l’exposition des contenus au système d’administration. Dans son architecture hybride, par exemple, [Kong précise que son control plane reste hors du chemin des données](https://developer.konghq.com/ai-gateway/architecture/) et que la remontée des corps de requêtes et réponses est optionnelle. Le même principe est valable dans une architecture maison : une console de gouvernance ne doit pas recevoir les prompts simplement parce qu’elle distribue la configuration.

## 🏗️ Les composants du pattern

Une architecture générique fait intervenir plusieurs rôles distincts :

1. **Les consommateurs** : applications web, services backend, assistants, IDE, pipelines de données ou agents autonomes.
2. **L’AI Gateway** : point d’entrée, moteur de politiques et couche d’adaptation.
3. **Les fournisseurs LLM** : services cloud, endpoints privés ou modèles auto-hébergés.
4. **Les hôtes d’agents** : runtimes qui maintiennent l’état, construisent le contexte, interrogent le modèle et exécutent sa boucle de décision.
5. **Les clients MCP** : connecteurs gérés par l’hôte, chacun dialoguant avec un serveur MCP.
6. **Les serveurs MCP** : services qui publient des ressources, prompts et outils.
7. **Les systèmes métier** : CRM, ERP, bases documentaires, Git, messagerie, infrastructures cloud ou API internes réellement manipulés par les outils.

La distinction entre agent, client MCP et serveur MCP est essentielle. Selon [l’architecture officielle MCP](https://modelcontextprotocol.io/specification/2026-07-28/architecture), l’hôte coordonne le modèle, le consentement et plusieurs clients ; chaque client communique avec un serveur précis ; le serveur expose des capacités spécialisées. Le LLM ne se connecte donc pas directement à un serveur MCP. Il suggère ou structure un appel d’outil, puis l’hôte et son client MCP décident de l’exécuter selon leurs règles.

## 🔀 Quatre flux à ne pas confondre

### Application vers modèle

Le flux le plus simple est :

`Application → AI Gateway → fournisseur LLM → AI Gateway → application`

La gateway authentifie l’application, valide la requête, choisit un modèle, ajoute le credential du fournisseur, transmet le flux — éventuellement en streaming — puis mesure latence, tokens et coût. Une interface compatible avec un protocole connu peut réduire le couplage, mais elle ne garantit pas l’équivalence fonctionnelle entre modèles : paramètres, fenêtres de contexte, outils natifs et formats d’erreur restent différents.

### Agent vers modèle

Un agent ajoute une boucle d’orchestration :

`Utilisateur → hôte d’agent → AI Gateway → LLM → hôte d’agent`

Le modèle produit une réponse, une demande d’outil ou une étape intermédiaire. L’état de la tâche, la mémoire, le plan et les confirmations doivent rester dans l’hôte d’agent, pas être implicitement confiés à la gateway. Celle-ci gouverne le transport et les politiques transverses ; elle n’est pas, par défaut, le moteur de raisonnement de l’agent.

### Client MCP vers serveur MCP

Lorsqu’un agent a besoin d’une capacité externe, le flux devient :

`Hôte d’agent → client MCP → AI Gateway MCP → serveur MCP → outil ou API métier`

Le client découvre les outils autorisés, puis invoque `tools/call`. La [spécification MCP 2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28) définit ces échanges en JSON-RPC. Une gateway réellement consciente de MCP peut donc appliquer des politiques tenant compte de la méthode appelée et de l’outil demandé, au lieu de se limiter à l’URL et à l’identité du client.

Une gateway MCP peut fonctionner en simple reverse proxy, agréger plusieurs serveurs ou convertir des API existantes en outils. Azure API Management documente par exemple la possibilité d’[exposer une API REST comme serveur MCP et de gouverner les outils](https://learn.microsoft.com/azure/api-management/mcp-server-overview). Kong distingue également le proxy d’un serveur existant, la conversion d’API et l’agrégation de plusieurs sources dans son [modèle de serveur MCP](https://developer.konghq.com/ai-gateway/entities/ai-mcp-server/).

### Serveur MCP vers système métier

Le dernier segment est souvent le plus sensible :

`Serveur MCP → API métier, base de données ou service SaaS`

Une autorisation valide sur l’endpoint MCP ne donne pas automatiquement le droit d’effectuer n’importe quelle opération métier. Il faut décider si le serveur agit avec une identité technique partagée, une délégation utilisateur ou des credentials spécifiques au tenant. Les outils de lecture et d’écriture doivent être séparés, avec des scopes et des confirmations adaptés. Sans cela, la gateway protège l’entrée tout en laissant subsister un « super-compte » derrière le serveur.

## 🧠 Routage, résilience et compatibilité

Le routage peut être statique — un alias vers un modèle — ou dynamique selon le tenant, le type de tâche, la région, le coût, la latence, la disponibilité ou la sensibilité des données. Les stratégies courantes incluent round-robin, pondération, priorité, moindre latence et routage sémantique. Les [capacités AI Gateway d’Azure API Management](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities) associent load balancing, priorités et circuit breakers ; [Cloudflare Dynamic Routing](https://developers.cloudflare.com/ai-gateway/features/routing/dynamic-routing/) combine conditions, répartition progressive, limites de débit, budgets et fallbacks.

Le fallback doit rester explicite. Remplacer un modèle indisponible par un autre peut modifier la qualité, la localisation des données, le coût, les capacités d’appel d’outils ou le format de sortie. Une route de secours doit donc préciser les erreurs déclenchantes, le timeout, le nombre de retries, l’idempotence et les modèles compatibles. Pour un appel d’outil ayant un effet de bord, répéter aveuglément la requête peut créer une commande, un ticket ou un paiement en double.

La gateway peut aussi utiliser un circuit breaker pour retirer temporairement un backend défaillant. Elle ne supprime cependant pas les limites du fournisseur : quotas régionaux, saturation, tailles maximales et incidents globaux doivent être intégrés au dimensionnement.

## 🔐 Deux authentifications, plusieurs autorisations

Le pattern sépare au minimum deux relations :

- **authentification entrante** : qui appelle la gateway et avec quels droits ;
- **authentification sortante** : comment la gateway appelle le modèle, le serveur MCP ou le backend.

Les applications ne devraient pas détenir les clés des fournisseurs. La gateway peut les conserver dans un coffre, utiliser une identité managée ou signer les requêtes. Cette centralisation facilite la rotation, mais transforme la gateway en cible critique : accès d’administration, secrets, sauvegardes et journaux doivent être protégés et audités.

Pour MCP sur HTTP, la [spécification d’autorisation](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization) définit le serveur protégé comme resource server OAuth, le client MCP comme client OAuth et impose la découverte des métadonnées de ressource protégée lorsque l’autorisation est activée. Les scopes demandés doivent suivre le moindre privilège et peuvent être élevés progressivement pour une opération plus sensible.

Il faut éviter le simple passthrough d’un token destiné à la gateway vers un backend différent. Audience, émetteur et scopes peuvent ne pas correspondre. La documentation MCP sur les [bonnes pratiques de sécurité](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices) détaille notamment le confused deputy, le token passthrough, le SSRF et les attaques sur les flux OAuth. La bonne frontière consiste à valider l’identité entrante, prendre une décision d’autorisation, puis obtenir ou générer un credential sortant destiné exactement au service cible.

## 📊 Observabilité et maîtrise des coûts

Une gateway rend possible une télémétrie homogène entre fournisseurs. Les métriques utiles incluent : requêtes, erreurs, timeouts, latence jusqu’au premier token, latence totale, tokens d’entrée et de sortie, modèle réellement utilisé, hit ratio du cache, fallback, appels d’outils et coût estimé. Des dimensions comme application, équipe, environnement, tenant et centre de coûts permettent le showback ou le chargeback.

[Cloudflare documente](https://developers.cloudflare.com/ai-gateway/observability/) logs, analytics, coûts, métadonnées personnalisées et OpenTelemetry. Azure API Management peut [émettre des métriques de tokens et journaliser prompts et complétions](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities). Kong expose également des attributs et métriques OpenTelemetry pour les flux LLM et MCP.

Mais « tout logger » n’est pas une stratégie d’observabilité. Prompts, réponses et arguments d’outils peuvent contenir données personnelles, secrets, code source ou documents confidentiels. Il faut privilégier les métadonnées, hacher ou pseudonymiser les identifiants, filtrer les payloads et définir rétention, résidence et accès. L’activation du contenu complet doit être une décision de sécurité, pas une option de debug permanente.

Les budgets complètent les quotas techniques. Une politique peut limiter les tokens ou dépenses par utilisateur, équipe, modèle ou période. Les [spend limits de Cloudflare](https://developers.cloudflare.com/ai-gateway/features/) illustrent des budgets fondés sur le coût cumulé et les métadonnées. Ces montants restent souvent estimatifs : prix négociés, cache fournisseur, batch, outils et facturation différée exigent une réconciliation avec les factures officielles.

## ⚡ Cache et limitation de débit

Un cache exact réutilise une réponse lorsque la requête et ses paramètres sont identiques. Un cache sémantique recherche une requête suffisamment proche via des embeddings. Azure API Management décrit ce mécanisme avec un [cache externe compatible avec la recherche vectorielle](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities), tandis que Kong propose une [politique de cache sémantique](https://developer.konghq.com/ai-gateway/policies/ai-semantic-cache/).

Le gain de coût peut être important, mais le cache est dangereux pour les réponses personnalisées, temporelles ou autorisées selon l’utilisateur. La clé doit intégrer modèle, version, paramètres, politique, tenant et contexte pertinent. Les résultats d’outils qui modifient un système ne doivent évidemment pas être rejoués comme des réponses de cache.

Le rate limiting peut compter les requêtes, les tokens, le coût ou les appels par outil. Une limite globale protège l’infrastructure ; des limites par identité évitent qu’un tenant monopolise la capacité. Il faut distinguer le contrôle de débit, qui lisse les pics, du budget, qui borne une consommation cumulée. Pour MCP, une limite par outil est souvent plus pertinente qu’une limite uniforme : `search_documents` et `delete_project` n’ont ni le même coût ni le même risque.

## 🛡️ Gouvernance et guardrails

La gateway est un bon emplacement pour imposer un socle commun : modèles approuvés, régions autorisées, tailles maximales, blocage de catégories, détection de données sensibles et règles de conservation. Des guardrails peuvent inspecter les prompts avant transmission et les réponses avant restitution. [Cloudflare Guardrails](https://developers.cloudflare.com/ai-gateway/features/guardrails/) illustre ce filtrage bidirectionnel ; les [politiques Kong](https://developer.konghq.com/ai-gateway/policies/) couvrent notamment filtrage de prompts, garde sémantique et masquage de PII ; Azure permet d’intégrer Content Safety.

Ces contrôles ne constituent pas une preuve de sûreté. Un filtre peut produire des faux positifs, manquer une injection indirecte contenue dans un document ou altérer un format structuré. Les guardrails doivent être testés par cas d’usage, versionnés et observés. Les actions irréversibles exigent encore une autorisation métier, une validation des paramètres et, selon le risque, une confirmation humaine.

La gouvernance doit enfin rester déclarative : politiques versionnées, revue de changement, promotion entre environnements, journal d’audit et exceptions à durée limitée. Une gateway administrée uniquement par clics finit par devenir une configuration centrale impossible à reproduire.

## 🏢 Cas d’usage en entreprise

**Plateforme IA interne multi-équipes.** Une direction technique fournit un endpoint commun vers plusieurs modèles. Chaque équipe reçoit une identité, un catalogue autorisé et un budget. La plateforme centralise les credentials fournisseurs, la télémétrie et les règles de données sans imposer un modèle unique.

**Assistant de support client.** Les requêtes simples sont routées vers un modèle économique ; les cas complexes vers un modèle plus puissant. Les réponses répétitives utilisent un cache contrôlé. Les données personnelles sont masquées et les appels au CRM passent par des outils MCP séparant lecture et modification.

**Copilote de développement.** Les développeurs accèdent aux modèles depuis l’IDE sans clé personnelle. La gateway applique des limites par équipe, bloque certains dépôts ou environnements et mesure le coût. Les outils Git, CI/CD et ticketing sont exposés avec des scopes distincts ; un merge ou un déploiement conserve une approbation explicite.

**Agent d’exploitation cloud.** L’agent consulte métriques, logs et inventaire via MCP. Les outils de diagnostic sont accessibles largement, tandis que redémarrage, changement réseau ou suppression nécessitent une identité renforcée et une confirmation. Les appels sont corrélés de la demande utilisateur jusqu’à l’API cloud.

**SaaS multi-tenant.** La gateway associe chaque requête à un tenant, applique son plan tarifaire, son plafond de dépense et ses modèles autorisés. Le routage et le cache sont cloisonnés par tenant afin d’éviter toute fuite croisée.

## ⚠️ Limites et décisions d’architecture

Un AI Gateway ajoute un saut réseau, de la latence et un point de défaillance potentiel. Il doit être déployé en haute disponibilité, dimensionné pour le streaming et surveillé comme une dépendance critique. Une panne de configuration peut affecter tous les usages simultanément.

La normalisation a aussi une limite : l’intersection commune entre fournisseurs est plus pauvre que leurs fonctions natives. Traduire tous les protocoles vers une API uniforme peut masquer de nouveaux paramètres, casser le streaming ou retarder l’accès à une capacité. Il faut accepter des routes passthrough lorsque la portabilité ne justifie pas la perte fonctionnelle.

La gateway ne remplace ni l’orchestrateur d’agent, ni l’autorisation métier, ni les contrôles du serveur MCP, ni la sécurité des systèmes finaux. Elle ne garantit pas davantage la qualité d’une réponse ou l’absence d’hallucination. Enfin, les versions de protocole et les implémentations MCP évoluent : clients, serveurs et gateways doivent négocier leurs capacités et être testés ensemble plutôt que supposés compatibles.

Le pattern apporte surtout une **frontière opérationnelle commune**. Bien conçu, il découple les applications des fournisseurs, rend les usages mesurables et applique des règles cohérentes aux modèles comme aux outils. Mal conçu, il centralise les secrets, les données sensibles et les risques sans réellement réduire la complexité. La bonne question n’est donc pas « quel AI Gateway installer ? », mais « quelles frontières de confiance, quelles identités et quelles politiques devons-nous rendre explicites ? »
