## To `panic!` or Not to `panic!`

Donc, comment décider quand on doit faire un `panic!` et quand nous devons
retourner un `Result` ? Quand un code fait un panic, il n'y a pas de moyen de
continuer l'exécution. Vous pouriez faire appel à `panic!` pour n'importe
quelle situation d'erreur, peux importe s'il est possible de continuer
l'exécution ou non, mais alors vous prenez la décision de tout arrêter à la
place du code appelant. Quand vous choisissez de retourner une valeur `Result`,
vous donnez plus de choix au code appelant plutôt que si vous preniez des
décisions à sa place. Le code appelant peut choisir d'essayer de récupérer
l'erreur de manière appropriée à cette situation, ou il peut décider que dans
ce cas une valeur `Err` est irrécupérable, donc utiliser `panic!` et changer
votre erreur récupérable en erreur irrécupérable. Ainsi, retourner `Result` est
un bon choix par défaut quand vous construisez une fonction qui peux échouer.

Dans quelques situations, c'est plus approprié d'écrire du code qui fait un
panic au lieu de retourner un `Result`, mais elles sont moins fréquentes.
Voyons pourquoi ils est approprié pourquoi il est plus approprié de faire un
panic dans des examples, des prototypes, et des tests; ensuite des situations
où vous savez qu'en tant qu'humain qu'une méthode ne peux pas échouer mais que
le compilateur n'a pas de raison de le faire; et nous alons conclure par
quelques lignes conductrices générales sur comment décider s'il faut paniquer
dans le code des librairies.

### Les examples, prototypes, et les tests sont les endroit parfaits pour Panic

Quand vous écrivez un example pour illustrer un concept, avoir un code de
gestion des erreurs très résiliente peut rendre l'exemple moins claire. Dans
les examples, il est courrant d'utiliser une méthode comme `unwrap` qui peut
faire un `panic!` qui remplace le code de gestion de l'erreur que vous
utiliseriez dans votre application, qui peux changer en fonction de ce que le
reste de votre code va faire.

De la même manière, les méthodes `unwrap` et `expect` sont très pratiques pour
coder des prototypes, avant de décider comment gérer les erreurs. Ce sont des
indicateurs claires dans votre code pour plus tard quand vous serez prêt à
rendre votre code plus résilient aux échecs.

Si l'appel à une méthode échoue dans un test, nous voulons que tout le test
échoue, même si cette méthode n'est pas la fonctionnalité que nous testons.
Parceque `panic!` est la manière de le marquer un échec, utiliser `unwrap` ou
`expect` est exactement ce qui est nécessaire.

### Les cas où vous avez plus d'informations que le compilateur

Il peut parfois être approprié d'utiliser `unwrap` quand vous avez un code
logique qui garanti que `Result` aura toujours une valeur de type `Ok`, mais
que c'est une logique que le compilateur ne comprends pas. Vous travaillez
toujours une valeur de type `Result` que vous devez gérer : quelque soit
l'instruction que vous appelez, elle a toujours la possibilité d'échouer,
même si dans ce cas particulier, c'est logiquement impossible. Si vous êtes
sûr en inspectant le code manuellement que vous n'allez jamais obtenir une
variante de `Err`, il est tout à fait acceptable d'utiliser `unwrap`. Voici un
example :

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```

Nous créons une instance de `IpAddr` en parsant une chaine de caractères pure.
Nous savons que `127.0.0.1` est une adresse IP valide, donc il est convenable
d'utiliser `unwrap` ici. Toutefois, avoir une chaine de caractères valide,
codée en dur ne change jamais le type de retour de la méthode `parse` : nous
obtenons toujours une valeur de type `Result`, et le compilateur va nous faire
gérer le `Result` comme si la variante `Err` est toujours probable car le
compilateur n'est pas encore suffisamment intelligent pour analyser que cette
chaine de caractères est toujours une adresse IP valide. Si le texte de
l'adresse IP provient de l'utilisateur plutôt que d'être codé en dur dans le
programme, et fait en sorte qu'il y a désormais une possibilité d'erreur, nous
devrions gérer le `Result` de manière plus résiliente désormais.

### Recommandations pour gérer les erreurs

Il est recommandé de faire un `panic!` dans votre code lorsqu'il s'exécute dans
de mauvaises conditions. Dans ce sens, les mauvaises conditions sont lorsque un
postulat, une garantie, un contrat ou une invariance a été rompue, comme des
valeurs invalides, contradictoires ou manquantes fournies à votre code, par un
ou plusieurs des éléments suivants :

* Ces mauvaises conditions ne sont pas *prévues* pour surgir de temps en temps.
* Après cette instruction, votre code a besoin de ne pas être dans ces
mauvaises conditions.
* Il n'y pas de bonne façon de coder ces informations dans les types que vous
utilisez.

Si quelqu'un utilise votre code et lui fournit des valeurs qui n'ont pas de
sens, la meilleure des choses à faire et de faire un `panic!` et avertir
le développeur de votre librairie du bogue dans leur code afin qu'il le règle
pendant la phase de développement. De la même manière, `panic!` est parfois
approprié si vous appellez un code externe dont vous n'avez pas la main dessus,
et qu'il retourne de mauvaises conditions que vous ne pouvez pas corriger.

Lorsque les condtions sont mauvaises, mais qui est prévu que cela arrive peu
importe la façon que vous écrivez votre code, il est plus aproprié de retourner
un `Result` plutôt que faire appel à `panic!`. Il peut s'agir par exemple d'un
un parseur qui reçoit des données éronnées, ou une requête HTTP qui renvoit un
statut qui indique que vous avez atteint une limite de débit. Dans ce cas, vous
devriez que cet échec est un possibilité en retournant un `Result` pour
propager ces mauvaises conditions vers le haut pour que le code appelant puisse
décider quoi faire pour gérer le problème. Faire un `panic!` ne serait pas la
manière la approprié ici pour gérer ces problèmes.

Lorsque votre code effecture des opérations sur des valeurs, votre code devrait
d'abord vérifier que ces valeurs sont valides, et faire un `panic!` si les
valeurs ne sont sont pas correctes. C'est pour essentiellements des raisons de
sécurité : tenter de travailler avec des données invalides peut exposer votre
code à des vulnérabilités. C'est la raison principale pour laquelle la
librairie standard va faire un `panic!` si vous essayez d'accéder d'accéder à
la mémoire en dehors des limites : essayer d'accéder à de la mémoire qui n'a
pas de rapport avec la structure des données actuelle est un problème de
sécurité courant. Les fonctions ont parfois des *contrats* : leur comportement
est garanti uniquement si les données d'entrée remplissent des conditions
particulières. Faire un panic quand le contrat est violé est sensé car une
violation de contrat indique toujours un bogue du coté de l'appelant, et ce
n'est le genre d'erreur que vous voulez que le code appelant gère
explicitement. En fait, il n'y a aucun moyen raisonable de récupérer le code
d'appel : le *développeur* du code appelant doit corriger le code. Les contrats
de foncion, en particulier quand une violation va faire un panic, doivent être
expliqués dans la documentation de l'API pour la fonction.

Cependant, avoir beaucoup de vérifications d'erreurs dans toutes vos fonctions
risque d'être verbeux et ennuyeux. Heureusement, vous pouvez utiliser le
système de type de Rust (et donc la vérification de type du compilateur) pour
faire une partie des vérifcations à votre place. Si votre fonction un paramètre
d'un type particulier, vous pouvez continuer à écrire votre code en sachant que
le compilateur s'est déjà assuré que vous avez une valeur valide. Par exemple,
si vous avez un type de valeur plutôt que un `Option`, votre programme
s'attends d'avoir *autre chose* plutôt que *rien*. Votre code n'a donc pas à
gérer les deux cas de variantes `Some` et `None` : il n'aura qu'un seul cas
pour avoir une valeur. Du code qui essaye de ne rien fournir à votre fonction
ne compilera même pas, donc votre fonction n'a pas besoin de vérifier ce cas-ci
lors de l'exécution. Un autre exemple est d'utiliser un type unsigned integer
comme `u32`, qui garantit que le paramètre n'est jamais négatif.

### Créer des types personnalisés pour la Validation

Let’s take the idea of using Rust’s type system to ensure we have a valid value
one step further and look at creating a custom type for validation. Recall the
guessing game in Chapter 2 where our code asked the user to guess a number
between 1 and 100. We never validated that the user’s guess was between those
numbers before checking it against our secret number; we only validated that
the guess was positive. In this case, the consequences were not very dire: our
output of “Too high” or “Too low” would still be correct. It would be a useful
enhancement to guide the user toward valid guesses and have different behavior
when a user guesses a number that’s out of range versus when a user types, for
example, letters instead.

One way to do this would be to parse the guess as an `i32` instead of only a
`u32` to allow potentially negative numbers, and then add a check for the
number being in range, like so:

```rust,ignore
loop {
    // --snip--

    let guess: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if guess < 1 || guess > 100 {
        println!("The secret number will be between 1 and 100.");
        continue;
    }

    match guess.cmp(&secret_number) {
    // --snip--
}
```

The `if` expression checks whether our value is out of range, tells the user
about the problem, and calls `continue` to start the next iteration of the loop
and ask for another guess. After the `if` expression, we can proceed with the
comparisons between `guess` and the secret number knowing that `guess` is
between 1 and 100.

However, this is not an ideal solution: if it was absolutely critical that the
program only operated on values between 1 and 100, and it had many functions
with this requirement, it would be tedious (and potentially impact performance)
to have a check like this in every function.

Instead, we can make a new type and put the validations in a function to create
an instance of the type rather than repeating the validations everywhere. That
way, it’s safe for functions to use the new type in their signatures and
confidently use the values they receive. Listing 9-9 shows one way to define a
`Guess` type that will only create an instance of `Guess` if the `new` function
receives a value between 1 and 100:

```rust
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }

    pub fn value(&self) -> u32 {
        self.value
    }
}
```

<span class="caption">Listing 9-9: A `Guess` type that will only continue with
values between 1 and 100</span>

First, we define a struct named `Guess` that has a field named `value` that
holds a `u32`. This is where the number will be stored.

Then we implement an associated function named `new` on `Guess` that creates
instances of `Guess` values. The `new` function is defined to have one
parameter named `value` of type `u32` and to return a `Guess`. The code in the
body of the `new` function tests `value` to make sure it’s between 1 and 100.
If `value` doesn’t pass this test, we make a `panic!` call, which will alert
the programmer who is writing the calling code that they have a bug they need
to fix, because creating a `Guess` with a `value` outside this range would
violate the contract that `Guess::new` is relying on. The conditions in which
`Guess::new` might panic should be discussed in its public-facing API
documentation; we’ll cover documentation conventions indicating the possibility
of a `panic!` in the API documentation that you create in Chapter 14. If
`value` does pass the test, we create a new `Guess` with its `value` field set
to the `value` parameter and return the `Guess`.

Next, we implement a method named `value` that borrows `self`, doesn’t have any
other parameters, and returns a `u32`. This is a kind of method sometimes
called a *getter*, because its purpose is to get some data from its fields and
return it. This public method is necessary because the `value` field of the
`Guess` struct is private. It’s important that the `value` field is private so
code using the `Guess` struct is not allowed to set `value` directly: code
outside the module *must* use the `Guess::new` function to create an instance
of `Guess`, which ensures there’s no way for a `Guess` to have a `value` that
hasn’t been checked by the conditions in the `Guess::new` function.

A function that has a parameter or returns only numbers between 1 and 100 could
then declare in its signature that it takes or returns a `Guess` rather than a
`u32` and wouldn’t need to do any additional checks in its body.

## Summary

Rust’s error handling features are designed to help you write more robust code.
The `panic!` macro signals that your program is in a state it can’t handle and
lets you tell the process to stop instead of trying to proceed with invalid or
incorrect values. The `Result` enum uses Rust’s type system to indicate that
operations might fail in a way that your code could recover from. You can use
`Result` to tell code that calls your code that it needs to handle potential
success or failure as well. Using `panic!` and `Result` in the appropriate
situations will make your code more reliable in the face of inevitable problems.

Now that you’ve seen useful ways that the standard library uses generics with
the `Option` and `Result` enums, we’ll talk about how generics work and how you
can use them in your code in the next chapter.
