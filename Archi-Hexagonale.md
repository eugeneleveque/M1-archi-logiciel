# Architecture Hexagonale (Ports & Adapters)

> **Objectif** : fournir une documentation claire, complète et actionnable pour concevoir, implémenter, tester et faire évoluer une application selon l’architecture hexagonale.

---

## 1) Vue d’ensemble

L’**architecture hexagonale** (aussi appelée **Ports & Adapters**), formalisée par **Alistair Cockburn**, vise à **isoler le cœur métier** (domaine) des détails techniques (base de données, Web, CLI, message broker, etc.).

- **Le domaine** contient les **règles métier** et doit être **indépendant** de tout framework, UI, DB, réseau.
- Les **ports** sont des **interfaces** qui décrivent **ce que** le domaine offre (ports sortants) ou **ce dont** il a besoin (ports entrants), sans exposer le **comment**.
- Les **adapters** implémentent ces ports pour se brancher à des technologies concrètes (HTTP, SQL/NoSQL, Kafka, Email, Filesystem, etc.).
- L’**inversion des dépendances** garantit que **les dépendances pointent vers le domaine**, jamais l’inverse.

**Bénéfices clés** : testabilité élevée, séparation des préoccupations, remplaçabilité des technologies, longévité du code métier.

## 2) Concepts et vocabulaire

- **Domaine** : modèles métier (Aggregates, Entities, Value Objects), règles, services de domaine.
- **Application (Cas d’usage)** : orchestration de scénarios métier, transactions, sécurité d’accès, idempotence. Ne contient **pas** de logique métier complexe (plutôt du **workflow**).
- **Port entrant** (Driving Port) : interface offerte par l’application/domaine pour déclencher un cas d’usage (ex. `CreateOrderUseCase`).
- **Port sortant** (Driven Port) : interface dont le domaine a besoin pour interagir avec l’extérieur (ex. `OrderRepository`, `PaymentGateway`).
- **Adapter entrant** : traduit des requêtes externes (REST/CLI/Message) en appels vers les ports entrants.
- **Adapter sortant** : implémente les ports sortants en s’appuyant sur une technologie (JPA, HTTP client, S3, Redis…).
- **Anticorruption Layer (ACL)** : protège le domaine d’un modèle externe (mapping/translation).
- **DTO** : objets de transfert à la frontière ; éviter qu’ils contaminent le domaine.
- **Mapper** : convertit DTO ⇄ Domain (et Domain ⇄ Persistence models).

## 3) Principes structurants

1. **Indépendance du domaine** : pas de dépendances vers des libs techniques.
2. **Dépendances dirigées vers l’intérieur** : adapters → ports → domain.
3. **Interfaces au centre** : les ports sont définis côté domaine/application.
4. **Remplaçabilité** : chaque adapter peut être substitué sans toucher au domaine.
5. **Test d’abord** : tests unitaires du domaine **sans** infrastructure ; contract tests sur ports ; tests d’intégration sur adapters ; tests end-to-end sur le système.

## 4) Schéma (ASCII)

```
            +------------------ Adapter entrant (HTTP/CLI/AMQP) ------------------+
            |                                                                     |
Client ---> |  Controller  ->  RequestMapper  ->  Port entrant (UseCase)          |
            |                                     |                               |
            +-------------------------------------|-------------------------------+
                                                  v
                                           Application Service
                                                  |
                                                  v
                                               Domaine
                                       (Entities / VOs / Rules)
                                                  |
                                                  v
                             Port sortant (Repository, ExternalService)
                                                  |
            +-------------------------------------|-------------------------------+
            |                                Adapter sortant                      |
            |   ORM/JPA/SQL, REST client, Filesystem, Kafka, Email, Cache, etc.  |
            +---------------------------------------------------------------------+
```

## 5) Organisation de projet (exemples)

### 5.1 Java (Gradle/Maven)

```
com.company.project
├─ domain
│  ├─ model/ (Entity, ValueObject)
│  ├─ service/ (Domain services)
│  ├─ port/
│  │  ├─ in/ (UseCase interfaces)
│  │  └─ out/ (Repository, External ports)
│  └─ event/ (Domain events)
├─ application
│  ├─ service/ (UseCase implementations)
│  └─ mapper/ (DTO↔Domain)
├─ adapters
│  ├─ in/
│  │  ├─ web/ (Controllers, request/response DTO)
│  │  └─ messaging/ (Consumers, handlers)
│  └─ out/
│     ├─ persistence/ (JPA repos, DAOs, entities)
│     ├─ http/ (clients)
│     └─ cache/ (Redis, etc.)
└─ config/ (DI, wiring)
```

### 5.2 Node.js (TypeScript)

```
src/
├─ domain/
│  ├─ entities/
│  ├─ value-objects/
│  ├─ services/
│  └─ ports/
│     ├─ in/
│     └─ out/
├─ application/
│  ├─ use-cases/
│  └─ mappers/
├─ adapters/
│  ├─ in/
│  │  ├─ http/
│  │  └─ messaging/
│  └─ out/
│     ├─ persistence/
│     ├─ http/
│     └─ cache/
└─ config/
```

### 5.3 Python

```
project/
├─ domain/
│  ├─ models/
│  ├─ services/
│  └─ ports/
├─ application/
│  ├─ use_cases/
│  └─ mappers/
├─ adapters/
│  ├─ in/
│  │  ├─ fastapi/
│  │  └─ cli/
│  └─ out/
│     ├─ sqlalchemy/
│     └─ http/
└─ config/
```

## 6) exemple connu et open-source : JHipster Lite.

- Le projet lui-même est codé en architecture hexagonale (et il génère des applis qui suivent la même approche). La doc et le dépôt officiel indiquent explicitement que le code est structuré en domaine / cas d’usage (ports) / adaptateurs, isolant les détails techniques du cœur métier.

- En pratique, dans JHipster Lite :
  le domaine expose des ports (use cases) indépendants de toute techno, des adapters entrants (ex. REST/UI) pilotent ces cas d’usage,
  des adapters sortants gèrent la persistance, la conf, etc., et sont remplaçables sans toucher au domaine. (Voir leur doc “the generated code uses Hexagonal Architecture”.)
