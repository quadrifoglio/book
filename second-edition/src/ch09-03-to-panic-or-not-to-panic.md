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

Allons plus loin dans l'idée d'utiliser le système de types de Rust pour
assurer d'avoir une valeur valide en créant un type personnalisé pour
validation. Souvenez-vous du jeu du plus ou du moins du Chapitre 2 où notre
code demandait à l'utilisateur de deviner un nombre entre 1 et 100. Nous
n'avons jamais validé que le nombre fourni par l'utilisateur était entre ces
nombres avant de le comparer à notre nombre secret; nous avons seulement validé
que le nombre était positif. Dans ce cas, les conséquences ne sont pas très
graves : notre resultat "Too high" ou "Too low" sera toujours correct. Ce
serait une amélioration utile pour guider les suppositions de l'utilisateur
vers des valeurs valides et d'avoir différents comportements quand un
utilisateur propose un nombre en dehors des limites versus quand un utilisateur
renseigne, par exemple, des lettres à la place.

Une façon de faire cela serait de faire un parse du nombre rensigné en un `i32`
plutôt que seulement un `u32` pour permettre des potentiels nombres négatifs,
et ensuite vérifier que le nombre est dans la plage autorisée, comme ceci :

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

L'expression du `if` vérifie si nous valeur est en dehors des limites, et
informe l'utilisateur du problème, et utilise `continue` pour passer à la
prochaine itération de la boucle et demander un nouveau nombre à deviner. Après
l'expression `if`, nous pouvons continuer avec la comparaison entre `guess` et
le nombre secret en sachant que `guess` est entre 1 et 100.

Cependant, ce n'est pas une solution idéale : si c'était absolument critique
que le programme travaille avec des valeurs entre 1 et 100, et qu'il aurait de
nombreuses fonctions qui exige cela, cela pourait être fastidieux (et cela
impacterait potentiellement la performance) de faire une validation comme
celle-ci dans chaque fonction.

A la place, nous pourrions construire un nouveau type et y intégrer les
validations dans une fonction pour créer une instance de ce type plutôt que de
répliquer les validations partout. De cette manière, c'est plus sûr pour les
fonctions d'utiliser le nouveau type dans leurs signatures et d'utiliser avec
confiance les valeurs qu'ils reçoivent. L'entrée 9-9 montre une façon de
définir un type `Guess` qui ne créera une instance de `Guess` uniquement si la
fonction `new` reçoit une valeur entre 1 et 100 :

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

<span class="caption">Entrée 9-9: un type `Guess` qui ne va continuer
uniquement su la valeur est entre 1 et 100</span>

Premièrement, nous définissons un struct qui s'appelle `Guess` qui a un champ
`value` qui stocke un `u32`. C'est la que le nombre sera stocké.

Ensuite, nous ajoutons un fonction associée `new` sur `Guess` qui crée des
instances de `Guess`. La fonction `new` est conçue pour avoir un paramètre
`value` de type `u32` et de retourner un `Guess`. Le code dans le corps de la
fonction `new` vérifie `value` pour s'assurer que c'est bien entre 1 et 100.
Si `value` ne échoue à ce test, nous faisons un `panic!`, qui alertera le
développeur qui écrit le code appellant qu'ils ont un bogue qu'ils ont besoin
de régler, car créer un `Guess` avec un `value` en dehors de cette plage va
violer le contrat sur lequel `Guess::new` s'appuie. Les conditions dans
lesquels `Guess::new` va faire un panic devrait être explicité dans sa
documentationsur l'API ouverte au public; nous verrons les conventions de
documentation pour indiquer un `panic!` lorsque vous créerez votre
documentation d'API au Chapitre 14. Si `value` réussit le test, nous allons
créer un nouveau `Guess` avec son champ `value` assigné au paramètre `value`
et retourner le `Guess`.

Ensuite, nous implémentons une méthode `value` qui emprunte `self`, qui n'a
aucun autre paramètre, et retourne un `u32`. C'est un type de méthode qui est
parfois appellé un *getter*, car sa fonction est de récupérer (NdT: get) des
données des champs et de les retourner. Cette méthode publique est nécessaire
car le champ `value` du stuct `Guess` est privé. C'est important que le champ
`value` soit privé pour que le code qui utilise la struct `Guess` ne puisse pas
assigner `value` directement : le code en dehors du module *doit* utiliser la
fonction `Guess::new` pour créer une instance de `Guess`, qui s'assure qu'il
n'y a pas de façon pour un `Guess` d'avoir un `value` qui n'a pas été vérifié
par les fonctions dans la fonction `Guess:new`.

Une fonction qui a un paramètre ou qui retourne des nombres uniquement entre 1
et 100 peut ensuite utiliser dans sa signature qu'elle prend en paramètre ou
retourne un `Guess` plutôt qu'un `u32` et n'aura pas besoin de faire des
vérifications supplémentaires dans son code.

## Résumé

Les fonctionnalités de gestion d'erreurs de Rust sont conçues pour vous aider à
écrire du code plus résilient. La macro `panic!` signale que votre programme
fonctionne dans ce mauvaises conditions qu'il ne peut pas gérer et vous permet
de dire au processus de s'arrêter au lieu d'essayer de continuer avec des
valeurs invalides ou incorrectes. Le enum `Result` utilise le système de type
de Rust pour avertir que l'opération peut échouer d'une manière que votre code
pourrait corriger. Vous pouvez utiliser `Result` pour dire au code qui appelle
votre code qu'il a besoin de gérer le résultat et aussi les potentielles
erreurs. Utiliser `panic!` et `Result` de manière appropriée va faire de votre
code plus fiable face à des problèmes inévitables.

Maintenant que vous avez vu les pratiques utiles que la librairie standard
utilise avec les enum `Option` et `Result`, nous allons voir au chapitre
suivant comment les génériques fonctionnent et comment vous pouvez les utiliser
dans votre code.
