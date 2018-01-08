## Les *Closures*: des fonctions anonymes qui capturent leur environnement

<!-- Bill's suggested we flesh out some of these subtitles, which I think we
did more with earlier chapters but we (I, included!) have got a bit lax with. I
don't think this is quite right, is there a shorter heading we could use to
capture what a closure is/is for? -->
<!-- I've attempted a more descriptive subtitle, what do you think? /Carol -->

Les *closures* en Rust sont des fonctions anonymes qui peuvent être sauvegardés dans une variable ou qui peuvent être passées en argument à d'autres fonctions. I est possible de créer une *closure* à un endroit du code et ensuite de l'appeler dans un contexte différent pour l'évaluer. Contrairement aux fonctions, les *closures* ont la possibilité de capturer les valeurs présentes dans le contexte oú elles sont appelées. Nous allons montrer comment les caractéristiques des *closures* permette de faire du recyclage de code et des comportements personnalisables.


<!-- Can you say what sets closures apart from functions, explicitly, above? I
can't see it clearly enough to be confident, after one read through this
chapter. I think it would help to have the closure definition up front, to help
to let the reader know what they are looking out for in the examples. When
would you use a closure, for example, rather than using a function? And is it
the closure that's stored in the variable, or is it the result of applying the
closure to a value passed as an argument? -->
<!-- I've tried reworking the above paragraph and restructuring the examples to
be more motivating. I've also tried to make it clear throughout that storing a
closure is storing the *unevaluated* code, and then you call the closure in
order to get the result. /Carol -->

### Création d'une Abstraction de Comportement à l'aide d'une *Closure*

Travaillons sur un exemple d'une situation oú stocker une *closure* qui s'exécutera ultérieurement s’avère utile. Nous allons parler de la syntaxe des *closures*, de l'inférence de type, et des traits au cours de ce chapitre.

La situation hypothétique dans laquelle nous nous trouvons est la suivante: nous travaillons dans une startup qui construit une app capable de générer des plans d'entraînement de musculation personnalisés. Le backend est écrit en Rust et l'algorithme qui génère les exercices prend en compte beaucoup de facteurs comme l'age de l'utilisateur, son indice de masse corporelle, ses préférences et une intensité (un integer) paramétrable par l'utilisateur. L'algorithme réellement utilisé n'est pas important pour cet exemple: ce qui est important est que le calcul prend plusieurs secondes. Nous ne voulons appeler l'algorithme uniquement quand nous avons besoin, et seulement une fois, afin que l'utilisateur n'est pas à attendre plus que nécessaire. Nous allons pour simuler l'appel à cet algorithme hypothétique utiliser la fonction `simulated_expensive_calculation` montré dans le Listing 13-1, qui écrit sur la sortie standard `calculating slowly...`, attend 2 secondes, et retourne le nombre qui lui a été passé:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn simulated_expensive_calculation(intensity: i32) -> i32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    intensity
}
```

<span class="caption">Listing 13-1: Une fonction qui mocke le calcul hypothétique prenant 2 secondes à parcourir</span>

Ensuite, nous avons une fonction `main` qui contient les parties de l'application de musculation qui
sont importantes pour cet exemple. Ceci représente le code que l'application devrait appeler
lorsqu'un utilisateur demande un plan d'entraînement. Parce que l'interaction avec le
le frontend de l'application n'est pas pertinente à l'utilisation des *closures*, nous allons hardcoder les valeurs
représentant les entrées de notre programme et écrit sur la sortie standard les résultats.


 Les entrées de notre programme sont:

- Un nombre `intensité` de l'utilisateur, spécifié quand ils demandent un entraînement, afin qu'ils puissent indiquer s'ils veulent un entraînement de faible intensité ou un entraînement de haute intensité.

- Un nombre aléatoire qui produira une certaine variété dans les plans d'entraînement

Le résultat imprimé par le programme sera le plan d'entraînement recommandé.

Le Listing 13-2 montre la fonction main que nous allons utiliser. Nous avons codé en dur la variable `simulated_user_specified_value` à 10 et la variable `simulated_random_number` à 7 pour des raisons de simplicité; dans un programme réel, nous obtiendrions le nombre d'intensité à partir du frontend de l'application et nous utiliserions la crate `rand` pour générer un nombre aléatoire comme nous l'avons fait dans l'exemple du Guessing Game dans le chapitre 2.
La fonction `main` appelle une fonction `generate_workout` avec les valeurs d'entrée simulées suivantes:


<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(simulated_user_specified_value, simulated_random_number);
}
# fn generate_workout(intensity: i32, random_number: i32) {}
```

<span class="caption">Listing 13-2: Une fonction `main` contenant des valeurs codées en dur pour simuler l'entrée d'un utilisateur et la génération aléatoire de nombres dans la fonction `generate_workout`.</span>

C'est le contexte dans lequel nous travaillons. La fonction `generate_workout` dans Listing 13-3 contient la logique métier de l'application qui nous préoccupe le plus dans cet exemple. Le reste des changements de code dans cet exemple sera apporté à cette fonction:

<span class="filename">Filename: src/main.rs</span>

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

Le code dans le Listing 13-3 a plusieurs appels à la fonction de calcul lent: le premier `if` bloc appelle `simulated_expensive_calculation` deux fois, le `if`à l'intérieur de l'`else` extérieur ne l'appelle pas du tout, et le code du `else`  à l'intérieur de l'`else` extérieur l'appelle une fois.

<!-- Will add wingdings in libreoffice /Carol -->

Le comportement souhaité de la fonction `generate_workout` est de commencer par vérifier si l'utilisateur veut un entraînement de faible intensité (indiqué par un nombre inférieur à 25) ou un entraînement de haute intensité (25 ou plus). Les plans d'entraînement à faible intensité recommanderont un certain nombre de pompes et d'abdominaux basés sur l'algorithme complexe que nous simulons avec la fonction `simulated_expensive_calculation`, qui a besoin du nombre d'intensité comme entrée.

Si l'utilisateur souhaite un entraînement de haute intensité, il y a une logique supplémentaire: si la valeur du nombre aléatoire généré par l'application est 3, l'application recommandera une pause et une hydratation à la place. Si ce n'est pas le cas, l'utilisateur recevra un entraînement de haute intensité d'un nombre de minutes de jogging qui provient de l'algorithme complexe.


L'équipe de data science nous a fait savoir qu'il va y avoir des changements dans la façon dont nous devons appeler l'algorithme, donc nous voulons refactorer ce code pour avoir un seul endroit qui appelle la fonction `simulated_expensive_calculation` pour la mettre à jour plus simplement quand ces changements se produiront. Nous voulons également nous débarrasser de l'endroit où nous appelons actuellement la fonction deux fois inutilement, et nous ne voulons pas ajouter d'autres appels à cette fonction dans ce refactoring. En d'autres termes, nous ne voulons pas l'appeler si nous sommes dans le cas où le résultat n'est pas du tout nécessaire, et nous voulons uniquement l'appeler une seule fois dans le dernier cas.

Il y a plusieurs façons de restructurer ce programme. La façon dont nous allons d'abord essayer est d'extraire l'appel dupliqué à la fonction de calcul coûteux dans une variable, comme le montre le Listing 13-4:

<span class="filename">Filename: src/main.rs</span>

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

<!-- Will add ghosting and wingdings in libreoffice /Carol -->

Ce changement unifie tous les appels à `simulated_expensive_calculation` et résout le problème du premier bloc `if` qui appelle la fonction deux fois inutilement. Malheureusement, nous appelons maintenant cette fonction et attendons le résultat dans tous les cas, ce qui inclut le bloc `if` interne qui n'utilise pas du tout la valeur du résultat.

Nous voulons être en mesure de spécifier un certain code à un endroit dans notre programme, mais alors seulement exécuter ce code si nous avons réellement besoin du résultat à un autre endroit dans notre programme. C'est un cas d'utilisation pour les *closures*!

### Les Closures stockent du code pour une exécution ultérieure

Au lieu d'appeler toujours la fonction `simulated_expensive_calculation` avant les blocs `if`, nous pouvons définir une closure et enregistrer la closure dans une variable au lieu du résultat comme le montre le Listing 13-5. Nous pouvons en fait choisir de déplacer l'ensemble du corps de `simulated_expensive_calculation` dans la closure que nous introduisons ici:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-5: Définir une closure avec le corps qui était dans la fonction coûteuse et stocker la closure dans la variable `expensive_closure`</span>

<!-- Can you elaborate on *how* to define the closure first? I've had a go here
based on what I can see but not sure it's correct. Are we saying that a closure
is function that assigned its result to a variable you can then use? -->
<!-- I've attempted to elaborate on defining a closure in the next few
paragraphs starting here, is this better? /Carol -->

La définition de la closure  est la partie qui suit le `=` que nous attribuons à la variable `expensive_closure`. Pour définir une closure, on commence par une paire de pipes verticaux (`|`). C'est à l'intérieur des pipes que nous spécifions les paramètres de la closure; cette syntaxe a été choisie en raison de sa similitude avec les définitions des closure en Smalltalk et en Ruby. Cette closure a un paramètre nommé `num`; si nous avions plus d'un paramètre, nous les séparerions par des virgules, comme `|param1, param2|`.

Après les paramètres, on met des accolades qui contiennent le corps de la closure.
Les accolades sont facultatives si le corps de la closure n' a qu'une seule ligne. Après les accolades, nous avons besoin d'un point-virgule pour aller avec le `let` statement. La valeur à la dernière ligne dans le corps de la closure (`num`), comme cette ligne ne se termine pas par un point-virgule, sera la valeur renvoyée par la closure lorsqu'elle est appelée, tout comme dans les corps de fonction.

Notez que cette instruction `let` signifie que la variable `expensive_closure` contient la *définition* d'une fonction anonyme, pas la valeur *résultant* de l'appel de la fonction anonyme. Rappelons la raison pour laquelle nous utilisons une closure est parce que nous voulons définir le code à appeler à un point, stocker ce code, et effectivement l'appeler à un point ultérieur; le code que nous voulons appeler est maintenant stocké dans `expensive_closure`.

Maintenant que nous avons défini la closure, nous pouvons changer le code dans les blocs `if` pour appeler la closure afin d'exécuter le code et obtenir la valeur résultante. L'appel à une closure ressemble beaucoup à l'appel d'une fonction; nous spécifions le nom de la variable qui détient la définition de la closure et la suivons avec des parenthèses contenant les valeurs du ou des arguments que nous voulons utiliser pour cet appel comme indiqué dans le Listing13-6:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-6: Appel de la closure `expensive_closure` que nous avons défini</span>

Maintenant, nous avons atteint l'objectif d'unifier là où le calcul coûteux est appelé à un seul endroit, et nous n'exécutons ce code que là où nous avons besoin des résultats. Cependant, nous avons réintroduit l'un des problèmes du Listing 13-3: nous continuons d'appeler la closure deux fois dans le premier bloc `if`, qui appellera le code coûteux deux fois et fera attendre l'utilisateur deux fois plus longtemps que nécessaire. Nous y reviendrons dans un instant; commençons par expliquer pourquoi il n' y a pas d'annotations de type dans la définition des closures et les traits impliqués dans les closures.

### Closure: inférence de type et annotations

Les closures ne nécessitent pas d'annoter le type des paramètres ou la valeur de retour comme le font les fonctions `fn`. Les annotations de type sont nécessaires pour les fonctions car elles font partie d'une interface explicite exposée à vos utilisateurs. Définir cette interface de manière rigide est important pour s'assurer que tout le monde s'accorde sur les types de valeurs qu'une fonction utilise et retourne. Mais les closures ne sont pas utilisées dans une interface exposée comme cela: elles sont stockées dans des variables et utilisées sans les nommer ni les exposer aux utilisateurs de notre librairie.



En outre, les closures sont généralement brèves et ne sont pertinentes que dans un contexte plutôt restreint que dans un scénario arbitraire. Dans ce contexte limité, le compilateur est capable d'inférer le type des paramètres et le type de retour, comme il est capable d'inférer le type de la plupart des variables.

Faire annoter par les programmeurs les types de ces petites fonctions anonymes serait agaçant et largement redondant avec l'information dont dispose déjà le compilateur.

Comme les variables, nous pouvons ajouter des annotations de type si nous voulons expliciter et augmenter la clarté  du code au risque d'être plus verbeux que ce qui est strictement nécessaire; annoter les types pour la closure que nous avons définie dans le Listing 13-4 ressemblerait à la définition présentée dans le Listing 13-7:

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

La syntaxe des closures et des fonctions semble plus similaire avec les annotations de type. Ce qui suit est une comparaison verticale de la syntaxe pour la définition d'une fonction qui ajoute une fonction à son paramètre, et une closures qui a le même comportement. Nous avons ajouté des espaces pour aligner les parties pertinentes. Ceci illustre comment la syntaxe des closures est similaire à la syntaxe des fonctions sauf pour l'utilisation des pipeset la quantité de code facultatif:


```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

La première ligne affiche une définition de fonction et la deuxième ligne une définition de closure entièrement annotée. La troisième ligne supprime les annotations de type de la définition de la closure, et la quatrième ligne supprime les crochets qui sont facultatifs, parce que le corps d'une closure n' a qu'une seule expression. Ce sont toutes des définitions valides qui produiront le même comportement quand on les appelle.

Les définitions de closures auront un type concret déduit pour chacun de leurs paramètres et pour leur valeur de retour. Par exemple, le Listing 13-8 montre la définition d'une petite closure qui renvoie simplement la valeur qu'elle reçoit comme paramètre. Cette closure n'est pas très utile sauf pour les besoins de cet exemple. Notez que nous n'avons pas ajouté d'annotations de type à la définition: si nous essayons alors d'appeler la closure deux fois, en utilisant une `String` comme argument la première fois et un `u32` la deuxième fois, nous obtiendrons une erreur:

<span class="filename">Fichier: src/main.rs</span>

```rust,ignore
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

<span class="caption">Listing 13-8: Tentative d'appeler une closure dont les types sont inférés avec deux types différents</span>

Le compilateur nous renvoie l'erreur suivante:

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

### Storage de Closures avec des paramètres génériques et le Trait `Fn`

Revenons à notre application de génération d'entraînement. Dans le Listing 13-6, notre code appelait toujours la closure de calcul coûteux plus de fois qu'il n'en avait besoin. Une option pour résoudre ce problème est de sauvegarder le résultat de la closure coûteuse dans une variable pour une future utilisation et d'utiliser la variable à  chaque endroit où nous en avons besoin au lieu de rappeler la closure plusieurs fois. Cependant, cette méthode pourrait donner lieu à un code très redondant.

Heureusement, une autre solution s'offre à nous. Nous pouvons créer une struct qui sauvegardera la closure et la valeur qui en résulte. La structure n'exécutera la closure que si nous avons besoin de la valeur résultante, et elle cachera la valeur résultante pour que le reste de notre code n'ait pas à être responsable de la sauvegarde et de la réutilisation du résultat. Vous connaissez peut-être ce pattern sous le nom de *mémoization* ou *évaluation larmoyante*.
<!--  I don't think that for the translation, pattern should be translated-->

Pour faire qu'une struct détiennne une closure, il faut spécifier le type de closure, car une définition de structure a besoin de connaître les types de chacun de ses champs. Chaque instance de closure a son propre type anonyme unique: c'est-à-dire que même si deux closures ont la même signature, leurs types sont toujours considérés comme différents. Pour définir des structs, des enums ou des paramètres de fonction qui utilisent les closures, nous utilisons des génériques et des limites de "traits", comme nous l'avons vu au chapitre 10.

Les traits `Fn` sont fournis par la librairie standard. Toutes les closures implémentent un des traits: `Fn`, `FnMut`, ou `FnOnce`. Nous discuterons de la différence entre ces traits dans la prochaine section sur la capture de l'environnement; dans cet exemple, nous pouvons utiliser le caractère `Fn`.

Nous ajoutons des types au trait `Fn` pour représenter les types de paramètres et les valeurs de retour que les closures doivent avoir pour correspondre à ce trait lié. Dans ce cas, notre closure a un paramètre de type `u32` et renvoie un `u32`, donc le trait lié que nous spécifions est `Fn (u32) -> u32`.

Le Listing 13-9 montre la définition de la struct`Cacher` qui possède une closure et une valeur de résultat optionnelle:

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

> Remarque: Les fonctions implémentent aussi les trois traits `Fn`. Si ce que nous voulons faire n' a pas besoin de capturer une
> valeur de l'environnement, nous pouvons utiliser une fonction plutôt qu'une closure où nous avons besoin de quelque chose qui
> implémente un trait `Fn`.


Avant d'exécuter la fermeture, `value` sera initialisée à `None`. Lorsque ducode utilisant un `Cacher` demande le *result* de la closure, le `Cacher` exécutera la closureà ce moment-là et stockera le résultat dans une variante `Some` dans le champ `value`. Ensuite, si le code demande à nouveau le résultat de la closure, au lieu d'exécuter à nouveau la closure, le `Cacher` renverra le résultat contenu dans la variante `Some`.

La logique autour du champ `value` que nous venons de décrire est définie dans le Listing 13-10:

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

La fonction `Cacher:: new` prend un paramètre générique `T`, que nous avons défini comme ayant le même caractère lié que la struct `Cacher`. Puis `Cacher:: new` renvoie une instance `Cacher` qui contient la closure spécifiée dans le champ `calculation` et une valeur `None` dans le champ `value`, parce que nous n'avons pas encore exécuté la closure.

Lorsque le code appelant veut le résultat de l'évaluation de la closure, au lieu d'appeler directement la clôture, il appellera la méthode `value`. Cette méthode vérifie si nous avons déjà une valeur  dans `self.value` dans un `Some`; si tel est le cas, elle renvoie la valeur contenue dans le `Some` sans exécuter de nouveau la closure.

Si `self.value` est à `None`, nous appelons la closure stockée dans `self.calculation`, sauvegardons le résultat dans `self. value` pour une utilisation future, et retournons la valeur après calcul.

Le Listing 13-11 montre comment utiliser cette struct `Cacher` dans la fonction `generate_workout` du Listing 13-6:

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

Au lieu de sauvegarder la closure dans une variable directement, nous sauvegardons une nouvelle instance de `Cacher` qui contient la closure. Ensuite, à chaque fois que nous voulons le résultat, nous appelons la méthode `value` sur l'instance de `Cacher`. Nous pouvons appeler la méthode `value` autant de fois que nous voulons, ou ne pas l'appeler du tout, et le calcul coûteux sera exécuté un maximum une fois.

Essayez d'exécuter ce programme avec la fonction `main` du Listing 13-2. Modifiez les valeurs des variables `simulated_user_specified_value` et `simulated_random_number` pour vérifier que dans tous les cas, dans les différents blocs `if` et `else`, `calculating slowly...` n'apparaît qu'une seule fois et seulement si nécessaire. Le `Cacher` prend soin de la logique nécessaire pour s'assurer que nous n'appelons pas le calcul coûteux plus que nous n'en avons besoin, ainsi `generate_workout` peut se concentrer sur la logique business.

### Limitations de l'implémentation de `Cacher`

Cacher des valeurs est un comportement généralement utile que nous pourrions vouloir utiliser dans d'autres parties de notre code avec différentes closures. Cependant, il y a deux problèmes avec la mise en oeuvre actuelle de `Cacher` qui rendraient difficile sa réutilisation dans des contextes différents.

Le premier problème est qu'une instance de `Cacher` suppose qu'elle obtiendra toujours la même valeur pour le paramètre `arg` à la méthode `value`. Autrement dit, ce test sur `Cacher` échouera:

```rust,ignore
#[test]
fn call_with_different_values() {
    let mut c = Cacher::new(|a| a);

    let v1 = c.value(1);
    let v2 = c.value(2);

    assert_eq!(v2, 2);
}
```

Ce test crée une nouvelle instance de `Cacher` avec une closure qui retourne la valeur qui lui est passée. Nous appelons la méthode `value` sur cette instance de `Cacher` avec une valeur `arg` de 1 et ensuite une valeur `arg` de 2, et nous nous attendons à ce que l'appel à `value` avec la valeur `arg` de 2 devrait retourner 2.

Exécutez ce test avec l'implémentation de `Cacher` duListing 13-9 et du Listing 13-10, et le test échouera sur le `assert_eq! ` avec ce message:

```text
thread 'call_with_different_values' panicked at 'assertion failed: `(left == right)`
  left: `1`,
 right: `2`', src/main.rs
```

Le problème est que la première fois que nous avons appelé `c. value` avec 1, l'instance `Cacher` a sauvé `Some (1)` dans `self. value`. Par la suite, peu importe ce que nous passons à la méthode `value`, elle retournera toujours 1.

Essayez de modifier `Cacher` pour tenir une hashmap plutôt qu'une seule valeur. Les clés de la hashmap seront les valeurs `arg` qui lui sont passées, et les valeurs de la hashmap  seront le résultat de l'appel de la fermeture sur cette clé. Plutôt que de regarder si `self. value` a directement une valeur `Some` ou une valeur `None`, la fonction `value` recherchera `arg` dans la hashmap et retournera la valeur si elle est présente. S'il n'est pas présent, le `Cacher` appellera la closure et sauvegardera la valeur résultante dans la hashmap  associée à sa valeur `arg`.

Le second problème avec l'implémentation actuelle de `Cacher` est qu'il n'accepte que les closures qui prennent un paramètre de type `u32` et renvoient un `u32`. Nous pourrions vouloir mettre en cache les résultats des fermetures qui prennent une slice d'une string et renvoient des valeurs `usize`, par exemple. Pour corriger ce problème, essayez d'introduire des paramètres plus génériques pour augmenter la flexibilité de la fonctionnalité que possède `Cacher`.

### Capturer l'environnement avec des Closures

Dans l'exemple du générateur d'entraînement, nous n'avons utilisé les closures que comme des fonctions anonymes "inline". Cependant, les closures ont une capacité supplémentaire que les fonctions n'ont pas: elles peuvent capturer leur environnement et accéder aux variables à partir du scope dans lequel elles sont définies.

Le Listing 13-12 a un exemple de closure stockée dans la variable `equal_to_x` qui utilise la variable `x` de l'environnement environnant de la closure:

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

Ici, même si `x` n'est pas un des paramètres de `equal_to_x`, la closure `equal_to_x` est autorisée à utiliser la variable `x` définie dans la même portée que `equal_to_x`.

Nous ne pouvons pas faire la même chose avec les fonctions; si nous essayons avec l'exemple suivant, notre code ne compilera pas:

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

Le compilateur nous rappelle même que cela ne fonctionne qu'avec les closures!

Lorsqu'une closure capture une valeur de son environnement, elle utilise la mémoire pour stocker les valeurs à utiliser dans son corps. Cette utilisation de la mémoire a un coût supplémentaire que nous ne voulons pas payer dans les cas les plus courants où nous voulons exécuter du code qui ne capture pas leur environnement. Parce que les fonctions ne sont jamais autorisées à capturer leur environnement, la définition et l'utilisation des fonctions n'occasionneront jamais cette surcharge.

Les closures peuvent captuer les valeurs de leur environnement de trois façons, ce qui correspond directement aux trois façons dont une fonction peut prendre un paramètre: la prise de propriété (ownership), l'emprunt immutable et l'emprunt mutable. Ceux-ci sont codés dans les trois traits `Fn` comme suit:

* `FnOnce` consomme les variables qu'il capture à partir de son scope, connu sous le nom de l'*environnement* de la closure.      Pour consommer les variables capturées, la closure doit s'approprier ces variables et les déplacer dans la closure lorsqu'elle est définie. La partie `Once` du nom représente le fait que la closure ne peut pas prendre la propriété (ou ownership) des mêmes variables plus d'une fois, donc elle ne peut être appelée qu'une seule fois.
* `Fn` emprunte des valeurs à l'environnement de façon immutable.
* `FnMut` peut changer l'environnement parce qu'il emprunte des valeurs de manière mutable.

Lorsque nous créons une closure, Rust déduit quel caractère utiliser en se basant sur la façon dont la closure utilise les valeurs de l'environnement. Dans le Listing 13-12, la closure `equal_to_x` emprunte `x` immutablement (donc `equal_to_x` a le trait `Fn`) parce que le corps de la closure ne fait que lire la valeur de `x`.

Si nous voulons forcer la closure à s'approprier les valeurs qu'elle utilise dans l'environnement, nous pouvons utiliser le mot-clé `move` avant la liste des paramètres. Cette technique est surtout utile lorsque vous passez une fermeture à un nouveau thread pour déplacer les données afin qu'elles appartiennent au nouveau thread.

Nous aurons d'autres exemples de closures utilisant `move` au chapitre 16 lorsque nous parlerons du multithreading. Pour l'instant, voici le code du Listing 13-12 avec le mot-clé `move` ajouté à la définition de la closure et utilisant des vecteurs au lieu d'entiers, car les entiers peuvent être copiés plutôt que déplacés; notez que ce code ne compilera pas encore:

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

Pour illustrer les situations où les closures capturant leur environnement sont utiles comme paramètres de fonction, passons à notre prochain sujet: les itérateurs.

Closure are different than functions defined with the `fn` keyword in a few
ways. The first is that closures don't require you to annotate the types of the
parameters or the return value like `fn` functions do.

<!-- I've suggested moving this next paragraph up from below, I found this
section difficult to follow with this next paragraph -->

Type annotations are required on functions because they're are part of an
explicit interface exposed to your users. Defining this interface rigidly is
important for ensuring that everyone agrees on what types of values a function
uses and returns. Closures aren't used in an exposed interface like this,
though: they're stored in variables and used without naming them and exposing
them to be invoked by users of our library.

Additionally, closures are usually short and only relevant within a narrow
context rather than in any arbitrary scenario. Within these limited contexts,
the compiler is reliably able to infer the types of the parameters and return
type similarly to how it's able to infer the types of most variables. Being
forced to annotate the types in these small, anonymous functions would be
annoying and largely redundant with the information the compiler already has
available.

<!--Can you expand above on what you mean by "stored in bindings and called
directly"? Do you mean stored in a variable? I'm struggling to visualize how
closures are used, and what the important difference is between them and
functions. I think a clearer definition of what they are, what they do, and
what they're used for at the start of the closures section would help clear
this up -->
<!-- Yes, sorry, in Rust terminology "binding" is mostly synonymous to
"variable", but when we started working on the book we decided to be consistent
and more like what people are used to by referring to the concept as "variable"
throughout, but we missed this spot. /Carol -->

Like variables, we can choose to add type annotations if we want to increase
explicitness and clarity in exchange for being more verbose than is strictly
necessary; annotating the types for the closure we defined in Listing 13-4
would look like the definition shown here in Listing 13-7:

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num: i32| -> i32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

<span class="caption">Listing 13-7: Adding optional type annotations of the
parameter and return value types in the closure</span>

<!-- Why might you want to, if you don't need to? In a particular situation? -->
<!-- I've added an explanation /Carol -->

<!-- Below -- Am I right in assuming the closures below are doing the same
thing as the functions? -->
<!-- Yes /Carol -->

The syntax of closures and functions looks more similar with type annotations.
Here's a vertical comparison of the syntax for the definition of a function
that adds one to its parameter, and a closure that has the same behavior. We've
added some spaces here to line up the relevant parts). This illustrates how
closure syntax is similar to function syntax except for the use of pipes rather
than parentheses and the amount of syntax that is optional:

<!-- Prod: can you align this as shown in the text? -->
<!-- I'm confused, does this note mean that production *won't* be aligning all
of our other code examples as shown in the text? That's concerning to me, we're
trying to illustrate idiomatic Rust style in all the code examples, so our
alignment is always intentional... /Carol -->

```rust,ignore
fn  add_one_v1   (x: i32) -> i32 { x + 1 }
let add_one_v2 = |x: i32| -> i32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

<!-- Can you point out where we're looking at here, where the important
differences lie? -->
<!-- The importance isn't the difference but the similarity... I've tried to
clarify /Carol -->

<!-- Will add wingdings and ghosting in libreoffice /Carol -->

The first line shows a function definition, and the second line shows a fully
annotated closure definition. The third line removes the type annotations from
the closure definition, and the fourth line removes the braces that are
optional since the closure body only has one line. These are all valid
definitions that will produce the same behavior when they're called.

<!--Below--I'm not sure I'm following, is the i8 type being inferred? It seems
like we're annotating it. -->
<!-- The types in the function definitions are being inferred, but since Rust's
variable types for numbers defaults to `i32`, in order to illustrate our point
here we're forcing the type of the *variable* to be `i8` by annotating it in
the *variable* declaration. I've changed the example to hopefully be less
confusing and convey our point better. /Carol -->

Closure definitions will have one concrete type inferred for each of their
parameters and for their return value. For instance, Listing 13-8 shows the
definition of a short closure that just returns the value it gets as a
parameter. This closure isn't very useful except for the purposes of this
example. Note that we haven't added any type annotations to the definition: if
we then try to call the closure twice, using a `String` as an argument the
first time and an `i32` the second time, we'll get an error:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

<span class="caption">Listing 13-8: Attempting to call a closure whose types
are inferred with two different types</span>

The compiler gives us this error:

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

<!-- Will add wingdings in libreoffice /Carol -->

The first time we call `example_closure` with the `String` value, the compiler
infers the type of `x` and the return type of the closure to be `String`. Those
types are then locked in to the closure in `example_closure`, and we get a type
error if we try to use a different type with the same closure.

### Using Closures with Generic Parameters and the `Fn` Traits

Returning to our workout generation app, in Listing 13-6 we left our code still
calling the expensive calculation closure more times than it needs to. In each
place throughout our code, if we need the results of the expensive closure more
than once, we could save the result in a variable for reuse and use the
variable instead of calling the closure again. This could be a lot of repeated
code saving the results in a variety of places.

However, because we have a closure for the expensive calculation, we have
another solution available to us. We can create a struct that will hold the
closure and the resulting value of calling the closure. The struct will only
execute the closure if we need the resulting value, and it will cache the
resulting value so that the rest of our code doesn't have to be responsible for
saving and reusing the result. You may know this pattern as *memoization* or
*lazy evaluation*.

In order to make a struct that holds a closure, we need to be able to specify
the type of the closure. Each closure instance has its own unique anonymous
type: that is, even if two closures have the same signature, their types are
still considered to be different. In order to define structs, enums, or
function parameters that use closures, we use generics and trait bounds like we
discussed in Chapter 10.

<!-- So Fn is a trait built into the language, is that right? I wasn't sure if
it was just a placeholder here -->
<!-- Fn is provided by the standard library; I've clarified here. /Carol -->

The `Fn` traits are provided by the standard library. All closures implement
one of the traits `Fn`, `FnMut`, or `FnOnce`. We'll discuss the difference
between these traits in the next section on capturing the environment; in this
example, we can use the `Fn` trait.

We add types to the `Fn` trait bound to represent the types of the parameters
and return values that the closures must have in order to match this trait
bound. In this case, our closure has a parameter of type `i32` and returns an
`i32`, so the trait bound we specify is `Fn(i32) -> i32`.

Listing 13-9 shows the definition of the `Cacher` struct that holds a closure
and an optional result value:

<span class="filename">Filename: src/main.rs</span>

```rust
struct Cacher<T>
    where T: Fn(i32) -> i32
{
    calculation: T,
    value: Option<i32>,
}
```

<span class="caption">Listing 13-9: Defining a `Cacher` struct that holds a
closure in `calculation` and an optional result in `value`</span>

The `Cacher` struct has a `calculation` field of the generic type `T`. The
trait bounds on `T` specify that `T` is a closure by using the `Fn` trait. Any
closure we want to store in the `calculation` field of a `Cacher` instance must
have one `i32` parameter (specified within the parentheses after `Fn`) and must
return an `i32` (specified after the `->`).

The `value` field is of type `Option<i32>`. Before we execute the closure,
`value` will be `None`. If the code using a `Cacher` asks for the result of the
closure, we'll execute the closure at that time and store the result within a
`Some` variant in the `value` field. Then if the code asks for the result of
the closure again, instead of executing the closure again, we'll return the
result that we're holding in the `Some` variant.

The logic around the `value` field that we've just described is defined in
Listing 13-10:

<span class="filename">Filename: src/main.rs</span>

```rust
# struct Cacher<T>
#     where T: Fn(i32) -> i32
# {
#     calculation: T,
#     value: Option<i32>,
# }
#
impl<T> Cacher<T>
    where T: Fn(i32) -> i32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: i32) -> i32 {
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

<!-- Liz: the `new` function is using the "struct init shorthand" by just
saying `calculation` instead of `calculation: calculation`, since the parameter
matches the struct field name. This was recently added to stable Rust and we've
added an introduction of it in Chapter 5. Just adding an explanation here for
you in case you read this chapter before the changes we've made to Chapter 5!
/Carol -->

<span class="caption">Listing 13-10: Implementations on `Cacher` of an
associated function named `new` and a method named `value` that manage the
caching logic</span>

The fields on the `Cacher` struct are private since we want `Cacher` to manage
their values rather than letting the calling code potentially change the values
in these fields directly. The `Cacher::new` function takes a generic parameter
`T`, which we've defined in the context of the `impl` block to have the same
trait bound as the `Cacher` struct. `Cacher::new` returns a `Cacher` instance
that holds the closure specified in the `calculation` field and a `None` value
in the `value` field, since we haven't executed the closure yet.

When the calling code wants the result of evaluating the closure, instead of
calling the closure directly, it will call the `value` method. This method
checks to see if we already have a resulting value in `self.value` in a `Some`;
if we do, it returns the value within the `Some` without executing the closure
again.

If `self.value` is `None`, we call the closure stored in `self.calculation`,
save the result in `self.value` for future use, and return the value as well.

Listing 13-11 shows how we can use this `Cacher` struct in the
`generate_workout` function from Listing 13-6:

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# struct Cacher<T>
#     where T: Fn(i32) -> i32
# {
#     calculation: T,
#     value: Option<i32>,
# }
#
# impl<T> Cacher<T>
#     where T: Fn(i32) -> i32
# {
#     fn new(calculation: T) -> Cacher<T> {
#         Cacher {
#             calculation,
#             value: None,
#         }
#     }
#
#     fn value(&mut self, arg: i32) -> i32 {
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
fn generate_workout(intensity: i32, random_number: i32) {
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
            )
        }
    }
}
```

<span class="caption">Listing 13-11: Using `Cacher` in the `generate_workout`
function to abstract away the caching logic</span>

<!-- Will add ghosting and wingdings in libreoffice /Carol -->

Instead of saving the closure in a variable directly, we save a new instance of
`Cacher` that holds the closure. Then, in each place we want the result, we
call the `value` method on the `Cacher` instance. We can call the `value`
method as many times as we want, or not call it at all, and the expensive
calculation will be run a maximum of once. Try running this program with the
`main` function from Listing 13-2, and change the values in the
`simulated_user_specified_value` and `simulated_random_number` variables to
verify that in all of the cases in the various `if` and `else` blocks,
`calculating slowly...` printed by the closure only shows up once and only when
needed.

The `Cacher` takes care of the logic necessary to ensure we aren't calling the
expensive calculation more than we need to be so that `generate_workout` can
focus on the business logic. Caching values is a more generally useful behavior
that we might want to use in other parts of our code with other closures as
well. However, there are a few problems with the current implementation of
`Cacher` that would make reusing it in different contexts difficult.

The first problem is a `Cacher` instance assumes it will always get the same
value for the parameter `arg` to the `value` method. That is, this test of
`Cacher` will fail:

```rust,ignore
#[test]
fn call_with_different_values() {
    let mut c = Cacher::new(|a| a);

    let v1 = c.value(1);
    let v2 = c.value(2);

    assert_eq!(v2, 2);
}
```

This test creates a new `Cacher` instance with a closure that returns the value
passed into it. We call the `value` method on this `Cacher` instance with
an `arg` value of 1 and then an `arg` value of 2, and we expect that the call
to `value` with the `arg` value of 2 returns 2.

Run this with the `Cacher` implementation from Listing 13-9 and Listing 13-10
and the test will fail on the `assert_eq!` with this message:

```text
thread 'call_with_different_arg_values' panicked at 'assertion failed:
`(left == right)` (left: `1`, right: `2`)', src/main.rs
```

The problem is that the first time we called `c.value` with 1, the `Cacher`
instance saved `Some(1)` in `self.value`. After that, no matter what we pass
in to the `value` method, it will always return 1.

Try modifying `Cacher` to hold a hash map rather than a single value. The keys
of the hash map will be the `arg` values that are passed in, and the values of
the hash map will be the result of calling the closure on that key. Instead of
looking at whether `self.value` directly has a `Some` or a `None` value, the
`value` function will look up the `arg` in the hash map and return the value if
it's present. If it's not present, the `Cacher` will call the closure and save
the resulting value in the hash map associated with its `arg` value.

Another problem with the current `Cacher` implementation that restricts its use
is that it only accepts closures that take one parameter of type `i32` and
return an `i32`. We might want to be able to cache the results of closures that
take a string slice as an argument and return `usize` values, for example. Try
introducing more generic parameters to increase the flexibility of the `Cacher`
functionality.

### Closures Can Capture Their Environment

In the workout generator example, we only used closures as inline anonymous
functions. Closures have an additional ability we can use that functions don't
have, however: they can capture their environment and access variables from the
scope in which they're defined.

<!-- To clarify, by enclosing scope, do you mean the scope that the closure is
inside? Can you expand on that?-->
<!-- Yes, I've tried here to clarify that it's the scope in which the closure
is defined /Carol -->

Listing 13-12 has an example of a closure stored in the variable `equal_to_x`
that uses the variable `x` from the closure's surrounding environment:

<!-- To clarify how we talk about a closure, does the closure include the
variable name, or are we referring to the closure as the functionality that is
on the right side of the = and so not including to variable name? I thought it
was the former, but it seems like the latter above. If it's the former, would
"an example of a closure with the variable `equal_to_x`" make more sense? -->
<!-- No, the closure does not include the variable name in which it's stored;
storing a closure in a variable is optional. It should always be the latter,
and I hope the reworking of the example used throughout the closure section
and making the wording consistent has cleared this confusion up by this point.
/Carol -->

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```

<span class="caption">Listing 13-12: Example of a closure that refers to a
variable in its enclosing scope</span>

Here, even though `x` is not one of the parameters of `equal_to_x`, the
`equal_to_x` closure is allowed to use the `x` variable that's defined in the
same scope that `equal_to_x` is defined in.

<!-- So *why* is this allowed with closures and not functions, what about
closures makes this safe? -->
<!-- It's not really about safety; capturing the environment is the defining
functionality a closure has. The reason functions *don't* capture their
environment is mostly around storage overhead; I've added an explanation of
that aspect. Can you elaborate on what led to the conclusion that allowing
captures with functions wouldn't be safe and what you mean by "safe" here?
/Carol -->

We can't do the same with functions; let's see what happens if we try:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 4;

    fn equal_to_x(z: i32) -> bool { z == x }

    let y = 4;

    assert!(equal_to_x(y));
}
```

We get an error:

```text
error[E0434]: can't capture dynamic environment in a fn item; use the || { ... }
closure form instead
 -->
  |
4 |     fn equal_to_x(z: i32) -> bool { z == x }
  |                                          ^
```

The compiler even reminds us that this only works with closures!

When a closure captures a value from its environment, the closure uses memory
to store the values for use in the closure body. This use of memory is overhead
that we don't want pay for in the more common case where we want to execute
code that doesn't capture its environment. Because functions are never allowed
to capture their environment, defining and using functions will never incur
this overhead.

<!-- Why didn't this work, is there a reason ingrained in the language? Or is
that not really relevant? -->
<!-- I've added an explanation /Carol -->

Closures can capture values from their environment in three ways, which
directly map to the three ways a function can take a parameter: taking
ownership, borrowing immutably, and borrowing mutably. These ways of capturing
values are encoded in the three `Fn` traits as follows:

* `FnOnce` takes ownership of the variables it captures from the environment
  and moves those variables into the closure when the closure is defined.
  Therefore, a `FnOnce` closure cannot be called more than once in the same
  context.
* `Fn` borrows values from the environment immutably.
* `FnMut` can change the environment since it mutably borrows values.

When we create a closure, Rust infers how we want to reference the environment
based on how the closure uses the values from the environment. In Listing
13-12, the `equal_to_x` closure borrows `x` immutably (so `equal_to_x` has the
`Fn` trait) since the body of the closure only needs to read the value in `x`.

If we want to force the closure to take ownership of the values it uses in the
environment, we can use the `move` keyword before the parameter list. This is
mostly useful when passing a closure to a new thread in order to move the data
to be owned by the new thread. We'll have more examples of `move` closures in
Chapter 16 when we talk about concurrency, but for now here's the code from
Listing 13-12 with the `move` keyword added to the closure definition and using
vectors instead of integers, since integers can be copied rather than moved:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x;

    println!("can't use x here: {:?}", x);

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
```

This example doesn't compile:

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

The `x` value is moved into the closure when the closure is defined because of
the `move` keyword. The closure then has ownership of `x`, and `main` isn't
allowed to use `x` anymore. Removing the `println!` will fix this example.

Most of the time when specifying one of the `Fn` trait bounds, you can start
with `Fn` and the compiler will tell you if you need `FnMut` or `FnOnce` based
on what happens in the closure body.

To illustrate situations where closures that can capture their environment are
useful as function parameters, let's move on to our next topic: iterators.
