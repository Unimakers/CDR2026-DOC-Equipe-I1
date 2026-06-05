---
layout: default
title: Robot Principal
nav_order: 2
has_children: true
---

# 🤖 Présentation du projet — UNIMAKERS CDR 2025

> **Équipe :** UNIMAKERS  
> **Compétition :** Coupe de France de Robotique 2025 — catégorie Senior  
> **Thème :** *Winter is coming*
> **Lieu :** Parc des Expositions, La Roche-sur-Yon (Vendée)  
> **Date :** mai 2026  

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

### 1.3 Thème 2025 — *winter is coming*

L'édition 2026 plonge les robots dans l'univers de **d'un frigo en lisière de foret**. Les robots doivent prendre la place d'écureuilles, en accomplissant les tâches suivantes sur la table de jeu :

- **s'acaparer des caisses de noisetes** : prendre des blocs et les retourner sur ça propre couleur dans les nids
- **augmenter la température** : placer un curseur le plus proche du 10 
- **Lâcher les PAMIs** : libérer de petits actionneurs mobiles indépendants (PAMIs) qui se déplacent seuls sur la table pour marquer des points supplémentaires
- **Revenir en zone de départ** : en fin de match, le robot doit se garer dans les coulisses pour marquer des points bonus
- **ramener les noisettes dans le nids** : remener des bloc dans sa propre zone

La table de jeu est un tapis vinyle antidérapant de 3 × 2 mètres, divisé entre la zone de l'équipe bleue et celle de l'équipe jaune.

(image de la table de jeu officielle CDR 2026)

---

## 2. L'équipe UNIMAKERS

### 2.1 Présentation

UNIMAKERS est une équipe de robotique passionnée qui a participé à la Coupe de France de Robotique 2026 dans la catégorie **Senior**. L'équipe regroupe des membres aux compétences complémentaires couvrant la conception mécanique, l'électronique et la programmation embarquée.

> ℹ️ 15 participant

(image de l'équipe UNIMAKERS)

### 2.2 Objectifs de la saison

Pour cette édition, l'équipe s'est fixé les objectifs suivants :

- Concevoir un robot entièrement original, en partant de zéro par rapport aux année précédentes
- Passer l'homologation officielle
- Jouer des matchs compétitifs en maximisant le score atteignable avec les actionneurs disponibles
- Documenter l'ensemble du projet pour partager les apprentissages

---

## 3. Le robot principale

### 3.1 Vue d'ensemble

(image du rendu CAO 3D du robot — vue isométrique)

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

> ℹ️ 29e sur 90 joueur

---
