## Les *Closures*: des fonctions anonymes qui capturent leur environnement


Les *closures* en Rust sont des fonctions anonymes qui peuvent être sauvegardés dans une variable ou qui peuvent être passées en argument à d'autres fonctions. Il est possible de créer une *closure* à un endroit du code et ensuite de l'appeler dans un contexte différent pour l'évaluer. Contrairement aux fonctions, les *closures* ont la possibilité de capturer les valeurs présentes dans le contexte où elles sont appelées. Nous allons montrer comment les caractéristiques des *closures* permet de faire de la réutilisation de code et des comportements personnalisés.



### Création d'une Abstraction de Comportement à l'aide d'une *Closure*

Travaillons sur un exemple d'une situation où il est utile de stocker une *closure* qui s'exécutera ultérieurement. Nous allons parler de la syntaxe des *closures*, de l'inférence de type, et des traits au cours de ce chapitre.

Considérez cette situation hypothétique: nous travaillons au démarrage d'une application pour générer des plans d'entraînement personnalisés. Le backend est écrit en Rust et l'algorithme qui génère les exercices prend en compte beaucoup de facteurs comme l'age de l'utilisateur, son indice de masse corporelle, ses préférences et une intensité paramétrable par l'utilisateur. L'algorithme réellement utilisé n'est pas important pour cet exemple: ce qui est important est que le calcul prenne plusieurs secondes. Nous voulons appeler l'algorithme uniquement quand nous avons besoin, et seulement une fois, afin que l'utilisateur n'est pas à attendre plus que nécessaire.

Nous allons, pour simuler l'appel à cet algorithme hypothétique, utiliser la fonction `simulated_expensive_calculation` montré dans le Listing 13-1, qui affichera `calculating slowly...`, attend 2 secondes, et ensuite retourne le nombre qui lui a été passé :

<span class="filename">Fichier: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn simulated_expensive_calculation(intensity: i32) -> i32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    intensity
}
```

<span class="caption">Listing 13-1: Une fonction pour remplacer un calcul hypothétique qui prend environ deux secondes à exécuter</span>

Ensuite, vient la fonction `main` qui contient les parties, de l'application d'entraînement, importantes pour cet exemple. Cette fonction représente le code que l'application appellera quand un utilisateur demande un plan d'entraînement. Parce que l'interaction avec le frontend de l'application n'est pas pertinente à l'utilisation des *closures*, nous allons coder en dur les valeurs représentant les entrées de notre programme et afficher les résultats.


 Les paramètres d'entrées requis sont:

- Un nombre `intensité` de l'utilisateur, spécifié quand ils demandent un entraînement, afin qu'ils puissent indiquer s'ils veulent un entraînement de faible ou de haute intensité.

- Un nombre aléatoire qui produira une certaine variété dans les plans d'entraînement

Le résultat imprimé par le programme sera le plan d'entraînement recommandé.

Le résultat sera le plan d'entraînement recommandé. Le Listing 13-2 montre la fonction `main` que nous allons utiliser :


<span class="filename">Fichier: src/main.rs</span>

```rust
fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(simulated_user_specified_value, simulated_random_number);
}
# fn generate_workout(intensity: i32, random_number: i32) {}
```

<span class="caption">Listing 13-2: Une fonction `main` avec des valeurs codées en dur pour simuler l'entrée d'un utilisateur et la génération de nombres aléatoires</span>

Nous avons codé en dur la variable `simulated_user_specified_value` à 10 et la variable `simulated_random_number` à 7 pour des raisons de simplicité ; dans un programme réel, nous obtiendrions le nombre d'intensité à partir du frontend de l'application et nous utiliserions la crate `rand` pour générer un nombre aléatoire, comme nous l'avons fait dans l'exemple du Guessing Game dans le chapitre 2. La fonction `main` appelle une fonction `generate_workout` avec les valeurs d'entrée simulées.

Maintenant que nous avons le contexte, passons à l'algorithme. La fonction `generate_workout` dans le Listing 13-3 contient la logique métier de l'application qui nous préoccupe le plus dans cet exemple. Le reste des changements de code dans cet exemple sera apporté à cette fonction :

<span class="filename">Fichier: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# fn simulated_expensive_calculation(num: i32) -> i32 {
#     println!("calculating slowly...");
#     thread::sleep(Duration::from_secs(2));
#     num
# }
#
fn generate_workout(intensity: i32, random_number: i32) {
    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            simulated_expensive_calculation(intensity)
        );
        println!(
            "Next, do {} situps!",
            simulated_expensive_calculation(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                simulated_expensive_calculation(intensity)
            )
        }
    }
}
```

<span class="caption">Listing 13-3: La logique de gestion du programme qui imprime les plans d'entraînement basés sur les entrées et les appels à la fonction `simulated_expensive_calculation`.</span>

Le code dans le Listing 13-3 a plusieurs appels à la fonction de calcul lent: le premier bloc `if` appelle `simulated_expensive_calculation` deux fois, le `if`à l'intérieur de l'`else` extérieur ne l'appelle pas du tout, et le code à l'intérieur du second `else` cas l'appelle une fois.

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

Le comportement souhaité de la fonction `generate_workout` est de vérifier d'abord si l'utilisateur veut un entraînement de faible intensité (indiqué par un nombre inférieur à 25) ou un entraînement de haute intensité (un nombre de 25 ou plus).

Les plans d'entraînement à faible intensité recommanderont un certain nombre de pompes et d'abdominaux basés sur l'algorithme complexe que nous simulons.

Si l'utilisateur souhaite un entraînement de haute intensité, il y a une logique supplémentaire: si la valeur du nombre aléatoire généré par l'application est 3, l'application recommandera une pause et une hydratation à la place. Sinon, l'utilisateur recevra un nombre de minutes de course qui provient de l'algorithme complexe.


L'équipe de data science nous a fait savoir qu'il va y avoir des changements dans la façon dont nous devrons appeler l'algorithme à l'avenir. Pour simplifier la mise à jour lorsque ces changements se produisent, nous voulons refactorer ce code pour qu'il n'appelle la fonction `simulated_expensive_calculation` qu'une seule fois. Nous voulons également nous débarrasser de l'endroit où nous appelons actuellement la fonction deux fois inutilement, sans ajouter d'autres appels à cette fonction au cours de ce processus. C'est-à-dire, nous ne voulons pas l'appeler si le résultat n'est pas nécessaire, et nous voulons quand même l'appeler une seule fois.

Nous pourrions restructurer le programme d'entraînement de plusieurs façons. Tout d'abord, nous allons essayer d'extraire l'appel dupliqué à la fonction `expensive_calculation` dans une variable, comme le montre le Listing 13-4 :

<span class="filename">Fichier: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# fn simulated_expensive_calculation(num: i32) -> i32 {
#     println!("calculating slowly...");
#     thread::sleep(Duration::from_secs(2));
#     num
# }
#
fn generate_workout(intensity: i32, random_number: i32) {
    let expensive_result =
        simulated_expensive_calculation(intensity);

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result
        );
        println!(
            "Next, do {} situps!",
            expensive_result
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result
            )
        }
    }
}
```

<span class="caption">Listing 13-4: Extraction des appels à `simulated_expensive_calculation` à un seul endroit avant les blocs `if` et stockage du résultat dans la variable `expensive_result`.</span>

Ce changement unifie tous les appels à `simulated_expensive_calculation` et résout le problème du premier bloc `if` qui appelle la fonction deux fois inutilement. Malheureusement, nous appelons maintenant cette fonction et attendons le résultat dans tous les cas, ce qui inclut le bloc `if` interne qui n'utilise pas du tout la valeur du résultat.

Nous voulons définir le code à un seul endroit dans notre programme, mais seulement *exécuter* ce code-là où nous avons réellement besoin du résultat. C'est un cas d'utilisation pour les *closures*!

### Les Closures stockent du code pour une exécution ultérieure

Au lieu d'appeler toujours la fonction `simulated_expensive_calculation` avant les blocs `if`, nous pouvons définir une closure et enregistrer la closure dans une variable au lieu du résultat comme le montre le Listing 13-5. Nous pouvons en fait choisir de déplacer l'ensemble du corps de `simulated_expensive_calculation` dans la closure que nous introduisons ici :

<span class="filename">Fichier: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
# expensive_closure(5);
```

<span class="caption">Listing 13-5: Définition d'une *closure* et son enregistrement dans la variable `expensive_closure`.</span>

La définition de la *closure* vient après le `=` pour l'assigner à la variable `expensive_closure`. Pour définir une fermeture, on commence par une paire de tubes verticaux (`|`), à l'intérieur desquels on spécifie les paramètres à la fermeture; cette syntaxe a été choisie en raison de sa similitude avec les définitions de *closure* en Smalltalk et en Ruby. Cette closure a un paramètre appelé `num` : si nous avions plus d'un paramètre, nous les séparerions par des virgules, comme `|param1, param2|`.

Après les paramètres, on met des accolades qui contiennent le corps de la closure, celles-ci sont facultatives si le corps de la *closure* est une expression unique. Après les accolades, nous avons besoin d'un point-virgule pour terminer la déclaration commencée avec `let`. La valeur à la dernière ligne dans le corps de la closure (`num`), comme cette ligne ne se termine pas par un point-virgule, sera la valeur renvoyée par la closure lorsqu'elle est appelée, exactement  comme dans les corps de fonction.

Notez que cette instruction `let` signifie que la variable `expensive_closure` contient la *définition* d'une fonction anonyme, pas la valeur *résultant* de l'appel à cette fonction anonyme. Rappelons la raison pour laquelle nous utilisons une *closure* est parce que nous voulons définir le code à appeler à un point, stocker ce code, et l'appeler à un point ultérieur; le code que nous voulons appeler est maintenant stocké dans `expensive_closure`.

Maintenant que nous avons défini la *closure*, nous pouvons changer le code dans les blocs `if` pour appeler la closure afin d'exécuter le code et obtenir la valeur résultante. L'appel d'une closure est comme l'appel d'une fonction ; nous spécifions le nom de la variable qui détient la définition de la closure et la complétons avec des parenthèses contenant les valeurs du ou des arguments que nous voulons utiliser pour cet appel comme indiqué dans le Listing 13-6 :

<span class="filename">Fichier: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
fn generate_workout(intensity: i32, random_number: i32) {
    let expensive_closure = |num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_closure(intensity)
        );
        println!(
            "Next, do {} situps!",
            expensive_closure(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_closure(intensity)
            )
        }
    }
}
```

<span class="caption">Listing 13-6: Appel de la *closure* `expensive_closure` que nous avons défini</span>

Maintenant, le calcul coûteux n'est appelé qu'à un seul endroit, et nous n'exécutons ce code que là où nous avons besoin des résultats.

Cependant, nous avons réintroduit l'un des problèmes du Listing 13-3: nous continuons d'appeler la *closure* deux fois dans le premier bloc `if`, qui appellera le code coûteux deux fois et fera attendre l'utilisateur deux fois plus longtemps que nécessaire. Nous pourrions résoudre ce problème en créant une variable locale à ce bloc  `if` pour conserver le résultat de l'appel à la *closure*, mais les *closures* nous fournissent une autre solution. Mais commençons d'abord par expliquer pourquoi il n'y a pas d'annotation de type dans la définition des *closures* et les traits impliqués dans les *closures*.

### Closure: Inférence de Type et Annotations


Les *closures* ne nécessitent pas d'annoter le type des paramètres ou la valeur de retour comme le font les fonctions `fn`. Les annotations de type sont nécessaires pour les fonctions, car elles font partie d'une interface explicite exposée à vos utilisateurs. Définir cette interface de manière rigide est important pour s'assurer que tout le monde s'accorde sur les types de valeurs qu'une fonction utilise et retourne. Mais les *closures* ne sont pas utilisées dans une interface exposée comme cela: elles sont stockées dans des variables et utilisées sans les nommer ni les exposer aux utilisateurs de notre bibliothèque.



En outre, les *closures* sont généralement brèves et ne sont pertinentes que dans un contexte plutôt restreint que dans un scénario arbitraire. Dans ce contexte limité, le compilateur est capable d'inférer le type des paramètres et le type de retour, comme il est capable d'inférer le type de la plupart des variables.

Faire annoter par les programmeurs le type de ces petites fonctions anonymes serait agaçant et largement redondant avec l'information dont dispose déjà le compilateur.

Comme les variables, nous pouvons ajouter des annotations de type si nous voulons expliciter et augmenter la clarté  du code au risque d'être plus verbeux que ce qui est strictement nécessaire; annoter les types pour la closure que nous avons définie dans le Listing 13-4 ressemblerait à la définition présentée dans le Listing 13-7 :

<span class="filename">Fichier: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num: u32| -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

<span class="caption">Listing 13-7: Ajout d'annotations de type optionnel sur les paramètres et les valeurs de retour dans la closure</span>

La syntaxe des *closures* et des fonctions semble plus similaire avec les annotations de type. Ce qui suit est une comparaison verticale de la syntaxe pour la définition d'une fonction qui ajoute une fonction à son paramètre, et une *closures* qui a le même comportement. Nous avons ajouté des espaces pour aligner les parties pertinentes. Ceci illustre comment la syntaxe des *closures* est similaire à la syntaxe des fonctions, sauf pour l'utilisation des tubes verticaux et la quantité de syntaxe facultative :


```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

La première ligne affiche une définition de fonction et la deuxième ligne une définition de closure entièrement annotée. La troisième ligne supprime les annotations de type de la définition de la closure, et la quatrième ligne supprime les crochets qui sont facultatifs, parce que le corps d'une closure n' a qu'une seule expression. Ce sont toutes des définitions valides qui produiront le même comportement quand on les appelle.

Les définitions de *closures* auront un type concret déduit pour chacun de leurs paramètres et pour leur valeur de retour. Par exemple, le Listing 13-8 montre la définition d'une petite closure qui renvoie simplement la valeur qu'elle reçoit comme paramètre. Cette closure n'est pas très utile sauf pour les besoins de cet exemple. Notez que nous n'avons pas ajouté d'annotations de type à la définition: si nous essayons alors d'appeler la closure deux fois, en utilisant une `String` comme argument la première fois et un `u32` la deuxième fois, nous obtiendrons une erreur :

<span class="filename">Fichier: src/main.rs</span>

```rust,ignore
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

<span class="caption">Listing 13-8: Tentative d'appeler une closure dont les types sont inférés avec deux types différents</span>

Le compilateur nous renvoie l'erreur suivante :

```text
error[E0308]: mismatched types
 --> src/main.rs
  |
  | let n = example_closure(5);
  |                         ^ expected struct `std::string::String`, found
  integral variable
  |
  = note: expected type `std::string::String`
             found type `{integer}`
```

La première fois que nous appelons `example_closure` avec une `String`, le compilateur infère le type de `x` et le type de retour de la closure comme étant de type `String`. Ces types sont ensuite verrouillés dans `example_closure`, et nous obtenons une erreur de type si nous essayons d'utiliser un type différent avec la même closure.

### Stockage de Closures avec des paramètres génériques et le Trait `Fn`

Revenons à notre application de génération d'entraînement. Dans le Listing 13-6, notre code appelait toujours la closure de calcul coûteux plus de fois qu'il n'en avait besoin. Une option pour résoudre ce problème est de sauvegarder le résultat de la closure coûteuse dans une variable pour une future utilisation et d'utiliser la variable à  chaque endroit où nous en avons besoin au lieu de rappeler la closure plusieurs fois. Cependant, cette méthode pourrait donner lieu à un code très redondant.

Heureusement, une autre solution s'offre à nous. Nous pouvons créer une struct qui sauvegardera la closure et la valeur qui en résulte. La structure n'exécutera la closure que si nous avons besoin de la valeur résultante, et elle cachera la valeur résultante pour que le reste de notre code n'ait pas à être responsable de la sauvegarde et de la réutilisation du résultat. Vous connaissez peut-être ce pattern sous le nom de *mémoization* ou *évaluation larmoyante*.

Pour faire qu'une struct détiennne une closure, il faut spécifier le type de closure, car une définition de structure a besoin de connaître les types de chacun de ses champs. Chaque instance de closure a son propre type anonyme unique: c'est-à-dire que même si deux *closures* ont la même signature, leurs types sont toujours considérés comme différents. Pour définir des structs, des enums ou des paramètres de fonction qui utilisent les *closures*, nous utilisons des génériques et des limites de "traits", comme nous l'avons vu au chapitre 10.

Les traits `Fn` sont fournis par la bibliothèque standard. Toutes les *closures* implémentent un des traits suivants : `Fn`, `FnMut`, ou `FnOnce`. Nous discuterons de la différence entre ces traits dans la prochaine section sur la capture de l'environnement ; dans cet exemple, nous pouvons utiliser le caractère `Fn`.

Nous ajoutons des types au trait `Fn` pour représenter les types de paramètres et les valeurs de retour que les *closures* doivent avoir pour correspondre à ce trait lié. Dans ce cas, notre closure a un paramètre de type `u32` et renvoie un `u32`, donc le trait lié que nous spécifions est `Fn (u32) -> u32`.

Le Listing 13-9 montre la définition de la struct`Cacher` qui possède une closure et une valeur de résultat optionnelle :

<span class="filename">Filename: src/main.rs</span>

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}
```

<span class="caption">Listing 13-9: Définition d'une struct `Cacher` qui possède une closure dans `calculation` et un résultat facultatif dans `value`.</span>

La struct `Cacher` a un champ `calculation` du type générique `T`. Les limites du trait sur `T` spécifient que c'est une closure en utilisant le trait `Fn`. Toute closure que l'on veut stocker dans le champ `calculation` doit avoir un paramètre `u32` (spécifié entre parenthèse après `->`) et doit retourner un `u32` (spécifié après `->`).

> Remarque : Les fonctions implémentent aussi les trois traits `Fn`. Si ce que nous voulons faire n' a pas besoin de capturer une
> valeur de l'environnement, nous pouvons utiliser une fonction plutôt qu'une closure où nous avons besoin de quelque chose qui
> implémente un trait `Fn`.


Avant d'exécuter la fermeture, `value` sera initialisée à `None`. Lorsque ducode utilisant un `Cacher` demande le *result* de la closure, le `Cacher` exécutera la closureà ce moment-là et stockera le résultat dans une variante `Some` dans le champ `value`. Ensuite, si le code demande à nouveau le résultat de la closure, au lieu d'exécuter à nouveau la closure, le `Cacher` renverra le résultat contenu dans la variante `Some`.

La logique autour du champ `value` que nous venons de décrire est définie dans le Listing 13-10 :

<span class="filename">Fichier: src/main.rs</span>

```rust
# struct Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     calculation: T,
#     value: Option<u32>,
# }
#
impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```

<span class="caption">Listing 13-10: La logique de caching de `Cacher`</span>

Nous voulons que `Cacher` gère les valeurs des champs de structure plutôt que de laisser le code appelant, la possibilité de  modifier les valeurs dans ces champs directement, donc nous laissons ces champs privés.

La fonction `Cacher::new` prend un paramètre générique `T`, que nous avons défini comme ayant le même caractère lié que la struct `Cacher`. Puis `Cacher::new` renvoie une instance `Cacher` qui contient la closure spécifiée dans le champ `calculation` et une valeur `None` dans le champ `value`, parce que nous n'avons pas encore exécuté la closure.

Lorsque le code appelant veut le résultat de l'évaluation de la closure, au lieu d'appeler directement la clôture, il appellera la méthode `value`. Cette méthode vérifie si nous avons déjà une valeur  dans `self.value` dans un `Some`; si tel est le cas, elle renvoie la valeur contenue dans le `Some` sans exécuter de nouveau la closure.

Si `self.value` est à `None`, nous appelons la closure stockée dans `self.calculation`, sauvegardons le résultat dans `self. value` pour une utilisation future, et retournons la valeur après calcul.

Le Listing 13-11 montre comment utiliser cette struct `Cacher` dans la fonction `generate_workout` du Listing 13-6 :

<span class="filename">Fichier: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# struct Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     calculation: T,
#     value: Option<u32>,
# }
#
# impl<T> Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     fn new(calculation: T) -> Cacher<T> {
#         Cacher {
#             calculation,
#             value: None,
#         }
#     }
#
#     fn value(&mut self, arg: u32) -> u32 {
#         match self.value {
#             Some(v) => v,
#             None => {
#                 let v = (self.calculation)(arg);
#                 self.value = Some(v);
#                 v
#             },
#         }
#     }
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    });

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result.value(intensity)
        );
        println!(
            "Next, do {} situps!",
            expensive_result.value(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result.value(intensity)
            );
        }
    }
}
```

<span class="caption">Listing 13-11: Utilisation de `Cacher` dans la fonction `generate_workout` pour rendre abstrait la logique de caching.</span>

Au lieu de sauvegarder la closure dans une variable directement, nous sauvegardons une nouvelle instance de `Cacher` qui contient la closure. Ensuite, à chaque fois que nous voulons le résultat, nous appelons la méthode `value` sur l'instance de `Cacher`. Nous pouvons appeler la méthode `value` autant de fois que nous voulons, ou ne pas l'appeler du tout, et le calcul coûteux sera exécuté un maximum, une fois.

Essayez d'exécuter ce programme avec la fonction `main` du Listing 13-2. Modifiez les valeurs des variables `simulated_user_specified_value` et `simulated_random_number` pour vérifier que dans tous les cas, dans les différents blocs `if` et `else`, `calculating slowly...` n'apparaît qu'une seule fois et seulement si nécessaire. Le `Cacher` prend soin de la logique nécessaire pour s'assurer que nous n'appelons pas le calcul coûteux plus que nous n'en avons besoin, ainsi `generate_workout` peut se concentrer sur la logique business.

### Limitations de l'implémentation de `Cacher`

Cacher des valeurs est un comportement généralement utile que nous pourrions vouloir utiliser dans d'autres parties de notre code avec différentes *closures*. Cependant, il y a deux problèmes avec la mise en oeuvre actuelle de `Cacher` qui rendraient difficile sa réutilisation dans des contextes différents.

Le premier problème est qu'une instance de `Cacher` suppose qu'elle obtiennne toujours la même valeur pour le paramètre `arg` à la méthode `value`. Autrement dit, ce test sur `Cacher` échouera:

```rust,ignore
#[test]
fn call_with_different_values() {
    let mut c = Cacher::new(|a| a);

    let v1 = c.value(1);
    let v2 = c.value(2);

    assert_eq!(v2, 2);
}
```

Ce test créé une nouvelle instance de `Cacher` avec une closure qui retourne la valeur qui lui est passée. Nous appelons la méthode `value` sur cette instance de `Cacher` avec une valeur `arg` de 1 et ensuite une valeur `arg` de 2, et nous nous attendons à ce que l'appel à `value` avec la valeur `arg` de 2 devrait retourner 2.

Exécutez ce test avec l'implémentation de `Cacher` du Listing 13-9 et du Listing 13-10, et le test échouera sur le `assert_eq! ` avec ce message:

```text
thread 'call_with_different_values' panicked at 'assertion failed: `(left == right)`
  left: `1`,
 right: `2`', src/main.rs
```

Le problème est que la première fois que nous avons appelé `c. value` avec 1, l'instance `Cacher` a sauvé `Some(1)` dans `self.value`. Par la suite, peu importe ce que nous passons à la méthode `value`, elle retournera toujours 1.

Essayez de modifier `Cacher` pour tenir une hashmap plutôt qu'une seule valeur. Les clés de la hashmap seront les valeurs `arg` qui lui sont passées, et les valeurs de la hashmap  seront le résultat de l'appel de la fermeture sur cette clé. Plutôt que de regarder si `self.value` a directement une valeur `Some` ou une valeur `None`, la fonction `value` recherchera `arg` dans la hashmap et retournera la valeur si elle est présente. S'il n'est pas présent, le `Cacher` appellera la closure et sauvegardera la valeur résultante dans la hashmap  associée à sa valeur `arg`.

Le second problème avec l'implémentation actuelle de `Cacher` est qu'il n'accepte que les *closures* qui prennent un paramètre de type `u32` et renvoient un `u32`. Nous pourrions vouloir mettre en cache les résultats des fermetures qui prennent une slice d'une string et renvoient des valeurs `usize`, par exemple. Pour corriger ce problème, essayez d'introduire des paramètres plus génériques pour augmenter la flexibilité de la fonctionnalité que possède `Cacher`.

### Capturer l'environnement avec des Closures

Dans l'exemple du générateur d'entraînement, nous n'avons utilisé les *closures* que comme des fonctions anonymes "inline". Cependant, les *closures* ont une capacité supplémentaire que les fonctions n'ont pas : elles peuvent capturer leur environnement et accéder aux variables à partir du scope dans lequel elles sont définies.

Le Listing 13-12 a un exemple de closure stockée dans la variable `equal_to_x` qui utilise la variable `x` de l'environnement environnant de la closure :

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```

<span class="caption">Listing 13-12: Exemple d'une closure qui réfère à une variable du scope la contenant.</span>

Ici, même si `x` n'est pas un des paramètres de `equal_to_x`, la closure `equal_to_x` est autorisée à utiliser la variable `x` définie dans la même portée (ou environnement) que `equal_to_x`.

Nous ne pouvons pas faire la même chose avec les fonctions; si nous essayons avec l'exemple suivant, notre code ne compilera pas :

<span class="filename">Fichier: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 4;

    fn equal_to_x(z: i32) -> bool { z == x }

    let y = 4;

    assert!(equal_to_x(y));
}
```

Nous obtenons l'erreur suivante:

```text
error[E0434]: can't capture dynamic environment in a fn item; use the || { ...
} closure form instead
 --> src/main.rs
  |
4 |     fn equal_to_x(z: i32) -> bool { z == x }
  |                                          ^
```

Le compilateur nous rappelle même que cela ne fonctionne qu'avec les *closures*!

Lorsqu'une closure capture une valeur de son environnement, elle utilise la mémoire pour stocker les valeurs à utiliser dans son corps. Cette utilisation de la mémoire a un coût supplémentaire que nous ne voulons pas payer dans les cas les plus courants où nous voulons exécuter du code qui ne capture pas leur environnement. Comme les fonctions ne sont jamais autorisées à capturer leur environnement, la définition et l'utilisation des fonctions n'occasionneront jamais cette surcharge.

Les *closures* peuvent captuer les valeurs de leur environnement de trois façons, ce qui correspond directement aux trois façons dont une fonction peut prendre un paramètre: la prise de propriété (ownership), l'emprunt immutable et l'emprunt mutable. Ceux-ci sont codés dans les trois traits `Fn` comme suit:

* `FnOnce` consomme les variables qu'il capture à partir de son scope, connu sous le nom de l'*environnement* de la closure.      Pour consommer les variables capturées, la closure doit s'approprier ces variables et les déplacer dans la closure lorsqu'elle est définie. La partie `Once` du nom représente le fait que la closure ne puisse pas prendre la propriété (ou l'ownership) des mêmes variables plus d'une fois, donc elle ne peut être appelée qu'une seule fois.
* `Fn` emprunte des valeurs à l'environnement de façon immutable.
* `FnMut` peut changer l'environnement parce qu'il emprunte des valeurs de manière mutable.

Lorsque nous créons une closure, Rust déduit quel caractère utiliser en se basant sur la façon dont la closure utilise les valeurs de l'environnement. Dans le Listing 13-12, la closure `equal_to_x` emprunte `x` immutablement (donc `equal_to_x` a le trait `Fn`) parce que le corps de la closure ne fait que lire la valeur de `x`.

Si nous voulons forcer la closure à s'approprier les valeurs qu'elle utilise dans l'environnement, nous pouvons utiliser le mot-clé `move` avant la liste des paramètres. Cette technique est surtout utile lorsque vous passez une fermeture à un nouveau thread pour déplacer les données afin qu'elles appartiennent au nouveau thread.

Nous aurons d'autres exemples de *closures* utilisant `move` au chapitre 16 lorsque nous parlerons du multithreading. Pour l'instant, voici le code du Listing 13-12 avec le mot-clé `move` ajouté à la définition de la closure et utilisant des vecteurs au lieu d'entiers, car les entiers peuvent être copiés plutôt que déplacés ; notez que ce code ne compilera pas encore :

<span class="filename">Fichier: src/main.rs</span>

```rust,ignore
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x;

    println!("can't use x here: {:?}", x);

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
```

Nous obtenons l'erreur suivante:

```text
error[E0382]: use of moved value: `x`
 --> src/main.rs:6:40
  |
4 |     let equal_to_x = move |z| z == x;
  |                      -------- value moved (into closure) here
5 |
6 |     println!("can't use x here: {:?}", x);
  |                                        ^ value used here after move
  |
  = note: move occurs because `x` has type `std::vec::Vec<i32>`, which does not
  implement the `Copy` trait
```

La valeur `x` est déplacée dans la closure lorsque la fermeture est définie, parce que nous avons ajouté le mot-clé `move`. La closure a alors la propriété de `x`, et `main` n'est plus autorisé à utiliser `x` dans l'instruction `println! `. Supprimer `println! ` corrigera cet exemple.

La plupart du temps, lorsque vous spécifiez l'un des traits `Fn`, vous pouvez commencer par `Fn` et le compilateur vous dira si vous avez besoin de `FnMut` ou `FnOnce` basé sur ce qui se passe dans le corps de la closure.

Pour illustrer les situations où les *closures*, capturant leur environnement, sont utiles comme paramètres de fonction, passons à notre prochain sujet: les itérateurs.
