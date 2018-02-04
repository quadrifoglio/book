## Variables et Mutabilité

Comme mentionné dans le Chapitre 2, par défaut, les variables sont *immuables*.
C'est un des nombreux coups de pouces de Rust vous encourageant à écrire votre
code d'une façon à tirer avantage de la sûreté et de la concurrence facilitée
que Rust propose. Cependant, vous avez tout de même la possibilité de rendre
vos variables mutables. Explorons comment et pourquoi Rust vous encourage à
favoriser l'immutabilité, et pourquoi vous pourriez choisir d'y renoncer.

Lorsque qu'une variable est immuable, cela signifie qu'une fois qu'une valeur
est liée à un nom, vous ne pouvez pas changer cette valeur. À titre
d'illustration, générons un nouveau projet appelé *variables* dans votre
dossier *projects* en utilisant `cargo new --bin variables`.

Ensuite, dans votre nouveau dossier *variables*, ouvrez *src/main.rs* et remplacez son contenu par ceci :

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

Sauvegardez et lancez le programme en utilisant `cargo run`. Vous devriez obtenir un message d'erreur, tel qu'indiqué par la sortie suivante :

```text
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         - first assignment to `x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

Cet exemple montre comment le compilateur vous aide à identifier les erreurs
dans vos programmes. Même si les erreurs de compilation peuvent s'avérer
frustrantes, elles signifient uniquement que votre programme n'est pour le
moment pas en train de faire ce que vous voulez qu'il fasse en toute sécurité ;
elles ne signifient *pas* que vous êtes un mauvais en programmation ! Même les
Rustacéens et Rustacéennes ayant de l'expérience continuent d'avoir des erreurs de compilation.

Cette erreur indique que la cause du problème est qu'il est `impossible 
d'assigner à deux reprises la variable immuable x`, car nous avons essayé de 
donner à `x`, qui est une variable immuable, une seconde valeur.

Il est important que nous obtenions des erreurs lors de la compilation quand
nous essayons de changer une valeur qui a précédemment été désignée comme
immuable, car cette situation précise peut donner lieu à des bugs. Si une
partie de notre code opère sur le postulat qu'une valeur ne changera jamais et
qu'une autre partie de notre code modifie cette valeur, il est possible que la
première partie du code ne fera pas ce pour quoi elle a été conçue. Cette
source d'erreur peut être difficile à identifier après coup, particulièrement
lorsque la seconde partie du code ne modifie que *quelquefois* cette valeur.

En Rust, le compilateur garantie que lorsque nous déclarons qu'une variable ne
changera pas, elle ne changera vraiment pas. Cela signifie que lorsque que vous
lisez et écrivez du code, vous n'avez pas à vous souvenir de comment et où une
valeur pourrait changer, ce qui peut rendre le code plus facile à comprendre.

Mais la mutabilité peut s'avérer très utile. Les variables ne sont seulement
qu'immuables par défaut ; nous pouvons les rendre mutables en ajoutant `mut`
devant notre nom de variable. En plus d'autoriser cette valeur à changer, cela
communique l'intention aux futurs lecteurs de ce code que d'autres parties du
code vont modifier cette valeur variable.

For example, change *src/main.rs* to the following:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

When we run this program, we get the following:

```text
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6
```

Using `mut`, we’re allowed to change the value that `x` binds to from `5` to
`6`. In some cases, you’ll want to make a variable mutable because it makes the
code more convenient to write than an implementation that only uses immutable
variables.

There are multiple trade-offs to consider, in addition to the prevention of
bugs. For example, in cases where you’re using large data structures, mutating
an instance in place may be faster than copying and returning newly allocated
instances. With smaller data structures, creating new instances and writing in
a more functional programming style may be easier to reason about, so the lower
performance might be a worthwhile penalty for gaining that clarity.

### Differences Between Variables and Constants

Being unable to change the value of a variable might have reminded you of
another programming concept that most other languages have: *constants*. Like
immutable variables, constants are also values  that are bound to a name and
are not allowed to change, but there are a few differences between constants
and variables.

First, we aren’t allowed to use `mut` with constants: constants aren’t only
immutable by default, they’re always immutable.

We declare constants using the `const` keyword instead of the `let` keyword,
and the type of the value *must* be annotated. We’re about to cover types and
type annotations in the next section, “Data Types,” so don’t worry about the
details right now, just know that we must always annotate the type.

Constants can be declared in any scope, including the global scope, which makes
them useful for values that many parts of code need to know about.

The last difference is that constants may only be set to a constant expression,
not the result of a function call or any other value that could only be
computed at runtime.

Here’s an example of a constant declaration where the constant’s name is
`MAX_POINTS` and its value is set to 100,000. (Rust constant naming convention
is to use all upper case with underscores between words):

```rust
const MAX_POINTS: u32 = 100_000;
```

Constants are valid for the entire time a program runs, within the scope they
were declared in, making them a useful choice for values in your application
domain that multiple parts of the program might need to know about, such as the
maximum number of points any player of a game is allowed to earn or the speed
of light.

Naming hardcoded values used throughout your program as constants is useful in
conveying the meaning of that value to future maintainers of the code. It also
helps to have only one place in your code you would need to change if the
hardcoded value needed to be updated in the future.

### Shadowing

As we saw in the guessing game tutorial in Chapter 2, we can declare a new
variable with the same name as a previous variable, and the new variable
*shadows* the previous variable. Rustaceans say that the first variable is
*shadowed* by the second, which means that the second variable’s value is what
we’ll see when we use the variable. We can shadow a variable by using the same
variable’s name and repeating the use of the `let` keyword as follows:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);
}
```

This program first binds `x` to a value of `5`. Then it shadows `x` by
repeating `let x =`, taking the original value and adding `1` so the value of
`x` is then `6`. The third `let` statement also shadows `x`, taking the
previous value and multiplying it by `2` to give `x` a final value of `12`.
When you run this program, it will output the following:

```text
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/variables`
The value of x is: 12
```

This is different than marking a variable as `mut`, because unless we use the
`let` keyword again, we’ll get a compile-time error if we accidentally try to
reassign to this variable. We can perform a few transformations on a value but
have the variable be immutable after those transformations have been completed.

The other difference between `mut` and shadowing is that because we’re
effectively creating a new variable when we use the `let` keyword again, we can
change the type of the value, but reuse the same name. For example, say our
program asks a user to show how many spaces they want between some text by
inputting space characters, but we really want to store that input as a number:

```rust
let spaces = "   ";
let spaces = spaces.len();
```

This construct is allowed because the first `spaces` variable is a string type,
and the second `spaces` variable, which is a brand-new variable that happens to
have the same name as the first one, is a number type. Shadowing thus spares us
from having to come up with different names, like `spaces_str` and
`spaces_num`; instead, we can reuse the simpler `spaces` name. However, if we
try to use `mut` for this, as shown here, we’ll get a compile-time error:

```rust,ignore
let mut spaces = "   ";
spaces = spaces.len();
```

The error says we’re not allowed to mutate a variable’s type:

```text
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected &str, found usize
  |
  = note: expected type `&str`
             found type `usize`
```

Now that we’ve explored how variables work, let’s look at more data types they
can have.
