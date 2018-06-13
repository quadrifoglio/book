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

Rust abat ces barrières en éliminant ces vieux écueils et en fournissant un ensemble
d'outils amicaux qui vous aident tout au long de votre développement. Les développeurs
qui ont besoin de "toucher" aux contrôles bas niveaux peuvent le faire avec
Rust, sans
prendre le risque trop commun de provoquer des plantages ou d'ajouter des failles de
sécurité, et sans à avoir à apprendre en détails toutes les subtilités d'une vague
chaîne d'outils. Encore mieux, le langage est conçu pour vous guider naturellement
vers l'écriture d'un code sûr qui est efficient en termes de vitesse d'exécution
et d'usage de la mémoire.

Les développeurs qui ont déjà travaillé sur du code bas niveau peuvent également 
utiliser Rust pour revoir à la hausse leurs ambitions. Par exemple, introduire
la parallélisme en Rust est une opération relativement peu risquée : le compilateur
va vous signaler toutes les erreurs classiques pour vous. Ainsi vous pourrez vous
attaquer à des optimisations plus agressives de votre code avec la certitude que
vous n'introduisez aucun plantages ou failles de sécurité par accident.

Mais Rust n'est pas seulement limité à la programmation système bas niveau. Il
est suffisamment expressif et ergonomique pour rendre les applications en ligne de
commande, les serveurs web, et bien d'autres types de code très agréables à développer
(vous trouverez des exemples simples de ces deux cas d'utilisation plus loin dans le livre).
Travailler avec Rust vous permet de développer des compétences qui peuvent être appliquées
dans plusieurs domaines ; vous pouvez apprendre le Rust en développant une application web,
puis utiliser ces mêmes acquis pour du développement sur Raspberry Pi.

Ce livre englobe pleinement le potentiel du Rust pour permettre à ses utilisateurs
d'être *autonome*. C'est un ouvrage convivial et accessible destiné à vous aider à
améliorer non seulement votre connaissance du langage, mais aussi votre compétence
et votre confiance cen tant que développeur en général. Alors plongez vous dans ce livre,
préparez vous à apprendre... et bienvenue dans la communauté Rust !

— Nicholas Matsakis and Aaron Turon
