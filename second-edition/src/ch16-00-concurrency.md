# La concurrence intrépide

Gérer de manière sûre et efficace la concurrence est un autre des objectifs de
Rust. La *programmation simultanée (NdT: concurrent programming)*, dans laquelle
les différents éléments d'un programme s'exécute indépendamment, et la
*programmation parralèle (NdT: parallel programming)*, dans laquelle les
différents éléments s'exécutent en même temps, ont pris de l'importance au fur
et à mesure que les ordinateurs tirent profit de leurs processeurs multiples.
Historiquement, programmer dans ces domaines a toujours été difficile et source
d'erreurs : Rust espère changer cela.

Au départ, l'équipe de Rust pensait que veiller à la sécurité de la mémoire et
éviter les problèmes de concurrence étaient deux challenges séparés qui devaient
être résolus avec des méthodes distinctes. Progressivement, l'équipe a découvert
que l'appropriation *(NdT: ownership)* et le système de typage sont des jeux
d'outils puissants pour gérer la sécurité de la mémoire *et* les problèmes de
concurrence ! En capitalisant sur l'appropriation et la vérification de type, de
nombreuses erreurs de concurrence deviennent des erreurs au moment de la
compilation avec Rust plutôt que des erreurs au moment de l'exécution. Par
conséquent, plutôt que vous faire perdre beaucoup de temps à essayer de
reproduire les conditions exactes que lors de l'apparition de l'erreur de
concurrence, le code incorrect va refuser de compiler et afficher une erreur
expliquant le problème. Au final, vous pouvez corriger votre code pendant que
vous travaillez dessus plutôt que potentiellement après qu'il est été envoyé en
production. Nous avons surnommé élément de Rust la *concurence intrépide (NdT :
fearless concurrency)*. La concurence intrépide vous permet d'écrire du code qui
est exempt de bogues discrets et qu'il est facile de remanier *(NdT : refactor)*
sans introduire de nous bogues.

> Note: For simplicity’s sake, we’ll refer to many of the problems as
> *concurrent* rather than being more precise by saying *concurrent and/or
> parallel*. If this book were about concurrency and/or parallelism, we’d be
> more specific. For this chapter, please mentally substitute *concurrent
> and/or parallel* whenever we use *concurrent*.

Many languages are dogmatic about the solutions they offer for handling
concurrent problems. For example, Erlang has elegant functionality for
message-passing concurrency but has only obscure ways to share state between
threads. Supporting only a subset of possible solutions is a reasonable
strategy for higher-level languages, because a higher-level language promises
benefits from giving up some control to gain abstractions. However, lower-level
languages are expected to provide the solution with the best performance in any
given situation and have fewer abstractions over the hardware. Therefore, Rust
offers a variety of tools for modeling problems in whatever way is appropriate
for your situation and requirements.

Here are the topics we’ll cover in this chapter:

* How to create threads to run multiple pieces of code at the same time
* *Message-passing* concurrency, where channels send messages between threads
* *Shared-state* concurrency, where multiple threads have access to some piece
  of data
* The `Sync` and `Send` traits, which extend Rust’s concurrency guarantees to
  user-defined types as well as types provided by the standard library
