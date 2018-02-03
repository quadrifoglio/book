# Les éléments de Langage Fonctionnel en Rust: les Itérateurs et les *Closures*

Le design de Rust s'est inspiré de nombreux langages et techniques existants,
et a été influencé de manière significative par la *programmation
fonctionnelle*. La programmation dans un style fonctionnel implique souvent de
passer des fonctions comme valeurs en les passant dans des arguments en les
retournant à partir d'autres fonctions, en les assignant à des variables pour
une exécution ultérieure, ainsi de suite. Dans ce chapitre, nous ne nous
poserons pas la question de savoir ce qu'est ou n'est pas la programmation
fonctionnelle, mais nous discuterons plutôt de certaines fonctionnalités de
Rust qui sont similaires à celles que de nombreux langages souvent appelés
fonctionnels.

Plus exactement, nous allons aborder :

* Les *Closures* : une forme de fonction, que vous pouvez stocker dans une
  variable
* Les *Itérateurs* : une manière de traiter une série d'éléments.
* Comment utiliser ces deux éléments afin d'améliorer le projet d'I/O du
  Chapitre 12.
* Quelles sont les performances associées à ces caractéristiques. *(Attention,
  spoiler : elles sont plus rapides que vous ne le pensez !)*

D'autres éléments de Rust ont été influencés par le style fonctionnel comme le
pattern matching et les énumérations, que nous avons déjà traité dans d'autres
chapitres. Maîtriser les *closures* et les itérateurs est une partie importante
de l'écriture de code Rust typique et rapide et c'est pourquoi nous
consacrerons un chapitre entier à chacun.
