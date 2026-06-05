---
layout: default
parent: Robot Principal
nav_order: 3
title: Electronique
---

# ⚡ Électronique — Architecture et câblage

> **Équipe :** UNIMAKERS | **CDR 2026** | *winter is coming*

---

## 1. Vue d'ensemble du système électronique

L'ensemble du système électronique du robot principal est centralisé autour d'un unique microcontrôleur : l'**ESP32-S3 Plus**. Il orchestre tous les sous-systèmes : déplacement, actionneurs, LIDAR, interfaces utilisateur et sécurités.

(image du schéma bloc général de l'architecture électronique)

### 1.1 Carte de l'architecture

```
Batterie Parkside 24V Li-Ion
        │
        ├──── [BAU] ──── Coupure générale
        │
        ├──── Alimentation 24V ──── Mini-pompes à air (8×)
        │                      └── Moteurs NEMA 23 (via TMC2209)
        │                      └── Moteurs NEMA 17 (via TMC2209)
        │
        └──── Convertisseur DC/DC ──── 5V / 3.3V
                    │
                    ├── ESP32-S3 Plus (cerveau central)
                    │       ├── LIDAR (UART)
                    │       ├── TMC2209 × 4 (UART)
                    │       ├── PCA9685 (I²C)
                    │       │       └── 9 servo-moteurs
                    │       ├── Tirette magnétique (GPIO)
                    │       ├── Switch stratégie (GPIO)
                    │       ├── Bouton initialisation (GPIO)
                    │       └── Commande pompes (GPIO → transistors)
                    └── BAU (coupure matérielle)
```

---

## 2. Alimentation

### 2.1 Batterie principale

| Paramètre | Valeur |
|---|---|
| Marque / Référence | Parkside — batterie Li-Ion |
| Tension nominale | 24 V |

La batterie Parkside 24 V alimente l'ensemble du robot. Elle a été choisie pour sa disponibilité, son rapport prix/capacité et la sécurité de sa chimie Li-Ion (BMS intégré).

(image de la batterie Parkside montée dans le robot)

### 2.2 Distribution et régulation

La tension 24 V de la batterie est distribuée :
- **Directement à 24 V** vers les moteurs pas à pas (via les drivers TMC2209) et les mini-pompes
- **Via un convertisseur DC/DC** abaisseur vers 5 V pour l'alimentation de l'ESP32, du PCA9685 et des servo-moteurs

### 2.3 Arrêt d'urgence (BAU)

Le **bouton d'arrêt d'urgence** (BAU) est câblé en coupure matérielle sur le positif de la batterie. Il coupe **toute alimentation** du robot instantanément, indépendamment du logiciel. Son emplacement sur le robot respecte l'exigence du règlement : accessible de l'extérieur, bien visible, de couleur rouge.

(image du BAU sur le robot)

---

## 3. Microcontrôleur — ESP32-S3 Plus

### 3.1 Présentation

L'**ESP32-S3 Plus** est le cerveau unique du robot. Ce microcontrôleur dispose de :

- Deux cœurs Xtensa LX7 cadencés à 240 MHz
- Wi-Fi 802.11 b/g/n et Bluetooth 5.0 (non utilisés en match, antenne couverte)
- USB natif (utilisé pour la programmation et le debug)
- Nombreux GPIO, interfaces UART, I²C, SPI
- RAM suffisante pour le traitement des données LIDAR en temps réel

### 3.2 Affectation des broches

|  Fonction | Composant |
|---|---|
| UART TX — LIDAR | LIDAR |
| UART RX — LIDAR | LIDAR |
| UART — TMC2209 NEMA 23 gauche | Driver 1 |
| UART — TMC2209 NEMA 23 droit | Driver 2 |
| UART — TMC2209 NEMA 17 haut | Driver 3 |
| UART — TMC2209 NEMA 17 bas | Driver 4 |
| I²C SDA — PCA9685 | PCA9685 |
| I²C SCL — PCA9685 | PCA9685 |
| Tirette magnétique | GPIO entrée |
| Switch stratégie | GPIO entrée |
| Bouton initialisation | GPIO entrée |
| Commande pompes (groupe 1) | Transistor/relais |
| Commande pompes (groupe 2) | Transistor/relais |

### 3.3 Environnement de développement

Le code du rbot est développé sous **Visual Studio Code** avec l'extension **PlatformIO**
Le code source est disponible sur le dépôt GitHub d'unimakers.

---

## 4. Moteurs pas à pas et drivers TMC2209

### 4.1 Moteurs de déplacement — NEMA 23

Deux moteurs pas à pas **NEMA 23** entraînent les roues motrices gauche et droite du robot.

| Paramètre | Valeur |
|---|---|
| Format | NEMA 23 |
| Pas angulaire | 1,8° (200 pas/tour) |

### 4.2 Moteurs d'élévation — NEMA 17

Deux moteurs pas à pas **NEMA 17** actionnent l'axe vertical de l'actionneur.

| Paramètre | Valeur |
|---|---|
| Format | NEMA 17 |
| Pas angulaire | 1,8° (200 pas/tour) |

### 4.3 Drivers TMC2209 SilentStepStick

Chaque moteur pas à pas est piloté par un driver **TMC2209 SilentStepStick** de Trinamic. Ces drivers ont été retenus pour plusieurs raisons :

- **Mode StealthChop** : déplacement ultra-silencieux, sans bruit de pas caractéristique
- **Communication UART** : configuration avancée depuis l'ESP32 (courant, micro-pas, protection thermique)
- **Détection de blocage (StallGuard)** : peut servir de fin de course logiciel sur l'axe d'élévation
- **Micro-stepping** : jusqu'à 256 micro-pas pour un mouvement très fluide

---

## 5. Extension servo-moteurs — PCA9685

### 5.1 Présentation

L'**ESP32-S3 Plus** ne disposant pas de suffisamment de sorties PWM matérielles pour piloter 9 servo-moteurs simultanément, un **expandeur PCA9685** est utilisé.

Le PCA9685 est un contrôleur PWM 16 canaux communicant en **I²C**. Il génère de façon autonome les signaux PWM pour chaque servo-moteur, déchargeant totalement l'ESP32 de cette tâche.

| Paramètre | Valeur |
|---|---|
| Interface | I²C |
| Adresse I²C | `0x40` (par défaut) |
| Nombre de canaux | 16 (9 utilisés) |
| Fréquence PWM | 50 Hz (standard servo) |

### 5.2 Problème rencontré — PCA9685 non fonctionnel en compétition

⚠️ **Problème critique identifié**

Lors de la compétition, le PCA9685 n'a **pas pu être codé dans les temps**. La bibliothèque et la communication I²C n'ont pas été finalisées avant les matchs, ce qui a rendu **8 des 9 servo-moteurs inopérants** (hors servo thermomètre qui était peut-être piloté différemment).

**Conséquence :** le système de préhension à ventouses n'a pas pu être utilisé lors des matchs. La stratégie a dû être adaptée (poussée des blocs uniquement).

**Piste de correction pour la prochaine édition :**
- Tester l'I²C et le PCA9685 dès le début du projet, sur une maquette isolée
- Utiliser la bibliothèque `Adafruit_PWMServoDriver` qui est éprouvée sur ESP32
- Mettre en place des tests unitaires électroniques indépendants du reste du code

---

## 6. Capteur de navigation — LIDAR

### 6.1 Présentation

Le robot est équipé d'un **LIDAR** placé au sommet du robot, centré, avec un dégagement à 360°. Ce capteur mesure la distance aux obstacles dans toutes les directions, permettant à l'ESP32 de détecter le robot adverse et les éléments de la table.

(image du LIDAR monté sur le robot)

### 6.2 Problème rencontré — points fantômes

⚠️ **Problème rencontré et résolu**

En phase de test, le LIDAR détectait des **points fantômes** : des obstacles inexistants qui déclenchaient des arrêts intempestifs du robot. Ce problème était dû à des **réflexions parasites** sur les parois transparentes du châssis en acrylique et/ou à des vibrations du robot en mouvement.

**Solution appliquée :** ajustement de la **tolérance de détection** dans le code — filtrage des points dont la distance est inférieure à un seuil (correspondant au rayon du robot) et des points dont la variation entre deux scans est supérieure à un seuil. Le problème a été résolu et le LIDAR fonctionne de façon fiable en match.

---

## 7. Interfaces utilisateur embarquées

### 7.1 Tirette magnétique de départ

La **tirette magnétique** est le dispositif de démarrage officiel conforme au règlement. Lorsqu'elle est extraite physiquement du connecteur, un signal GPIO passe à l'état haut et déclenche le démarrage du programme de match.

- **Type :** contact magnétique reed switch ou aimant + capteur Hall
- **Logique :** tirette présente = GPIO LOW → tirette retirée = GPIO HIGH → démarrage

### 7.2 Switch de stratégie (ON/ON)

Un **switch à deux positions** permet de sélectionner l'une des deux stratégies programmées avant le match, en fonction du côté assigné par les arbitres (équipe bleue ou équipe jaune).

- **Type :** interrupteur ON/ON (deux positions stables)
- **Position 0 :** stratégie équipe bleue
- **Position 1 :** stratégie équipe jaune

### 7.3 Bouton d'initialisation

Le **bouton poussoir d'initialisation** permet de lancer la séquence de mise à zéro du robot (recalage des moteurs pas à pas sur les fins de course, initialisation du LIDAR). Il est actionné manuellement avant chaque match, une fois le robot posé sur la table.

- **Type :** bouton poussoir normalement ouvert (NO)

---

## 8. Mini-pompes à air et ventouses

### 8.1 Architecture du système pneumatique

Le système de préhension utilise **8 mini-pompes à air** regroupées en **4 paires de 2**, chaque paire alimentant un ensemble de ventouses.

### 8.2 Commande des pompes

Les pompes sont alimentées en 24 V et commandées par l'ESP32 via des **transistors ou relais** (le signal logique 3,3 V de l'ESP32 ne peut pas piloter directement la charge 24 V).

---

## 9. Schéma électronique

Le schéma complet de câblage a été réalisé sous **KiCad** .

(image du schéma électronique complet — vue générale)

(image du schéma de détail alimentation et distribution 24V)

(image du schéma de détail ESP32 et périphériques)

Les fichiers source du schéma sont disponibles sur le dépôt GitHub d'unimakers.

---

## 10. Liste du matériel électronique (BOM)

| Composant | Référence / Modèle | Quantité | Rôle |
|---|---|---|---|
| Microcontrôleur | ESP32-S3 Plus | 1 | Cerveau central |
| Driver moteur | TMC2209 SilentStepStick | 4 | Pilotage moteurs pas à pas |
| Expandeur PWM | PCA9685 | 1 | Pilotage 9 servo-moteurs |
| LIDAR |  | 1 | Détection obstacles |
| Moteur pas à pas | NEMA 23 | 2 | Déplacement (roues) |
| Moteur pas à pas | NEMA 17 | 2 | Élévation actionneur |
| Servo-moteur |  | 9 | Actionneurs + thermomètre |
| Mini-pompe à air |  | 8 | Préhension ventouses |
| Batterie | Parkside Li-Ion 24 V | 1 | Alimentation principale |
| Convertisseur DC/DC |  | 1 | Régulation 5 V / 3,3 V |
| BAU | Bouton d'arrêt d'urgence rouge | 1 | Sécurité réglementaire |
| Tirette magnétique |  | 1 | Démarrage match |
| Switch stratégie | Interrupteur ON/ON | 1 | Sélection stratégie |
| Bouton initialisation | Bouton poussoir NO | 1 | Init moteurs |
---

## 11. Procédure de mise en route

Avant chaque match, suivre cette procédure dans l'ordre :

1. ☐ Vérifier l'état de charge de la batterie
2. ☐ Insérer la tirette magnétique (position « sécurité »)
3. ☐ Positionner le switch stratégie sur le bon côté (bleu / jaune)
4. ☐ Brancher la batterie
5. ☐ Attendre l'initialisation de l'ESP32
6. ☐ Poser le robot sur la table dans la zone de départ
7. ☐ Appuyer sur le bouton d'initialisation → séquence de recalage
8. ☐ Attendre la fin de l'initialisation
9. ☐ Sur signal des arbitres : extraire la tirette → démarrage du match

---
