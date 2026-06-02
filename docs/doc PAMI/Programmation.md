---
layout: default
parent: PAMI
nav_order: 2
title: Prototype
---

# Programmation du PAMI

## Objectifs

Le code devait permettre au PAMIs de faire autant de points qu'ils pouvaient réaliser, en effectuant les tâches suivantes :
* aller jusqu'à un garde-manger;
* éviter les adversaires et autres obstacles;
* activer son actionneur une fois arrivé.

## Logique de fonctionnement

Pour que le PAMI fonctionne, nous avons opté pour une division des tâches en fonction et procédures, dans le but d'éviter autant que possible les répétitions et d'avoir un code **optimisé**.

![algorithme du PAMI explicant la logique de la programmation](/docs/images/Flowchart.png "Algorithme du PAMI")

Pour réaliser notre code, nous avons utilisé les bibliothèques :
* **Accelstepper** pour piloter nos moteurs pas à pas,
* **Ultrasonic** pour la détection avec nos capteurs ultrasoniques HC-SR04,
* **Esp32Servo** afin de donner l'angle nécessaire pour le servomoteur.

Nous les avons mis à profit dans l'environnement **VScode** avec PlatformIO, ainsi nous pouvions envoyer notre code sur **github** et travailler ensemble en temps réel.

Pour les déplacements, nous avons configuré notre code pour que notre Pami ne se déplace qu'en ligne droite, et en évitant les diagonales.Ainsi, le robot peut garder en mémoire ses coordonnées et son angle, et ses séquences sont plus facilement vérifiables et corrigeables.
Le code en lui même utilise beaucoup d'interactions entre les fonctions et les protocoles, permettant de s'adapter à chaque situation et d'avoir un fonctionnement très séquencé permettant aussi de mieux travailler sur différentes fonctionnalités simultanément.