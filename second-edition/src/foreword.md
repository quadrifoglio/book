# Préface

Ce ne fut pas toujours très clair, mais le langage Rust a pour but de permettre
*l'autonomisation* : quelque soit le programme que vous êtes actuellement en train
d'écrire, Rust vous permet d'aller plus loin, de programmer avec confiance dans un
plus large champ de domaines qu'auparavant.

Prenons pour l'exemple, le travail de "system-level" qui gère tous les détails
bas niveaux de la gestion de la mémoire, de la représentation des données, et de
l'exécution en parallèle. Traditionnellement, ce domaine de la programmation
est souvent vu comme mystique, accessible seulement à un petit nombre d'élus qui ont
passé de nombreuses années à parfaire leur savoir pour éviter ces infâmes écueils. Et
même ceux qui ont été introduit à ces arcanes les applique avec précaution, de peur
d'ouvrir leur code aux exploitations, plantages ou corruptions.

Rust breaks down these barriers by eliminating the old pitfalls and providing a
friendly, polished set of tools to help you along the way. Programmers who need
to “dip down” into lower-level control can do so with Rust, without taking on
the customary risk of crashes or security holes, and without having to learn
the fine points of a fickle toolchain. Better yet, the language is designed to
guide you naturally towards reliable code that is efficient in terms of speed
and memory usage.

Programmers who are already working with low-level code can use Rust to raise
their ambitions. For example, introducing parallelism in Rust is a relatively
low-risk operation: the compiler will catch the classical mistakes for you. And
you can tackle more aggressive optimizations in your code with the confidence
that you won’t accidentally introduce crashes or exploits.

But Rust isn’t limited to low-level systems programming. It’s expressive and
ergonomic enough to make CLI apps, web servers, and many other kinds of code
quite pleasant to write — you’ll find simple examples of both later in the
book. Working with Rust allows you to build skills that transfer from one
domain to another; you can learn Rust by writing a web app, then apply those
same skills to target your Raspberry Pi.

This book fully embraces the potential of Rust to empower its users. It’s a
friendly and approachable text intended to help you level up not just your
knowledge of Rust, but also your reach and confidence as a programmer in
general. So dive in, get ready to learn—and welcome to the Rust community!

— Nicholas Matsakis and Aaron Turon
