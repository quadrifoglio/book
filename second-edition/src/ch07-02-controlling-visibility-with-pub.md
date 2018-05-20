## Gérer la visibilité avec `pub`

Nous avons résolu les messages d'erreur de l'entrée 7-5 en déplaçant le code de
`network` et de `network::server`, respectivement dans les fichiers
*src/network/mod.rs* et *src/network/server.rs*. A partir de là, nous avons pu
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

Donc, pourquoi avons-nous ces avertissements ? Après tout, nous construisons
une bibliothèque avec des fonctions qui devront être utilisées par ses
*utilisateurs*, et pas forcément par nous dans notre propre projet, donc ce
n'est pas grave si ces fonctions `connect` ne sont pas utilisées. Leur raison
d'être est qu'elles vont être utilisées par d'autres projets, pas par le nôtre.

Pour comprendre pourquoi ce programme lance ces avertissements, essayons
d'utiliser la bibliothèque `connect` à partir d'un autre projet, appelons-la à
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
fichier racine est *src/lib.rs*. Cette organisation est courante pour des
projets exécutables : la plupart des fonctionnalités sont dans le crate de
bibliothèque, et le crate de binaire utilise ce crate de bibliothèque. Au
final, les autres programmes peuvent aussi utiliser le crate de bibliothèque,
et c'est une bonne séparation des tâches.

Du point de vue d'un crate à l'extérieur de la bibliothèque `communicator`,
tous les modules que nous avons créés sont à l'intérieur d'un module qui a le
même nom que le crate, `communicator`. Nous allons appeler *module racine*, le
niveau le plus haut du module du crate.

Notez toutefois que même si nous utilisons un crate externe à l'intérieur d'un
sous-module de notre projet, le `extern crate` devrait se rendre dans notre
module racine (donc dans *src/main.rs* ou *src/lib.rs*). Ensuite, dans nos
sous-modules, nous pouvons nous référer à des éléments du crate externe comme
si ces éléments étaient du module du niveau le plus haut.

Maintenant, notre crate de binaire va simplement appeler la fonction `connect`
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
code est le seul autorisé à utiliser cette fonction, Rust va vous avertir que
la fonction n'est pas utilisée.

Après avoir précisé qu'une fonction comme `client::connect` est publique, non
seulement l'appel de cette fonction dans notre crate de binaire va être
autorisé, mais l'avertissement qui informait que la fonction n'était pas
utilisée va disparaître. Rendre publique une fonction fait comprendre à Rust
que cette fonction va être utilisée par du code en dehors de notre programme.
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
*src/lib.rs* pour rendre public le module `client` :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust,ignore
pub mod client;

mod network;
```

Le mot-clé `pub` est placé juste avant `mod`. Essayons de compiler à nouveau :

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Youpi ! Nous avons une erreur différente ! Oui, les messages d'erreurs
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

Le code s'est compilé, et l'avertissement à propos de `client::connect` qui
n'est pas utilisé a disparu !

Les avertissements à propos du code non utilisé ne signifient pas
systématiquement que votre code doit être passé en publique : si vous
*ne voulez pas* que ces fonctions fassent partie de votre API publique, les
avertissements de code non utilisé peuvent être le signal que ce code est
devenu inutile et que vous pouvez l'enlever sans risques. Ils peuvent aussi
vous avertir d'un bogue potentiel si vous avez accidentellement enlevé cet
appel de tous les endroits de votre bibliothèque.

Mais dans notre cas, nous *voulons* que les deux autres fonctions fassent
partie de l'API publique de notre crate, donc marquons-les elles aussi avec
`pub` afin d'enlever les derniers avertissements. Modifions
*src/network/mod.rs* pour qu'il ressemble à ceci :

<span class="filename">Nom du fichier : src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Nous compilons ensuite le code :

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

Hmmm, nous avons toujours un avertissement pour une fonction non utilisée, même
si `network::connect`est marqué avec `pub`. La raison à cela est que cette
fonction est publique dans le module, mais le module `network` qui contient
cette fonction n'est pas publique. Cette fois-ci, nous opérons depuis
l'intérieur de la bibliothèque, tandis qu'avec `client::connect` nous avions
opéré de l'extérieur vers l'intérieur. Il nous faut changer *src/lib.rs* pour
rendre aussi `network` publique, comme ceci :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Maintenons quand nous compilons, l'avertissement disparaît :

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

Il ne reste plus qu'un avertissement ! Entraînez-vous en réglant cela par
vous-même !

### Règles du mode privé

Dans l'ensemble, voici les règles pour la visibilité des éléments :

1. Si un élément est public, il peut être accessible via n'importe quel module
   parent.
2. Si un élément est privé, il n'est accessible que part par son module
   parent direct et tous les modules enfants de son parent.

### Exemples du mode privé

Analysons quelques exemples supplémentaires du mode privé pour pratiquer un
peu. Créez un nouveau projet de bibliothèque et saisissez le code dans l'entrée
7-6 dans le fichier *src/lib.rs* de votre nouveau projet :

<span class="filename">Nom du fichier : src/lib.rs</span>

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

<span class="caption">Entrée 7-6 : Exemples de fonctions privées et publiques,
quelques-unes d'entre elles ne sont pas valides</span>

Avant que vous essayez de compiler ce code, essayez de deviner quelles lignes
dans la fonction `try_me` vont générer des erreurs. Ensuite, compilez le code
pour voir celles pour lesquelles vous aviez raison, et lisez les explications
sur les erreurs !

#### Rechercher les erreurs

La fonction `try_me` est dans le module racine de notre projet. Le module
`outermost` est privé, mais la seconde règle du mode privé implique que la
fonction `try_me` est autorisée à accéder au module `outermost`, car
`outermost` est dans le même module (la racine) que `try_me`.

L'appel à `outermost::middle_function` va fonctionner, car `middle_function`
est publique, et `try_me` va utiliser `middle_function` par le biais de son
module parent `outermost`. Nous avons établi dans le paragraphe précédent que
ce module était accessible.

L'appel à la fonction `outermost::middle_secret_function` va provoquer une
erreur de compilation. `middle_secret_function` est privé, donc la seconde
règle s'applique. Le module racine n'est pas le module de
`middle_secret_function` (c'est `outermost` son module), et ce n'est pas non
plus un module enfant du module `middle_secret_function`.

Le module `inside` est privé et n'a pas de module enfant, donc il peut
seulement être utilisé par son module `outermost`. Cela veut dire que la
fonction `try_me` n'est pas autorisé à appeler
`outermost::inside::inner_function` ou `outermost::inside::secret_function`.

#### Résoudre les erreurs

Voici quelques suggestions de changement du code pour tenter de résoudre les
erreurs. Avant de les essayer, essayez de deviner quelle erreur chacune d'elles
résout, et compilez ensuite le code pour vérifier si vous avez raison, en
utilisant les règles du mode privé pour comprendre pourquoi.

* Que se passerait-il si le module `inside` devenait public ?
* Que se passerait-il si `outermost` devenait public et que `inside` restait
  privé ?
* Que se passerait-il si, dans le corps de `inner_function`, vous appeliez
  `::outermost::middle_secret_function()` ? (les deux double-points au début
  signifient que nous voulons faire référence aux modules à partir du module
  racine)

N'hésitez pas à concevoir d'autres expériences et à les essayer !

Dans la partie suivante, nous allons parler de la technique pour introduire des
éléments dans la portée avec le mot-clé `use`.
