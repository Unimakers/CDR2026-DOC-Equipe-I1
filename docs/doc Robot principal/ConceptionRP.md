---
layout: default
title: Conception
nav_order: 1
parent: Robot Principal
---

# 🔧 Conception — De la réflexion à l'assemblage

> **Équipe :** UNIMAKERS | **CDR 2026** | *winter is coming*

---

## 1. Phase de réflexion et cahier des charges

### 1.1 Analyse du règlement

La première étape du projet a été l'analyse approfondie du règlement officiel de la Coupe de France de Robotique 2026. Les contraintes mécaniques imposées par le règlement sont les suivantes :

- **Gabarit de départ :** le robot doit tenir dans un volume défini à l'homologation (périmètre de 1m30)
- **Pas de pièces détachées non contrôlées :** toute pièce doit rester solidaire du robot ou être libérée de façon maîtrisée
- **Arrêt d'urgence accessible :** le BAU doit être atteignable depuis l'extérieur du robot sans outil
- **Tirette magnétique de départ :** le robot ne doit pas pouvoir démarrer sans extraction physique de la tirette
- **Autonomie totale :** aucun signal de télécommande n'est autorisé pendant le match

Ces contraintes ont guidé l'ensemble des choix de conception.

### 1.2 Identification des actions à réaliser

À partir du règlement, l'équipe a listé les actions prioritaires à accomplir :

1. **Se déplacer** sur le tapis vinyle de façon précise et répétable
2. **Détecter les obstacles** (robot adverse) en temps réel
3. **Saisir et retourner des blocs** pour les stocker dans les zones
4. **Déployer le bras thermomètre** (actionneur bonus)
5. **Pousser des blocs** en stratégie de repli si la préhension échoue
6. **Revenir en zone de départ** en fin de match

### 1.3 Choix d'architecture globale

Après plusieurs sessions de réflexion en équipe, l'architecture retenue est la suivante :

- **Locomotion :** différentiel à 2 roues motrices (simplicité, robustesse, précision)
- **Préhension :** système à ventouses pneumatiques (adapté aux blocs plats du jeu)
- **Élévation :** deux moteurs pas à pas NEMA 17 sur axe vertical pour monter/descendre l'actionneur
- **Navigation :** LIDAR pour la détection d'obstacles à 360°
- **Structure :** châssis mixte acrylique / MakerBeam / pièces imprimées

(image du schéma de principe / croquis d'architecture initial)

---

## 2. Conception 3D sous OnShape

### 2.1 Outil utilisé

La totalité de la conception mécanique a été réalisée sous **OnShape**, une plateforme de CAO paramétrique entièrement en ligne. Ce choix a permis à tous les membres de l'équipe de travailler simultanément sur le même modèle, sans problème de synchronisation de fichiers.


### 2.2 Structure du châssis

Le châssis du robot principale est une structure composite faisant appel à trois types de matériaux :

#### Plaques acrylique 3 mm (structure principale)
Les panneaux principaux — plateau inférieur, flancs et plateau supérieur — sont découpés dans des plaques d'**acrylique transparent de 3 mm d'épaisseur**. Ce matériau a été retenu pour sa rigidité, sa légèreté, sa facilité de découpe laser et son faible coût.

Les fichiers de découpe (`.dxf`) sont disponibles dans onshape.


#### MakerBeam (profilés structurels)
Les **MakerBeam** forment la colonne vertébrale de la structure et les montants verticaux. Ils offrent un excellent rapport rigidité/masse et permettent de fixer facilement des composants via les rainures en T.


#### Pièces imprimées en 3D
De nombreux éléments de liaison, supports et pièces fonctionnelles ont été conçus sous OnShape et imprimés en 3D :

- Supports de moteurs NEMA 23 et NEMA 17
- Paliers et guides pour l'axe d'élévation
- Logements pour le LIDAR
- Supports du PCA9685 et de l'ESP32
- Pièces de l'actionneur thermomètre
- Pièce de l'actionneur block


(image du modèle 3D complet dans OnShape )

(image des pièces importantes)

### 2.3 Système de locomotion — les roues moulées

L'une des particularités mécaniques du robot est l'utilisation de **roues moulées maison**. Face à la difficulté de trouver des roues à la fois suffisamment souples pour adhérer sur le tapis vinyle antidérapant et aux bonnes dimensions, l'équipe a choisi de les fabriquer elle-même.

#### Procédé de moulage
Les roues ont été réalisées par coulée d'une résine polyuréthane bi-composant souple — un mélange de deux produits liquides qui, une fois mélangés et coulés dans un moule, donnent une matière élastique similaire à un caoutchouc.

#### Fabrication du moule
Le moule a été conçu sous OnShape et imprimé en 3D. Il se compose de deux demi-coques permettant de couler la résine autour d'un moyeu central (fixation sur l'axe du NEMA 23).

(image du moule de la roue imprimé 3D)

(image des roues finies montées sur le robot)

#### Résultat
Les roues obtenues présentent une excellente adhérence sur le tapis de jeu et ont supporté l'ensemble des tests et matchs sans détérioration notable.

### 2.4 Système d'élévation de l'actionneur

L'actionneur de préhension (ventouses) peut monter et descendre sur un axe vertical entraîné par **deux moteurs pas à pas NEMA 17**. Ce système permet de positionner précisément les ventouses à la hauteur des blocs à saisir.

(image du système d'élévation — vue CAO)

- **Guidage :** rails
- **Transmission :** courois

### 2.5 Actionneur de préhension — le système ventouses

Le robot dispose de **8 mini-pompes à air** disposées en 4 paires de 2, chacune reliée à une ventouse. Ce système permet de saisir les blocs de jeu par aspiration.

(image de l'actionneur ventouses — vue réelle)

- **Nombre de ventouses :** 8
- **Diamètre des ventouses :** 25 mm
- **Commande :** via transistors/relais pilotés par l'ESP32
- **Alimentation pompes :** 12 V 

### 2.6 Actionneur thermomètre

Le **bras thermomètre** est l'un des actionneurs bonus du jeu 2026. Il est animé par un servo-moteur et permet d'effectuer une action spécifique sur un élément de la table.

> ℹ️ cet actionneur est un petit bras qui descend sur le thermomètre afin de le pousser quand le robot avance

(image de l'actionneur thermomètre — vue CAO)

---

## 3. Impression 3D

### 3.1 Paramètres d'impression généraux

| Paramètre | Valeur standard |
|---|---|
| Matière principale | PLA / PETG |
| Épaisseur de couche | 0.2 mm |
| Remplissage | 20 – 60 % selon la pièce |
| Périmètres | 3 |
| Supports | Selon géométrie |

### 3.2 Problèmes rencontrés

> ℹ️ certaines impressions on été complexe a imprimer et ont dut etre découper en plusieurs pour faciliter les impressions

---

## 4. Assemblage

### 4.1 Ordre d'assemblage

L'assemblage du robot a suivi l'ordre suivant :

1. **Assemblage du châssis de base** : fixation des plaques acryliques sur les profilés MakerBeam, montage des renforts d'angle
2. **Intégration de la motorisation** : montage des NEMA 23 sur leurs supports, fixation des roues moulées
3. **Montage de l'axe d'élévation** : installation des rails, des NEMA 17 et de la transmission
4. **Fixation de l'électronique** : placement de l'ESP32, des drivers TMC2209, du PCA9685, de la batterie
5. **Câblage** : passage des faisceaux, sertissage des connecteurs
6. **Montage des actionneurs** : fixation des pompes, ventouses, servo-moteurs
7. **Installation du LIDAR** : centrage sur le toit, dégagement angulaire à 360°
8. **Tests de sous-ensembles** avant intégration complète

(image du robot en cours d'assemblage — vue de l'intérieur)

### 4.2 Points d'attention à l'assemblage

- Les plaques acrylique sont **fragiles en flexion** : éviter les contraintes en porte-à-faux sur les fixations
- Le LIDAR doit être **parfaitement horizontal** et dégagé de tout obstacle dans son plan de rotation
- Les câbles des moteurs pas à pas doivent être **fixés avec des colliers** pour éviter qu'ils ne se prennent dans les mécanismes mobiles
- La batterie doit être **accessible rapidement** pour les changements entre matchs

---

## 5. Fichiers disponibles

| Fichier | Format | Emplacement |
|---|---|---|
| Plans de découpe acrylique | `.dxf` | `hardware/cad/acrylic/` |
| Pièces imprimées | `.stl` | `hardware/cad/3d_print/` |
| Fichiers sources CAO | `.step` | `hardware/cad/step/` |

---
