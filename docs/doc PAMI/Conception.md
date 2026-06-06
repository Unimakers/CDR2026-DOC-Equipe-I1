---
layout: default
title: Conception
nav_order: 1
parent: PAMI
has_children: true
---

## 1 Conception et et méchanique du PAMI


Les PAMI possédent des contraintes méchanique ainsi que réglementaire. Au début du match, le PAMI doit reposer uniquement sur la table. Sa hauteur initiale est limitée à 150 mm. Son périmètre de départ ne doit pas dépasser 600 mm. Toutefois, sa taille globale doit être supérieure à celle d'un cube de 60 mm de côté.
Pendant le jeu, le PAMI a le droit de se déployer. Son périmètre peut alors s'étendre pour atteindre une limite de 700 mm. Son altitude maximale, une fois déployé, est fixée à 350 mm.
Concernant sa conception, la masse du PAMI ne doit jamais excéder 1,5 kg. Il faut impérativement prévoir une surface libre de 30 x 30 mm pour y coller le numéro de stand. Enfin, le robot doit être totalement autonome. Il est strictement interdit de le piloter avec une télécommande ou depuis l'extérieur de la table.

Pour la conception, nous avons cherché à optimiser l'espace afin de pouvoir intégrer plusieurs PAMI : il nous en fallait au moins six. C'est pourquoi nous avons opté pour un design le plus compact possible, tout en respectant les contraintes imposées par le règlement. Nous avons conçu notre PAMI autour des éléments dont nous disposions, notamment la carte électronique, qui a été notre principale contrainte. Ensuite, nous avons intégré la batterie externe, le BAU, la tirette, ainsi que le switch. Enfin, nous avons veillé à ce que le centre de gravité du robot soit le plus bas possible, afin de lui garantir un équilibre optimal.


## 1.1 Structure et Châssis

Nous avons opté pour une impression 3D dans sa version finale noire. Nous avons fait ce choix car le règlement impose un périmètre maximum de 600 mm et une hauteur maximale de 150 mm, et la place dans la zone de départ était critique pour y faire rentrer les six unités ainsi que le robot principal. Cela nous a permis de valider l'homologation dimensionnelle auprès des arbitres et d'optimiser totalement le volume disponible au départ. De plus, nous avons procédé à un abaissement maximal du centre de gravité en plaçant les composants lourds, comme les moteurs et la batterie, au plus près du sol. L'empilement vertical imposé par la réduction de la largeur augmentait le risque de basculement lors des phases de freinage ou de rotation. Grâce à cela, nous avons obtenu une excellente stabilité dynamique du robot, évitant ainsi les chutes et garantissant des trajectoires fiables sur la table. 

## 1.2 Motorisation et Actionneurs

 Concernant la motorisation, l'utilisation de moteurs pas-à-pas de format NEMA 10 a été privilégiée pour la propulsion. Le choix de ce modèle précis s'explique par la nécessité de trouver un actionneur offrant un couple mécanique suffisant pour déplacer le robot avec fluidité, tout en conservant des dimensions très réduites pour s'insérer parfaitement dans notre châssis exigu. De plus, la stratégie de fin de match exigeait d'atteindre des zones précises, comme les gardes-manger, et les moteurs à courant continu classiques manquaient de précision sans l'ajout d'encodeurs complexes. L'utilisation de ces NEMA 10 nous a ainsi apporté un contrôle strict et prévisible des distances parcourues ainsi que des angles de rotation, sans sacrifier notre besoin crucial de compacité. Nous avons également intégré un servomoteur dédié. Celui-ci permet au PAMI de réaliser l'action mécanique de déploiement requise par le cahier des charges de notre stratégie de jeu. Son intégration nous a offert un déploiement mécanique fiable et très facilement contrôlable via le signal envoyé par la carte. 

## 1.3 Électronique et Alimentation

Pour la partie électronique, nous avons dû procéder au remplacement de la carte standard par une carte sur-mesure, conçue par les étudiants de deuxième année. La carte de développement initiale a malheureusement grillé lors des tests et s'est avérée de toute façon beaucoup trop volumineuse pour le châssis exigu du PAMI. Ce changement nous a fait gagner un espace interne massif, essentiel pour finaliser la compacité du robot. L'alimentation du système repose quant à elle sur une batterie externe de 10 000 mAh. Il était indispensable de trouver une source d'énergie de 5V stable, très compacte, et qui n'obligeait pas à démonter toute l'électronique pour être rechargée. Cela nous a garanti une autonomie surdimensionnée pour enchaîner tous les matchs de la compétition sans aucune appréhension, avec l'avantage d'un rechargement simple via USB. 

Détails de la Carte Électronique L'architecture de notre carte électronique s'articulait autour d'un microcontrôleur XIAO ESP32-S3. Nous avons choisi ce composant central car il fallait un véritable cerveau capable de traiter de multiples informations tout en occupant un espace minimal. Cela nous a permis de centraliser toutes les commandes vitales du robot dans une puce de très petite taille. Pour la gestion des déplacements, le circuit intégrait des connexions spécifiques pour diriger les moteurs pas-à-pas, en envoyant des signaux de direction et d'impulsion. Il était nécessaire de séparer les commandes du côté gauche et du côté droit pour que le robot puisse tourner et s'orienter avec précision. Du côté de la perception, des broches étaient dédiées à la réception des signaux du capteur à ultrasons. Le robot devait absolument pouvoir envoyer un son et écouter son écho pour calculer les distances de manière autonome. Cela lui a donné la capacité d'éviter les obstacles sans aucune aide extérieure. Enfin, le schéma comprenait une connexion pour la tirette de démarrage ainsi qu'une diode de sécurité. La tirette était obligatoire pour garantir un départ synchronisé au début du match, tandis que la diode protégeait le système contre les retours de courant lors de la programmation sur ordinateur. 

## 1.4 Capteurs et Interfaces Utilisateur

Du côté de la détection, l'installation d'un capteur à ultrasons en façade s'est imposée d'elle-même. Le règlement de la compétition exigeant une stricte autonomie des PAMI sans aucune commande externe, ce capteur donne au robot la capacité de détecter les obstacles de manière autonome, afin d'adapter sa trajectoire ou de déclencher des arrêts de sécurité. Nous avons achevé la conception en intégrant un panneau supérieur comprenant un interrupteur classique, une tirette de démarrage et un Bouton d'Arrêt d'Urgence. Ces éléments étaient nécessaires pour se conformer rigoureusement aux normes de sécurité et aux procédures de lancement. L'interrupteur a d'ailleurs été spécifiquement câblé pour la sélection de la stratégie, ce qui permet à l'équipe de choisir instantanément le comportement du robot en fonction de la couleur assignée par les arbitres juste avant le match, tout en assurant un départ parfaitement synchronisé. 

 