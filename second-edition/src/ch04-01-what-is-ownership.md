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

#### Les interactions entre les variables et les données : les déplacements

Plusieurs variables peuvent interragir avec les mêmes données de différentes
manières avec Rust. Regardons un exemple avec un entier dans l'entrée 4-2 :

```rust
let x = 5;
let y = x;
```

<span class="caption">Entrée 4-2 : assigner la valeur entière de la variable
`x` à la `y`</span>

Nous pouvons deviner ce qui va probablement se passer grâce à notre expérience
avec d'autres langages : “Assigner la valeur `5` à `x`; ensuite faire une copie
de la valeur de `x` et l'assigner à `y`.” Nous avons maintenant deux variables,
`x` et `y`, et toutes les deux sont égales à `5`. C'est effectivement ce qui se
passe car les entiers sont des valeurs simples avec une taille connue et fixée,
et ces deux valeurs `5` sont stockées sur la Stack.

Maintenant, regardons une nouvelle version avec `String` :

```rust
let s1 = String::from("hello");
let s2 = s1;
```

Cela ressemble beaucoup au code précédent, donc nous allons supposer que cela
fonctionne pareil que précédemment : ainsi, la seconde ligne va faire une copie
de la valeur dans `s1` et l'assigner à `s2`. Mais ce n'est pas tout à fait ce
qu'il se passe.

Pour expliquer cela plus précisémment, regardons à quoi ressemble `String` avec
l'illustration 4-1. Un `String` est constitué de trois éléments, présentes sur
la gauche : un pointeur vers la mémoire qui contient le contennu de la chaine
de caractères, une taille, et une capacité. Ces données sont stockées sur la
Stack. Sur la droite est la mémoire sur la Heap qui contient les données.

<img alt="String dans la mémoire" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Illustration 4-1 : représentation d'un `String` dans la
mémoire qui contient la valeur `"hello"` assignée à `s1`.</span>

La taille est combien de mémoire, en octets, le contennu du `String` utilise
actuellement. La capacité est la quantité totale de mémoire, en octets, que
`String` a reçu du système d'exploitation. La différence entre la taille et la
capacité est importante, mais pas dans ce contexte, donc pour l'instant, ce
n'est pas grave de mettre de coté la capacité.

Quand nous assignons `s1` à `s2`, les données de `String` sont copiées, ce qui
veut dire que nous copions le pointeur, la taille, et la capacité qu'il y a sur
la Stack. Mais nous ne copions pas les données sur la Heap que dont le pointeur
fait référence. Autrement dit, la représentation des données dans la mémoire
ressemble à l'illustration 4-2.

<img alt="s1 et s2 pointent sur la même valeur" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Illustration 4-2 : représentation dans la mémoire de la
variable `s2` qui est une copie du pointeur, taille et capacité de `s1`</span>

La représentation *n'est pas* comme l'illustration 4-3, qui serait la mémoire
si Rust avait aussi copié les données sur la Heap. Si Rust fait cela,
l'opération `s2 = s1` pourrait potentiellement être coûteuse en terme de
performances d'exécution si les données sur la Heap étaient grosses.

<img alt="s1 et s2 dans deux emplacements" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Illustration 4-3 : une autre possiblité de ce que
pourrait faire `s2 = s1` si Rust copiait aussi les données sur la Heap</span>

Auparavent, nous avons que quand une variable sort de la portée, Rust appelait
automatiquement la fonction `drop` et nettoyait la mémoire sur la Heap
concernant cette variable. Mais l'illustration 4-2 montre que les deux
pointeurs de données pointaient au même endroit. C'est un problème : quand
`s2` et `s1` sortent de la portée, elles vont essayer toutes les deux de
libérer la même mémoire. C'est ce qu'on appelle une erreur de *double
libération* et c'est un des bogues de sécurité de la mémoire que nous avons
mentionné précédemment. Libérer la mémoire deux fois peut mener à des
corruptions de mémoire, qui peut potentiellement mener à des vulnérabilités
de sécurité.

Pour garantir la sécurité de la mémoire, il y a un autre détail en plus qui se
passe dans cette situation avec Rust. Plutôt qu'essayer de copier la mémoire
allouée, Rust considère que `s1` n'est plus en vigueur et du fait, Rust n'a pas
besoin de libérer quoi que ce soit lorsque `s1` sort de la portée. Regardes ce
qu'il se passe quand vous essayez d'utiliser `s1` après que `s2` soit créé,
cela ne vas pas fonctionner :

```rust,ignore
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

Vous allez avoir une erreur comme celle-ci car Rust vous défends d'utiliser la
référence qui n'est plus en vigueur :

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

Si vous avez déjà entendu parler de “copie de surface” et de “copie en
profondeur” en utilisant d'aitres langages, l'idée de copier le pointeur, la
taille et la capacité sans copier les données peut vous faire penser à de la
copie de surface. Mais parce que Rust invalide aussi la premère variable, au
lieu d'appeller cela une copie de surface, on appelle cela un *déplacement*.
Ici nous pourions lire ainsi en disant que `s1` a été *déplacé* dans `s2`. Donc
ce qui se passe réellement est montré dans l'illustration 4-4.

<img alt="s1 est déplacé dans s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Illustration 4-4 : Représentation de la mémoire après que
`s1` a été invalidé</span>

Cela résout notre problème ! Avec seuelement `s2` en vigueur, quand elle
sortira de la portée, elle veule va libérer la mémoire, et c'est tout.

De plus, cela signifie qu'il y a eu un choix de conception : Rust ne va jamais
créer des copies “profondes” de vos données. Par conséquent, toute copie
*automatique* peut être considérée comme peu coûteuse en termes de performances
d'exécution.

#### Les interactions entre les variables et les données : le clonage

Si nous *voulons* faire une copie profonde des données sur la Heap d'un
`String`, et pas seulement des données sur la Stack, nous povons utiliser une
méthode courante qio s'appelle `clone`. Nous allons aborderons la syntaxe des
méthodes dans le chapitre 5, mais parce que les méthodes sont un outil courant
dans de nombreux langages, vous les avez probablement utilisé auparavent.

Voici un exemple d'utilisation de la méthode `clone` :

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

Cela fonctionne très bien et c'est ainsi que vous pouvez reproduire le
comportement décrit dans l'illustration 4-3, dans les données de la Heap sont
copiées.

Quand vous voyez un appel à `clone`, vous savez que du code arbitraire est
exécuté et que ce code peut coûteux. C'est un indicateur visuel qu'il se passe
quelque chose de différent.

#### Données seulement sur la Stack : les copier

Il y a une autre faille qu'on a pas encore parlé. Le code suivant utilise des
entiers, dont une partie a déjà été montré dans l'entrée 4-2, ils fonctionne et
est correct :

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

Mais ce code semble contredire ce que nous venons d'apprendre : nous n'avons
pas appellé un `clone`, mais `x` est toujours en vigueur et n'a pas été déplacé
dans `y`.

La raison est que ces types tels que les entriers ont une taille connue au
moment de la compilation et sont entierement stockées sur la Stack, donc copie
les valeurs actuelles, ce qui est rapide à faire. Cela signifie qu'il n'y a pas
de raison que nous voudrions empêcher `x` d'être toujours en vigueur après
avoir créé la variable `y`. En d'autres termes, ici il n'y a pas de différence
entre la copie en surface et profonde, donc appeller `clone` ne ferait rien de
différent que la copie en surface habituelle et nous pouvons l'exclure.

Rust a une annotation spécial appelée le trait `Copy` que nous pouvons utiliser
sur des types comme les entiers qui sont stockés sur la Stack (nous verrons les
traits dans le chapitre 10). Si un type a le trait `Copy`, l'ancienne variable
est toujours utilisable après son affectation. Rust ne pas nous autoriser à
annoter type avec le trait `Copy` si ce type, ou un de ses éléments, a
implémenté le trait `Drop`. Si ce type a besoin que quelque chose de spécial se
passe quand la valeur sort de la portée et que nous ajoutons l'annotation
`Copy` sur ce type, nous allons avoir une erreur au moment de la compilation.
Pour en savoir plus sur comment ajouter l'annotation `Copy` sur votre type,
reférez-vous à l'annexe C sur les trraits dérivés.

Donc, quel sont les types qui ont `Copy` ? Vous pouvez regarder dans la
documentation pour un type donné pour vous en assurer, mais de manière
générale, tout groupe de simple scalaire peut être `Copy`, (TODO) and nothing that requires allocation or is some form of resource is `Copy`.
Voici quelques types qui sont `Copy` :

* Tous les types d'entiers, comme `u32`.
* Le type booléen, `bool`, avec les valeurs `true` et `false`.
* Le type de caractères, `char`.
* Tous les types de nombres à virgule, comme `f64`.
* Les Tuples, mais uniquement s'ils contiennent des types qui sont aussi
`Copy`. Le `(i32, i32)` est `Copy`, mais `(i32, String)` ne l'est pas.

### L'appropriation et les fonctions

La syntaxe pour passer une valeur à une fonction est similaire à celle pour
assigner une valeur à une variable. Passer une variable à une fonction va la
déplacer ou la copier, comme l'assignation. L'entrée 4-3 est un exemple avec
quelques commentaires qui montrent où les variables rentrent et sortent de la
portée :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
fn main() { 
    let s = String::from("hello");  // s rentre dans la portée.

    takes_ownership(s);             // La valeur de s est déplacée dans la fonction ...
                                    // ... et n'est plus en vigueur à partir d'ici

    let x = 5;                      // x rentre dans la portée.

    makes_copy(x);                  // x va être déplacée dans la fonction,
                                    // mais i32 implémente Copy, donc on peux
                                    // utiliser x ensuite.

} // Ici, x sort de la portée, puis ensuite s. Mais puisque la valeur de s a
  // été déplacée, il ne se passe rien de spécial.
 

fn takes_ownership(some_string: String) { // some_string rentre dans la portée.
    println!("{}", some_string);
} // Ici, some_string sort de la portée et `drop` est appellé. La mémoire est
  // libérée.

fn makes_copy(some_integer: i32) { // some_integer rentre dans la portée.
    println!("{}", some_integer);
} // Ici, some_integer sort de la portée. Il ne se passe rien de spécial.
```

<span class="caption">Entrée 4-3 : les fonction avec les appartenances et les
portées qui sont commentées</span>

Si nous essayons d'utiliser `s` après l'appel à `takes_ownership`, Rust va
lever et retourner un erreur au moment de la compilation. Ces vérifications
statiques nous protègent des erreurs. Essayez d'ajouter du code au `main` qui
utilise `s` et `x` pour voir quand vous pouvez les utiliser et quand les règles
de l'appartenance vous empêchent de le faire.

### Les valeurs de retour et la portée

Renvoyer des valeurs peut aussi transférer leur appartenance. Voici un exemple
avec des annotations similaires à celles de l'entrée 4-3 :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership déplace sa valeur de
                                        // retour dans s1

    let s2 = String::from("hello");     // s2 rentre dans la portée

    let s3 = takes_and_gives_back(s2);  // s2 est déplacée dans
                                        // takes_and_gives_back, qui elle aussi
                                        // déplace sa valeur de retour dans s3.
} // Ici, s3 sort de la portée et est éliminée. s2 sort de la portée mais a été
  // déplacée donc il ne se passe rien. s1 sort aussi de la portée et est
  // éliminée.

fn gives_ownership() -> String {             // gives_ownership va déplacer sa
                                             // valeur de retour dans la 
                                             // fonction qui l'appelle.

    let some_string = String::from("hello"); // some_string rentre dans la
                                             //portée.

    some_string                              // some_string est retrournée et
                                             // est déplacée au code qui
                                             // l'appelle.
}

// takes_and_gives_back va prendre un String et retourne aussi un String.
fn takes_and_gives_back(a_string: String) -> String { // a_string rentre dans
                                                      // la portée.

    a_string  // a_string est retournée et déplacée au code qui l'appelle.
}
```

L'appartenance d'une variable suit toujours le même processus à chaque fois :
assigner une valeur à une variable la déplace. Quand une variable qui contient
des données sur la Heap sort de la portée, la valeur va être nettoyée de la
mémoire avec `drop` à moins que la donnée ait été déplacée pour appartenir à
une autre variable.

Il est un peu fastidieux de prendre l'appartenance puis ensuite de retourner
l'appartenance avec chaque fonction. Et qu'est ce qu'il se passe si nous
voulons qu'une fonction utilise une valeur mais ne se l'approprie pas ? C'est
assez pénible que tout ce que nous envoyons doit être retourné si nous voulons
l'utiliser à nouveau, en plus de toutes les données qui découlent de
l'exécution de la fonction que nous voulons aussi récupérer.

Il est possible de retourner plusieurs valeurs en utilisant un tuple, comme
ceci :

<span class="filename">Nom du fichier : src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() renvoie le nombre de caractères d'un String.

    (s, length)
}
```

Mais c'est trop de cérémonies et beaucoup de travail pour un principe qui
devrait être courrant. Heureusement pour nous, Rust a une fonctionnalité pour
ce principe, et c'est ce qu'on appelle les *références*.
