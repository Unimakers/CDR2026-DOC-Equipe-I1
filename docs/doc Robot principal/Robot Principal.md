---
layout: default
title: Robot Principal
nav_order: 2
has_children: true
---

# 🤖 Présentation du projet — UNIMAKERS CDR 2025

> **Équipe :** UNIMAKERS  
> **Compétition :** Coupe de France de Robotique 2025 — catégorie Senior  
> **Thème :** *The Show Must Go On* — Robot-Rock-Tour  
> **Lieu :** Parc des Expositions, La Roche-sur-Yon (Vendée)  
> **Date :** 28 – 31 mai 2025  

---

## 1. La Coupe de France de Robotique

### 1.1 Présentation générale

La Coupe de France de Robotique (CDR) est le plus grand rassemblement de robotique étudiante en Europe. Organisée chaque année par **Planète Sciences** et **Oryon**, elle réunit plusieurs centaines d'équipes issues d'écoles, d'IUT, d'universités, de clubs et d'associations autour d'un défi commun : concevoir et programmer un robot **entièrement autonome** capable d'accomplir des actions précises sur une table de jeu thématisée.

Chaque édition introduit un nouveau thème qui redéfinit entièrement les objectifs, les objets à manipuler et la stratégie à adopter. Les équipes repartent donc de zéro — mécaniquement, électroniquement et logiciellement — à chaque saison.

### 1.2 Règles générales

- **Durée d'un match :** 100 secondes
- **Autonomie totale :** aucune téléopération n'est autorisée pendant le match
- **Gabarit robot :** le robot doit tenir dans un volume défini avant le départ (vérifié lors de l'homologation)
- **Homologation :** avant de jouer, chaque équipe doit passer une phase d'homologation qui valide la sécurité, le gabarit et les capacités minimales du robot
- **Arrêt d'urgence (BAU) :** obligatoire et accessible de l'extérieur
- **Tirette de départ :** le robot doit démarrer uniquement à l'extraction d'une tirette physique
- **Tension batterie :** limitée, la batterie doit être sécurisée et le robot ne doit présenter aucun risque électrique

### 1.3 Thème 2025 — *The Show Must Go On*

L'édition 2025 plonge les robots dans l'univers du **Robot-Rock-Tour**. Les robots doivent préparer une salle de concert, en accomplissant les tâches suivantes sur la table de jeu :

- **Construire des gradins** : empiler des blocs de jeu avec précision pour former des gradins et accueillir le maximum de spectateurs
- **Déployer une banderole** : déployer une banderole publicitaire pour attirer le public
- **Lâcher les PAMIs** : libérer de petits actionneurs mobiles indépendants (PAMIs) qui se déplacent seuls sur la table pour marquer des points supplémentaires
- **Revenir en zone de départ** : en fin de match, le robot doit se garer dans les coulisses pour marquer des points bonus
- **Estimer le public** : déclarer le nombre de spectateurs estimés pour tenter un bonus de points

La table de jeu est un tapis vinyle antidérapant de 3 × 2 mètres, divisé entre la zone de l'équipe bleue et celle de l'équipe jaune.

(image de la table de jeu officielle CDR 2025)

---

## 2. L'équipe UNIMAKERS

### 2.1 Présentation

UNIMAKERS est une équipe de robotique passionnée qui a participé à la Coupe de France de Robotique 2025 dans la catégorie **Senior**. L'équipe regroupe des membres aux compétences complémentaires couvrant la conception mécanique, l'électronique et la programmation embarquée.

> ℹ️ *[À compléter : nombre de membres, formation/école d'origine, ville]*

(image de l'équipe UNIMAKERS)

### 2.2 Objectifs de la saison

Pour cette édition, l'équipe s'est fixé les objectifs suivants :

- Concevoir un robot entièrement original, en partant de zéro
- Passer l'homologation officielle
- Jouer des matchs compétitifs en maximisant le score atteignable avec les actionneurs disponibles
- Documenter l'ensemble du projet pour partager les apprentissages

---

## 3. Le robot principale

### 3.1 Vue d'ensemble

(image du rendu CAO 3D du robot — vue isométrique)

(image du robot physique assemblé)

Le robot principale est un robot différentiel à deux roues motrices conçu pour naviguer sur la table de jeu, détecter les obstacles en temps réel grâce à un LIDAR, et interagir avec les blocs de jeu via un système d'actionneurs à ventouses.

### 3.2 Caractéristiques principales

| Paramètre | Valeur |
|---|---|
| Dimensions (L × l × H) | 100 mm × 350 mm × 420 mm |
| Masse totale | 2-3 kg |
| Type de locomotion | Différentiel — 2 roues motrices moulées |
| Alimentation | Batterie Parkside Li-Ion 24 V |
| Microcontrôleur | ESP32-S3 Plus |
| Capteur de navigation | LIDAR |
| Actionneurs | Ventouses (8 mini-pompes), 9 servo-moteurs, 2 NEMA 17 |

### 3.3 Stratégie adoptée en compétition

En raison de difficultés techniques sur le système d'actionneurs (voir section retours d'expérience), l'équipe a adapté sa stratégie en cours de compétition :

- **Stratégie principale :** pousser les blocs de jeu vers les zones de score à l'aide de la structure du robot, sans préhension ventouse
- **Optimisation :** deux stratégies programmées sélectionnables via le switch physique embarqué, permettant de choisir le côté de départ (équipe bleue ou jaune) et d'adapter le parcours

Cette stratégie dégradée a permis à l'équipe de **passer l'homologation et de jouer des matchs**.

### 3.4 Résultats

- ✅ Homologation réussie
- ✅ Matchs joués
- ⚠️ Actionneurs à ventouses non fonctionnels en compétition (hors actionneur thermomètre)

> ℹ️ *[À compléter : score obtenu, classement, nombre de matchs joués]*

---

## 4. Structure du dépôt GitHub

```
UNIMAKERS-CDR2025/
├── README.md                  ← Ce fichier : présentation rapide
├── docs/
│   ├── 01_presentation_projet.md     ← Ce document
│   ├── 02_conception.md              ← Conception 3D et mécanique
│   └── 03_electronique.md            ← Architecture électronique
├── src/                       ← Code source ESP32
├── hardware/
│   ├── cad/                   ← Fichiers OnShape (exports STEP/STL)
│   ├── kicad/                 ← Schémas électroniques KiCad
│   └── bom/                   ← Liste de matériel (BOM)
├── media/
│   ├── photos/
│   └── videos/
├── CONTRIBUTING.md
└── LICENSE
```

*Documentation rédigée par l'équipe UNIMAKERS — CDR 2025*