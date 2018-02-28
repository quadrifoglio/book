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

So how do we get back up one module in the module hierarchy to call the
`client::connect` function in the `tests` module? In the `tests` module, we can
either use leading colons to let Rust know that we want to start from the root
and list the whole path, like this:

```rust,ignore
::client::connect();
```

Or, we can use `super` to move up one module in the hierarchy from our current
module, like this:

```rust,ignore
super::client::connect();
```

These two options don’t look that different in this example, but if you’re
deeper in a module hierarchy, starting from the root every time would make your
code lengthy. In those cases, using `super` to get from the current module to
sibling modules is a good shortcut. Plus, if you’ve specified the path from the
root in many places in your code and then you rearrange your modules by moving
a subtree to another place, you’d end up needing to update the path in several
places, which would be tedious.

It would also be annoying to have to type `super::` in each test, but you’ve
already seen the tool for that solution: `use`! The `super::` functionality
changes the path you give to `use` so it is relative to the parent module
instead of to the root module.

For these reasons, in the `tests` module especially, `use super::something` is
usually the best solution. So now our test looks like this:

<span class="filename">Filename: src/lib.rs</span>

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

When we run `cargo test` again, the test will pass and the first part of the
test result output will be the following:

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
     Running target/debug/communicator-92007ddb5330fa5a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

## Summary

Now you know some new techniques for organizing your code! Use these techniques
to group related functionality together, keep files from becoming too long, and
present a tidy public API to your library users.

Next, we’ll look at some collection data structures in the standard library
that you can use in your nice, neat code!

