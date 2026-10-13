# Dashboard Home Assistant — vue « Ioniq 5 »

Intégration véhicule : `kia_uvo`.
Vue `ioniq-5` du dashboard `dashboard-perso` (type *storage*, mode `sections`).

## Carte « batterie liquide »
- `custom:mushroom-entity-card` sur `sensor.ioniq_5_ev_battery_level`
- `card_mod` : niveau numérique (0–100 %) avec barre + vague animée.
- Seuils de couleur : `≤20` rouge, `≤50` ambre, sinon vert ; **cyan + vague rapide** en charge.
- Points clés du CSS :
  - `--liq-level: {{ level }}%` (niveau clampé 0–100)
  - `ha-card::before` = remplissage (`width: var(--liq-level)`)
  - `ha-card::after` = vague qui tourne (`left: calc(var(--liq-level) - 60px)`)
  - `animation: spin-wave var(--wave-speed) linear infinite` (`3s` en charge, `15s` sinon)

## Carte « Résumé recharge »
- `custom:stack-in-card` (fond arrondi, **teinte verte** au repos → dégradé quand ça charge)
  - `custom:mushroom-template-card` : `68 % · 3.4 kW` + sous-titre `→ 80 % · 267 km · 10 A · fin ≈ 20:15`
  - `custom:mini-graph-card` : puissance de charge sur **24 h**
- État « en charge » = `switch.ioniq_5_ev_charging == on` **OU** `sensor.de_ev_charger_etat == charging`

## Autres cartes de la vue
- **Climatisation** : `tile` sur `climate.ioniq_5_climate_control` (modes + température cible)
- **Données véhicule** : fraîcheur (`sensor.ioniq_5_derniere_interrogation`)
- **Pression pneus** : ok / alerte sur les 4 roues
- **Position** : ville + « à la maison » (`device_tracker.ioniq_5_location`)
- **Historique batterie** (24 h)
- **Limites de charge** : `Limite AC` / `Limite DC`

## Application via MCP
Écriture d'un dashboard *storage* avec `ha_config_set_dashboard` :
- `python_transform` pour modifier une carte de façon **ciblée** (sans renvoyer tout le config)
- requiert le `config_hash` obtenu via `ha_config_get_dashboard` (verrou optimiste)
- `return_screenshot` existe mais nécessite le moteur beta (add-on/sidecar)

## Doublons à surveiller
- `binary_sensor.ioniq_5_defrost` présent 2× (section « Commandes »)
