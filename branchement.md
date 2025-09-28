README - Tutoriel pour le Circuit de Mesure de Poids
Introduction

Ce projet utilise des capteurs de charge de 50 kg, un amplificateur HX711 et un microcontrôleur ESP32 pour mesurer le poids. Ce document fournit des instructions sur la configuration du circuit et son utilisation.
Matériel Nécessaire

    4 x Capteurs de Charge de 50 kg
    1 x HX711 - Amplificateur de Pont
    1 x ESP32 Wroom Dev Kit
    Fils de connexion
    Une planche à pain (breadboard) pour le prototypage

Schéma de Connexion
Connexions des Capteurs de Charge

Chaque capteur de charge a trois broches :

    W (Masse)
    R (Signal de Sortie)
    B (Connexion Commune)

<img width="3000" height="2104" alt="balance_eau" src="https://github.com/user-attachments/assets/6a5d547d-3ff6-4e13-be14-8cdf29776a75" />


Connectez les capteurs comme suit :

    Connectez les broches W de tous les capteurs ensemble.
    Connectez les broches R de chaque capteur aux broches correspondantes de l'HX711.

Connexions de l'HX711

    GND : Connectez à la broche GND de l'ESP32.
    3.3/3.5V Supply : Connectez à la broche 3V3 de l'ESP32.
    DATA (OUT) : Connectez à GPIO 16 de l'ESP32.
    SCK - CLOCK (IN) : Connectez à GPIO 4 de l'ESP32.
    E+ : Connectez à la broche R d'un des capteurs de charge.
    E- : Connectez à la broche R d'un autre capteur de charge.

Connexions de l'ESP32

    Alimentez l'ESP32 avec une source de 3.3V.
    Assurez-vous que les broches GPIO 16 et GPIO 4 sont correctement connectées à l'HX711.
