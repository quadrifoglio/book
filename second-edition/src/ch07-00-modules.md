# Utiliser les Modules pour réutiliser et organiser le code

Quand vous commencez à écrire des programmes en Rust, votre code pourrait
demeurer uniquement dans la fonction `main`. Au fur et à mesure que votre code
grossit, vous pourriez éventuellement déplacer des fonctionnalités dans
d'autres fonctions pour les réutiliser et améliorer son organisation. En
découpant votre code en petits morceaux, chaque morceau devient plus facile à
comprendre. Mais qu'est-ce qui se passe quand vous avez trop de fonctions ?
Rust dispose d'un système de modules qui permet la réutilisation du code de
manière organisée.

De la même façon que vous déplacez des lignes de code dans une fonction, vous
pouvez déplacer des fonctions (et d'autres codes, comme les structs et les
enum) dans différents modules. Un *module* est un espace de nom qui contient
des définitions de fonctions ou de types, et vous pouvez choisir quelles
définitions sont visibles en dehors de leur module (elle est publique) ou non
(elle est privée). Voici un aperçu du fonctionnement des modules :

* Le mot-clé `mod` déclare un nouveau module. Le code dans le module apparaît
  immédiatement après cette déclaration à l'intérieur d'accolades, ou dans un
  autre fichier.
* Par défaut, les fonctions, types, constantes et modules sont privés. Le
  mot-clé `pub` rends cet élément public et par conséquent visible en dehors de
  son espace de nom.
* Le mot-clé `use` remonte des modules, ou des définitions à l'intérieur des
  modules, dans la portée actuelle pour que ce soit plus facile de s'y référer.

Nous allons examiner chacune de ces parties pour voir comment elles s'intègrent
dans toute la chaîne.
