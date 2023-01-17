# Prédiction de victoire sur League of legends

Ce projet a pour but d'étudier la possibilité de prédir l'équipe gagnante d'une partie de league of legends, avant que celle-ci commence.

Commençons par brièvement expliquer ce qu'est league of legends pour les néophytes:

League of Legends est un jeu de stratégie en temps réel massivement multijoueur (MOBA) où des équipes de 5 joueurs s'affrontent pour détruire la base de l'équipe adverse. Les joueurs (qu'on appelent 'invocateurs' ou encore 'summoners') choisissent un personnage appelé "Champion" avec des compétences uniques et combattent ensemble pour progresser sur le terrain de jeu et détruire la base adverse. Le jeu est gratuit, mais offre des achats intégrés pour des objets cosmétiques. Il existe également un mode de jeu classé où les joueurs peuvent affronter des adversaires de leur niveau pour tenter de monter dans les classements et débloquer des récompenses.

C'est un jeu sur lequel j'ai passé beaucoup de temps, ainsi en me formant à la data science, èl'idée de réaliser ce projet est venue d'elle même.

J'ai créer le dataset moi-même en récupérant les données à partir de l'API de riot games, j'ai stocké ces dernières sur une base de donnée PostgreSQL avec l'outil RDS de AWS, puis j'ai utilisé ces données pour créer un modèle de machine learning. Il ne manque plus qu'une étape : présenter ces résultats sur une interface web. 

# I/ Création du dataset

J'utilise le framework python 'Cassiopeia' qui a été développer dans le but d'utiliser l'API de RIOT GAMES pour league of legends. Celle-ci me permet de récupérer tout type d'informations sur les joueurs, les parties jouées, et plein d'autres choses. 

Ici, le but est de prendre un certain volume de parties qui ont été jouées, et de voir si il est possible de prédire l'équipe qui emporte la partie à partir d'informations sur les joueurs. Comme expliqué dans l'introduction, à chaque début de partie, les joueurs choissisent un cahmpion. Il existe aujourdhui plus de 180 champions, et vous vous doutez que chaque joueur à un niveau différent selon le champion qu'il joue. En général, les joueurs sont capables d'être performant sur 1 à 5 champions, et joueront ceux-xi beaucoup plus que les autres, surtout si ils sont adeptes du mode classé. Il est de savoir commun chez les joueurs, que la maitrise du champion est probablement le facteur de victoire le plus important.

Il existe deux métriques permettant d'évaluer la capacité d'un invocateur à être performant sur un champion : le win-rate, et le score de maitrises. Le win-rate représente le taux de parties gagnés sur celles qui on été jouées avec le champion (en général autour de 50%). Le score de maitrises est une donnée un peu plus particulière, il représente le nombre de parties jouées avec le champion et les performances réalisés avec celui-ci: plus on joue le champion, plus le score de maitrises est élevé. 

Nous allons donc sélectionner ces deux métriques pour chacun des 10 invocateurs qui participent à chaque partie. Les features du modèles seront les 10 win rates des jouers, les 10 scores de maitrises, les deux somme des win rates, les deux somme des maitrises, et un booléen qui représente la victoire (1) ou non(1) de l'équipe bleu contre l'équipe rouge.

L'API de riot fournit les masteries des invocateurs, mais malhereusement pas les win rates, il faut donc les récupérer autrement. 

Une première piste était de récupérer l'historique d'un joueur contenant toutes les games jouées dans l'année, garder celles où il a joué le champion dont on cherche le win rate, et la parcourir pour calculer le taux de victoire. Cette commande est très efficace mais malhereusement beaucoup trop longue, nottament à cause du nombre d'appel à l'API nécessaire.

On a préféré utilisé un outil de web-scrapping pour récupérer ces données sur un des sites spécialisés qui les comportent toute sorte de statistiques sur tous les invoccateurs (ceux-ci ont été construits par des équipes de dev avec l'API de RIOT également, mais je suppose en version prenium). On choisit d'utiliser le site mobalitycs.gg. Tous les sites concernés contiennent. Ainsi le module bs4 seul n'est pas suffisant, j'ai choisi d'utiliser selenium. Cet algorithme reste très lent à cause de l'utilisation de selenium, mais il est presque 10 fois plus rapide que le précédent.

On suit donc le processus dans le fichier Construction_dataset.ipynb, jusqu'a obtenir 1000 données pour un premier essai

# III/ Stockage dans lune bdd
 
 Récupérer les données est excessivement long, et pour diverses raisons (bugs, pc qui séteint, fin de validité de la clé API) il peut parfois s'innterrompre. Plutot que de créer un x fichiers xlsx à chaque fois que l'algo s'arrête, j'ai décidé d'utiliser le service RDS de AWS pour stocker dans le cloud une base de données PostgreSQL et y envoyer les dernières lignes sorties à chaque fin d'algorithme. Le fichier Stockage data.ipynb explique cette démarche.

# IV/ Entrainement du modèle

 On arrive à l'avant-dernière étape du projet, c'est à dire (enfin) entrainer un algorithme. Ici on va utiliser plusieurs algorthme de classification classiques (Régression logistique, arbre de décision, SVC, random forest). On va créer un modèle à partir de tous ces algos, comparer leur performances et sélectionner le plus performant.

# V/ Résultats et conclusion

On obtient jusqu'a 82% d'efficacité pour l'arbre de décision. Très honnetement c'est une grosse réussite, car l'objectif était simplement d'avoir un résultat supérieur à 50% (win rate moyen), ici on arrive avec une plutôt bonne accuracy à prévoir l'issue d'une partie de league of legends. Il est fort possible d'obtenir de meilleur résultats en utilisant des algortihmes plus compliqués de deep learning, cependant, je n'en comprends pas encore l'application.

Au niveau des features il serait intéressant d'ajouter une variable correspondant à la forme du joueur en regardant sont win rates sur les 3 dernières parties. Le mental est un élément très important dans lol, ainsi un jioueur qui vient de perdre 3 parties de suite est très succeptible d'être énervé et jouera moins bien, contrairement à un joueur qui est sur une série de victoire, qui sera de bonne humeur et détendu.

La prochaine étape est de créer une fonction prévision(invocateur) qui récupère les données d'une partie en cours et prévois la victoire ou non à partir du modèle. Elle serait accesible sur une interface web (avec streamlit par exemple, ou même flask/django quand je m'y serai formé). Il faudrait aussi grandement élargir le dataset pour avoir un modèle plus précis, mais le soucis étant qu'une nouvelle saison viens de commencer, et les sites où je récupères les win rates ne montrent plus les performances de la dernière saisons, qui étaient très complètes, mais maintenant plus du tout. Je pourrais finaliser ce projet une fois que quelque mois auront passé et que l'historique des joueurs sera à nouveau bien fourni.

