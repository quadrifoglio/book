## Comparatif de Performance: Boucles contre Itérateurs

Pour déterminer s'il faut utiliser des boucles ou des itérateurs, nous devons savoir quelle version parmis nos fonctions `search` est la plus rapide: la version avec une boucle `for` explicite ou la version avec des itérateurs?

Nous avons lancé un benchmark en chargeant tout le contenu de *The Adventures of Sherlock Holmes* de Sir Arthur Conan Doyle dans une `String` et en cherchant le mot "the" dans le contenu. Voici les résultats du benchmark sur la version de `search` en utilisant une boucle `for` et un itérateur:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

La version avec l'itérateur était un peu plus rapide! Nous n'expliquerons pas le code de référence ici, car il ne s'agit pas de prouver que les deux versions sont équivalentes, mais d'avoir une idée générale de la manière dont ces deux implémentations se comparent en termes de performances.

Pour un benchmark plus complet, nous vous conseillons de consulter des textes de différentes tailles, avec des mots différents, avec différentes longueurs et avec toutes sortes d'autres variantes. Le point important est le suivant: les itérateurs, bien qu'il s'agisse d'une abstraction de haut niveau, sont compilés à peu près comme si vous aviez écrit vous-même le code de niveau plus bas. Les itérateurs sont l'une des abstractions à *coût zéro* de Rust, c'est-à-dire que l'utilisation de l'abstraction n'impose aucun surcoût d'exécution supplémentaire de la même manière que Bjarne Stroustrup, le concepteur et implémenteur original de C++, définit le *coût zéro*:


> En général, les implémentations de C++ obéissent au principe du coût zéro : Ce que vous
> n'utilisez pas, vous ne payez pas. Et plus encore. Ce que vous utilisez, vous ne pouvez pas mieux le
> coder.  
>
> "Foundations of C++" par Bjarne Stroustrup

Comme autre exemple, le code suivant est tiré d'un décodeur audio. L'algorithme de décodage utilise l'opération mathématique de prédiction linéaire pour estimer les valeurs futures à partir d'une fonction linéaire des échantillons précédents. Ce code utilise une chaîne d'itérateurs pour faire quelques calculs sur trois variables dans le scope: une slice de données `buffer`, un tableau de 12 `coefficients`, et un montant par lequel déplacer les données dans `qlp_shift`. Nous avons déclaré les variables dans cet exemple mais nous ne leur avons pas donné de valeurs; bien que ce code n'ait pas beaucoup de signification en dehors de son contexte, c'est toutefois un exemple concis et concret de la façon dont Rust traduit des idées de haut niveau en code plus bas niveau:

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

Pour calculer la valeur de `prédiction`, ce code itère à travers chacune des 12 valeurs dans `coefficients` et utilise la méthode `zip` pour apparier les valeurs de coefficient avec les 12 valeurs précédentes dans `buffer`. Ensuite, pour chaque paire, nous multiplions les valeurs ensemble, additionnons tous les résultats et shiftons les bits de ` qlp_shift` vers la droite avec la somme obtenue.

Les calculs dans des applications comme les décodeurs audio donnent souvent la priorité aux performances. Ici, nous créons un itérateur à l'aide de deux adaptateurs, puis nous en consommons la valeur. A quel code d'assemblage ce code Rust compilera-t-il? Eh bien, à compter de ce moment, il est compilé au même code assembleur que vous écririez à la main. Il n' y a pas de boucle du tout correspondant à l'itération sur les valeurs dans `coefficients`: Rust sait qu'il y a 12 itérations, donc il "déroule" la boucle. Le *déroulage* est une optimisation qui supprime la surcharge du code de contrôle de boucle et génère un code redondant pour chaque itération de la boucle.

Tous les coefficients sont stockés dans des registres, ce qui signifie qu'il est très rapide d'accéder aux valeurs. Il n' y a pas de vérification des bornes sur les accès au tableau à l'exécution. Toutes ces optimisations que Rust est capable d'appliquer rendent le code produit extrêmement efficace. Maintenant que vous êtes au courant, vous pouvez utiliser des itérateurs et des closures sans crainte! Ils font en sorte que le code semble être de niveau supérieur, mais n'imposent pas de pénalité de performance à l'exécution.

## Résumé

Les closures et les itérateurs sont des fonctionnalités de Rust inspirées par des idées de langage de programmation fonctionnelle. Ils contribuent à la capacité de Rust d'exprimer clairement des idées de haut niveau avec des performances dignes d'un langage de bas niveau. Les implémentations des closures et des itérateurs sont telles que les performances à l'exécution ne sont pas affectées. Cela fait partie de l'objectif de Rust de s'efforcer de fournir des abstractions à coût zéro.

Maintenant que nous avons amélioré l'expressivité de notre projet d'I/O, regardons d'autres fonctionnalités fournis par `cargo` qui nous aideront à partager le projet avec le monde entier.
