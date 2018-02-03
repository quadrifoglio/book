## Amélioration de notre Projet d'I/O

Grâce à ces nouvelles connaissances sur les itérateurs, nous pouvons améliorer
le projet d'I/O du chapitre 12 en utilisant des itérateurs pour rendre certains
endroits du code plus clairs et plus concis. Voyons comment les itérateurs
peuvent améliorer notre implémentation de la fonction `Config::new` et de la
fonction `search`.

### Suppression de l'appel à `clone` à l'aide d'un Itérateur

Dans le Listing 12-6, nous avons ajouté du code qui a pris une *slice* de
valeurs `String` et créé une instance de la structure `Config` en utilisant les
index de la *slice* et en clonant les valeurs, permettant ainsi à la structure
`Config` de posséder ces valeurs. Dans le Listing 13-24, nous avons reproduit
l'implémentation de la fonction `Config:: new` comme dans le Listing 12-23 à la
fin du chapitre 12:

<span class="filename">Fichier: src/lib.rs</span>

```rust,ignore
impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

<span class="caption">Listing 13-24: Reproduction de la fonction `Config::new`
à la fin du Chapitre 12</span>

À ce moment-là, nous avions dit de ne pas s'inquiéter des appels inefficaces à
`clone` parce que nous les supprimerions à l'avenir. Eh bien, le moment est
venu!

Nous avions besoin de `clone` ici parce que nous avons une *slice* avec des
éléments `String` dans le paramètre `args`, mais la fonction `new` ne possède
pas `args`. Pour rendre la propriété d'une instance `Config`, nous avons dû
cloner les valeurs des champs `query` et `filename` de `Config` pour que
l'instance `Config` puisse posséder ses propres valeurs.

Avec nos nouvelles connaissances sur les itérateurs, nous pouvons changer la
fonction `new` pour prendre en charge un itérateur comme argument au lieu
d'emprunter une *slice*. Nous utiliserons la fonctionnalité `next` des
itérateurs au lieu du code qui vérifie la longueur de la *slice* et les index
des emplacements spécifiques. Ceci clarifiera ce que la fonction `Config:: new`
fait parce que l'itérateur accédera aux valeurs.

Une fois que `Config::new` prend possession de l'itérateur et cesse d'utiliser
les opérations d'indexation qui empruntent, nous pouvons déplacer les valeurs
`String` de l'iterator dans `Config` plutôt que d'appeler `clone` et de faire
une nouvelle allocation.

### Utilisation directe de l'Itérateur renvoyé

Ouvrez le fichier *src/main.rs* de votre projet d'I/O, qui devrait ressembler à
ceci:

<span class="filename">Fichier: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}
```

Nous allons changer le début de la fonction `main` que nous avions dans le
Listing 12-24 à la fin du Chapitre 12 pour le code dans le Listing 13-25. Ceci
ne compilera pas encore jusqu'à ce que nous mettions également à jour
`Config::new`:

<span class="filename">Fichier: src/main.rs</span>

```rust,ignore
fn main() {
    let config = Config::new(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}
```

<span class="caption">Listing 13-25: Transmission de la valeur de retour de
`env::args` à `Config::new`.</span>

La fonction `env::args` retourne un itérateur! Plutôt que de collecter les
valeurs de l'itérateur dans un vecteur et de passer ensuite une *slice* à
`Config::new`, maintenant nous passons la propriété de l'itérateur directement
de `env::args` à `Config::new`.

Ensuite, nous devons mettre à jour la définition de `Config::new`. Dans le
fichier *src/lib.rs* de votre projet d'I/O, modifions la signature de
`Config::new` pour ressembler au Listing 13-26. Ceci ne compilera pas encore
parce que nous devons mettre à jour le corps de la fonction:

<span class="filename">Fichier: src/lib.rs</span>

```rust,ignore
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        // --snip--
```

<span class="caption">Listing 13-26:Mise à jour de la signature de
`Config::new` pour recevoir un itérateur</span>

La documentation standard de la bibliothèque pour la fonction `env::args`
montre que le type de l'itérateur qu'elle renvoie est `std::env::Args`. Nous
avons mis à jour la signature de la fonction `Config::new` pour que le
paramètre `args` ait le type `std::env::Args` au lieu de `&[String]`. Etant
donné que nous prenons la propriété de `args` et que nous allons muter `args`
en itérant dessus, nous pouvons ajouter le mot-clé `mut` dans la spécification
du paramètre `args` pour le rendre mutable.

#### Utilisation des Méthodes du Trait `Iterator` au lieu de l'Indexation

Ensuite, nous allons réparer le corps de `Config::new`. La documentation
standard de la bibliothèque mentionne aussi que `std::env::Args` implémente le
trait `Iterator`, donc nous savons que nous pouvons appeler la méthode `next`
dessus! Le listing 13-27 met à jour le code du Listing 12-23 afin d'utiliser la
méthode `next`:

<span class="filename">Fichier: src/lib.rs</span>

```rust
# use std::env;
#
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }
#
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

<span class="caption">Listing 13-27: Changement du corps de `Config::new` afin
d'utiliser les méthodes d'itération</span>

Rappelez-vous que la première valeur dans ce qui est retourné par `env::args`
est le nom du programme. Nous voulons ignorer cette valeur et passer à la
valeur suivante, donc d'abord nous appelons `next` et nous ne faisons rien avec
la valeur de retour. Deuxièmement, nous appelons `next` sur la valeur que nous
voulons mettre dans le champ `query` de `Config`. Si `next` renvoie un `Some`,
nous utilisons un `match` pour extraire la valeur. S'il retourne `None`, cela
signifie qu'il n'y a pas assez d'arguments donnés et nous revenons plus tôt
avec une valeur `Err`. De même pour la valeur `filename`.

### Rendre le Code plus Clair avec des Adaptateurs d'Itérateurs

Nous pouvons également tirer parti des itérateurs dans la fonction `search` de
notre projet d'I/O, qui est reproduite ici dans le Listing 13-28, comme elle
l'était dans Listing 12-19 à la fin du chapitre 12:

<span class="filename">Fichier: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

<span class="caption">Listing 13-28: La mise en uvre de la fonction `search`
telle qu'au chapitre 12</span>

Nous pouvons écrire ce code de façon plus concise en utilisant des méthodes
d'adaptateur d'itérateur. Ce faisant, nous évitons aussi d'avoir un vecteur
intermédiaire mutable: `results`. Le style de programmation fonctionnelle
préfère minimiser la quantité d'état modifiable pour rendre le code plus clair.
Supprimer l'état mutable pourrait nous aider à faire une amélioration future
pour que la recherche se fasse en parallèle, car nous n'aurions pas à gérer
l'accès simultané au vecteur `results`. Le Listing 13-29 montre ce changement:

<span class="filename">Fichier: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

<span class="caption">Listing 13-29: Utilisation de méthodes d'adaptateurs
d'itérateurs dans l'implémentation de la fonction `search`</span>

Souvenez-vous que le but de la fonction `search` est de renvoyer toutes les
lignes dans `contents` qui contiennent `query`. Comme dans l'exemple de
`filter` dans le Listing 13-19, nous pouvons utiliser l'adaptateur `filter`
pour ne garder que les lignes pour lesquelles `line.contains(query)` renvoie
true. Nous collectons ensuite les lignes correspondantes dans un autre vecteur
avec `collect`. Bien plus simple ! N'hésitez pas à faire le même changement
pour utiliser les méthodes d'itération dans la fonction
`search_case_insensitive`.

Logiquement la question suivante est de savoir quel style utiliser dans votre
propre code et pourquoi: l'implémentation originale dans le Listing 13-28 ou la
version utilisant les itérateurs dans le Listing 13-29. La plupart des
développeurs Rust préfèrent utiliser le style itérateur. C'est un peu plus
difficile au début, mais une fois que vous avez une idée des différents
adaptateurs d'itérateur et de ce qu'ils font, les itérateurs peuvent être plus
faciles à comprendre. Au lieu de jouer avec les différentes boucles et de
construire de nouveaux vecteurs, le code se concentre sur l'objectif de haut
niveau de la boucle. Cette abstraction élimine une partie du code trivial, de
sorte qu'il soit plus facile de voir les concepts qui sont uniques à ce code,
comme la condition de filtration que chaque élément de l'itérateur doit passer.

Mais les deux implémentations sont-elles réellement équivalentes? L'hypothèse
intuitive pourrait être que la boucle de plus bas niveau sera plus rapide.
Intéressons nous aux performances.
