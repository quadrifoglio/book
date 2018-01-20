## Les Slices

Un autre type de données qui n'applique pas le principe d'appartenance est le
*slice*. Un slice vous permet d'avoir une référence vers une suite continue
d'éléments dans une collection plutôt que toute la collection.

Voici un petit problème de programmation : écrire une fonction qui prend une
chaîne de caractères et retourne le premier mot qu'elle trouve dans cette
chaîne. Si la fonction ne trouve pas d'espace dans la chaîne, cela veut dire
que toute la chaîne est un seul mot, donc la chaîne en entier doit être
retournée.

Imaginons la signature de cette fonction :

```rust,ignore
fn first_word(s: &String) -> ?
```

Cette fonction, `first_word`, prends un `&String` comme paramètre. Nous ne
voulons pas se l'approprier, donc tout va bien. Mais que devons-nous
retourner ? Nous n'avons pas de moyen de désigner une *partie* de chaîne de
caractères. Cependant, nous pouvons retourner l'index de la fin du mot.
Essayons cela dans l'entrée 4-5 :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

<span class="caption">Entrée 4-5 : la fonction `first_word` qui retourne la
valeur d'index d'octet dans le paramètre `String`</span>

Regardons un peu plus en détail ce code. Puisque nous avons besoin de parcourir
le `String` éléments par éléments et vérifier si leur valeur est un espace,
nous allons convertir notre `String` en tableau d'octets en utilisant la
méthode `as_bytes` :

```rust,ignore
let bytes = s.as_bytes();
```
Ensuite, nous créons un itérateur sur le tableau d'octets en utilisant la
méthode `iter` :

```rust,ignore
for (i, &item) in bytes.iter().enumerate() {
```

Nous discuterons plus en détail des itérateur dans le chapitre 13. Pour le
moment, sachez que ce `iter` est une méthode qui retourne chaque élément dans
une collection, et que `enumerate` enveloppe le résultat de `iter` et retourne
plutôt chaque élément comme une partie d'un tuple. Le premier élément du tuple
retourné est l'index, et le second  élément est une référence vers l'élément.
C'est un peu plus pratique que de calculer les index par nous-mêmes.

Comme la méthode `enumerate` retourne un tuple, ne pouvons utiliser une
technique pour décomposer ce tuple, comme nous pourrions le faire n'importe où
avec Rust. Donc dans la boucle `for`, nous précisions un schéma qui indique que
nous définissons `i` pour l'index à partir du tuple et `&item` for chaque octet
dans le tuple. Comme nous obtenons une référence vers l'élément avec
`.iter().enumerate()`, nous utilisons `&` dans le schéma.

Nous recherchons l'octet qui représente l'espace en utilisant la syntaxe des
mots binaires. Si nous trouvons un espace, nous retournons sa position. Sinon,
nous retournons la taille du string en utilisant `s.len()` :

```rust,ignore
    if item == b' ' {
        return i;
    }
}
s.len()
```

Nous avons maintenant une façon de trouver l'index de la fin du premier mot
dans la chaîne de caractères, mais il y a un problème. Nous retournons un
`usize` seul, mais il n'est important que lorsqu'il est mis en rapport avec
le `&String`. Autrement dit, parce qu'il a une valeur séparée du `String`, il
n'y a pas de garantie qu'il sera toujours valide dans le futur. Imaginons
le programme dans l'entrée 4-6 qui utilise la fonction de l'entrée 4-5 :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
# fn first_word(s: &String) -> usize {
#     let bytes = s.as_bytes();
#
#     for (i, &item) in bytes.iter().enumerate() {
#         if item == b' ' {
#             return i;
#         }
#     }
#
#     s.len()
# }
#
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word aura 5 comme valeur.

    s.clear(); // Ceci vide le String, il faut maintenant "".

    // word a toujours la valeur 5 ici, mais il n'y a plus de chaîne qui donne
    // du sens à la valeur 5. word est maintenant complètement invalide !
}
```

<span class="caption">Entrée 4-6 : On stocke le résultat de l'appel à la
fonction `first_word` et ensuite on change le contenu du `String`</span>

Ce programme se compile sans aucune erreur et serait toujours OK si nous
utilisions `word` après avoir appelé `s.clear()`. `word` n'est pas du tout lié
à l'état de `s`, donc `word` contient toujours la valeur `5`. Nous pourrions
utiliser cette valeur `5` avec la variable `s` pour essayer d'en extraire le
premier mot, mais cela serait un bogue, car le contenu de `s` a changé depuis
que nous avons enregistré `5` dans `word`.

Se préoccuper en permanence que l'index dans `word` ne soit plus synchronisé
avec les données dans `s` est fastidieux et source d'erreur ! La gestion de ces
index est encore plus risquée si nous écrivons une fonction second_word. Sa
signature ressemblerait à quelque chose comme ceci :

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Maintenant nous gérons un index de début *et* un index de fin, et nous avons
encore plus de valeurs qui sont calculées à partir de la donnée dans un
état particulier, mais qui n'est pas lié à son état en temps réel. Nous avons
maintenant trois variables isolées qui ont besoin d'être maintenu à jour.

Heureusement, Rust a une solution pour ce problème : les slices de chaînes de
caractères.

### Les slices de chaînes de caractères

Un *slice de chaîne de caractère* est une référence à une partie d'un `String`,
et ressemble à ceci :

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

Ce serait comme prendre une référence pour tout le `String`, mais avec en plus le
mot `[0..5]`. Plutôt qu'une référence vers tout le `String`, c'est une
référence à une partie du `String`. La syntaxe `début..fin` est une intervalle
qui commence à `start` et comprends la suite jusqu'à `end` exclus.

Nous pouvons créer des slices en utilisant une intervalle entre crochets en
spécifiant `[index_debut..index_fin]`, où `index_debut` est la première
position dans le slice et `index_fin` est une position en plus que la dernière
position dans le slice. En interne, la structure de données du slice enregistre
la position de départ et la longeur du slice, ce qui correspond à `index_fin`
moins `index_debut`. Donc dans le cas de `let world = &s[6..11];`, `world` va
être un slice qui a un pointeur vers le sixième octet de `s` et une longueur
de 5.

L'illustration 4-6 montre cela dans un diagramme.


<img alt="world contient un pointeur vers le sixième octet du String s et une longueur de 5" src="img/trpl04-06.svg" class="center" style="width: 50%;" />

<span class="caption">Illustration 4-6 : un slice de String qui pointe vers
une partie de `String`</span>

Avec la syntaxe d'interface `..` de Rust, si vous voulez commencer au premier
index (zéro), vous pouvez ne rien mettre avant les deux points. Autrement dit,
ceci est identique :

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

De la même manière, si votre slice contient les derniers octets du `String`,
vous pouvez ne rien mettre à la fin. Cela veut dire que ces deux instructions
sont identiques :

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Vous pouvez aussi ne mettre aucune limite pour faire un slice de toute la
chaîne de caractères. Donc ces deux cas sont identiques :

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Note : Les indexes de l'intervalle d'un slice d'un String doivent toujours
> être des valeurs compatibles avec l'UTF-8. Si vous essayez de créer un slice
> d'une chaîne de caractères au millieu d'un caractère codé sur plusieurs
> octets, votre programme va se fermer avec une erreur. Pour que nous abordions
> simplement les slice de chaînes de caractères, nous supposerons que nous
> utilisons l'ASCII uniquement dans cette section; nous discuterons plus en
> détails de la gestion UTF-8 dans la section “Chaînes de caractères” au
> chapitre 8.

Avec toutes ces informations, essayons de ré-écrire `first_word` pour retourner
un slice. Le type pour les “slices de chaînes de caractères” s'écrit `&str` :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

Nous récupérons l'index de la fin du mot de la même façon que nous l'avons fait
dans l'entrée 4-5, en cherchant la première occurrence d'un espace. Quand nous
trouvons un espace, nous retournons un slice de chaîne de caractère en
utilisant le début de la chaîne de caractères et l'index de l'espace comme
indices de début et fin.

Maintenant, quand nous appelons `first_word`, nous récupérons une seule valeur
qui est liée à la donnée de base. La valeur est construite avec une référence
vers le point de départ du slice et nombre d'éléments dans le slice.

Retourner un slice fonctionnerait aussi pour une fonction `second_word` :

```rust,ignore
fn second_word(s: &String) -> &str {
```

Nous avons maintenant une API simple qui est bien plus difficile à perturber,
puisque le compilateur va s'assurer que les références dans le `String` seront
toujours en vigueur. Souvenez-vous du bogue dans le programme de l'entrée 4-6,
quand nous avions un index vers la fin du premier mot mais qu'ensuite nous
avions vidé la chaîne de caractères et que notre index n'était plus valide ?
Ce code était logiquement incorrect, mais nous n'avons pas immédiatement vu
d'erreurs. Les problèmes vont arriver plus tard si nous essayons d'utiliser
l'index du premier mot avec une chaîne de caractère qui a été vidée. Les slices
rendent ce bogue impossible et nous fait savoir bien plus tôt quand nous avons
un problème avec notre code. Utiliser la version avec le slice de `first_word`
va lever une erreur au moment de la compilation :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust,ignore
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // Erreur !
}
```

Voici l'erreur du compilateur :

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let word = first_word(&s);
  |                            - immutable borrow occurs here
5 |
6 |     s.clear(); // Error!
  |     ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

Rappellons-nous que d'après les règles de référencement, si nous avons une
référence immuable vers quelque chose, nous ne pouvons pas avoir une référence
modifiable en même temps. Etant donné que `clear` a besoin de raccourcir le
`String`, il essaye de prendre une référence modifiable, ce qui échoue. Non
seulement Rust a simplifié l'utilisation de notre API, mais il a aussi éliminé
une catégorie entière d'erreurs au moment de la compilation !

#### Les chaînes de caractères pures sont des Slices

Souvenez-vous lorsque nous avons vu les chaînes des caractères pures qui
étaient enregistrées dans le binaire. Maintenant que nous connaissons les
slices, nous pouvons comprendre comme il faut les chaînes des caractères pures.

```rust
let s = "Hello, world!";
```

Ici, le type de `s` est un `&str` : c'est un slice qui pointe vers un endroit
spécifique du binaire. C'est pourquoi les chaînes des caractères pures sont
immuables; `&str` est une référence immuable.

#### Des slices de chaînes de caractères en paramètres

Apprendre que vous pouvez utiliser des slices de texte et de `String` nous
amène à apporter quelques améliorations sur `first_word`, voici sa signature :

```rust,ignore
fn first_word(s: &String) -> &str {
```

Un Rustacéen plus expérimenté écrirait plutôt la ligne suivante, car cela nous
permet d'utiliser la même fonction sur les `String` et les `&str` :

```rust,ignore
fn first_word(s: &str) -> &str {
```

Si nous avions un slice de chaîne de caractères, nous pouvons lui envoyer
directement. Si nous avions un `String`, nous pourrions envoyer un slice de
tout le `String`. Concevoir une fonction pour prendre un slice de chaîne de
caractères plutôt qu'une référence à une chaîne de caractères rend notre API
plus générique et plus utile sans perdre aucune fonctionnalité :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
# fn first_word(s: &str) -> &str {
#     let bytes = s.as_bytes();
#
#     for (i, &item) in bytes.iter().enumerate() {
#         if item == b' ' {
#             return &s[0..i];
#         }
#     }
#
#     &s[..]
# }
fn main() {
    let my_string = String::from("hello world");

    // first_word travaille avec un slice de `String`
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // first_word travaille avec un slice de chaîne de caractères pure
    let word = first_word(&my_string_literal[..]);

    // puisque les chaînes de caractères *sont* déjà des slices de chaînes
    // de caractères, ceci fonctionne aussi, sans la syntaxe de slice !
    let word = first_word(my_string_literal);
}
```

### Les autres slices

Les slices de chaînes de caractères, comme vous pouvez l'imaginer, sont
spécifiques aux chaînes de caractères. Mais il y a aussi un type plus
générique. Admettons ce tableau :

```rust
let a = [1, 2, 3, 4, 5];
```

Comme nous pouvons nous référer à une partie de chaîne de caractères, nous
pouvons nous référer à une partie d'un tableau et nous le faisons comme ceci :

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```

Ce slice est de type `&[i32]`. Il fonctionne de la même manière que les slices
de chaînes de caractères, en enregistrant une référence vers le premier élément
et une longueur. Vous pouvez utiliser ce type de slice pour tous les autres
types de collections. Nous discuterons de ces collections en détail quand nous
verrons les vecteurs au Chapitre 8.

## Résumé

Les concepts d'appartenance, d'emprunt, et les slices garantissent la sécurité
de la mémoire dans les programmes Rust au moment de la compilation. Le langage
Rust vous donne le contrôle sur l'utilisation de la mémoire comme tous les
systèmes de langages de programmation, mais avoir le propriétaire des données
qui nettoie automatiquement ces données quand il sort de la portée vous permet
de ne pas avoir à écrire et déboguer du code en plus pour avoir ce contrôle.

L'appropriation influe sur de nombreux fonctionnements de Rust, donc nous
allons encore parler de ces concepts plus loin dans le livre. Allons
maintenant au chapitre suivant et regardons comment regrouper des données
ensemble dans un `struct`.
