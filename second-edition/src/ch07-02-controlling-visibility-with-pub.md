## Gérer la visibilité avec `pub`

Nous avons résolu les messages d'erreur de l'entrée 7-5 en déplaçant le code de
`network` et de `network::server`, respectivement dans les fichiers
*src/network/mod.rs* et *src/network/server.rs*. A partir de la, nous avons pu
compiler notre projet avec `cargo build`, mais nous avions toujours des
messages d'avertissement à propos des fonctions `client::connect`,
`network::connect` et `network::server::connect` qui n'étaient pas utilisées :

```text
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

Donc, pourquoi avons-nous ces avertissements ? Après tout, nous contruisons une
bibliothèque avec des fonctions qui devrons être utilisées par ses
*utilisateurs*, et pas forcément par nous dans notre propre projet, donc ce
n'est pas grave si ces fonctions `connect` ne sont pas utilisées. Leur raison
d'être est qu'elles vont être utilisées par d'autres projets, pas par le nôtre.

Pour comprendre pourquoi ce programme lance ces avertissements, essayons
d'utiliser la bibliothèque `connect` à partir d'un autre projet, appellons-la à
partir de l'extérieur. Pour faire ceci, nous allons créer un crate pour binaire
dans le même dossier que notre crate de bibliothèque en créant un fichier
*src/main.rs* qui contient le code suivant :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

Nous utilisons la commande `extern crate` pour apporter le crate de
bibliothèque dans notre portée. Notre package contient maintenant *deux*
crates. Cargo considère que le fichier *src/main.rs* est le fichier racine du
crate de binaire, qui est séparé du crate de bibliothèque existant dont le
fichier racine est *src/lib.rs*. Cette organisation est courrant pour des
projets exécutables : la pluspart des fonctionnalités sont dans le crate de
bibliothèque, et le crate de binaire utilise ce crate de bibliothèque. Au
final, les autres programmes peuvent aussi utiliser le crate de bibliothèque,
et c'est une bonne séparations des tâches.

Du point de vue d'un crate à l'extérieur de la bibliothèque `communicator`,
tous les modules que nous avons créé sont à l'intérieur d'un module qui a le
même nom que le crate, `communicator`. Nous allons appeller *module racine*, le
niveau le plus haut du module du crate.

Notez toutefois que même si nous utilisons un crate externe à l'intérieur d'un
sous-module de notre projet, le `extern crate` devrait se rendre dans notre
module racine (donc dans *src/main.rs* ou *src/lib.rs*). Ensuite, dans nos
sous-modules, nous pouvons nous référer à des éléments du crate externe comme
si ces éléments étaient du module du niveau le plus haut.

Maintenant, notre crate de binaire va simplement appeller la fonction `connect`
de notre bibliothèque à partir du module `client`. Cependant, utiliser
`cargo build` va maintenant nous faire une erreur après les avertissements :

```text
error[E0603]: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Cette erreur nous avertis que le module `client` est privé, ce qui est la
raison des avertissements. C'est aussi la première fois que nous rencontrons
les concepts de *public* et *privé* avec Rust. Par défaut, tout le code dans
Rust est privé : personne d'autre n'est autorisé à utiliser ce code. Si vous
n'utilisez pas une fonction privée dans votre programme, et comme votre
code est le seul autorisé à utiliser cette fonction, Rust va vous avertir que la
fonction n'est pas utilisée.

Après avoir précisé qu'une fonction comme `client::connect` est publique, non
seulement l'appel de cette fonction dans notre crate de binaire va être
autorisé, mais l'avertissement qui informait que la fonction n'était pas
utilisée va disparaître. Rendre publique un fonction fait comprendre à Rust
que cette fonction va être utilisé par du code en dehors de notre programme.
Rust considère que l'utilisation externe est maintenant théoriquement possible
comme si elle était “en cours d'utilisation”. De plus, quand une fonction est
marquée comme publique, Rust ne va pas nécessiter qu'elle soit effectivement
utilisée dans notre programme et va arrêter d'avertir que la fonction n'est
pas utilisée.

### Rendre une fonction publique

Pour expliquer à Rust de rendre une fonction publique, nous ajoutons le mot-clé
`pub` au début de sa déclaration. Nous allons résoudre les avertissements qui
informent que `client::connect` n'a pas encore été utilisé, ainsi que l'erreur
`` module `client` is private`` dans notre crate de binaire. Modifions ainsi
*src/lib.rs* pour rendre le module `client` publique :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust,ignore
pub mod client;

mod network;
```

Le mot-clé `pub` est placé just avant `mod`. Essayons de compiler à nouveau :

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Youpi ! Nous avons une erreur différente ! Oui, les messages d'erreur
différents sont une bonne raison de se réjouir. La nouvelle erreur informe que
`` function `connect` is private ``, donc modifions *src/client.rs* pour
rendre aussi `client::connect` publique :

<span class="filename">Nom du fichier : src/client.rs</span>

```rust
pub fn connect() {
}
```

Maintenant, lançons `cargo build` à nouveau :

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

The code compiled, and the warning about `client::connect` not being used is
gone!

Unused code warnings don’t always indicate that an item in your code needs to
be made public: if you *didn’t* want these functions to be part of your public
API, unused code warnings could be alerting you to code you no longer need that
you can safely delete. They could also be alerting you to a bug if you had just
accidentally removed all places within your library where this function is
called.

But in this case, we *do* want the other two functions to be part of our
crate’s public API, so let’s mark them as `pub` as well to get rid of the
remaining warnings. Modify *src/network/mod.rs* to look like the following:

<span class="filename">Filename: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Then compile the code:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / pub fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

Hmmm, we’re still getting an unused function warning, even though
`network::connect` is set to `pub`. The reason is that the function is public
within the module, but the `network` module that the function resides in is not
public. We’re working from the interior of the library out this time, whereas
with `client::connect` we worked from the outside in. We need to change
*src/lib.rs* to make `network` public too, like so:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Now when we compile, that warning is gone:

```text
warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default
```

Only one warning is left! Try to fix this one on your own!

### Privacy Rules

Overall, these are the rules for item visibility:

1. If an item is public, it can be accessed through any of its parent modules.
2. If an item is private, it can be accessed only by its immediate parent
   module and any of the parent’s child modules.

### Privacy Examples

Let’s look at a few more privacy examples to get some practice. Create a new
library project and enter the code in Listing 7-6 into your new project’s
*src/lib.rs*:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    outermost::middle_function();
    outermost::middle_secret_function();
    outermost::inside::inner_function();
    outermost::inside::secret_function();
}
```

<span class="caption">Listing 7-6: Examples of private and public functions,
some of which are incorrect</span>

Before you try to compile this code, make a guess about which lines in the
`try_me` function will have errors. Then, try compiling the code to see whether
you were right, and read on for the discussion of the errors!

#### Looking at the Errors

The `try_me` function is in the root module of our project. The module named
`outermost` is private, but the second privacy rule states that the `try_me`
function is allowed to access the `outermost` module because `outermost` is in
the current (root) module, as is `try_me`.

The call to `outermost::middle_function` will work because `middle_function` is
public, and `try_me` is accessing `middle_function` through its parent module
`outermost`. We determined in the previous paragraph that this module is
accessible.

The call to `outermost::middle_secret_function` will cause a compilation error.
`middle_secret_function` is private, so the second rule applies. The root
module is neither the current module of `middle_secret_function` (`outermost`
is), nor is it a child module of the current module of `middle_secret_function`.

The module named `inside` is private and has no child modules, so it can only
be accessed by its current module `outermost`. That means the `try_me` function
is not allowed to call `outermost::inside::inner_function` or
`outermost::inside::secret_function`.

#### Fixing the Errors

Here are some suggestions for changing the code in an attempt to fix the
errors. Before you try each one, make a guess as to whether it will fix the
errors, and then compile the code to see whether or not you’re right, using the
privacy rules to understand why.

* What if the `inside` module was public?
* What if `outermost` was public and `inside` was private?
* What if, in the body of `inner_function`, you called
  `::outermost::middle_secret_function()`? (The two colons at the beginning mean
  that we want to refer to the modules starting from the root module.)

Feel free to design more experiments and try them out!

Next, let’s talk about bringing items into scope with the `use` keyword.
