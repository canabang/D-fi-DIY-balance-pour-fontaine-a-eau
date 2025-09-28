# Configuration ESPHome - Guide Technique

Ce document détaille la configuration ESPHome de la balance connectée.

## Architecture des Fichiers

### `esp-pet-scales.yaml` - Point d'Entrée
```yaml
substitutions:
  name: "esp-pet-scales"          # Nom du device
  idname: "esp_pet_scales"        # ID interne (underscores seulement)
  board: esp32dev                 # Type de carte ESP32
  type: esp-idf                   # Framework (esp-idf ou arduino)
  loglevel: DEBUG                 # Niveau de logs
  static_ip: 192.168.xxx.xxx      # IP fixe

packages:
  base: !include .base.yaml       # Configuration ESP32 de base
  hx711: !include .balance_eau.yaml  # Logique spécifique balance
```

### `base.yaml` - Configuration ESP32 Standard
Contient tous les éléments standards d'un ESP32 :
- Configuration WiFi avec IP fixe
- OTA (mise à jour sans fil)
- Serveur web intégré
- Capteurs de diagnostic (WiFi, température, uptime)
- Synchronisation temporelle avec Home Assistant

### `balance_eau.yaml` - Cœur de la Balance

#### Variables Globales
```yaml
globals:
  - id: water_tare_offset         # Point zéro (valeur brute à vide)
    type: float
    restore_value: true
    initial_value: "186585.0"     # À ajuster selon votre capteur
  - id: water_full_scale_raw      # Point haut (valeur brute à plein)
    type: float
    restore_value: true
    initial_value: "229548.0"     # À ajuster selon votre capteur
  - id: water_full_scale_weight   # Poids de référence connu
    initial_value: "1.715"        # Votre poids de test en kg
  - id: water_scale_factor        # Facteur de conversion calculé
    initial_value: "25.051"       # Sera recalculé automatiquement
```

### ⚠️ Configuration Initiale des Valeurs

**Première utilisation** : Les valeurs `186585.0` et `229548.0` sont des exemples de mon capteur. Voici comment configurer les vôtres :

#### Méthode 1 : Démarrage par défaut (Recommandée)
1. **Laissez les valeurs d'exemple** lors de la première compilation
2. **Flashez** votre ESP32
3. **Observez les logs** ou le capteur "Eau Valeur Brute" dans Home Assistant
4. **Notez la valeur brute à vide** (exemple: 150000)
5. **Modifiez `water_tare_offset`** dans le code avec cette valeur
6. **Recompilez et reflashez**

#### Méthode 2 : Estimation initiale
Si vous voulez partir sur de bonnes bases :
```yaml
globals:
  - id: water_tare_offset
    initial_value: "100000.0"    # Valeur conservatrice
  - id: water_full_scale_raw  
    initial_value: "150000.0"    # Différence de ~50000 typique
```

#### Méthode 3 : Calibration complète après flash
1. **Utilisez n'importe quelles valeurs** initiales
2. **Une fois flashé**, placez la fontaine vide
3. **Cliquez "Eau Tare (Vide)"** → met à jour `water_tare_offset` automatiquement
4. **Ajoutez un poids connu**, cliquez "Eau Tare (Plein)" → met à jour `water_full_scale_raw`

> 💡 **Astuce** : Les valeurs sont sauvegardées automatiquement (`restore_value: true`), donc une fois calibré, ça reste même après redémarrage !

#### Capteur HX711 Principal
```yaml
sensor:
  - platform: hx711
    dout_pin: 16      # Pin DATA
    clk_pin: 4        # Pin CLOCK
    gain: 128         # Amplification
    update_interval: 30s
```

#### Conversion en Poids
Le capteur template `water_scale_value` :
1. Calcule le facteur d'échelle dynamiquement
2. Convertit la valeur brute en poids
3. Applique des vérifications de sécurité
4. Log toutes les étapes pour le debug

#### Filtrage Multicouche
```yaml
filters:
  - median:                          # Filtre médiane (7 valeurs)
      window_size: 7
      send_first_at: 4
  - lambda: |                        # Suppression variations <10g
      if (x < 0.010) return 0.000;
  - sliding_window_moving_average:   # Moyenne glissante (5 valeurs)
      window_size: 5
  - lambda: |                        # Bornes finales
      if (x > 2.000) return 0.000;
      if (x < 0.000) return 0.000;
```

## Calibration Dynamique

### Principe
Au lieu d'une calibration fixe, le système utilise 2 points de référence :
- **Point zéro** : Valeur brute à vide (`water_tare_offset`)
- **Point plein** : Valeur brute à un poids connu (`water_full_scale_raw`)

### Calcul du Facteur
```cpp
float scale_factor = (full_scale_raw - tare_offset) / reference_weight;
float weight = (current_raw - tare_offset) / scale_factor;
```

### Boutons de Calibration
- `"Eau Tare (Vide)"` : Définit le point zéro
- `"Eau Tare (Plein)"` : Définit le point haut
- Curseur `"Eau Poids de référence"` : Ajuste le poids de référence

## Auto-Diagnostic

### Codes d'Erreur
- **0** : Erreur capteur (valeur brute < 1000)
- **1** : Hors calibration (écart > 100000 du point zéro)
- **2** : OK

### Vérifications Automatiques
```yaml
interval:
  - interval: 12h    # Toutes les 12h
    then:
      - if:           # Si dérive détectée
          condition:
            lambda: 'return id(water_scale_value).state < -0.05;'
          then:
            - lambda: |   # Recalibration automatique du zéro
                id(water_tare_offset) = id(water_scale_raw).state;
```

## Personnalisation

### Changer les GPIO
Dans `balance_eau.yaml` :
```yaml
sensor:
  - platform: hx711
    dout_pin: 16    # Changez ici pour DATA
    clk_pin: 4      # Changez ici pour CLOCK
```

### Ajuster les Filtres
- `window_size` : Taille des fenêtres de filtrage
- `send_every` : Fréquence d'envoi des valeurs
- Seuils dans les `lambda` : Limites acceptables

### Modifier la Précision
```yaml
accuracy_decimals: 3    # Nombre de décimales
step: 0.005            # Pas du curseur de référence
```

## Logs et Debug

### Niveaux de Log
- `DEBUG` : Recommandé pour setup initial
- `INFO` : Usage normal
- `WARN` : Erreurs importantes seulement

### Messages Utiles
```
[D][water_scale] Valeur brute: 200000, Tare: 186585, Facteur: 25.051, Poids: 0.536
[I][water_scale] Nouvelle tare définie: 186585.0
[W][water_scale] Valeur hors limites détectée: -0.120, ignorée
```

## Optimisations

### Performances
- `update_interval: 30s` : Balance entre réactivité et stabilité
- `restore_value: true` : Sauvegarde des calibrations
- `internal: false` : Expose les capteurs de debug si nécessaire

### Stabilité
- Filtrages multiples pour éliminer le bruit
- Vérifications de cohérence à chaque mesure
- Recalibration automatique en cas de dérive

## Intégration Home Assistant

### Entités Créées
- Capteurs principaux (poids, %)
- Contrôles (boutons, curseurs)
- Diagnostics (état, valeurs brutes)
- Informations système (WiFi, uptime)

### Automatisations Recommandées
- Notification niveau bas
- Historique de consommation
- Maintenance préventive

---

*Configuration robuste et flexible pour une mesure précise !*
