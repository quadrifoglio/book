## Les références et l'emprunt

Le problème avec le code du tuple à la fin de la section précédente c'est que
nous avons besoin de retourner la `String` au code appelant pour qu'il puisse
encore utiliser la `String` après l'appel à `calculate_length`, car
l'appartenance de la `String` a été déplacée dans `calculate_length`.

Ici vous allez définir et utiliser une fonction `calculate_length` qui prend
une *référence* à un objet en paramètre plutôt que de s'approprier la valeur :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

Premièrement, remarquez que tout le code avec le tuple dans la déclaration des
variables et dans le retour de la fonction a été enlevé. Ensuite, on peut
constater que nous envoyons `&s1` dans `calculate_length`, et que dans sa
définition, nous utilisons `&String` plutôt que `String`.

Ces esperluettes (`&`) sont des *références*, et elles permettent de vous référer à
une valeur sans se l'approprier. L'illustration 4-5 nous montre cela dans un
diagramme.

<img alt="&String s pointe sur String s1" src="img/trpl04-05.svg" class="center" />

<span class="caption">Illustration 4-5 : `&String s` pointe sur `String s1`</span>

> Note : l'opposé du référencement en utilisant `&` est le *déréférencement*,
> qui est effectué par l'opérateur de déréférencement, `*`. Nous allons voir
> quelques utilisations de l'opérateur de déréférencement dans le Chapitre 8 et
> nous discuterons des détails du déréférencement dans le Chapitre 15.

Regardons ici de plus près à l'appel de fonction :

```rust
# fn calculate_length(s: &String) -> usize {
#     s.len()
# }
let s1 = String::from("hello");

let len = calculate_length(&s1);
```

La syntaxe `&s1` nous permet de créer une référence qui se *réfère* à la valeur
de `s1` mais ne se l'approprie pas. Parce qu'elle ne se l'approprie pas, la
valeur qu'elle désigne ne sera pas libérée quand la référence sortira de la
portée.

De la même manière, la signature de la fonction utilise `&` pour indiquer que
le type de paramètre `s` est une référence. Ajoutons quelques commentaires
explicatifs :

```rust
fn calculate_length(s: &String) -> usize { // s est une référence à une String
    s.len()
} // Ici, s sort de la porté. Mais comme elle ne s'approprie pas ce dont elle
  // fait référence, il ne se passe rien.
```

La portée dans laquelle la variable `s` est en vigueur est la même que toute
portée de paramètre de fonction, mais nous ne libérons pas ce sur quoi cette
référence pointe quand elle sort de la portée, car nous ne nous l'avons pas
approprié. Les fonctions qui ont des références en paramètres au lieu des
valeurs effectives veulent dire que nous n'avons pas besoin de retourner les
valeurs pour rendre leur appartenance, puisque nous n'en avons jamais pris possession.

Quand nous avons des références dans les paramètres d'une fonction, nous
appelons cela *l'emprunt*. Comme dans la vie réelle, quand un objet appartient
à quelqu'un, vous pouvez le lui emprunter. Quand vous avez fini, vous devez le lui
rendre.

Que se passe-t-il si nous essayons de modifier quelque chose que nous
empruntons ? Essayez le code dans l'entrée 4-4. Spoiler : il ne fonctionne
pas !

<span class="filename">Nom du fichier : src/main.rs</span>

```rust,ignore
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

<span class="caption">Entrée 4-4: tentative de modification d'une valeur
empruntée.</span>

Voici l'erreur :

```text
error[E0596]: cannot borrow immutable borrowed content `*some_string` as mutable
 --> error.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- use `&mut String` here to make mutable
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ cannot borrow as mutable
```

Comme les variables sont immuables par défaut, les références le sont aussi.
Nous ne sommes pas autorisés à modifier une chose quand nous avons une
référence vers elle.

### Les références mutables

Nous pouvons résoudre l'erreur dans le code de l'entrée 4-4 avec une petite
modification :

<span class="filename">Nom de fichier : src/main.rs</span>

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

Premièrement, nous devons modifier `s` pour être `mut`. Ensuite, nous devons
créer une référence modifiable avec `&mut s` et prendre une référence
modifiable avec `some_string: &mut String`.

Mais les références modifiables ont une grosse restriction : vous ne pouvez
avoir qu'une seule référence mutable à un élément de donnée particulier dans
une portée précise. Le code suivant va échouer :

<span class="filename">Nom de fichier : src/main.rs</span>

```rust,ignore
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;
```

Voici l'erreur :

```text
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> borrow_twice.rs:5:19
  |
4 |     let r1 = &mut s;
  |                   - first mutable borrow occurs here
5 |     let r2 = &mut s;
  |                   ^ second mutable borrow occurs here
6 | }
  | - first borrow ends here
```

Cette restriction autorise les mutations, mais de façon très contrôlée. C'est
ce avec quoi les nouveaux Rustacéens ont du mal, car la plupart des langages
vous permettent de modifier les données quand vous voulez. L'avantage de cette
restriction est que Rust peut empêcher la concurrence des données au moment
de la compilation.

La *concurrence de données* (ou "data concurrency") ressemble à une
concurrence critique (ou "race condition") et se produit quand ces trois facteurs se combinent :

1. Un ou plusieurs pointeurs accèdent à la même donnée au même moment.
2. Au moins un des pointeurs est utilisé pour écrire dans les données.
3. On n'utilise pas de méchanisme pour synchroniser l'accès aux données.

La concurrence de données provoque des comportements inexpliqués et il peut
alors être difficile de diagnostiquer et résoudre le problème lorsque vous
essayez de les traquer au moment de l'exécution; Rust évite que ce problème
arrive parce qu'il ne va même pas compiler le code avec de la concurrence de
données !

Comme toujours, nous pouvons utiliser des accolades pour créer une nouvelle
portée, pour nous permettre d'avoir plusieurs références modifiables, mais pas
*simultanées* :

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1 sort de la portée ici, donc nous pouvons faire une nouvelle référence
  // sans problèmes.

let r2 = &mut s;
```

Une règle similaire existe pour mélanger les références immuables et
mutables. Ce code va mener à une erreur :

```rust,ignore
let mut s = String::from("hello");

let r1 = &s; // sans problème
let r2 = &s; // sans problème
let r3 = &mut s; // GROS PROBLEME
```

Voici l'erreur :

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as
immutable
 --> borrow_thrice.rs:6:19
  |
4 |     let r1 = &s; // no problem
  |               - immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |                   ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

Ouah ! Nous ne pouvons pas *non plus* avoir une référence mutable si nous en
avons une d'immuable. Les utilisateurs d'une référence immuable ne s'attendent
pas à ce que se valeur change soudainement ! Cependant, l'utilisation de
plusieurs références immuables ne pose pas de problème, car personne de ceux
qui lisent uniquement la donnée n'a la possibilité de modifier la lecture de la
donnée par les autres.

Même si ces erreurs peuvent parfois être frustrantes, souvenez-vous que le
compilateur de Rust nous fait remarquer un futur *bug* en avance (au moment
de la compilation plutôt que lors de l'exécution) et vous montre exactement où est
le problème plutôt que vous ayez à traquer pourquoi à certains moments vos
données ne correspondent pas à ce que vous pensiez qu'elles étaient.

### Références en suspension (ou "Dangling references")

Avec les langages qui utilisent les pointeurs, c'est facile de créer par erreur
un *pointeur en suspension*, qui est un pointeur qui désigne un endroit dans la
mémoire qui a été donné par quelqu'un d'autre, en libérant une partie de la
mémoire, mais en conservant un pointeur vers cette mémoire. En revanche, en
Rust, le compilateur garantie que les références ne seront jamais des
références en suspension : si nous avons une référence vers des données, le
compilateur va s'assurer que cette donnée ne vas pas sortir de la portée avant
que la référence vers cette donnée soit sortie.

Essayons de créer une référence en suspension, que Rust va empêcher avec une
erreur au moment de la compilation :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust,ignore
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

Voici l'erreur :

```text
error[E0106]: missing lifetime specifier
 --> dangle.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is
  no value for it to be borrowed from
  = help: consider giving it a 'static lifetime
```

Ce message d'erreur parle d'une fonctionnalité que nous n'avons pas encore vu :
les *durées de vie* (ou "lifetime"). Nous allons voir plus en détail les durées de vie dans le
Chapitre 10. Mais, si vous mettez de côté les parties qui parlent de la durée
de vie, le message désigne l'élément de code qui pose problème :

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from.
```

Regardons de plus près ce qui se passe à chaque étape de notre code `dangle` :

```rust,ignore
fn dangle() -> &String { // dangle retournes une référence vers un String

    let s = String::from("hello"); // s est une nouvelle String

    &s // nous retournons une référence vers la String, s
} // Ici, s sort de la portée, et est libéré. Sa mémoire disparait. Danger !
```

Parce que `s` est créé dans `dangle`, quand le code de `dangle` est terminé,
`s` va être désalouée. Mais nous avions essayé de renvoyer une référence vers
elle. Cela veut dire que cette référence va pointer vers une `String` invalide !
Ce n'est pas bon. Rust ne nous laisse pas faire cela.

Ici la solution est de renvoyer la `String` directement :

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

Cela fonctionne sans problème. L'appartenance est déplacée, et rien n'est
désaloué.

### Les règles de référencement

Récapitulons ce que nous avons vu à propos des références :

1. Vous pouvez avoir *un* des deux cas suivants, mais pas les deux en même
temps :
  * Une référence modifiable.
  * Un nombre illimité de références immuables.
2. Les références doivent toujours être en vigueur.

A l'étape suivante, nous allons aborder un type différent de référence : les
slices.
