# Comprendre l'appropriation (en anglais "ownership")

L'appropriation est la fonctionnalité la plus caractéristique de Rust, et elle
permet à Rust de garantir la sécurité des accès mémoires sans avoir besoin d'un
ramasse-miettes (en anglais "garbage collector"). Par conséquent, il est important de comprendre comment 
l'appropriation fonctionne dans Rust. Dans ce chapitre nous allons parler de
l'appropriation ainsi que plusieurs fonctionnalités associées : l'emprunt (en anglais "borrow"), les
slices, et comment Rust conserve les données en mémoire.
