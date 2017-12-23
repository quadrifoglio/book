## Erreurs récupérables avec `Result`

La plupart des erreurs ne sont pas assez grave au point d'arrêter complètement
le programme. Parfois, quand une fonction échoue, c'est pour une raison que
nous pouvons facilement comprendre et réagir en conséquence. Par exemple, si
nous essayons d'ouvrir un fichier et que l'opération échoue parce que le
fichier n'existe pas, nous pourrions créer le fichier plutôt que d'arrêter le
processus.

Souvenez-vous dans le Chapitre 2 dans la section “[Gérer les potentielles
erreurs avec `Result`][handle_failure]<!-- ignore -->” quand le enum `Rust` est
défini selon deux variantes, `Ok` et `Err`, comme ci-dessous :

[handle_failure]: ch02-00-guessing-game-tutorial.html#gérer-les-potentielles-erreurs-avec-result

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Le `T` et `E` sont des paramètres de type génériques : nous alons parler plus
en détail des génériques au Chapitre 10. Ce que vous avez besoin de savoir pour
le moment c'est que `T` représente le type de valeur nichée dans la variante
`Ok` qui sera retournée dans le cas d'un succès, et `E` représente le type
d'erreur nichée dans la variante `Err` qui sera retournée dans le cas d'un
échec. Parce que `Result` a ces types de paramètres génériques, nous pouvons
utiliser le type `Result` et les fonctions définies dans la librairie standard
qui l'utilisent dans différentes situations où les valeurs en cas de succès et
les valeurs en cas d'erreur que nous attendons en retour peuvent différer.

Utilisons une fonction qui retourne une valeur de type `Result` car la fonction
peut échouer : dans l'entrée 9-3 nous essayons d'ouvrir un Nom du fichier :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

<span class="caption">Entrée 9-3 : Ouvrir un fichier</span>

Comment nous savons que `File::open` retourne un `Result` ? Nous pouvons
regarder la documentation de l'API et de la librairie standard, ou nous pouvons
demander au compilateur ! Si nous affectons un type à `f` dont nous savons que
le type de retour de la fonction n'est *pas* correcte et puis que nous essayons
de compiler le code, le compilateur va nous dire que les types ne coïncident
pas. Le message d'erreur va ensuite nous dire de quel type `f` *est*. Essayons
cela : nous savons que le retour de `File::open` n'est pas du type `u32`, alors
essayons de changer l'instruction `let f` comme ceci :

```rust,ignore
let f: u32 = File::open("hello.txt");
```

La compilation nous donne maintenant le résultat suivant :

```text
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
  = note:    found type `std::result::Result<std::fs::File, std::io::Error>`
```

Cela nous dit que type de retour de la fonction `File::open` est un
`Result<T, E>`. Le paramètre générique `T` a été remplacé dans ce cas par le
type en cas de succès, `std::fs::File`, qui est un manipulateur de fichier. Le
type de `E` utilisé pour la valeur d'erreur est `std::io::Error`.

Ce type de retour veut dire que l'appel à `File::open` peut réussir et nous
retourner un manipulateur de fichier qui peut le lire ou l'écrire.
L'utilisation de cette fonction peut aussi échouer : par exemple, le fichier
peut ne pas exister ou nous n'avons pas le droit d'accéder au fichier. La
fonction `File::open` doit avoir un moyen de nous dire si son utilisation a
réussi ou échoué et en même temps nous fournir soit le manipulateur de fichier
soit des informations sur l'erreur. C'est exactement ces informations que le
enum `Result` nous transmet.

Dans le cas où `File::open` réussit, la valeur que nous aurons dans la variable
`f` sera une instance de `Ok` qui contiendra un manipulateur de fichier. Dans
le cas où cela échoue, la valeur dans `f` sera une instance de `Err` qui
contiendra plus d'information sur le type d'erreur qui a eu lieu.

Nous avons besoin d'ajouter différentes actions dans le code de l'entrée 9-3 en
fonction de la valeur que `File::open` a retourné. L'entrée 9-4 montre une
façon de gérer `Result` en utilisant un outil basique : l'expression `match`
que nous avons abordé au Chapitre 6.

<span class="filename">Nom du fichier : src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

<span class="caption">Entrée 9-4: Utilisation de l'expression `match` pour
gérer les variantes que `Result` pourrait retourner.</span>

Veuillez noter que, comme l'enum `Option`, l'enum `Result` et ses variantes ont
été importés dans le prelude, donc vous n'avez pas besoin de préciser
`Result::` avant les variantes `Ok` et `Err` dans le bloc du `match`.

Ici nous indiquons à Rust que quand le resultat est `Ok`, il faut sortir la
valeur `file` de la variante `Ok`, et nous assignons ensuite cette valeur à la
variable `f`. Après le `match`, nous pourrons ensuite utiliser le manipulateur
de fichier pour lire ou écrire.

L'autre partie du bloc `match` gère le cas où nous obtenons un `Err` de
l'appel à `File::open`. Dans cet exemple, nous avons choisi d'utiliser la macro
`panic!`. S'il n'y a pas de fichier qui s'appelle *hello.txt* dans notre
répertoire actuel et que nous exécutons ce code, nous allons voir le texte
suivant suite à l'appel de la macro `panic!` :

```text
thread 'main' panicked at 'There was a problem opening the file: Error { repr:
Os { code: 2, message: "No such file or directory" } }', src/main.rs:9:12
```

Comme d'habitude, ce texte nous dit avec précision ce qui s'est mal passé.

### Tester les différentes erreurs

Le code dans l'entrée 9-4 va faire un `panic!`, peut importe la raison lorsque
`File::open` échoue. Ce que nous voudrions plutôt faire est de régir
différemment en fonction des cas d'erreurs : si `File::open` a échoué parce que
n'existe pas, nous voulons créer le fichier et renvoyer un manipulateur de
fichier pour ce nouveau fichier. Si `File::open` échoue pour toute autre
raison, par exemple si nous n'avons pas l'autorisation d'ouvrir le fichier,
nous voulons quand même que le code fasse un `panic!` de la même manière qu'il
l'a fait dans l'entrée 9-4. Dans l'entrée 9-5, nous avons ajouté un nouveau cas
au bloc `match` :

<span class="filename">Nom du fichier : src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tried to create file but there was a problem: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "There was a problem opening the file: {:?}",
                error
            )
        },
    };
}
```

<span class="caption">Entrée 9-5 : Gestion des différents cas d'erreurs de
avec des actions différentes.</span>

La valeur de retour de `File::open` nichée dans la variante `Err` est de type
`io::Error`, ce qui est un struct fourni par la librairie standard. Ce struct
a une méthode `kind` que nous pouvons utiliser pour obtenir un retour de type
`io::ErrorKind`.
`io::ErrorKind` est un enum fourni lui aussi par la librairie standard qui a
des variantes qui représentent différents types d'erreurs qui pourraient
résulter d'une opération dans le module `io`. La variante que nous voulons
utiliser est `ErrorKind::NotFound`, qui nous informe que le fichier que nous
essayons d'ouvrir n'existe pas encore.

La condition `if error.kind() == ErrorKind::NotFound` est ce qu'ont appelle un
*match guard* : c'est une condition supplémentaire sur une branche d'un bloc
`match` qui raffine le pattern d'une branche. Cette condition doit être valide
pour que le code de cette branche soit exécuté; autrement, le pattern matching
s'orientera sur la branche suivante dans le `match`. ** (TODO) The `ref` in the
pattern is needed so `error` is not moved into the guard condition but is
merely referenced by it.** La raison pour la quelle `ref` est utilisé pour
stocker une référence dans le pattern plutôt que un `&` va être expliquée en
détails dans le Chapitre 18. Pour faire court, dans le cas d'un pattern, `&`
est associé à une référence et nous retourne sa valeur, mais `ref` associe une
valeur et nous donne une référence vers elle.

Le cas que nous voulons vérifier dans le match guard est lorsque la valeur
retournée par `error.kind()` est la variante de `NotFound` de l'enum
`ErrorKind`. Si c'est le cas, nous essayons de créer le fichier avec
`File::create`. Cependant, parce que `File::create` peut aussi échouer, nous
avons besoin d'ajouter a nouveau un `match` à l'intérieur du bloc. Quand le
fichier ne peut pas être ouvert, un message d'erreur différent sera affiché. La
dernière branche du `match` principal reste identique donc le programme fait un
panic sur toute autre erreur que celle du fichier inexistant.

### Raccourci pour faire un Panic sur une erreur : `unwrap` et `expect`

L'utilisation de `match` fonctionne assez bien, mais il peut être un peux
verbeux et ne communique pas forcément comme il le faut. Le type `Result<T, R>`
a de nombreuses méthodes pour nous aider qui lui on été définies pour faire
plusieurs choses. Une de ces méthodes, qu'on appelle `unwrap`, a été implémenté
comme le `match` que nous avons écris dans l'entrée 9-4 : si la valeur de
`Result` est une variante de `Ok`, `unwrap` va retourner la valeur dans le
`Ok`, et si le `Result` est une variante de `Err`, `unwrap` va appeller la
macro `panic!` pour nous. Voici un example de `unwrap` à l'action :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

Si nous exécutons ce code sans le fichier *hello.txt*, nous allons voir un
message d'erreur d'un appel à `panic!` que la méthode `unwrap` a fait :

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

L'autre méthode, `expect`, qui est similaire à `unwrap`, nous donne la
possibilité de choisir le message d'erreur du `panic!`. Utiliser `expect`
plutôt que `unwrap` et lui fournir des bons messages d'erreurs permet de mieux
exprimer le problème et faciliter la recherche de la source d'erreur. La
syntaxe de `expect` est la suivante :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

Nous utilisons `expect` de la même manière que `unwrap` : pour retourner le
manipulateur de fichier ou appeller la macro `panic!`. Le message d'erreur
utilisé par `expect` lors de son appel au `panic!` sera le paramètre que nous
avons donnée à `expect`, plutôt que le message par défaut de `panic!`
qu'utilise `unwrap`. Voici ce que cela donne :

```text
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

Parce que ce message d'erreur commence par le texte que nous avons précisé,
`Failed to open hello.txt`, ce sera plus facile de trouver d'où dans le code
ce message d'erreur proviens. Si nous utilisons `unwrap` dans plusieurs
endroits, cela peut prendre plus de temps de comprendre exactement quel
`unwrap` a déclanché le panic car tous les appels au `unwrap` vont afficher le
même message.

### Propager les Erreurs

Quand vous écrivez une fonction dont son implémentation utilise quelque chose
qui pourrait échouer, plutôt que de gérer l'erreur dans cette fonction, vous
pouvez retourner cette erreur au code qui l'appelle pour qu'il décide quoi
faire. C'est ce qu'on appelle *propager* l'erreur et donne ainsi plus de
pouvoir au code qui appelle la fonction où il pourrait y avoir plus
d'informations ou d'instructions pour traiter l'erreur que si c'était dans le
contexte de votre code.

Par exemple, l'entrée 9-6 montre une fonction qui lit un nom d'utilisateur à
partir d'un fichier. Si le fichier n'existe pas ou ne peux pas être lu, cette
fonction va retourner ces erreurs au code qui a appellé cette fonction :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

<span class="caption">Entrée 9-6 : une fonction qui retourne les erreurs au
code qui l'appelle en utilisant `match`</span>

Regardons d'abord le type de retour de la fonction :
`Result<String, io::Error>`. Cela signifie que la fonction retourne une valeur
de type `Result<T, E>` où le paramètre générique `T` a été rempli avec le type
`String`, et le paramètre générique `E` a été rempli avec le type `io::Error`.
Si cette fonction réussi sans aucun problème, le code qui appelle cette
fonction va récupérer une valeur `Ok` qui contient un `String`, le nom
d'utilisateur que cette fonction lit dans le fichier. Si cette fonction
rencontre n'importe quel problème, le code qui appelle cette fonction va
récupérer une valeur `Err` qui contient une instance de `io::Error` qui apporte
plus d'informations sur la raison du problème. Nous avons choisi `io::Error`
comme type de retour de cette fonction parce que c'est le type d'erreur de
retour par chacune des opérations qu'on appelle dans le corps de cette fonction
qui peuvent échouer : la fonction `File::open` et la méthode `read_to_string`.

Le corps de la fonction commence par appeller la fonction `File::open`. Ensuite
nous gérons la valeur `Result` retourné, avec un `match` similaire au `match`
dans l'entrée 9-4, seulement, au lieu d'appeller `panic!` dans le cas de `Err`,
nous retournons prématurément la fonction et nous retournons la valeur d'erreur
de `File::open` au code appellant avec la valeur d'erreur de cette fonction. Si
`File::open` réussit, nous enregistrons le manipulateur de fichier dans la
variable `f` et nous continuons.

Ensuite nous créons un nouveau `String` dans la variable `s` et nous appellons
la méthode `read_to_string` sur le manipulateur de fichier dans `f` pour lire
le contennu du fichier dans `s`. La méthode `read_to_string` retourne aussi un
`Result` parce qu'elle peut échouer, même si `File::open` réussit. Donc nous
avons un nouveau `match` pour gérer ce `Result` : si `read_to_string` réussit,
alors notre fonction a a réussi, et nous retournons le nom d'utilisateur à
partir du fichier qui est maintenant dans `s`, envelopé dans un `Ok`. Si
`read_to_string` échoue, nous retournons la valeur d'erreur de la même façon
que nous avons retourné la valeur d'erreur dans le `match` qui gèrait la valeur
de retour de `File::open`. Cependant, nous n'avons pas besoin de dire
explicitement `return`, car c'est la dernière instruction dans la fonction.

Le code qui appelle ce code va devoir ensuite gérer soit une valeur `Ok` qui
contient le nom d'utilisateur, ou une valeur `Err` qui contient une
`io::Error`. Nous ne savons pas ce que va faire le code appellant avec ces
valeurs. Si le code appellant obtient une valeur `Err`, il peut utiliser un
`panic!` et faire planter le programme, utiliser un nom d'utilisateur par
défaut, ou chercher le nom d'utilisateur autre part que dans ce fichier, par
example. Nous n'avons n'avons pas assez d'informations sur ce que le code
appellant a l'intention de faire, donc nous remontons toutes les informations
de succès ou d'erreur vers le haut pour qu'elles soient traitées correctement.

Cette façon de propager les erreurs est si courrant dans Rust que Rust fournit
l'opérateur du point d'interrogation `?` pour faciliter ceci.

#### Un raccourci pour propager les erreurs : `?`

L'entrée 9-7 montre une implémentation de `read_username_from_file` qui a les
mêmes fonctionnalités qu'elle a dans l'entrée 9-6, mais cette implémentation
utilise l'opérateur du point d'interrogation :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

<span class="caption">Entrée 9-7: une fonction qui retourne les erreurs au code
appellant en utilisant `?`</span>

The `?` placed after a `Result` value is defined to work in almost the same way
as the `match` expressions we defined to handle the `Result` values in Listing
9-6. If the value of the `Result` is an `Ok`, the value inside the `Ok` will
get returned from this expression and the program will continue. If the value
is an `Err`, the value inside the `Err` will be returned from the whole
function as if we had used the `return` keyword so the error value gets
propagated to the calling code.

The one difference between the `match` expression from Listing 9-6 and what the
question mark operator does is that when using the question mark operator,
error values go through the `from` function defined in the `From` trait in the
standard library. Many error types implement the `from` function to convert an
error of one type into an error of another type. When used by the question mark
operator, the call to the `from` function converts the error type that the
question mark operator gets into the error type defined in the return type of
the current function that we’re using `?` in. This is useful when parts of a
function might fail for many different reasons, but the function returns one
error type that represents all the ways the function might fail. As long as
each error type implements the `from` function to define how to convert itself
to the returned error type, the question mark operator takes care of the
conversion automatically.

In the context of Listing 9-7, the `?` at the end of the `File::open` call will
return the value inside an `Ok` to the variable `f`. If an error occurs, `?`
will return early out of the whole function and give any `Err` value to the
calling code. The same thing applies to the `?` at the end of the
`read_to_string` call.

The `?` eliminates a lot of boilerplate and makes this function’s
implementation simpler. We could even shorten this code further by chaining
method calls immediately after the `?` as shown in Listing 9-8:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

<span class="caption">Listing 9-8: Chaining method calls after the question
mark operator</span>

We’ve moved the creation of the new `String` in `s` to the beginning of the
function; that part hasn’t changed. Instead of creating a variable `f`, we’ve
chained the call to `read_to_string` directly onto the result of
`File::open("hello.txt")?`. We still have a `?` at the end of the
`read_to_string` call, and we still return an `Ok` value containing the
username in `s` when both `File::open` and `read_to_string` succeed rather than
returning errors. The functionality is again the same as in Listing 9-6 and
Listing 9-7; this is just a different, more ergonomic way to write it.

#### `?` Can Only Be Used in Functions That Return Result

The `?` can only be used in functions that have a return type of `Result`,
because it is defined to work in the same way as the `match` expression we
defined in Listing 9-6. The part of the `match` that requires a return type of
`Result` is `return Err(e)`, so the return type of the function must be a
`Result` to be compatible with this `return`.

Let’s look at what happens if we use `?` in the `main` function, which you’ll
recall has a return type of `()`:

```rust,ignore
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

When we compile this code, we get the following error message:

```text
error[E0277]: the `?` operator can only be used in a function that returns
`Result` (or another type that implements `std::ops::Try`)
 --> src/main.rs:4:13
  |
4 |     let f = File::open("hello.txt")?;
  |             ------------------------
  |             |
  |             cannot use the `?` operator in a function that returns `()`
  |             in this macro invocation
  |
  = help: the trait `std::ops::Try` is not implemented for `()`
  = note: required by `std::ops::Try::from_error`
```

This error points out that we’re only allowed to use the question mark operator
in a function that returns `Result`. In functions that don’t return `Result`,
when you call other functions that return `Result`, you’ll need to use a
`match` or one of the `Result` methods to handle it instead of using `?` to
potentially propagate the error to the calling code.

Now that we’ve discussed the details of calling `panic!` or returning `Result`,
let’s return to the topic of how to decide which is appropriate to use in which
cases.
