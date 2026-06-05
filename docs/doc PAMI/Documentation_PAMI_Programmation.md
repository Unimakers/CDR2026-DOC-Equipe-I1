---
layout: default
parent: PAMI
nav_order: 2
title: Programmation
---
# Documentation Technique — PAMI
## Partie : Programmation (ESP32-S3 / C++)

> **Plateforme :** XIAO ESP32-S3 · C++ · Espressif IDF / Arduino  
> **Contexte :** Coupe de France de Robotique — Activation autonome à T−85 s  
> **Destinataires :** Professeur encadrant · Équipe successeur  
> **Version :** 1.0 — 2025

---

## Table des matières

1. [Présentation générale](#1-présentation-générale)
2. [Architecture logicielle](#2-architecture-logicielle)
3. [Gestion des actionneurs](#3-gestion-des-actionneurs)
4. [Communication & entrées/sorties](#4-communication--entréessorties)
5. [Stratégie et séquencement](#5-stratégie-et-séquencement)
6. [Tâches FreeRTOS](#6-tâches-freertos)
7. [Points d'attention & travaux futurs](#7-points-dattention--travaux-futurs)

---

## 1. Présentation générale

Le **PAMI** (Petit Actionneur Mobile Indépendant) est un robot entièrement autonome conçu pour la Coupe de France de Robotique. Il n'est relié à aucun robot principal pendant le match : il démarre seul, se déplace et réalise ses actions sans intervention extérieure.

Il s'active automatiquement **85 secondes avant la fin du match**, déclenché par le retrait d'une tirette (corde de démarrage). L'arrêt d'urgence matériel est géré par un **BAU (Bouton d'Arrêt d'Urgence)** câblé en dehors du logiciel.

### 1.1 Carte de contrôle

| Paramètre | Valeur |
|---|---|
| Microcontrôleur | Seeed XIAO ESP32-S3 (DIP) |
| Cœur | Xtensa LX7 dual-core (240 MHz) |
| Environnement | Espressif IDF via couche Arduino (PlatformIO) |
| Langage | C++ |
| Tâches RTOS | FreeRTOS — 2 tâches (setup/loop + detection) |
| Alimentation | +5 V via diode Schottky D1 |

---

## 2. Architecture logicielle

Le code est organisé en un fichier principal `main.cpp` accompagné d'un fichier de définitions des broches `Def_PAMI.h`.

### 2.1 Bibliothèques tierces

| Bibliothèque | Rôle |
|---|---|
| `AccelStepper` | Contrôle moteurs pas-à-pas avec profil d'accélération |
| `ESP32Servo` | Pilotage du servo SG90 via PWM |
| `Ultrasonic` | Lecture du capteur HC-SR04 |
| `Wire` | Bus I2C pour l'expander GPIO PCF8574 |

### 2.2 Découpage en modules fonctionnels

| Fonction | Rôle |
|---|---|
| `configStepper()` | Initialise vitesse et accélération des deux steppers |
| `modif_config_stepper()` | Reconfigure dynamiquement les steppers (lent pour les rotations) |
| `waitingTirette()` | Boucle bloquante jusqu'au retrait de la corde de démarrage |
| `runStepper()` | Appel périodique `AccelStepper::run()` pour les deux moteurs |
| `goTo()` | Déplacement vers position absolue avec surveillance obstacle |
| `goToRelative()` | Déplacement relatif avec surveillance obstacle et mise à jour odométrie |
| `move_co_abs()` | Navigation 2D vers une cible en coordonnées absolues (mm) |
| `esquive()` | Contournement d'obstacle en U (4 mouvements) |
| `detection()` *(tâche)* | Lecture sonar sur Core 1 toutes les 50 ms |
| `strategy()` | Séquence d'actions selon la couleur d'équipe |
| `boucle_actionneur()` | Animation du servo SG90 (0 → 70 → 0°) |

### 2.3 Flux d'exécution global

Au démarrage (`setup()`), la séquence est :

```
1. Initialisation périphériques (Serial, broches, servo, I2C)
2. Lecture switch d'équipe via PCF8574 (I2C 0x20, bit 0)
3. Configuration steppers (configStepper)
4. Création mutex FreeRTOS
5. Lancement tâche detection → Core 1
6. Attente retrait tirette (waitingTirette)
7. Activation drivers moteurs (pinEnable → LOW)
8. Appel move_co_abs(CO_POINT_ABS) → navigation vers la cible
```

> La fonction `loop()` est **volontairement vide**. Toute la logique est dans `setup()` et dans la tâche FreeRTOS.

---

## 3. Gestion des actionneurs

### 3.1 Moteurs pas-à-pas

Le PAMI utilise deux moteurs pas-à-pas en configuration **DRIVER** (signaux STEP + DIR). Les broches sont définies dans `Def_PAMI.h`.

| Paramètre | Moteur Gauche (`stepperG`) | Moteur Droit (`stepperD`) |
|---|---|---|
| Broche STEP | `pinStep1` (D1 / GPIO2) | `pinStep2` (D3 / GPIO4) |
| Broche DIR | `pinDir1` (D0 / GPIO1) | `pinDir2` (D2 / GPIO3) |
| Vitesse max | 8 000 pas/s | 8 000 pas/s |
| Accélération | 3 000 pas/s² | 3 000 pas/s² |
| Vitesse rotation lente | 2 000 pas/s | 2 000 pas/s |

> **Note :** Le moteur droit est inversé en logiciel (signe négatif dans `moveSteppers_abs` et `moveSteppers_Relative`) pour compenser le montage mécanique symétrique.

### 3.2 Conversion pas ↔ distance

| Constante | Valeur | Description |
|---|---|---|
| `diametre_roue` | 75 mm | Diamètre roue motrice |
| `step_to_mm` | 650 / 5205 ≈ 0,125 | Ratio pas → mm (calibré empiriquement) |
| `step_90` | −590 pas | Nombre de pas pour une rotation de 90° |
| Facteur avance | ×8 | Conversion mm → pas dans `move_co_abs` (empirique) |

> ⚠️ **Bug connu :** `float step_to_mm = 650/5205;` effectue une **division entière** à la compilation → résultat = 0. Corriger en : `float step_to_mm = 650.0f / 5205.0f;`

### 3.3 Servo SG90

- **Broche :** D4 (GPIO5 / I2C_SDA)
- **Fréquence PWM :** 50 Hz
- **Usage :** `boucle_actionneur()` — balayage 0 → 70° → 0° par pas de 1° toutes les 5 ms

> ⚠️ **Conflit de broche :** D4 est partagée entre le servo SG90 et le SDA I2C. `sg90.attach(D4)` est appelé avant `Wire.begin(D4, D5)`. Vérifier la compatibilité si les deux sont actifs simultanément.

---

## 4. Communication & entrées/sorties

### 4.1 Schéma de câblage

*(Voir fichier image : `schema_elec.png`)*

### 4.2 Tableau des broches

| Broche ESP32 | Signal | Périphérique | Description |
|---|---|---|---|
| D0 / GPIO1 | DIR G | Stepper Gauche | Direction moteur gauche |
| D1 / GPIO2 | STEP G | Stepper Gauche | Impulsion pas moteur gauche |
| D2 / GPIO3 | DIR D | Stepper Droit | Direction moteur droit |
| D3 / GPIO4 | STEP D | Stepper Droit | Impulsion pas moteur droit |
| D4 / GPIO5 | SDA / Servo | I2C + SG90 | Bus I2C SDA + PWM servo |
| D5 / GPIO9 | SCL | I2C (PCF8574) | Bus I2C SCL — expander GPIO |
| D6 / UOTXD | RX-LIDAR | Lidar (optionnel) | UART réception lidar |
| D7 / UORXD | TX-LIDAR | Lidar (optionnel) | UART émission lidar |
| D8 / GPIO7 | trig | HC-SR04 | Trigger ultrason |
| D9 / GPIO8 | echo | HC-SR04 | Echo ultrason |
| D10 / GPIO9 | TIRETTE | Switch J1 | Détection retrait corde (`INPUT_PULLUP`) |
| D9 / GPIO8 | EN | Drivers steppers | Enable moteurs (LOW = actif) |
| D10 / GPIO10 | Lidar_PWM | Lidar | Signal PWM lidar |

### 4.3 I2C — Expander PCF8574

- **Adresse :** `0x20`
- **Usage :** lecture unique au démarrage pour détecter la couleur d'équipe
- **Protocole :** bit 0 du registre d'entrée → `0` = équipe 1, `1` = équipe 2

```cpp
Wire.begin(D4, D5);
Wire.beginTransmission(0x20);
Wire.write(0xFF);           // tous pins en entrée
Wire.endTransmission();

Wire.requestFrom(0x20, 1);
byte etat = Wire.read();
team = ((etat & 0b00000001) == 0) ? 1 : 2;
```

### 4.4 Capteur ultrason HC-SR04

- **Bibliothèque :** `Ultrasonic`
- **Tâche :** Core 1, priorité 1, lecture toutes les **50 ms**
- **Seuil de détection :** distance ≤ **10 cm** → `obstacle = true`
- **Partage inter-core :** variable `volatile bool obstacle` + mutex `obstacleMutex`

> ⚠️ **Bug connu :** la protection mutex dans `detection()` est actuellement **commentée**. La variable `obstacle` est donc accédée sans verrou depuis les deux cores → risque de comportement indéterminé. À réactiver.

---

## 5. Stratégie et séquencement

### 5.1 Démarrage — Tirette

`waitingTirette()` bloque jusqu'au cycle insertion + retrait de la corde, avec debounce de 200 ms :

```cpp
while ( digitalRead(pinTirette));  // attend l'insertion (pin → LOW)
delay(200);
while (!digitalRead(pinTirette));  // attend le retrait (pin → HIGH)
delay(200);
```

### 5.2 Système de coordonnées

| Variable | Description |
|---|---|
| `CO_PAMI_ABS[3]` | Position courante : `{x (mm), y (mm), angle (°)}` |
| `CO_POINT_ABS[2]` | Point d'arrivée cible : `{x=1200, y=650}` en mm |
| `step_90 = −590` | Pas pour une rotation de 90° (à recalibrer si mécanique change) |
| Angles | 0°=avant · 90°=droite · 180°=arrière · 270°=gauche |

### 5.3 Navigation absolue — `move_co_abs()`

Déplace le PAMI de sa position courante vers la cible en deux phases :

```
Phase 1 — Axe X :
  → Calculer l'angle à corriger
  → Rotation lente (modif_config_stepper)
  → Avancer sur X (goToRelative)
  → Mettre à jour CO_PAMI_ABS[0]

Phase 2 — Axe Y :
  → Calculer l'angle à corriger
  → Rotation lente
  → Avancer sur Y (goToRelative)
  → Mettre à jour CO_PAMI_ABS[1]
```

### 5.4 Détection et évitement d'obstacles

Lors d'un déplacement (`goToRelative`), si un obstacle est détecté :

1. Mémoriser les pas restants
2. Mettre à jour `CO_PAMI_ABS` avec les pas déjà effectués
3. `stopMotors()`
4. Attendre 250 ms
5. Si l'obstacle a disparu → reprendre le déplacement
6. Si l'obstacle persiste → appeler `esquive()` *(actuellement commenté)*

**Manœuvre d'esquive en U (`esquive()`) :**

```
1. Rotation gauche 90°
2. Avance ~150 mm
3. Rotation droite 90°
4. Avance ~150 mm  (passage devant l'obstacle)
5. Rotation droite 90°
6. Avance ~150 mm
7. Rotation gauche 90°  (retour à l'angle initial)
8. Reprendre move_co_abs() depuis la nouvelle position
```

> ⚠️ `esquive()` est implémentée mais son appel est **commenté** dans `goToRelative()`. À tester et valider en conditions réelles.

### 5.5 Stratégie par équipe — `strategy()`

| Équipe | Action |
|---|---|
| Équipe 1 | Déplacement relatif (−500, +500 pas) — repositionnement final |
| Équipe 2 | Déplacement relatif (+500, −500 pas) + `boucle_actionneur()` (servo SG90) |

---

## 6. Tâches FreeRTOS

| Tâche | Core | Priorité | Description |
|---|---|---|---|
| `setup()` / `loop()` | Core 0 | — | Navigation, stratégie, séquencement principal |
| `detection()` | Core 1 | 1 | Lecture sonar, mise à jour du flag `obstacle` |

La séparation sur deux cores garantit que la **lecture ultrason** (toutes les 50 ms) ne bloque jamais le **calcul des pas moteurs**, qui doit être appelé très fréquemment pour ne pas perdre de pas (`runStepper()` doit tourner en boucle serrée).

---

## 7. Points d'attention & travaux futurs

### 7.1 Bugs connus

| # | Localisation | Problème | Correction |
|---|---|---|---|
| 1 | `main.cpp` L.27 | `float step_to_mm = 650/5205` → division entière, résultat = 0 | Remplacer par `650.0f / 5205.0f` |
| 2 | `detection()` | Mutex `obstacleMutex` commenté → accès non protégé à `obstacle` | Réactiver le bloc `xSemaphoreTake` |
| 3 | `setup()` | D4 partagée servo + I2C SDA | Tester la cohabitation ou dédier une broche |
| 4 | `move_co_abs()` | Facteur ×8 (mm→pas) non documenté et empirique | Mesurer et formaliser avec diamètre + entraxe réels |
| 5 | `goToRelative()` | Appel `esquive()` commenté | Tester et activer en conditions réelles |

### 7.2 Recommandations pour les successeurs

- Ajouter `Def_PAMI.h` commenté (liste complète des constantes et broches) à cette documentation
- Mettre en place un **log série structuré** (positions, états) pour le débogage en match
- Remplacer la navigation cartésienne par une **odométrie avec encodeurs** pour plus de précision
- Envisager **ESP-NOW** pour recevoir le signal de départ du robot principal (en remplacement de la tirette)
- Créer des **tests unitaires** des fonctions de conversion (pas ↔ mm, angles) avant chaque compétition
- Documenter `step_90` : valeur mesurée en fonction de l'entraxe des roues et du nombre de pas/tour du moteur

---

*Documentation rédigée par l'équipe Programmation — Projet PAMI 2025*
