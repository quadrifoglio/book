## Qu'est-ce que l'appropriation ?

La fonctionnalité centrale de Rust est *l'appropriation*. Bien que cette
fonctionnalité soit simple à expliquer, elle a de profondes conséquences sur le
reste du langage.

Tous les programmes doivent gérer la façon dont ils utilisent la mémoire
(vive, RAM) de l'ordinateur lorsqu'ils s'exécutent. Certains langages ont un
ramasse-miettes qui scrute constamment la mémoire qui n'est plus utilisée par
le programme pendant qu'il tourne; dans d'autres langages, le développeur doit
explicitement alouer et libérer la mémoire. Rust aborde une troisième
approche : la mémoire est gérée avec un système d'appropriation avec un jeu de
règles que le compilateur vérifie au moment de la compilation. Il n'y a pas
d'impact sur les performances au moment de l'exécution pour toutes les
fonctionnalités d'appropriation.

Parce que l'appropriation est un nouveau principe pour de nombreux
développeurs, cela prends un certain temps à se familliariser. La bonne
nouvelle c'est que plus vous devenez expérimenté avec Rust et ses règles
d'appropriation, plus vous pourrez développer naturellement du code sûr et
efficace. Gardez bien cela à l'esprit !

Quand vous comprennez l'appropriation, vous aurrez une base solide pour
comprendre les fonctionnalités qui font la singularité de Rust. Dans ce
chapitre, vous allez apprendre l'appropriation en travaillant sur plusieurs
exemples qui se concentrent sur un structure de données très courrante : les
chaines de caractères.

<!-- PROD: START BOX -->

> ### La Stack et la Heap
>
> Dans de nombreux langages, nous n'avons pas besoin souvent de penser à la
> Stack et la Heap. Mais dans un système de langage de programmation comme
> Rust, si une donnée est sur la Stack ou sur la Heap a plus d'effet sur 
> comment la langage se comporte et pourquoi nous devons faire certains choix.
> Nous décrirons plus loins dans ce chapitre les différences entre la Stack et
> la Heap, voici donc une brève explication en attendant.
>
> La Stack et la Head sont toutes les deux des emplacements de la mémoire qui
> sont à disposition de votre code lors de son exécution, mais elles sont
> construites de manière différentes. La Stack enregistre les valeurs dans
> l'ordre qu'elle les reçoit et enlève les valeurs dans l'autre sens. C'est
> ce que l'on appelle *dernier entré, premier sorti*. C'est comme une pile
> d'assiettes : quand vous ajoutez des nouvelles assiettes, vous les deposez
> sur le dessus de la pile, et quand vous avez besoin d'une assiette, vous en
> prenez une sur le dessus. Ajouter ou enlever des assiettes au millieu ou en
> dessous ne serait pas aussi efficace ! Ajouter une donnée est appellé
> *pousser sur la Stack* et en enlever une se dit *sortir de la Stack*.
>
> La Stack est rapide grâce à la façon dont elle accède aux données : elle ne
> vas jamais avoir besoin de chercher un emplacement pour y mettre des données
> car c'est toujours au dessus. Une autre caractéristique qui fait que la Stack
> est rapide c'est que toutes les données de la Stack doivent avoir une taille
> connue et fixe.
> 
> Si les données ont une taille qui nous est inconnue au moment de la
> compilation ou une taille qui peut changer, nous pouvons plutôt les stocker
> dans la Heap. La Heap est moins organisée : quand nous poussons de la donnée
> sur la Heap, nous demandons une certaine quantité de place. Le système
> d'exploitation trouve un emplacement vide suffisamment grand quelque part
> dans la Heap, le marque comme en cours d'utilisation, et nous retourne un
> *pointeur*, qui est l'adresse de cet emplacement. Ce processus est appellé
> *allouer de la place sur la Heap*, et parfois nous raccourcissons cette
> phrase à simplement *allouer*. Mais pousser des valeurs sur Stack n'est pas
> considéré comme allouer. Parce que le pointeur a une taille connue et fixée,
> nous pouvons stocker ce pointeur sur la Stack, mais quand nous voulons la
> donnée concernée, nous avons besoin de suivre le pointeur.
> 
> C'est comme si vous vouliez manger à un restaurant. Quand vous entrez, vous
> indiquez le nombre de personnes dans votre groupe, et le personnel trouve une
> table vide qui peut reçevoir tout le monde, et vous y conduit. Si quelqu'un
> dans votre groupe arrive en retard, il peut leur demander où vous êtes assis
> pour vous rejoindre.
> 
> Accéder à des données dans la Heap est plus lent que d'accéder aux données
> sur la Stack car nous devons suivre un pointeur pour l'obtenir. Les
> processeurs modernes sont plus rapide s'ils sautent moins dans la mémoire.
> Pour continuer avec notre analogie, immaginez une serveur dans un restaurant
> qui prends les commandes de nombreuses tables. C'est plus efficace de
> récupérer toutes les commandes à une seule table avant de passer à la table
> suivante. Prendre une commande à la table A, puis prendre une commande à la
> table B, puis ensuite une à la table A à nouveau, puis suite une à la table B
> à nouveau sera un processus bien plus lent. De la même manière, un processeur
> sera plus efficace dans sa tâche s'il travaille sur des données qui sont
> proches l'une de l'autre (comme c'est sur la Stack) plutôt que si elles sont
> plus éloignées (comme cela peut être le cas sur la Heap). Allouer une grande
> quantité d'espace sur la Heap peut aussi prendre plus de temps.
> 
> Quand notre code utilise une fonction, les valeurs envoyées à la fonction
> (incluant, potentiellement, des pointeurs vers des données sur la Heap) et
> les variables locales de la fonction sont poussées sur la Stack. Quand la
> fonction est terminée, ces données sont sorties de la Stack.
> 
> Faire attention à telles parties du code utilise telles parties de données
> sur la Heap, minimiser la quantité de données en double sur la Heap, et
> libérer les données inutilisées sur la Heap pour que nous ne soyons pas à
> court d'espace, sont touts les problèmes que règle l'appropriation. Quand
> vous aurez compris l'appropriation, vous n'aurez plus souvent besoin de
> penser à la Stack et la Heap, mais savoir que l'appropriation existe pour
> gérer les données de la Heap peut vous aider à comprendre pourquoi elle
> fonctionne de cette façon.
>
<!-- PROD: END BOX -->

### Les règles de l'appropriation

Tout d'abord, définissons les règles de l'appropriation. Gardez à l'esprit ces
règles pendant que nous travaillons sur des exemples qui les illustrent :

> 1. Chaque valeur dans Rust a une variable qui s'appelle son *propriétaire*.
> 2. Il ne peut y avoir qu'un seul propriétaire au même moment.
> 3. Quand le propriétaire sort du bloc d'instructions, la valeur est supprimée.

### Portée de la variable

Nous avons déjà vu un exemple dans le programme Rust du Chapitre 2. Maintenant
que nous avons terminé la syntaxe de base, nous n'allons pas mettre tout le
code de `fn main() {` dans des exemples, donc si vous suivez, vous devez mettre
les exemples suivants manuellement dans un fonction `main`. De fait, nos
exemples seront plus concis, nous permettant de nous concentrer sur les détails
actuels plutôt que sur du code conventionnel.

Pour le premier exemple d'appropriation, nous allons analyser la *portée* de
certaines variables. Une portée est une zone dans un programme dans lequel un
objet est en vigueur. Imaginons que nous avons une variable qui ressemble à
ceci :

```rust
let s = "hello";
```

La variable `s` fait référence à une chaine de caractères pure, où la valeur de
la chaine est codé en dur dans notre programme. La variable est en vigueur à
partir du point où elle est déclarée jusqu'à la fin de la *portée* actuelle.
L'entrée 4-1 a des commentaires pour indiquer quand la variable `s` est en
vigueur :

```rust
{                      // s n'est pas en vigueur ici, elle n'est pas encore déclarée
    let s = "hello";   // s est en vigueur à partir de ce point

    // on fait des choses avec s ici
}                      // cette portée est maintenant terminée, et s n'est plus en vigueur
```

<span class="caption">Entrée 4-1 : une variable et la portée dans laquelle elle
est en vigueur.</span>

Autrement dis, il y a ici deux étapes importantes :

1. Quand `s` rentre *dans la portée*, elle est en vigueur.
2. Cela reste ainsi jusqu'à ce qu'elle *sort de la portée*.

Pour le moment, la relation entre les portées et quand les variables sont en
vigueur sont similaires à d'autres langages de programmation. Maintenant nous
allons aller plus loins dans la compréhension en y ajoutant le type `String`.

### Le type `String`

Pour illustrer les règles de l'appartenance, nous avons besoin d'un type de
données qui est plus complexe que ceux que nous avons rencontré au chapitre 3.
Les types que nous avons vu dans la section “Types de données” sont toutes
stockées sur la Stack et sont sorties de la Stack quand elles sont sortent de
la portée, mais nous voulons examiner les données stockées sur la Heap et
regarder comment Rust sait quand il doit nettoyer ces données.

Nous allons utiliser ici `String` pour l'exemple et nous concentrer sur les
élements de `String` qui sont liés à l'appropriation. Ces caractéristiques
s'appliquent aussi pour d'autres types de données complexes fournis par la
librairie standard et ceux que vous créez. Nous verons `String` plus en détail
dans le Chapitre 8.

Nous avons déjà vu les chaines de caractères pures, quand une valeur de chaine
est codée en dur dans notre programme. Les chaines de caractères pures sont
pratiques, mais elles ne conviennent pas toujours à tous les cas où vous voulez
utiliser du texte. Une des raisons est qu'elle est immuable. Une autre est
qu'on ne connait pas forcément toutes les chaines de caractères quand nous
écrivons notre code : par exemple, comment faire si nous voulons récupérer du
texte saisi par l'utilisateur et l'enregistrer ? Pour ces cas-ci, Rust a un
second type de chaine de caractères, `String`. Ce type est alloué sur la Heap
et est ainsi capable de stocker une quantité de texte qui nous est inconnue au
moment de la compilation. Vous pouvez créer un `String` à partir d'une chaine
de caractères pure en utilisant la fonction `from`, comme ceci :

```rust
let s = String::from("hello");
```

Le deux-points double est une opérateur qui nous permet d'appeler cette
fonction spécifique dans l'espace de nom du type `String` plutôt que d'utiliser
un nom comme `string_from`. Nous allons voir cette syntaxe plus en détail dans
la section “Syntaxe de méthode” du Chapitre 5 et lorsque nous allons aborder
l'espace de nom avec les modules au Chapitre 7.

Ce type de chaine de caractères *peut* être mutable :

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() ajoute du texte dans un String

println!("{}", s); // Cela va afficher `hello, world!`
```

Donc, quelle est la différence ici ? Pourquoi `String` peut être mutable mais
que les chaines pures ne peuvent pas l'être ? La différence est comment ces
deux type travaillent avec la mémoire.

### Mémoire et attribution

Dans le cas d'une chaine de caractères pure, nous connaissons le contennu au
moment de la compilation donc le texte est codé en dur directement dans
l'exécutable final, ce qui fait que ces chaines de caractères pures sont
performantes et rapides. Mais ces caractéristiques viennent de leur
immuabilité. Malheureusement, nous ne pouvons pas sotcker un blob de mémoire
dans le binaire pour chaque morceau de texte qui n'a pas de taille connue au
moment de la compilation et dont la taille pourrait changer pendant l'exécution
de ce programme.

Avec le type `String`, afin de permettre un texte mutable et qui peut
s'aggrandir, nous devons allouer une quantité de mémoire sur la Heap, inconnue
au moment de la compilation, pour stocker le contennu. Cela signifie que :

1. La mémoie doit être sollicitée auprès du système d'exploitation lors de
l'exécution.
2. Nous avons besoin d'un moyen de rendre cette mémoire au système
d'exploitation quand nous avons fini avec notre `String`.

Le premier point est fait par nous : quand nous appelons `String::from`, sa
définition demande la mémoire qu'elle a besoin. C'est pratiquement toujours le
cas dans les langages de programmation.

Cependant, le deuxième point est différent. Dans des langages avec un
*ramasse-miettes*, le ramasse-miettes surveille et néttoie la mémoire qui n'est
plus utilisée, sans que nous, les développeurs, nous ayons à nous en
préhocuper. Sans un ramasse-miettes, c'est de la responsabilité du développeur
d'identifier quand la mémoire n'est plus utilisée et d'appeler du code pour
explicitement la libérer, comme nous l'avons fait pour la demander auparavent.
Historiquement, faire ceci correctement a toujours été une difficulté pour
développer. Si nous oublions de le faire, nous alons gaspiller de la mémoire.
Si nous le faisons trop tôt, nous allons avoir une variable incorrecte. Si nous
le faisons deux fois, c'est aussi un bogue. Nous avons besoin d'associer
exactement un `allocate` avec exactement un `free`.

Rust prends un chemin différent : la mémoire est automatiquement libérée dès
que la variable qui la possède sort de la portée. Voici une version de notre
exemple de portée de l'entrée 4-1 qui utilise un `String` plutôt qu'une chaine
de caractères pure :

```rust
{
    let s = String::from("hello"); // s est en vigueur à partir de ce point

    // on fait des choses avec s ici
}                                  // cette portée est désormais finie, et s
                                   // n'est plus en vigueur maintenant
```

Il y a un cas naturel auquel nous devons rendre la mémoire de notre `String` au
système d'exploitation : quand `s` sort de la portée. Quand une variable sort
de la portée, Rust utilise une fonction spéciale pour nous. Cette fonction
s'appelle `drop`, et ceci que l'auteur du `String` peut le mettre dans le code
pour libérer la mémoire. Rust utilise automatiquement `drop` à l'accolade
fermante `}`.

> Note : dans du C++, cette façon de désalouer des ressources à la fin de la
> durée de vie d'un objet est parfois appelé *Resource Acquisition Is
> Initialization (RAII)*. La fonction `drop` de Rust vous sera famillière si
> vous avez déjà utilisé des fonctions de RAII. 

Cette façon de faire a un impact profond sur la façon dont le code de Rust est
écrit. Cela peut sembler simple ici, mais le comportement du code peut être
inattendu dans des situations plus compliquées quand nous voulons avoir
plusieures variables avec des données que nous avons allouées sur la Heap.
Examinons une de ces situations dès à présent.

#### Ways Variables and Data Interact: Move

Multiple variables can interact with the same data in different ways in Rust.
Let’s look at an example using an integer in Listing 4-2:

```rust
let x = 5;
let y = x;
```

<span class="caption">Listing 4-2: Assigning the integer value of variable `x`
to `y`</span>

We can probably guess what this is doing based on our experience with other
languages: “Bind the value `5` to `x`; then make a copy of the value in `x` and
bind it to `y`.” We now have two variables, `x` and `y`, and both equal `5`.
This is indeed what is happening because integers are simple values with a
known, fixed size, and these two `5` values are pushed onto the stack.

Now let’s look at the `String` version:

```rust
let s1 = String::from("hello");
let s2 = s1;
```

This looks very similar to the previous code, so we might assume that the way
it works would be the same: that is, the second line would make a copy of the
value in `s1` and bind it to `s2`. But this isn’t quite what happens.

To explain this more thoroughly, let’s look at what `String` looks like under
the covers in Figure 4-1. A `String` is made up of three parts, shown on the
left: a pointer to the memory that holds the contents of the string, a length,
and a capacity. This group of data is stored on the stack. On the right is the
memory on the heap that holds the contents.

<img alt="String in memory" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-1: Representation in memory of a `String`
holding the value `"hello"` bound to `s1`</span>

The length is how much memory, in bytes, the contents of the `String` is
currently using. The capacity is the total amount of memory, in bytes, that the
`String` has received from the operating system. The difference between length
and capacity matters, but not in this context, so for now, it’s fine to ignore
the capacity.

When we assign `s1` to `s2`, the `String` data is copied, meaning we copy the
pointer, the length, and the capacity that are on the stack. We do not copy the
data on the heap that the pointer refers to. In other words, the data
representation in memory looks like Figure 4-2.

<img alt="s1 and s2 pointing to the same value" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-2: Representation in memory of the variable `s2`
that has a copy of the pointer, length, and capacity of `s1`</span>

The representation does *not* look like Figure 4-3, which is what memory would
look like if Rust instead copied the heap data as well. If Rust did this, the
operation `s2 = s1` could potentially be very expensive in terms of runtime
performance if the data on the heap was large.

<img alt="s1 and s2 to two places" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-3: Another possibility of what `s2 = s1` might
do if Rust copied the heap data as well</span>

Earlier, we said that when a variable goes out of scope, Rust automatically
calls the `drop` function and cleans up the heap memory for that variable. But
Figure 4-2 shows both data pointers pointing to the same location. This is a
problem: when `s2` and `s1` go out of scope, they will both try to free the
same memory. This is known as a *double free* error and is one of the memory
safety bugs we mentioned previously. Freeing memory twice can lead to memory
corruption, which can potentially lead to security vulnerabilities.

To ensure memory safety, there’s one more detail to what happens in this
situation in Rust. Instead of trying to copy the allocated memory, Rust
considers `s1` to no longer be valid and therefore, Rust doesn’t need to free
anything when `s1` goes out of scope. Check out what happens when you try to
use `s1` after `s2` is created, it won’t work:

```rust,ignore
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

You’ll get an error like this because Rust prevents you from using the
invalidated reference:

```text
error[E0382]: use of moved value: `s1`
 --> src/main.rs:5:28
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`, which does
  not implement the `Copy` trait
```

If you’ve heard the terms “shallow copy” and “deep copy” while working with
other languages, the concept of copying the pointer, length, and capacity
without copying the data probably sounds like a shallow copy. But because Rust
also invalidates the first variable, instead of calling this a shallow copy,
it’s known as a *move*. Here we would read this by saying that `s1` was *moved*
into `s2`. So what actually happens is shown in Figure 4-4.

<img alt="s1 moved to s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-4: Representation in memory after `s1` has been
invalidated</span>

That solves our problem! With only `s2` valid, when it goes out of scope, it
alone will free the memory, and we’re done.

In addition, there’s a design choice that’s implied by this: Rust will never
automatically create “deep” copies of your data. Therefore, any *automatic*
copying can be assumed to be inexpensive in terms of runtime performance.

#### Ways Variables and Data Interact: Clone

If we *do* want to deeply copy the heap data of the `String`, not just the
stack data, we can use a common method called `clone`. We’ll discuss method
syntax in Chapter 5, but because methods are a common feature in many
programming languages, you’ve probably seen them before.

Here’s an example of the `clone` method in action:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

This works just fine and is how you can explicitly produce the behavior shown
in Figure 4-3, where the heap data *does* get copied.

When you see a call to `clone`, you know that some arbitrary code is being
executed and that code may be expensive. It’s a visual indicator that something
different is going on.

#### Stack-Only Data: Copy

There’s another wrinkle we haven’t talked about yet. This code using integers,
part of which was shown earlier in Listing 4-2, works and is valid:

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

But this code seems to contradict what we just learned: we don’t have a call to
`clone`, but `x` is still valid and wasn’t moved into `y`.

The reason is that types like integers that have a known size at compile time
are stored entirely on the stack, so copies of the actual values are quick to
make. That means there’s no reason we would want to prevent `x` from being
valid after we create the variable `y`. In other words, there’s no difference
between deep and shallow copying here, so calling `clone` wouldn’t do anything
differently from the usual shallow copying and we can leave it out.

Rust has a special annotation called the `Copy` trait that we can place on
types like integers that are stored on the stack (we’ll talk more about traits
in Chapter 10). If a type has the `Copy` trait, an older variable is still
usable after assignment. Rust won’t let us annotate a type with the `Copy`
trait if the type, or any of its parts, has implemented the `Drop` trait. If
the type needs something special to happen when the value goes out of scope and
we add the `Copy` annotation to that type, we’ll get a compile time error. To
learn about how to add the `Copy` annotation to your type, see Appendix C on
Derivable Traits.

So what types are `Copy`? You can check the documentation for the given type to
be sure, but as a general rule, any group of simple scalar values can be
`Copy`, and nothing that requires allocation or is some form of resource is
`Copy`. Here are some of the types that are `Copy`:

* All the integer types, like `u32`.
* The Boolean type, `bool`, with values `true` and `false`.
* The character type, `char`.
* All the floating point types, like `f64`.
* Tuples, but only if they contain types that are also `Copy`. `(i32, i32)` is
`Copy`, but `(i32, String)` is not.

### Ownership and Functions

The semantics for passing a value to a function are similar to assigning a
value to a variable. Passing a variable to a function will move or copy, just
like assignment. Listing 4-3 has an example with some annotations showing where
variables go into and out of scope:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope.

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here.

    let x = 5;                      // x comes into scope.

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it’s okay to still
                                    // use x afterward.

} // Here, x goes out of scope, then s. But since s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope.
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope.
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

<span class="caption">Listing 4-3: Functions with ownership and scope
annotated</span>

If we tried to use `s` after the call to `takes_ownership`, Rust would throw a
compile time error. These static checks protect us from mistakes. Try adding
code to `main` that uses `s` and `x` to see where you can use them and where
the ownership rules prevent you from doing so.

### Return Values and Scope

Returning values can also transfer ownership. Here’s an example with similar
annotations to those in Listing 4-3:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1.

    let s2 = String::from("hello");     // s2 comes into scope.

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3.
} // Here, s3 goes out of scope and is dropped. s2 goes out of scope but was
  // moved, so nothing happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it.

    let some_string = String::from("hello"); // some_string comes into scope.

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function.
}

// takes_and_gives_back will take a String and return one.
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope.

    a_string  // a_string is returned and moves out to the calling function.
}
```

The ownership of a variable follows the same pattern every time: assigning a
value to another variable moves it. When a variable that includes data on the
heap goes out of scope, the value will be cleaned up by `drop` unless the data
has been moved to be owned by another variable.

Taking ownership and then returning ownership with every function is a bit
tedious. What if we want to let a function use a value but not take ownership?
It’s quite annoying that anything we pass in also needs to be passed back if we
want to use it again, in addition to any data resulting from the body of the
function that we might want to return as well.

It’s possible to return multiple values using a tuple, like this:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String.

    (s, length)
}
```

But this is too much ceremony and a lot of work for a concept that should be
common. Luckily for us, Rust has a feature for this concept, and it’s called
*references*.
