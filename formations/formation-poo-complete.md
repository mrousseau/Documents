# 🎓 Formation Complète — POO, Design Patterns & Architecture

> **Langages :** C# · Java · C++ · Python
> **Objectif :** Maîtriser la POO jusqu'à l'architecture logicielle — rapidement et efficacement.
> **Format :** Chaque concept est présenté dans les 4 langages côte à côte pour comparer et comprendre vite.

---

# 📋 Sommaire Général

- [PARTIE 1 — Fondamentaux de la POO](#partie-1--fondamentaux-de-la-poo)
  - [1.1 Classes & Objets](#11-classes--objets)
  - [1.2 Encapsulation](#12-encapsulation)
  - [1.3 Héritage](#13-héritage)
  - [1.4 Polymorphisme](#14-polymorphisme)
  - [1.5 Abstraction & Interfaces](#15-abstraction--interfaces)
  - [1.6 Composition vs Héritage](#16-composition-vs-héritage)
- [PARTIE 2 — Principes SOLID](#partie-2--principes-solid)
  - [2.1 S — Single Responsibility](#21-s--single-responsibility-principle)
  - [2.2 O — Open/Closed](#22-o--openclosed-principle)
  - [2.3 L — Liskov Substitution](#23-l--liskov-substitution-principle)
  - [2.4 I — Interface Segregation](#24-i--interface-segregation-principle)
  - [2.5 D — Dependency Inversion](#25-d--dependency-inversion-principle)
- [PARTIE 3 — Design Patterns](#partie-3--design-patterns)
  - [3.1 Singleton](#31-singleton)
  - [3.2 Factory Method](#32-factory-method)
  - [3.3 Abstract Factory](#33-abstract-factory)
  - [3.4 Builder](#34-builder)
  - [3.5 Adapter](#35-adapter)
  - [3.6 Decorator](#36-decorator)
  - [3.7 Facade](#37-facade)
  - [3.8 Proxy](#38-proxy)
  - [3.9 Observer](#39-observer)
  - [3.10 Strategy](#310-strategy)
  - [3.11 Command](#311-command)
  - [3.12 State](#312-state)
- [PARTIE 4 — Architecture Logicielle](#partie-4--architecture-logicielle)
  - [4.1 MVC](#41-mvc--model-view-controller)
  - [4.2 Clean Architecture](#42-clean-architecture)
  - [4.3 Architecture Hexagonale](#43-architecture-hexagonale-ports--adapters)
  - [4.4 Microservices](#44-microservices)

---

# PARTIE 1 — Fondamentaux de la POO

## Sommaire Partie 1

| # | Chapitre | Ce que tu apprends |
|---|----------|--------------------|
| 1.1 | [Classes & Objets](#11-classes--objets) | Classe, instanciation, constructeur, attributs, méthodes |
| 1.2 | [Encapsulation](#12-encapsulation) | Visibilité, getters/setters, propriétés, protection des données |
| 1.3 | [Héritage](#13-héritage) | Extends, super/base, surcharge, classes abstraites |
| 1.4 | [Polymorphisme](#14-polymorphisme) | Override, overload, polymorphisme dynamique |
| 1.5 | [Abstraction & Interfaces](#15-abstraction--interfaces) | Interfaces, classes abstraites, contrats |
| 1.6 | [Composition vs Héritage](#16-composition-vs-héritage) | has-a vs is-a, injection, délégation |

### Pourquoi la POO ?

Avant la POO, la programmation procédurale organisait le code en **fonctions** et **données séparées**. Ça fonctionne pour de petits programmes, mais quand un projet grandit (des dizaines de milliers de lignes, plusieurs développeurs), le code procédural devient un cauchemar : les données sont accessibles de partout, les fonctions sont interdépendantes, et un changement à un endroit casse tout ailleurs.

La **Programmation Orientée Objet** résout ce problème en regroupant les **données** et les **comportements** qui vont ensemble dans une même unité : l'**objet**. Cela permet de modéliser le monde réel (un client, une commande, un véhicule) et de créer du code **modulaire**, **réutilisable** et **maintenable**.

Les 4 piliers de la POO sont : **Encapsulation**, **Héritage**, **Polymorphisme** et **Abstraction**. Chacun résout un problème précis, et ensemble ils forment un système cohérent pour gérer la complexité logicielle.

---

## 1.1 Classes & Objets

> **Concept :** Une **classe** est un plan (blueprint). Un **objet** est une instance concrète de ce plan.

### 🎯 Pourquoi c'est important ?

La classe est le **fondement absolu** de la POO. Sans comprendre la relation classe/objet, rien d'autre n'a de sens. C'est le mécanisme qui permet de passer de "j'ai des variables et des fonctions en vrac" à "j'ai des entités cohérentes qui représentent des concepts de mon domaine".

**Analogie concrète :** Imagine un plan d'architecte pour une maison. Le plan (la classe) décrit le nombre de pièces, la taille, les matériaux. Mais tu ne peux pas vivre dans un plan. Tu dois **construire** la maison (créer un objet/instance). À partir du même plan, tu peux construire 10 maisons différentes, chacune avec sa propre adresse, ses propres habitants — mais toutes suivent la même structure.

### 📚 Vocabulaire essentiel

| Terme | Définition | Analogie |
|-------|-----------|----------|
| **Classe** | Modèle qui définit attributs et méthodes | Le plan de la maison |
| **Objet** | Instance vivante d'une classe en mémoire | La maison construite |
| **Constructeur** | Méthode spéciale appelée à la création | L'équipe de construction |
| **Attribut** | Variable appartenant à l'objet (son état) | Les caractéristiques (couleur, taille) |
| **Méthode** | Fonction appartenant à la classe (son comportement) | Ce qu'on peut faire (ouvrir la porte) |
| **Instanciation** | Le fait de créer un objet à partir d'une classe | Le moment de la construction |

### 🧩 Le concept d'état et de comportement

Chaque objet possède :
- Un **état** : l'ensemble de ses attributs à un instant donné. Deux voitures de la même classe peuvent avoir des états différents (l'une roule à 50 km/h, l'autre est à l'arrêt).
- Un **comportement** : les méthodes qui modifient ou consultent l'état. La méthode `accelerer()` modifie l'attribut `vitesse`.

C'est cette combinaison état + comportement qui fait la puissance de l'objet par rapport à une simple structure de données.

### 💻 Code — Classe `Voiture`

### 🔷 C#
```csharp
public class Voiture
{
    // Attributs (état de l'objet)
    public string Marque;
    public int Vitesse;

    // Constructeur — appelé automatiquement à la création
    public Voiture(string marque)
    {
        Marque = marque;
        Vitesse = 0;  // état initial
    }

    // Méthode (comportement)
    public void Accelerer(int kmh)
    {
        Vitesse += kmh;
    }

    public override string ToString()
    {
        return $"{Marque} roule à {Vitesse} km/h";
    }
}

// Instanciation — on crée des objets concrets à partir du plan
var maVoiture = new Voiture("Peugeot");   // objet 1
var taVoiture = new Voiture("Renault");   // objet 2 — même classe, objet différent

maVoiture.Accelerer(50);
Console.WriteLine(maVoiture); // Peugeot roule à 50 km/h
Console.WriteLine(taVoiture); // Renault roule à 0 km/h — état indépendant
```

### 🟠 Java
```java
public class Voiture {
    public String marque;
    public int vitesse;

    public Voiture(String marque) {
        this.marque = marque;    // this = l'objet en cours de construction
        this.vitesse = 0;
    }

    public void accelerer(int kmh) {
        vitesse += kmh;
    }

    @Override
    public String toString() {
        return marque + " roule à " + vitesse + " km/h";
    }
}

Voiture maVoiture = new Voiture("Peugeot");
maVoiture.accelerer(50);
System.out.println(maVoiture);
```

### 🔵 C++
```cpp
#include <iostream>
#include <string>

class Voiture {
public:
    std::string marque;
    int vitesse;

    // Liste d'initialisation (idiomatique C++) — plus performant qu'assigner dans le corps
    Voiture(const std::string& marque) : marque(marque), vitesse(0) {}

    void accelerer(int kmh) { vitesse += kmh; }

    // const = promesse de ne pas modifier l'objet
    void afficher() const {
        std::cout << marque << " roule à " << vitesse << " km/h" << std::endl;
    }
};

int main() {
    Voiture maVoiture("Peugeot");     // créé sur la pile (stack)
    maVoiture.accelerer(50);
    maVoiture.afficher();
}
```

### 🟡 Python
```python
class Voiture:
    # __init__ est le constructeur (méthode magique / dunder method)
    # self = référence à l'objet courant (OBLIGATOIRE en Python)
    def __init__(self, marque: str):
        self.marque = marque
        self.vitesse = 0

    def accelerer(self, kmh: int):
        self.vitesse += kmh

    def __str__(self):
        return f"{self.marque} roule à {self.vitesse} km/h"

# Pas de mot-clé "new" en Python
ma_voiture = Voiture("Peugeot")
ma_voiture.accelerer(50)
print(ma_voiture)  # Peugeot roule à 50 km/h
```

### 🔑 Comparaison rapide

| Point | C# | Java | C++ | Python |
|-------|-----|------|-----|--------|
| Constructeur | Nom de la classe | Nom de la classe | Nom + liste init | `__init__` |
| Instanciation | `new Voiture()` | `new Voiture()` | `Voiture()` ou `new` | `Voiture()` |
| `this`/`self` | implicite | `this` (implicite) | implicite | `self` (obligatoire) |
| `toString` | `ToString()` | `toString()` | méthode custom | `__str__` |
| Gestion mémoire | Garbage Collector | Garbage Collector | Manuelle ou smart pointers | Garbage Collector |

### ⚠️ Pièges courants

**Piège 1 — Confondre classe et objet :** La classe `Voiture` n'est PAS une voiture. C'est le PLAN. `maVoiture` est une voiture concrète en mémoire.

**Piège 2 — Oublier que chaque objet a son propre état :** `maVoiture` et `taVoiture` sont indépendants. Modifier l'un ne change pas l'autre.

**Piège 3 — C++ : pile vs tas.** `Voiture v("X")` = pile (automatique). `new Voiture("X")` = tas (tu gères la mémoire, ou utilise `std::unique_ptr`).

---

## 1.2 Encapsulation

> **Concept :** Protéger les données internes d'un objet et contrôler l'accès via des méthodes dédiées.

### 🎯 Pourquoi c'est important ?

Sans encapsulation, n'importe quel code peut modifier les données internes directement. Imagine un compte bancaire dont le solde est public : `compte.solde = 1000000`. L'encapsulation force le passage par des **règles** (vérifier le montant, le solde suffisant, etc.).

**Les 3 bénéfices concrets :**
1. **Intégrité des données :** L'objet est toujours dans un état cohérent.
2. **Liberté d'évolution :** Tu peux changer l'implémentation interne sans casser le code externe.
3. **Moins de bugs :** En limitant les points d'accès, tu réduis la surface d'erreur.

**Analogie :** Un distributeur de billets. Tu ne touches jamais les billets dans le coffre (privé). Tu utilises l'écran et le clavier (interface publique) qui vérifient ton code PIN et ton solde.

### 📚 Les niveaux de visibilité

| Modificateur | Qui peut accéder ? | Quand l'utiliser ? |
|---|---|---|
| **Public** | Tout le monde | Interface de ta classe |
| **Privé** | La classe uniquement | Données internes |
| **Protégé** | La classe + ses enfants | Données utiles aux sous-classes |

**Règle d'or : attributs privés + accès contrôlé. Expose le minimum nécessaire.**

### 💻 Code — Compte Bancaire sécurisé

### 🔷 C#
```csharp
public class CompteBancaire
{
    private decimal _solde;                       // privé
    public string Titulaire { get; private set; } // lecture publique, écriture interne

    public CompteBancaire(string titulaire, decimal soldeInitial)
    {
        Titulaire = titulaire;
        _solde = soldeInitial;
    }

    public decimal Solde => _solde; // lecture seule

    public void Deposer(decimal montant)
    {
        if (montant <= 0) throw new ArgumentException("Montant invalide");
        _solde += montant;
    }

    public bool Retirer(decimal montant)
    {
        if (montant <= 0) throw new ArgumentException("Montant invalide");
        if (montant > _solde) return false;
        _solde -= montant;
        return true;
    }
}
```

### 🟠 Java
```java
public class CompteBancaire {
    private double solde;
    private final String titulaire;  // final = immutable après construction

    public CompteBancaire(String titulaire, double soldeInitial) {
        this.titulaire = titulaire;
        this.solde = soldeInitial;
    }

    public double getSolde() { return solde; }
    public String getTitulaire() { return titulaire; }
    // Pas de setSolde() → le solde ne se modifie que via deposer/retirer

    public void deposer(double montant) {
        if (montant <= 0) throw new IllegalArgumentException("Montant invalide");
        solde += montant;
    }

    public boolean retirer(double montant) {
        if (montant <= 0) throw new IllegalArgumentException("Montant invalide");
        if (montant > solde) return false;
        solde -= montant;
        return true;
    }
}
```

### 🔵 C++
```cpp
class CompteBancaire {
private:
    double solde_;
    std::string titulaire_;
public:
    CompteBancaire(const std::string& titulaire, double soldeInitial)
        : titulaire_(titulaire), solde_(soldeInitial) {}

    // const = promesse de ne pas modifier l'objet
    double getSolde() const { return solde_; }
    const std::string& getTitulaire() const { return titulaire_; }

    void deposer(double montant) {
        if (montant <= 0) throw std::invalid_argument("Montant invalide");
        solde_ += montant;
    }

    bool retirer(double montant) {
        if (montant > solde_) return false;
        solde_ -= montant;
        return true;
    }
};
```

### 🟡 Python
```python
class CompteBancaire:
    def __init__(self, titulaire: str, solde_initial: float):
        self._titulaire = titulaire   # _ = protégé par CONVENTION
        self.__solde = solde_initial   # __ = name mangling (renommé en _CompteBancaire__solde)

    @property
    def solde(self) -> float:
        return self.__solde

    @property
    def titulaire(self) -> str:
        return self._titulaire

    def deposer(self, montant: float):
        if montant <= 0:
            raise ValueError("Montant invalide")
        self.__solde += montant

    def retirer(self, montant: float) -> bool:
        if montant > self.__solde:
            return False
        self.__solde -= montant
        return True
```

### 🔑 Comparaison des mécanismes

| Langage | Mécanisme | Force | Particularité |
|---------|-----------|-------|---------------|
| **C#** | Propriétés `{ get; set; }` | Forte (compilateur) | Syntaxe la plus élégante |
| **Java** | Getters/Setters manuels | Forte (compilateur) | Verbeux mais standard |
| **C++** | Getters/Setters + `const` | Forte (compilateur) | `const` = garantie supplémentaire |
| **Python** | `@property` / `_` / `__` | Faible (convention) | "We're all consenting adults" |

### ⚠️ Pièges courants

**Piège 1 — Tout en public "pour aller plus vite" :** Dette technique garantie. Le jour où tu veux ajouter une validation, tu modifies 50 fichiers.

**Piège 2 — Python `__` = fausse sécurité :** `obj._Classe__attribut` y accède quand même. C'est une convention, pas un mur.

**Piège 3 — Getters/setters sans logique :** `getSolde()` + `setSolde()` sans validation = attribut public déguisé. L'encapsulation a du sens seulement avec du contrôle.

---

## 1.3 Héritage

> **Concept :** Une classe enfant **hérite** des attributs et méthodes d'une classe parent, et peut les étendre ou les modifier.

### 🎯 Pourquoi c'est important ?

L'héritage résout la **duplication de code** entre classes similaires. Si `Chien`, `Chat`, `Oiseau` partagent `nom`, `age`, `manger()`, écrire ça 3 fois est une source de bugs. L'héritage factorise le commun dans `Animal`, et chaque enfant ne définit que le spécifique.

**Bénéfices :** Réutilisation du code, hiérarchie naturelle ("est-un"), extensibilité (ajouter `Poisson` sans toucher l'existant).

**Attention :** L'héritage est puissant mais dangereux si mal utilisé (voir 1.6 Composition vs Héritage).

```
        Animal (parent / superclasse)
       /       \
    Chien     Chat  (enfants / sous-classes)
```

### 💻 Code — Animal → Chien / Chat

### 🔷 C#
```csharp
public class Animal
{
    public string Nom { get; protected set; }
    public Animal(string nom) { Nom = nom; }

    // virtual = cette méthode PEUT être redéfinie par les enfants
    public virtual string Parler() => "...";
}

public class Chien : Animal  // : = hérite de
{
    public string Race { get; }
    public Chien(string nom, string race) : base(nom) { Race = race; } // base = parent

    public override string Parler() => "Woof!"; // override = redéfinition
}

public class Chat : Animal
{
    public Chat(string nom) : base(nom) {}
    public override string Parler() => "Miaou!";
}

Animal rex = new Chien("Rex", "Berger");
Console.WriteLine($"{rex.Nom} dit {rex.Parler()}"); // Rex dit Woof!
```

### 🟠 Java
```java
public class Animal {
    protected String nom;
    public Animal(String nom) { this.nom = nom; }

    // En Java, toutes les méthodes sont redéfinissables par défaut
    public String parler() { return "..."; }
}

public class Chien extends Animal {  // extends = hérite de
    private String race;
    public Chien(String nom, String race) {
        super(nom);  // super = appel parent
        this.race = race;
    }

    @Override  // annotation recommandée
    public String parler() { return "Woof!"; }
}
```

### 🔵 C++
```cpp
class Animal {
protected:
    std::string nom_;
public:
    Animal(const std::string& nom) : nom_(nom) {}
    virtual std::string parler() const { return "..."; }
    virtual ~Animal() = default; // ⚠️ destructeur virtuel OBLIGATOIRE
};

class Chien : public Animal {  // : public = hérite publiquement
    std::string race_;
public:
    Chien(const std::string& nom, const std::string& race)
        : Animal(nom), race_(race) {}
    std::string parler() const override { return "Woof!"; }
};
```

### 🟡 Python
```python
class Animal:
    def __init__(self, nom: str):
        self.nom = nom
    def parler(self) -> str:  # redéfinissable par défaut
        return "..."

class Chien(Animal):  # () = hérite de
    def __init__(self, nom: str, race: str):
        super().__init__(nom)  # super() = appel parent
        self.race = race
    def parler(self) -> str:  # redéfinition automatique
        return "Woof!"
```

### 🔑 Comparaison syntaxique

| Action | C# | Java | C++ | Python |
|--------|-----|------|-----|--------|
| Hériter | `: Parent` | `extends Parent` | `: public Parent` | `(Parent)` |
| Appel parent | `base(...)` | `super(...)` | `Parent(...)` | `super().__init__(...)` |
| Redéfinissable | `virtual` requis | par défaut | `virtual` requis | par défaut |
| Redéfinir | `override` | `@Override` | `override` | automatique |
| Bloquer | `sealed` | `final` | `final` | — |

### ⚠️ Pièges

- **C++ :** Oublier le destructeur virtuel = fuite mémoire en détruisant via un pointeur parent.
- **Héritage en diamant :** C++ → `virtual inheritance`. Python → MRO. C#/Java → interdit (interfaces seulement).
- **Abus d'héritage :** Préfère la composition quand il n'y a pas de vraie relation "est-un" (voir 1.6).

---

## 1.4 Polymorphisme

> **Concept :** Un même appel de méthode produit un comportement différent selon le type réel de l'objet.

### 🎯 Pourquoi c'est le concept le plus puissant ?

Le polymorphisme te permet d'écrire du code **générique** qui fonctionne avec des types que tu ne connais même pas encore.

**Sans polymorphisme :**
```python
# ❌ Chaque nouveau type = modifier le code
if isinstance(forme, Cercle): return 3.14 * forme.rayon ** 2
elif isinstance(forme, Rectangle): return forme.largeur * forme.hauteur
# Et si j'ajoute un Triangle ? Encore un elif...
```

**Avec polymorphisme :**
```python
# ✅ Le code ne change JAMAIS, même avec de nouveaux types
return forme.aire()  # Chaque forme sait calculer sa propre aire
```

**Analogie :** Un chef d'orchestre fait un geste "jouez". Chaque musicien interprète différemment selon son instrument. Le chef n'a pas besoin de dire "violon, fais ceci, flûte, fais cela".

### 📚 Les 3 types de polymorphisme

| Type | Nom | Quand ? | Exemple |
|------|-----|---------|---------|
| **Sous-type** | Override | Runtime | `forme.aire()` → Cercle ou Rectangle |
| **Surcharge** | Overload | Compilation | `add(int, int)` vs `add(double, double)` |
| **Paramétrique** | Generics | Compilation | `List<int>` vs `List<string>` |

### 💻 Code — Polymorphisme en action

### 🔷 C#
```csharp
public abstract class Forme
{
    public abstract double Aire(); // les enfants DOIVENT implémenter
}

public class Cercle : Forme
{
    public double Rayon { get; }
    public Cercle(double rayon) => Rayon = rayon;
    public override double Aire() => Math.PI * Rayon * Rayon;
}

public class Rectangle : Forme
{
    public double Largeur { get; }
    public double Hauteur { get; }
    public Rectangle(double l, double h) { Largeur = l; Hauteur = h; }
    public override double Aire() => Largeur * Hauteur;
}

// ✅ Ce code fonctionne avec TOUTE forme — existante ou future
void AfficherAires(List<Forme> formes)
{
    foreach (var forme in formes)
        Console.WriteLine($"Aire = {forme.Aire():F2}");
        // Le RUNTIME appelle la bonne méthode selon le type réel
}
```

### 🟡 Python
```python
from abc import ABC, abstractmethod
import math

class Forme(ABC):
    @abstractmethod
    def aire(self) -> float: ...

class Cercle(Forme):
    def __init__(self, rayon: float): self.rayon = rayon
    def aire(self) -> float: return math.pi * self.rayon ** 2

class Rectangle(Forme):
    def __init__(self, l: float, h: float): self.largeur, self.hauteur = l, h
    def aire(self) -> float: return self.largeur * self.hauteur

# Python a aussi le "duck typing" : si ça a .aire(), ça marche
def afficher_aires(formes: list[Forme]):
    for f in formes:
        print(f"Aire = {f.aire():.2f}")
```

### 🔑 Résumé

| Langage | Polymorphisme auto ? | Nécessite `virtual` ? |
|---------|---------------------|----------------------|
| C# | si `virtual`/`abstract` | ✅ |
| Java | par défaut | ❌ |
| C++ | si `virtual` | ✅ |
| Python | toujours (duck typing) | ❌ |

### ⚠️ Pièges

- **Oublier `virtual` (C#/C++) :** La méthode du PARENT est appelée au lieu de l'enfant.
- **if/else sur le type au lieu du polymorphisme :** Anti-pattern classique. Utilise `forme.Aire()` pas `if (forme is Cercle)`.
- **C++ "slicing" :** Passer un enfant par valeur = le polymorphisme est perdu. Utilise des pointeurs/références.

---

## 1.5 Abstraction & Interfaces

> **Concept :** Définir un **contrat** (ce que doit faire un objet) sans imposer le comment.

### 🎯 Pourquoi c'est important ?

L'abstraction sépare le **"quoi"** du **"comment"**. Quand tu utilises un `ILogger`, tu sais appeler `Log(message)` sans savoir si ça écrit dans un fichier, un email, ou une console. Cela permet la **testabilité** (mock), la **flexibilité** (changer d'implémentation), et le **travail d'équipe** (accord sur l'interface, travail parallèle).

**Analogie :** Une prise électrique est une interface. Le contrat : 220V, type E/F. Ton aspirateur et ta lampe respectent ce contrat sans savoir si l'électricité vient du nucléaire ou du solaire.

### 📚 Classe abstraite vs Interface

| | Classe Abstraite | Interface |
|---|---|---|
| Code concret ? | Oui | Non (sauf méthodes par défaut) |
| Héritage multiple ? | Non (1 seule) | Oui (plusieurs) |
| Constructeur ? | Oui | Non |
| Quand ? | Base commune + logique partagée | Contrat pur, capacité transversale |
| Relation ? | "EST-UN" | "PEUT-FAIRE" |

### 💻 Code complet (C# — les autres langages suivent la même logique)

```csharp
// INTERFACE = contrat pur ("ce que l'objet PEUT FAIRE")
public interface IStockable
{
    void Sauvegarder(string chemin);
    void Charger(string chemin);
}

public interface IExportable { byte[] ExporterPdf(); }

// CLASSE ABSTRAITE = base commune ("ce que l'objet EST" + code partagé)
public abstract class Document : IStockable
{
    public string Titre { get; set; }
    protected Document(string titre) { Titre = titre; }

    public string GetResume() => $"{Titre} — document"; // concrète (partagée)
    public abstract int CompterMots();                   // abstraite (à implémenter)
    public abstract void Sauvegarder(string chemin);
    public abstract void Charger(string chemin);
}

// CONCRÈTE — implémente tout : 1 abstraite + N interfaces
public class Rapport : Document, IExportable
{
    private string _contenu;
    public Rapport(string titre, string contenu) : base(titre) { _contenu = contenu; }

    public override int CompterMots() => _contenu.Split(' ').Length;
    public override void Sauvegarder(string chemin) { /* ... */ }
    public override void Charger(string chemin) { /* ... */ }
    public byte[] ExporterPdf() { return Array.Empty<byte>(); }
}
```

### 🔑 Règle de décision

```
"EST-UN type de base avec du code commun ?" → Classe abstraite
"PEUT-FAIRE quelque chose (capacité) ?"     → Interface
"Besoin de plusieurs contrats ?"             → Interfaces (héritage multiple)
```

---

## 1.6 Composition vs Héritage

> **Concept :** Préférer "**a-un**" (composition) à "**est-un**" (héritage) pour un code plus flexible.

### 🎯 Pourquoi c'est important ?

L'héritage crée un **couplage fort** : toute modification du parent se propage aux enfants. La composition assemble des **composants indépendants** remplaçables, combinables et testables séparément.

**Le problème de l'héritage :**
```
         Employe → Manager → ManagerTechnique
                 → Developpeur
         Et si ManagerTechnique est aussi Developpeur ? → Explosion combinatoire
```

**La solution par composition :**
```
Employe ──HAS──→ Role[] (combinable à volonté)
        ──HAS──→ Competences[] (modifiable dynamiquement)
```

**Analogie :** Un PC = composition pure. Tu combines carte mère + CPU + GPU + RAM. Tu changes la GPU sans toucher au reste. Si les PC étaient basés sur l'héritage, changer un composant = tout reconstruire.

### 💻 Code — Notification par composition (C#)

```csharp
public interface IMessageSender { void Envoyer(string dest, string msg); }
public interface IMessageFormatter { string Formater(string msg); }

public class EmailSender : IMessageSender { /* ... */ }
public class SmsSender : IMessageSender { /* ... */ }
public class HtmlFormatter : IMessageFormatter { /* ... */ }
public class PlainFormatter : IMessageFormatter { /* ... */ }

// Composition : assemble les comportements via injection
public class NotificationService
{
    private readonly IMessageSender _sender;
    private readonly IMessageFormatter _formatter;

    public NotificationService(IMessageSender sender, IMessageFormatter formatter)
    { _sender = sender; _formatter = formatter; }

    public void Notifier(string dest, string msg)
    { _sender.Envoyer(dest, _formatter.Formater(msg)); }
}

// 3 senders × 2 formatters = 6 combinaisons SANS écrire 6 classes
var emailHtml = new NotificationService(new EmailSender(), new HtmlFormatter());
var smsPlain = new NotificationService(new SmsSender(), new PlainFormatter());
```

### 🔑 Arbre de décision

```
1. Vraie relation "est-un" PERMANENTE ?       → Héritage OK
2. Comportement qui peut changer/se combiner ? → Composition
3. Besoin de plusieurs fonctionnalités ?       → Composition
4. Doute ?                                     → Composition. Toujours.
```

---

# PARTIE 2 — Principes SOLID

## Sommaire Partie 2

| # | Principe | En une phrase |
|---|----------|---------------|
| 2.1 | [S — Single Responsibility](#21-s--single-responsibility-principle) | Une classe = une seule raison de changer |
| 2.2 | [O — Open/Closed](#22-o--openclosed-principle) | Ouvert à l'extension, fermé à la modification |
| 2.3 | [L — Liskov Substitution](#23-l--liskov-substitution-principle) | Un enfant remplace son parent sans surprise |
| 2.4 | [I — Interface Segregation](#24-i--interface-segregation-principle) | Plusieurs petites interfaces > une grosse |
| 2.5 | [D — Dependency Inversion](#25-d--dependency-inversion-principle) | Dépendre d'abstractions, pas de concretions |

### Pourquoi SOLID ?

Formulés par Robert C. Martin (Uncle Bob), les principes SOLID synthétisent des décennies d'expérience. Sans SOLID, les projets suivent souvent cette trajectoire : mois 1 → tout va bien. Mois 6 → ajouter une feature prend 3× plus de temps. Mois 12 → personne n'ose toucher à certaines classes. Mois 18 → "il faut tout réécrire".

**SOLID = boussole, pas religion.** Sur un petit script, abstraire partout est de l'over-engineering. Sur un projet d'équipe de 6 mois+, ignorer SOLID mène au chaos.

---

## 2.1 S — Single Responsibility Principle

> **Une classe ne doit avoir qu'une seule raison de changer.**

Si ta classe fait 3 choses (logique métier + accès BDD + formatage), un changement de schéma BDD te force à toucher une classe contenant aussi la logique métier. SRP dit : sépare pour localiser les changements.

### ❌ Violation
```python
class Employe:
    def calculer_salaire(self): ...   # modifié par le comptable
    def sauvegarder_bdd(self): ...    # modifié par le DBA
    def generer_rapport(self): ...    # modifié par le designer
```

### ✅ Correct
```python
class Employe:               # logique métier
    def calculer_salaire(self): ...
class EmployeRepository:     # persistence
    def sauvegarder(self, e): ...
class EmployeRapport:        # présentation
    def generer(self, e): ...
```

---

## 2.2 O — Open/Closed Principle

> **Ouvert à l'extension, fermé à la modification.**

Ajouter un nouveau comportement = ajouter du code (nouvelle classe), PAS modifier du code existant.

### ❌ Violation
```csharp
public double Calculer(string type, double montant) {
    if (type == "VIP") return montant * 0.2;
    if (type == "Premium") return montant * 0.1;
    // Chaque nouveau type = modifier ici ❌
}
```

### ✅ Correct
```csharp
public interface IRemise { double Calculer(double montant); }
public class RemiseVIP : IRemise { public double Calculer(double m) => m * 0.2; }
public class RemisePremium : IRemise { public double Calculer(double m) => m * 0.1; }
// Ajouter RemiseEtudiant ? Nouvelle classe, ZERO modification ✅
```

---

## 2.3 L — Liskov Substitution Principle

> **Un enfant doit pouvoir remplacer son parent sans casser le programme.**

### ❌ Violation classique
```java
Rectangle r = new Carre();
r.setLargeur(5);
r.setHauteur(3);
assert r.aire() == 15; // FAIL → 9 car Carre force largeur = hauteur
```

### ✅ Correct — modéliser autrement
```java
public interface Forme { int aire(); }
public class Rectangle implements Forme { /* ... */ }
public class Carre implements Forme { /* ... */ }
```

**Leçon :** "est-un" dans le monde réel ≠ "est-un" en POO mutable.

---

## 2.4 I — Interface Segregation Principle

> **Ne pas forcer à implémenter des méthodes inutiles.**

### ❌ Violation
```cpp
class IMachine { // interface trop grosse
    virtual void imprimer() = 0;
    virtual void scanner() = 0;  // inutile pour une imprimante simple
    virtual void faxer() = 0;    // inutile aussi
};
```

### ✅ Correct — interfaces séparées
```cpp
class IImprimable { virtual void imprimer() = 0; };
class IScannable  { virtual void scanner() = 0; };
class IFaxable    { virtual void faxer() = 0; };

class ImprimanteSimple : public IImprimable { /* que imprimer */ };
class MultiFunction : public IImprimable, public IScannable, public IFaxable { /* tout */ };
```

---

## 2.5 D — Dependency Inversion Principle

> **Dépendre d'abstractions, pas de concretions.**

La logique métier définit une interface, l'infrastructure l'implémente. Résultat : changer de BDD = modifier UNE classe.

### ❌ Violation
```python
class UserService:
    def __init__(self):
        self.db = MySQLDatabase()  # couplage direct
```

### ✅ Correct
```python
class Database(ABC):
    @abstractmethod
    def save(self, data): ...

class UserService:
    def __init__(self, db: Database):  # abstraction
        self.db = db

service = UserService(PostgresDatabase())     # production
test_service = UserService(InMemoryDatabase()) # tests
```

### 🔑 Résumé SOLID

```
S → Localise les changements      O → Protège le code existant
L → Garantit la substitution      I → Évite le code inutile
D → Découple haut et bas niveau

= Code testable, extensible, maintenable, résistant au changement.
```

---

# PARTIE 3 — Design Patterns

## Sommaire Partie 3

| # | Catégorie | Pattern | Problème résolu |
|---|-----------|---------|----------------|
| 3.1 | Créationnel | [Singleton](#31-singleton) | Instance unique garantie |
| 3.2 | Créationnel | [Factory Method](#32-factory-method) | Déléguer la création |
| 3.3 | Créationnel | [Abstract Factory](#33-abstract-factory) | Familles d'objets cohérents |
| 3.4 | Créationnel | [Builder](#34-builder) | Construction pas à pas |
| 3.5 | Structurel | [Adapter](#35-adapter) | Rendre compatible deux interfaces |
| 3.6 | Structurel | [Decorator](#36-decorator) | Ajouter des comportements dynamiquement |
| 3.7 | Structurel | [Facade](#37-facade) | Simplifier un sous-système complexe |
| 3.8 | Structurel | [Proxy](#38-proxy) | Contrôler l'accès à un objet |
| 3.9 | Comportemental | [Observer](#39-observer) | Notifier d'un changement |
| 3.10 | Comportemental | [Strategy](#310-strategy) | Algorithme interchangeable |
| 3.11 | Comportemental | [Command](#311-command) | Encapsuler une action (undo/redo) |
| 3.12 | Comportemental | [State](#312-state) | Comportement selon l'état |

### Qu'est-ce qu'un Design Pattern ?

Les Design Patterns sont des **solutions éprouvées à des problèmes récurrents**, popularisés par le "Gang of Four" (Gamma, Helm, Johnson, Vlissides) en 1994.

**Ce que les patterns NE sont PAS :** du code à copier-coller aveuglément.
**Ce que les patterns SONT :** un vocabulaire commun + des guides éprouvés.

**Règle d'or :** Identifie d'abord le PROBLÈME, puis choisis le pattern. Jamais l'inverse.

---

## 3.1 Singleton

> **Une seule instance** dans tout le programme + accès global.

### 🎯 Quand ? Config, pool de connexions, cache, logger centralisé.

### 🔷 C#
```csharp
public sealed class Configuration
{
    private static readonly Lazy<Configuration> _instance = new(() => new Configuration());
    public static Configuration Instance => _instance.Value;
    private Configuration() { }
    public string AppName { get; set; } = "MonApp";
}
```

### 🟠 Java
```java
public class Configuration {
    private static final Configuration INSTANCE = new Configuration();
    private Configuration() {}
    public static Configuration getInstance() { return INSTANCE; }
}
```

### 🔵 C++
```cpp
class Configuration {
public:
    static Configuration& getInstance() {
        static Configuration instance; // thread-safe C++11+
        return instance;
    }
    Configuration(const Configuration&) = delete;
private:
    Configuration() {}
};
```

### 🟡 Python
```python
class Configuration:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.app_name = "MonApp"
        return cls._instance
# Alternative : un module Python EST déjà un singleton
```

> ⚠️ Souvent un **anti-pattern** (état global, tests difficiles). Préfère l'injection de dépendances.

---

## 3.2 Factory Method

> Délègue la création sans que l'appelant connaisse la classe concrète.

### 🎯 Quand ? Créer le bon type de connexion, parser, document selon une config/paramètre.

### 🔷 C#
```csharp
public interface INotification { void Envoyer(string message); }

public static class NotificationFactory
{
    public static INotification Creer(string type) => type switch
    {
        "email" => new EmailNotification(),
        "sms"   => new SmsNotification(),
        _ => throw new ArgumentException($"Type inconnu: {type}")
    };
}

INotification notif = NotificationFactory.Creer("email");
notif.Envoyer("Bienvenue!");
```

### 🟡 Python
```python
class NotificationFactory:
    _registry = {"email": EmailNotification, "sms": SmsNotification}

    @classmethod
    def creer(cls, type_: str) -> Notification:
        return cls._registry[type_]()

    @classmethod
    def enregistrer(cls, type_: str, classe: type):
        cls._registry[type_] = classe  # extensible (OCP)
```

---

## 3.3 Abstract Factory

> Créer des **familles** d'objets liés et cohérents (ex: thème UI Dark = boutons sombres + inputs sombres).

### 🎯 Quand ? Thème UI, support multi-plateforme, base de données multi-vendor.

### 🔷 C#
```csharp
public interface IThemeFactory
{
    IBouton CreerBouton();
    IInput CreerInput();
}

public class DarkThemeFactory : IThemeFactory
{
    public IBouton CreerBouton() => new BoutonDark();
    public IInput CreerInput() => new InputDark();
}

// Le code UI est découplé du thème concret
void ConstruireUI(IThemeFactory factory)
{
    factory.CreerBouton().Afficher();
    factory.CreerInput().Afficher();
}
```

---

## 3.4 Builder

> Construire un objet complexe **étape par étape** avec une API fluide et lisible.

### 🎯 Quand ? Objet avec beaucoup de paramètres optionnels (requêtes HTTP, configs, documents).

### 🔷 C#
```csharp
var req = new RequeteHttpBuilder()
    .Url("https://api.exemple.com/users")
    .Post()
    .Header("Authorization", "Bearer xxx")
    .Body("{\"name\": \"Alice\"}")
    .Timeout(3000)
    .Build();

// Comparé à : new Requete("https://...", "POST", null, headers, body, 3000, null, true)
// → Le Builder est auto-documenté et impossible à confondre
```

### 🟡 Python
```python
req = (RequeteHttpBuilder()
    .url("https://api.exemple.com/users")
    .post()
    .header("Authorization", "Bearer xxx")
    .body('{"name": "Alice"}')
    .build())
```

---

## 3.5 Adapter

> Rendre **compatibles** deux interfaces incompatibles.

### 🎯 Quand ? Intégrer une librairie tierce, adapter une API legacy.

**Analogie :** Adaptateur de prise US → EU.

### 🔷 C#
```csharp
public interface ILogger { void Log(string message); }

public class LibExterneLogger  // API différente
{ public void WriteEntry(string text, int severity) { /* ... */ } }

public class LoggerAdapter : ILogger  // traduit
{
    private readonly LibExterneLogger _lib;
    public LoggerAdapter(LibExterneLogger lib) => _lib = lib;
    public void Log(string message) => _lib.WriteEntry(message, 1);
}
```

---

## 3.6 Decorator

> Ajouter des responsabilités **dynamiquement** en empilant des couches.

### 🎯 Quand ? Chiffrement + compression, middleware HTTP, streams Java.

**Analogie :** Vêtements. T-shirt (base) + pull (decorator 1) + manteau (decorator 2). Chaque couche ajoute sans modifier.

### 🔷 C#
```csharp
IDataSource source = new FileDataSource();
source = new EncryptionDecorator(source);  // +chiffrement
source = new CompressionDecorator(source); // +compression

source.Write("données sensibles");
// → Écriture: 📦[🔒[données sensibles]]
```

---

## 3.7 Facade

> **Interface simplifiée** pour un sous-système complexe.

### 🎯 Quand ? Lecteur multimédia, ORM, SDK client.

**Analogie :** "Hey Siri, bonne nuit" = éteint lumières + verrouille portes + baisse thermostat.

### 🔷 C#
```csharp
public class MediaPlayerFacade
{
    public void Lire(string fichier)
    {
        _video.Decode(fichier);
        _audio.Decode(fichier);
        _subs.Load(fichier.Replace(".mp4", ".srt"));
        _screen.Render();
    }
}

var player = new MediaPlayerFacade();
player.Lire("film.mp4"); // un seul appel = tout coordonné
```

---

## 3.8 Proxy

> Contrôler l'accès à un objet de manière **transparente** — cache, auth, lazy loading.

### 🎯 Quand ? Cache API, contrôle d'accès, logging transparent.

### 🔷 C#
```csharp
IServiceApi api = new ServiceApiCacheProxy(new ServiceApiReel());
api.GetDonnees("/users");  // 🌐 Appel réseau (1ère fois)
api.GetDonnees("/users");  // ⚡ Cache (2ème fois — instantané)
```

---

## 3.9 Observer

> Notifier automatiquement tous les **abonnés** quand un objet change d'état.

### 🎯 Quand ? Events DOM, pub/sub, data binding (React state, Vue reactivity), notifications push.

**Analogie :** Abonnement YouTube. Le créateur publie → tous les abonnés sont notifiés.

### 🔷 C#
```csharp
public class PrixProduit
{
    private decimal _prix;
    public event Action<decimal>? PrixChange; // event natif C#

    public decimal Prix {
        get => _prix;
        set { _prix = value; PrixChange?.Invoke(value); }
    }
}

var produit = new PrixProduit();
produit.PrixChange += prix => Console.WriteLine($"🖥️ Web: {prix}€");
produit.PrixChange += prix => Console.WriteLine($"📧 Email: {prix}€");
produit.Prix = 29.99m; // → notifie les deux
```

### 🟡 Python
```python
class PrixProduit:
    def __init__(self):
        self._prix, self._observers = 0.0, []

    def ajouter_observer(self, obs): self._observers.append(obs)

    @property
    def prix(self): return self._prix

    @prix.setter
    def prix(self, value):
        self._prix = value
        for obs in self._observers: obs(value)
```

---

## 3.10 Strategy

> Famille d'**algorithmes interchangeables** à la volée.

### 🎯 Quand ? Tri, calcul de prix, compression, validation, routage GPS.

**Analogie :** GPS — même destination, tu choisis la stratégie (rapide, court, sans péage).

### 🔷 C#
```csharp
public class DataProcessor
{
    private ITriStrategy _strategy;
    public DataProcessor(ITriStrategy s) => _strategy = s;
    public void SetStrategy(ITriStrategy s) => _strategy = s; // changeable !
    public List<int> Traiter(List<int> data) => _strategy.Trier(data);
}

var proc = new DataProcessor(new TriRapide());
proc.SetStrategy(new TriBulle()); // switch dynamique
```

---

## 3.11 Command

> Encapsuler une action en objet — **undo/redo**, file d'attente, historique.

### 🎯 Quand ? Ctrl+Z dans un éditeur, job queue, transactions avec rollback, macros.

**Analogie :** Ticket de commande au restaurant. Le serveur ne cuisine pas — il écrit sur un ticket, le passe au chef. Le ticket peut être annulé, modifié, mis en attente.

### 🔷 C#
```csharp
public interface ICommand { void Executer(); void Annuler(); }

public class EditeurTexte
{
    private readonly Stack<ICommand> _historique = new();

    public void Executer(ICommand cmd) { cmd.Executer(); _historique.Push(cmd); }
    public void Annuler() { if (_historique.TryPop(out var cmd)) cmd.Annuler(); }
}
```

---

## 3.12 State

> Comportement qui **change selon l'état** interne — automate à états finis.

### 🎯 Quand ? Commande e-commerce (EnAttente → Expédiée → Livrée), lecteur audio, connexion réseau, personnage de jeu.

**Analogie :** Feu tricolore. Quand vert → les voitures passent. Quand rouge → elles s'arrêtent. Le feu change d'état, le comportement suit.

### 🔷 C#
```csharp
public class Commande
{
    public IEtatCommande Etat { get; set; } = new EnAttente();
    public void Avancer() => Etat.Suivant(this); // délègue à l'état courant
    public void Afficher() => Etat.Afficher();
}

var cmd = new Commande();
cmd.Afficher();                    // ⏳ En attente
cmd.Avancer(); cmd.Afficher();     // 📦 En préparation
cmd.Avancer(); cmd.Afficher();     // 🚚 Expédiée
cmd.Avancer(); cmd.Afficher();     // ✅ Livrée
```

### 🔑 Résumé — Quel pattern pour quel problème ?

```
CRÉATIONNELS — Comment créer ?
  Instance unique                    → Singleton (prudence)
  Créer sans connaître le type       → Factory
  Familles cohérentes                → Abstract Factory
  Objet complexe, plein de params    → Builder

STRUCTURELS — Comment organiser ?
  Interface incompatible             → Adapter
  Ajouter des comportements          → Decorator
  Simplifier un sous-système         → Facade
  Contrôler l'accès                  → Proxy

COMPORTEMENTAUX — Comment communiquer ?
  Notifier des abonnés               → Observer
  Algorithme interchangeable         → Strategy
  Undo/Redo, historique              → Command
  Comportement selon l'état          → State
```

---

# PARTIE 4 — Architecture Logicielle

## Sommaire Partie 4

| # | Architecture | Idée centrale | Taille projet |
|---|-------------|---------------|---------------|
| 4.1 | [MVC](#41-mvc--model-view-controller) | Séparer données, logique, affichage | Petit → Moyen |
| 4.2 | [Clean Architecture](#42-clean-architecture) | Dépendances vers le centre | Moyen → Grand |
| 4.3 | [Hexagonale](#43-architecture-hexagonale-ports--adapters) | Le domaine ne dépend de rien | Moyen → Grand |
| 4.4 | [Microservices](#44-microservices) | Services indépendants | Grand → Très grand |

### Pourquoi l'architecture logicielle ?

Les Design Patterns traitent du **micro** (quelques classes). L'architecture traite du **macro** (les grandes couches d'un projet entier).

Sans architecture, un projet qui grandit devient un "Big Ball of Mud" — tout dépend de tout, la base de données est appelée depuis l'UI, la logique est mélangée au HTML. Ajouter une feature prend toujours plus de temps. Tester devient impossible.

L'architecture définit des **frontières** entre les parties du système avec des règles de dépendances claires.

---

## 4.1 MVC — Model View Controller

> Séparer en **3 responsabilités** : données, affichage, orchestration.

```
┌──────────┐     ┌────────────┐     ┌──────────┐
│   View   │ ←── │ Controller │ ──→ │  Model   │
│ (affiche)│     │ (orchestre)│     │ (données)│
└──────────┘     └────────────┘     └──────────┘
```

| Composant | Rôle | Exemples |
|-----------|------|---------|
| **Model** | Données + logique métier | `User`, `Order`, calculs |
| **View** | Affichage uniquement | Template HTML, React, console |
| **Controller** | Reçoit requêtes, coordonne | Route HTTP, CLI |

**Où on le retrouve :** ASP.NET MVC, Spring MVC, Django (variante MTV), Rails, Laravel.

**Limites :** Sur des projets complexes, le Controller devient un "God Controller". Le Model mélange parfois logique et accès données. → Clean Architecture résout ça.

---

## 4.2 Clean Architecture

> **Règle d'or :** Les dépendances pointent vers l'**intérieur**. Le domaine ne dépend de RIEN d'externe.

Formulée par Robert C. Martin : le code métier est le plus important et le plus stable. Les frameworks, BDD, API changent — les règles métier restent.

```
┌───────────────────────────────────────────┐
│           Frameworks & Drivers            │  ← BD, Web, UI
│  ┌─────────────────────────────────────┐  │
│  │       Interface Adapters            │  │  ← Controllers, Repos
│  │  ┌───────────────────────────────┐  │  │
│  │  │      Application Layer       │  │  │  ← Use Cases
│  │  │  ┌─────────────────────────┐  │  │  │
│  │  │  │    Domain / Entities    │  │  │  │  ← Règles métier PURES
│  │  │  └─────────────────────────┘  │  │  │
│  │  └───────────────────────────────┘  │  │
│  └─────────────────────────────────────┘  │
└───────────────────────────────────────────┘
```

**Bénéfices concrets :**
1. **Testabilité :** Use Case testé avec un `InMemoryRepository`.
2. **Indépendance techno :** PostgreSQL → MongoDB = modifier UNE classe.
3. **Clarté :** Chaque couche a un rôle clair.

### Structure de dossiers
```
src/
├── domain/          # 0 dépendance externe (entités + ports/interfaces)
├── application/     # Use cases (dépend du domain uniquement)
├── infrastructure/  # Implémentations concrètes (BDD, API, email)
└── presentation/    # UI, Controllers, CLI
```

---

## 4.3 Architecture Hexagonale (Ports & Adapters)

> Le domaine communique via des **ports** (interfaces) et des **adapters** (implémentations).

Formulée par Alistair Cockburn. Même philosophie que Clean Architecture, vocabulaire plus concret.

```
   Adapter HTTP ──→ Port Primaire (input)
   Adapter CLI ───→       ┌───────────┐
                          │  DOMAINE  │
                          │  MÉTIER   │
                          └───────────┘
                    Port Secondaire (output) ──→ Adapter PostgreSQL
                                               ──→ Adapter Email SMTP
```

| Concept | Rôle | Exemple |
|---------|------|---------|
| **Port primaire** | Ce que le domaine offre | `CommandeService.creer(...)` |
| **Port secondaire** | Ce dont le domaine a besoin | `ICommandeRepository` |
| **Adapter primaire** | Implémente l'entrée | REST Controller, CLI |
| **Adapter secondaire** | Implémente la sortie | PostgresRepo, SendGrid |

---

## 4.4 Microservices

> **Services indépendants**, chacun responsable d'un domaine métier, déployable séparément.

```
┌──────────────┐    HTTP/gRPC    ┌──────────────┐
│  Service     │ ──────────────→ │  Service     │
│  Utilisateur │ ←── Queue ───→ │  Commande    │
│  (sa BDD)    │                 │  (sa BDD)    │
└──────────────┘                 └──────────────┘
```

| Avantage | Inconvénient |
|----------|-------------|
| Déploiement indépendant | Complexité réseau |
| Scalabilité ciblée | Debugging distribué |
| Équipes autonomes | Cohérence des données |
| Liberté techno par service | Infrastructure lourde (K8s) |

### Règles clés

1. **1 service = 1 responsabilité métier** (Bounded Context).
2. **Chaque service = sa propre BDD** (sinon c'est un monolithe distribué).
3. **Communication async** (events Kafka/RabbitMQ) quand possible.
4. **API Gateway** comme point d'entrée unique.
5. **Commence MONOLITHE, découpe ENSUITE** quand la douleur l'exige.

> Martin Fowler : "Don't start with microservices. Start with a monolith."

---

## 🔑 Quelle architecture choisir ?

```
Projet perso / MVP          → MVC simple
Moyen (3-10 devs, > 6 mois) → Clean / Hexagonale
Large (multi-équipes, scale) → Microservices (si compétences DevOps)

🏆 "Commence simple, complexifie quand la DOULEUR l'exige."
```

### ⚠️ Erreurs classiques

- **Over-engineering :** Clean Architecture sur un script de 200 lignes.
- **Microservices prématurés :** Sans maîtriser le monolithe = monolithe distribué (le pire des deux mondes).
- **Mélanger les couches :** SQL dans les entités, HTML dans le contrôleur.
- **Ignorer les tests :** L'architecture propre existe POUR tester facilement.

---

# 🏁 Conclusion

```
Tu as maintenant les clés pour :

1. ✅ Modéliser proprement avec la POO
2. ✅ Écrire du code maintenable avec SOLID
3. ✅ Résoudre des problèmes récurrents avec les Design Patterns
4. ✅ Structurer tes projets avec la bonne architecture

La théorie sans pratique est inutile.
Prochaine étape : prends un projet, applique ces concepts.
Commence simple. Itère. Apprends de tes erreurs.

"Le meilleur code est celui qui résout le problème le plus simplement possible."
```
