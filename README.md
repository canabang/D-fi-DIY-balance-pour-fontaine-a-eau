# Balance Connectée pour Fontaine à Eau 💧

Une balance DIY qui surveille le niveau d'eau de votre fontaine pour animaux en temps réel via Home Assistant.

## Le Problème

Les fontaines à eau connectées ne proposent pas de retour sur le niveau d'eau restant. Impossible de savoir s'il faut remplir sans soulever le réservoir !

## La Solution

Cette balance se glisse sous votre fontaine et vous donne :
- Le poids exact de l'eau (en kg)
- Le niveau en pourcentage (0-100%)
- Des alertes automatiques
- Un historique de consommation

## Matériel Nécessaire

- 4 capteurs de charge (load cells) 3 fils
- 1 module HX711
- 1 ESP32 WROOM Dev Kit
- Fils de connexion
- Support (planche à découper par exemple)

**Coût total : ~25€**

## Installation

### 1. Câblage

Suivez le guide détaillé dans [**branchement.md**](branchement.md) qui contient :
- Schéma de câblage complet avec image
- Position des 4 capteurs aux coins
- Code couleur des fils (Rouge, Noir, Blanc)
- Configuration en pont de Wheatstone
- Connexions HX711 vers ESP32

### 2. Configuration ESPHome

#### Structure des fichiers
```
├── esp-pet-scales.yaml    # Configuration principale
├── base.yaml             # Base ESP32 (WiFi, OTA, etc.)
├── balance_eau.yaml      # Logique balance complète
└── branchement.md        # Guide de câblage détaillé
```

#### Configuration principale
1. **Modifiez `esp-pet-scales.yaml`** :
   ```yaml
   substitutions:
     name: "votre-balance"
     static_ip: 192.168.1.100  # Votre IP
     loglevel: DEBUG           # Pour le setup initial
   ```

2. **Adaptez `base.yaml`** (si nécessaire) :
   - Gateway et DNS de votre réseau
   - Timezone si différente d'Europe/Paris

3. **Personnalisez `balance_eau.yaml`** (optionnel) :
   - GPIO pins (par défaut 16 et 4)
   - Seuils de filtrage selon vos besoins

#### Première utilisation - Valeurs de calibration
Les valeurs d'exemple dans `balance_eau.yaml` sont spécifiques à mon capteur :
```yaml
globals:
  - id: water_tare_offset
    initial_value: "186585.0"  # À adapter à votre capteur
  - id: water_full_scale_raw
    initial_value: "229548.0"  # À adapter à votre capteur
```

**3 méthodes pour démarrer :**

**Méthode recommandée** : Gardez les valeurs d'exemple, flashez, puis calibrez via Home Assistant
1. Compilez et flashez avec les valeurs par défaut
2. Observez le capteur "Eau Valeur Brute" dans Home Assistant
3. Calibrez directement via les boutons (voir section Calibration)

**Méthode estimation** : Utilisez des valeurs conservatrices
```yaml
water_tare_offset: "100000.0"
water_full_scale_raw: "150000.0"
```

**Méthode manuelle** : Si vous voulez partir sur de bonnes bases
1. Flashez avec n'importe quelles valeurs
2. Notez la valeur brute à vide dans les logs
3. Modifiez `water_tare_offset`, recompilez

4. **Compilez et flashez** avec ESPHome

### 3. Calibration

Une fois l'ESP32 flashé et connecté à Home Assistant :

1. **Placez la fontaine vide** sur la balance
2. Dans Home Assistant, cliquez sur **"Eau Tare (Vide)"**
3. **Ajoutez un poids connu** (ex: 1.5L = 1.5kg)
4. **Ajustez "Eau Poids de référence"** à la valeur exacte via le curseur
5. Cliquez sur **"Eau Tare (Plein)"**

✅ **C'est calibré !** Les valeurs sont automatiquement sauvegardées.

## Fonctionnalités

### Capteurs dans Home Assistant
- `sensor.eau_poids_gamelle` - Poids en kg (précision 3 décimales)
- `sensor.eau_percentage` - Niveau en % (0-100)
- `sensor.eau_etat_capteur` - État (OK/Erreur/Hors calibration)
- `sensor.eau_valeur_brute` - Valeur brute pour diagnostic

### Interface de contrôle
- **Boutons de calibration** : Tare vide/plein
- **Curseur de référence** : Ajustement du poids de référence (0.5-3.0 kg)
- **Instructions intégrées** : Guide de calibration toujours disponible
- **Diagnostics** : Codes d'erreur et états en temps réel

### Filtrage intelligent multicouche
Le système filtre automatiquement :
- **Filtre médiane** (7 valeurs) - Élimine les valeurs aberrantes
- **Seuil de variation** - Ignore les fluctuations < 10g
- **Moyenne glissante** (5 valeurs) - Stabilise les mesures
- **Bornes de sécurité** - Évite les erreurs de lecture

### Auto-diagnostic et maintenance
- **Codes d'erreur explicites** : 0=Erreur capteur, 1=Hors calibration, 2=OK
- **Recalibration automatique** toutes les 12h si dérive détectée
- **Logs détaillés** pour debugging
- **Vérifications de cohérence** à chaque mesure

## Automatisations Suggérées

### Notification niveau bas
```yaml
automation:
  - trigger:
      platform: numeric_state
      entity_id: sensor.eau_percentage
      below: 20
    action:
      service: notify.mobile_app
      data:
        message: "Niveau d'eau bas : {{ states('sensor.eau_percentage') }}%"
```

### Maintenance préventive
```yaml
automation:
  - trigger:
      platform: state
      entity_id: sensor.eau_etat_capteur
      to: "Hors calibration"
    action:
      service: notify.mobile_app
      data:
        message: "Balance nécessite une recalibration"
```

## Personnalisation Avancée

### Changer les GPIO
Dans `balance_eau.yaml` :
```yaml
sensor:
  - platform: hx711
    dout_pin: 16  # Pin DATA - modifiable
    clk_pin: 4    # Pin CLOCK - modifiable
```

### Ajuster les seuils de filtrage
```yaml
# Exemple : balance pour gros réservoir
filters:
  - lambda: |-
      if (x < 0.050) return 0.000;  # Seuil 50g au lieu de 10g
  - lambda: |-
      if (x > 5.000) return 0.000;  # Limite 5kg au lieu de 2kg
```

### Modifier la précision
```yaml
sensor:
  - platform: template
    accuracy_decimals: 2    # 2 décimales au lieu de 3
    
number:
  - platform: template
    step: 0.010            # Pas de 10g au lieu de 5g
```

## Dépannage

### La balance ne répond pas
- Vérifiez les connexions GPIO 16 (DATA) et 4 (CLOCK)
- Contrôlez l'alimentation 3.3V du HX711
- Consultez les logs ESPHome (`loglevel: DEBUG`)

### Valeurs erratiques
- Vérifiez les soudures des load cells (pont de Wheatstone crucial)
- Recalibrez avec le bouton "Eau Tare (Vide)"
- Vérifiez la stabilité du support

### Messages d'erreur courants
```
[W][water_scale] Facteur d'échelle invalide
```
→ Recalibrez les points vide/plein

```
[W][water_scale] Valeur hors limites détectée
```
→ Normal, le filtrage élimine les aberrations

### Dérive dans le temps
- La recalibration automatique se déclenche toutes les 12h
- Forcez avec les boutons si nécessaire
- Vérifiez la température ambiante (influence possible)

## Documentation Technique

Pour comprendre en détail le fonctionnement du code ESPHome, consultez [**esphome.md**](esphome.md) qui contient :
- Architecture détaillée des fichiers
- Explication de la calibration dynamique
- Détail des filtres et algorithmes
- Codes d'erreur et logs
- Guide de personnalisation avancée

## Évolutions Possibles

- Ajout capteur de température de l'eau
- Détection de mouvement des animaux
- Prédiction d'autonomie restante
- Intégration avec d'autres capteurs (croquettes, etc.)
- Graphiques de consommation avancés
- Alertes prédictives basées sur l'historique

## Contribution

N'hésitez pas à :
- Signaler des bugs via les issues
- Proposer des améliorations
- Partager vos adaptations et personnalisations
- Contribuer à la documentation

---

**Projet Open Source** - Adaptable pour d'autres usages : balance de croquettes, stock, pluviomètre, pesage divers.

*Transformez votre fontaine en fontaine vraiment intelligente !* 🐱💧
