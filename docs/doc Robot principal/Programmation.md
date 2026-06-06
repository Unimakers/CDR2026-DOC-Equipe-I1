---
layout: default
parent: Robot Principal
nav_order: 2
title: Programmation
---

## Table des matières

1. [Présentation générale](#1-présentation-générale)
2. [Architecture logicielle](#2-architecture-logicielle)
3. [Machine à états — main.cpp](#3-machine-à-états--maincpp)
4. [Module Robot — Robot.h / Robot.cpp](#4-module-robot--roboth--robotcpp)
5. [Module Moteurs — MotorDriver.h / MotorDriver.cpp](#5-module-moteurs--motordriverh--motordrivercpp)
6. [Module Lidar — Lidar.h / Lidar.cpp / ld06](#6-module-lidar--lidarh--lidarcpp--ld06)
7. [Module Actionneurs — Actions.h / Actions.cpp](#7-module-actionneurs--actionsh--actionscpp)
8. [Stratégies — Strategies.h / Strategies.cpp](#8-stratégies--strategiesh--strategiescpp)
9. [Configuration centrale — Config.h](#9-configuration-centrale--configh)
10. [Tâches FreeRTOS](#10-tâches-freertos)
11. [Points d'attention & travaux futurs](#11-points-dattention--travaux-futurs)

---

## 1. Présentation générale

Le robot principal est un robot à **entraînement différentiel** (deux roues motrices gauche/droite) complété par un **système élévateur** (deux moteurs supplémentaires avant/arrière). Il est contrôlé par un ESP32 et évolue de façon autonome pendant un match de **100 secondes**.

### Caractéristiques principales

| Paramètre | Valeur |
|---|---|
| Durée du match | 100 000 ms (`MATCH_DURATION_MS`) |
| Moteurs de déplacement | 2 steppers gauche/droit (TMC2209, 1/16 microstep) |
| Moteurs élévateur | 2 steppers avant/arrière |
| Détection obstacles | Lidar LD06 (360°, UART 230 400 baud) |
| Actionneurs | 4 servos via PCA9685 (I2C 0x40) |
| Expander GPIO | PCF8574 (I2C 0x20) — enable moteurs + microstepping |
| Couleur d'équipe | Jaune ou Bleu, switch physique sur `COLOR_SWITCH_PIN` |
| Démarrage | Retrait de la tirette (`TIRETTE_PIN`) |
| Arrêt d'urgence | BAU matériel (hors logiciel) |

---

## 2. Architecture logicielle

### 2.1 Arborescence des fichiers

```
├── main.cpp          Machine à états principale (boot → match → fin)
├── Config.h          Toutes les constantes matérielles et paramètres
├── Robot.h / .cpp    API haut niveau (déplacements, pose, obstacles)
├── MotorDriver.h/.cpp  Contrôle bas niveau FastAccelStepper + PCF8574
├── Lidar.h / .cpp    Lecteur LD06, détection obstacles (tâche FreeRTOS)
├── Actions.h / .cpp  Contrôle servos via PCA9685
├── Strategies.h/.cpp Stratégies de match Jaune et Bleu
└── Lidarld06/
    ├── ld06.h / .cpp Driver bas niveau LD06 (parsing trame UART)
    └── ld06crc.h     Table CRC pour validation paquets LD06
```

### 2.2 Bibliothèques tierces

| Bibliothèque | Rôle |
|---|---|
| `FastAccelStepper` | Génération hardware de signaux STEP/DIR à haute fréquence |
| `Adafruit_PCF8574` | Contrôle de l'expander I2C PCF8574 |
| `Adafruit_PWMServoDriver` | Contrôle du driver servo PCA9685 |
| `Wire` | Bus I2C (PCF8574 + PCA9685) |
| `FreeRTOS` | Multitâche (intégré ESP32) |

### 2.3 Couches logicielles

```
┌─────────────────────────────────────┐
│  main.cpp   — Machine à états       │  ← Orchestration
├─────────────────────────────────────┤
│  Strategies.cpp — Logique de match  │  ← Code étudiant
├──────────────────┬──────────────────┤
│  Robot.cpp       │  Actions.cpp     │  ← API haut niveau
├──────────────────┼──────────────────┤
│  MotorDriver.cpp │  Lidar.cpp       │  ← Drivers mid-level
├──────────────────┼──────────────────┤
│  FastAccelStepper│  ld06.cpp        │  ← Pilotes bas niveau
└──────────────────┴──────────────────┘
       Hardware (TMC2209, LD06, PCA9685, PCF8574)
```

---

## 3. Machine à états — main.cpp

`main.cpp` orchestre le cycle de vie complet du robot via une **machine à états explicite** (`RobotState`). La variable `currentState` est testée à chaque itération de `loop()`.

### 3.1 États et transitions

```
Boot
  │  Initialisation Serial, I2C, broches, robot.begin()
  ▼
SelectColor
  │  Lecture du switch physique (Jaune / Bleu)
  ▼
WaitTiretteInserted
  │  Attente de l'insertion de la tirette
  │  → Lidar streamé sur Teleplot pendant l'attente
  │  → Couleur peut être changée à tout moment
  ▼
Initializing
  │  enableMotors() + initYellow() ou initBlue()
  ▼
WaitTiretteRemoved
  │  Attente du retrait de la tirette → départ match
  ▼
MatchRunning
  │  strategyYellow() ou strategyBlue() (bloquant)
  ▼
MatchEnded
  │  stop() + disableMotors() + stopActions()
  ▼
(idle forever)
```

> En cas d'échec de `robot.begin()`, la machine passe directement à l'état `Error` et reste bloquée.

### 3.2 Tableau des états

| État | Condition d'entrée | Actions | Sortie |
|---|---|---|---|
| `Boot` | Démarrage | Init Serial, I2C, broches, robot | → `SelectColor` ou `Error` |
| `SelectColor` | Après boot | Lecture switch couleur | → `WaitTiretteInserted` |
| `WaitTiretteInserted` | Après sélection couleur | Boucle : Teleplot lidar + lecture switch | → `Initializing` |
| `Initializing` | Tirette insérée | `enableMotors()` + init couleur | → `WaitTiretteRemoved` |
| `WaitTiretteRemoved` | Fin init | Boucle attente retrait tirette | → `MatchRunning` |
| `MatchRunning` | Tirette retirée | `startMatch()` + stratégie couleur | → `MatchEnded` |
| `MatchEnded` | Stratégie terminée | `stop()` + `disableMotors()` + log | Idle |
| `Error` | Échec robot.begin() | `stop()` + message | Idle |

### 3.3 Helpers de lecture

```cpp
// Lecture du switch couleur (tient compte de COLOR_SWITCH_YELLOW_STATE)
static TeamColor readColorSwitch();

// Lecture d'un bouton avec prise en compte de BUTTON_ACTIVE_LOW
static bool isButtonPressed(int pin);
```

---

## 4. Module Robot — Robot.h / Robot.cpp

La classe `Robot` est **l'interface principale pour les étudiants**. Elle encapsule `MotorDriver` et `Lidar` et expose des commandes lisibles en mm et degrés.

### 4.1 Système de coordonnées

```
Origine : coin haut-gauche du terrain (3000 × 2000 mm)
  X  → positif vers la droite
  Y  ↓ positif vers le bas (convention "écran")
  θ  = 0° → robot face à +X (droite)
  Angles positifs = sens horaire (CW) vu de dessus
```

> **Limitation importante :** le robot n'a **pas d'encodeurs**. La pose (X, Y, θ) est estimée uniquement à partir des distances commandées. Un glissement de roue ou un pas manqué introduit une dérive permanente. Utiliser les séquences `datum` pour recalibrer.

### 4.2 API de déplacement

| Méthode | Description |
|---|---|
| `go(distance_mm)` | Avance (+) ou recule (−) en ligne droite. Bloquant, pause si obstacle. |
| `turn(angle_deg)` | Rotation relative. Positif = horaire. **Sans détection obstacle.** |
| `turnTo(theta_deg)` | Rotation vers un cap absolu (chemin le plus court). |
| `gotoXY(x_mm, y_mm)` | Tourne vers la cible puis avance. |
| `gotoPose(x, y, theta)` | `gotoXY` puis `turnTo`. |
| `datumBack(dist, x, y, θ)` | Recul lent contre un bord, puis force la pose connue. |
| `wait(ms)` | Pause temporisée (respecte le timer de match). |
| `stop()` | Arrêt immédiat des deux moteurs. |

### 4.3 Gestion de la pose

```cpp
robot.setPose(x_mm, y_mm, theta_deg);  // Force la position estimée
robot.getPoseX();     // mm
robot.getPoseY();     // mm
robot.getPoseTheta(); // degrés, normalisé dans [-180, 180)
```

### 4.4 Gestion du match

```cpp
robot.startMatch();           // Démarre le chronomètre 100 s
robot.matchHasEnded();        // true si 100 s écoulées
robot.getMatchElapsedMs();    // Millisecondes depuis startMatch()
robot.requestEmergencyStop(); // Arrêt d'urgence logiciel
```

### 4.5 Détection d'obstacles

| Méthode | Description |
|---|---|
| `enableObstacleDetection()` | Active la pause automatique sur obstacle |
| `disableObstacleDetection()` | Désactive (robot ignore les obstacles) |
| `setObstacleStopDistance(mm)` | Seuil d'arrêt avant et arrière |
| `setObstacleClearDistance(mm)` | Seuil de reprise avant et arrière |
| `setObstacleFrontStopDistance(mm)` | Seuil avant uniquement |
| `setObstacleRearStopDistance(mm)` | Seuil arrière uniquement |

**Comportement de `go()` avec obstacle :**

```
1. Commande un déplacement relatif (en pas)
2. En boucle :
   a. Si shouldStop() → arrêt immédiat
   b. Si obstacle détecté → brake() (décélération haute)
   c. Attente arrêt complet des moteurs
   d. waitForObstacleToClear() (boucle jusqu'à distance > clear)
   e. setDefaultSpeed() + reprend les pas restants
3. Met à jour la pose avec la distance réellement parcourue
```

### 4.6 Contrôle de l'élévateur

```cpp
robot.frontUp();     // Monte la fourche avant (FRONT_UP_POS)
robot.frontMiddle(); // Position intermédiaire
robot.frontDown();   // Descend (FRONT_DOWN_POS)
robot.backUp();      // Idem pour l'arrière
robot.backMiddle();
robot.backDown();
robot.frontPos(steps); // Position absolue en pas (appel direct)
robot.backPos(steps);
```

---

## 5. Module Moteurs — MotorDriver.h / MotorDriver.cpp

`MotorDriver` encapsule **FastAccelStepper** (génération STEP/DIR hardware) et le **PCF8574** (enable + microstepping via I2C).

### 5.1 Moteurs et broches

| Moteur | STEP | DIR | Inversion |
|---|---|---|---|
| Gauche | `GPIO_NUM_40` | `GPIO_NUM_10` | `true` (synchronisation roues) |
| Droit | `GPIO_NUM_38` | `GPIO_NUM_39` | `false` |
| Arrière (élévateur) | `BACK_STEP_PIN` (3) | `BACK_DIR_PIN` (4) | `false` |
| Avant (élévateur) | `FRONT_STEP_PIN` (A0) | `FRONT_DIR_PIN` (A1) | `false` |

### 5.2 Géométrie et conversion pas ↔ mm

La conversion est calculée une seule fois dans `begin()` :

```
steps_per_mm = (MOTOR_STEPS_PER_REV × MICROSTEPS × GEAR_RATIO) / (π × WHEEL_DIAMETER_MM)
             = (200 × 16 × 1.0) / (π × 68.0)
             ≈ 14.96 pas/mm
```

| Constante | Valeur | Description |
|---|---|---|
| `WHEEL_DIAMETER_MM` | 68.0 mm | Diamètre extérieur de la roue |
| `WHEEL_BASE_MM` | 227.0 mm | Entraxe des roues |
| `MOTOR_STEPS_PER_REV` | 200 | Pas/tour moteur (1,8°/pas) |
| `MICROSTEPS` | 16 | Diviseur microstepping TMC2209 |
| `GEAR_RATIO` | 1.0 | Pas de réducteur |

**Calcul de la rotation :**

```
arc_mm = π × WHEEL_BASE_MM × angle_deg / 360
arcSteps = mmToSteps(arc_mm)
Roue gauche : +arcSteps  (avance)
Roue droite : −arcSteps  (recule)
```

### 5.3 Expander PCF8574 (I2C 0x20)

Le PCF8574 contrôle les signaux d'enable et de microstepping des drivers TMC2209 via I2C.

| Bit PCF | Signal | Rôle |
|---|---|---|
| 4 | `PCF_MOTOR_ENABLE_PIN` | Enable moteurs gauche/droit (LOW = actif) |
| 3 | `PCF_MOTOR_EL_ENABLE_PIN` | Enable moteurs élévateur (LOW = actif) |
| 5 | `PCF_TMC_MS1_PIN` | MS1 moteurs déplacement |
| 7 | `PCF_TMC_MS2_PIN` | MS2 moteurs déplacement |
| 2 | `PCF_TMC_EL_MS1_PIN` | MS1 moteurs élévateur |
| 0 | `PCF_TMC_EL_MS2_PIN` | MS2 moteurs élévateur |

**Microstepping TMC2209 (standalone) :**

| MS1 | MS2 | Diviseur |
|---|---|---|
| 0 | 0 | 1/8 |
| 1 | 0 | 1/2 |
| 0 | 1 | 1/4 |
| **1** | **1** | **1/16** ← configuré |

### 5.4 Profils de vitesse

| Profil | Vitesse | Accélération | Usage |
|---|---|---|---|
| Normal (`setDefaultSpeed`) | 300 mm/s | 500 mm/s² | Déplacements de match |
| Lent (`setDatumSpeed`) | 80 mm/s | 100 mm/s² | Calage contre le bord |
| Freinage (`brake`) | — | 2 000 mm/s² | Obstacle détecté |

### 5.5 Commandes principales

```cpp
motor.moveLinear(distance_mm);           // Ligne droite (non bloquant)
motor.moveTurn(angle_deg);               // Rotation sur place
motor.moveLinearSteps(steps_L, steps_R); // Pas explicites par roue
motor.isMotionComplete();                // true si les deux roues sont arrêtées
motor.stop();                            // forceStop() immédiat
motor.enableMotors();                    // PCF8574 EN → LOW
motor.disableMotors();                   // PCF8574 EN → HIGH
motor.mmToSteps(mm);                     // Conversion mm → pas
motor.stepsToMm(steps);                  // Conversion pas → mm
```

---

## 6. Module Lidar — Lidar.h / Lidar.cpp / ld06

### 6.1 Architecture

Le Lidar LD06 tourne sur **Core 0** de l'ESP32 (core dédié à la communication), dans une tâche FreeRTOS indépendante. Le Core 1 (navigation) interroge les résultats via un mutex.

```
Core 0                          Core 1 (navigation)
─────────────────────────       ──────────────────────────
lidarTask()                     Robot::waitForMotion()
  └─ Serial1.read()               └─ lidar.isObstacleDetected()
  └─ _driver.readScan()               └─ xSemaphoreTake(_mutex)
  └─ xSemaphoreTake(_mutex)           └─ parcourt _scanData
  └─ _scanData.clear() + fill         └─ xSemaphoreGive(_mutex)
  └─ _scanReady = true
  └─ xSemaphoreGive(_mutex)
  └─ vTaskDelay(5 ms)
```

### 6.2 Configuration UART (Serial1)

| Paramètre | Valeur |
|---|---|
| Baudrate | 230 400 |
| RX Pin | `LIDAR_RX_PIN` (TX de l'ESP32) |
| TX Pin | `LIDAR_TX_PIN` (RX de l'ESP32) |
| Format | `SERIAL_8N1` |

### 6.3 Paramètres de détection

| Paramètre | Valeur par défaut | Description |
|---|---|---|
| `DEFAULT_OBSTACLE_FRONT_STOP_DISTANCE_MM` | 300 mm | Distance d'arrêt avant |
| `DEFAULT_OBSTACLE_FRONT_CLEAR_DISTANCE_MM` | 400 mm | Distance de reprise avant |
| `DEFAULT_OBSTACLE_REAR_STOP_DISTANCE_MM` | 200 mm | Distance d'arrêt arrière |
| `DEFAULT_OBSTACLE_REAR_CLEAR_DISTANCE_MM` | 300 mm | Distance de reprise arrière |
| `OBSTACLE_FRONT_ANGLE_DEG` | ±30° | Demi-angle du cône avant |
| `OBSTACLE_REAR_ANGLE_DEG` | ±30° | Demi-angle du cône arrière |
| `LIDAR_MAX_DISTANCE_MM` | 1 000 mm | Points au-delà ignorés |
| `LIDAR_INTENSITY_THRESHOLD` | 100 | Seuil d'intensité (0-255) |
| `LIDAR_MOUNT_ANGLE_DEG` | 0° | Offset de montage du lidar |

### 6.4 Convention d'angle LD06

Le driver `ld06.cpp` convertit les angles natifs du LD06 (**CW**) en convention mathématique (**CCW**) :

```
angle_stocké = 360° − angle_brut_LD06
```

La détection de secteur (`isInSector`) normalise ensuite la différence angulaire dans `[−180, +180]` pour gérer les discontinuités à 0°/360°.

### 6.5 Visualisation Teleplot

Pendant la phase `WaitTiretteInserted`, le lidar est streamé vers [https://teleplot.fr](https://teleplot.fr) via Serial :

```
>lidar:x1:y1;x2:y2;...|xy          // Tous les points du scan
>lidar_front:x1:y1;...|xy          // Zone danger avant (< stop distance)
>lidar_rear:x1:y1;...|xy           // Zone danger arrière (< stop distance)
```

### 6.6 Driver bas niveau LD06

Le fichier `ld06.cpp` implémente le parsing des **trames UART LD06** :

| Champ trame | Taille | Description |
|---|---|---|
| Header | 1 octet | `0x54` |
| Version/Size | 1 octet | `0x2C` |
| Speed | 2 octets | Vitesse rotation (°/s) |
| Start Angle | 2 octets | Angle début (× 0,01°) |
| Mesures × 12 | 36 octets | Distance (mm) + Intensité (0-255) chacune |
| End Angle | 2 octets | Angle fin (× 0,01°) |
| Timestamp | 2 octets | Horodatage (ms, max 30 000) |
| CRC | 1 octet | Checksum (table `ld06crc.h`) |

Un **double buffer** (`_scanA` / `_scanB`) est utilisé : le buffer courant est rempli pendant la lecture, le précédent est disponible pour la détection. Un swap est effectué à chaque scan 360° complet.

---

## 7. Module Actionneurs — Actions.h / Actions.cpp

### 7.1 PCA9685 — Driver servo I2C

| Paramètre | Valeur |
|---|---|
| Adresse I2C | `0x40` |
| Fréquence PWM | 50 Hz |
| Résolution | 4 096 comptes (12 bits) |
| Pulse 0° | 125 comptes (≈ 610 µs) |
| Pulse 180° | 575 comptes (≈ 2 806 µs) |

```cpp
// Conversion angle → pulse
pulse = map(angleDeg, 0, 180, SERVO_PULSE_MIN, SERVO_PULSE_MAX);
pca.setPWM(channel, 0, pulse);
```

### 7.2 Canaux servos actifs

| Canal PCA9685 | Constante | Rôle |
|---|---|---|
| 0 | `SERVO_CURSOR` | Servo curseur (stratégie) |
| 1 | `FRONT_ARM_1` | Bras avant 1 |
| 2 | `FRONT_ARM_2` | Bras avant 2 |
| 3 | `FRONT_ARM_3` | Bras avant 3 |
| 4 | `FRONT_ARM_4` | Bras avant 4 |

### 7.3 Actions nommées

```cpp
initActions();            // Initialise PCA9685, servos en position d'origine
stopActions();            // Sécurise tous les servos (fin de match)
lever_bras_devant();      // Bras 1/2/3/4 → 0°/180°/0°/180°
baisser_bras_devant();    // Bras 1/2/3/4 → 40°/140°/40°/140°
servoAngle(channel, deg); // Contrôle direct d'un canal (0-15), angle 0-180°
```

### 7.4 Scan I2C (`scanI2C()`)

Disponible pour le debug, balaye les adresses 1-126 et identifie automatiquement PCF8574 (0x20) et PCA9685 (0x40). Appelé au démarrage depuis `main.cpp`.

---

## 8. Stratégies — Strategies.h / Strategies.cpp

### 8.1 Structure

Quatre fonctions appelées par `main.cpp` selon la couleur sélectionnée :

| Fonction | Moment d'appel | Rôle |
|---|---|---|
| `initYellow(robot)` | Pendant `Initializing` | Initialisation servos, activation détection, pose de départ |
| `initBlue(robot)` | Pendant `Initializing` | Idem côté bleu |
| `strategyYellow(robot)` | Pendant `MatchRunning` | Stratégie 100 s équipe jaune |
| `strategyBlue(robot)` | Pendant `MatchRunning` | Stratégie 100 s équipe bleu |

### 8.2 Initialisation (init*)

Les deux fonctions font la même séquence :

```
1. enableObstacleDetection()
2. setObstacleStopDistance(300) / setObstacleClearDistance(500)
3. initActions()  ← initialise PCA9685
4. Séquence test servo curseur : 90° → 45° → 90° (1 s entre chaque)
5. initActions()  ← remise à zéro des actionneurs
```

### 8.3 Stratégie jaune (`strategyYellow`)

Objectif déclaré dans le code : aller au fond du terrain, se placer derrière les planches et les ramener au spawn.

**Paramètre de calibration :** `float offangle = 1.081` — facteur correctif appliqué à toutes les rotations de 90° pour compenser une dérive mécanique mesurée empiriquement.

Séquence simplifiée :

```
go(650)  → go(-350)  → turn(90°×1.081)  → go(300)
→ turn(-90°×1.081)  → go(520)  → turn(90°×1.081)
→ go(700)  → go(-1100)  ...
→ [Positionnement curseur : servoAngle(SERVO_CURSOR, 45)]
→ go(-580)  → [Curseur retour : servoAngle(SERVO_CURSOR, 90)]
→ ... → disableObstacleDetection()
→ wait(26000)  ← attente 26 secondes
→ enableObstacleDetection()  → go(-1700)
```

### 8.4 Stratégie bleu (`strategyBlue`)

Symétrique de la stratégie jaune (rotations inversées). Même `offangle = 1.081`. Attente finale de **27 000 ms** au lieu de 26 000 ms.

### 8.5 Corriger `offangle`

`offangle` compense l'imprécision de rotation. Pour le recalibrer :

```
1. Faire tourner le robot de 360° (4 × turn(90))
2. Mesurer l'écart en degrés (ex : −8°)
3. offangle = 360 / (360 − erreur)
   Ex : 360 / 352 = 1.0227
```

---

## 9. Configuration centrale — Config.h

`Config.h` est **l'unique fichier à modifier** pour adapter le code à un nouveau robot. Toutes les valeurs matérielles y sont centralisées.

### 9.1 Paramètres essentiels à vérifier avant chaque saison

| Constante | Valeur actuelle | À vérifier |
|---|---|---|
| `WHEEL_DIAMETER_MM` | 68.0 mm | Mesurer avec pied à coulisse |
| `WHEEL_BASE_MM` | 227.0 mm | Mesurer l'entraxe réel |
| `LEFT_MOTOR_INVERTED` | `true` | Tester si le robot avance droit |
| `COLOR_SWITCH_YELLOW_STATE` | `LOW` | Confirmer le câblage du switch |
| `TIRETTE_INSERTED_STATE` | `LOW` | Confirmer avec multimètre |
| `LIDAR_MOUNT_ANGLE_DEG` | 0° | Vérifier l'orientation du lidar |
| `SERVO_PULSE_MIN/MAX` | 125 / 575 | Calibrer si les servos ne font pas 0-180° |

### 9.2 Extraits importants

```cpp
// Géométrie
#define WHEEL_DIAMETER_MM    68.0f
#define WHEEL_BASE_MM        227.0f
#define MOTOR_STEPS_PER_REV  200
#define MICROSTEPS           16

// Vitesses
#define DEFAULT_SPEED_MM_S        300.0f
#define DEFAULT_ACCEL_MM_S2       500.0f
#define DEFAULT_TURN_SPEED_DEG_S   90.0f
#define DATUM_SPEED_MM_S           80.0f
#define OBSTACLE_BRAKE_ACCEL_MM_S2 2000.0f

// Lidar
#define DEFAULT_OBSTACLE_FRONT_STOP_DISTANCE_MM   300.0f
#define DEFAULT_OBSTACLE_FRONT_CLEAR_DISTANCE_MM  400.0f
#define OBSTACLE_FRONT_ANGLE_DEG  30.0f
#define LIDAR_MAX_DISTANCE_MM     1000
#define LIDAR_INTENSITY_THRESHOLD 100
```

---

## 10. Tâches FreeRTOS

| Tâche | Core | Priorité | Stack | Description |
|---|---|---|---|---|
| `setup()` / `loop()` | Core 1 | — | défaut | Machine à états, navigation, stratégie |
| `lidarTask()` | Core 0 | 1 | 4 096 octets | Lecture UART LD06, mise à jour `_scanData` |

### Partage de données

La seule variable partagée entre les deux cores est `_scanData` (vecteur de mesures lidar), protégée par `_mutex` (SemaphoreHandle FreeRTOS). La lecture (Core 1) et l'écriture (Core 0) utilisent toutes deux `xSemaphoreTake` avec un timeout de 10 ms.

```cpp
// Écriture (Core 0 — lidarTask)
if (xSemaphoreTake(self->_mutex, pdMS_TO_TICKS(10)) == pdTRUE) {
    self->_scanData.clear();
    // ... remplissage ...
    self->_scanReady = true;
    xSemaphoreGive(self->_mutex);
}

// Lecture (Core 1 — isObstacleDetected)
if (xSemaphoreTake(_mutex, pdMS_TO_TICKS(10)) != pdTRUE) return false;
// ... parcours _scanData ...
xSemaphoreGive(_mutex);
```

---

## 11. Points d'attention & travaux futurs

### 11.1 Limitations connues

| # | Fichier | Problème | Impact |
|---|---|---|---|
| 1 | `Robot.cpp` | Pas d'encodeurs → dérive de pose si glissement | Navigation imprécise sur grande distance |
| 2 | `Strategies.cpp` | `offangle = 1.081` empirique, non documenté | À recalibrer si mécanique change |
| 3 | `Strategies.cpp` | `initActions()` appelé 2× en init | Vérifier qu'il n'y a pas de conflit servo |
| 4 | `Lidar.cpp` | `vTaskDelay(5 ms)` entre lectures — peut manquer des octets à 230 400 baud | Tester avec buffer UART plus grand |
| 5 | `Config.h` | `DEBUG_SERIAL true` en prod → Serial.printf dans lidarTask peut ralentir le parsing | Mettre `false` pour les matchs |
| 6 | `Robot.cpp` (`frontPos`) | Boucle `while (!isElevatorMotionComplete())` rappelle `elevatorPosFront(pos)` en continu | Peut surcharger FastAccelStepper ; un seul appel suffit avant la boucle d'attente |

### 11.2 Recommandations pour les successeurs

- **Ajouter des encodeurs** (roues codeuses ou sur moteur) pour une odométrie réelle au lieu de l'estimation par commande.
- **Documenter `offangle`** avec la procédure de mesure et la valeur obtenue à chaque saison.
- **Ajouter un datum systématique** (`datumBack`) au début de chaque stratégie pour recaler la pose initiale.
- **Créer des tests de calibration** avant le match : déplacement de 1 m → mesurer l'erreur, rotation de 360° → mesurer l'écart.
- **Séparer le servo curseur** du reste des actionneurs bras (classe ou fichier dédié) pour clarifier les responsabilités.
- **Versionner `Config.h`** avec la date et le numéro de robot pour retrouver les valeurs d'une saison à l'autre.
- **Utiliser `gotoXY` / `gotoPose`** plutôt que des séquences `go` / `turn` hardcodées pour rendre la stratégie plus lisible et plus facile à modifier.

---

