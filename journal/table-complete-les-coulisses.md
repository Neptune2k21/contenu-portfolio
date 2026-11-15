---
title: Table complète, les coulisses
summary: Ce trimestre à l'ESGI, on construit à deux une application Android qui sert le plat, le verre et le dessert. Tout est parti d'une question que je n'aurais pas dû poser.
tags: [android, kotlin, compose, esgi]
date: 2026-11-15
---

Tout est parti d'une question que je n'aurais pas dû poser.

Début de trimestre, M1 à l'ESGI : il faut construire une application Android à deux, en Kotlin et Jetpack Compose. Mon binôme avait déjà une piste.

:::dialogue
::say[J'ai trouvé TheCocktailDB, une API de cocktails. Elle est pas mal.]{who="Mon binôme"}
::say[Pourquoi pas celle de la nourriture ? TheMealDB, elle est pas mal aussi.]{who="Moi" side=me}
::say[Et qu'est-ce qui nous empêche de faire les deux ?]{who="Mon binôme"}
:::

:::avatar{mood=dizzy}
Sur le coup, je proposais de remplacer son idée par la mienne. La sienne les additionnait.
:::

On s'est mis d'accord sur les deux. [TheMealDB](https://www.themealdb.com/) donne le plat, [TheCocktailDB](https://www.thecocktaildb.com/) donne le verre. L'application s'appelle Table complète : chaque jour, elle sert un repas entier, un plat, un verre et un dessert, et elle accompagne la préparation jusqu'à la liste de courses.

Le cahier des charges est rendu depuis le 8 novembre. Il dit ce que l'application fera. Ici, je raconte ce qu'il ne dit pas.

## Zéro serveur

Le sujet exclut tout backend. Les données viennent des deux API et de fichiers embarqués dans l'application. Les filtres, les accords, les menus et la liste de courses tournent sur le téléphone.

Sans serveur à construire, il ne reste que ce que l'utilisateur touche, et c'est là que tout se joue. On ne veut pas d'un moteur de recherche de recettes de plus. On veut une carte de restaurant qui donne envie de passer en cuisine.

:::screens{caption="La carte du jour, la table dressée, le mode cuisine. Les captures suivent le thème du site : passe-le en sombre pour voir l'autre version."}
::phone{src="/ecrits/table-complete/carte-du-jour.webp" dark="/ecrits/table-complete/carte-du-jour-sombre.webp" alt="Écran d'accueil : la salade de pâtes méditerranéenne du jour, avec un Vermentino et un affogato" label="Carte du jour"}
::phone{src="/ecrits/table-complete/accords.webp" dark="/ecrits/table-complete/accords-sombre.webp" alt="Écran des accords : un couvert vu du dessus avec le plat, le vin, le cocktail et le dessert numérotés" label="Accords"}
::phone{src="/ecrits/table-complete/mode-cuisine.webp" dark="/ecrits/table-complete/mode-cuisine-sombre.webp" alt="Mode cuisine : l'étape 3 sur 6 en grand, ses ingrédients et un minuteur en cours" label="Mode cuisine"}
:::

## Ce que les API ne savent pas

C'est la vraie surprise de la rédaction. En dessinant les maquettes, on a affiché des choses qu'aucune des deux API ne connaît. L'écran des accords en est le meilleur exemple : TheMealDB ne sait pas qu'un plat de fruits de mer appelle un vin blanc sec, et TheCocktailDB encore moins.

Alors on l'écrit nous-mêmes. Des tables JSON embarquées dans l'application associent la catégorie et le pays d'un plat à des styles de vin et de cocktail, chaque ingrédient à son rayon, chaque pays à son plat emblématique. Le catalogue des vins, les allergènes et les substitutions viennent aussi de nous.

Le reste se devine dans le texte des recettes. Le matériel et les durées sont repérés par mots-clés (« four », « minutes »). Le nombre de portions n'est jamais donné : on a décidé qu'une recette nourrit quatre personnes, on recalcule les quantités chiffrées, et on laisse « au goût » et « une pincée » tranquilles.

## Trois cartes pour un seul écran

Pour l'identité visuelle, on a redessiné le même écran d'accueil dans trois directions.

::palette{letter=A name="Épicerie" fonts="Bricolage Grotesque condensé · Geist · Geist Mono pour les mesures" colors="Fond #F3F3EF, Encre #141414, Tomate #D62F1C, Basilic #2F6B3F, Citron #F2D64B, Gris #5E5E58"}

::palette{letter=B name="Cave" fonts="Funnel Display pour les titres · Funnel Sans pour le texte" colors="Fond #F1F2EE, Encre #0E100E, Prune #6E1F3B, Chartreuse #C9D96A, Pierre #5B6159, Blanc #FFFFFF"}

::palette{letter=C name="Marché" fonts="Schibsted Grotesk pour les titres · Onest pour le texte et les chiffres" colors="Blanc #FFFFFF, Brume #F2F5F1, Encre #0F1F17, Herbe #16764A, Agrume #FFB020, Sauge #55635B" chosen}

Cave utilise les mêmes polices que ce site. Ce n'est pas elle qu'on a gardée.

Les captures plus haut trahissent un détail : elles sont encore dans la première version du design system, crème et terracotta. Ce n'est pas un oubli. Les couleurs, les polices et les numéros sont centralisés dans le thème Compose, donc changer de direction ne touche aucun écran. Les maquettes ont pu avancer pendant qu'on hésitait encore sur la palette.

## À deux sur le même dépôt

On a découpé par fonctionnalité plutôt que par couche. Je prends les plats : exploration, fiche recette, mode cuisine, liste de courses. Mon binôme prend les boissons : cocktails, vins, accords, composition des menus. L'architecture, le design system, le premier lancement, les favoris et le hors-ligne sont en commun. Chacun va de l'écran jusqu'aux données et aux tests.

Côté Git, les règles tiennent en une ligne : `main` toujours compilable, une branche par fonctionnalité, des commits au format Conventional Commits, et aucune fusion sans pull request relue par l'autre.

À deux, la relecture n'est pas une formalité. C'est le seul moyen de savoir ce qui a changé dans une partie du code que je n'aurais sinon jamais ouverte.

## La suite

Le rendu est prévu à la fin du trimestre, avec une soutenance de trente minutes. La démonstration suivra le parcours d'un utilisateur : premier lancement, carte du jour, fiche recette, mode cuisine, accords, menu, liste de courses, puis passage hors ligne. Les options (suggestions selon la météo, scan de code-barres en magasin, Nutri-Score estimé) attendront que la base tienne debout.

Prochaine entrée quand le premier écran tournera sur un vrai téléphone.
