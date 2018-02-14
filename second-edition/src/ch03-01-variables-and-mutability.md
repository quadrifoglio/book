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

<span class="filename">Fichier : src/main.rs</span>

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

Par exemple, modifions *src/main.rs* par :

<span class="filename">Fichier : src/main.rs</span>

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

Lorsque nous exécutons le programme, nous obtenons :

```text
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6
```

En utilisant `mut`, nous sommes autorisés à changer la valeur à laquelle `x`
est reliée de `5` à `6`. Dans certains cas, vous allez vouloir rendre une
variable mutable car cela rend le code plus pratique à écrire qu'une
implémentation n'utilisant que des variables immuables.

Il y a plusieurs compromis à prendre en considération, outre la prévention des
bugs. Par exemple, dans le cas où vous utiliseriez de larges structures de
données, muter une instance déjà existante peut être plus rapide que copier et
retourner une instance nouvellement allouée. Sur des structures de données plus
petites, créer de nouvelles instances et écrire dans un style de programmation
plus fonctionnel peut rendre votre code plus facile à comprendre, ainsi, peut
être qu'un coût plus élevé en performance est un moindre mal face au gain de
clareté apporté.

### Différences Entre Variable et Constante

Être incapable de changer la valeur d'une variable peut vous avoir rappelé un autre concept de programmation que de nombreux autres langages possèdent : les *constantes*. Comme les variables immuables, les constantes sont également des valeurs qui sont liées à un nom et qui ne peuvent être modifiées, mais il y a quelques différences entre les constantes et les variables.

D'abord, nous ne sommes pas autorisés à utiliser `mut` avec les constantes : les constantes ne sont pas seulement immuables par défaut, elles le sont toujours.

Nous déclarons les constantes en utilisant le mot-clé `const` à la place du
mot-clé `let`, et le type de la valeur *doit* être annoté. Nous sommes sur le
point de traiter des types et des annotations de types dans la prochaine
section, “Types de données,” donc ne vous inquiétez pas des détails pour le
moment, rappelez-vous juste que vous devez toujours annoter leur type.

Les constantes peuvent être déclarées dans n'importe quelle portée, y compris
la portée globale, ce qui les rend très utiles pour des valeurs que de
nombreuses parties de votre code ont besoin de connaître.

La dernière différence est que les constantes ne peuvent être définies que par
une expression constante, et non pas le résultat d'un appel de fonction ou
n'importe quelle autre valeur qui ne pourrait être calculée qu'à l'exécution.

Voici un exemple d'une déclaration de constante où le nom de la constante est 
MAX_POINTS` et où sa valeur est définie à 100 000 (en Rust, la convention de 
nommage des constantes est d'utiliser des majuscules pour chaque lettre et des
tirets bas entre chaque mot) :

```rust
const MAX_POINTS: u32 = 100_000;
```

Les constantes sont valables pendant toute la durée d'exécution du programme à
l'intérieur de la portée dans laquelle elles sont déclarées, ce qui en font de
très bon choix lorsque plusieurs parties d'un programme doivent connaître
certaines valeurs, comme par exemple le nombre maximum de points qu'un joueur
est autorisé à gagner ou la vitesse de la lumière.

Déclarer des valeurs codées en dur et utilisées tout le long de votre programme
comme constantes est utile car cela transmet la signification de ces valeurs
aux futurs mainteneurs de votre code. Cela permet également de n'avoir qu'un
seul endroit de votre code à modifier si une valeur codée en dur doit être mise
à jour dans le futur.

### *Shadowing*

Comme nous l'avons vu dans le jeu du Chapitre 2, nous pouvons déclarer de
nouvelles variables avec le même nom qu'une variable précédente, et que la
nouvelle variable *shadow*, occulte la première. Les Rustacéens disent que la
première variable est *shadowed*, occultée par la seconde, ce qui signifie que
la valeur de la seconde variable sera ce que nous obtiendrons lorsque nous
utiliserons cette variable. Nous pouvons occulter une variable en utilisant le
même nom de variable et en réutilisant le mot-clé `let` comme suit :

<span class="filename">Fichier : src/main.rs</span>

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);
}
```

En premier lieu, ce programme lie `x` à la valeur `5`. Puis il *shadow* `x` en
répétant `let x =`, ce qui récupère la valeur originelle et lui ajoute `1` : la
valeur de `x` est désormais `6`. La troisième déclaration `let` *shadow*
également `x`, prenant la précédente valeur et la multipliant par `2` pour
donner à `x` une valeur finale de `12`. Lorsque nous exécutons ce programme,
nous obtenons en sortie ceci :

```text
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/variables`
The value of x is: 12
```

Ceci est différent que marquer une variable comme `mut`, car à moins d'utiliser
le mot-clé `let` une nouvelle fois, nous obtenons une erreur de compilation si
nous essayons accidentellement de réassigner cette variable. Nous pouvons
effectuer quelques transformations sur une valeur puis faire en sorte que la
variable soit immuable après que ces transformations soient terminées.

L'autre différence entre le `mut` et le *shadowing* est la création effective
d'une nouvelle variable lorsque nous utilisons le mot-clé `let` une nouvelle
fois, ce qui nous permet de changer le type de la valeur, mais en réutilisant
le même nom. Par exemple, disons que notre programme demande à un utilisateur
le nombre d'espaces à afficher entre du texte en saisissant des caractères
d'espace, mais que nous voulions quand même enregistrer cette saisie comme un
nombre :

```rust
let spaces = "   ";
let spaces = spaces.len();
```

Cette conception est autorisée car la première variable `spaces` est du type
*string*, alors que la seconde variable `spaces`, qui est une toute nouvelle
variable se trouvant avoir le même nom que la première, est du type nombre.
L'utilisation du *shadowing* nous évite ainsi d'avoir à trouver des noms
différents, comme `spaces_str` et `spaces_num` ; nous pouvons plutôt utiliser
le simple nom de `spaces`. En revanche, si nous essayons d'utiliser `mut` pour
cela, comme montré ci-dessous, nous obtenons une erreur de compilation :

```rust,ignore
let mut spaces = "   ";
spaces = spaces.len();
```

Cette erreur indique que nous ne pouvons pas muer le type d'une variable :

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

Maintenant que nous avons exploré comment fonctionne les variables, étudions de
nouveaux types de données qu'elles peuvent avoir.
