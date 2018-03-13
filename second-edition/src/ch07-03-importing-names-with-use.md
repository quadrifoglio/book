## Importer des dénominations de différents modules 

Nous avons vu comment appeler des fonctions définies dans un module en
utilisant le nom du module comme une partie de l'appel, comme dans l'appel à la
fonction `nested_modules` tel que montrée ici dans l'entrée 7-7 :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules();
}
```

<span class="caption">Entrée 7-7 : appel d'une fonction en précisant
entièrement le chemin des modules qui l'enveloppent</span>

Comme vous pouvez le constater, utiliser le chemin complet peut être un peu
long. Heureusement, Rust a un mot-clé pour rendre ces appels plus concis.

### Importer des dénominations dans la portée avec le mot-clé `use`

Le mot-clé `use` de Rust simplifie les longs appels des fonctions en important
les modules des fonctions que vous voulez appeler dans la portée (NdT : scope).
Voici un exemple d'ajout du module `a::series::of` dans la portée de la racine
d'un crate binaire :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of;

fn main() {
    of::nested_modules();
}
```

La ligne `use a::series::of;` signifie que plutôt qu'utiliser tout le chemin
`a::series::of` a chaque fois nous voulons utiliser le module `of`, nous
pouvons utiliser `of`.

Le mot-clé `use` importe uniquement ce que nous avons demandé dans la portée :
cela n'importe pas les enfants du module dans la portée. C'est pourquoi nous
devons toujours utiliser `of::nested_modules` losque nous souhaitons appeler
la fonction `nested_modules`.

Nous aurions pu choisir d'importer la fonction dans la portée en précisant
plutôt la fonction dans le `use`, comme ci-dessous :

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of::nested_modules;

fn main() {
    nested_modules();
}
```

Faire ainsi nous permet d'exclure tous les modules et de référencer directement
la fonction.

Comme les énumérateurs (NdT : enums) forment eux aussi un sorte d'espace de nom
(NdT : namespace) comme les modules, nous pouvons ici aussi importer une
variante d'énumérateurs dans la portée. Pour n'importe quel type d'instruction
`use`, si vous voulez importer plusieurs éléments d'un même espace de nom dans
la portée, vous pouvez les lister en utilisant des accollades et des virgules
dans le dernier emplacement, comme ceci :

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}
```

Nous continuons à préciser l'espace de nom `TrafficLight` pour la variante
`Green` car nous n'avons pas inclus `Green` dans l'instruction `use`.

### Importer toutes les dénominations dans la portée

Pour importer tous les éléments d'un espace de nom dans la portée en une seule
fois, nous pouvons utiliser la syntaxe `*`, qui s'appelle
*l'opérateur global (NdT : glob operator)*. L'exemple suivant importe toutes
les variantes d'un énumérateur dans la portée sans avoir à les lister un par
un :

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::*;

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = Green;
}
```

Le `*` va importer tout les éléments visibles dans l'espace de nom de
`TrafficLight`. Vous devriez utiliser les opérateurs globaux avec modération :
ils sont partiques, mais cela peut aussi importer plus d'éléments que vous
aviez prévu et mener à des conflits de noms.

### Utiliser `super` pour accéder à un module parent

Comme nous l'avons vu au début de ce chapitre, quand vous créez un crate de
bibliothèque, Cargo crée un module `tests` pour vous. Analysons cela plus en
détail maintenant. Dans notre projet `communicator`, ouvrez *src/lib.rs* :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Le chapitre 11 expliquera plus en détail les tests, mais les éléments de cet
exemple devrait avoir du sens désormais : nous avons un module qui s'appelle
`tests` qui vie a coté de nos autres modules et contient une fonction
`it_works`. Même si nous avons des annotations spéciales, le module de tests
n'est qu'un module en plus ! Donc notre hierarchie de modules ressemble à
ceci :

```text
communicator
 ├── client
 ├── network
 |   └── client
 └── tests
```

Les tests permettent de vérifier le code au sein de notre bibliothèque, donc
essayons d'appeler notre fonction `client::connect` dans cette fonction
`it_works`, même si nous ne vérifions aucune fonctionnalité pour le moment.
Ceci ne fonctionnera pas pour le moment :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Lancez les tests en utilisant la commande `cargo test` :

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
error[E0433]: failed to resolve. Use of undeclared type or module `client`
 --> src/lib.rs:9:9
  |
9 |         client::connect();
  |         ^^^^^^ Use of undeclared type or module `client`
```

La compilation a échoué, mais pourquoi ? Nous n'avons pas besoin d'ajouter
`communicator::` devant les fonctions telle que nous les avons créés dans
*src/main.rs* car nous sommes bien ici dans le crate de la bibliothèque
`communicator`. La raison est que les chemins sont toujours relatifs au module
courant, qui est ici `tests`. La seule exception est dans une instruction
`use`, où les chemins sont par défaut relatifs à la racine du crate. Notre
module `tests` a besoin du module `client` dans sa portée !

Donc comment pouvons-nous remonter un module dans la hierarchie de modules afin
d'utiliser la fonction `client::connect` dans le module `tests` ? Dans le
module `tests`, nous pouvons utiliser les doubles deux-points pour faire
comprendre à Rust que nous souhaitons commencer à partir de la racine et lister
le chemin complet, comme ceci :

```rust,ignore
::client::connect();
```

Ou sinon, nous pouvons utiliser `super` pour remonter d'un module dans la
hiérarchie par rapport au module actuel, comme ceci :

```rust,ignore
super::client::connect();
```

Ces deux options ne semble pas être différentes dans le cas de cet exemple,
mais si vous êtes plus profondémment dans la hierarchie de modules, commencer
le chemin à partir de la racine à chaque fois rendra votre code plus long. Dans
ce cas, utiliser `super` pour vous déplacer du module courrant vers des modules
frères est un bon raccourci. De plus, si vous précisez le chemin à partir de la
racine dans de nombreux endroits de votre code et qu'ensuite vous réagencez vos
modules en déplaçant un sous-arbre vers un autre emplacement, vous allez avoir
besoin de mettre à jour le chemin dans de nombreux endroits, ce qui serait
pénible.

Cela pourrait aussi être pénible d'avoir à écrire `super::` dans chaque test,
mais vous avez déjà vu l'outil pour résoudre ce problème : `use` ! La
fonctionnalité `super::` change le chemin que vous donnez à `use` afin qu'il
soit relatif au module parent plutôt qu'au module racine.

C'est pour cela que, en particulier dans le module `tests`,
`use super::something` est bien souvent la meilleur solution. Donc maintenant
notre test ressemble à ceci :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    use super::client;

    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Quand nous lancons `cargo test` à nouveau, le test va être un succès et la
première partie du resultat du test donnera ceci :

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
     Running target/debug/communicator-92007ddb5330fa5a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

## Résumé

Maintenant vous connaissez quelques techniques pour organiser votre code !
Utilisez-les pour regrouper des fonctionnalités similaires ensemble, éviter
d'avoir des fichiers trop longs, et présenter une API publique organisée aux
utilisateurs de votre bibliothèque.

Au point suivant, nous allons étudier quelques structures de collections de
données de la bibliothèque standard que vous pourrez utiliser dans votre super
code bien organisé !
