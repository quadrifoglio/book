# Les éléments de langage fonctionnel en Rust: les Itérateurs et les *Closures*
<!-- Are closures unique to Rust? -->
<!-- No, they're from functional languages, which is why they're discussed in
this chapter. Do you have a suggestion on how to make that clearer than the
text in the intro paragraph here? /Carol -->

Le design de Rust s'est inspiré de nombreux langages et techniques existants,
et a été influencé de manière significative par la *programmation fonctionnelle*. Programmer dans un style fonctionnelle implique souvent de passer des fonctions en tant que valeurs en argument ou de retourner en fin de fonction une autre fonction, ainsi que d'assigner des fonctions à   des variables pour un usage plus lointain...etc Le but ici n'étant pas de débattre exactement de ce qu'est ou n'est pas la programmation fonctionnelle, le but est de montrer plusieurs caractéristiques de Rust qui sont similaires à plein de langages considérés comme fonctionnels.

Plus exactement, nous allons couvrir:

* Les *Closures*: une fonction pouvant être assignée et conservée dans une variable.
* Les *Itérateurs*: une manière de traiter une série d'éléments.
* Comment utiliser ces éléments afin d'améliorer le projet sur les I/O du Chapitre 12.
* Les performances associés à ces caractéristiques. Attention: elles pourraient être plus rapides que vous ne le pensez.

D'autres éléments de Rust ont été influencés par le style fonctionnel comme la reconnaissance de pattern (ou pattern matching) et les énumérations, que nous avons déjà couvert dans d'autres chapitres. Maîtriser les *closures* et les itérateurs est une partie importante permettant d'écrire du code Rust idiomatique et rapide, et c'est pourquoi nous dévouons un chapitre entier à ce sujet.
