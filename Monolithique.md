# L’architecture monolithique
## Définition simple
Une application monolithique est une application livrée et exécutée comme une seule unité (un seul binaire/processus ou une seule image/paquet), où l’UI, la logique métier et l’accès aux données sont regroupés dans le même déploiement.
On l’oppose aux microservices, qui décomposent le système en services indépendants.
## Caractéristiques clés
### Structure
* Un seul codebase, souvent organisé en couches ou modules internes.
* Couplage interne fort : les modules partagent le même processus et les mêmes dépendances.
* UI, logique métier et accès aux données regroupés dans un même artefact.
### Déploiement & exploitation
* Une seule unité de déploiement : on scale en répliquant la même application derrière un load balancer.
* Rollback ou blue-green déployments simplifiés.
* Observabilité centralisée (logs, métriques, traces dans un même process).
### Données
* Souvent un seul datastore partagé → transactions locales simples (ACID).
### Évolutivité
* *cale vertical (machine plus puissante) ou horizontal (répliques identiques).
Pas de mise à l’échelle indépendante par domaine.
### Organisation & cycle de vie
* Démarrage rapide : moins d’infrastructure, un pipeline CI/CD unique.
* En grandissant → risque de fort couplage, builds longs, difficulté à répartir le travail.
* Possibilité de structurer en monolithe modulaire pour maintenir des frontières internes claires.
## Exemples d’implémentation
### 1. Monolithe en couches (classique)
```text
[ Client Web/Mobile ]
         │ HTTP
         ▼
┌───────────────────────────────┐
│           Application         │   ← 1 déploiement
│  ┌─────────────┬───────────┐  │
│  │ Présentation│    API    │  │
│  └─────────────┴───────────┘  │
│  ┌──────────────────────────┐ │
│  │      Logique métier      │ │
│  └──────────────────────────┘ │
│  ┌──────────────────────────┐ │
│  │     Accès aux données    │ │
│  └──────────────────────────┘ │
└───────────────────────────────┘
              │
              ▼
        [ Base de données ]
```
* Outils typiques :
  * Java / Spring Boot (.jar/.war)
C# / .NET MVC
Ruby on Rails
Django
  * Déploiement sur VMs ou conteneurs identiques derrière un load balancer.
### 2. Monolithe conteneurisé (cloud)
            Internet
                │
        [ Load Balancer ]
          /     |     \
        ┌─┐    ┌─┐    ┌─┐
        │C│    │C│    │C│   C = même image Docker du monolithe
        └─┘    └─┘    └─┘
          \     |     /
               [DB]
* Déploiement sur Azure App Service, VM Scale Sets, ou un orchestrateur léger (ECS, AKS) pour scaler le même conteneur.
### 3. Monolithe modulaire (frontières internes)
```text
┌─────────────────────────────────────────────┐
│                 Monolithe                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│  │  Ventes  │ │ Paiement │ │  Stock   │ …   │  = modules/domains
│  └──────────┘ └──────────┘ └──────────┘     │
└─────────────────────────────────────────────┘
```
* Frontières claires entre modules (packages, libraries, etc.)
* Règles de dépendances strictes
* Tests contractuels internes
* Prépare le terrain pour d’éventuelles extractions ciblées
## Projets connus (usages concrets)
* GitHub.com — « depuis le début, un monolithe Ruby on Rails », ~2 M de lignes, >1000 ingénieurs, déploiements très fréquents.
* Shopify — monolithe Rails modulaire (“Modular Monolith”) avec outillage pour garder des frontières internes nettes.
* Stack Overflow — monolithe .NET très optimisé, répliqué, détaillé par Nick Craver (série d’articles d’architecture). 
* Prime Video (cas spécifique) — une brique de monitoring est passée d’un design serverless/microservices à un monolithe, –90% de coûts et meilleure montée en charge pour l’usage visé (article d’ingénierie). 
## Avantages / limites (en bref)
#### Atouts
* Simplicité opérationnelle : un pipeline, un déploiement, observabilité centralisée. 
* Time-to-market rapide, cohérence transactionnelle simple. 
#### Points d’attention
* Couplage et taille qui croissent : builds/tests plus longs, friction entre équipes. 
* Scale indépendant par domaine impossible (on scale tout ensemble). 
#### Bon sens architectural
* Commencer monolithe d’abord (Monolith-First) puis n’extraire que si les contraintes réelles (organisationnelles ou techniques) l’exigent. 
## Pistes d’implémentation (stack)
* Web MVC : Rails / Django / Spring Boot / ASP.NET MVC en monolithe.
* Build & CI : pipeline unique (tests unitaires/contractuels par module).
* Déploiement : VM(s) ou conteneurs identiques ; blue-green/rollbacks. 
* Observabilité : logs + metrics + traces au niveau process ; export vers stack centralisée. 
* Modularisation : “modular monolith” (frontières de dépendances, design DDD).
## Sources fiables
* [IBM – What is monolithic architecture?](https://www.ibm.com/think/topics/monolithic-architecture)
* [Red Hat – What is an application architecture?](https://www.redhat.com/en/topics/cloud-native-apps/what-is-an-application-architecture?utm_source=chatgpt.com)
* [Microsoft Azure – Common web application architectures](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures?utm_source=chatgpt.com)
* [Martin Fowler – Monolith First](https://martinfowler.com/bliki/MonolithFirst.html?utm_source=chatgpt.com)
* [GitHub Engineering – Building GitHub with Ruby and Rails](https://github.blog/engineering/architecture-optimization/building-github-with-ruby-and-rails/?utm_source=chatgpt.com)
* [Shopify Engineering – The State of Shopify’s Monolith](https://shopify.engineering/shopify-monolith?utm_source=chatgpt.com)
* [Stack Overflow Architecture – Nick Craver](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/?utm_source=chatgpt.com)
* [Prime Video Tech Blog – Scaling up the A/V monitoring service](https://www.wudsn.com/productions/www/site/news/2023/2023-05-08-microservices-01.pdf?utm_source=chatgpt.com)
* [Dan Manges – The Modular Monolith](https://medium.com/%40dan_manges/the-modular-monolith-rails-architecture-fb1023826fc4?utm_source=chatgpt.com)
## Résumé
* Le monolithe, surtout en version modulaire, est souvent la meilleure approche pour démarrer rapidement, limiter la complexité opérationnelle et garder une cohérence transactionnelle.
* La migration vers des microservices doit être guidée par des besoins réels, pas par la mode architecturale.
