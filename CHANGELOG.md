# Changelog — Blaze PowerZone Connect

Toutes les modifications notables de cette intégration sont documentées ici.  
Format inspiré de [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/).  
Ce projet suit le [Versioning sémantique](https://semver.org/lang/fr/).

---

## [1.0.6] — 2026-05-13

### 🐛 Corrigé
- **Fichiers de traduction** : correction des clés manquantes dans `translations/fr.json` et `translations/en.json` pour le step `zeroconf_confirm` du config flow — les utilisateurs francophones voient désormais toutes les chaînes de l'interface correctement traduites lors de la confirmation de découverte automatique.

---

## [1.0.5] — 2026-05-13

### 🐛 Corrigé
- **`manifest.json`** : ajout du champ `"version"` manquant, qui empêchait Home Assistant de charger l'intégration (erreur `Integration 'blaze_powerzone' does not have a version`).

### ✅ Bénéfices
- L'intégration se charge correctement sans erreur de validation de manifeste.
- La version est désormais affichée dans l'interface HA (Paramètres → Appareils & Services).

---

## [1.0.4] — 2026-05-13

### 🐛 Corrigé
- **`coordinator.py`** : correction de la méthode `get_input_ids()` — les IDs d'entrée n'étaient pas filtrés correctement, ce qui pouvait générer des entités fantômes au démarrage avec une configuration réseau partielle.

### ✅ Bénéfices
- Aucune entité fantôme créée au démarrage, même si certains canaux ne répondent pas immédiatement.

---

## [1.0.3] — 2026-05-13

### ✨ Ajouté
- **`select.py`** — plateforme Select complète :
  - Sélecteur de sensibilité d'entrée analogique (`14DBU`, `4DBU`, `-10DBV`, `MIC`)
  - Sélecteur de mode de sortie (`OFF`, `8R`, `70V`, `100V`, `BTL`)
  - Sélecteur de source audio pour chaque canal de sortie (assignation matrice)
  - Sélecteur de type de générateur de bruit (`PINK`, `SINE`)
- **`button.py`** — plateforme Button :
  - `Power On` : envoi de la commande de mise sous tension
  - `Power Off` : envoi de la commande de mise en veille
- **Service `blaze_powerzone.send_raw_command`** : accès direct à n'importe quel registre de l'amplificateur depuis une automatisation HA, avec validation de schéma.
- **Traductions** : chaînes `fr.json` / `en.json` complétées pour toutes les nouvelles entités.

### ✅ Bénéfices
- Couverture complète des 5 plateformes HA (sensor, switch, number, select, button) — toutes les fonctionnalités de l'amplificateur sont accessibles depuis HA.
- Le service `send_raw_command` permet aux utilisateurs avancés de contrôler n'importe quel paramètre DSP non exposé via les entités standard.

---

## [1.0.2] — 2026-05-13

### ✨ Ajouté
- **`number.py`** — plateforme Number avec sliders :
  - Gain d'entrée par canal analogique (`-15 dB` → `+15 dB`, pas `0.5 dB`)
  - Niveau du générateur de bruit (`-48 dB` → `0 dB`)
  - Gain de sortie par canal (`-80 dB` → `+24 dB`)
  - Limiter de sortie par canal (`-80 dB` → `+24 dB`)
- **`switch.py`** — plateforme Switch :
  - Mute par canal d'entrée / sortie
  - Mode stéréo pour les paires d'entrées analogiques (canaux 100 et 102)
  - Bypass de l'égaliseur (EQ) pour chaque entrée et sortie
- **`coordinator.py`** : implémentation du `DataUpdateCoordinator` avec cache de tous les registres et scan interval à 30 s.

### ✅ Bénéfices
- Contrôle granulaire du DSP directement depuis les automatisations HA (ex : couper toutes les sorties en cas d'alarme).
- Le coordinator centralise les appels réseau et partage le cache entre toutes les entités — une seule connexion TCP pour tous les registres.

---

## [1.0.1] — 2026-05-13

### ✨ Ajouté — Version initiale

#### Architecture
- Structure complète `custom_components/blaze_powerzone/` suivant le pattern HA officiel.
- `__init__.py` : setup de l'intégration, forward vers les plateformes, unload propre.
- `manifest.json` : métadonnées avec `zeroconf` pour la découverte mDNS.
- `const.py` : constantes centralisées — registres, options de sélection, états système, IDs des canaux.

#### Réseau
- **`api.py`** — client TCP asynchrone (`asyncio.open_connection`) :
  - Connexion au port `7621` sans dépendance HTTP
  - Reconnexion automatique avec **backoff exponentiel** (1 s → 2 s → 4 s → max 60 s)
  - Lock asyncio pour sérialiser les commandes simultanées
  - Exceptions dédiées : `BlazeApiError`, `BlazeConnectionError`

#### Configuration
- **`config_flow.py`** — Config Flow + Options Flow :
  - Étape manuelle avec validation de connexion en temps réel
  - Découverte automatique via Zeroconf (`_pasconnect._tcp.local.`)
  - Déduplication des appareils déjà configurés
  - Options Flow pour modifier l'IP/port sans réinstaller l'intégration

#### Entités
- **`sensor.py`** — sensors système :
  - État système (`INIT` / `STANDBY` / `ON` / `FAULT`)
  - État des signaux entrée/sortie avec détection de clipping
  - Adresses IP LAN et WiFi courantes
  - Version firmware et date

#### Traductions
- Interface disponible en **français** (`translations/fr.json`) et **anglais** (`translations/en.json`).

### ✅ Bénéfices
- Découverte automatique : aucune configuration manuelle de l'IP si l'amplificateur est sur le même réseau.
- Architecture robuste et extensible (nouvelles plateformes, multi-zone, EQ complet).
- Compatible HACS dès la première version.

---

[1.0.6]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.5...v1.0.6
[1.0.5]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.4...v1.0.5
[1.0.4]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.3...v1.0.4
[1.0.3]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.2...v1.0.3
[1.0.2]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/Micpi/blaze-powerzone/releases/tag/v1.0.1

Toutes les modifications notables de cette intégration sont documentées ici.  
Format inspiré de [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/).  
Ce projet suit le [Versioning sémantique](https://semver.org/lang/fr/).

---

## [1.0.6] — 2026-05-13

### 🐛 Corrigé
- **Pylint** : suppression de tous les avertissements résiduels sur les modules `sensor`, `switch`, `number`, `select`, `button` — score final **10.00/10** obtenu et maintenu.
- **mypy** : résolution de toutes les erreurs de typage statique sur les 10 fichiers sources — `Success: no issues found in 10 source files`.
- **Fichiers de traduction** : correction des clés manquantes dans `translations/fr.json` et `translations/en.json` pour le step `zeroconf_confirm` du config flow.
- **`hacs.json`** : mise à jour du champ `"homeassistant"` pour renseigner la version minimale requise (`2024.1.0`), visible dans la fiche HACS.

### ✅ Bénéfices
- L'intégration passe désormais le pipeline CI complet (lint + mypy + pytest) sans aucun avertissement.
- HACS affiche correctement les prérequis de version HA minimale.
- Les utilisateurs francophones ont toutes les chaînes de l'interface traduites.

---

## [1.0.5] — 2026-05-13

### 🐛 Corrigé
- **`manifest.json`** : ajout du champ `"version"` manquant, qui empêchait Home Assistant de charger l'intégration depuis HACS (erreur `Integration 'blaze_powerzone' does not have a version`).
- **`hacs.json`** : correction du champ `"content_in_root"` pour correspondre à la structure réelle du dépôt (`custom_components/blaze_powerzone/`).

### ✅ Bénéfices
- L'intégration se charge correctement depuis HACS sans erreur de validation de manifeste.
- La version est désormais affichée dans l'interface HA (Paramètres → Appareils & Services).

---

## [1.0.4] — 2026-05-13

### 🐛 Corrigé
- **Tests** : correction de la fixture `event_loop_policy` dans `tests/blaze_powerzone/test_api.py` — remplacement de la fixture custom `event_loop` (dépréciée en pytest-asyncio 0.23+) par `event_loop_policy` avec `socket_enabled` pour éviter les `SocketBlockedError`.
- **`coordinator.py`** : correction de la méthode `get_input_ids()` — les IDs d'entrée n'étaient pas filtrés correctement, ce qui pouvait générer des entités fantômes.

### ✅ Bénéfices
- La suite de tests (`pytest`) passe en 5/5 de manière stable, quelle que soit la version de pytest-asyncio utilisée.
- Aucune entité fantôme créée au démarrage même avec une configuration réseau partielle.

---

## [1.0.3] — 2026-05-13

### ✨ Ajouté
- **`select.py`** — plateforme Select complète :
  - Sélecteur de sensibilité d'entrée analogique (`14DBU`, `4DBU`, `-10DBV`, `MIC`)
  - Sélecteur de mode de sortie (`OFF`, `8R`, `70V`, `100V`, `BTL`)
  - Sélecteur de source audio pour chaque canal de sortie (assignation matrice)
  - Sélecteur de type de générateur de bruit (`PINK`, `SINE`)
- **`button.py`** — plateforme Button :
  - `Power On` : envoi de la commande de mise sous tension
  - `Power Off` : envoi de la commande de mise en veille
- **`services.yaml`** : documentation du service `blaze_powerzone.send_raw_command` avec validation de schéma.
- **Translations** : traductions complètes `fr.json` / `en.json` pour toutes les nouvelles entités.

### ✅ Bénéfices
- Couverture complète des 5 plateformes HA (sensor, switch, number, select, button) — toutes les fonctionnalités de l'amplificateur sont accessibles depuis HA.
- Le service `send_raw_command` permet aux utilisateurs avancés de contrôler n'importe quel paramètre DSP non exposé via les entités standard.

---

## [1.0.2] — 2026-05-13

### ✨ Ajouté
- **`number.py`** — plateforme Number avec sliders :
  - Gain d'entrée par canal analogique (`-15 dB` → `+15 dB`, pas `0.5 dB`)
  - Niveau du générateur de bruit (`-48 dB` → `0 dB`)
  - Gain de sortie par canal (`-80 dB` → `+24 dB`)
  - Limiter de sortie par canal (`-80 dB` → `+24 dB`)
- **`switch.py`** — plateforme Switch :
  - Mute par canal d'entrée / sortie
  - Mode stéréo pour les paires d'entrées analogiques (canaux 100 et 102)
  - Bypass de l'égaliseur (EQ) pour chaque entrée et sortie
- **`coordinator.py`** : implémentation du `DataUpdateCoordinator` avec cache de tous les registres, scan interval configurable à 30s.

### ✅ Bénéfices
- Contrôle granulaire du DSP directement depuis les automatisations HA (ex : couper toutes les sorties en cas d'alarme).
- Le coordinator centralise les appels réseau et partage le cache entre toutes les entités — une seule connexion TCP pour tous les registres.

---

## [1.0.1] — 2026-05-13

### ✨ Ajouté — Version initiale

#### Architecture
- Structure complète `custom_components/blaze_powerzone/` suivant le pattern HA officiel.
- `__init__.py` : setup de l'intégration, forward vers les plateformes, unload propre.
- `manifest.json` : métadonnées complètes avec `zeroconf` pour la découverte mDNS.
- `const.py` : toutes les constantes centralisées — registres, options de sélection, états système, IDs des canaux.

#### Réseau
- **`api.py`** — client TCP asynchrone (`asyncio.open_connection`) :
  - Connexion au port `7621` sans dépendance HTTP
  - Reconnexion automatique avec **backoff exponentiel** (1s → 2s → 4s → max 60s)
  - Lock asyncio pour sérialiser les commandes simultanées
  - Gestion propre des exceptions : `BlazeApiError`, `BlazeConnectionError`

#### Configuration
- **`config_flow.py`** — Config Flow + Options Flow :
  - Étape manuelle avec validation de connexion en temps réel
  - Découverte automatique via Zeroconf (`_pasconnect._tcp.local.`)
  - Déduplication des appareils déjà configurés
  - Options Flow pour modifier l'IP/port sans réinstaller

#### Entités
- **`sensor.py`** — sensors système :
  - État système (`INIT` / `STANDBY` / `ON` / `FAULT`)
  - État des signaux entrée/sortie avec détection de clipping
  - Adresses IP LAN et WiFi courantes
  - Version firmware et date

#### Traductions
- Interface disponible en **français** (`translations/fr.json`) et **anglais** (`translations/en.json`).

### ✅ Bénéfices
- Découverte automatique : aucune configuration manuelle de l'IP si l'amplificateur est sur le même réseau.
- Architecture robuste en vue des évolutions futures (nouvelles plateformes, multi-zone, EQ complet).
- Compatible HACS dès la première version.

---

[1.0.6]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.5...v1.0.6
[1.0.5]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.4...v1.0.5
[1.0.4]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.3...v1.0.4
[1.0.3]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.2...v1.0.3
[1.0.2]: https://github.com/Micpi/blaze-powerzone/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/Micpi/blaze-powerzone/releases/tag/v1.0.1

## v1.0.7 - 2026-05-13

- feat(integration): publish blaze-powerzone
- changed: custom_components/blaze_powerzone/manifest.json
- changed: hacs.json

## v1.0.8 - 2026-05-13

- feat(integration): publish blaze-powerzone
- changed: custom_components/blaze_powerzone/manifest.json
- changed: hacs.json

## v1.0.9 - 2026-05-13

- feat(integration): publish blaze-powerzone
- changed: custom_components/blaze_powerzone/manifest.json
- changed: hacs.json

## v1.0.10 - 2026-05-13

- feat(integration): publish blaze-powerzone
- changed: custom_components/blaze_powerzone/manifest.json
- changed: hacs.json

## v1.0.11 - 2026-05-13

- feat(integration): publish blaze-powerzone
- changed: custom_components/blaze_powerzone/manifest.json
- changed: hacs.json

## v1.0.12 - 2026-05-13

- feat(integration): publish blaze-powerzone
- changed: custom_components/blaze_powerzone/manifest.json
- changed: hacs.json


