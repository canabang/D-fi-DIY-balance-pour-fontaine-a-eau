Tutorial - Montage d'une Balance avec 4 Capteurs de Charge et ESP32
Ce guide détaille le câblage d'une balance utilisant 4 capteurs de charge (load cells) connectés à un module HX711 et un ESP32, compatible avec ESPHome.


<img width="3000" height="2104" alt="balance_eau" src="https://github.com/user-attachments/assets/11a10e9f-0038-48f1-a99d-082ba87a90b9" />


Composants requis

4 capteurs de charge (load cells) à 3 fils
1 module HX711 (amplificateur pour capteurs de charge)
1 ESP32 WROOM Dev Kit
Fils de connexion (respecter le code couleur)

Position des capteurs
Les 4 capteurs sont positionnés aux coins d'une plateforme :

Haut Gauche (Instance 1)
Haut Droit (Instance 2)
Bas Gauche (Instance 3)
Bas Droit (Instance 4)

Code couleur des fils
Chaque capteur possède 3 fils :

R (Rouge) : Signal
B (Noir/Bleu) : Masse/Référence
W (Blanc) : Excitation

Schéma de câblage
1. Connexions entre capteurs (Configuration en pont de Wheatstone)
Haut Gauche (Instance 1) :

B (noir) → B (noir) du Bas Gauche (Instance 3)
W (blanc) → W (blanc) du Bas Gauche (Instance 3)
R (rouge) → HX711 (E+)

Haut Droit (Instance 2) :

B (noir) → B (noir) du Bas Droit (Instance 4)
W (blanc) → W (blanc) du Bas Droit (Instance 4)
R (rouge) → HX711 (A-)

Bas Gauche (Instance 3) :

B (noir) → B (noir) du Haut Gauche (Instance 1)
W (blanc) → W (blanc) du Haut Gauche (Instance 1)
R (rouge) → HX711 (A+)

Bas Droit (Instance 4) :

B (noir) → B (noir) du Haut Droit (Instance 2)
W (blanc) → W (blanc) du Haut Droit (Instance 2)
R (rouge) → HX711 (E-)

2. Connexions HX711 vers ESP32
Module HX711 :

GND → ESP32 GND
VCC/3.3V → ESP32 3V3
DT/DATA → ESP32 GPIO 16
SCK/CLOCK → ESP32 GPIO 4

Résumé des connexions par côtés
Côté Gauche (Haut ↔ Bas)

Haut Gauche B ↔ Bas Gauche B
Haut Gauche W ↔ Bas Gauche W

Côté Droit (Haut ↔ Bas)

Haut Droit B ↔ Bas Droit B
Haut Droit W ↔ Bas Droit W

Signaux vers HX711

Haut Gauche R → E+
Haut Droit R → A-
Bas Gauche R → A+
Bas Droit R → E-

Points importants
⚠️ Attention aux connexions :

Vérifiez bien le code couleur de vos capteurs (peut varier selon le fabricant)
Les connexions en pont de Wheatstone sont cruciales pour le bon fonctionnement
Cette configuration connecte les capteurs du même côté ensemble (gauche avec gauche, droit avec droit)
Utilisez des connexions soudées pour éviter les faux contacts
Testez la continuité avant la mise sous tension
