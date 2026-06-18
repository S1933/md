# Formation Go pour développeurs PHP — Backend / BFF

> Objectif : apprendre Go progressivement en partant de notions connues côté PHP, avec beaucoup d’exemples de code et des équivalents PHP à chaque étape.

---

## Sommaire

1. [Introduction : Go vs PHP](#1-introduction--go-vs-php)
2. [Fichiers, modules, librairies et fonctionnement de Go](#2-fichiers-modules-librairies-et-fonctionnement-de-go)
3. [Variables, types, tableaux, slices, maps et structs](#3-variables-types-tableaux-slices-maps-et-structs)
4. [Fonctions en Go et équivalents PHP](#4-fonctions-en-go-et-équivalents-php)
5. [Imports, packages et organisation du code](#5-imports-packages-et-organisation-du-code)
6. [Conditions, boucles, loops et range](#6-conditions-boucles-loops-et-range)
7. [Goroutines, channels et concurrence](#7-goroutines-channels-et-concurrence)
8. [Clean architecture pour un BFF](#8-clean-architecture-pour-un-bff)
9. [Création et utilisation des endpoints HTTP](#9-création-et-utilisation-des-endpoints-http)
10. [Décisions techniques pour les endpoints](#10-décisions-techniques-pour-les-endpoints)
11. [Tests unitaires Go et équivalents PHP](#11-tests-unitaires-go-et-équivalents-php)
12. [Projet fil rouge : mini BFF Products API](#12-projet-fil-rouge--mini-bff-products-api)
13. [Exercices en ligne et progression](#13-exercices-en-ligne-et-progression)

---

# 1. Introduction : Go vs PHP

Go et PHP permettent tous les deux de construire des applications web et des API, mais leur modèle d’exécution est différent. En PHP, une requête déclenche généralement l’exécution d’un script par PHP-FPM ou un serveur applicatif. En Go, l’application est compilée en un binaire qui reste actif, écoute les requêtes et conserve certaines ressources pendant toute sa durée de vie.

Cette différence a des conséquences concrètes : une connexion, une variable globale ou une goroutine peut survivre après la fin d’une requête. Il faut donc penser davantage au cycle de vie de l’application, au partage de la mémoire et à l’arrêt propre du serveur.

Les équivalents PHP présentés dans cette formation servent de repères, pas de traductions exactes. Une struct n’est pas une classe, un package n’est pas seulement un namespace et une goroutine n’est pas un processus. L’objectif est de partir de concepts connus pour construire progressivement un modèle mental adapté à Go.

## 1.1 Différences principales

| Sujet | Go | PHP |
|---|---|---|
| Type de langage | Compilé | Interprété |
| Typage | Statique | Dynamique avec typage possible |
| Exécution | Binaire compilé | Runtime PHP/FPM/CLI |
| Gestion des erreurs | `error` explicite | Exceptions, erreurs, warnings |
| Concurrence | Goroutines natives | Généralement via processus, workers, queues, Fibers |
| Organisation | Packages/modules | Namespaces/Composer |
| Tests | `testing` intégré | PHPUnit ou Pest |

---

## 1.2 Premier programme

### Go

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello Go")
}
```

**Explication :** `package main` indique que le package produit un programme exécutable, et `func main()` en est le point d’entrée. Ce bloc assemble les dépendances nécessaires à « Go » puis conserve le processus actif jusqu’à la fin du travail. Comme l’application Go vit plus longtemps qu’une requête PHP-FPM, les ressources créées ici doivent avoir un cycle de vie et un arrêt maîtrisés.

Exécution :

```bash
go run main.go
```

**Explication :** `go run` compile dans un emplacement temporaire puis démarre le programme, ce qui est pratique pendant l’apprentissage. `go build` produit au contraire un binaire réutilisable et déployable sans installer Go sur la machine cible. Dans les deux cas, les erreurs de syntaxe et de types sont détectées avant le démarrage.

Compilation :

```bash
go build -o app
./app
```

**Explication :** `go run` compile dans un emplacement temporaire puis démarre le programme, ce qui est pratique pendant l’apprentissage. `go build` produit au contraire un binaire réutilisable et déployable sans installer Go sur la machine cible. Dans les deux cas, les erreurs de syntaxe et de types sont détectées avant le démarrage.

### PHP équivalent

```php
<?php

echo "Hello PHP\n";
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

Exécution :

```bash
php index.php
```

**Explication :** Cette commande sert à exécuter ou vérifier l’étape décrite dans « PHP équivalent ». Lancez-la depuis la racine du projet afin que Go ou PHP retrouve sa configuration et ses dépendances. Observez la sortie et le code de retour : une commande silencieuse avec un statut nul signifie généralement que l’étape a réussi.

---

## 1.3 Ce qu’il faut retenir

En Go :

- chaque fichier appartient à un `package` ;
- le point d’entrée d’une application exécutable est `package main` + `func main()` ;
- le code est compilé ;
- le formatage est standardisé avec `gofmt` ou `go fmt`.

En PHP :

- un fichier peut être exécuté directement ;
- l’organisation moderne repose souvent sur Composer, namespaces et autoloading ;
- l’exécution passe par PHP CLI, PHP-FPM ou un serveur applicatif.

---

# 2. Fichiers, modules, librairies et fonctionnement de Go

Un projet Go s’organise autour de trois niveaux. Un fichier contient du code source, plusieurs fichiers d’un même dossier forment généralement un package, et un module regroupe les packages appartenant au projet. Le module fournit aussi le chemin utilisé dans les imports.

Le fichier `go.mod` joue un rôle comparable à celui de `composer.json`, mais l’outil `go` couvre également la compilation, les tests, le formatage et la gestion des dépendances. Il est donc au centre du workflow quotidien.

La librairie standard est volontairement riche. Avant d’ajouter une dépendance, il est utile de vérifier si `net/http`, `encoding/json`, `context`, `time` ou un autre package standard répond déjà au besoin. Cette approche limite la complexité du projet et facilite les mises à jour.

## 2.1 Créer un module Go

Un module Go est l’équivalent approximatif d’un projet Composer en PHP.

### Go

```bash
mkdir go-training
cd go-training
go mod init example.com/go-training
```

**Explication :** La commande `go mod init` crée le fichier `go.mod` et fixe le chemin canonique du module. Ce chemin préfixera les imports internes, par exemple `example.com/go-training/internal/product`. Dans un vrai dépôt, utilisez généralement l’URL du dépôt comme nom de module.

Structure :

```text
go-training/
├── go.mod
└── main.go
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « Go ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

Contenu de `go.mod` :

```go
module example.com/go-training

go 1.23
```

**Explication :** La directive `module` fixe l’identité du projet et le préfixe de ses imports internes. La directive `go` indique la version de langage minimale attendue par les outils, sans installer ni verrouiller elle-même le compilateur. Les dépendances externes seront ajoutées dans ce fichier et leurs sommes de contrôle seront conservées dans `go.sum`.

### PHP équivalent

```bash
mkdir php-training
cd php-training
composer init
```

**Explication :** Ces commandes montrent le workflow PHP équivalent autour de Composer. Composer gère principalement les dépendances et l’autoload, alors que la commande `go` regroupe aussi compilation, formatage et tests. Le parallèle aide à se repérer, mais les deux outils n’ont pas exactement le même périmètre.

Structure :

```text
php-training/
├── composer.json
└── src/
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « PHP équivalent ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

Contenu simplifié de `composer.json` :

```json
{
  "name": "example/php-training",
  "autoload": {
    "psr-4": {
      "App\\": "src/"
    }
  }
}
```

**Explication :** Ce `composer.json` configure l’autoload PSR-4 : les classes du namespace `App\\` sont recherchées dans `src/`. Le parallèle avec `go.mod` est utile pour comprendre la racine du projet, mais Go n’utilise pas d’autoloader à l’exécution : les imports sont résolus et vérifiés lors de la compilation.

---

## 2.2 Fichier Go minimal

### Go

```go
package main

import "fmt"

func main() {
    fmt.Println("Application started")
}
```

**Explication :** `package main` indique que le package produit un programme exécutable, et `func main()` en est le point d’entrée. Ce bloc assemble les dépendances nécessaires à « Go » puis conserve le processus actif jusqu’à la fin du travail. Comme l’application Go vit plus longtemps qu’une requête PHP-FPM, les ressources créées ici doivent avoir un cycle de vie et un arrêt maîtrisés.

### PHP équivalent

```php
<?php

echo "Application started\n";
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 2.3 Commandes essentielles Go

```bash
go run main.go       # exécuter
go build             # compiler
go test ./...        # lancer tous les tests
go fmt ./...         # formater le code
go mod tidy          # nettoyer les dépendances
go env               # afficher l'environnement Go
```

**Explication :** Cette commande compile puis exécute les tests Go. Le motif `./...` inclut le package courant et tous ses sous-packages, ce qui convient à la validation complète d’un module. Une erreur de compilation interrompt les tests : le typage fait donc partie du premier niveau de vérification.

### PHP équivalent approximatif

```bash
php index.php                  # exécuter
composer install               # installer les dépendances
composer dump-autoload          # régénérer l'autoload
vendor/bin/phpunit             # lancer les tests
vendor/bin/php-cs-fixer fix    # formater le code si configuré
```

**Explication :** Ces commandes montrent le workflow PHP équivalent autour de Composer. Composer gère principalement les dépendances et l’autoload, alors que la commande `go` regroupe aussi compilation, formatage et tests. Le parallèle aide à se repérer, mais les deux outils n’ont pas exactement le même périmètre.

---

## 2.4 Librairie standard Go

Go vient avec une librairie standard très complète.

Exemples :

```go
import "fmt"       // affichage console
import "net/http"  // serveur HTTP
import "encoding/json" // JSON
import "time"      // dates
import "errors"    // erreurs
import "context"   // contexte, timeout, annulation
```

**Explication :** Les imports déclarent les packages utilisés par ce fichier. Leur chemin identifie le package, tandis que son nom qualifie les symboles appelés, par exemple `fmt.Println`. Go refuse les imports inutilisés, ce qui empêche l’accumulation de dépendances mortes.

### PHP équivalent

En PHP, beaucoup de fonctionnalités sont natives, mais les projets modernes utilisent souvent des composants :

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\JsonResponse;
use Psr\Log\LoggerInterface;
```

**Explication :** Symfony prend ici en charge le routage et une partie de la construction de la réponse HTTP. Avec `net/http`, Go expose des primitives plus basses et rend l’enchaînement plus explicite. Le handler doit néanmoins rester mince dans les deux écosystèmes : décoder, valider la forme, appeler le cas d’usage et traduire le résultat en HTTP.

---

# 3. Variables, types, tableaux, slices, maps et structs

Le typage statique de Go oblige à définir clairement la forme des données. De nombreuses incohérences sont ainsi détectées pendant la compilation, avant qu’une requête n’atteigne l’application en production.

En PHP, un tableau peut représenter une liste, un dictionnaire ou une structure métier improvisée. Go sépare ces usages : un array possède une taille fixe, une slice représente une séquence dynamique, une map associe des clés à des valeurs et une struct décrit explicitement une donnée.

Dans un backend, les structs sont omniprésentes. Elles peuvent représenter une entité métier, le corps d’une requête HTTP, la réponse envoyée au frontend ou la configuration de l’application. Donner un rôle précis à chaque struct évite de mélanger le modèle interne et le contrat public de l’API.

## 3.1 Déclaration de variables

### Go : déclaration explicite

```go
var name string = "Alice"
var age int = 30
var active bool = true

fmt.Println(name, age, active)
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### Go : déclaration courte

```go
name := "Alice"
age := 30
active := true

fmt.Println(name, age, active)
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

$name = "Alice";
$age = 30;
$active = true;

echo $name . " " . $age . " " . ($active ? "true" : "false") . PHP_EOL;
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 3.2 Types simples

### Go

```go
var title string = "Formation Go"
var count int = 10
var price float64 = 19.99
var enabled bool = true

fmt.Println(title, count, price, enabled)
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

$title = "Formation Go";
$count = 10;
$price = 19.99;
$enabled = true;

var_dump($title, $count, $price, $enabled);
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 3.3 Typage strict

### Go

```go
var age int = 30

// Erreur de compilation : impossible d'assigner une string dans un int.
// age = "thirty"

fmt.Println(age)
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent avec typage

```php
<?php

declare(strict_types=1);

$age = 30;

// PHP permettrait souvent plus de souplesse,
// mais avec des types de fonctions stricts, on peut se rapprocher du typage Go.
function displayAge(int $age): void
{
    echo $age . PHP_EOL;
}

displayAge($age);
```

**Explication :** Cette fonction PHP sert de repère pour la syntaxe Go présentée juste avant. Les types de paramètres et de retour peuvent être déclarés en PHP, mais ils sont systématiques et vérifiés à la compilation en Go. Comparez surtout le contrat de la fonction et la manière dont chaque langage représente les erreurs.

---

## 3.4 Constantes

### Go

```go
const AppName = "Product API"
const MaxItems = 100

fmt.Println(AppName, MaxItems)
```

**Explication :** Ces constantes sont déterminées à la compilation et ne peuvent plus être réassignées. Go autorise des constantes non typées, dont le type concret est déduit lorsqu’elles sont utilisées. Réservez-les aux valeurs réellement immuables et stables dans le code, pas à la configuration d’un environnement.

### PHP équivalent

```php
<?php

const APP_NAME = 'Product API';
const MAX_ITEMS = 100;

echo APP_NAME . PHP_EOL;
echo MAX_ITEMS . PHP_EOL;
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 3.5 Valeur zéro

En Go, chaque type a une valeur par défaut.

### Go

```go
var name string
var count int
var price float64
var active bool

fmt.Printf("name=%q count=%d price=%f active=%t\n", name, count, price, active)
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

Sortie :

```text
name="" count=0 price=0.000000 active=false
```

**Explication :** Cette sortie confirme les valeurs zéro attribuées automatiquement par Go. Une chaîne vaut `""`, un nombre vaut `0` et un booléen vaut `false` tant qu’aucune autre valeur n’est assignée. Il faut distinguer une valeur zéro valide d’une donnée réellement absente, pour laquelle un pointeur ou un type optionnel peut être nécessaire.

### PHP équivalent

En PHP, une variable non initialisée produit généralement une notice/warning selon le contexte.

```php
<?php

$name = '';
$count = 0;
$price = 0.0;
$active = false;

var_dump($name, $count, $price, $active);
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 3.6 Arrays Go

Un array Go a une taille fixe.

### Go

```go
var numbers [3]int

numbers[0] = 10
numbers[1] = 20
numbers[2] = 30

fmt.Println(numbers)
fmt.Println(numbers[1])
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

PHP n’a pas vraiment d’array fixe natif. Un tableau PHP est dynamique.

```php
<?php

$numbers = [10, 20, 30];

echo $numbers[1] . PHP_EOL;
print_r($numbers);
```

**Explication :** Le tableau PHP est utilisé ici comme collection flexible. Selon ses clés, il peut représenter une liste, une map ou une structure ad hoc ; Go emploie des types distincts pour chacun de ces rôles. Cette contrainte supplémentaire évite qu’une collection change accidentellement de forme au cours du programme.

---

## 3.7 Slices Go

Le slice est le type le plus utilisé pour les listes en Go.

### Go

```go
numbers := []int{10, 20, 30}

numbers = append(numbers, 40)

fmt.Println(numbers)
fmt.Println(len(numbers))
```

**Explication :** La slice représente une vue dynamique sur un tableau sous-jacent. `append` peut réutiliser ce stockage ou en allouer un nouveau, d’où la nécessité de récupérer la slice retournée. Lors d’un `range`, Go fournit l’index et une copie de la valeur ; utilisez l’index si vous devez modifier l’élément stocké.

### PHP équivalent

```php
<?php

$numbers = [10, 20, 30];

$numbers[] = 40;

print_r($numbers);
echo count($numbers) . PHP_EOL;
```

**Explication :** Le tableau PHP est utilisé ici comme collection flexible. Selon ses clés, il peut représenter une liste, une map ou une structure ad hoc ; Go emploie des types distincts pour chacun de ces rôles. Cette contrainte supplémentaire évite qu’une collection change accidentellement de forme au cours du programme.

---

## 3.8 Slice vide

### Go

```go
users := []string{}

users = append(users, "Alice")
users = append(users, "Bob")

fmt.Println(users)
```

**Explication :** La slice représente une vue dynamique sur un tableau sous-jacent. `append` peut réutiliser ce stockage ou en allouer un nouveau, d’où la nécessité de récupérer la slice retournée. Lors d’un `range`, Go fournit l’index et une copie de la valeur ; utilisez l’index si vous devez modifier l’élément stocké.

### PHP équivalent

```php
<?php

$users = [];

$users[] = "Alice";
$users[] = "Bob";

print_r($users);
```

**Explication :** Le tableau PHP est utilisé ici comme collection flexible. Selon ses clés, il peut représenter une liste, une map ou une structure ad hoc ; Go emploie des types distincts pour chacun de ces rôles. Cette contrainte supplémentaire évite qu’une collection change accidentellement de forme au cours du programme.

---

## 3.9 Slice avec `make`

### Go

```go
users := make([]string, 0, 10)

users = append(users, "Alice")
users = append(users, "Bob")

fmt.Println(users)
fmt.Println("length:", len(users))
fmt.Println("capacity:", cap(users))
```

**Explication :** La slice représente une vue dynamique sur un tableau sous-jacent. `append` peut réutiliser ce stockage ou en allouer un nouveau, d’où la nécessité de récupérer la slice retournée. Lors d’un `range`, Go fournit l’index et une copie de la valeur ; utilisez l’index si vous devez modifier l’élément stocké.

### PHP équivalent

PHP ne gère pas explicitement la capacité d’un tableau.

```php
<?php

$users = [];

$users[] = "Alice";
$users[] = "Bob";

echo count($users) . PHP_EOL;
```

**Explication :** Le tableau PHP est utilisé ici comme collection flexible. Selon ses clés, il peut représenter une liste, une map ou une structure ad hoc ; Go emploie des types distincts pour chacun de ces rôles. Cette contrainte supplémentaire évite qu’une collection change accidentellement de forme au cours du programme.

---

## 3.10 Parcourir une slice

### Go

```go
users := []string{"Alice", "Bob", "Charlie"}

for index, user := range users {
    fmt.Println(index, user)
}
```

**Explication :** La slice représente une vue dynamique sur un tableau sous-jacent. `append` peut réutiliser ce stockage ou en allouer un nouveau, d’où la nécessité de récupérer la slice retournée. Lors d’un `range`, Go fournit l’index et une copie de la valeur ; utilisez l’index si vous devez modifier l’élément stocké.

### PHP équivalent

```php
<?php

$users = ["Alice", "Bob", "Charlie"];

foreach ($users as $index => $user) {
    echo $index . " " . $user . PHP_EOL;
}
```

**Explication :** `foreach` joue ici le même rôle que `range` en Go : il fournit la clé ou l’index et la valeur courante. Le tableau PHP peut mélanger usages de liste et de dictionnaire, alors que Go distingue slices et maps. Cette séparation rend l’intention et les types accessibles au compilateur.

---

## 3.11 Maps Go

Une map Go est une structure clé-valeur.

### Go

```go
scores := map[string]int{
    "Alice": 10,
    "Bob":   15,
}

scores["Charlie"] = 20

fmt.Println(scores["Alice"])
fmt.Println(scores)
```

**Explication :** La map associe des clés et des valeurs de types fixes. Une lecture peut retourner la valeur zéro même si la clé n’existe pas ; la forme `value, ok := m[key]` permet de distinguer ces deux cas. Une map n’est pas sûre pour des écritures concurrentes et son ordre d’itération n’est pas garanti.

### PHP équivalent

```php
<?php

$scores = [
    "Alice" => 10,
    "Bob" => 15,
];

$scores["Charlie"] = 20;

echo $scores["Alice"] . PHP_EOL;
print_r($scores);
```

**Explication :** Le tableau PHP est utilisé ici comme collection flexible. Selon ses clés, il peut représenter une liste, une map ou une structure ad hoc ; Go emploie des types distincts pour chacun de ces rôles. Cette contrainte supplémentaire évite qu’une collection change accidentellement de forme au cours du programme.

---

## 3.12 Vérifier l’existence d’une clé

### Go

```go
scores := map[string]int{
    "Alice": 10,
}

score, exists := scores["Bob"]

if !exists {
    fmt.Println("Bob not found")
} else {
    fmt.Println(score)
}
```

**Explication :** La map associe des clés et des valeurs de types fixes. Une lecture peut retourner la valeur zéro même si la clé n’existe pas ; la forme `value, ok := m[key]` permet de distinguer ces deux cas. Une map n’est pas sûre pour des écritures concurrentes et son ordre d’itération n’est pas garanti.

### PHP équivalent

```php
<?php

$scores = [
    "Alice" => 10,
];

if (!array_key_exists("Bob", $scores)) {
    echo "Bob not found\n";
} else {
    echo $scores["Bob"] . PHP_EOL;
}
```

**Explication :** Le tableau PHP est utilisé ici comme collection flexible. Selon ses clés, il peut représenter une liste, une map ou une structure ad hoc ; Go emploie des types distincts pour chacun de ces rôles. Cette contrainte supplémentaire évite qu’une collection change accidentellement de forme au cours du programme.

---

## 3.13 Structs Go

Une struct est proche d’une classe PHP sans méthode héritée.

### Go

```go
type User struct {
    ID    int
    Name  string
    Email string
}

user := User{
    ID:    1,
    Name:  "Alice",
    Email: "alice@example.com",
}

fmt.Println(user.Name)
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
    ) {}
}

$user = new User(1, "Alice", "alice@example.com");

echo $user->name . PHP_EOL;
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

## 3.14 Méthodes sur struct

### Go

```go
type Product struct {
    Name  string
    Price float64
    Stock int
}

func (p Product) IsAvailable() bool {
    return p.Stock > 0
}

product := Product{Name: "Keyboard", Price: 49.99, Stock: 5}

fmt.Println(product.IsAvailable())
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

class Product
{
    public function __construct(
        public string $name,
        public float $price,
        public int $stock,
    ) {}

    public function isAvailable(): bool
    {
        return $this->stock > 0;
    }
}

$product = new Product("Keyboard", 49.99, 5);

var_dump($product->isAvailable());
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

## 3.15 Pointeurs : introduction

Un pointeur contient l’adresse mémoire d’une valeur. En Go, les pointeurs sont utiles pour modifier une valeur depuis une fonction ou éviter certaines copies.

### Go sans pointeur

```go
func Rename(user User, newName string) {
    user.Name = newName
}

user := User{ID: 1, Name: "Alice"}
Rename(user, "Bob")

fmt.Println(user.Name) // Alice
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### Go avec pointeur

```go
func Rename(user *User, newName string) {
    user.Name = newName
}

user := User{ID: 1, Name: "Alice"}
Rename(&user, "Bob")

fmt.Println(user.Name) // Bob
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

En PHP, les objets sont manipulés via une référence vers l’objet.

```php
<?php

class User
{
    public function __construct(
        public int $id,
        public string $name,
    ) {}
}

function renameUser(User $user, string $newName): void
{
    $user->name = $newName;
}

$user = new User(1, "Alice");
renameUser($user, "Bob");

echo $user->name . PHP_EOL; // Bob
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

# 4. Fonctions en Go et équivalents PHP

Une fonction Go rend ses entrées et ses sorties visibles dans sa signature. Elle peut retourner plusieurs valeurs, ce qui permet notamment de renvoyer un résultat accompagné d’une erreur. Ce mécanisme remplace une grande partie des exceptions utilisées dans les applications PHP.

Une erreur Go est une valeur normale. L’appelant doit décider s’il peut la traiter, l’enrichir avec du contexte ou la transmettre au niveau supérieur. Cette gestion explicite ajoute parfois quelques lignes, mais rend le chemin d’échec lisible et prévisible.

Les méthodes sont des fonctions associées à un type grâce à un receiver. Un receiver par valeur reçoit une copie ; un receiver par pointeur peut modifier l’instance d’origine. Le choix dépend donc du comportement attendu, pas seulement d’une préférence de syntaxe.

## 4.1 Fonction simple

### Go

```go
func Add(a int, b int) int {
    return a + b
}

result := Add(2, 3)
fmt.Println(result)
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

function add(int $a, int $b): int
{
    return $a + $b;
}

$result = add(2, 3);
echo $result . PHP_EOL;
```

**Explication :** Cette fonction PHP sert de repère pour la syntaxe Go présentée juste avant. Les types de paramètres et de retour peuvent être déclarés en PHP, mais ils sont systématiques et vérifiés à la compilation en Go. Comparez surtout le contrat de la fonction et la manière dont chaque langage représente les erreurs.

---

## 4.2 Paramètres du même type

### Go

```go
func Multiply(a, b int) int {
    return a * b
}
```

**Explication :** La signature rend explicites les paramètres et les valeurs retournées par cette fonction. Go peut retourner plusieurs valeurs, notamment un résultat et une erreur, ce qui rend le chemin d’échec visible chez l’appelant. Gardez la fonction centrée sur une responsabilité et préférez des paramètres métier à des dépendances globales.

### PHP équivalent

```php
<?php

function multiply(int $a, int $b): int
{
    return $a * $b;
}
```

**Explication :** Cette fonction PHP sert de repère pour la syntaxe Go présentée juste avant. Les types de paramètres et de retour peuvent être déclarés en PHP, mais ils sont systématiques et vérifiés à la compilation en Go. Comparez surtout le contrat de la fonction et la manière dont chaque langage représente les erreurs.

---

## 4.3 Plusieurs valeurs de retour

### Go

```go
func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }

    return a / b, nil
}

result, err := Divide(10, 2)
if err != nil {
    fmt.Println("error:", err)
    return
}

fmt.Println(result)
```

**Explication :** L’erreur est une valeur retournée avec le résultat normal. L’appelant la vérifie immédiatement, ajoute éventuellement du contexte avec `fmt.Errorf(... %w ...)`, puis décide de la traiter ou de la propager. Ne masquez pas l’erreur : son message et sa cause sont essentiels pour le diagnostic et pour la traduction vers une réponse HTTP adaptée.

### PHP équivalent avec exception

```php
<?php

function divide(int $a, int $b): int
{
    if ($b === 0) {
        throw new InvalidArgumentException("division by zero");
    }

    return intdiv($a, $b);
}

try {
    $result = divide(10, 2);
    echo $result . PHP_EOL;
} catch (InvalidArgumentException $e) {
    echo "error: " . $e->getMessage() . PHP_EOL;
}
```

**Explication :** PHP signale ici l’échec avec une exception qui interrompt le flux normal jusqu’à ce qu’un `catch` la traite. Go préfère généralement retourner explicitement une valeur `error` et oblige chaque appelant à décider quoi en faire. L’équivalence est donc fonctionnelle, mais le chemin de contrôle reste différent.

---

## 4.4 Retourner une valeur et une erreur

C’est un pattern très courant en Go.

### Go

```go
func FindUser(id int) (User, error) {
    if id <= 0 {
        return User{}, errors.New("invalid user id")
    }

    return User{
        ID:    id,
        Name:  "Alice",
        Email: "alice@example.com",
    }, nil
}
```

**Explication :** L’erreur est une valeur retournée avec le résultat normal. L’appelant la vérifie immédiatement, ajoute éventuellement du contexte avec `fmt.Errorf(... %w ...)`, puis décide de la traiter ou de la propager. Ne masquez pas l’erreur : son message et sa cause sont essentiels pour le diagnostic et pour la traduction vers une réponse HTTP adaptée.

Utilisation :

```go
user, err := FindUser(1)
if err != nil {
    fmt.Println(err)
    return
}

fmt.Println(user.Name)
```

**Explication :** L’erreur est une valeur retournée avec le résultat normal. L’appelant la vérifie immédiatement, ajoute éventuellement du contexte avec `fmt.Errorf(... %w ...)`, puis décide de la traiter ou de la propager. Ne masquez pas l’erreur : son message et sa cause sont essentiels pour le diagnostic et pour la traduction vers une réponse HTTP adaptée.

### PHP équivalent

```php
<?php

function findUser(int $id): User
{
    if ($id <= 0) {
        throw new InvalidArgumentException("invalid user id");
    }

    return new User($id, "Alice", "alice@example.com");
}

try {
    $user = findUser(1);
    echo $user->name . PHP_EOL;
} catch (InvalidArgumentException $e) {
    echo $e->getMessage() . PHP_EOL;
}
```

**Explication :** PHP signale ici l’échec avec une exception qui interrompt le flux normal jusqu’à ce qu’un `catch` la traite. Go préfère généralement retourner explicitement une valeur `error` et oblige chaque appelant à décider quoi en faire. L’équivalence est donc fonctionnelle, mais le chemin de contrôle reste différent.

---

## 4.5 Fonction privée ou publique

En Go, la visibilité dépend de la première lettre.

### Go

```go
func publicFunction() {
    // non exportée car commence par minuscule
}

func PublicFunction() {
    // exportée car commence par majuscule
}
```

**Explication :** La signature rend explicites les paramètres et les valeurs retournées par cette fonction. Go peut retourner plusieurs valeurs, notamment un résultat et une erreur, ce qui rend le chemin d’échec visible chez l’appelant. Gardez la fonction centrée sur une responsabilité et préférez des paramètres métier à des dépendances globales.

### PHP équivalent

```php
<?php

class Example
{
    private function privateFunction(): void
    {
    }

    public function publicFunction(): void
    {
    }
}
```

**Explication :** Cette fonction PHP sert de repère pour la syntaxe Go présentée juste avant. Les types de paramètres et de retour peuvent être déclarés en PHP, mais ils sont systématiques et vérifiés à la compilation en Go. Comparez surtout le contrat de la fonction et la manière dont chaque langage représente les erreurs.

---

## 4.6 Fonctions anonymes

### Go

```go
greet := func(name string) string {
    return "Hello " + name
}

fmt.Println(greet("Alice"))
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

$greet = function (string $name): string {
    return "Hello " . $name;
};

echo $greet("Alice") . PHP_EOL;
```

**Explication :** Cette fonction PHP sert de repère pour la syntaxe Go présentée juste avant. Les types de paramètres et de retour peuvent être déclarés en PHP, mais ils sont systématiques et vérifiés à la compilation en Go. Comparez surtout le contrat de la fonction et la manière dont chaque langage représente les erreurs.

---

## 4.7 Closure

### Go

```go
func Counter() func() int {
    count := 0

    return func() int {
        count++
        return count
    }
}

counter := Counter()

fmt.Println(counter()) // 1
fmt.Println(counter()) // 2
fmt.Println(counter()) // 3
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

function counter(): callable
{
    $count = 0;

    return function () use (&$count): int {
        $count++;
        return $count;
    };
}

$counter = counter();

echo $counter() . PHP_EOL; // 1
echo $counter() . PHP_EOL; // 2
echo $counter() . PHP_EOL; // 3
```

**Explication :** Cette fonction PHP sert de repère pour la syntaxe Go présentée juste avant. Les types de paramètres et de retour peuvent être déclarés en PHP, mais ils sont systématiques et vérifiés à la compilation en Go. Comparez surtout le contrat de la fonction et la manière dont chaque langage représente les erreurs.

---

## 4.8 Méthodes avec receiver

### Go

```go
type CartItem struct {
    Price    float64
    Quantity int
}

func (item CartItem) Total() float64 {
    return item.Price * float64(item.Quantity)
}

item := CartItem{Price: 12.5, Quantity: 4}

fmt.Println(item.Total())
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

class CartItem
{
    public function __construct(
        public float $price,
        public int $quantity,
    ) {}

    public function total(): float
    {
        return $this->price * $this->quantity;
    }
}

$item = new CartItem(12.5, 4);

echo $item->total() . PHP_EOL;
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

# 5. Imports, packages et organisation du code

Les packages définissent les frontières du code. Un package cohérent regroupe des éléments qui travaillent ensemble et expose seulement ce qui doit être utilisé par les autres parties de l’application.

Il ne faut pas transposer mécaniquement chaque classe, service ou dossier Symfony en package Go. Trop de petits packages produisent des imports circulaires, dispersent les concepts et rendent la navigation difficile. Une bonne frontière correspond généralement à une responsabilité ou à un domaine fonctionnel identifiable.

La visibilité repose sur la première lettre du nom : une majuscule exporte le symbole, une minuscule le garde interne au package. Cette règle encourage une API publique réduite. Le dossier `internal` renforce cette intention en empêchant l’import du code depuis l’extérieur du module autorisé.

## 5.1 Import simple

### Go

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello")
}
```

**Explication :** `package main` indique que le package produit un programme exécutable, et `func main()` en est le point d’entrée. Ce bloc assemble les dépendances nécessaires à « Go » puis conserve le processus actif jusqu’à la fin du travail. Comme l’application Go vit plus longtemps qu’une requête PHP-FPM, les ressources créées ici doivent avoir un cycle de vie et un arrêt maîtrisés.

### PHP équivalent

```php
<?php

echo "Hello\n";
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 5.2 Imports multiples

### Go

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

func main() {
    payload := map[string]string{
        "now": time.Now().Format(time.RFC3339),
    }

    data, err := json.Marshal(payload)
    if err != nil {
        panic(err)
    }

    fmt.Println(string(data))
}
```

**Explication :** L’erreur est une valeur retournée avec le résultat normal. L’appelant la vérifie immédiatement, ajoute éventuellement du contexte avec `fmt.Errorf(... %w ...)`, puis décide de la traiter ou de la propager. Ne masquez pas l’erreur : son message et sa cause sont essentiels pour le diagnostic et pour la traduction vers une réponse HTTP adaptée.

### PHP équivalent

```php
<?php

$payload = [
    "now" => (new DateTimeImmutable())->format(DateTimeInterface::ATOM),
];

echo json_encode($payload, JSON_THROW_ON_ERROR) . PHP_EOL;
```

**Explication :** Le code PHP transforme le JSON en tableau ou objet dynamique, puis construit la réponse. Cette souplesse réduit le code initial mais reporte plusieurs erreurs de forme à l’exécution. En Go, une struct de requête ou de réponse documente les champs attendus et permet au compilateur de vérifier leur utilisation.

---

## 5.3 Créer un package interne

Structure :

```text
go-training/
├── go.mod
├── main.go
└── internal/
    └── product/
        └── product.go
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « 5.3 Créer un package interne ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

### Go : `internal/product/product.go`

```go
package product

type Product struct {
    ID    int
    Name  string
    Price float64
    Stock int
}

func IsAvailable(p Product) bool {
    return p.Stock > 0
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go : internal/product/product.go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### Go : `main.go`

```go
package main

import (
    "fmt"

    "example.com/go-training/internal/product"
)

func main() {
    p := product.Product{
        ID:    1,
        Name:  "Keyboard",
        Price: 49.99,
        Stock: 3,
    }

    fmt.Println(product.IsAvailable(p))
}
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

Structure :

```text
php-training/
├── composer.json
├── index.php
└── src/
    └── Product/
        └── Product.php
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « PHP équivalent ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

`src/Product/Product.php`

```php
<?php

namespace App\Product;

class Product
{
    public function __construct(
        public int $id,
        public string $name,
        public float $price,
        public int $stock,
    ) {}

    public function isAvailable(): bool
    {
        return $this->stock > 0;
    }
}
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

`index.php`

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use App\Product\Product;

$product = new Product(1, "Keyboard", 49.99, 3);

var_dump($product->isAvailable());
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 5.4 Nom exporté / non exporté

### Go

```go
package product

// Exporté, utilisable depuis un autre package.
type Product struct {
    ID   int
    Name string
}

// Non exporté, seulement utilisable dans le package product.
type internalProductCache struct {
    items []Product
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

namespace App\Product;

class Product
{
    public function __construct(
        public int $id,
        public string $name,
    ) {}
}

final class InternalProductCache
{
    /** @var Product[] */
    private array $items = [];
}
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

En PHP, la visibilité est souvent gérée avec `public`, `private`, `protected`, mais le nom de la classe reste accessible si l’autoload la charge. En Go, la majuscule/minuscule contrôle vraiment l’export du symbole.

---

# 6. Conditions, boucles, loops et range

Go propose peu de constructions de contrôle, mais elles couvrent les besoins habituels d’un backend. `if` gère les décisions, `switch` simplifie les branches multiples et `for` remplace à la fois les boucles `for`, `while` et infinies de PHP.

Le mot-clé `range` permet de parcourir les slices, maps, chaînes et channels. Les valeurs retournées dépendent du type parcouru : index et valeur pour une slice, clé et valeur pour une map. `_` sert à ignorer explicitement une valeur dont le programme n’a pas besoin.

L’ordre de parcours d’une map n’est pas garanti. Si une réponse JSON, un test ou une règle métier exige un ordre stable, les clés ou les valeurs doivent être copiées dans une slice puis triées.

## 6.1 If simple

### Go

```go
age := 20

if age >= 18 {
    fmt.Println("adult")
} else {
    fmt.Println("minor")
}
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

$age = 20;

if ($age >= 18) {
    echo "adult\n";
} else {
    echo "minor\n";
}
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 6.2 If avec initialisation

### Go

```go
if score := 85; score >= 50 {
    fmt.Println("passed")
} else {
    fmt.Println("failed")
}
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

$score = 85;

if ($score >= 50) {
    echo "passed\n";
} else {
    echo "failed\n";
}
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 6.3 Switch

### Go

```go
status := "pending"

switch status {
case "pending":
    fmt.Println("waiting")
case "done":
    fmt.Println("completed")
case "failed":
    fmt.Println("error")
default:
    fmt.Println("unknown")
}
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

$status = "pending";

switch ($status) {
    case "pending":
        echo "waiting\n";
        break;
    case "done":
        echo "completed\n";
        break;
    case "failed":
        echo "error\n";
        break;
    default:
        echo "unknown\n";
}
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 6.4 For classique

### Go

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

for ($i = 0; $i < 5; $i++) {
    echo $i . PHP_EOL;
}
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 6.5 For comme while

### Go

```go
count := 0

for count < 5 {
    fmt.Println(count)
    count++
}
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

### PHP équivalent

```php
<?php

$count = 0;

while ($count < 5) {
    echo $count . PHP_EOL;
    $count++;
}
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 6.6 Boucle infinie

### Go

```go
for {
    fmt.Println("running")
    break
}
```

**Explication :** `for` est l’unique mot-clé de boucle en Go : sa forme couvre les usages de `for`, `while` et boucle infinie de PHP. La condition est évaluée à chaque itération et `break` ou `continue` modifie le flux. Vérifiez toujours que la boucle possède une condition de sortie ou répond à une annulation.

### PHP équivalent

```php
<?php

while (true) {
    echo "running\n";
    break;
}
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 6.7 Range sur slice

### Go

```go
products := []string{"Keyboard", "Mouse", "Screen"}

for index, product := range products {
    fmt.Println(index, product)
}
```

**Explication :** La slice représente une vue dynamique sur un tableau sous-jacent. `append` peut réutiliser ce stockage ou en allouer un nouveau, d’où la nécessité de récupérer la slice retournée. Lors d’un `range`, Go fournit l’index et une copie de la valeur ; utilisez l’index si vous devez modifier l’élément stocké.

### PHP équivalent

```php
<?php

$products = ["Keyboard", "Mouse", "Screen"];

foreach ($products as $index => $product) {
    echo $index . " " . $product . PHP_EOL;
}
```

**Explication :** `foreach` joue ici le même rôle que `range` en Go : il fournit la clé ou l’index et la valeur courante. Le tableau PHP peut mélanger usages de liste et de dictionnaire, alors que Go distingue slices et maps. Cette séparation rend l’intention et les types accessibles au compilateur.

---

## 6.8 Range sur map

### Go

```go
stock := map[string]int{
    "Keyboard": 10,
    "Mouse":    0,
    "Screen":   5,
}

for product, quantity := range stock {
    fmt.Println(product, quantity)
}
```

**Explication :** La map associe des clés et des valeurs de types fixes. Une lecture peut retourner la valeur zéro même si la clé n’existe pas ; la forme `value, ok := m[key]` permet de distinguer ces deux cas. Une map n’est pas sûre pour des écritures concurrentes et son ordre d’itération n’est pas garanti.

### PHP équivalent

```php
<?php

$stock = [
    "Keyboard" => 10,
    "Mouse" => 0,
    "Screen" => 5,
];

foreach ($stock as $product => $quantity) {
    echo $product . " " . $quantity . PHP_EOL;
}
```

**Explication :** `foreach` joue ici le même rôle que `range` en Go : il fournit la clé ou l’index et la valeur courante. Le tableau PHP peut mélanger usages de liste et de dictionnaire, alors que Go distingue slices et maps. Cette séparation rend l’intention et les types accessibles au compilateur.

---

## 6.9 Ignorer l’index avec `_`

### Go

```go
products := []string{"Keyboard", "Mouse", "Screen"}

for _, product := range products {
    fmt.Println(product)
}
```

**Explication :** La slice représente une vue dynamique sur un tableau sous-jacent. `append` peut réutiliser ce stockage ou en allouer un nouveau, d’où la nécessité de récupérer la slice retournée. Lors d’un `range`, Go fournit l’index et une copie de la valeur ; utilisez l’index si vous devez modifier l’élément stocké.

### PHP équivalent

```php
<?php

$products = ["Keyboard", "Mouse", "Screen"];

foreach ($products as $product) {
    echo $product . PHP_EOL;
}
```

**Explication :** `foreach` joue ici le même rôle que `range` en Go : il fournit la clé ou l’index et la valeur courante. Le tableau PHP peut mélanger usages de liste et de dictionnaire, alors que Go distingue slices et maps. Cette séparation rend l’intention et les types accessibles au compilateur.

---

## 6.10 Filtrer une liste

### Go

```go
type Product struct {
    Name  string
    Stock int
}

products := []Product{
    {Name: "Keyboard", Stock: 10},
    {Name: "Mouse", Stock: 0},
    {Name: "Screen", Stock: 5},
}

available := []Product{}

for _, product := range products {
    if product.Stock > 0 {
        available = append(available, product)
    }
}

fmt.Println(available)
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

$products = [
    ["name" => "Keyboard", "stock" => 10],
    ["name" => "Mouse", "stock" => 0],
    ["name" => "Screen", "stock" => 5],
];

$available = array_filter($products, static function (array $product): bool {
    return $product["stock"] > 0;
});

print_r($available);
```

**Explication :** Cette fonction PHP sert de repère pour la syntaxe Go présentée juste avant. Les types de paramètres et de retour peuvent être déclarés en PHP, mais ils sont systématiques et vérifiés à la compilation en Go. Comparez surtout le contrat de la fonction et la manière dont chaque langage représente les erreurs.

---

# 7. Goroutines, channels et concurrence

La concurrence permet à plusieurs tâches de progresser sans être exécutées strictement l’une après l’autre. Dans un BFF, elle peut réduire la latence lorsqu’une réponse dépend de plusieurs services distants indépendants.

Une goroutine est légère, mais elle n’est pas gratuite. Elle doit avoir une condition de fin, respecter l’annulation de la requête et transmettre son résultat ou son erreur. Une goroutine qui reste bloquée après le départ du client peut provoquer une fuite de ressources.

Les channels servent à transmettre des valeurs et à synchroniser des goroutines. Ils ne remplacent pas automatiquement les mutex : un mutex est souvent plus simple pour protéger un état partagé, tandis qu’un channel convient lorsqu’on veut modéliser un échange de données.

`context.Context` transporte les délais et l’annulation à travers les couches. Lorsqu’un client ferme la connexion ou qu’un timeout expire, les appels HTTP, les requêtes SQL et les traitements associés doivent pouvoir s’arrêter rapidement.

## 7.1 Goroutine simple

Une goroutine est une fonction exécutée de manière concurrente.

### Go

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    go sayHello()

    time.Sleep(100 * time.Millisecond)
}

func sayHello() {
    fmt.Println("Hello from goroutine")
}
```

**Explication :** Le mot-clé `go` démarre la fonction dans une goroutine et rend immédiatement la main à l’appelant. Le résultat n’est pas récupéré automatiquement : il faut utiliser un channel, un `WaitGroup` ou une abstraction de plus haut niveau pour coordonner la fin et les erreurs. Une goroutine ne doit pas survivre sans limite ni ignorer l’annulation de la requête.

### PHP équivalent approximatif

PHP standard n’a pas d’équivalent direct aussi simple. On peut simuler avec un processus séparé, une queue, un worker, ou des Fibers selon le besoin.

```php
<?php

echo "Hello from PHP\n";

// Pour du vrai asynchrone en PHP, on utilisera plutôt :
// - Messenger Symfony
// - RabbitMQ
// - Redis queue
// - ReactPHP
// - Swoole
// - Fibers
```

**Explication :** Symfony prend ici en charge le routage et une partie de la construction de la réponse HTTP. Avec `net/http`, Go expose des primitives plus basses et rend l’enchaînement plus explicite. Le handler doit néanmoins rester mince dans les deux écosystèmes : décoder, valider la forme, appeler le cas d’usage et traduire le résultat en HTTP.

---

## 7.2 Attendre des goroutines avec WaitGroup

### Go

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup

    tasks := []string{"user", "orders", "preferences"}

    for _, task := range tasks {
        wg.Add(1)

        go func(taskName string) {
            defer wg.Done()
            fmt.Println("fetching", taskName)
        }(task)
    }

    wg.Wait()

    fmt.Println("all tasks completed")
}
```

**Explication :** Le `WaitGroup` compte les tâches concurrentes encore actives. Chaque `Add` annonce une goroutine, chaque `Done` signale sa fin et `Wait` bloque jusqu’à ce que le compteur revienne à zéro. Placez généralement `Done` dans un `defer` pour garantir son exécution même si la goroutine quitte plus tôt.

### PHP équivalent avec traitement séquentiel

```php
<?php

$tasks = ["user", "orders", "preferences"];

foreach ($tasks as $task) {
    echo "fetching " . $task . PHP_EOL;
}

echo "all tasks completed\n";
```

**Explication :** `foreach` joue ici le même rôle que `range` en Go : il fournit la clé ou l’index et la valeur courante. Le tableau PHP peut mélanger usages de liste et de dictionnaire, alors que Go distingue slices et maps. Cette séparation rend l’intention et les types accessibles au compilateur.

### PHP équivalent conceptuel avec Symfony Messenger

```php
<?php

// Conceptuellement :
// $bus->dispatch(new FetchUserMessage($userId));
// $bus->dispatch(new FetchOrdersMessage($userId));
// $bus->dispatch(new FetchPreferencesMessage($userId));
//
// Les workers traitent ensuite les messages en parallèle.
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent conceptuel avec Symfony Messenger » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 7.3 Channel simple

Un channel permet de communiquer entre goroutines.

### Go

```go
package main

import "fmt"

func main() {
    messages := make(chan string)

    go func() {
        messages <- "hello"
    }()

    msg := <-messages

    fmt.Println(msg)
}
```

**Explication :** Le channel transporte des valeurs typées entre goroutines et synchronise leur progression. Un envoi sur un channel non bufferisé attend qu’un récepteur soit prêt ; avec un buffer, l’envoi peut avancer tant qu’une place reste disponible. Le propriétaire de la production doit fermer le channel lorsqu’aucune autre valeur ne sera envoyée.

### PHP équivalent approximatif

```php
<?php

// En PHP classique, on passerait souvent par :
// - une queue
// - Redis
// - RabbitMQ
// - une base de données
// - un système d'événements
//
// Exemple conceptuel :
$message = "hello";
echo $message . PHP_EOL;
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent approximatif » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

## 7.4 Channel avec buffer

### Go

```go
messages := make(chan string, 2)

messages <- "hello"
messages <- "world"

fmt.Println(<-messages)
fmt.Println(<-messages)
```

**Explication :** Le channel transporte des valeurs typées entre goroutines et synchronise leur progression. Un envoi sur un channel non bufferisé attend qu’un récepteur soit prêt ; avec un buffer, l’envoi peut avancer tant qu’une place reste disponible. Le propriétaire de la production doit fermer le channel lorsqu’aucune autre valeur ne sera envoyée.

### PHP équivalent conceptuel

```php
<?php

$messages = [];

$messages[] = "hello";
$messages[] = "world";

echo array_shift($messages) . PHP_EOL;
echo array_shift($messages) . PHP_EOL;
```

**Explication :** Le tableau PHP est utilisé ici comme collection flexible. Selon ses clés, il peut représenter une liste, une map ou une structure ad hoc ; Go emploie des types distincts pour chacun de ces rôles. Cette contrainte supplémentaire évite qu’une collection change accidentellement de forme au cours du programme.

---

## 7.5 Worker pool simple

### Go

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    jobs := make(chan int, 5)
    var wg sync.WaitGroup

    for workerID := 1; workerID <= 3; workerID++ {
        wg.Add(1)

        go func(id int) {
            defer wg.Done()

            for job := range jobs {
                fmt.Printf("worker %d processed job %d\n", id, job)
            }
        }(workerID)
    }

    for job := 1; job <= 5; job++ {
        jobs <- job
    }

    close(jobs)
    wg.Wait()
}
```

**Explication :** Le `WaitGroup` compte les tâches concurrentes encore actives. Chaque `Add` annonce une goroutine, chaque `Done` signale sa fin et `Wait` bloque jusqu’à ce que le compteur revienne à zéro. Placez généralement `Done` dans un `defer` pour garantir son exécution même si la goroutine quitte plus tôt.

### PHP équivalent conceptuel

```php
<?php

// En PHP, le worker pool est souvent implémenté avec une queue.
// Exemple conceptuel :
$jobs = [1, 2, 3, 4, 5];

foreach ($jobs as $job) {
    echo "processed job " . $job . PHP_EOL;
}
```

**Explication :** `foreach` joue ici le même rôle que `range` en Go : il fournit la clé ou l’index et la valeur courante. Le tableau PHP peut mélanger usages de liste et de dictionnaire, alors que Go distingue slices et maps. Cette séparation rend l’intention et les types accessibles au compilateur.

---

## 7.6 Context avec timeout

Très utile dans un BFF pour éviter d’attendre trop longtemps un service externe.

### Go

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
    defer cancel()

    err := fetchRemoteData(ctx)

    if err != nil {
        fmt.Println("error:", err)
        return
    }

    fmt.Println("success")
}

func fetchRemoteData(ctx context.Context) error {
    select {
    case <-time.After(1 * time.Second):
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}
```

**Explication :** Le contexte propage ici une échéance et un signal d’annulation. Toutes les opérations lentes appelées en dessous — HTTP, base de données ou attente sur channel — devraient accepter ce contexte et s’arrêter lorsque `ctx.Done()` est fermé. L’appel à `cancel` libère les ressources associées, même si l’opération termine avant le timeout.

### PHP équivalent conceptuel avec timeout HTTP

```php
<?php

// Exemple conceptuel avec Symfony HttpClient :
// $client->request('GET', 'https://api.example.com/products', [
//     'timeout' => 0.5,
// ]);
//
// En PHP, on gère souvent le timeout au niveau du client HTTP.
```

**Explication :** Symfony prend ici en charge le routage et une partie de la construction de la réponse HTTP. Avec `net/http`, Go expose des primitives plus basses et rend l’enchaînement plus explicite. Le handler doit néanmoins rester mince dans les deux écosystèmes : décoder, valider la forme, appeler le cas d’usage et traduire le résultat en HTTP.

---

## 7.7 Race condition

### Go : mauvais exemple

```go
counter := 0

for i := 0; i < 1000; i++ {
    go func() {
        counter++
    }()
}

time.Sleep(time.Second)
fmt.Println(counter)
```

**Explication :** Le mot-clé `go` démarre la fonction dans une goroutine et rend immédiatement la main à l’appelant. Le résultat n’est pas récupéré automatiquement : il faut utiliser un channel, un `WaitGroup` ou une abstraction de plus haut niveau pour coordonner la fin et les erreurs. Une goroutine ne doit pas survivre sans limite ni ignorer l’annulation de la requête.

Ce code peut produire un résultat incorrect, car plusieurs goroutines modifient `counter` en même temps.

### Go : correction avec mutex

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    counter := 0

    var mu sync.Mutex
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)

        go func() {
            defer wg.Done()

            mu.Lock()
            counter++
            mu.Unlock()
        }()
    }

    wg.Wait()

    fmt.Println(counter)
}
```

**Explication :** Le `WaitGroup` compte les tâches concurrentes encore actives. Chaque `Add` annonce une goroutine, chaque `Done` signale sa fin et `Wait` bloque jusqu’à ce que le compteur revienne à zéro. Placez généralement `Done` dans un `defer` pour garantir son exécution même si la goroutine quitte plus tôt.

### PHP équivalent conceptuel

En PHP classique, les requêtes sont souvent isolées. Les race conditions apparaissent plutôt avec :

- accès concurrent à une base de données ;
- workers qui consomment la même queue ;
- écriture concurrente dans un fichier ;
- cache partagé.

```php
<?php

// Exemple conceptuel : préférer une transaction DB ou un lock distribué.
// $connection->beginTransaction();
// ...
// $connection->commit();
```

**Explication :** Cet équivalent PHP permet de rattacher « PHP équivalent conceptuel » à une syntaxe connue. Il faut comparer l’intention plutôt que traduire ligne par ligne : Go privilégie des types explicites, des dépendances visibles et des erreurs retournées, là où PHP s’appuie souvent sur le runtime et les exceptions.

---

# 8. Clean architecture pour un BFF

Une architecture propre sépare les règles métier des détails techniques. Le domaine représente les concepts importants, le usecase orchestre une action, le repository abstrait l’accès aux données et le handler traduit le protocole HTTP vers l’application.

Cette séparation facilite les tests et limite l’impact des changements. Remplacer un repository mémoire par PostgreSQL ne devrait pas obliger à réécrire le usecase. Modifier le format JSON d’une réponse ne devrait pas contaminer le modèle métier.

Il ne faut cependant pas créer une couche ou une interface pour chaque fonction. Une abstraction est utile lorsqu’elle protège une règle, permet de remplacer une dépendance ou clarifie une frontière. Sans bénéfice concret, elle ajoute seulement de la navigation.

Le flux typique d’une requête est : requête HTTP → validation du format → usecase → repository ou service distant → domaine → mapper → réponse HTTP.

## 8.1 Rôle du BFF

Un BFF, Backend For Frontend, sert à adapter les données aux besoins d’un front spécifique.

Il peut :

- agréger plusieurs APIs ;
- transformer des modèles complexes en réponses simples ;
- appliquer des règles de présentation ;
- gérer des erreurs de services externes ;
- exposer des endpoints adaptés à l’UX.

---

## 8.2 Principe de séparation

Mauvais pattern :

```text
handler HTTP
  ├── lit la requête
  ├── appelle une API externe
  ├── applique toute la logique métier
  ├── mappe la réponse
  └── écrit le JSON
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « 8.2 Principe de séparation ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

Meilleur pattern :

```text
handler HTTP
  └── usecase
      └── repository/client externe
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « 8.2 Principe de séparation ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

---

## 8.3 Structure recommandée

```text
product-api/
├── go.mod
├── cmd/
│   └── api/
│       └── main.go
└── internal/
    └── product/
        ├── domain.go
        ├── handler.go
        ├── usecase.go
        ├── repository.go
        ├── memory_repository.go
        ├── mapper.go
        └── usecase_test.go
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « 8.3 Structure recommandée ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

### PHP équivalent Symfony

```text
src/
├── Controller/
│   └── ProductController.php
├── Application/
│   └── Product/
│       └── GetProductUseCase.php
├── Domain/
│   └── Product.php
├── Infrastructure/
│   └── ProductRepository.php
└── Tests/
    └── Product/
        └── GetProductUseCaseTest.php
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « PHP équivalent Symfony ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

---

## 8.4 Domaine

### Go : `domain.go`

```go
package product

type Product struct {
    ID          string
    Name        string
    Description string
    Price       float64
    Stock       int
}

func (p Product) IsAvailable() bool {
    return p.Stock > 0
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go : domain.go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

namespace App\Domain;

final class Product
{
    public function __construct(
        public readonly string $id,
        public readonly string $name,
        public readonly string $description,
        public readonly float $price,
        public readonly int $stock,
    ) {}

    public function isAvailable(): bool
    {
        return $this->stock > 0;
    }
}
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

## 8.5 Repository interface

### Go : `repository.go`

```go
package product

import "context"

type Repository interface {
    FindByID(ctx context.Context, id string) (Product, error)
    FindAll(ctx context.Context) ([]Product, error)
    Save(ctx context.Context, product Product) error
}
```

**Explication :** Cette interface décrit uniquement les opérations dont l’appelant a besoin. En Go, un type satisfait l’interface implicitement dès qu’il possède les bonnes méthodes : le domaine n’a donc pas à dépendre d’une implémentation mémoire, SQL ou HTTP. Gardez l’interface petite et définissez-la près du code qui la consomme.

### PHP équivalent

```php
<?php

namespace App\Domain;

interface ProductRepositoryInterface
{
    public function findById(string $id): Product;

    /**
     * @return Product[]
     */
    public function findAll(): array;

    public function save(Product $product): void;
}
```

**Explication :** Cette fonction PHP sert de repère pour la syntaxe Go présentée juste avant. Les types de paramètres et de retour peuvent être déclarés en PHP, mais ils sont systématiques et vérifiés à la compilation en Go. Comparez surtout le contrat de la fonction et la manière dont chaque langage représente les erreurs.

---

## 8.6 Usecase

### Go : `usecase.go`

```go
package product

import (
    "context"
    "errors"
)

var ErrProductNotFound = errors.New("product not found")

type UseCase struct {
    repository Repository
}

func NewUseCase(repository Repository) UseCase {
    return UseCase{
        repository: repository,
    }
}

func (uc UseCase) GetProduct(ctx context.Context, id string) (Product, error) {
    if id == "" {
        return Product{}, errors.New("product id is required")
    }

    product, err := uc.repository.FindByID(ctx, id)
    if err != nil {
        return Product{}, err
    }

    return product, nil
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go : usecase.go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

namespace App\Application\Product;

use App\Domain\Product;
use App\Domain\ProductRepositoryInterface;
use InvalidArgumentException;

final class GetProductUseCase
{
    public function __construct(
        private ProductRepositoryInterface $repository,
    ) {}

    public function execute(string $id): Product
    {
        if ($id === '') {
            throw new InvalidArgumentException('product id is required');
        }

        return $this->repository->findById($id);
    }
}
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

## 8.7 Repository en mémoire

### Go : `memory_repository.go`

```go
package product

import (
    "context"
)

type MemoryRepository struct {
    products map[string]Product
}

func NewMemoryRepository() *MemoryRepository {
    return &MemoryRepository{
        products: map[string]Product{
            "1": {
                ID:          "1",
                Name:        "Keyboard",
                Description: "Mechanical keyboard",
                Price:       79.99,
                Stock:       10,
            },
            "2": {
                ID:          "2",
                Name:        "Mouse",
                Description: "Wireless mouse",
                Price:       29.99,
                Stock:       0,
            },
        },
    }
}

func (r *MemoryRepository) FindByID(ctx context.Context, id string) (Product, error) {
    product, exists := r.products[id]
    if !exists {
        return Product{}, ErrProductNotFound
    }

    return product, nil
}

func (r *MemoryRepository) FindAll(ctx context.Context) ([]Product, error) {
    products := make([]Product, 0, len(r.products))

    for _, product := range r.products {
        products = append(products, product)
    }

    return products, nil
}

func (r *MemoryRepository) Save(ctx context.Context, product Product) error {
    r.products[product.ID] = product
    return nil
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go : memory_repository.go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

namespace App\Infrastructure;

use App\Domain\Product;
use App\Domain\ProductRepositoryInterface;
use RuntimeException;

final class MemoryProductRepository implements ProductRepositoryInterface
{
    /**
     * @var array<string, Product>
     */
    private array $products;

    public function __construct()
    {
        $this->products = [
            '1' => new Product('1', 'Keyboard', 'Mechanical keyboard', 79.99, 10),
            '2' => new Product('2', 'Mouse', 'Wireless mouse', 29.99, 0),
        ];
    }

    public function findById(string $id): Product
    {
        if (!array_key_exists($id, $this->products)) {
            throw new RuntimeException('product not found');
        }

        return $this->products[$id];
    }

    public function findAll(): array
    {
        return array_values($this->products);
    }

    public function save(Product $product): void
    {
        $this->products[$product->id] = $product;
    }
}
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

# 9. Création et utilisation des endpoints HTTP

Un endpoint est la porte d’entrée HTTP de l’application. Il interprète la méthode, le chemin, les paramètres, les en-têtes et le corps de la requête, puis construit une réponse avec un statut, des en-têtes et éventuellement un document JSON.

Le handler doit rester fin. Il décode les données entrantes, vérifie leur format, appelle le usecase et transforme le résultat en réponse. Les règles métier ne doivent pas dépendre de `http.ResponseWriter` ou de `http.Request`, afin de rester testables sans serveur HTTP.

Une API robuste traite aussi les chemins d’erreur : JSON invalide, paramètre absent, validation refusée, ressource inconnue, timeout et erreur interne. Le client doit recevoir un statut cohérent et un message exploitable, sans détails techniques sensibles.

## 9.1 Endpoint simple en Go avec `net/http`

### Go

```go
package main

import (
    "encoding/json"
    "net/http"
)

func main() {
    mux := http.NewServeMux()

    mux.HandleFunc("GET /health", healthHandler)

    http.ListenAndServe(":8080", mux)
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]string{
        "status": "ok",
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
```

**Explication :** Le serveur associe ici une route à un handler, puis écoute les connexions HTTP. Chaque requête est traitée concurremment par `net/http`, même si aucune goroutine n’est créée explicitement dans le handler. Les dépendances partagées doivent donc être sûres pour un accès concurrent et le serveur doit gérer proprement ses erreurs de démarrage.

Test :

```bash
curl http://localhost:8080/health
```

**Explication :** Cette requête `curl` permet de vérifier manuellement le contrat HTTP sans frontend. Les options indiquent la méthode, les en-têtes et éventuellement le corps JSON envoyé au serveur. Ce contrôle rapide complète les tests automatisés, mais ne les remplace pas car il ne vérifie pas durablement les régressions.

### PHP équivalent natif

```php
<?php

if ($_SERVER['REQUEST_METHOD'] === 'GET' && $_SERVER['REQUEST_URI'] === '/health') {
    header('Content-Type: application/json');

    echo json_encode([
        'status' => 'ok',
    ]);

    exit;
}

http_response_code(404);
echo 'Not found';
```

**Explication :** Le code PHP transforme le JSON en tableau ou objet dynamique, puis construit la réponse. Cette souplesse réduit le code initial mais reporte plusieurs erreurs de forme à l’exécution. En Go, une struct de requête ou de réponse documente les champs attendus et permet au compilateur de vérifier leur utilisation.

### PHP équivalent Symfony

```php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Attribute\Route;

final class HealthController
{
    #[Route('/health', methods: ['GET'])]
    public function __invoke(): JsonResponse
    {
        return new JsonResponse([
            'status' => 'ok',
        ]);
    }
}
```

**Explication :** Symfony prend ici en charge le routage et une partie de la construction de la réponse HTTP. Avec `net/http`, Go expose des primitives plus basses et rend l’enchaînement plus explicite. Le handler doit néanmoins rester mince dans les deux écosystèmes : décoder, valider la forme, appeler le cas d’usage et traduire le résultat en HTTP.

---

## 9.2 Handler avec path parameter

### Go

```go
func main() {
    mux := http.NewServeMux()

    mux.HandleFunc("GET /products/{id}", getProductHandler)

    http.ListenAndServe(":8080", mux)
}

func getProductHandler(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")

    response := map[string]string{
        "id":   id,
        "name": "Keyboard",
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
```

**Explication :** Le serveur associe ici une route à un handler, puis écoute les connexions HTTP. Chaque requête est traitée concurremment par `net/http`, même si aucune goroutine n’est créée explicitement dans le handler. Les dépendances partagées doivent donc être sûres pour un accès concurrent et le serveur doit gérer proprement ses erreurs de démarrage.

### PHP équivalent Symfony

```php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Attribute\Route;

final class ProductController
{
    #[Route('/products/{id}', methods: ['GET'])]
    public function getProduct(string $id): JsonResponse
    {
        return new JsonResponse([
            'id' => $id,
            'name' => 'Keyboard',
        ]);
    }
}
```

**Explication :** Symfony prend ici en charge le routage et une partie de la construction de la réponse HTTP. Avec `net/http`, Go expose des primitives plus basses et rend l’enchaînement plus explicite. Le handler doit néanmoins rester mince dans les deux écosystèmes : décoder, valider la forme, appeler le cas d’usage et traduire le résultat en HTTP.

---

## 9.3 Lire des query parameters

### Go

```go
func listProductsHandler(w http.ResponseWriter, r *http.Request) {
    query := r.URL.Query()

    search := query.Get("search")
    limit := query.Get("limit")

    response := map[string]string{
        "search": search,
        "limit":  limit,
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
```

**Explication :** Le package `encoding/json` convertit le corps HTTP vers une struct typée ou encode une valeur dans la réponse. Il faut traiter l’erreur de décodage avant d’appeler le métier et définir `Content-Type: application/json` avant d’écrire le statut. Pour une API stricte, limitez aussi la taille du body et refusez les champs inconnus.

URL :

```text
GET /products?search=keyboard&limit=10
```

**Explication :** Ce bloc représente le contrat HTTP utilisé dans « Go » : la méthode exprime l’intention et le chemin identifie la ressource. Il doit rester stable pour le frontend, même si l’implémentation interne change. Les entrées, réponses et erreurs associées doivent être documentées et testées avec le même niveau de précision.

### PHP équivalent

```php
<?php

$search = $_GET['search'] ?? '';
$limit = $_GET['limit'] ?? '';

header('Content-Type: application/json');

echo json_encode([
    'search' => $search,
    'limit' => $limit,
]);
```

**Explication :** Le code PHP transforme le JSON en tableau ou objet dynamique, puis construit la réponse. Cette souplesse réduit le code initial mais reporte plusieurs erreurs de forme à l’exécution. En Go, une struct de requête ou de réponse documente les champs attendus et permet au compilateur de vérifier leur utilisation.

### PHP équivalent Symfony

```php
<?php

public function listProducts(Request $request): JsonResponse
{
    $search = $request->query->get('search', '');
    $limit = $request->query->get('limit', '');

    return new JsonResponse([
        'search' => $search,
        'limit' => $limit,
    ]);
}
```

**Explication :** Symfony prend ici en charge le routage et une partie de la construction de la réponse HTTP. Avec `net/http`, Go expose des primitives plus basses et rend l’enchaînement plus explicite. Le handler doit néanmoins rester mince dans les deux écosystèmes : décoder, valider la forme, appeler le cas d’usage et traduire le résultat en HTTP.

---

## 9.4 Lire un body JSON

### Go

```go
type CreateProductRequest struct {
    Name  string  `json:"name"`
    Price float64 `json:"price"`
    Stock int     `json:"stock"`
}

func createProductHandler(w http.ResponseWriter, r *http.Request) {
    var request CreateProductRequest

    err := json.NewDecoder(r.Body).Decode(&request)
    if err != nil {
        http.Error(w, "invalid json", http.StatusBadRequest)
        return
    }

    response := map[string]any{
        "id":    "new-id",
        "name":  request.Name,
        "price": request.Price,
        "stock": request.Stock,
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(response)
}
```

**Explication :** Le package `encoding/json` convertit le corps HTTP vers une struct typée ou encode une valeur dans la réponse. Il faut traiter l’erreur de décodage avant d’appeler le métier et définir `Content-Type: application/json` avant d’écrire le statut. Pour une API stricte, limitez aussi la taille du body et refusez les champs inconnus.

### PHP équivalent natif

```php
<?php

$payload = json_decode(file_get_contents('php://input'), true, 512, JSON_THROW_ON_ERROR);

$response = [
    'id' => 'new-id',
    'name' => $payload['name'],
    'price' => $payload['price'],
    'stock' => $payload['stock'],
];

http_response_code(201);
header('Content-Type: application/json');

echo json_encode($response, JSON_THROW_ON_ERROR);
```

**Explication :** Le code PHP transforme le JSON en tableau ou objet dynamique, puis construit la réponse. Cette souplesse réduit le code initial mais reporte plusieurs erreurs de forme à l’exécution. En Go, une struct de requête ou de réponse documente les champs attendus et permet au compilateur de vérifier leur utilisation.

### PHP équivalent Symfony

```php
<?php

public function createProduct(Request $request): JsonResponse
{
    $payload = json_decode($request->getContent(), true, 512, JSON_THROW_ON_ERROR);

    return new JsonResponse([
        'id' => 'new-id',
        'name' => $payload['name'],
        'price' => $payload['price'],
        'stock' => $payload['stock'],
    ], 201);
}
```

**Explication :** Le code PHP transforme le JSON en tableau ou objet dynamique, puis construit la réponse. Cette souplesse réduit le code initial mais reporte plusieurs erreurs de forme à l’exécution. En Go, une struct de requête ou de réponse documente les champs attendus et permet au compilateur de vérifier leur utilisation.

---

## 9.5 Réponse JSON helper

### Go

```go
func writeJSON(w http.ResponseWriter, statusCode int, payload any) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(statusCode)

    err := json.NewEncoder(w).Encode(payload)
    if err != nil {
        http.Error(w, "failed to encode response", http.StatusInternalServerError)
    }
}
```

**Explication :** Le package `encoding/json` convertit le corps HTTP vers une struct typée ou encode une valeur dans la réponse. Il faut traiter l’erreur de décodage avant d’appeler le métier et définir `Content-Type: application/json` avant d’écrire le statut. Pour une API stricte, limitez aussi la taille du body et refusez les champs inconnus.

Utilisation :

```go
writeJSON(w, http.StatusOK, map[string]string{
    "status": "ok",
})
```

**Explication :** La map associe des clés et des valeurs de types fixes. Une lecture peut retourner la valeur zéro même si la clé n’existe pas ; la forme `value, ok := m[key]` permet de distinguer ces deux cas. Une map n’est pas sûre pour des écritures concurrentes et son ordre d’itération n’est pas garanti.

### PHP équivalent

```php
<?php

function writeJson(array $payload, int $statusCode = 200): void
{
    http_response_code($statusCode);
    header('Content-Type: application/json');

    echo json_encode($payload, JSON_THROW_ON_ERROR);
}
```

**Explication :** Le code PHP transforme le JSON en tableau ou objet dynamique, puis construit la réponse. Cette souplesse réduit le code initial mais reporte plusieurs erreurs de forme à l’exécution. En Go, une struct de requête ou de réponse documente les champs attendus et permet au compilateur de vérifier leur utilisation.

---

## 9.6 Gestion d’erreur HTTP

### Go

```go
func getProductHandler(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")

    if id == "" {
        writeJSON(w, http.StatusBadRequest, map[string]string{
            "error": "product id is required",
        })
        return
    }

    if id == "404" {
        writeJSON(w, http.StatusNotFound, map[string]string{
            "error": "product not found",
        })
        return
    }

    writeJSON(w, http.StatusOK, map[string]string{
        "id":   id,
        "name": "Keyboard",
    })
}
```

**Explication :** La map associe des clés et des valeurs de types fixes. Une lecture peut retourner la valeur zéro même si la clé n’existe pas ; la forme `value, ok := m[key]` permet de distinguer ces deux cas. Une map n’est pas sûre pour des écritures concurrentes et son ordre d’itération n’est pas garanti.

### PHP équivalent Symfony

```php
<?php

public function getProduct(string $id): JsonResponse
{
    if ($id === '') {
        return new JsonResponse([
            'error' => 'product id is required',
        ], 400);
    }

    if ($id === '404') {
        return new JsonResponse([
            'error' => 'product not found',
        ], 404);
    }

    return new JsonResponse([
        'id' => $id,
        'name' => 'Keyboard',
    ]);
}
```

**Explication :** Symfony prend ici en charge le routage et une partie de la construction de la réponse HTTP. Avec `net/http`, Go expose des primitives plus basses et rend l’enchaînement plus explicite. Le handler doit néanmoins rester mince dans les deux écosystèmes : décoder, valider la forme, appeler le cas d’usage et traduire le résultat en HTTP.

---

# 10. Décisions techniques pour les endpoints

Créer une route ne consiste pas seulement à choisir une URL. Chaque endpoint définit un contrat : données acceptées, règles de validation, statut de succès, structure des erreurs, délais et comportement lorsque les dépendances sont indisponibles.

Ces décisions doivent rester cohérentes dans toute l’API. Deux erreurs de validation comparables devraient avoir la même structure, et une ressource absente devrait produire le même type de réponse sur toutes les routes.

Dans un BFF, il faut également décider comment réagir à l’échec d’un service distant. Selon le besoin produit, l’application peut échouer complètement, retourner une réponse partielle, utiliser une valeur en cache ou appliquer un timeout plus court. Ce choix fait partie du contrat fonctionnel.

## 10.1 Choisir la méthode HTTP

| Action | Méthode | Exemple |
|---|---|---|
| Lire une ressource | GET | `GET /products/{id}` |
| Lister | GET | `GET /products` |
| Créer | POST | `POST /products` |
| Remplacer | PUT | `PUT /products/{id}` |
| Modifier partiellement | PATCH | `PATCH /products/{id}` |
| Supprimer | DELETE | `DELETE /products/{id}` |

---

## 10.2 Exemple de décision : GET `/products/{id}`

### Besoin front

Le front a besoin d’afficher la fiche produit.

### Route

```text
GET /products/{id}
```

**Explication :** Ce bloc représente le contrat HTTP utilisé dans « Route » : la méthode exprime l’intention et le chemin identifie la ressource. Il doit rester stable pour le frontend, même si l’implémentation interne change. Les entrées, réponses et erreurs associées doivent être documentées et testées avec le même niveau de précision.

### Réponse succès

```json
{
  "id": "1",
  "name": "Keyboard",
  "description": "Mechanical keyboard",
  "price": 79.99,
  "available": true
}
```

**Explication :** Ce JSON illustre le contrat public de « Réponse succès ». Les noms et types des champs font partie de l’API consommée par le frontend ; ils ne doivent donc pas dépendre directement de la structure interne du domaine. Un DTO et un mapper permettent de faire évoluer ces deux modèles séparément.

### Erreur produit introuvable

```json
{
  "error": "product not found"
}
```

**Explication :** Cette réponse JSON fournit au client une erreur structurée plutôt qu’un texte libre. Le champ `code` est destiné au traitement automatisé côté frontend, tandis que `message` reste lisible par un humain. Le statut HTTP doit porter la catégorie générale de l’échec et le corps doit apporter les détails applicatifs.

Code HTTP :

```text
404 Not Found
```

**Explication :** Ce bloc synthétise la forme attendue pour « Erreur produit introuvable ». Il sert de représentation intermédiaire avant l’implémentation : vérifiez que chaque élément a une responsabilité claire et qu’aucune dépendance technique ne remonte vers le domaine.

---

## 10.3 Mapping domaine vers réponse API

### Go

```go
type ProductResponse struct {
    ID          string  `json:"id"`
    Name        string  `json:"name"`
    Description string  `json:"description"`
    Price       float64 `json:"price"`
    Available   bool    `json:"available"`
}

func MapProductToResponse(product Product) ProductResponse {
    return ProductResponse{
        ID:          product.ID,
        Name:        product.Name,
        Description: product.Description,
        Price:       product.Price,
        Available:   product.IsAvailable(),
    }
}
```

**Explication :** Cette struct joue le rôle de contrat JSON grâce aux tags placés après chaque champ. Les champs exportés commencent par une majuscule afin que `encoding/json` puisse les lire ou les écrire. Séparez les DTO HTTP des structures du domaine lorsque leurs contraintes ou leur rythme d’évolution diffèrent.

### PHP équivalent

```php
<?php

final class ProductResponse
{
    public function __construct(
        public string $id,
        public string $name,
        public string $description,
        public float $price,
        public bool $available,
    ) {}
}

function mapProductToResponse(Product $product): ProductResponse
{
    return new ProductResponse(
        id: $product->id,
        name: $product->name,
        description: $product->description,
        price: $product->price,
        available: $product->isAvailable(),
    );
}
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

## 10.4 Pourquoi ne pas exposer directement le modèle domaine ?

### Mauvais exemple Go

```go
json.NewEncoder(w).Encode(product)
```

**Explication :** Le package `encoding/json` convertit le corps HTTP vers une struct typée ou encode une valeur dans la réponse. Il faut traiter l’erreur de décodage avant d’appeler le métier et définir `Content-Type: application/json` avant d’écrire le statut. Pour une API stricte, limitez aussi la taille du body et refusez les champs inconnus.

Problème :

- le domaine peut changer ;
- le front devient dépendant du modèle interne ;
- on expose parfois trop d’informations ;
- le format API n’est pas maîtrisé.

### Meilleur exemple Go

```go
response := MapProductToResponse(product)
json.NewEncoder(w).Encode(response)
```

**Explication :** Le package `encoding/json` convertit le corps HTTP vers une struct typée ou encode une valeur dans la réponse. Il faut traiter l’erreur de décodage avant d’appeler le métier et définir `Content-Type: application/json` avant d’écrire le statut. Pour une API stricte, limitez aussi la taille du body et refusez les champs inconnus.

### PHP équivalent

```php
<?php

// Mauvais :
return new JsonResponse($product);

// Mieux :
return new JsonResponse($this->productPresenter->present($product));
```

**Explication :** Symfony prend ici en charge le routage et une partie de la construction de la réponse HTTP. Avec `net/http`, Go expose des primitives plus basses et rend l’enchaînement plus explicite. Le handler doit néanmoins rester mince dans les deux écosystèmes : décoder, valider la forme, appeler le cas d’usage et traduire le résultat en HTTP.

---

## 10.5 Codes HTTP recommandés

| Cas | Code |
|---|---|
| Succès lecture | 200 |
| Création | 201 |
| Requête invalide | 400 |
| Non authentifié | 401 |
| Non autorisé | 403 |
| Ressource introuvable | 404 |
| Conflit métier | 409 |
| Erreur serveur | 500 |
| Service externe indisponible | 502 ou 503 |
| Timeout service externe | 504 |

---

## 10.6 Validation : handler ou usecase ?

Règle pratique :

| Validation | Où ? |
|---|---|
| JSON invalide | Handler |
| Paramètre manquant | Handler ou usecase |
| Format HTTP invalide | Handler |
| Règle métier | Usecase |
| Contrôle d’accès | Middleware ou usecase selon contexte |

---

## 10.7 Exemple de validation complète

### Go

```go
type CreateProductRequest struct {
    Name  string  `json:"name"`
    Price float64 `json:"price"`
    Stock int     `json:"stock"`
}

func (r CreateProductRequest) Validate() error {
    if r.Name == "" {
        return errors.New("name is required")
    }

    if r.Price <= 0 {
        return errors.New("price must be greater than zero")
    }

    if r.Stock < 0 {
        return errors.New("stock cannot be negative")
    }

    return nil
}
```

**Explication :** Cette struct joue le rôle de contrat JSON grâce aux tags placés après chaque champ. Les champs exportés commencent par une majuscule afin que `encoding/json` puisse les lire ou les écrire. Séparez les DTO HTTP des structures du domaine lorsque leurs contraintes ou leur rythme d’évolution diffèrent.

### PHP équivalent

```php
<?php

final class CreateProductRequest
{
    public function __construct(
        public string $name,
        public float $price,
        public int $stock,
    ) {}

    public function validate(): void
    {
        if ($this->name === '') {
            throw new InvalidArgumentException('name is required');
        }

        if ($this->price <= 0) {
            throw new InvalidArgumentException('price must be greater than zero');
        }

        if ($this->stock < 0) {
            throw new InvalidArgumentException('stock cannot be negative');
        }
    }
}
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

# 11. Tests unitaires Go et équivalents PHP

Un test unitaire vérifie un comportement précis avec des dépendances contrôlées. Le package `testing` est intégré au langage : les fonctions de test commencent par `Test`, reçoivent un `*testing.T` et sont lancées avec `go test`.

Les table-driven tests regroupent plusieurs scénarios autour du même comportement. Chaque cas décrit les entrées, le résultat attendu et, si nécessaire, l’erreur attendue. Cette structure est particulièrement utile pour les validations et les fonctions comportant plusieurs cas limites.

Pour tester un usecase sans base de données, on lui injecte une fausse implémentation du repository. Le test vérifie alors la règle métier sans dépendre du réseau, du stockage ou du protocole HTTP.

Un bon test décrit un résultat observable plutôt que l’implémentation ligne par ligne. Le code peut ainsi être refactoré sans rendre toute la suite de tests obsolète.

## 11.1 Test simple

### Go

Fichier : `calculator_test.go`

```go
package calculator

import "testing"

func Add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    result := Add(2, 3)

    if result != 5 {
        t.Fatalf("expected 5, got %d", result)
    }
}
```

**Explication :** Go déduit les types à partir des valeurs tout en conservant un typage statique. `:=` déclare au moins une nouvelle variable dans la portée courante, tandis que `var` convient aux déclarations explicites, aux valeurs zéro et au niveau package. Après la déclaration, une variable ne peut pas recevoir une valeur d’un autre type.

Commande :

```bash
go test
```

**Explication :** Cette commande compile puis exécute les tests Go. Le motif `./...` inclut le package courant et tous ses sous-packages, ce qui convient à la validation complète d’un module. Une erreur de compilation interrompt les tests : le typage fait donc partie du premier niveau de vérification.

### PHP équivalent avec PHPUnit

```php
<?php

use PHPUnit\Framework\TestCase;

final class CalculatorTest extends TestCase
{
    public function testAdd(): void
    {
        $result = add(2, 3);

        self::assertSame(5, $result);
    }
}

function add(int $a, int $b): int
{
    return $a + $b;
}
```

**Explication :** Ce test PHP exprime le même scénario que le test Go voisin : préparation des données, exécution puis assertion sur le résultat. PHPUnit fournit davantage de méthodes d’assertion prêtes à l’emploi, tandis que le package `testing` de Go reste volontairement minimal. Dans les deux cas, un test doit vérifier un comportement observable plutôt qu’un détail interne.

Commande :

```bash
vendor/bin/phpunit
```

**Explication :** Cette commande sert à exécuter ou vérifier l’étape décrite dans « PHP équivalent avec PHPUnit ». Lancez-la depuis la racine du projet afin que Go ou PHP retrouve sa configuration et ses dépendances. Observez la sortie et le code de retour : une commande silencieuse avec un statut nul signifie généralement que l’étape a réussi.

---

## 11.2 Table-driven tests

Très courant en Go.

### Go

```go
func TestAddWithCases(t *testing.T) {
    tests := []struct {
        name     string
        a        int
        b        int
        expected int
    }{
        {
            name:     "positive numbers",
            a:        2,
            b:        3,
            expected: 5,
        },
        {
            name:     "negative numbers",
            a:        -2,
            b:        -3,
            expected: -5,
        },
        {
            name:     "zero",
            a:        0,
            b:        3,
            expected: 3,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)

            if result != tt.expected {
                t.Fatalf("expected %d, got %d", tt.expected, result)
            }
        })
    }
}
```

**Explication :** La slice représente une vue dynamique sur un tableau sous-jacent. `append` peut réutiliser ce stockage ou en allouer un nouveau, d’où la nécessité de récupérer la slice retournée. Lors d’un `range`, Go fournit l’index et une copie de la valeur ; utilisez l’index si vous devez modifier l’élément stocké.

### PHP équivalent avec data provider

```php
<?php

use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\TestCase;

final class CalculatorTest extends TestCase
{
    #[DataProvider('addProvider')]
    public function testAddWithCases(int $a, int $b, int $expected): void
    {
        self::assertSame($expected, add($a, $b));
    }

    public static function addProvider(): array
    {
        return [
            'positive numbers' => [2, 3, 5],
            'negative numbers' => [-2, -3, -5],
            'zero' => [0, 3, 3],
        ];
    }
}
```

**Explication :** Ce test PHP exprime le même scénario que le test Go voisin : préparation des données, exécution puis assertion sur le résultat. PHPUnit fournit davantage de méthodes d’assertion prêtes à l’emploi, tandis que le package `testing` de Go reste volontairement minimal. Dans les deux cas, un test doit vérifier un comportement observable plutôt qu’un détail interne.

---

## 11.3 Tester une erreur

### Go

```go
func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }

    return a / b, nil
}

func TestDivideByZero(t *testing.T) {
    _, err := Divide(10, 0)

    if err == nil {
        t.Fatal("expected error, got nil")
    }
}
```

**Explication :** L’erreur est une valeur retournée avec le résultat normal. L’appelant la vérifie immédiatement, ajoute éventuellement du contexte avec `fmt.Errorf(... %w ...)`, puis décide de la traiter ou de la propager. Ne masquez pas l’erreur : son message et sa cause sont essentiels pour le diagnostic et pour la traduction vers une réponse HTTP adaptée.

### PHP équivalent

```php
<?php

function divide(int $a, int $b): int
{
    if ($b === 0) {
        throw new InvalidArgumentException('division by zero');
    }

    return intdiv($a, $b);
}

final class DivideTest extends TestCase
{
    public function testDivideByZero(): void
    {
        $this->expectException(InvalidArgumentException::class);

        divide(10, 0);
    }
}
```

**Explication :** PHP signale ici l’échec avec une exception qui interrompt le flux normal jusqu’à ce qu’un `catch` la traite. Go préfère généralement retourner explicitement une valeur `error` et oblige chaque appelant à décider quoi en faire. L’équivalence est donc fonctionnelle, mais le chemin de contrôle reste différent.

---

## 11.4 Tester un usecase avec fake repository

### Go

```go
type fakeRepository struct {
    product Product
    err     error
}

func (r fakeRepository) FindByID(ctx context.Context, id string) (Product, error) {
    if r.err != nil {
        return Product{}, r.err
    }

    return r.product, nil
}

func (r fakeRepository) FindAll(ctx context.Context) ([]Product, error) {
    return []Product{r.product}, nil
}

func (r fakeRepository) Save(ctx context.Context, product Product) error {
    return nil
}

func TestGetProduct(t *testing.T) {
    repo := fakeRepository{
        product: Product{
            ID:    "1",
            Name:  "Keyboard",
            Price: 79.99,
            Stock: 10,
        },
    }

    usecase := NewUseCase(repo)

    product, err := usecase.GetProduct(context.Background(), "1")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }

    if product.Name != "Keyboard" {
        t.Fatalf("expected Keyboard, got %s", product.Name)
    }
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « Go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

### PHP équivalent

```php
<?php

final class FakeProductRepository implements ProductRepositoryInterface
{
    public function __construct(
        private Product $product,
    ) {}

    public function findById(string $id): Product
    {
        return $this->product;
    }

    public function findAll(): array
    {
        return [$this->product];
    }

    public function save(Product $product): void
    {
    }
}

final class GetProductUseCaseTest extends TestCase
{
    public function testGetProduct(): void
    {
        $repository = new FakeProductRepository(
            new Product('1', 'Keyboard', 'Mechanical keyboard', 79.99, 10)
        );

        $useCase = new GetProductUseCase($repository);

        $product = $useCase->execute('1');

        self::assertSame('Keyboard', $product->name);
    }
}
```

**Explication :** Cette version PHP utilise une classe et la promotion des propriétés du constructeur pour représenter les mêmes données que l’exemple Go. En PHP, l’objet conserve une identité et est manipulé indirectement ; une struct Go est une valeur copiée par défaut. Cette différence devient importante lorsqu’une fonction ou une méthode doit modifier l’instance d’origine.

---

## 11.5 Tester un handler HTTP

### Go

```go
func TestHealthHandler(t *testing.T) {
    request := httptest.NewRequest(http.MethodGet, "/health", nil)
    recorder := httptest.NewRecorder()

    healthHandler(recorder, request)

    response := recorder.Result()

    if response.StatusCode != http.StatusOK {
        t.Fatalf("expected status 200, got %d", response.StatusCode)
    }
}
```

**Explication :** `httptest` exécute le handler en mémoire avec une requête et un enregistreur de réponse, sans ouvrir de port réseau. Le test vérifie ainsi la traduction HTTP — statut, en-têtes et corps — tout en restant rapide et déterministe. La logique métier complexe doit être testée séparément au niveau du usecase.

### PHP équivalent Symfony WebTestCase

```php
<?php

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

final class HealthControllerTest extends WebTestCase
{
    public function testHealth(): void
    {
        $client = static::createClient();

        $client->request('GET', '/health');

        self::assertResponseIsSuccessful();
        self::assertJson($client->getResponse()->getContent());
    }
}
```

**Explication :** Ce test PHP exprime le même scénario que le test Go voisin : préparation des données, exécution puis assertion sur le résultat. PHPUnit fournit davantage de méthodes d’assertion prêtes à l’emploi, tandis que le package `testing` de Go reste volontairement minimal. Dans les deux cas, un test doit vérifier un comportement observable plutôt qu’un détail interne.

---

# 12. Projet fil rouge : mini BFF Products API

Ce projet rassemble les notions précédentes dans une application exécutable. Il montre comment les packages, interfaces, handlers, usecases, repositories, mappers et tests collaborent dans un cas concret.

La structure proposée est une base pédagogique, pas une architecture universelle. Chaque fichier doit répondre à un besoin identifiable. Si une couche n’apporte aucune séparation ou testabilité dans un petit projet réel, elle peut être simplifiée.

La progression recommandée consiste à construire une tranche fonctionnelle après l’autre : endpoint de santé, liste des produits, lecture d’un produit, puis création. À chaque étape, le comportement doit être testé avant d’ajouter la suivante.

Le repository mémoire permet de se concentrer sur les règles sans installer de base de données. Une implémentation PostgreSQL ou un client HTTP pourra ensuite le remplacer derrière la même interface.

Pour comprendre l’ensemble, il faut suivre une requête complète : la route sélectionne le handler, le handler décode l’entrée, le usecase applique les règles, le repository fournit les données, puis le mapper construit la réponse JSON.

## 12.1 Objectif

Créer une API Go avec :

```text
GET /health
GET /products
GET /products/{id}
POST /products
```

**Explication :** Ce bloc représente le contrat HTTP utilisé dans « 12.1 Objectif » : la méthode exprime l’intention et le chemin identifie la ressource. Il doit rester stable pour le frontend, même si l’implémentation interne change. Les entrées, réponses et erreurs associées doivent être documentées et testées avec le même niveau de précision.

Avec :

- architecture propre ;
- handlers fins ;
- usecases testables ;
- repository abstrait ;
- mapping de réponse ;
- tests unitaires.

---

## 12.2 Structure complète

```text
product-api/
├── go.mod
├── cmd/
│   └── api/
│       └── main.go
└── internal/
    └── product/
        ├── domain.go
        ├── dto.go
        ├── handler.go
        ├── mapper.go
        ├── memory_repository.go
        ├── repository.go
        ├── usecase.go
        └── usecase_test.go
```

**Explication :** Cette arborescence montre la responsabilité de chaque fichier dans « 12.2 Structure complète ». Les dossiers servent à séparer les couches techniques, mais ce sont surtout les packages et leurs dépendances qui imposent l’architecture. En pratique, commencez avec cette structure minimale, puis ajoutez un fichier seulement lorsqu’une responsabilité distincte apparaît.

---

## 12.3 `go.mod`

```go
module example.com/product-api

go 1.23
```

**Explication :** La directive `module` fixe l’identité du projet et le préfixe de ses imports internes. La directive `go` indique la version de langage minimale attendue par les outils, sans installer ni verrouiller elle-même le compilateur. Les dépendances externes seront ajoutées dans ce fichier et leurs sommes de contrôle seront conservées dans `go.sum`.

---

## 12.4 `internal/product/domain.go`

```go
package product

type Product struct {
    ID          string
    Name        string
    Description string
    Price       float64
    Stock       int
}

func (p Product) IsAvailable() bool {
    return p.Stock > 0
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « 12.4 internal/product/domain.go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

---

## 12.5 `internal/product/dto.go`

```go
package product

type ProductResponse struct {
    ID          string  `json:"id"`
    Name        string  `json:"name"`
    Description string  `json:"description"`
    Price       float64 `json:"price"`
    Available   bool    `json:"available"`
}

type CreateProductRequest struct {
    Name        string  `json:"name"`
    Description string  `json:"description"`
    Price       float64 `json:"price"`
    Stock       int     `json:"stock"`
}
```

**Explication :** Cette struct joue le rôle de contrat JSON grâce aux tags placés après chaque champ. Les champs exportés commencent par une majuscule afin que `encoding/json` puisse les lire ou les écrire. Séparez les DTO HTTP des structures du domaine lorsque leurs contraintes ou leur rythme d’évolution diffèrent.

---

## 12.6 `internal/product/mapper.go`

```go
package product

func MapProductToResponse(product Product) ProductResponse {
    return ProductResponse{
        ID:          product.ID,
        Name:        product.Name,
        Description: product.Description,
        Price:       product.Price,
        Available:   product.IsAvailable(),
    }
}

func MapProductsToResponse(products []Product) []ProductResponse {
    response := make([]ProductResponse, 0, len(products))

    for _, product := range products {
        response = append(response, MapProductToResponse(product))
    }

    return response
}
```

**Explication :** La slice représente une vue dynamique sur un tableau sous-jacent. `append` peut réutiliser ce stockage ou en allouer un nouveau, d’où la nécessité de récupérer la slice retournée. Lors d’un `range`, Go fournit l’index et une copie de la valeur ; utilisez l’index si vous devez modifier l’élément stocké.

---

## 12.7 `internal/product/repository.go`

```go
package product

import "context"

type Repository interface {
    FindAll(ctx context.Context) ([]Product, error)
    FindByID(ctx context.Context, id string) (Product, error)
    Save(ctx context.Context, product Product) error
}
```

**Explication :** Cette interface décrit uniquement les opérations dont l’appelant a besoin. En Go, un type satisfait l’interface implicitement dès qu’il possède les bonnes méthodes : le domaine n’a donc pas à dépendre d’une implémentation mémoire, SQL ou HTTP. Gardez l’interface petite et définissez-la près du code qui la consomme.

---

## 12.8 `internal/product/usecase.go`

```go
package product

import (
    "context"
    "errors"
    "strconv"
    "time"
)

var ErrProductNotFound = errors.New("product not found")

type UseCase struct {
    repository Repository
}

func NewUseCase(repository Repository) UseCase {
    return UseCase{
        repository: repository,
    }
}

func (uc UseCase) ListProducts(ctx context.Context) ([]Product, error) {
    return uc.repository.FindAll(ctx)
}

func (uc UseCase) GetProduct(ctx context.Context, id string) (Product, error) {
    if id == "" {
        return Product{}, errors.New("product id is required")
    }

    return uc.repository.FindByID(ctx, id)
}

func (uc UseCase) CreateProduct(ctx context.Context, request CreateProductRequest) (Product, error) {
    if request.Name == "" {
        return Product{}, errors.New("name is required")
    }

    if request.Price <= 0 {
        return Product{}, errors.New("price must be greater than zero")
    }

    if request.Stock < 0 {
        return Product{}, errors.New("stock cannot be negative")
    }

    newProduct := Product{
        ID:          strconv.FormatInt(time.Now().UnixNano(), 10),
        Name:        request.Name,
        Description: request.Description,
        Price:       request.Price,
        Stock:       request.Stock,
    }

    err := uc.repository.Save(ctx, newProduct)
    if err != nil {
        return Product{}, err
    }

    return newProduct, nil
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « 12.8 internal/product/usecase.go ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

---

## 12.9 `internal/product/memory_repository.go`

```go
package product

import (
    "context"
    "sync"
)

type MemoryRepository struct {
    mu       sync.RWMutex
    products map[string]Product
}

func NewMemoryRepository() *MemoryRepository {
    return &MemoryRepository{
        products: map[string]Product{
            "1": {
                ID:          "1",
                Name:        "Keyboard",
                Description: "Mechanical keyboard",
                Price:       79.99,
                Stock:       10,
            },
            "2": {
                ID:          "2",
                Name:        "Mouse",
                Description: "Wireless mouse",
                Price:       29.99,
                Stock:       0,
            },
        },
    }
}

func (r *MemoryRepository) FindAll(ctx context.Context) ([]Product, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    products := make([]Product, 0, len(r.products))

    for _, product := range r.products {
        products = append(products, product)
    }

    return products, nil
}

func (r *MemoryRepository) FindByID(ctx context.Context, id string) (Product, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    product, exists := r.products[id]
    if !exists {
        return Product{}, ErrProductNotFound
    }

    return product, nil
}

func (r *MemoryRepository) Save(ctx context.Context, product Product) error {
    r.mu.Lock()
    defer r.mu.Unlock()

    r.products[product.ID] = product

    return nil
}
```

**Explication :** Le mutex protège la section critique qui lit ou modifie l’état partagé. Une seule goroutine peut exécuter cette zone à la fois, ce qui supprime la race condition montrée précédemment. La section verrouillée doit rester courte et toujours libérer le verrou, généralement avec `defer mu.Unlock()`.

---

## 12.10 `internal/product/handler.go`

```go
package product

import (
    "encoding/json"
    "errors"
    "net/http"
)

type Handler struct {
    usecase UseCase
}

func NewHandler(usecase UseCase) Handler {
    return Handler{
        usecase: usecase,
    }
}

func (h Handler) RegisterRoutes(mux *http.ServeMux) {
    mux.HandleFunc("GET /products", h.ListProducts)
    mux.HandleFunc("GET /products/{id}", h.GetProduct)
    mux.HandleFunc("POST /products", h.CreateProduct)
}

func (h Handler) ListProducts(w http.ResponseWriter, r *http.Request) {
    products, err := h.usecase.ListProducts(r.Context())
    if err != nil {
        writeJSON(w, http.StatusInternalServerError, map[string]string{
            "error": "failed to list products",
        })
        return
    }

    writeJSON(w, http.StatusOK, MapProductsToResponse(products))
}

func (h Handler) GetProduct(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")

    product, err := h.usecase.GetProduct(r.Context(), id)
    if err != nil {
        if errors.Is(err, ErrProductNotFound) {
            writeJSON(w, http.StatusNotFound, map[string]string{
                "error": "product not found",
            })
            return
        }

        writeJSON(w, http.StatusBadRequest, map[string]string{
            "error": err.Error(),
        })
        return
    }

    writeJSON(w, http.StatusOK, MapProductToResponse(product))
}

func (h Handler) CreateProduct(w http.ResponseWriter, r *http.Request) {
    var request CreateProductRequest

    err := json.NewDecoder(r.Body).Decode(&request)
    if err != nil {
        writeJSON(w, http.StatusBadRequest, map[string]string{
            "error": "invalid json",
        })
        return
    }

    product, err := h.usecase.CreateProduct(r.Context(), request)
    if err != nil {
        writeJSON(w, http.StatusBadRequest, map[string]string{
            "error": err.Error(),
        })
        return
    }

    writeJSON(w, http.StatusCreated, MapProductToResponse(product))
}

func writeJSON(w http.ResponseWriter, statusCode int, payload any) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(statusCode)

    err := json.NewEncoder(w).Encode(payload)
    if err != nil {
        http.Error(w, "failed to encode response", http.StatusInternalServerError)
    }
}
```

**Explication :** Le package `encoding/json` convertit le corps HTTP vers une struct typée ou encode une valeur dans la réponse. Il faut traiter l’erreur de décodage avant d’appeler le métier et définir `Content-Type: application/json` avant d’écrire le statut. Pour une API stricte, limitez aussi la taille du body et refusez les champs inconnus.

---

## 12.11 `cmd/api/main.go`

```go
package main

import (
    "fmt"
    "net/http"

    "example.com/product-api/internal/product"
)

func main() {
    mux := http.NewServeMux()

    mux.HandleFunc("GET /health", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusOK)
        _, _ = w.Write([]byte(`{"status":"ok"}`))
    })

    repository := product.NewMemoryRepository()
    usecase := product.NewUseCase(repository)
    handler := product.NewHandler(usecase)

    handler.RegisterRoutes(mux)

    fmt.Println("server started on :8080")

    err := http.ListenAndServe(":8080", mux)
    if err != nil {
        panic(err)
    }
}
```

**Explication :** Le serveur associe ici une route à un handler, puis écoute les connexions HTTP. Chaque requête est traitée concurremment par `net/http`, même si aucune goroutine n’est créée explicitement dans le handler. Les dépendances partagées doivent donc être sûres pour un accès concurrent et le serveur doit gérer proprement ses erreurs de démarrage.

---

## 12.12 Tester avec curl

```bash
curl http://localhost:8080/health
```

**Explication :** Cette requête `curl` permet de vérifier manuellement le contrat HTTP sans frontend. Les options indiquent la méthode, les en-têtes et éventuellement le corps JSON envoyé au serveur. Ce contrôle rapide complète les tests automatisés, mais ne les remplace pas car il ne vérifie pas durablement les régressions.

```bash
curl http://localhost:8080/products
```

**Explication :** Cette requête `curl` permet de vérifier manuellement le contrat HTTP sans frontend. Les options indiquent la méthode, les en-têtes et éventuellement le corps JSON envoyé au serveur. Ce contrôle rapide complète les tests automatisés, mais ne les remplace pas car il ne vérifie pas durablement les régressions.

```bash
curl http://localhost:8080/products/1
```

**Explication :** Cette requête `curl` permet de vérifier manuellement le contrat HTTP sans frontend. Les options indiquent la méthode, les en-têtes et éventuellement le corps JSON envoyé au serveur. Ce contrôle rapide complète les tests automatisés, mais ne les remplace pas car il ne vérifie pas durablement les régressions.

```bash
curl -X POST http://localhost:8080/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "USB-C Cable",
    "description": "Fast charging cable",
    "price": 12.99,
    "stock": 20
  }'
```

**Explication :** Cette requête `curl` permet de vérifier manuellement le contrat HTTP sans frontend. Les options indiquent la méthode, les en-têtes et éventuellement le corps JSON envoyé au serveur. Ce contrôle rapide complète les tests automatisés, mais ne les remplace pas car il ne vérifie pas durablement les régressions.

---

## 12.13 Test du usecase

```go
package product

import (
    "context"
    "testing"
)

type fakeRepository struct {
    products map[string]Product
}

func (r fakeRepository) FindAll(ctx context.Context) ([]Product, error) {
    products := make([]Product, 0, len(r.products))

    for _, product := range r.products {
        products = append(products, product)
    }

    return products, nil
}

func (r fakeRepository) FindByID(ctx context.Context, id string) (Product, error) {
    product, exists := r.products[id]
    if !exists {
        return Product{}, ErrProductNotFound
    }

    return product, nil
}

func (r fakeRepository) Save(ctx context.Context, product Product) error {
    r.products[product.ID] = product
    return nil
}

func TestGetProduct(t *testing.T) {
    repository := fakeRepository{
        products: map[string]Product{
            "1": {
                ID:    "1",
                Name:  "Keyboard",
                Price: 79.99,
                Stock: 10,
            },
        },
    }

    usecase := NewUseCase(repository)

    product, err := usecase.GetProduct(context.Background(), "1")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }

    if product.Name != "Keyboard" {
        t.Fatalf("expected Keyboard, got %s", product.Name)
    }
}

func TestGetProductNotFound(t *testing.T) {
    repository := fakeRepository{
        products: map[string]Product{},
    }

    usecase := NewUseCase(repository)

    _, err := usecase.GetProduct(context.Background(), "404")
    if err == nil {
        t.Fatal("expected error, got nil")
    }
}

func TestCreateProduct(t *testing.T) {
    repository := fakeRepository{
        products: map[string]Product{},
    }

    usecase := NewUseCase(repository)

    product, err := usecase.CreateProduct(context.Background(), CreateProductRequest{
        Name:        "USB-C Cable",
        Description: "Fast charging cable",
        Price:       12.99,
        Stock:       20,
    })

    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }

    if product.ID == "" {
        t.Fatal("expected product ID to be generated")
    }

    if product.Name != "USB-C Cable" {
        t.Fatalf("expected USB-C Cable, got %s", product.Name)
    }
}
```

**Explication :** Cette struct regroupe des champs nommés et typés pour représenter « 12.13 Test du usecase ». Contrairement à un tableau associatif PHP, sa forme est connue à la compilation et tous les accès aux champs sont vérifiés. Une struct est copiée lorsqu’elle est passée par valeur ; utilisez un pointeur seulement si la mutation, l’identité ou le coût de copie le justifie.

---

# 13. Exercices en ligne et progression

Les exercices doivent être réalisés progressivement, sans recopier immédiatement le projet final. L’objectif est d’apprendre à interpréter les erreurs du compilateur, rechercher dans la documentation, formater le code et vérifier le comportement avec un test.

Pour chaque exercice, commencez par formuler le résultat attendu, écrivez la version minimale qui fonctionne, puis ajoutez au moins un cas d’erreur. Comparez ensuite avec une implémentation PHP afin d’identifier ce que le typage statique et les erreurs explicites changent.

La concurrence vient après les bases, les fonctions et HTTP. Une goroutine ne doit pas être ajoutée uniquement pour rendre le code « plus Go », mais lorsqu’elle représente réellement une tâche indépendante ou qu’elle réduit un temps d’attente mesurable.

## 13.1 Exercices niveau débutant

À faire dans cet ordre :

1. Hello World
2. Variables
3. Constants
4. Functions
5. Multiple return values
6. Arrays
7. Slices
8. Maps
9. Structs
10. Methods
11. Errors

---

## 13.2 Exercices niveau backend

1. Créer un endpoint `/health`.
2. Créer un endpoint `GET /products`.
3. Créer un endpoint `GET /products/{id}`.
4. Créer un endpoint `POST /products`.
5. Ajouter une validation.
6. Ajouter une erreur `404`.
7. Ajouter une réponse JSON standardisée.
8. Ajouter un usecase.
9. Ajouter une interface repository.
10. Ajouter un repository mémoire.
11. Ajouter des tests unitaires.

---

## 13.3 Exercices niveau concurrence

1. Lancer 3 goroutines qui affichent un message.
2. Utiliser un `WaitGroup`.
3. Envoyer une valeur dans un channel.
4. Créer un worker pool.
5. Ajouter un timeout avec `context.WithTimeout`.
6. Protéger une map avec `sync.Mutex`.
7. Lancer les tests avec le race detector :

```bash
go test -race ./...
```

**Explication :** L’option `-race` instrumente le programme pendant les tests afin de détecter des accès concurrents non synchronisés à la même mémoire. Elle consomme davantage de temps et de mémoire qu’un test normal, mais doit être exécutée régulièrement sur tout code utilisant des goroutines ou un état partagé. Un test réussi sans `-race` ne garantit pas l’absence de race condition.

---

## 13.4 Ressources recommandées

### Documentation officielle Go

- https://go.dev/doc/
- https://go.dev/tour/
- https://go.dev/doc/effective_go
- https://go.dev/doc/code

### Exercices

- https://exercism.org/tracks/go
- https://gobyexample.com/
- https://go.dev/tour/

### PHP pour les comparaisons

- https://www.php.net/manual/en/
- https://www.php.net/manual/en/language.types.array.php
- https://www.php.net/manual/en/language.oop5.basic.php
- https://www.php.net/manual/en/language.exceptions.php

---

# Checklist finale de maîtrise

À la fin de cette formation, le participant doit être capable de :

- expliquer la structure d’un fichier Go ;
- créer un module Go ;
- utiliser `go run`, `go build`, `go test`, `go fmt` ;
- déclarer des variables ;
- utiliser arrays, slices, maps et structs ;
- écrire des fonctions avec retours multiples ;
- gérer les erreurs avec `error` ;
- organiser le code en packages ;
- utiliser `for` et `range` ;
- comprendre les goroutines ;
- utiliser un `WaitGroup` ;
- comprendre les channels ;
- créer un endpoint HTTP ;
- lire path params, query params et body JSON ;
- écrire une réponse JSON ;
- séparer handler, usecase, repository et domaine ;
- écrire des tests unitaires ;
- expliquer les différences principales avec PHP.

---

# Mini-évaluation finale

## Questions

1. Quelle est la différence entre un array et une slice en Go ?
2. Pourquoi Go retourne souvent `(value, error)` ?
3. Quelle est la différence entre `var` et `:=` ?
4. À quoi sert `go.mod` ?
5. Pourquoi ne faut-il pas mettre toute la logique dans un handler HTTP ?
6. Quand utiliser une interface ?
7. Quelle est la différence entre une map Go et un array PHP ?
8. À quoi sert `context.Context` ?
9. Quel est le rôle d’un mapper dans un BFF ?
10. Comment tester un usecase sans base de données ?

## Exercice final

Créer une API Go `orders-api` avec :

```text
GET /health
GET /orders
GET /orders/{id}
POST /orders
```

**Explication :** Ce bloc représente le contrat HTTP utilisé dans « Exercice final » : la méthode exprime l’intention et le chemin identifie la ressource. Il doit rester stable pour le frontend, même si l’implémentation interne change. Les entrées, réponses et erreurs associées doivent être documentées et testées avec le même niveau de précision.

Avec :

- domaine `Order` ;
- usecase ;
- repository interface ;
- repository mémoire ;
- handler HTTP ;
- mapper ;
- tests unitaires ;
- gestion d’erreurs ;
- réponse JSON propre.
