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
éviter les problèmes de concurrence étaFR
9+
ient deux challenges séparés qui devaient
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

> Note : pour des raisons de simplicité, nous qualifirons de problèmes
> *de concurence* plutôt que d'utiliser précisément les termes
> *de concurrence et/ou de parallélisme*. Si ce livre portait uniquement sur la
> concurence et/ou le parallélisme, nous serions plus précis. Pour ce chapitre,
> merci de remplacer mentalement *concurrence* par
> *concurrence et/ou parallélisme*.

De nombreux languages proposent des solutions dogmatiques pour gérer les
problèmes de concurence. Par exemple, Erlang a une fonctionnalité élégante pour
échanger des messages pour la concurence mais partage l'état entre les tâches de
manière obscure. Implémenter seulement un sous-ensemble de solutions possibles
est une stratégie acceptable pour les languages de haut niveau. Cependant, les
languages de bas niveau se doivent de fournir la solution qui offre la meilleur
performance quelle que soit la situation et ne doit pas être influencée par le
matériel. C'est pourquoi Rust offre une variété d'outils pour gérer les
problèmes de manière appropriée à votre situation et à vos besoins.

Voici les sujets que nous allons aborder dans ce chapitre :
* Comment créer des tâches pour exécuter différentes parties du code au même
  moment
* La concurence avec *l'envoi de messages*, où des canaux envoient des messages
  entre les tâches
* La concurence avec *le partage d'état*, où plusieures tâches accèdent à
  certaines parties des données
* Les traits ` Sync` et `Send`, qui étendent les garanties de Rust pour la
  concurence aux types définis par l'utilisateur ainsi qu'aux types fournis par
  la bibliothèque standard
