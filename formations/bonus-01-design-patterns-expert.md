# 🏆 BONUS 1 — Design Patterns : Niveau Expert

> **Prérequis :** Avoir lu la formation principale (Partie 3 — Design Patterns).
> **Objectif :** Aller au-delà des patterns classiques — patterns avancés, anti-patterns, combinaisons, et application dans des systèmes réels.

---

# 📋 Sommaire

- [1. Patterns Créationnels Avancés](#1-patterns-créationnels-avancés)
  - [1.1 Prototype](#11-prototype)
  - [1.2 Object Pool](#12-object-pool)
  - [1.3 Multiton](#13-multiton)
- [2. Patterns Structurels Avancés](#2-patterns-structurels-avancés)
  - [2.1 Bridge](#21-bridge)
  - [2.2 Composite](#22-composite)
  - [2.3 Flyweight](#23-flyweight)
- [3. Patterns Comportementaux Avancés](#3-patterns-comportementaux-avancés)
  - [3.1 Chain of Responsibility](#31-chain-of-responsibility)
  - [3.2 Mediator](#32-mediator)
  - [3.3 Visitor](#33-visitor)
  - [3.4 Template Method](#34-template-method)
  - [3.5 Iterator](#35-iterator)
  - [3.6 Memento](#36-memento)
- [4. Patterns d'Entreprise](#4-patterns-dentreprise)
  - [4.1 Repository](#41-repository)
  - [4.2 Unit of Work](#42-unit-of-work)
  - [4.3 Specification](#43-specification)
  - [4.4 CQRS](#44-cqrs)
  - [4.5 Event Sourcing](#45-event-sourcing)
- [5. Patterns de Concurrence](#5-patterns-de-concurrence)
  - [5.1 Producer/Consumer](#51-producerconsumer)
  - [5.2 Read-Write Lock](#52-read-write-lock)
  - [5.3 Circuit Breaker](#53-circuit-breaker)
  - [5.4 Retry avec Backoff](#54-retry-avec-backoff-exponentiel)
- [6. Anti-Patterns à Connaître](#6-anti-patterns-à-connaître)
- [7. Combinaisons de Patterns](#7-combinaisons-de-patterns)
- [8. Choisir le Bon Pattern — Arbre de Décision](#8-choisir-le-bon-pattern--arbre-de-décision)

---

# 1. Patterns Créationnels Avancés

## 1.1 Prototype

> **Concept :** Créer de nouveaux objets en **clonant** un prototype existant plutôt qu'en les construisant depuis zéro.

### 🎯 Quand l'utiliser ?

- La création d'un objet est **coûteuse** (requête BDD, parsing de fichier, calcul lourd) et tu as besoin de copies.
- Tu veux créer des variantes d'un objet de référence (templates de documents, configurations par défaut).
- Tu ne connais pas le type exact à la compilation (tu reçois un objet via une interface et tu dois le dupliquer).

### 🛠️ Cas réels

- Cloner des cellules dans un tableur (chaque cellule est un prototype modifiable).
- Dupliquer des game objects dans un moteur de jeu.
- Templates d'emails préconfiguré qu'on clone et personnalise.

### 🔷 C#
```csharp
public interface IPrototype<T>
{
    T Clone();
}

public class ConfigServeur : IPrototype<ConfigServeur>
{
    public string Host { get; set; }
    public int Port { get; set; }
    public Dictionary<string, string> Options { get; set; }
    public TimeSpan Timeout { get; set; }

    // Clone profond — chaque clone a ses propres données
    public ConfigServeur Clone()
    {
        return new ConfigServeur
        {
            Host = Host,
            Port = Port,
            Options = new Dictionary<string, string>(Options), // copie profonde
            Timeout = Timeout
        };
    }
}

// Prototype de base (cher à construire)
var templateProd = new ConfigServeur
{
    Host = "prod.example.com", Port = 443,
    Options = new() { ["ssl"] = "true", ["pool_size"] = "20" },
    Timeout = TimeSpan.FromSeconds(30)
};

// Cloner et personnaliser — rapide !
var serveurEurope = templateProd.Clone();
serveurEurope.Host = "eu.prod.example.com";

var serveurStaging = templateProd.Clone();
serveurStaging.Host = "staging.example.com";
serveurStaging.Options["pool_size"] = "5";
```

### 🟡 Python
```python
import copy

class ConfigServeur:
    def __init__(self, host: str, port: int, options: dict, timeout: int):
        self.host = host
        self.port = port
        self.options = options
        self.timeout = timeout

    def clone(self) -> "ConfigServeur":
        return copy.deepcopy(self)  # clone profond natif Python

# Prototype
template = ConfigServeur("prod.example.com", 443, {"ssl": "true"}, 30)

# Cloner et personnaliser
europe = template.clone()
europe.host = "eu.prod.example.com"
```

### ⚠️ Piège : Clone superficiel vs profond

```
Superficiel (shallow copy) : copie les références → les deux objets partagent les mêmes sous-objets
Profond (deep copy)        : copie tout récursivement → objets complètement indépendants

→ Pour des objets avec des collections ou des objets imbriqués, TOUJOURS faire un clone profond.
```

---

## 1.2 Object Pool

> **Concept :** Réutiliser des objets **coûteux à créer** au lieu de les recréer à chaque fois.

### 🎯 Quand l'utiliser ?

- Créer l'objet est **significativement plus cher** que le réinitialiser (connexions BDD, threads, sockets).
- Tu as un nombre limité de ressources (pool de connexions max 20).
- L'objet est utilisé fréquemment et brièvement.

### 🛠️ Cas réels

- Pool de connexions (ADO.NET, HikariCP, SQLAlchemy pool).
- Pool de threads (ThreadPool).
- Pool d'objets graphiques dans un jeu (projectiles, particules).

### 🔷 C#
```csharp
public class ObjectPool<T> where T : class
{
    private readonly ConcurrentBag<T> _pool = new();
    private readonly Func<T> _factory;
    private readonly int _maxSize;

    public ObjectPool(Func<T> factory, int maxSize = 50)
    {
        _factory = factory;
        _maxSize = maxSize;
    }

    public T Emprunter()
    {
        // Essaie de prendre un objet existant, sinon en crée un nouveau
        return _pool.TryTake(out var item) ? item : _factory();
    }

    public void Rendre(T item)
    {
        // Remet dans le pool si pas plein
        if (_pool.Count < _maxSize)
            _pool.Add(item);
    }
}

// Utilisation — pool de connexions
var pool = new ObjectPool<DbConnection>(
    factory: () => new SqlConnection("Server=..."),
    maxSize: 20
);

var conn = pool.Emprunter();
try
{
    // utiliser la connexion...
}
finally
{
    pool.Rendre(conn); // retour au pool, pas de destruction
}
```

### 🟡 Python
```python
from queue import Queue
from contextlib import contextmanager

class ObjectPool:
    def __init__(self, factory, max_size: int = 20):
        self._pool = Queue(maxsize=max_size)
        self._factory = factory
        # Pré-remplir le pool
        for _ in range(max_size):
            self._pool.put(factory())

    @contextmanager
    def emprunter(self):
        """Context manager pour garantir le retour au pool."""
        obj = self._pool.get()
        try:
            yield obj
        finally:
            self._pool.put(obj)

# Utilisation
pool = ObjectPool(factory=lambda: create_db_connection(), max_size=10)

with pool.emprunter() as conn:
    conn.execute("SELECT * FROM users")
# conn est automatiquement rendu au pool
```

---

## 1.3 Multiton

> **Concept :** Comme le Singleton, mais avec **plusieurs instances nommées** (une par clé).

### 🎯 Quand ?

- Tu as besoin de N instances fixes identifiées par une clé (un logger par module, une connexion par base, un cache par région).

### 🔷 C#
```csharp
public class Logger
{
    private static readonly Dictionary<string, Logger> _instances = new();
    private static readonly object _lock = new();

    public string Module { get; }

    private Logger(string module) { Module = module; }

    public static Logger GetInstance(string module)
    {
        lock (_lock)
        {
            if (!_instances.ContainsKey(module))
                _instances[module] = new Logger(module);
            return _instances[module];
        }
    }

    public void Log(string msg) => Console.WriteLine($"[{Module}] {msg}");
}

// Même module = même instance
var authLogger = Logger.GetInstance("Auth");
var payLogger = Logger.GetInstance("Payment");
var authLogger2 = Logger.GetInstance("Auth"); // même objet que authLogger
```

---

# 2. Patterns Structurels Avancés

## 2.1 Bridge

> **Concept :** Séparer une **abstraction** de son **implémentation** pour qu'elles puissent varier indépendamment.

### 🎯 Pourquoi ?

Tu as deux axes de variation. Par exemple : des formes (Cercle, Carré) ET des renderers (SVG, Canvas, OpenGL). Sans Bridge, tu aurais `CercleSVG`, `CercleCanvas`, `CarréSVG`, `CarréCanvas`... soit N×M classes. Avec Bridge : N + M classes.

### 🛠️ Cas réels

- Abstraction UI + implémentation plateforme (Windows, Mac, Linux).
- Messages + canaux d'envoi (email, SMS, Slack) — c'est un Bridge !
- Persistence + stockage (fichier, BDD, cloud).

### 🔷 C#
```csharp
// IMPLÉMENTATION — l'axe technique
public interface IRenderer
{
    void RenderCercle(double x, double y, double rayon);
    void RenderRectangle(double x, double y, double w, double h);
}

public class SvgRenderer : IRenderer
{
    public void RenderCercle(double x, double y, double r)
        => Console.WriteLine($"<circle cx='{x}' cy='{y}' r='{r}'/>");
    public void RenderRectangle(double x, double y, double w, double h)
        => Console.WriteLine($"<rect x='{x}' y='{y}' width='{w}' height='{h}'/>");
}

public class CanvasRenderer : IRenderer
{
    public void RenderCercle(double x, double y, double r)
        => Console.WriteLine($"ctx.arc({x}, {y}, {r}, 0, 2*PI)");
    public void RenderRectangle(double x, double y, double w, double h)
        => Console.WriteLine($"ctx.fillRect({x}, {y}, {w}, {h})");
}

// ABSTRACTION — l'axe métier
public abstract class Forme
{
    protected IRenderer Renderer; // ← le PONT (bridge)

    protected Forme(IRenderer renderer) { Renderer = renderer; }

    public abstract void Dessiner();
}

public class Cercle : Forme
{
    private double _x, _y, _rayon;

    public Cercle(IRenderer renderer, double x, double y, double r)
        : base(renderer) { _x = x; _y = y; _rayon = r; }

    public override void Dessiner() => Renderer.RenderCercle(_x, _y, _rayon);
}

// 2 formes × 2 renderers = 4 combinaisons avec seulement 4 classes (pas 4 + bridge)
var cercleSvg = new Cercle(new SvgRenderer(), 10, 10, 5);
var cercleCanvas = new Cercle(new CanvasRenderer(), 10, 10, 5);
```

### 💡 Bridge vs Strategy

Les deux injectent un comportement. La différence est conceptuelle :
- **Strategy :** Un algorithme interchangeable (TRI rapide vs bulle).
- **Bridge :** Deux hiérarchies qui varient indépendamment (Forme × Renderer).

---

## 2.2 Composite

> **Concept :** Traiter un objet individuel ET un groupe d'objets de la **même manière** — structure en arbre.

### 🎯 Quand ?

- Système de fichiers : un fichier et un dossier s'affichent pareil, mais le dossier contient des enfants.
- UI : un bouton et un panneau (qui contient des boutons) répondent tous à `render()`.
- Organisation : un employé et un département répondent à `calculerCout()`.

### 🔷 C#
```csharp
// Composant commun
public interface IComposantUI
{
    void Render(int indent = 0);
}

// Feuille — pas d'enfants
public class Bouton : IComposantUI
{
    public string Label { get; }
    public Bouton(string label) => Label = label;
    public void Render(int indent = 0)
        => Console.WriteLine($"{new string(' ', indent)}[Bouton: {Label}]");
}

// Composite — contient des enfants
public class Panneau : IComposantUI
{
    public string Nom { get; }
    private readonly List<IComposantUI> _enfants = new();

    public Panneau(string nom) => Nom = nom;

    public void Ajouter(IComposantUI enfant) => _enfants.Add(enfant);

    public void Render(int indent = 0)
    {
        Console.WriteLine($"{new string(' ', indent)}[Panneau: {Nom}]");
        foreach (var enfant in _enfants)
            enfant.Render(indent + 2); // récursif !
    }
}

// Utilisation — arbre UI
var mainPanel = new Panneau("Main");
var toolbar = new Panneau("Toolbar");
toolbar.Ajouter(new Bouton("Sauver"));
toolbar.Ajouter(new Bouton("Annuler"));
mainPanel.Ajouter(toolbar);
mainPanel.Ajouter(new Bouton("Fermer"));

mainPanel.Render();
// [Panneau: Main]
//   [Panneau: Toolbar]
//     [Bouton: Sauver]
//     [Bouton: Annuler]
//   [Bouton: Fermer]
```

---

## 2.3 Flyweight

> **Concept :** Partager des objets pour **économiser la mémoire** quand un grand nombre d'objets similaires sont nécessaires.

### 🎯 Quand ?

- Éditeur de texte : chaque caractère pourrait être un objet, mais partager les données communes (police, taille) économise énormément de mémoire.
- Jeu vidéo : 10 000 arbres dans une forêt partagent le même mesh 3D et la même texture — seule la position change.
- Cache de String (Java String Pool, C# string interning).

### 🔷 C#
```csharp
// Flyweight — données partagées (intrinsèques)
public class TreeType
{
    public string Nom { get; }
    public string Texture { get; }    // lourd en mémoire
    public string MeshData { get; }   // lourd en mémoire

    public TreeType(string nom, string texture, string mesh)
    { Nom = nom; Texture = texture; MeshData = mesh; }
}

// Factory qui garantit le partage
public class TreeTypeFactory
{
    private static readonly Dictionary<string, TreeType> _cache = new();

    public static TreeType Get(string nom, string texture, string mesh)
    {
        var key = $"{nom}_{texture}_{mesh}";
        if (!_cache.ContainsKey(key))
            _cache[key] = new TreeType(nom, texture, mesh);
        return _cache[key]; // réutilise l'existant
    }
}

// Contexte — données uniques (extrinsèques)
public class Tree
{
    public double X { get; }
    public double Y { get; }
    public TreeType Type { get; } // référence partagée

    public Tree(double x, double y, TreeType type)
    { X = x; Y = y; Type = type; }
}

// 10 000 arbres mais seulement ~5 TreeType en mémoire
var foret = new List<Tree>();
var chene = TreeTypeFactory.Get("Chêne", "chene.png", "chene.obj");
for (int i = 0; i < 10000; i++)
    foret.Add(new Tree(Random.Shared.NextDouble() * 1000, Random.Shared.NextDouble() * 1000, chene));
// 10 000 objets Tree légers, 1 seul TreeType lourd en mémoire
```

---

# 3. Patterns Comportementaux Avancés

## 3.1 Chain of Responsibility

> **Concept :** Faire passer une requête le long d'une **chaîne de handlers**. Chaque handler décide soit de traiter la requête, soit de la passer au suivant.

### 🎯 Cas réels

- **Middleware HTTP** (Express.js, ASP.NET Core, Django) : Auth → Logging → CORS → Rate Limit → Handler.
- Validation multi-niveaux.
- Système de support : Level 1 → Level 2 → Level 3.

### 🔷 C#
```csharp
public abstract class Handler
{
    private Handler? _next;

    public Handler SetNext(Handler next) { _next = next; return next; }

    public virtual string? Handle(string request)
    {
        return _next?.Handle(request);
    }
}

public class AuthHandler : Handler
{
    public override string? Handle(string request)
    {
        if (!request.Contains("token="))
            return "❌ 401 Non autorisé";
        Console.WriteLine("✅ Auth OK");
        return base.Handle(request); // passe au suivant
    }
}

public class RateLimitHandler : Handler
{
    private int _count = 0;
    public override string? Handle(string request)
    {
        if (++_count > 100)
            return "❌ 429 Trop de requêtes";
        Console.WriteLine("✅ Rate limit OK");
        return base.Handle(request);
    }
}

public class LoggingHandler : Handler
{
    public override string? Handle(string request)
    {
        Console.WriteLine($"📝 Log: {request}");
        return base.Handle(request);
    }
}

// Construire la chaîne
var auth = new AuthHandler();
auth.SetNext(new RateLimitHandler())
    .SetNext(new LoggingHandler());

auth.Handle("GET /api/users?token=abc123");
// ✅ Auth OK → ✅ Rate limit OK → 📝 Log: GET /api/users?token=abc123
```

---

## 3.2 Mediator

> **Concept :** Un objet central qui **coordonne** les communications entre objets, pour qu'ils ne se connaissent pas directement.

### 🎯 Quand ?

- Chat room : les utilisateurs envoient des messages au mediator (la room), pas directement aux autres.
- Formulaire UI : quand un champ change, d'autres champs doivent se mettre à jour (le mediator coordonne).
- Contrôle aérien : les avions ne communiquent pas entre eux, ils passent par la tour de contrôle.

### 🔷 C#
```csharp
public interface IChatRoom
{
    void Envoyer(string message, Utilisateur expediteur);
    void Rejoindre(Utilisateur user);
}

public class ChatRoom : IChatRoom
{
    private readonly List<Utilisateur> _users = new();

    public void Rejoindre(Utilisateur user) { _users.Add(user); user.Room = this; }

    public void Envoyer(string message, Utilisateur expediteur)
    {
        foreach (var user in _users.Where(u => u != expediteur))
            user.Recevoir(message, expediteur.Nom);
    }
}

public class Utilisateur
{
    public string Nom { get; }
    public IChatRoom? Room { get; set; }

    public Utilisateur(string nom) => Nom = nom;

    public void Envoyer(string message) => Room?.Envoyer(message, this);
    public void Recevoir(string message, string de)
        => Console.WriteLine($"[{Nom}] reçoit de {de}: {message}");
}

var room = new ChatRoom();
var alice = new Utilisateur("Alice");
var bob = new Utilisateur("Bob");
room.Rejoindre(alice);
room.Rejoindre(bob);

alice.Envoyer("Salut!"); // Bob reçoit "Salut!" de Alice
```

---

## 3.3 Visitor

> **Concept :** Ajouter de **nouvelles opérations** à une hiérarchie d'objets sans modifier leurs classes.

### 🎯 Quand ?

- Compilateur : l'AST (arbre syntaxique) est fixe, mais les opérations (type checking, optimisation, code generation) varient.
- Export multi-format : même structure de document, mais export en PDF, HTML, Markdown.
- Analyse : parcourir un graphe d'objets pour collecter des statistiques sans modifier les classes.

### 🔷 C#
```csharp
// Les éléments acceptent un visiteur
public interface IDocElement
{
    void Accept(IDocVisitor visitor);
}

public class Paragraphe : IDocElement
{
    public string Texte { get; }
    public Paragraphe(string texte) => Texte = texte;
    public void Accept(IDocVisitor visitor) => visitor.Visit(this);
}

public class Image : IDocElement
{
    public string Url { get; }
    public Image(string url) => Url = url;
    public void Accept(IDocVisitor visitor) => visitor.Visit(this);
}

// Visiteurs — nouvelles opérations sans toucher aux éléments
public interface IDocVisitor
{
    void Visit(Paragraphe p);
    void Visit(Image img);
}

public class HtmlExportVisitor : IDocVisitor
{
    public void Visit(Paragraphe p) => Console.WriteLine($"<p>{p.Texte}</p>");
    public void Visit(Image img) => Console.WriteLine($"<img src='{img.Url}'/>");
}

public class MarkdownExportVisitor : IDocVisitor
{
    public void Visit(Paragraphe p) => Console.WriteLine(p.Texte);
    public void Visit(Image img) => Console.WriteLine($"![image]({img.Url})");
}

public class WordCountVisitor : IDocVisitor
{
    public int Count { get; private set; }
    public void Visit(Paragraphe p) => Count += p.Texte.Split(' ').Length;
    public void Visit(Image img) { } // pas de mots dans une image
}

// Utilisation
var doc = new List<IDocElement>
{
    new Paragraphe("Bonjour le monde"),
    new Image("photo.png"),
    new Paragraphe("Fin du document")
};

var html = new HtmlExportVisitor();
foreach (var elem in doc) elem.Accept(html);
// <p>Bonjour le monde</p>
// <img src='photo.png'/>
// <p>Fin du document</p>
```

---

## 3.4 Template Method

> **Concept :** Définir le **squelette** d'un algorithme dans une méthode, en laissant les sous-classes redéfinir certaines étapes.

### 🎯 Quand ?

- ETL : Extract → Transform → Load — le flux est fixe, mais chaque étape varie.
- Processus de build : compile → test → package — le squelette est le même, les détails changent.
- Jeu : `gameLoop()` appelle `init()`, `update()`, `render()` — les sous-classes implémentent chaque étape.

### 🔷 C#
```csharp
public abstract class DataMiner
{
    // Template Method — le squelette FIXE de l'algorithme
    public void Mine(string source)
    {
        var rawData = Extract(source);
        var data = Transform(rawData);
        Analyze(data);
        GenerateReport(data);
    }

    // Étapes à implémenter par les sous-classes
    protected abstract string Extract(string source);
    protected abstract string[] Transform(string raw);

    // Étapes avec implémentation par défaut (overridable)
    protected virtual void Analyze(string[] data)
        => Console.WriteLine($"Analyse de {data.Length} éléments");

    protected virtual void GenerateReport(string[] data)
        => Console.WriteLine("Rapport généré.");
}

public class CsvDataMiner : DataMiner
{
    protected override string Extract(string source) => File.ReadAllText(source);
    protected override string[] Transform(string raw) => raw.Split('\n');
}

public class ApiDataMiner : DataMiner
{
    protected override string Extract(string source) => HttpClient.Get(source);
    protected override string[] Transform(string raw) => JsonParser.Parse(raw);
}
```

---

## 3.5 Iterator

> **Concept :** Parcourir une collection **sans exposer** sa structure interne.

La plupart des langages modernes implémentent l'Iterator nativement : `foreach` en C#, `for-in` en Java/Python, range-based for en C++. Mais le pattern reste utile pour des traversées personnalisées.

### 🔷 C# (implémentation native)
```csharp
public class Playlist : IEnumerable<Song>
{
    private readonly List<Song> _songs = new();

    public void Ajouter(Song song) => _songs.Add(song);

    // Itérateur inversé — parcours personnalisé
    public IEnumerable<Song> EnOrdreInverse()
    {
        for (int i = _songs.Count - 1; i >= 0; i--)
            yield return _songs[i]; // yield = lazy evaluation
    }

    // Itérateur aléatoire
    public IEnumerable<Song> Shuffle()
    {
        var shuffled = _songs.OrderBy(_ => Random.Shared.Next()).ToList();
        foreach (var song in shuffled)
            yield return song;
    }

    public IEnumerator<Song> GetEnumerator() => _songs.GetEnumerator();
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

### 🟡 Python (protocole itérateur natif)
```python
class Playlist:
    def __init__(self):
        self._songs = []

    def ajouter(self, song): self._songs.append(song)

    def __iter__(self):
        """Itérateur standard."""
        return iter(self._songs)

    def shuffle(self):
        """Itérateur personnalisé avec yield (générateur)."""
        import random
        shuffled = self._songs[:]
        random.shuffle(shuffled)
        for song in shuffled:
            yield song  # lazy — génère une chanson à la fois
```

---

## 3.6 Memento

> **Concept :** Capturer et restaurer l'**état interne** d'un objet sans violer l'encapsulation.

### 🎯 Quand ?

- Undo/Redo (complémentaire au Command — Command gère les actions, Memento gère les états).
- Sauvegarde/restauration de jeu vidéo.
- Points de restauration de base de données.

### 🔷 C#
```csharp
// Memento — snapshot de l'état
public record EditorMemento(string Contenu, int CursorPosition);

public class Editor
{
    public string Contenu { get; set; } = "";
    public int CursorPosition { get; set; } = 0;

    public EditorMemento Sauvegarder()
        => new(Contenu, CursorPosition);

    public void Restaurer(EditorMemento memento)
    {
        Contenu = memento.Contenu;
        CursorPosition = memento.CursorPosition;
    }
}

public class HistoriqueEditeur
{
    private readonly Stack<EditorMemento> _snapshots = new();

    public void Push(EditorMemento memento) => _snapshots.Push(memento);
    public EditorMemento? Pop() => _snapshots.TryPop(out var m) ? m : null;
}

// Utilisation
var editor = new Editor();
var historique = new HistoriqueEditeur();

editor.Contenu = "Bonjour";
historique.Push(editor.Sauvegarder()); // snapshot

editor.Contenu = "Bonjour le monde";
historique.Push(editor.Sauvegarder()); // snapshot

editor.Contenu = "ERREUR !!!";

// Undo !
editor.Restaurer(historique.Pop()!);
Console.WriteLine(editor.Contenu); // "Bonjour le monde"
```

---

# 4. Patterns d'Entreprise

Ces patterns viennent du livre "Patterns of Enterprise Application Architecture" (Martin Fowler) et de la communauté DDD (Domain-Driven Design). Ils sont utilisés dans les projets professionnels de grande taille.

## 4.1 Repository

> **Concept :** Abstraire l'accès aux données derrière une **collection en mémoire**. Le code métier ne sait pas qu'il y a une base de données.

### 🔷 C#
```csharp
public interface IUserRepository
{
    Task<User?> FindById(Guid id);
    Task<User?> FindByEmail(string email);
    Task<IReadOnlyList<User>> FindAll();
    Task Add(User user);
    Task Update(User user);
    Task Delete(Guid id);
}

// Implémentation concrète — Entity Framework
public class EfUserRepository : IUserRepository
{
    private readonly AppDbContext _db;
    public EfUserRepository(AppDbContext db) => _db = db;

    public async Task<User?> FindById(Guid id) => await _db.Users.FindAsync(id);
    public async Task<User?> FindByEmail(string email)
        => await _db.Users.FirstOrDefaultAsync(u => u.Email == email);
    public async Task Add(User user) { _db.Users.Add(user); await _db.SaveChangesAsync(); }
    // ...
}

// Implémentation pour les tests
public class InMemoryUserRepository : IUserRepository
{
    private readonly List<User> _users = new();
    public Task<User?> FindById(Guid id) => Task.FromResult(_users.FirstOrDefault(u => u.Id == id));
    // ...
}
```

---

## 4.2 Unit of Work

> **Concept :** Regrouper plusieurs opérations de persistence en une **seule transaction**. Tout réussit ou tout échoue.

### 🔷 C#
```csharp
public interface IUnitOfWork : IDisposable
{
    IUserRepository Users { get; }
    IOrderRepository Orders { get; }
    Task<int> CommitAsync();  // sauvegarde tout en une transaction
    Task RollbackAsync();
}

// Utilisation
public class PlaceOrderUseCase
{
    private readonly IUnitOfWork _uow;

    public async Task Execute(Guid userId, OrderDto dto)
    {
        var user = await _uow.Users.FindById(userId);
        var order = new Order(user, dto.Items);

        await _uow.Orders.Add(order);
        user.OrderCount++;
        await _uow.Users.Update(user);

        await _uow.CommitAsync(); // tout en une transaction atomique
    }
}
```

---

## 4.3 Specification

> **Concept :** Encapsuler une **règle métier** dans un objet réutilisable et combinable.

### 🔷 C#
```csharp
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T entity);
    Expression<Func<T, bool>> ToExpression(); // pour les requêtes BDD
}

public class UserIsActive : ISpecification<User>
{
    public bool IsSatisfiedBy(User u) => u.IsActive && u.LastLogin > DateTime.Now.AddDays(-90);
    public Expression<Func<User, bool>> ToExpression()
        => u => u.IsActive && u.LastLogin > DateTime.Now.AddDays(-90);
}

public class UserIsAdult : ISpecification<User>
{
    public bool IsSatisfiedBy(User u) => u.Age >= 18;
    public Expression<Func<User, bool>> ToExpression() => u => u.Age >= 18;
}

// Combinaison
var spec = new AndSpecification<User>(new UserIsActive(), new UserIsAdult());
var eligibleUsers = users.Where(u => spec.IsSatisfiedBy(u));
```

---

## 4.4 CQRS

> **Command Query Responsibility Segregation :** Séparer les opérations de **lecture** (Query) et d'**écriture** (Command) en modèles différents.

### 🎯 Pourquoi ?

Les besoins en lecture et écriture sont souvent très différents. La lecture doit être rapide et peut utiliser des vues dénormalisées. L'écriture doit valider et maintenir la cohérence. Les mélanger crée des compromis.

```
AVANT (CRUD classique) :
  UserService.GetUser()     ← requête simple
  UserService.CreateUser()  ← validation complexe
  → même modèle, mêmes contraintes

APRÈS (CQRS) :
  Queries → ReadModel (rapide, dénormalisé, optimisé lecture)
  Commands → WriteModel (validé, cohérent, règles métier)
```

### 🔷 C#
```csharp
// COMMAND (écriture) — passe par la validation métier
public record CreateUserCommand(string Email, string Name);

public class CreateUserHandler
{
    private readonly IUserRepository _repo;

    public async Task Handle(CreateUserCommand cmd)
    {
        // Validation métier
        if (await _repo.FindByEmail(cmd.Email) != null)
            throw new DomainException("Email existe déjà");

        var user = new User(cmd.Email, cmd.Name);
        await _repo.Add(user);
    }
}

// QUERY (lecture) — accès direct, optimisé, pas de logique métier
public record UserDto(Guid Id, string Email, string Name, int OrderCount);

public class GetUserByIdQuery
{
    private readonly IDbConnection _db; // accès direct, pas de repository

    public async Task<UserDto?> Handle(Guid id)
    {
        return await _db.QuerySingleOrDefaultAsync<UserDto>(
            "SELECT Id, Email, Name, OrderCount FROM UsersView WHERE Id = @Id",
            new { Id = id }
        );
    }
}
```

---

## 4.5 Event Sourcing

> **Concept :** Au lieu de stocker l'**état actuel**, stocker tous les **événements** qui ont mené à cet état. L'état se reconstruit en rejouant les événements.

### 🎯 Pourquoi ?

- **Audit complet :** Tu sais TOUT ce qui s'est passé, dans quel ordre, par qui.
- **Voyage dans le temps :** Tu peux reconstruire l'état à n'importe quel moment passé.
- **Correction :** Tu peux ajouter un événement compensatoire au lieu de modifier l'historique.

### Principe
```
CRUD traditionnel :
  Compte { solde: 750 }  ← on ne sait pas comment on en est arrivé là

Event Sourcing :
  CompteCreé { solde_initial: 1000 }
  Retrait { montant: 200 }
  Dépôt { montant: 50 }
  Retrait { montant: 100 }
  → Replay : 1000 - 200 + 50 - 100 = 750
  → On sait TOUT
```

### 🔷 C#
```csharp
public abstract record DomainEvent(DateTime Timestamp);

public record CompteCreé(decimal SoldeInitial, DateTime Timestamp) : DomainEvent(Timestamp);
public record ArgentDéposé(decimal Montant, DateTime Timestamp) : DomainEvent(Timestamp);
public record ArgentRetiré(decimal Montant, DateTime Timestamp) : DomainEvent(Timestamp);

public class CompteBancaire
{
    private readonly List<DomainEvent> _events = new();
    public decimal Solde { get; private set; }

    // Appliquer un événement = modifier l'état
    public void Apply(DomainEvent evt)
    {
        switch (evt)
        {
            case CompteCreé e: Solde = e.SoldeInitial; break;
            case ArgentDéposé e: Solde += e.Montant; break;
            case ArgentRetiré e: Solde -= e.Montant; break;
        }
        _events.Add(evt);
    }

    // Reconstruire depuis l'historique
    public static CompteBancaire FromEvents(IEnumerable<DomainEvent> events)
    {
        var compte = new CompteBancaire();
        foreach (var evt in events) compte.Apply(evt);
        return compte;
    }

    public IReadOnlyList<DomainEvent> Events => _events;
}
```

---

# 5. Patterns de Concurrence

Ces patterns sont essentiels dans les systèmes modernes (multi-thread, distribués, microservices).

## 5.1 Producer/Consumer

> **Concept :** Un ou plusieurs producteurs génèrent des tâches dans une **file d'attente**, un ou plusieurs consommateurs les traitent.

### 🎯 Cas réels

- File d'attente de messages (RabbitMQ, Kafka, SQS).
- Pipeline de traitement d'images (upload → resize → compress → store).
- Système de logging asynchrone.

### 🔷 C#
```csharp
var queue = new BlockingCollection<string>(boundedCapacity: 100);

// Producer
Task.Run(() =>
{
    for (int i = 0; i < 1000; i++)
        queue.Add($"Tâche {i}"); // bloque si la file est pleine
    queue.CompleteAdding();
});

// Consumers (multiples)
Parallel.ForEach(queue.GetConsumingEnumerable(), tache =>
{
    Console.WriteLine($"[Thread {Thread.CurrentThread.ManagedThreadId}] Traite: {tache}");
});
```

---

## 5.2 Read-Write Lock

> **Concept :** Plusieurs lecteurs simultanés, mais un seul écrivain exclusif.

### 🔷 C#
```csharp
public class ThreadSafeCache<TKey, TValue>
{
    private readonly Dictionary<TKey, TValue> _cache = new();
    private readonly ReaderWriterLockSlim _lock = new();

    public TValue? Get(TKey key)
    {
        _lock.EnterReadLock(); // multiples lecteurs OK
        try { return _cache.GetValueOrDefault(key); }
        finally { _lock.ExitReadLock(); }
    }

    public void Set(TKey key, TValue value)
    {
        _lock.EnterWriteLock(); // un seul écrivain
        try { _cache[key] = value; }
        finally { _lock.ExitWriteLock(); }
    }
}
```

---

## 5.3 Circuit Breaker

> **Concept :** Couper temporairement les appels à un service défaillant pour éviter l'**effet cascade** et permettre la récupération.

### 🎯 Pourquoi ?

Si un microservice est down et que tu continues à l'appeler, les requêtes s'empilent, les timeouts s'accumulent, et ton propre service devient lent puis tombe. Le Circuit Breaker coupe les appels après N échecs et les reprend progressivement.

```
FERMÉ (normal)    → les appels passent normalement
  ↓ (N échecs)
OUVERT (coupé)    → les appels échouent immédiatement (fail fast)
  ↓ (après timeout)
SEMI-OUVERT       → laisse passer 1 appel test
  ↓ (succès)          ↓ (échec)
FERMÉ              OUVERT
```

### 🔷 C#
```csharp
public class CircuitBreaker
{
    private enum State { Ferme, Ouvert, SemiOuvert }

    private State _state = State.Ferme;
    private int _echecCount = 0;
    private DateTime _ouvertJusque = DateTime.MinValue;
    private readonly int _seuilEchecs;
    private readonly TimeSpan _dureeOuverture;

    public CircuitBreaker(int seuilEchecs = 5, int secondesOuverture = 30)
    {
        _seuilEchecs = seuilEchecs;
        _dureeOuverture = TimeSpan.FromSeconds(secondesOuverture);
    }

    public async Task<T> Execute<T>(Func<Task<T>> action)
    {
        if (_state == State.Ouvert)
        {
            if (DateTime.Now < _ouvertJusque)
                throw new CircuitBreakerOpenException("Circuit ouvert — fail fast");
            _state = State.SemiOuvert;
        }

        try
        {
            var result = await action();
            OnSuccess();
            return result;
        }
        catch (Exception)
        {
            OnFailure();
            throw;
        }
    }

    private void OnSuccess() { _echecCount = 0; _state = State.Ferme; }
    private void OnFailure()
    {
        _echecCount++;
        if (_echecCount >= _seuilEchecs)
        {
            _state = State.Ouvert;
            _ouvertJusque = DateTime.Now.Add(_dureeOuverture);
        }
    }
}
```

---

## 5.4 Retry avec Backoff Exponentiel

> **Concept :** Réessayer une opération qui échoue avec un **délai croissant** entre chaque tentative.

### 🎯 Pourquoi ?

Un service momentanément indisponible peut revenir en quelques secondes. Mais bombarder de requêtes aggrave le problème. Le backoff exponentiel espace les tentatives : 1s → 2s → 4s → 8s...

### 🟡 Python
```python
import asyncio
import random

async def retry_with_backoff(func, max_retries=5, base_delay=1.0):
    for attempt in range(max_retries):
        try:
            return await func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise  # dernier essai, on propage l'erreur

            delay = base_delay * (2 ** attempt)  # exponentiel
            jitter = random.uniform(0, delay * 0.1)  # anti-synchronisation
            print(f"⚠️ Échec (tentative {attempt + 1}), retry dans {delay + jitter:.1f}s")
            await asyncio.sleep(delay + jitter)
```

---

# 6. Anti-Patterns à Connaître

Connaître les anti-patterns est aussi important que connaître les patterns. Ce sont des **erreurs de conception récurrentes** qui semblent bonnes mais mènent à des problèmes.

| Anti-Pattern | Description | Solution |
|---|---|---|
| **God Object** | Une classe qui fait TOUT (5000 lignes, 100 méthodes) | Découper en classes SRP |
| **Spaghetti Code** | Flux de contrôle incompréhensible, goto partout | Refactoriser, appliquer SOLID |
| **Golden Hammer** | Utiliser le même outil pour tout ("tout est un Singleton") | Choisir le pattern adapté au problème |
| **Lava Flow** | Du code mort/legacy que personne n'ose supprimer | Couverture de tests + nettoyage |
| **Copy-Paste Programming** | Dupliquer au lieu de factoriser | Extract method, héritage, composition |
| **Premature Optimization** | Optimiser avant de savoir ce qui est lent | Profiler d'abord, optimiser ensuite |
| **Singleton Abuse** | Tout mettre en Singleton = état global partout | Injection de dépendances |
| **Anemic Domain Model** | Entités sans logique (que des getters/setters), toute la logique dans les services | Rich Domain Model (DDD) |
| **Circular Dependencies** | A dépend de B qui dépend de A | Inverser les dépendances (DIP), interfaces |
| **Magic Numbers** | `if (status == 3)` — c'est quoi 3 ? | Constantes nommées, enums |

---

# 7. Combinaisons de Patterns

Les patterns ne vivent pas en isolation. Voici des combinaisons classiques dans les projets réels :

| Combinaison | Cas d'usage |
|---|---|
| **Repository + Unit of Work** | Persistence avec transactions propres |
| **Factory + Strategy** | Créer le bon algorithme selon un contexte |
| **Observer + Mediator** | Event bus centralisé (MediatR, EventEmitter) |
| **Decorator + Chain of Responsibility** | Pipeline middleware (ASP.NET Core, Express) |
| **Command + Memento** | Undo/Redo complet (Command pour les actions, Memento pour les états) |
| **State + Observer** | Notifier quand l'état change (machines à états réactives) |
| **Abstract Factory + Singleton** | Factory unique qui crée des familles d'objets |
| **CQRS + Event Sourcing** | Architecture event-driven complète |
| **Proxy + Circuit Breaker** | Proxy réseau qui coupe les appels vers un service down |
| **Builder + Prototype** | Construire un template, puis le cloner et personnaliser |

---

# 8. Choisir le Bon Pattern — Arbre de Décision

```
🤔 Quel est mon problème ?

CRÉATION D'OBJETS :
├── "Je veux une seule instance"                    → Singleton (ou DI)
├── "Je veux N instances nommées"                   → Multiton
├── "Je ne connais pas le type à l'avance"          → Factory Method
├── "Je dois créer des familles cohérentes"          → Abstract Factory
├── "Mon objet est complexe avec plein d'options"   → Builder
├── "Cloner est plus rapide que construire"          → Prototype
└── "Créer coûte cher, je veux réutiliser"          → Object Pool

STRUCTURE / ORGANISATION :
├── "Deux interfaces incompatibles"                 → Adapter
├── "Ajouter des comportements sans modifier"       → Decorator
├── "Simplifier un sous-système complexe"           → Facade
├── "Contrôler / cacher l'accès à un objet"         → Proxy
├── "Deux axes de variation indépendants"           → Bridge
├── "Traiter arbre et feuille de la même façon"     → Composite
└── "Économiser la mémoire (objets partagés)"       → Flyweight

COMMUNICATION / COMPORTEMENT :
├── "Notifier plusieurs abonnés"                    → Observer
├── "Changer d'algorithme à la volée"               → Strategy
├── "Undo/Redo, historique d'actions"               → Command
├── "Sauvegarder/restaurer un état"                 → Memento
├── "Comportement qui change selon l'état"          → State
├── "Chaîne de handlers (middleware)"               → Chain of Responsibility
├── "Coordonner sans couplage direct"               → Mediator
├── "Ajouter des opérations à une hiérarchie fixe"  → Visitor
├── "Squelette d'algo avec étapes variables"        → Template Method
└── "Parcourir sans exposer la structure"            → Iterator

ENTREPRISE / DISTRIBUTION :
├── "Abstraire l'accès aux données"                 → Repository
├── "Transaction multi-entités"                     → Unit of Work
├── "Règle métier réutilisable et combinable"       → Specification
├── "Séparer lecture et écriture"                   → CQRS
├── "Historique complet d'événements"               → Event Sourcing
├── "Protéger contre les services down"             → Circuit Breaker
└── "Réessayer avec délai croissant"                → Retry + Backoff
```
