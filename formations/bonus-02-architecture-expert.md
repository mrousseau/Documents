# 🏆 BONUS 2 — Architecture Logicielle : Niveau Expert

> **Prérequis :** Avoir lu la formation principale (Partie 4 — Architecture).
> **Objectif :** Maîtriser les architectures modernes, les patterns distribués, le DDD, et les principes de scalabilité.

---

# 📋 Sommaire

- [1. Domain-Driven Design (DDD)](#1-domain-driven-design-ddd)
  - [1.1 Concepts Stratégiques](#11-concepts-stratégiques)
  - [1.2 Concepts Tactiques](#12-concepts-tactiques)
  - [1.3 Exemple Complet DDD](#13-exemple-complet-ddd)
- [2. Event-Driven Architecture (EDA)](#2-event-driven-architecture-eda)
  - [2.1 Concepts Fondamentaux](#21-concepts-fondamentaux)
  - [2.2 Messaging Patterns](#22-messaging-patterns)
  - [2.3 Event Storming](#23-event-storming)
- [3. Architecture Distribuée](#3-architecture-distribuée)
  - [3.1 API Gateway](#31-api-gateway)
  - [3.2 Service Mesh](#32-service-mesh)
  - [3.3 Saga Pattern](#33-saga-pattern)
  - [3.4 Théorème CAP](#34-théorème-cap)
  - [3.5 Patterns de Base de Données Distribuée](#35-patterns-de-base-de-données-distribuée)
- [4. Scalabilité et Performance](#4-scalabilité-et-performance)
  - [4.1 Scaling Horizontal vs Vertical](#41-scaling-horizontal-vs-vertical)
  - [4.2 Caching Strategies](#42-caching-strategies)
  - [4.3 Load Balancing](#43-load-balancing)
  - [4.4 Database Sharding](#44-database-sharding)
- [5. Architecture Cloud-Native](#5-architecture-cloud-native)
  - [5.1 Les 12 Factors](#51-les-12-factors)
  - [5.2 Serverless](#52-serverless)
  - [5.3 Containers & Orchestration](#53-containers--orchestration)
- [6. Sécurité Architecturale](#6-sécurité-architecturale)
  - [6.1 Zero Trust](#61-zero-trust)
  - [6.2 OAuth2 / OpenID Connect](#62-oauth2--openid-connect)
  - [6.3 Principes de Sécurité](#63-principes-de-sécurité)
- [7. Observabilité](#7-observabilité)
- [8. Architecture Decision Records (ADR)](#8-architecture-decision-records-adr)
- [9. Choisir son Architecture — Matrice de Décision](#9-choisir-son-architecture--matrice-de-décision)

---

# 1. Domain-Driven Design (DDD)

Le DDD est une **philosophie de conception** formulée par Eric Evans (2003). L'idée centrale : le code doit refléter fidèlement le **domaine métier**. La structure du logiciel doit être dictée par les experts métier, pas par les contraintes techniques.

## 1.1 Concepts Stratégiques

Ce sont les concepts de **haut niveau** qui structurent l'architecture globale.

### Ubiquitous Language (Langage Ubiquitaire)

Un vocabulaire commun entre développeurs et experts métier, utilisé partout : dans le code, les tests, la documentation, les conversations.

```
❌ Sans Ubiquitous Language :
  Dev : "Le UserEntity a un foreignKey vers OrderTable via le OrderService"
  Métier : "... quoi ?"

✅ Avec Ubiquitous Language :
  Dev : "Un Client passe une Commande qui contient des LignesDeCommande"
  Métier : "Oui, exactement !"
  → Et le code utilise ces mêmes termes : class Client, class Commande, class LigneDeCommande
```

### Bounded Context (Contexte Borné)

Un périmètre clair où un modèle de domaine est **cohérent et valide**. Un même concept peut avoir des significations différentes dans des contextes différents.

```
Exemple : le concept "Produit"

Contexte CATALOGUE :
  Produit = { nom, description, images, catégorie, spécifications }
  → Objectif : présenter le produit au client

Contexte INVENTAIRE :
  Produit = { sku, quantité_stock, emplacement_entrepôt, fournisseur }
  → Objectif : gérer le stock

Contexte FACTURATION :
  Produit = { référence, prix_ht, tva, remise_applicable }
  → Objectif : calculer les factures

→ MÊME mot "Produit", TROIS modèles différents.
→ Chaque contexte a SON modèle, SES règles, SA base de données.
→ Forcer un seul modèle "Produit" pour tout = catastrophe.
```

### Context Map (Carte des Contextes)

Comment les Bounded Contexts communiquent entre eux.

```
┌──────────────┐      Shared Kernel      ┌──────────────┐
│  Catalogue   │◄───────────────────────►│  Inventaire  │
└──────┬───────┘                         └──────┬───────┘
       │                                        │
       │  Upstream/Downstream                   │ Events
       │  (Customer-Supplier)                   │
       ▼                                        ▼
┌──────────────┐                         ┌──────────────┐
│ Facturation  │                         │  Livraison   │
└──────────────┘                         └──────────────┘
```

| Relation | Description |
|---|---|
| **Shared Kernel** | Deux contextes partagent un sous-ensemble de modèle |
| **Customer-Supplier** | Un contexte (supplier) fournit des données à un autre (customer) |
| **Conformist** | Un contexte se conforme au modèle d'un autre (pas le choix) |
| **Anti-Corruption Layer (ACL)** | Couche de traduction pour se protéger d'un modèle externe |
| **Published Language** | Interface publique documentée (API, événements) |

---

## 1.2 Concepts Tactiques

Ce sont les **patterns de code** utilisés à l'intérieur d'un Bounded Context.

### Entity (Entité)

Objet défini par son **identité**, pas par ses attributs. Deux utilisateurs avec le même nom sont des entités différentes.

```csharp
public class User
{
    public Guid Id { get; }  // ← identité
    public string Email { get; private set; }
    public string Name { get; private set; }

    public User(string email, string name)
    {
        Id = Guid.NewGuid();
        Email = email;
        Name = name;
    }

    // Égalité basée sur l'identité, pas les attributs
    public override bool Equals(object? obj) => obj is User u && u.Id == Id;
    public override int GetHashCode() => Id.GetHashCode();
}
```

### Value Object (Objet Valeur)

Objet défini par ses **attributs**, pas par son identité. Deux adresses identiques sont le même objet valeur. Immuable.

```csharp
public record Address(string Rue, string Ville, string CodePostal, string Pays)
{
    // record en C# = immuable + égalité par valeur par défaut
    // Deux Address("10 rue X", "Paris", ...) sont égaux
}

public record Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Devises différentes");
        return new Money(Amount + other.Amount, Currency);
    }
}
```

### Aggregate (Agrégat)

Un **cluster d'entités et d'objets valeur** traités comme une unité de cohérence. L'Aggregate Root est le point d'entrée — tout accès passe par lui.

```csharp
// Commande est l'Aggregate Root
public class Commande  // ← Aggregate Root
{
    public Guid Id { get; }
    public Guid ClientId { get; }
    private readonly List<LigneCommande> _lignes = new();  // ← entités enfants
    public IReadOnlyList<LigneCommande> Lignes => _lignes;
    public EtatCommande Etat { get; private set; }

    public Commande(Guid clientId)
    {
        Id = Guid.NewGuid();
        ClientId = clientId;
        Etat = EtatCommande.Brouillon;
    }

    // ✅ Toute modification passe par l'Aggregate Root
    public void AjouterLigne(Guid produitId, int quantite, Money prixUnitaire)
    {
        if (Etat != EtatCommande.Brouillon)
            throw new DomainException("Impossible de modifier une commande validée");

        var ligne = new LigneCommande(produitId, quantite, prixUnitaire);
        _lignes.Add(ligne);
    }

    public Money Total => _lignes.Aggregate(
        new Money(0, "EUR"),
        (acc, ligne) => acc.Add(ligne.SousTotal)
    );

    public void Valider()
    {
        if (!_lignes.Any())
            throw new DomainException("Commande vide");
        Etat = EtatCommande.Validee;
    }
}

// ❌ On ne modifie JAMAIS une LigneCommande directement depuis l'extérieur
// ✅ On passe toujours par Commande (l'Aggregate Root)
```

### Domain Event (Événement de Domaine)

Quelque chose d'**important qui s'est passé** dans le domaine. Temps passé, factuel, immuable.

```csharp
public record CommandeValidee(
    Guid CommandeId,
    Guid ClientId,
    Money Total,
    DateTime Timestamp
);

// Quand on valide une commande, on émet un événement
public class Commande
{
    private readonly List<IDomainEvent> _events = new();

    public void Valider()
    {
        // ... validation ...
        Etat = EtatCommande.Validee;
        _events.Add(new CommandeValidee(Id, ClientId, Total, DateTime.UtcNow));
    }

    public IReadOnlyList<IDomainEvent> Events => _events;
}
```

### Domain Service

Logique métier qui ne **appartient à aucune entité** en particulier. Exemple : le calcul d'un prix impliquant un produit, un client ET une promotion.

```csharp
public class PriceCalculationService
{
    private readonly IPromotionRepository _promos;

    public Money CalculerPrix(Produit produit, Client client, int quantite)
    {
        var prixBase = produit.Prix.Multiply(quantite);
        var promotion = _promos.GetActivePromotion(produit.Id, client.Segment);
        return promotion?.Appliquer(prixBase) ?? prixBase;
    }
}
```

---

## 1.3 Exemple Complet DDD

### Structure de projet
```
src/
├── Domain/                          # Couche domaine (0 dépendance)
│   ├── Commandes/                   # Aggregate Commande
│   │   ├── Commande.cs             # Aggregate Root
│   │   ├── LigneCommande.cs        # Entity enfant
│   │   ├── EtatCommande.cs         # Enum / Value Object
│   │   ├── ICommandeRepository.cs  # Port
│   │   └── Events/
│   │       ├── CommandeValidee.cs
│   │       └── CommandeAnnulee.cs
│   ├── Clients/                     # Aggregate Client
│   │   ├── Client.cs
│   │   └── Address.cs              # Value Object
│   └── SharedKernel/                # Objets partagés entre agrégats
│       ├── Money.cs
│       └── IDomainEvent.cs
├── Application/                     # Use Cases
│   ├── Commands/
│   │   ├── ValiderCommandeHandler.cs
│   │   └── AjouterLigneHandler.cs
│   └── Queries/
│       └── GetCommandeHandler.cs
├── Infrastructure/                  # Implémentations
│   ├── Persistence/
│   │   └── EfCommandeRepository.cs
│   └── Messaging/
│       └── RabbitMqEventPublisher.cs
└── Api/                             # Présentation
    └── Controllers/
        └── CommandeController.cs
```

---

# 2. Event-Driven Architecture (EDA)

## 2.1 Concepts Fondamentaux

L'EDA est une architecture où les composants communiquent via des **événements** plutôt que des appels directs. C'est le découplage ultime : le producteur d'événements ne connaît même pas ses consommateurs.

```
Architecture Synchrone (classique) :
  Service A ──HTTP──→ Service B ──HTTP──→ Service C
  → Couplage fort, cascade de pannes, latence cumulée

Architecture Event-Driven :
  Service A ──publie──→ [Event Bus] ──notifie──→ Service B
                                    ──notifie──→ Service C
                                    ──notifie──→ Service D (ajouté après, ZERO modification)
  → Découplage total, résilience, extensibilité
```

### Les 3 types d'événements

| Type | Description | Exemple |
|---|---|---|
| **Domain Event** | Fait métier important | `CommandeValidée`, `PaiementReçu` |
| **Integration Event** | Communication entre Bounded Contexts | `StockMisÀJour` → envoyé au contexte Livraison |
| **Notification Event** | Signal léger (souvent sans données) | `NouvelleCommandeDisponible` → le consommateur va chercher les détails |

---

## 2.2 Messaging Patterns

### Pub/Sub (Publish/Subscribe)

Un producteur publie, N consommateurs reçoivent. Le producteur ne sait pas qui consomme.

```
Producteur → [Topic: commande.validée]
                    ├──→ Service Facturation (crée la facture)
                    ├──→ Service Stock (réserve les produits)
                    └──→ Service Notification (envoie l'email)
```

### Event Queue (Point-to-Point)

Un message est consommé par **un seul** consommateur (le premier disponible). Pour la répartition de charge.

```
Producteurs → [Queue: tâches_export]
                    ├──→ Worker 1 (prend la tâche A)
                    ├──→ Worker 2 (prend la tâche B)
                    └──→ Worker 3 (prend la tâche C)
```

### Technologies courantes

| Technologie | Type | Usage principal |
|---|---|---|
| **RabbitMQ** | Message broker | Queues, routing flexible, RPC |
| **Apache Kafka** | Event streaming | Haut débit, replay, event sourcing |
| **AWS SQS/SNS** | Cloud managed | Queues (SQS) + Pub/Sub (SNS) |
| **Redis Streams** | In-memory | Faible latence, cache + events |

---

## 2.3 Event Storming

L'Event Storming est un **atelier collaboratif** (devs + métier) pour modéliser un système par ses événements.

```
1. 🟧 Domain Events (orange)     — "Que s'est-il passé ?"
   CommandePassée, PaiementReçu, ProduitExpédié

2. 🟦 Commands (bleu)            — "Qui a déclenché ça ?"
   PasserCommande, EffectuerPaiement

3. 🟨 Actors (jaune)             — "Qui fait l'action ?"
   Client, Admin, Système automatique

4. 🟩 Aggregates (vert)          — "Quel cluster gère ça ?"
   Commande, Paiement, Stock

5. 🟪 Policies (violet)          — "Quand X arrive, alors Y"
   Quand PaiementReçu → LancerPréparation

→ Résultat : une carte complète du domaine, compréhensible par TOUT le monde
```

---

# 3. Architecture Distribuée

## 3.1 API Gateway

Point d'entrée **unique** pour tous les clients. Gère le routing, l'authentification, le rate limiting, l'agrégation.

```
                           ┌─────────────┐
Mobile ────────────────→   │             │  ──→ Service Users
Web    ────────────────→   │ API Gateway │  ──→ Service Orders
Partenaire ────────────→   │             │  ──→ Service Products
                           └─────────────┘

Le Gateway gère :
✅ Routing (route vers le bon service)
✅ Authentication (vérifie le JWT une seule fois)
✅ Rate Limiting (protège les services)
✅ Agrégation (combine les réponses de plusieurs services)
✅ Transformation (adapte le format selon le client)
✅ Cache (évite les appels inutiles)
```

Technologies : Kong, AWS API Gateway, Nginx, Envoy, Ocelot (.NET).

---

## 3.2 Service Mesh

Couche d'infrastructure qui gère la communication **service-à-service** de manière transparente (pas de code à écrire dans les services).

```
┌──────────────────┐        ┌──────────────────┐
│ Service A        │        │ Service B        │
│  ┌────────────┐  │  TLS   │  ┌────────────┐  │
│  │ App Code   │  │◄──────►│  │ App Code   │  │
│  └────────────┘  │        │  └────────────┘  │
│  ┌────────────┐  │        │  ┌────────────┐  │
│  │ Sidecar    │  │        │  │ Sidecar    │  │
│  │ Proxy      │  │        │  │ Proxy      │  │
│  └────────────┘  │        │  └────────────┘  │
└──────────────────┘        └──────────────────┘

Le sidecar proxy gère (SANS modifier le code) :
✅ mTLS (chiffrement service-à-service)
✅ Load balancing intelligent
✅ Circuit breaking automatique
✅ Retry avec backoff
✅ Métriques et tracing distribué
✅ Canary deployments
```

Technologies : Istio, Linkerd, Consul Connect.

---

## 3.3 Saga Pattern

Comment gérer une **transaction distribuée** sur plusieurs services ? On ne peut pas faire de `BEGIN TRANSACTION` sur 3 bases de données différentes. La Saga coordonne une séquence d'actions locales avec des **compensations** en cas d'échec.

### Saga Chorégraphique (événements)

Chaque service écoute les événements et réagit. Pas de coordinateur central.

```
1. Service Commande → publie "CommandeCreée"
2. Service Paiement → écoute → tente le paiement
   ├── succès → publie "PaiementEffectué"
   └── échec  → publie "PaiementÉchoué"
3. Service Stock → écoute "PaiementEffectué" → réserve le stock
   ├── succès → publie "StockRéservé"
   └── échec  → publie "StockInsuffisant"
       → Service Paiement écoute → REMBOURSE (compensation)
       → Service Commande écoute → ANNULE (compensation)
```

### Saga Orchestrée (coordinateur)

Un orchestrateur central dirige la séquence.

```csharp
public class CommandeSagaOrchestrator
{
    public async Task<SagaResult> Executer(NouvelleCommande cmd)
    {
        // Étape 1 : Paiement
        var paiement = await _paiementService.Debiter(cmd.ClientId, cmd.Total);
        if (!paiement.Success)
            return SagaResult.Failed("Paiement refusé");

        // Étape 2 : Stock
        var stock = await _stockService.Reserver(cmd.Produits);
        if (!stock.Success)
        {
            // COMPENSATION : rembourser le paiement
            await _paiementService.Rembourser(paiement.TransactionId);
            return SagaResult.Failed("Stock insuffisant");
        }

        // Étape 3 : Création commande
        var commande = await _commandeService.Creer(cmd);
        if (!commande.Success)
        {
            // COMPENSATION : libérer le stock + rembourser
            await _stockService.Liberer(stock.ReservationId);
            await _paiementService.Rembourser(paiement.TransactionId);
            return SagaResult.Failed("Erreur création commande");
        }

        return SagaResult.Completed(commande.Id);
    }
}
```

| | Chorégraphique | Orchestrée |
|---|---|---|
| Coordinateur | Aucun (événements) | Un orchestrateur central |
| Couplage | Très faible | Modéré |
| Visibilité | Flux difficile à suivre | Flux clair et centralisé |
| Complexité | Augmente avec le nombre de services | Reste gérable |
| Bon pour | 3-5 étapes simples | Flux complexes, avec conditions |

---

## 3.4 Théorème CAP

Dans un système distribué, tu ne peux garantir **que 2 des 3** propriétés simultanément :

```
       Consistency (C)
       /           \
      /             \
     CP              CA
    /                 \
Partition          Availability
Tolerance (P) ─────── (A)
                 AP
```

| Propriété | Signification |
|---|---|
| **Consistency** | Tous les nœuds voient les mêmes données au même moment |
| **Availability** | Chaque requête reçoit une réponse (même si les données sont stales) |
| **Partition Tolerance** | Le système fonctionne même si des nœuds sont isolés |

**En pratique :** La partition réseau est inévitable, donc le vrai choix est entre **CP** (cohérence, au prix de la disponibilité) et **AP** (disponibilité, au prix de la cohérence temporaire).

| Choix | Exemples | Cas d'usage |
|---|---|---|
| **CP** | MongoDB (par défaut), HBase, Redis (cluster) | Transactions financières, inventaire |
| **AP** | Cassandra, DynamoDB, CouchDB | Réseaux sociaux, IoT, analytics |
| **CA** | PostgreSQL (single node) | N'existe pas en distribué (partition inévitable) |

---

## 3.5 Patterns de Base de Données Distribuée

### Database per Service

Chaque microservice a sa **propre base de données**. Pas de BDD partagée.

```
✅ Avantage : indépendance, choix techno libre, pas de couplage
❌ Défi : cohérence entre services (résolu par Saga, Event Sourcing)
```

### CQRS + Read Replicas

Séparer les écritures (une BDD master) et les lectures (plusieurs replicas optimisées).

```
Écriture → PostgreSQL Master → Synchronisation → Read Replica 1
                                               → Read Replica 2
                                               → Elasticsearch (recherche)
Lecture ← n'importe quel replica (rapide, scalable)
```

### Event Sourcing + Projections

Stocker les événements, projeter en vues matérialisées pour la lecture.

```
Événements → Event Store (Kafka, EventStoreDB)
                ↓ Projection
            → Vue "Commandes en cours" (pour le dashboard)
            → Vue "Statistiques ventes" (pour les analytics)
            → Vue "Recherche" (Elasticsearch)
```

---

# 4. Scalabilité et Performance

## 4.1 Scaling Horizontal vs Vertical

```
VERTICAL (Scale Up) :
  Ajouter plus de RAM/CPU à une seule machine.
  ✅ Simple
  ❌ Limite physique, single point of failure

HORIZONTAL (Scale Out) :
  Ajouter plus de machines identiques.
  ✅ Pas de limite théorique, résilience
  ❌ Complexité (state, sessions, cohérence)
```

### Règle : Design for horizontal scaling

Pour scaler horizontalement, tes services doivent être **stateless** (pas d'état en mémoire locale). L'état va dans une base de données ou un cache partagé.

---

## 4.2 Caching Strategies

| Stratégie | Fonctionnement | Cas d'usage |
|---|---|---|
| **Cache-Aside** | L'app vérifie le cache avant la BDD | Lectures fréquentes, données rarement modifiées |
| **Write-Through** | L'écriture va dans le cache ET la BDD | Cohérence forte |
| **Write-Behind** | L'écriture va dans le cache, BDD mise à jour en asynchrone | Performance max, risque de perte |
| **Read-Through** | Le cache charge depuis la BDD automatiquement si absent | Simplification du code |

```
Cache-Aside (le plus courant) :

1. App demande les données
2. Vérifie Redis (cache)
   ├── ✅ Cache hit → retourne directement (rapide)
   └── ❌ Cache miss → requête BDD → stocke en cache → retourne

Invalidation : TTL (expiration automatique) ou invalidation explicite à l'écriture
```

Technologies : Redis, Memcached, Varnish (HTTP), CDN (Cloudflare, CloudFront).

---

## 4.3 Load Balancing

```
                    ┌──────────────┐
Clients ──────────→ │ Load Balancer│
                    └──────┬───────┘
                   ┌───────┼───────┐
                   ▼       ▼       ▼
               Instance  Instance  Instance
                  1         2         3
```

| Algorithme | Principe | Quand ? |
|---|---|---|
| **Round Robin** | Chacun son tour | Instances identiques |
| **Least Connections** | Vers celle qui a le moins de connexions actives | Charges hétérogènes |
| **IP Hash** | Même client → même instance | Sessions sticky |
| **Weighted** | Plus de trafic vers les instances plus puissantes | Instances hétérogènes |

Technologies : Nginx, HAProxy, AWS ALB/NLB, Kubernetes Ingress.

---

## 4.4 Database Sharding

Diviser les données sur **plusieurs bases** pour dépasser les limites d'une seule machine.

```
Clé de sharding : user_id

Shard 1 : users 1-1000
Shard 2 : users 1001-2000
Shard 3 : users 2001-3000

Requête "GET user 1500" → Router → Shard 2
```

| Stratégie | Principe |
|---|---|
| **Range-based** | Par plage (1-1000, 1001-2000) — simple mais risque de hotspot |
| **Hash-based** | `hash(user_id) % N` — distribution uniforme |
| **Geographic** | Par région (EU, US, Asia) — latence optimale |

**⚠️ Le sharding est complexe.** Ne pas sharder avant d'en avoir besoin. Optimise d'abord (index, cache, read replicas).

---

# 5. Architecture Cloud-Native

## 5.1 Les 12 Factors

Méthodologie pour construire des applications cloud-native, formulée par Heroku.

| # | Factor | Principe |
|---|--------|----------|
| 1 | **Codebase** | Un seul repo par app, plusieurs déploiements |
| 2 | **Dependencies** | Déclarer et isoler explicitement les dépendances |
| 3 | **Config** | Configuration dans l'environnement (env vars), pas dans le code |
| 4 | **Backing services** | Traiter les services externes (BDD, cache) comme des ressources attachées |
| 5 | **Build, release, run** | Séparer strictement build, release, et exécution |
| 6 | **Processes** | Exécuter l'app comme des processus **stateless** |
| 7 | **Port binding** | Exporter les services via un port |
| 8 | **Concurrency** | Scaler via le modèle de processus (horizontal) |
| 9 | **Disposability** | Démarrage rapide, arrêt graceful |
| 10 | **Dev/prod parity** | Garder dev, staging et prod aussi similaires que possible |
| 11 | **Logs** | Traiter les logs comme des flux d'événements (stdout) |
| 12 | **Admin processes** | Exécuter les tâches admin comme des processus ponctuels |

---

## 5.2 Serverless

Le cloud exécute ton code **à la demande**. Pas de serveurs à gérer, facturation à l'exécution.

```
Requête HTTP → API Gateway → Lambda Function → Réponse
                                    ↓
                              DynamoDB / S3

✅ Avantages : pas de serveurs, scale automatique, coût proportionnel
❌ Limites : cold start, durée limitée, vendor lock-in, debugging difficile
```

### Quand utiliser Serverless ?

```
✅ Bon pour : APIs simples, webhooks, traitement d'événements, tâches planifiées, MVPs
❌ Pas bon pour : flux temps réel, tâches longues, calcul intensif, applications avec état
```

---

## 5.3 Containers & Orchestration

### Containers (Docker)

Emballe l'application + ses dépendances dans une image portable.

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Orchestration (Kubernetes)

Gère le déploiement, le scaling, et la résilience de centaines de containers.

```
Kubernetes gère :
✅ Déploiement automatisé (rolling updates, rollbacks)
✅ Auto-scaling (plus d'instances si la charge monte)
✅ Self-healing (redémarre les containers crashés)
✅ Service discovery (les services se trouvent entre eux)
✅ Load balancing interne
✅ Secret management
```

---

# 6. Sécurité Architecturale

## 6.1 Zero Trust

```
Ancien modèle ("château fort") :
  Dehors = dangereux, Dedans = confiance → un attaquant à l'intérieur a accès à tout

Zero Trust :
  "Ne jamais faire confiance, toujours vérifier"
  → Chaque requête est authentifiée et autorisée
  → Même les communications internes service-à-service
  → Principe du moindre privilège partout
```

## 6.2 OAuth2 / OpenID Connect

```
Flux Authorization Code (le plus sécurisé pour les apps web) :

1. User → App : "Je veux me connecter"
2. App → Authorization Server : redirect
3. User → Auth Server : login + consent
4. Auth Server → App : authorization code
5. App → Auth Server : échange code contre tokens (backend-to-backend)
6. Auth Server → App : access_token + refresh_token + id_token
7. App → API : requête avec access_token dans le header Authorization

Tokens :
- access_token : court (15min), pour accéder aux APIs
- refresh_token : long (jours), pour renouveler l'access_token
- id_token (OIDC) : informations sur l'utilisateur
```

## 6.3 Principes de Sécurité

| Principe | Application |
|---|---|
| **Least Privilege** | Un service n'a accès qu'à ce dont il a besoin |
| **Defense in Depth** | Plusieurs couches de sécurité (firewall + auth + encryption + audit) |
| **Fail Secure** | En cas d'erreur, refuser l'accès (pas l'accorder) |
| **Input Validation** | Valider TOUTES les entrées (SQL injection, XSS, path traversal) |
| **Encryption at Rest + in Transit** | TLS pour le réseau, encryption pour le stockage |
| **Audit Logging** | Logger toutes les actions sensibles (qui, quoi, quand) |
| **Secret Management** | Jamais de secrets dans le code (utiliser Vault, AWS Secrets Manager) |

---

# 7. Observabilité

L'observabilité est la capacité à **comprendre l'état interne** d'un système à partir de ses sorties. C'est le fondement du debugging en production.

### Les 3 Piliers

```
1. LOGS — "Que s'est-il passé ?"
   Événements textuels horodatés.
   → Structured logging (JSON) pour pouvoir filtrer/chercher
   → Centralisé (ELK Stack, Datadog, Loki)

2. METRICS — "Comment ça se passe globalement ?"
   Données numériques agrégées dans le temps.
   → Latence (P50, P95, P99), throughput, taux d'erreur, CPU, mémoire
   → Prometheus + Grafana, Datadog, CloudWatch

3. TRACES — "Quel chemin a suivi cette requête ?"
   Suivi d'une requête à travers tous les services.
   → Trace ID propagé entre les services
   → Jaeger, Zipkin, AWS X-Ray, OpenTelemetry
```

### Les 4 Golden Signals (Google SRE)

| Signal | Mesure | Alerte si... |
|---|---|---|
| **Latency** | Temps de réponse | P95 > 500ms |
| **Traffic** | Requêtes par seconde | Chute soudaine ou pic anormal |
| **Errors** | Taux d'erreur | > 1% de 5xx |
| **Saturation** | Utilisation des ressources | CPU > 80%, mémoire > 90% |

---

# 8. Architecture Decision Records (ADR)

Un ADR documente **pourquoi** une décision architecturale a été prise. C'est essentiel pour la mémoire d'équipe.

### Template

```markdown
# ADR-001 : Choix de PostgreSQL comme base de données principale

## Statut
Accepté — 2024-03-15

## Contexte
Nous devons choisir une base de données pour le service Commandes.
Le service gère des transactions ACID, des requêtes complexes avec jointures,
et doit supporter ~1000 écritures/seconde.

## Options considérées
1. PostgreSQL — relationnel, ACID, mature
2. MongoDB — document, flexible, scale horizontal natif
3. DynamoDB — serverless, scale automatique, clé-valeur

## Décision
PostgreSQL, car nos données sont hautement relationnelles (commandes → lignes → produits),
nous avons besoin de transactions ACID, et l'équipe a une forte expertise PostgreSQL.

## Conséquences
- ✅ Transactions ACID garanties
- ✅ Requêtes SQL complexes supportées
- ✅ Expertise équipe existante
- ❌ Scaling horizontal plus complexe (read replicas OK, sharding non natif)
- ❌ Schéma rigide (migrations nécessaires pour les changements)

## Métriques de suivi
- Latence P95 des requêtes < 50ms
- Revoir si le volume dépasse 10 000 écritures/seconde
```

---

# 9. Choisir son Architecture — Matrice de Décision

```
TAILLE DE L'ÉQUIPE :
  1-3 devs     → Monolithe MVC, pas de micro-services
  3-10 devs    → Monolithe modulaire ou Clean Architecture
  10-50 devs   → Microservices (si nécessaire)
  50+ devs     → Microservices + Platform team

COMPLEXITÉ MÉTIER :
  Simple (CRUD)           → MVC + Repository
  Modérée (quelques règles) → Clean Architecture + SOLID
  Complexe (beaucoup de règles, processus) → DDD + Event-Driven

SCALE :
  < 1K requêtes/min       → Monolithe, une seule BDD
  1K-10K requêtes/min     → Cache, read replicas, CDN
  10K-100K requêtes/min   → Microservices, sharding, event-driven
  > 100K requêtes/min     → Tout le stack distribué

CHANGEMENT :
  Domaine stable           → Architecture simple, pas besoin de flexibilité
  Domaine qui évolue vite  → DDD, CQRS, architecture modulaire
  Domaine inconnu (startup) → MVP simple, itère, complexifie après

🏆 LA RÈGLE D'OR :
  "L'architecture est un investissement.
   Investis proportionnellement à la complexité et la durée du projet.
   Commence simple. Complexifie quand la douleur l'exige."
```
