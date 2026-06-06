---
layout: default
parent: Conception_Pami
nav_order: 2
title: Version Finale
---

## Difficultés rencontrées et notre démarche
La conception de ce PAMI nous a demandé de faire pas mal d'essais et de revoir nos plans en cours de route. Voici les principales étapes de notre réflexion et les problèmes que nous avons dû résoudre :

1. Le point de départ et l'agencement des pièces
Au tout début de la conception, nous sommes partis d'une base mécanique déjà existante : la base des roues conçue par les deuxièmes années. À partir de ce point de départ, tout le défi a été d'imaginer et de modéliser une structure autour. Il fallait trouver le moyen de positionner l'imposante batterie de 10 000 mAh et la carte électronique au-dessus de cette base, tout en réussissant à fermer le robot sans dépasser les limites de taille du règlement.

2. La stabilité du robot
Pour réussir à faire rentrer le robot dans un périmètre de 600 mm, nous avons été obligés d'empiler les composants à la verticale. Le souci avec cette forme, c'est que le centre de gravité se retrouve assez haut, ce qui augmente le risque de voir le robot basculer lors de ses déplacements. Pour contrer ce problème, nous avons fait attention à placer les éléments les plus lourds (la batterie et les moteurs NEMA 10) tout en bas du châssis.

3. Le problème de la carte électronique
Pendant nos phases de tests, la carte de développement que l'on utilisait a grillé. Ce problème s'est finalement transformé en avantage : l'ancienne carte était de toute façon très encombrante. Nous avons alors pu récupérer les nouvelles cartes (ESP32-S3) fabriquées par les deuxièmes années. Comme elles étaient pensées spécifiquement pour les PAMI, elles étaient beaucoup plus petites. C'est ce changement qui nous a permis de gagner la place nécessaire pour tout faire rentrer dans le châssis et finaliser la conception.

## L'aboutissement : Le PAMI Optimisé (Version Noir)

Suite à l'abandon du prototype initial, tous nos efforts se sont concentrés sur la création d'un modèle unique, capable de rapporter un maximum de points tout en occupant le volume le plus restreint possible. C'est ainsi qu'est née notre version finale, reconnaissable à son châssis noir.

### Un châssis monobloc pensé au millimètre

Pour réussir à faire rentrer six PAMI dans la zone de départ, nous ne pouvions plus nous permettre le moindre espace vide. La structure finale a donc été entièrement modélisée d'un seul bloc et imprimée en 3D. Cette coque noire sur-mesure a permis d'encastrer chaque élément comme dans un puzzle :

* La base (L'énergie et la mobilité) : 
La imposante batterie externe de 10 000 mAh et les deux moteurs pas-à-pas NEMA 10 sont logés tout au fond du châssis. Ce choix de conception abaisse drastiquement le centre de gravité, empêchant le robot de basculer lors des freinages ou des rotations.

* Le cœur (L'électronique) : 
Juste au-dessus, vient se glisser la fameuse carte électronique sur-mesure réalisée par les étudiants de deuxième année. Conçue exclusivement pour le PAMI, elle s'intègre parfaitement dans la largeur restreinte du robot, là où l'ancienne carte de développement bloquait tout.

* La face avant (La détection) : 
Le capteur à ultrasons possède son propre logement encastré à l'avant, protégeant ses composants tout en lui laissant un champ de vision totalement dégagé.

* Le pont supérieur et les interfaces :
La partie haute du robot vient fermer la structure et maintient le panneau de commande. C'est ici que l'on retrouve tous les éléments obligatoires pour le lancement du match, accessibles facilement en un coup d'œil lors des 3 minutes de préparation : le Bouton d'Arrêt d'Urgence rouge, la tirette de démarrage, et l'interrupteur permettant de basculer d'une stratégie à l'autre selon notre couleur d'équipe.

* Bilan et Homologation :
Cette version finale a totalement rempli notre cahier des charges. L'optimisation extrême de l'espace nous a permis de respecter strictement le gabarit réglementaire imposé par les arbitres : une hauteur maintenue sous les 150 mm et un périmètre ne dépassant pas les 600 mm au départ. Le pari est réussi : nos six PAMI optimisés rentrent parfaitement dans la zone de départ aux côtés de notre robot principal, prêts à se déployer de manière autonome sur la table.