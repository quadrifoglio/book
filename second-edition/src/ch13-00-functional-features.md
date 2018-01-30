# Les éléments de Langage Fonctionnel en Rust: les Itérateurs et les *Closures*
<!-- Are closures unique to Rust? -->
<!-- No, they're from functional languages, which is why they're discussed in
this chapter. Do you have a suggestion on how to make that clearer than the
text in the intro paragraph here? /Carol -->

Le design de Rust s'est inspiré de nombreux langages et techniques existants,et a été influencé de manière significative par la *programmation fonctionnelle*. La programmation dans un style fonctionnel implique souvent de passer des fonctions comme valeurs en les passant dans des arguments en les retournant à partir d'autres fonctions, en les assignant à des variables pour une exécution ultérieure ...etc. Dans ce chapitre, nous n'aborderons pas la question de savoir ce qu'est ou n'est pas la programmation fonctionnelle, mais nous discuterons plutôt de certaines fonctionnalités de Rust qui sont similaires à celles de nombreux langages souvent appelés fonctionnels.

Plus exactement, nous allons couvrir:

* Les *Closures*: une construction de type fonction que vous pouvez stocker dans une variable
* Les *Itérateurs*: une manière de traiter une série d'éléments.
* Comment utiliser ces deux éléments afin d'améliorer le projet d'I/O du Chapitre 12.
* Les performances associées à ces caractéristiques. Attention: elles sont plus rapides que vous ne le pensez!

D'autres éléments de Rust ont été influencés par le style fonctionnel comme la reconnaissance de pattern (ou pattern matching) et les énumérations, que nous avons déjà traitées dans d'autres chapitres. Maîtriser les *closures* et les itérateurs est une partie importante de l'écriture de code Rust idiomatique et rapide et c'est pourquoi nous leur consacrons un chapitre entier à ce sujet.
