# Xiaomi Home pour Jeedom — fork corrigé

Fork communautaire du plugin **Xiaomi Home** (`xiaomihome`) pour Jeedom : gateway Aqara en mode LAN, ampoules Yeelight, appareils Xiaomi Wifi.

Il part de la dernière version publiée sur le Market (`2022-06-06`), qui n'évolue plus, et y ajoute les corrections nécessaires pour qu'elle fonctionne sans retouche manuelle sous Debian 11 et Debian 12 :

- la **fuite mémoire du démon** est corrigée ;
- les **dépendances s'installent** sous Debian 12 (plus de statut « NOK ») ;
- le démon démarre sous **Python 3.10 et suivants**.

## Origine et remerciements

- Auteurs du plugin : **lunarok** et **sarakha63** (licence AGPL).
- Ce dépôt est un fork de [titof2375/jeedom-xiaomihome-fixed](https://github.com/titof2375/jeedom-xiaomihome-fixed), qui a publié la version `2022-06-06` avec les correctifs Debian 12 du forum déjà appliqués (`collections.abc.Mapping`, `system::getCmdPython3`, `buildSelectCmd`).
- Ces correctifs viennent du sujet [Rendre le plugin XiaomiHome fonctionnel sous Debian 12](https://community.jeedom.com/t/rendre-le-plugin-xiaomihome-fonctionnel-sous-debian-12/130576) (fishton, Eridani78, akenad, Mips).
- Le fichier `jeedom.py` est celui du [plugin-template Jeedom](https://github.com/jeedom/plugin-template/blob/master/resources/demond/jeedom/jeedom.py), corrigé par Mips.

Ce fork n'est pas le plugin officiel et n'est pas distribué par le Market.

## Ce que ce fork corrige

| Problème constaté | Cause | Correction |
|---|---|---|
| La mémoire et le swap du démon grossissent sans fin, il faut le relancer régulièrement | `jeedom.py` créait un nouveau thread (`threading.Timer`) à chaque cycle, soit 20 par seconde. Sous Python < 3.9.6 (Debian 11), chaque thread laisse environ 144 octets qui ne sont jamais libérés ([bpo-37788](https://bugs.python.org/issue37788)) | `jeedom.py` remplacé par la version du plugin-template, qui utilise un seul thread permanent |
| Dépendances « NOK » sous Debian 12, et message « Attention je pense qu'il y a un soucis avec le démon que j'ai relancé plus de 3 fois consécutivement » toutes les 5 minutes | `packages.json` demandait `pycrypto`, qui ne se compile plus sous Python 3.11. Tant que les dépendances sont NOK, Jeedom relance leur installation et le démon | `packages.json` réduit aux deux paquets réellement utilisés : `requests` et `cryptography` |
| `No module named 'future'`, `No module named 'serial'`, `No module named 'pyudev'` | Modules importés par le démon mais pas toujours installés | Le code n'utilise plus `future` ; `serial` et `pyudev` sont devenus optionnels (le plugin ne s'en sert pas) |
| `error: externally-managed-environment` en lançant `install_apt.sh` | Ces scripts installaient avec le `pip` système, ce que Debian 12 refuse | Scripts supprimés : Jeedom installe les dépendances dans son environnement virtuel à partir de `packages.json` |
| `module 'collections' has no attribute 'Mapping'` | Python 3.10+ | Déjà corrigé dans le fork de titof2375, conservé |
| Le niveau de log « Aucun » remplit le log en mode debug | `none` était traduit en « tout afficher » | Corrigé par le `jeedom.py` du plugin-template |
| `SOCKET-READ------Exception on socket : '04cf8c…'` | Une commande est envoyée alors qu'aucun token n'a été reçu de la gateway | Le message d'erreur dit maintenant de quoi il s'agit (voir Dépannage) |

### Détail de la fuite mémoire

Avec le réglage par défaut (`--cycle 0.05`), l'ancien `jeedom.py` créait un thread toutes les 50 ms, même sans aucune donnée à envoyer. Mesures faites sur une installation Debian 11 (Python 3.9.2) :

- avant correction : environ 6,4 Go occupés (RAM et swap) après 27 jours sans redémarrage du démon ;
- après correction : mémoire stable à 33 Mo.

Les versions récentes de Python (3.9.6 et suivantes, donc Debian 12) ne sont pas touchées par ce bug précis, mais créer 20 threads par seconde reste inutile.

## Installation depuis GitHub

Le fork garde l'identifiant `xiaomihome`. Il **remplace** donc la version du Market : les équipements, commandes, historiques et scénarios existants sont conservés. Faites tout de même une sauvegarde Jeedom avant de commencer.

Jeedom 4.2 minimum.

### 1. Activer la source GitHub

**Réglages → Système → Configuration → Mises à jour/Market → onglet Github** : cocher **Activer**, puis sauvegarder.

### 2. Installer le plugin

**Plugins → Gestion des plugins → Plugins → Ajouter** (bouton « + », « Ajouter depuis une autre source »), puis renseigner :

| Champ | Valeur |
|---|---|
| Type de source | `Github` |
| ID logique du plugin | `xiaomihome` |
| Utilisateur ou organisation du dépôt | `joseluu` |
| Nom du dépôt | `jeedom-xiaomihome-fixed` |
| Branche | `master` |

Sauvegarder : Jeedom télécharge le plugin.

### 3. Dépendances et démon

Dans la page de configuration du plugin :

1. activer le plugin s'il ne l'est pas ;
2. cliquer sur **Relancer** dans le bloc Dépendances et attendre le statut **OK** ;
3. démarrer (ou redémarrer) le démon.

### Mises à jour

Le plugin apparaît dans le **Centre de mise à jour** avec la source `github`. Jeedom y propose les nouvelles versions de ce dépôt comme pour un plugin du Market.

### Revenir à la version du Market

Ne supprimez pas le plugin : la suppression efface aussi ses équipements. Réinstallez plutôt la version du Market par-dessus (même identifiant `xiaomihome`), puis relancez les dépendances. Une sauvegarde Jeedom faite avant l'installation du fork reste le retour arrière le plus sûr.

## Si vous aviez appliqué les modifications du forum

Rien à défaire : l'installation remplace tous les fichiers du plugin.

Si les dépendances restent NOK après l'installation, l'environnement virtuel Python d'une tentative précédente est peut-être abîmé. Supprimez-le puis relancez les dépendances depuis Jeedom :

```bash
sudo rm -rf /var/www/html/plugins/xiaomihome/resources/python_venv
```

Les paquets installés à la main avec `apt` (`python3-pyudev`, `python3-serial`, `python3-future`…) ne gênent pas et ne sont plus nécessaires.

## Dépannage

**`aucun token recu de la gateway …` dans le log.** Le démon n'a reçu aucun message de la gateway depuis son démarrage, il ne peut donc pas signer les commandes. Vérifiez que le protocole LAN (mode développeur) est toujours actif sur la gateway, que Jeedom et la gateway sont sur le même réseau, et que le port UDP 9898 et le multicast `224.0.0.50` ne sont pas filtrés. Certaines mises à jour du firmware de la gateway désactivent le mode LAN.

**Le log reste en debug.** Changez le niveau de log dans la configuration du plugin, **sauvegardez**, puis redémarrez le démon : le niveau n'est lu qu'au démarrage.

**Jeedom en Docker.** Le conteneur doit recevoir le multicast de la gateway : réseau `host` ou `macvlan`.

## Tests réalisés

| Environnement | Résultat |
|---|---|
| Jeedom 4.6.2, Debian 12, Python 3.11 (conteneur de test) : installation depuis GitHub, dépendances, démon, création d'équipements à partir de messages de gateway simulés | OK |
| Jeedom 4.6.1, Debian 11, Python 3.9.2 avec une vraie gateway Aqara : `jeedom.py` corrigé | Mémoire du démon stable, remontées des capteurs OK |
| Démon seul sous Python 3.9, 3.11 et 3.13 avec uniquement `requests` et `cryptography` | OK |

Les ampoules Yeelight et les appareils Wifi (aspirateur, purificateur…) n'ont pas été testés sur du matériel réel. Les retours sont les bienvenus dans les [issues](https://github.com/joseluu/jeedom-xiaomihome-fixed/issues).

## Licence

AGPL, comme le plugin d'origine.
