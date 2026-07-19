# Xiaomi Home pour Jeedom — Fork corrigé (Debian 12 / Python 3.11)

Fork personnel du plugin **Xiaomi Home** pour Jeedom (auteurs originaux : lunarok, sarakha63 — licence AGPL), avec le correctif de compatibilité Python 3.10+ appliqué.

## Pourquoi ce fork ?

Le plugin officiel (version `2022-06-06`) plante sous Python 3.10+ (Debian 12 / Jeedom avec venv Python 3.11) à cause de la suppression de `collections.Mapping` (déplacé vers `collections.abc.Mapping`). Le daemon `xiaomihomed` ne démarre pas ou boucle en erreur.

Voir la discussion communautaire : [Rendre le plugin XiaomiHome fonctionnel sous Debian 12](https://community.jeedom.com/t/rendre-le-plugin-xiaomihome-fonctionnel-sous-debian-12/130576)

## Correctif appliqué

Fichier `resources/xiaomihomed/jeedom/jeedom.py` :

| Ligne | Avant | Après |
|---|---|---|
| 23 | `import collections` | `from collections.abc import Mapping` |
| 130-131 | `collections.Mapping` | `Mapping` |

Les correctifs PHP (`xiaomihome.class.php` lignes 194/195, utilisation de `system::getCmdPython3`) et JS (`xiaomihome.js` ligne 424, `builSelectCmd` → `buildSelectCmd`) étaient déjà présents dans la version source utilisée pour ce fork.

## Installation

Sur Jeedom, activer "Alternative Market" puis :
`Gestion des plugins > Ajouter depuis une autre source` → renseigner l'URL de ce dépôt, branche `master`.

## Rappel important — dépendances Python

Si le daemon reste en erreur après installation, le venv Python du plugin peut être corrompu (paquets `future`, `cryptography`, etc. avec métadonnées invalides). Dans ce cas, supprimer et laisser Jeedom recréer le venv :

```bash
sudo rm -rf /var/www/html/plugins/xiaomihome/resources/python_venv
```

Puis relancer l'installation des dépendances depuis l'interface Jeedom.
