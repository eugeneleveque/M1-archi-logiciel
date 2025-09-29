# Architecture Logicielle Event-Driven (EDA)

> **Objectif** : fournir une documentation claire, complète et actionnable pour concevoir, implémenter, tester et faire évoluer un système **piloté par les événements** (Event-Driven Architecture, EDA).

---

## 1) Vue d’ensemble

L’**architecture Event-Driven** repose sur l’**émission**, la **diffusion** et le **traitement** d’**événements**. Au lieu d’appels synchrones et fortement couplés, les composants **publient** et **consomment** des messages via un **bus** ou un **broker** (Kafka, RabbitMQ, NATS, Pulsar, SNS/SQS, etc.).

**Bénéfices** : découplage temporel et technologique, évolutivité, résilience, extensibilité, intégration de systèmes hétérogènes.

**Contreparties** : complexité de la cohérence (éventuelle), débogage/tracing plus difficiles, gouvernance de schémas d’événements, gestion d’ordonnancement et d’idempotence.

## 2) Concepts et vocabulaire

- **Événement** : fait passé, immuable, descriptif (ex. `OrderPlaced`, `PaymentAuthorized`).
- **Producteur (Publisher)** : service qui **publie** un événement.
- **Consommateur (Subscriber/Consumer)** : service qui **réagit** à un événement.
- **Topic/Exchange/Queue** : canal de transport d’événements (pub-sub, fanout, direct, etc.).
- **Stream** : journal ordonné d’événements (ex. Kafka topic partitionné).
- **Clé de partition** : détermine l’ordre local et la distribution.
- **Contrat d’événement** : schéma (Avro/JSON/Protobuf) + sémantique + versioning.
- **Chorégraphie** : les services réagissent à des événements l’un de l’autre, sans orchestrateur central.
- **Orchestration** : un orchestrateur gère explicitement le flot (ex. Saga orchestrée).
- **Cohérence éventuelle** : l’état est convergent, non instantanément consistant.
- **Outbox/Inbox** : patrons pour fiabiliser la publication et la consommation.

## 3) Styles d’EDA

1. **Pub/Sub classique** : producteurs publient, multiples consommateurs réagissent.
2. **Event Streaming** : traitement d’un flux ordonné (Kafka, Pulsar) avec relecture, state stores.
3. **Event Sourcing** : source de vérité = **journal d’événements** (rebuild de l’état par projection).
4. **CQRS + Events** : séparation commandes/lectures, projections asynchrones.

## 4) Schéma (ASCII)

```
            +--------------------- Broker / Bus d’événements ---------------------+
            |             (Kafka / RabbitMQ / NATS / Pulsar / SNS/SQS)            |
            +---------------------------------------------------------------------+
               ^                        ^                          ^
               |                        |                          |
         [Producteur]             [Consommateur]             [Consommateur]
           Service A                 Service B                   Service C
         (publie E1)                (réagit E1)                (réagit E1)
               |                        |                          |
          Domaine A                Domaine B                   Domaine C
```

## 5) Design des événements

- **Nommer** au **passé** : `OrderPlaced`, `PaymentCaptured`.
- **Immutables** : pas de mutation après publication.
- **Auto-suffisants** : inclure les données nécessaires aux consommateurs (sans surcharger).
- **Contrats stables** : schémas versionnés (compatibilité **backward** privilégiée).
- **Sémantique claire** : qui publie, quand, pourquoi ; différence **commandes** vs **événements**.
- **Métadonnées** : `eventId`, `eventType`, `occurredAt`, `producer`, `traceId`, `version`.

## 6) Garanties & livraison

- **At-most-once** : jamais en double mais risque de perte.
- **At-least-once** (le plus courant) : possible **doublons** → **idempotence** requise.
- **Exactly-once** (rare/complexe) : dépend du broker et du design applicatif.
- **Ordre** : garanti **par partition** uniquement (Kafka) → choisir une **clé** cohérente (ex. `orderId`).

## 7) Idempotence & déduplication

- Stocker un **`eventId`** traité (Inbox) pour ignorer les doublons.
- Utiliser des **opérations idempotentes** (UPSERT, set-based, `PUT` au lieu de `POST`).
- **Transactions locales** + **Outbox** pour découpler écriture DB et publication.

## 8) Patterns essentiels

- **Outbox** : écrire événement dans la même **transaction** que l’état local, puis publier asynchrone.
- **Inbox** : journal des événements **consommés** pour déduplication et relecture.
- **Saga** (chorégraphie ou orchestration) : gérer les transactions distribuées avec **compensations**.
- **Projections** : vues matérialisées pour la lecture (CQRS).
- **Dead Letter Queue (DLQ)** : mettre à l’écart les messages non traitables.
- **Retry/Backoff/Circuit Breaker** : robustesse face aux pannes.

## 9) Organisation de projet (exemples)

### 9.1 Node.js (TypeScript)

```
src/
  domain/
  application/
  adapters/
    in/
      http/
    out/
      events/ (publisher, schemas)
      persistence/
  infra/
    broker/ (kafka-client, rabbitmq-client)
    http/
  config/
```

### 9.2 Java (Spring Boot)

```
com.company.project
├─ domain/
├─ application/
├─ adapters/
│  ├─ in/http/
│  └─ out/
│     ├─ events/ (KafkaTemplate, Channels)
│     └─ persistence/
└─ infra/
   ├─ broker/
   └─ config/
```

### 9.3 Python (FastAPI + Faust/Kafka)

```
project/
├─ domain/
├─ application/
├─ adapters/
│  ├─ out/events/
│  └─ out/persistence/
├─ workers/ (consumers, stream processors)
└─ config/
```

## 10) Exemple Netflix

- Netflix est un très bon exemple d’organisation qui utilise largement l’architecture event-driven. Leur Keystone data pipeline sert d’infrastructure unifiée de publication, collecte et routage d’événements (avec Kafka) pour l’analytique temps réel et le streaming, et alimente de nombreux cas d’usage (observabilité, personnalisation, pub, etc.).

- Concrètement, on retrouve chez Netflix :

  - des flux d’événements publiés/consommés via Kafka au cœur de Keystone, utilisés pour le traitement temps réel et le batch,
  - des plateformes annexes event-driven comme Delta (synchronisation et enrichissement de données à cohérence éventuelle),
  - des pipelines orientés “data in motion” (Keystone, Mantis, Spark/Flink) pour la surveillance et l’efficacité coût/perf, et même le tracking d’événements pour de nouvelles features (ex. lancement des ads).
