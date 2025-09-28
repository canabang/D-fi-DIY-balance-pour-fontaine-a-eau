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

**Coût total : ~25€**

## Installation

### 1. Câblage
Suivez le guide détaillé dans [`branchement.md`](branchement.md)

### 2. Configuration ESPHome

1. **Copiez les fichiers** dans votre dossier ESPHome
2. **Modifiez `esp-pet-scales.yaml`** :
   ```yaml
   substitutions:
     name: "votre-balance"
     static_ip: 192.168.1.100  # Votre IP
   ```
3. **Compilez et flashez** avec ESPHome

### 3. Calibration

1. Placez la fontaine vide sur la balance
2. Dans Home Assistant, cliquez sur "Eau Tare (Vide)"
3. Remplissez avec un poids connu (ex: 1.5L = 1.5kg)
4. Ajustez "Eau Poids de référence" à la valeur exacte
5. Cliquez sur "Eau Tare (Plein)"

C'est calibré ! ✅

## Fonctionnalités

### Capteurs dans Home Assistant
- `sensor.eau_poids_gamelle` - Poids en kg (précision 3 décimales)
- `sensor.eau_percentage` - Niveau en % (0-100)
- `sensor.eau_etat_capteur` - État (OK/Erreur/Hors calibration)

### Contrôles
- Boutons de calibration vide/plein
- Curseur de poids de référence ajustable
- Instructions intégrées

### Auto-diagnostic
- Détection automatique des erreurs
- Codes d'erreur explicites
- Recalibration automatique si dérive

## Filtrage Intelligent

Le système filtre automatiquement :
- Les vibrations et mouvements
- Les valeurs aberrantes
- Les fluctuations minimes (<10g)
- Les erreurs de lecture

## Structure des Fichiers

```
├── esp-pet-scales.yaml    # Configuration principale
├── base.yaml             # Base ESP32 (WiFi, OTA, etc.)
├── balance_eau.yaml      # Logique balance complète
└── branchement.md        # Guide de câblage détaillé
```

## Personnalisation

### Changer les seuils
Dans `balance_eau.yaml`, section `globals` :
- `water_full_scale_weight` : Poids maximum de votre fontaine
- Filtres `lambda` : Ajuster les seuils selon vos besoins

### Ajouter des automatisations
Exemples dans Home Assistant :
```yaml
# Notification niveau bas
- trigger:
    platform: numeric_state
    entity_id: sensor.eau_percentage
    below: 20
  action:
    service: notify.mobile_app
    data:
      message: "Niveau d'eau bas : {{ states('sensor.eau_percentage') }}%"
```

## Dépannage

### La balance ne répond pas
- Vérifiez les connexions GPIO 16 et 4
- Contrôlez l'alimentation 3.3V du HX711

### Valeurs erratiques
- Vérifiez les soudures des load cells
- Recalibrez avec le bouton "Tare (Vide)"
- Consultez les logs ESPHome

### Dérive dans le temps
- La recalibration automatique se déclenche toutes les 12h
- Forcer avec les boutons si nécessaire

## Évolutions Possibles

- Ajout capteur de température de l'eau
- Détection de mouvement des animaux
- Prédiction d'autonomie restante
- Graphiques de consommation avancés

## Contribution

N'hésitez pas à :
- Signaler des bugs via les issues
- Proposer des améliorations
- Partager vos adaptations

---

**Projet Open Source** - Adaptable pour d'autres usages : balance de croquettes, stock, pluviomètre, etc.

*Transformez votre fontaine en fontaine vraiment intelligente !* 🐱💧
