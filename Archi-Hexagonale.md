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

## 6) Ports & Adapters en pratique

### 6.1 Exemple minimal (pseudo-code orienté Java)

```java
// domain/port/in
public interface CreateOrderUseCase {
  OrderId handle(CreateOrderCommand command);
}

// domain/port/out
public interface OrderRepository {
  void save(Order order);
  Optional<Order> byId(OrderId id);
}

// application/service
public class CreateOrderService implements CreateOrderUseCase {
  private final OrderRepository repository;
  public CreateOrderService(OrderRepository repository) { this.repository = repository; }
  public OrderId handle(CreateOrderCommand cmd) {
    Order order = Order.create(cmd.customerId(), cmd.lines()); // règles métier
    repository.save(order);
    return order.id();
  }
}

// adapters/in/web (Spring)
@RestController
class OrderController {
  private final CreateOrderUseCase createOrder;
  @PostMapping("/orders")
  public ResponseEntity<CreateOrderResponse> create(@RequestBody CreateOrderRequest r) {
    OrderId id = createOrder.handle(r.toCommand());
    return ResponseEntity.status(CREATED).body(new CreateOrderResponse(id.value()));
  }
}

// adapters/out/persistence (JPA)
@Repository
class JpaOrderRepository implements OrderRepository {
  private final SpringDataOrderRepo jpa;
  public void save(Order order) { jpa.save(OrderEntity.from(order)); }
  public Optional<Order> byId(OrderId id) { return jpa.findById(id.value()).map(OrderEntity::toDomain); }
}
```

### 6.2 Mappers & DTOs

- **Request/Response DTOs** ne doivent **pas** entrer dans le domaine.
- Utiliser des **mappers explicites** pour conversion.
- Dans l’adapter persistance : **entities ORM ≠ domain entities**.

### 6.3 Gestion des erreurs

- Levée d’**exceptions métier** dans le domaine (ex. `InsufficientStock`).
- Traduction en **codes HTTP** dans l’adapter web (ex. 409 Conflict).
- Éviter de propager des exceptions techniques vers le domaine.

### 6.4 Transactions

- Encapsuler la **transaction** au niveau **application/use-case**.
- L’adapter persistance ne décide **pas** du découpage transactionnel.

### 6.5 Sécurité & validation

- **Validation de surface** (format, contraintes simples) côté adapter entrant.
- **Invariants métier** dans le domaine (toujours).
- Contrôles d’autorisations au niveau application (politiques, rôles).

## 7) Tests (stratégie recommandée)

1. **Unitaires domaine** : 100% sans infrastructure, rapides.
2. **Contract tests** sur ports sortants : le domaine suppose un contrat ; l’adapter doit le respecter.
3. **Intégration** : adapter + techno (JPA, HTTP, Kafka) dans des environnements éphémères (Testcontainers, WireMock).
4. **End-to-End** : parcours utilisateur via adapters entrants (API, UI) dans un environnement proche prod.

Conseils :

- Favoriser des **fakes/in-memory** pour `Repository` lors des tests d’application.
- Versionner des **fixtures** et **schemas** d’API/événements.

## 8) Domain-Driven Design (DDD) & Hexagonal

- L’architecture hexagonale **complète** le DDD :

  - **Domaine riche** : Entities, Value Objects, Domain Services, Domain Events.
  - **Bounded Contexts** : chaque contexte peut être un hexagone.
  - **ACL** entre contextes pour éviter la contamination des modèles.

## 9) CQRS et Événementiel

- **CQRS** : séparer **commande** (écriture) et **requête** (lecture). Chaque côté peut avoir ses ports/adapters.
- **Événements de domaine** : émis par le domaine ; adapters sortants peuvent les publier (Kafka, RabbitMQ).
- **Outbox pattern** : fiabilité entre transaction DB et publication d’événements.

## 10) Comparaisons rapides

- **Clean Architecture (Onion)** : philosophies proches (dépendances vers l’intérieur). Hexagonal met l’accent sur **les ports/adapters** et la **multiplicité d’IO**.
- **Layered (3-tier)** : souvent couplée (UI→Service→DAO) ; Hexagonal force l’abstraction par ports.

## 11) Bonnes pratiques

- Définir les **ports** dans le **domaine/application**.
- Limiter l’API des ports au **strict nécessaire** et **orientée métier**.
- Nommer clairement : `…UseCase`, `…Repository`, `…Service` (domaine vs app).
- **Aucun framework** dans `domain/` (pas d’annotations JPA, pas de Spring, pas d’HTTP).
- **Wiring DI** (Spring, NestJS, FastAPI) dans `config/` ou adapters.
- **Logique technique** (retries, timeouts, circuit breakers) dans adapters sortants.
- **Idempotence** et **réentrance** au niveau application.

## 12) Pièges fréquents (anti-patterns)

- Faire dépendre le domaine d’un **ORM** ou d’un **framework**.
- Laisser fuiter des **DTO/Entities ORM** dans le domaine.
- Ports trop génériques (ex. `CrudRepository<T>` côté domaine) → modèle appauvri.
- Logique métier dans les **controllers** ou **repositories**.
- Tests d’intégration lourds alors que les invariants sont unitaires.

## 13) Exemple plus complet (Node.js / NestJS style)

```ts
// domain/port/in/create-order.usecase.ts
export interface CreateOrderUseCase { handle(cmd: CreateOrderCommand): Promise<OrderId>; }

// domain/port/out/order-repository.ts
export interface OrderRepository {
  save(order: Order): Promise<void>;
  byId(id: OrderId): Promise<Order | null>;
}

// application/use-cases/create-order.service.ts
export class CreateOrderService implements CreateOrderUseCase {
  constructor(private readonly repo: OrderRepository) {}
  async handle(cmd: CreateOrderCommand): Promise<OrderId> {
    const order = Order.create(cmd.customerId, cmd.lines);
    await this.repo.save(order);
    return order.id;
  }
}

// adapters/in/http/order.controller.ts
@Post('/orders')
async create(@Body() req: CreateOrderRequest): Promise<CreateOrderResponse> {
  const id = await this.createOrder.handle(req.toCommand());
  return { id: id.value };
}

// adapters/out/persistence/order.repository.sql.ts
export class SqlOrderRepository implements OrderRepository {
  constructor(private readonly db: DataSource) {}
  async save(order: Order) { await this.db.query('INSERT …', map(order)); }
  async byId(id: OrderId) { const row = await this.db.query('SELECT …', [id.value]); return row ? toDomain(row) : null; }
}
```

## 14) Gestion des événements (Domain & Intégration)

- **Domain events** : objets immutables, synchrones dans le même process.
- **Integration events** : publient l’état vers d’autres systèmes (asynchrone, versionné).
- Utiliser un **publisher** comme port sortant (`EventPublisher`), avec adapter (Kafka, RabbitMQ, SNS/SQS).

## 15) Migration vers l’hexagonal (stratégie)

1. **Strangler Fig** : encapsuler l’existant derrière des ports, migrer progressivement.
2. **Découper par cas d’usage** : extraire un premier **use case** critique.
3. **Isoler les dépendances** : introduire adapters pour DB, API externes.
4. **Couverture de tests** avant refactor.
5. **Mesurer** : complexité, temps de build, MTTR, flakiness tests.

## 16) Observabilité & cross-cutting concerns

- **Logging** : contextualisé au use-case (corrélation `traceId`).
- **Metrics** : durée des use-cases, taux d’erreur par adapter.
- **Tracing** : spans par port/adapters (OpenTelemetry).
- **Feature flags** : au niveau application/adapters.

## 17) Sécurité (pratiques)

- **AuthN/AuthZ** à l’entrée (adapter entrant) ; **policies** au niveau application.
- **Secrets** : injectés via configuration, jamais dans le domaine.
- **Validation** des entrées (schema validation) **avant** d’appeler un use-case.

## 18) Checklist de revue

- [ ] Le domaine n’importe aucun framework.
- [ ] Les ports sont définis côté domaine/app.
- [ ] Les adapters n’exposent pas de types techniques au domaine.
- [ ] Transactions gérées au niveau application.
- [ ] Tests unitaires couvrent les invariants métier.
- [ ] Adapters sortants testés avec des contract tests.
- [ ] Mappings explicites et testés.
- [ ] Observabilité en place sur chaque adapter.

## 19) FAQ

**Q : L’hexagonal impose-t-elle des microservices ?**

- Non. Elle fonctionne en **monolithe modulaire** comme en microservices. Chaque contexte peut être un hexagone.

**Q : Où placer les validations ?**

- **Syntaxe/format** en adapter entrant ; **invariants** dans le domaine.

**Q : Puis-je utiliser un ORM dans le domaine ?**

- Non. L’ORM reste dans l’adapter persistance ; le domaine doit être **agnostique**.

**Q : Est-ce compatible avec GraphQL/REST/GRPC ?**

- Oui. Ce sont des **adapters entrants** interchangeables.

## 20) Références utiles (pour aller plus loin)

- Alistair Cockburn – "Hexagonal Architecture" (Ports & Adapters)
- Eric Evans – "Domain-Driven Design"
- Vernon – "Implementing Domain-Driven Design"
- Blogs/Conf talks sur Clean Architecture, Onion, CQRS, Event Sourcing

---

## 21) Modèle de gabarit (copier-coller)

```
src/
  domain/
    entities/
    value-objects/
    services/
    events/
    port/
      in/
      out/
  application/
    use-cases/
    mappers/
    policies/
    transactions/
  adapters/
    in/
      http/
      messaging/
      cli/
    out/
      persistence/
      http/
      cache/
      events/
  config/
    di/
    env/
```

---

**Conclusion** : En appliquant rigoureusement la séparation **domaine / ports / adapters**, vous obtenez un système **durable**, **testable** et **évolutif**. Adoptez une démarche incrémentale (Strangler), et sécurisez chaque frontière par des tests et des mappings explicites.
