# Gestion des Erreurs

L'engagement de Rust envers la fiabilité concerne aussi la gestion des erreurs.
Les erreurs font partie de la vie des programmes informatiques, c'est pourquoi
Rust a des dispositifs pour gérer les situations où les choses se passent mal.
Dans de nombreux cas, Rust exige que vous anticipiez les erreurs possibles et
que vous preniez des dispositions avant de compiler votre code. Cette exigence
rends votre programme plus résiliant en s'assurant que vous détectiez et gérez
les erreurs correctement avant même que vous déployez votre code en production
!

Rust classe les erreurs dans deux catégories principales : les erreurs
*récupérables* et *irrécupérables*. Les erreurs récupérables se produisent
dans des situations dans lesquelles il est utile de signaler l'erreur à
l'utilisateur et de relancer l'opération, comme par exemple une erreur lorsque
un fichier n'a pas été trouvé.
Les erreurs irrécupérables sont toujours liées à des bogues, comme essayer
d'accéder à un caractère au-delà de la fin d'un tableau.

La plupart des langages de programmation ne font pas de distinction entre ces
deux types d'erreurs et les gèrent de la même manière, comme les exceptions.
Rust n'a pas d'exceptions. À la place, il a les valeurs `Result<T, E>` pour les
erreurs récupérables, et la macro `panic!` qui arrête l'exécution quand il
se heurte à des erreurs irrécupérables. Nous allons commencer ce chapitre par
expliquer l'utilisation de `panic!`, puis ensuite les valeurs de retour
`Result<T, E>`. De plus, nous allons voir les arguments à prendre en compte
pour choisir si nous devons essayer de récupérer une erreur ou d'arrêter
l'exécution.
