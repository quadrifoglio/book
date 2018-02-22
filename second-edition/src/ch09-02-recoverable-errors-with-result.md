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
peut échouer : dans l'entrée 9-3 nous essayons d'ouvrir un fichier :

<span class="filename">Fichier : src/main.rs</span>

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

<span class="filename">Fichier : src/main.rs</span>

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

<span class="filename">Fichier : src/main.rs</span>

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

The type of the value that `File::open` returns inside the `Err` variant is
`io::Error`, which is a struct provided by the standard library. This struct
has a method `kind` that we can call to get an `io::ErrorKind` value.
`io::ErrorKind` is an enum provided by the standard library that has variants
representing the different kinds of errors that might result from an `io`
operation. The variant we want to use is `ErrorKind::NotFound`, which indicates
the file we’re trying to open doesn’t exist yet.

The condition `if error.kind() == ErrorKind::NotFound` is called a *match
guard*: it’s an extra condition on a `match` arm that further refines the arm’s
pattern. This condition must be true for that arm’s code to be run; otherwise,
the pattern matching will move on to consider the next arm in the `match`. The
`ref` in the pattern is needed so `error` is not moved into the guard condition
but is merely referenced by it. The reason `ref` is used to take a reference in
a pattern instead of `&` will be covered in detail in Chapter 18. In short, in
the context of a pattern, `&` matches a reference and gives us its value, but
`ref` matches a value and gives us a reference to it.

The condition we want to check in the match guard is whether the value returned
by `error.kind()` is the `NotFound` variant of the `ErrorKind` enum. If it is,
we try to create the file with `File::create`. However, because `File::create`
could also fail, we need to add an inner `match` statement as well. When the
file can’t be opened, a different error message will be printed. The last arm
of the outer `match` stays the same so the program panics on any error besides
the missing file error.

### Shortcuts for Panic on Error: `unwrap` and `expect`

Using `match` works well enough, but it can be a bit verbose and doesn’t always
communicate intent well. The `Result<T, E>` type has many helper methods
defined on it to do various tasks. One of those methods, called `unwrap`, is a
shortcut method that is implemented just like the `match` statement we wrote in
Listing 9-4. If the `Result` value is the `Ok` variant, `unwrap` will return
the value inside the `Ok`. If the `Result` is the `Err` variant, `unwrap` will
call the `panic!` macro for us. Here is an example of `unwrap` in action:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

If we run this code without a *hello.txt* file, we’ll see an error message from
the `panic!` call that the `unwrap` method makes:

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

Another method, `expect`, which is similar to `unwrap`, lets us also choose the
`panic!` error message. Using `expect` instead of `unwrap` and providing good
error messages can convey your intent and make tracking down the source of a
panic easier. The syntax of `expect` looks like this:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

We use `expect` in the same way as `unwrap`: to return the file handle or call
the `panic!` macro. The error message used by `expect` in its call to `panic!`
will be the parameter that we pass to `expect`, rather than the default
`panic!` message that `unwrap` uses. Here’s what it looks like:

```text
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

Because this error message starts with the text we specified, `Failed to open
hello.txt`, it will be easier to find where in the code this error message is
coming from. If we use `unwrap` in multiple places, it can take more time to
figure out exactly which `unwrap` is causing the panic because all `unwrap`
calls that panic print the same message.

### Propagating Errors

When you’re writing a function whose implementation calls something that might
fail, instead of handling the error within this function, you can return the
error to the calling code so that it can decide what to do. This is known as
*propagating* the error and gives more control to the calling code where there
might be more information or logic that dictates how the error should be
handled than what you have available in the context of your code.

For example, Listing 9-6 shows a function that reads a username from a file. If
the file doesn’t exist or can’t be read, this function will return those errors
to the code that called this function:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 9-6: A function that returns errors to the
calling code using `match`</span>

Let’s look at the return type of the function first: `Result<String,
io::Error>`. This means the function is returning a value of the type
`Result<T, E>` where the generic parameter `T` has been filled in with the
concrete type `String`, and the generic type `E` has been filled in with the
concrete type `io::Error`. If this function succeeds without any problems, the
code that calls this function will receive an `Ok` value that holds a
`String`—the username that this function read from the file. If this function
encounters any problems, the code that calls this function will receive an
`Err` value that holds an instance of `io::Error` that contains more
information about what the problems were. We chose `io::Error` as the return
type of this function because that happens to be the type of the error value
returned from both of the operations we’re calling in this function’s body that
might fail: the `File::open` function and the `read_to_string` method.

The body of the function starts by calling the `File::open` function. Then we
handle the `Result` value returned with a `match` similar to the `match` in
Listing 9-4, only instead of calling `panic!` in the `Err` case, we return
early from this function and pass the error value from `File::open` back to the
calling code as this function’s error value. If `File::open` succeeds, we store
the file handle in the variable `f` and continue.

Then we create a new `String` in variable `s` and call the `read_to_string`
method on the file handle in `f` to read the contents of the file into `s`. The
`read_to_string` method also returns a `Result` because it might fail, even
though `File::open` succeeded. So we need another `match` to handle that
`Result`: if `read_to_string` succeeds, then our function has succeeded, and we
return the username from the file that’s now in `s` wrapped in an `Ok`. If
`read_to_string` fails, we return the error value in the same way that we
returned the error value in the `match` that handled the return value of
`File::open`. However, we don’t need to explicitly say `return`, because this
is the last expression in the function.

The code that calls this code will then handle getting either an `Ok` value
that contains a username or an `Err` value that contains an `io::Error`. We
don’t know what the calling code will do with those values. If the calling code
gets an `Err` value, it could call `panic!` and crash the program, use a
default username, or look up the username from somewhere other than a file, for
example. We don’t have enough information on what the calling code is actually
trying to do, so we propagate all the success or error information upwards for
it to handle appropriately.

This pattern of propagating errors is so common in Rust that Rust provides the
question mark operator `?` to make this easier.

#### A Shortcut for Propagating Errors: `?`

Listing 9-7 shows an implementation of `read_username_from_file` that has the
same functionality as it had in Listing 9-6, but this implementation uses the
question mark operator:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 9-7: A function that returns errors to the
calling code using `?`</span>

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
