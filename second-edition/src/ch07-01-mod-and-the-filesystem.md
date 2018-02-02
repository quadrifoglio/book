## `mod` et le sytème de fichier

Nous alons commencer notre exemple de module en créant un nouveau projet avec
Cargo, mais au lieu de créer un crate pour un binaire, nous allons créer un
crate pour une bibliothèque : ce sera un projet que les autres personnes
pourront intégrer dans leurs projets comme étant une dépendance. Par exemple,
le crate `rand` utilisé dans le chapitre 2 est un crate de bibliothèque que
nous avons utilisé comme dépendance quand le projet du jeu du plus ou moins.

Nous alons créer une structure de bibliothèque qui va apporter des
fonctionnalités pour des réseaux génériques; nous nous concentrerons sur
l'organisation des modules et des fonctions mais nous ne nous préoccuperons pas
du code qui est dans le corps des fonctions. Nous allons appeller notre
bibliothèque `communicator`. Par défaut, Cargo va créer une bibliothèque sauf
si un autre type de projet est précisé : si nous enlevons l'option `--bin` que
nous avons utilisé tout au long des chapitres précédents, notre projet sera une
bibliothèque :

```text
$ cargo new communicator
$ cd communicator
```

Notez que Cargo a généré *src/lib.rs* au lieu de *src/main.rs*. Dans
*src/lib.rs* nous avons ceci :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Cargo crée un exemple de test pour nous aider à démarrer notre bibliothèque,
à la place du code "Hello, world!" que nous avons quand nous utilisons l'option
`--bin`. Nous verrons les syntaxes `#[]` et `mod tests` dans la section
“Utiliser `super` pour accéder au module parent” plus tard dans ce chapitre,
mais pour le moment, laissez ce code en bas de *src/lib.rs*.

Comme nous n'avons pas de fichier *src/main.rs*, il n'y a rien à exécuter pour
Cargo avec la commande `cargo run`. C'est pourquoi nous allons utiliser la
commande `cargo build` pour compiler le code de notre crate de bibliothèque.

Nous examinerons plusieures façons d'organiser le code de notre bibliothèque
qui se prêterons à différentes situations, en fonction de la finalité du code.

### Définitions des Modules

Pour notre bibliothèque `communicator`, nous alons commencer par définir un
module `network` qui contiendra la définition d'une fonction `connect`. Chaque
définition de module dans Rust commence par le mot-clé `mod`. Ajoutez ce code
au début du fichier *src/lib.rs*, devant le code de test :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

Après le mot-clé `mod`, nous ajoutons le nom du module, `network`, et ensuite
un bloc de code entre les acollades. Tout ce qui est dans ce bloc est à
l'intérieur de l'espace de nom `network`. Ainsi, nous avons une seule fonction,
`connect`. Si nous voulons appeler cette fonction à partir de code à
l'extérieur du module `network`, nous avons besoin de préciser le module et
d'utiliser la syntaxe d'espace de nom `::`, comme ceci : `network::connect()`
et non pas juste `connect()`.

Nous pouvons aussi avoir plusieurs modules, l'un à coté de l'autre, dans le
même fichier *src/lib.rs*. Par exemple, pour avoir aussi un module `client` qui
a lui aussi une fonction `connect`, nous pouvons l'ajouter comme dans l'entrée
7-1 :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

<span class="caption">Entrée 7-1 : le module `network` et le module `client`,
définis l'un à coté de l'autre dans *src/lib.rs*</span>

Nous avons maintenant une fonction `network::connect` et une fonction
`client::connect`. Elles peuvent fonctionner complètement différemment, et les
noms de fonctions ne seront pas en conflit l'un envers l'autre car elles sont
dans des modules différents.

Dans notre exemple, comme nous construisons une bibliothèque, le fichier qui
sert de point d'entrée pour construire notre bibliothèque est *src/lib.rs*.
Nous pouvons aussi créer des modules dans *src/main.rs* pour un crate de
binaire de la même façon que nous l'avons fait dans *src/lib.rs* pour le crate
de bibliothèque. En fait, nous pouvons insérer des modules dans des modules,
ce qui peut être pratique lorsque vos modules grossisent, afin de garder les
fonctionnalités liées entre elles et séparer les fonctionnalités indépendantes.
Vous devez choisir comment organiser votre code en fonction de comment vous
envisagez les relations entre vos parties de codes. Par exemple, le code de
`client` et sa fonction `connect` peut avoir plus de sens aux utilisateurs
de notre bibliothèque si ils étaient plutôt dans l'espace de nom `network`,
comme dans l'entrée 7-2 :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

<span class="caption">Entrée 7-2 : déplacement du module `client` à l'intérieur
du module `network`</span>

Dans votre fichier *src/lib.rs*, remplacez le définitions existantes
`mod network` et `mod client` par celles de l'entrée 7-2, qui ont le module
`client` à l'interieur du module `network`. Maintenant nous avons les fonctions
`network::connect` et `network::client::connect` : de nouveau, les deux
fonctions `connect` ne sont pas en conflit l'un envers l'autre car elmes sont
dans des espaces de nom différents.

De cette façon, les modules construisent une hierarchie. Le contennu de
*src/lib.rs* sont à la plus haute place. Voici ce à quoi ressemble
l'organisation de notre exemple dans le module 7-1 quand nous analysons la
hierarchie :

```text
communicator
 ├── network
 └── client
```

Et voici la hierarchie correspondant à l'exemple dans l'entrée 7-2 :

```text
communicator
 └── network
     └── client
```

La hierarchie montre que dans l'entrée 7-2, `client` est un enfant du module
`network` plutôt qu'un frère. Les projets plus compliqués peuvent avoir de
nombreux modules, et ils auront besoin d'être organisés logiquement pour
pouvoir les maintenir. Ce que “logiquement” signifie dans votre projet dépends
de comment vous et les utilisateurs de la bibliothèque immaginent le domaine
de votre projet. Utilisez les techniques montrées içi pour créer des modules
l'un à coté de l'autre et les modules imbriqués l'un dans l'autre dans
n'importe quelle structure que vous avez besoin.

### Déplacer les modules dans d'autres fichiers

Les modules créent une structure hiérarchique, un peu comme une autre structure
informatique que vous avez déjà utilisé : le système de fichiers ! Nous pouvons
utiliser le système de module de Rust avec plusieurs fichiers pour découper les
projets Rust afin que tout ne soit pas dans *src/lib.rs* ou *src/main.rs*. Dans
notre exemple, nous alons commencer avec le code dans l'entrée 7-3 :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Entrée 7-3: trois modules, `client`, `network`, et
`network::server`, tous définis dans *src/lib.rs*</span>

Le fichier *src/lib.rs* a cette hierarchie de modules :

```text
communicator
 ├── client
 └── network
     └── server
```

Si ces modules avaient beaucoup de fonctions, et que ces fonctions devenaient
très verbeuses, il serait difficile de parcourir ce fichier pour trouver le
code avec lequel nous voulons travailler. Parce que ces fonctions sont
imbriquées dans un ou plusieurs blocs `mod`, les lignes de code à l'intérieur
des fonctions vont également s'allonger. Ce sont des bonnes raisons pour
séparer les modules `client`, `network`, et `server` de *src/lib.rs* et les
placer dans leurs propres fichiers.

Premièrement, remplacez le code du module `client` avec seulement la
déclaration du module `client`, de sorte que votre *src/lib.rs* ressemble au
code montré dans l'entrée 7-4 :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust,ignore
mod client;

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Entrée 7-4 : on déplace le contenu du module `client`
mais on laisse la déclaration dans *src/lib.rs*</span>

Nous continuons à *déclarer* the module `client` içi, mais en remplaçant le
bloc par un point-virgule, nous disons à Rust de chercher le code à autre
endroit, défini dans la portée du module `client`. Pour dire autrement, la
ligne `mod client;` signifie ceci :

```rust,ignore
mod client {
    // contenu du fichier client.rs içi
}
```

Maintenant, nous avons besoin de créer les fichiers avec ces noms de modules.
Créez donc un fichier *client.rs* dans votre répertoire *src/* et modifiez-le.
Saisissez alors le code suivant, qui est la fonction `connect` dans le module
`client` que nous avons enlevé dans l'étape précédente :

<span class="filename">Nom du fichier : src/client.rs</span>

```rust
fn connect() {
}
```

Observez que nous n'avons pas besoin de faire une déclaration avec `mod` dans
ce fichier car nous avons déjà déclaré le module `client` avec `mod` dans
*src/lib.rs*. Ce fichier donne juste le *contenu* du module `client`. Si nous
ajoutons un `mod client` içi, on ajouterais alors au module `client` son propre
sous-module qui s'appelle aussi `client` !

Rust ne sait que chercher dans *src/lib.rs* par défaut. Si nous voulons ajouter
plus de fichiers à notre projet, nous devons dire à Rust dans *src/lib.rs* de
chercher dans d'autres fichiers; c'est pourquoi `mod client` doit être placé
dans *src/lib/rs* et ne peut pas être utilisé dans *src/client.rs*.

Maintenant le projet devrait être compilé sans problèmes, bien que vous
obtiendrez quelques avertissements. N'oubliez pas d'utiliser `cargo build`
plutôt que `cargo run` car nous avons un crate de bibliothèque plutôt qu'un
crate de binaire :

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/lib.rs:4:5
  |
4 | /     fn connect() {
5 | |     }
  | |_____^

warning: function is never used: `connect`
 --> src/lib.rs:8:9
  |
8 | /         fn connect() {
9 | |         }
  | |_________^
```

Ces avertissements nous préviennent que nous avons des fonctions qui ne sont
jamais utilisées. Ne vous préoccupez pas de ces avertissements pour le moment;
nous nous en occuperons plus tard dans ce chapitre dans la section “Gérer la
visibilité avec `pub`”. La bonne nouvelle c'est que ce sont juste des
avertissements; notre projet a été compilé avec succès !

Ensuite, nous allons déplacer le module `network` dans son propre fichier en
suivant le même principe. Dans *src/lib.rs*, supprimez le corps du module
`network` et ajoutez un point-virgule à la déclaration, comme ceci :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust,ignore
mod client;

mod network;
```

Créez ensuite un nouveau fichier *src/network.rs* et saisissez le code
suivant :

<span class="filename">Nom du fichier : src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Notez que nous avons toujours une déclaration `mod` dans ce fichier de module;
c'est parceque nous voulons toujours que `server` soit un sous-module de
`network`.

Lancez `cargo build` à nouveau. Cela fonctionne ! Nous avons encore un autre
module à extraire : `server`. Comme c'est un sous-module (c'est à dire, un
module dans un module), notre principe actuel de déplacement des modules dans
un fichier qui s'appelle selon le nom du module ne va pas fonctionner. Nous
allons essayer tout de même et vous allez constater l'erreur. Pour commencer,
changez *src/network.rs* pour avoir `mod server;` plutôt que le contenu du
module `server` :

<span class="filename">Nom du fichier : src/network.rs</span>

```rust,ignore
fn connect() {
}

mod server;
```

Ensuite, créez un fichier *src/server.ts* et saisissez le contenu du module
`server` que nous avons extrait :

<span class="filename">Nom du fichier : src/server.rs</span>

```rust
fn connect() {
}
```

Lorsque nous essayons `cargo build`, nous avons l'erreur montrée dans l'entrée
7-5 :

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
error: cannot declare a new module at this location
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
  |
note: maybe move this module `src/network.rs` to its own directory via `src/network/mod.rs`
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
note: ... or maybe `use` the module `server` instead of possibly redeclaring it
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
```

<span class="caption">Entrée 7-5 : erreur lorsque nous essayons de déplacer le
sous-module `server` dans *src/server.rs*</span>

Cette erreur nous dit que nous *ne pouvons pas déclarer un nouveau module à cet
emplacement* et qu'elle pointe sur la ligne du `mod server;` dans
*src/network.rs*. Donc *src/network.rs* est en-soi différent de *src/lib.rs* :
continuons à lire pour comprendre pourquoi.

La note au millieu de l'entrée 7-5 est très utile dans notre cas car elle
souligne un point que nous n'avons pas encore parlé :

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

Plutôt que de continuer à suite la même stratégie de nommage des fichiers que
nous avons utilisé précedemment, nous pouvons faire ce que recommende la note :

1. Créer un nouveau *dossier* *network*, le nom du module parent.
2. Déplacer le fichier *src/network.rs* dans ce nouveau dossier *network*, et
   le renomer en *src/network/mod.rs*.
3. Déplacer le fichier de sous-module *src/server.rs* dans le dossier *network*

Voici les commandes bash pour procéder à ces changements :

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Maintenant, quand nous essayons de lancer `cargo build`, la compilation devrait
fonctionner (cependant, nous avons toujours des avertissements). Notre
hierarchie de module ressemble toujours à ceci, c'est exactement le même que
nous avions défini quand nous avions tout le code dans *src/lib.rs* dans
l'entrée 7-3 :

```text
communicator
 ├── client
 └── network
     └── server
```

La hierarchie de fichier ressemble maintenant à ceci :

```text
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

Mais, quand nous avons voulu déplacer le module `network::server`, pourquoi
avons-nous aussi déplacé le fichier *src/network.rs* dans le fichier
*src/network/mod.rs* en plus de déplacer le code de `network::server` dans le
fichier *src/network/server.rs*, plutôt que de simplement déplacer le module
`network::server` dans *src/server.rs* ? La raison est que Rust ne serait pas
capable de comprendre que ce `server` est supposé être un sous-module de
`network` si le fichier *server.rs* était dans le dossier *src*. Pour éclaircir
le fonctionnement de Rust dans ce cas, immaginons un cas différent avec la
hierarchie de modules suivant, où toutes les définitions sont dans
*src/lib.rs* :

```text
communicator
 ├── client
 └── network
     └── client
```

Dans cet exemple, nous avons aussi trois modules : `client`, `network`, et
`network::client`. En suivant les mêmes la même stratégie que nous avons
employé précédemment pour déplacer les modules dans des fichiers, nous allons
créer *src/client.rs* pour le module `client`. Pour le module `network`, nous
allons créer *src/network.rs*. Mais nous ne pourrons pas déplacer le module
`network::client` dans un fichier *src/client.rs* car il existe déjà pour le
module `client` du premier niveau ! Si nous pouvions mettre le code pour
*chacun* des modules `client` et `network::client` dans le fichier
*src/client.rs*, Rust n'aurait aucun moyen de savoir quel code serait pour
`client` ou pour `network::client`.

Cependant, pour pouvoir déplacer dans un fichier le sous-module
`network::client` du module `network`, nous avons besoin de créer un dossier
pour le module `network` plutôt qu'un fichier *src/network.rs*. Le code
qu'il y aura dans le module `network` ira alors dans le fichier
*src/network/mod.rs*, et le sous-module `network::client` peut avoir son
propre fichier *src/network/client.rs*. Désormais le premier niveau
*src/client.rs* n'est plus en en conflit et le code est clairement utilisé pour
le module `client`.

### Rules of Module Filesystems

Let’s summarize the rules of modules with regard to files:

* If a module named `foo` has no submodules, you should put the declarations
  for `foo` in a file named *foo.rs*.
* If a module named `foo` does have submodules, you should put the declarations
  for `foo` in a file named *foo/mod.rs*.

These rules apply recursively, so if a module named `foo` has a submodule named
`bar` and `bar` does not have submodules, you should have the following files
in your *src* directory:

```text
├── foo
│   ├── bar.rs (contains the declarations in `foo::bar`)
│   └── mod.rs (contains the declarations in `foo`, including `mod bar`)
```

The modules should be declared in their parent module’s file using the `mod`
keyword.

Next, we’ll talk about the `pub` keyword and get rid of those warnings!
