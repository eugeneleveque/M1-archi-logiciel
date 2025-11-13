## Architecture Hexagonale (Ports & Adapters)

### 1. Définition simple

L’architecture hexagonale (ou **Ports & Adapters**) est un style d’architecture qui organise l’application autour de son **cœur métier**, totalement indépendant de la base de données, de l’UI et des frameworks techniques.

Toutes les interactions avec l’extérieur passent par des **ports** (contrats abstraits) que des **adaptateurs** implémentent pour parler à des technologies concrètes (HTTP, DB, message broker, etc.).  
Résultat : un code métier **facile à tester**, **stable dans le temps** et des technologies **faciles à remplacer**.

---

### 2. Liste de caractéristiques

- **Séparation stricte métier / technique**

  - Le domaine ne dépend d’aucune librairie technique (framework web, ORM, driver, etc.).
  - Le code métier reste “pur” : pas d’annotations de framework, pas de logique d’I/O.

- **Cœur de l’application = domaine**

  - Contient : Entités, Value Objects, règles métier, services de domaine.
  - Encapsule les invariants métier et la logique de décision.

- **Ports & Adapters**

  - **Ports entrants (driving ports)** : contrats pour déclencher un cas d’usage (ex. `CreateOrderUseCase`).
  - **Ports sortants (driven ports)** : contrats dont le domaine / l’application ont besoin (ex. `OrderRepository`, `PaymentGateway`).
  - **Adaptateurs entrants (primary adapters)** : HTTP controllers, CLI, UI, tests automatisés… ils appellent les ports entrants.
  - **Adaptateurs sortants (secondary adapters)** : implémentations techniques des ports sortants (ORM, client HTTP, message broker, cache, etc.).

- **Inversion des dépendances**

  - Les dépendances pointent **vers l’intérieur** :  
    `adapters → ports → application/domaine`.
  - Le domaine ne connaît ni les adaptateurs, ni les frameworks.

- **Ports = contrats, pas juste des interfaces**

  - Souvent modélisés par des `interface` (Java/TypeScript).
  - Plus généralement : ce sont des **contrats/protocoles** (signature de méthodes, schémas de messages, etc.).

- **Couches Domain / Application**

  - **Domaine** : règles métier, invariants, modèles riches (Entities/VO), événements de domaine.
  - **Application** : orchestration de cas d’usage, gestion de transactions, authZ (droits par cas d’usage), appels aux ports sortants.

- **Tests comme adaptateurs entrants**

  - Les tests automatisés pilotent l’application via les ports entrants.
  - Facile de tester le domaine sans base de données ni serveur HTTP.

- **Remplaçabilité des technologies**

  - On peut changer :
    - de DB (PostgreSQL → MongoDB),
    - de protocole (REST → gRPC),
    - de service externe (Stripe → autre PSP),
  - en modifiant l’adapter sortant correspondant, sans toucher au domaine.

- **Compatible DDD / Bounded Contexts**

  - Souvent utilisé pour structurer un **bounded context** avec un domaine riche.
  - Les échanges entre contexts ou systèmes externes se font via ports + adaptateurs, parfois avec **Anticorruption Layer (ACL)**.

- **Composition root / câblage**
  - Un endroit dédié (souvent `config/`) où l’on :
    - crée les implémentations concrètes des ports,
    - câble : `adapter entrant → port entrant → service d’application → port sortant → adapter sortant`,
    - branche les frameworks (Spring, NestJS, FastAPI, etc.).

---

### 3. Exemple d’implémentation (schéma)

#### Schéma conceptuel (ASCII)

```text
                 +----------- Adaptateurs entrants (HTTP/CLI/Tests) -----------+
                 |                                                              |
Client / Test -->|  Controller  -> RequestMapper -> Port entrant (UseCase)      |
                 |                                 |                            |
                 +---------------------------------|----------------------------+
                                                   v
                                          Application Service
                                                   |
                                           (orchestration,
                                            transactions,
                                            sécurité cas d’usage)
                                                   |
                                                   v
                                                Domaine
                                      (Entities / VOs / Rules)
                                                   |
                                                   v
                                Port sortant (Repository, ExternalService)
                                                   |
                 +---------------------------------|----------------------------+
                 |                       Adaptateurs sortants                   |
                 |   ORM/JPA/SQL, REST client, Message broker, Cache, Email...  |
                 +----------------------------------------------------------------

---

### 4. Exemples d’utilisation dans des projets connus

#### JHipster Lite

Générateur de projets Java (Spring Boot) utilisant explicitement l’architecture hexagonale.

Structure le code en :

- `domain` (métier)
- `application` / use cases
- `infrastructure` / adapters

Les adaptateurs (REST, persistance, etc.) sont remplaçables sans impacter le domaine.

---

#### Templates Spring Boot “Hexagonal / Clean Architecture” (projets GitHub)

De nombreux starters structurent un microservice comme :

- `domain/`, `application/` (use cases, ports)
- `infrastructure/` (adapters sortants)
- `web/` ou `interface/` (adapters entrants REST)

Utilisé pour des contextes variés : e-commerce, gestion de commandes, systèmes de réservation, etc.

---

#### Applications métiers avec DDD

Dans des projets DDD (systèmes de facturation, banking, e-commerce, logistique…),
l’architecture hexagonale est souvent utilisée pour :

- isoler le modèle métier,
- découpler la persistance,
- connecter des services externes (paiement, email, ERP) via des ports sortants.

---

### 5. Sources (liens web)

- Alistair Cockburn – “Hexagonal Architecture (Ports & Adapters)”
  <https://alistair.cockburn.us/hexagonal-architecture>

- Wikipédia – “Hexagonal architecture (software)”
  <https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)>

- AWS Prescriptive Guidance – “Hexagonal architecture pattern”
  <https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/hexagonal-architecture.html>

- Octo Technology (FR) – “Architecture Hexagonale : trois principes et un exemple d’implémentation”
  <https://blog.octo.com/architecture-hexagonale-trois-principes-et-un-exemple-dimplementation>

- Scalastic – “Everything You Need to Know About Hexagonal Architecture”
  <https://scalastic.io/en/hexagonal-architecture/>

- JHipster Lite – Site officiel
  <https://www.jhipster.tech/jhipster-lite/>

- JHipster Lite – Dépôt GitHub
  <https://github.com/jhipster/jhipster-lite/>
```
