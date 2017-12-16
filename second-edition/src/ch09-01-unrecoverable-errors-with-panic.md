## Erreurs Irrécupérables avec `panic!`

Parfois, les choses se passent mal dans votre code, et vous ne pouvez rien y
faire. Dans ce cas, Rust a la macro `panic!`. Quand la macro `panic!`
s'exécute, votre programme va afficher un message d'erreur, dévide et nettoie
la stack, et ensuite ferme le programme. Le cas le plus courant au cours
duquel cela se produit est quand une bogue a été détecté, et que ce n'est pas
clair pour le développeur de savoir comment gérer cette erreur.

> ### Dévider la stack ou abandon en réponse à un `panic!`
> Par défaut, quand un `panic!` se produit, le programme commence par
> *dévider*, ce qui veut dire que Rust retourne en arrière dans la stack et
> nettoie les données de chaque fonction qu'il trouve. Mais cette marche en
> arrière et le nettoyage demande beaucoup de ressources. L'alternative est
> d'abandonner *immédiatement* l'exécution, qui arrête le programme sans
> nettoyage. La mémoire qu'utilisait le programme va ensuite devoir être
> nettoyée par le système d'exploitation. Si dans votre projet vous avez besoin
> de construire un exécutable le plus petit possible, vous pouvez passer du
> dévidage à l'abandon lors d'un panic en ajoutant `panic = 'abort'` aux
> sections `[profile]` appropriées dans votre fichier *Cargo.toml*. Par
> exemple, si vous voulez abandonner lors d'un panic en mode relase, ajoutez
> ceci :
> 
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Essayons d'appeller `panic!` dans un programme simple :

<span class="filename">Fichier: src/main.rs</span>

```rust,should_panic
fn main() {
    panic!("crash and burn");
}
```

Quand vous lancez le programme, vous allez voir quelquechose qui ressemble à
cela :

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25 secs
     Running `target/debug/panic`
thread 'main' panicked at 'crash and burn', src/main.rs:2
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/panic` (exit code: 101)
```

L'utilisation de `panic!` déclenche le message d'erreur présent dans les trois
dernières lignes. La première ligne affiche le message associé au panic et
l'emplacement dans notre code source où se produit le panic : *src/main.rs:2*
indique que c'est la ligne 2 de notre fichier *src/main.rs*.

Dans cet exemple, la ligne indiquée fait partie de notre code, et si nous
allons voir cette ligne, nous verrons l'appel à la macro `panic!`. Dans
d'autres cas, l'appel de `panic!` pourrait se produire dans du code que nôtre
code utilise. Le nom du fichier et la ligne indiquée par le message d'erreur
sera alors du code de quelqu'un d'autre où la macro `panic!` est appellée, pas
la ligne de notre code qui pourrait mener à cet appel de `panic!`. Nous pouvons
utiliser la backtrace des fonctions qui appellent le `panic!` pour comprendre
la partie de notre code qui pose problème. Nous alons maintenant parler plus en
détail de ce qu'est une backtrace.

### Utiliser la Batrace de `panic!`

Analysons un autre exemple pour voir ce qui se passe lors d'un appel de
`panic!` se produit dans un librairie à cause d'un bug dans notre code plutôt
que d'appeller la macro directent. L'entrée 9-1 montre du code qui essaye
d'accéder aux éléments d'un vector via leurs index :

<span class="filename">Fichier: src/main.rs</span>

```rust,should_panic
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

<span class="caption">Entrée 9-1: Tentative d'accéder à un élément en dehors de
la fin d'un vector, ce qui provoque un `panic!`</span>

Ici, nous essayons d'accéder au centième élément (centième car l'indexation
commence à zero) de notre vector, mais il a seulement trois élements. Dans ce
cas, Rust va faire un panic. Utiliser `[]` est censé retourner un élement, mais
si vous lui founissez un index invalide, Rust ne pourra pas retourner un
élement acceptable.

Dans ce cas, d'autre languages, comme le C, vont tenter de vous donner
exactement ce que vous avez demandé, même si ce n'est pas ce que vous voulez :
vous allez récupérer quelquechose à l'emplacement mémoire demandé qui
correspond à l'élément demande démandé dans le vector, même si cette partie de
la mémoire n'appartient pas au vector. C'est ce qu'on appelle un
*buffer overread* et peut menner à une faille de sécurité si un attaquant a la
possibilité de piloter l'index de telle manière qu'il puisse lire les données
qui ne sont qui ne devraient pas être lisibles en dehors du array.

Afin de protéger votre programme de ce genre de vulnérabilité, si vous essayez
de lire un élement à un index qui n'existe pas, Rust va arrêter l'exécution et
refuser de continer. Essayez et vous verrez :

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is
100', /stable-dist-rustc/build/src/libcollections/vec.rs:1362
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/panic` (exit code: 101)
```

Cette erreur se réfère à un fichier que nous n'avons pas codé,
*libcollections/vec.rs*. C'est une implémentation de `Vec<T>` dans la librairie
standard. Le code qui est lancé quand nous utilisons `[]` sur notre vecteur `v`
est dans *libcollections/vec.rs*, et c'est ici que le `panic!` se produit.

La ligne suivante nous informe que nous pouvons régler la variable
d'environnement `RUST_BACKTRACE` pour obtenir la backtrace de ce qui s'est
exactement passé pour mener à cette erreur. Une *backtrace* est la liste de
toutes les fonctions qui ont été appellées pour arriver jusqur'à ce point. Dans
Rust, la backtrace fonctionne comme elle le fait dans d'autres languages : le
secret pour lire la bracktrace est de commencer d'en haut et lire jusqu'a ce
que vous voyez des fichiers que vous avez écrit. C'est l'emplacement où s'est
produit le problème. Les lignes avant celle qui mentionne vos fichier
représentent le code qu'à appellé votre code; les lignes qui suivent
représentent le code qui a appellé votre code. Ces lignes peuvent être du code
de Rust, du code de la librairie standard, ou des crates que vous utilisez.
Essayons une backtrace: l'entrée 9-2 contient un retour programme similaire à
celui que vous avez provoqué :

```text
$ RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 100', /stable-dist-rustc/build/src/libcollections/vec.rs:1392
stack backtrace:
   1:     0x560ed90ec04c - std::sys::imp::backtrace::tracing::imp::write::hf33ae72d0baa11ed
                        at /stable-dist-rustc/build/src/libstd/sys/unix/backtrace/tracing/gcc_s.rs:42
   2:     0x560ed90ee03e - std::panicking::default_hook::{{closure}}::h59672b733cc6a455
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:351
   3:     0x560ed90edc44 - std::panicking::default_hook::h1670459d2f3f8843
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:367
   4:     0x560ed90ee41b - std::panicking::rust_panic_with_hook::hcf0ddb069e7abcd7
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:555
   5:     0x560ed90ee2b4 - std::panicking::begin_panic::hd6eb68e27bdf6140
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:517
   6:     0x560ed90ee1d9 - std::panicking::begin_panic_fmt::abcd5965948b877f8
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:501
   7:     0x560ed90ee167 - rust_begin_unwind
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:477
   8:     0x560ed911401d - core::panicking::panic_fmt::hc0f6d7b2c300cdd9
                        at /stable-dist-rustc/build/src/libcore/panicking.rs:69
   9:     0x560ed9113fc8 - core::panicking::panic_bounds_check::h02a4af86d01b3e96
                        at /stable-dist-rustc/build/src/libcore/panicking.rs:56
  10:     0x560ed90e71c5 - <collections::vec::Vec<T> as core::ops::Index<usize>>::index::h98abcd4e2a74c41
                        at /stable-dist-rustc/build/src/libcollections/vec.rs:1392
  11:     0x560ed90e727a - panic::main::h5d6b77c20526bc35
                        at /home/you/projects/panic/src/main.rs:4
  12:     0x560ed90f5d6a - __rust_maybe_catch_panic
                        at /stable-dist-rustc/build/src/libpanic_unwind/lib.rs:98
  13:     0x560ed90ee926 - std::rt::lang_start::hd7c880a37a646e81
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:436
                        at /stable-dist-rustc/build/src/libstd/panic.rs:361
                        at /stable-dist-rustc/build/src/libstd/rt.rs:57
  14:     0x560ed90e7302 - main
  15:     0x7f0d53f16400 - __libc_start_main
  16:     0x560ed90e6659 - _start
  17:                0x0 - <unknown>
```

<span class="caption">Entrée 9-2: La backtrace générée par l'appel de `panic!`
est affichée quand la variable d'environnement `RUST_BACKTRACE` est définie
</span>

Cela fait beaucoup de texte ! L'exactitude de votre retour qui est affiché peut
être différent en fonction de votre système d'exploitation et votre version de
Rust. Pour avoir la backtrace avec ces informations, les instructions de
débogage doivent être activés. Ils sont activés par défaut quand on utilise
`cargo build` ou `cargo run` sans le flag --release, comme c'est le cas ici.

Sur la sortie dans l'entrée 9-2, la ligne 11 de la backtrace nous montre la
ligne de notre projet qui provoque le problème : *src/main.rs* à la ligne 4. Si
nous ne voulons pas que notre programme fasse un panic, l'emplacement cité par
la première ligne qui mentionne le code que nous avons écrit est le premier
endroit où nous devrions investiguer pour trouver les valeurs qui ont provoqué
ce panic. Dans l'entrée 9-1 où nous avons délibérément écrit du code qui fait
un panic dans le but de montrer comment nous utilisons les backtrace, la
solution pour ne pas provoquer de panic est de ne pas demander l'élement à
l'index 100 d'un vecteur quand il en contient seulement trois. A l'avenir quand
votre code provoquera des panic, vous aurez besoin de prendre des dispositions
dans votre code avec les valeurs qui provoquent un panic et de coder quoi faire
à la place.

Nous reviendrons sur le cas du `panic!` et sur les cas où nous devrions et ne
devrions pas utiliser `panic!` pour gérer les conditions d'erreur plus tard
dans ce chapitre. Maintenant, nous allons voir comment gérer une erreur qui
utilise `Result`.
