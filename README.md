# Radxa Rock Pro P11

**Version expérimentale : contrôles d’image et connexion SSH sous QEMU validés ; démarrage physique de la P11 testée et fonctionnelle.**

## Téléchargements

- [Image P11 compressée ZIP — environ 54 Mo](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/download/p11/alpine-rock-pro-nand-p11-linux618-ssh-public-rk269.img.zip)
- [RKDevTool 2.69 pour Windows](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/download/p11/RKDevTool-2.69.zip)
- [Notice P11 à télécharger](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/download/p11/README-P11.md)
- [Page de la version P11](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/tag/p11)

**Extraire le ZIP de la P11 avant le flashage.** Il contient l’IMG originale de 302 154 200 octets, inchangée. Son SHA-256 après extraction est indiqué ci-dessous. L’archive ZIP mesure 54 243 721 octets ; son SHA-256 est `a913659ad93dfdbd0edc9ca6083d29e0de3b221000ae434587f78045e97afeaf`.

Image serveur **Alpine Linux ARMv7** avec noyau **Linux 6.18.59-radxa1**, destinée à la **NAND interne du Radxa Rock Pro RK3188**. Cette version communautaire dérive de la P10 et fournit des identifiants initiaux publics pour permettre une première connexion SSH.

Projet de réemploi réalisé par [Mehdi El-Filahi](https://github.com/mehdielfi), avec l’aide de Codex d’OpenAI et de ChatGPT. Le modèle Rock Pro date de **mi-2014**. Il possède un processeur Cortex-A9 à quatre cœurs, 2 Go de RAM et 8 Go de NAND.

## Fichiers

| Fichier | Utilisation |
| --- | --- |
| `alpine-rock-pro-nand-p11-linux618-ssh-public-rk269.img.zip` | Archive à extraire pour obtenir le firmware IMG de la NAND |
| `alpine-rock-pro-nand-p11-linux618-ssh-public-rk269.img` | Firmware extrait, à charger avec RKDevTool 2.69 |
| `RKDevTool-2.69.zip` | Programme Windows de chargement, avec ses dépendances et les manuels d’origine |
| `README-P11.md` | Installation, connexion et état de validation de P11 |

L’IMG mesure **302 154 200 octets**, soit environ **302 Mo**. Elle contient le démarrage et un système de fichiers initial compact. Sa taille n’est pas la capacité finale de la partition racine.

SHA-256 de l’IMG :

```text
753d6aaf3f8dc223a7e2be753dcae152466f7f8ecf2d4d69cbad414f8bd36bc9
```

Sous Windows, contrôler le téléchargement depuis son dossier :

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath '.\alpine-rock-pro-nand-p11-linux618-ssh-public-rk269.img'
```

## Identifiants initiaux

| Paramètre | Valeur |
| --- | --- |
| Utilisateur SSH | `radaxrock2026` |
| Mot de passe initial | `radaxrock2026` |
| Port SSH | `22` |
| Shell administrateur | `sudo -i` |
| Connexion SSH de root | Interdite |

**Ce mot de passe est public. Le changer avec `passwd` avant toute exposition du SSH sur Internet.** Le compte principal dispose de `sudo` sans mot de passe supplémentaire.

La P11 ne contient plus la clé publique personnelle de maintenance de la P10, ni son ancien mot de passe personnel. Les clés d’hôte SSH sont générées au démarrage, elles ne sont pas partagées dans l’image.

## Matériel et préparation

- Un **Radxa Rock Pro RK3188**, avec NAND interne. Ce fichier cible ce modèle et ce support de stockage.
- Un PC Windows équipé du pilote **Rockusb**. Le pilote Windows est distinct des pilotes Linux présents dans l’image et n’est pas inclus dans le ZIP du programme.
- Un câble **USB avec données** pour relier le port micro-USB OTG au PC pendant le chargement.
- L’alimentation prévue pour la carte, **5 V / 2 A sur la prise ronde**, pour le démarrage et les essais normaux.
- Pour la première configuration : un écran HDMI et un clavier USB, ou un câble RJ45 vers le réseau local pour accéder en SSH.

## Charger la P11 dans la NAND

**Le chargement de ce firmware remplace le contenu de la NAND, y compris le système et les données précédents.**

1. Extraire **tout** le ZIP `RKDevTool-2.69.zip` dans un dossier local Windows. Ouvrir `RKDevTool-2.69\AndroidTool.exe`. Le programme peut afficher le nom **RKDevTool v2.69**, bien que son exécutable s’appelle `AndroidTool.exe`.
2. Passer le Radxa en mode **LOADER**. Carte éteinte : maintenir **RECOVERY**, puis brancher l’USB OTG au PC. Carte déjà alimentée : brancher l’OTG, maintenir **RECOVERY**, appuyer puis relâcher **RESET**, et garder RECOVERY appuyé jusqu’à la détection.
3. Vérifier l’indication **Found One LOADER Device** dans le programme, puis relâcher RECOVERY. Si la carte n’est pas détectée, contrôler le mode LOADER, le câble de données et le pilote Rockusb dans le Gestionnaire de périphériques.
4. Ouvrir l’onglet **Upgrade Firmware**, cliquer sur **Firmware** et sélectionner `alpine-rock-pro-nand-p11-linux618-ssh-public-rk269.img`.
5. Cliquer sur **Upgrade** et attendre la réussite complète du chargement. Le firmware contient déjà son chargeur : aucun fichier Boot/Loader supplémentaire n’est à sélectionner pour cette procédure.
6. Pour le démarrage normal, relâcher RECOVERY, débrancher la liaison OTG et alimenter la carte par sa prise ronde. Brancher l’HDMI et, si disponible, le RJ45.

Cette procédure emploie l’image complète dans **Upgrade Firmware**. Le tableau **Download Image** présente des exemples de partitions qui ne constituent pas la configuration P11 à renseigner manuellement.

Références du fabricant : [mode LOADER](https://wiki.radxa.com/Rock/Loader_mode) et [chargement du firmware](https://wiki.radxa.com/Rock/flash_the_image).

## Première connexion

Le réseau de base conserve le fonctionnement DHCP de la P10. Relever l’adresse attribuée à la carte dans le routeur, puis remplacer `ADRESSE_IP_DU_RADXA` dans cette commande :

```sh
ssh radaxrock2026@ADRESSE_IP_DU_RADXA
```

Saisir le mot de passe initial `radaxrock2026`, puis le changer :

```sh
passwd
```

Le pare-feu initial hérité de P10 autorise SSH depuis **192.168.1.0/24**, **192.168.137.0/24** et les adresses IPv6 de lien local. **Sur un autre sous-réseau, adapter d’abord ces règles depuis la console locale** ; posséder le bon mot de passe ne suffit pas à traverser le filtrage.

La bannière logicielle P10 a été conservée avec sa configuration. Identifier la variante P11 avec :

```sh
cat /etc/radxa/p11
```

Résultat attendu : `P11_PUBLIC_20261102`.

## Affichage HDMI et résolutions

Le portage de l’affichage **RK616 vers DRM/KMS** et le rétablissement de la console HDMI en **720p à 60 Hz**, avec le choix d’autres résolutions, toujours à **60 Hz**, sont conservés de la P10.

Ouvrir le menu :

```sh
sudo radxa-affichage
```

| Choix | Résolution | Fréquence |
| --- | --- | --- |
| 1 | 1024 × 768 | 60 Hz |
| 2 | 1280 × 720 — défaut | 60 Hz |
| 3 | 1600 × 900 | 60 Hz |
| 4 | 1920 × 1080 | 60 Hz |
| 0 | Annuler | Aucun changement |

Le mode choisi s’applique au prochain redémarrage. La présence d’un mode dans le menu ne garantit pas son fonctionnement sur tous les écrans. Cette image fournit une console serveur ; l’accélération graphique 3D n’a pas été validée.

## Stockage et mémoire

- `/` doit utiliser la NAND, avec une racine d’environ **6,7 Go observée sur la carte de référence** après initialisation et extension.
- `/dev`, `/run` et `/tmp` sont en **RAM**. Leur plafond de 1 Gio n’est pas une réservation de 1 Gio chacun et ne réduit pas la capacité de la NAND.
- Un **swap de 512 Mio** est prévu par le mécanisme de démarrage de P10 conservé dans P11.

Contrôler l’état réel après démarrage :

```sh
df -h /
df -h /dev /run /tmp
free -m
cat /proc/swaps
```

Une racine montée sur `/dev/rknand0p3` utilise la NAND. Si le système a démarré dans le secours en RAM, ses changements ne sont pas persistants : contrôler le montage avant d’installer des logiciels.

## Logiciels et scripts séparés

Le gestionnaire de paquets est **APK**, celui d’Alpine Linux. Exemple de rafraîchissement de l’index :

```sh
sudo apk update
```

La base P11 ne préinstalle pas Python, Node.js, PostgreSQL ou Nakama. Deux installateurs sont fournis séparément dans le projet :

- `Installer-Outils-Securite.sh` : outils, environnements de développement, sécurité et correctifs persistants prévus par ce script.
- `Installer-Nakama.sh` : PostgreSQL, instances Nakama et configuration des services associés, dont TLS.

Après avoir placé un script dans le dossier courant du Radxa :

```sh
sudo sh ./Installer-Outils-Securite.sh
```

Le script est interactif. Son installation complète après une remise à zéro en P11 reste à contrôler ; les tests du mot de passe P11 ne constituent pas un test complet de ces installateurs.

## Modifications et limites de cette version

La P11 est une **variante de la base P10 avec identifiants publics**, et non une copie du serveur déjà configuré :

- Mot de passe de `radaxrock2026` remplacé dans la racine NAND **et** dans le système de secours en RAM.
- Ancienne clé publique personnelle de maintenance et sauvegarde `shadow-` retirées.
- Système de fichiers reconstruit sur une image neuve afin de ne pas conserver l’ancien hash dans les blocs libres.
- Noyau, DTB, pilotes, réglages HDMI, USB, réseau, swap, chargeur et géométrie des partitions de P10 conservés.

**Les deux correctifs récents RX/TX Ethernet et intervalle USB/Wi-Fi du serveur ne sont pas intégrés dans cette IMG.** Ils restent fournis dans `Installer-Outils-Securite.sh`, comme pour la P10. Les erreurs FIFO Ethernet restent à analyser. Les essais microSD et d’usage Bluetooth ne sont pas complets.

Validations effectuées sur l’artefact P11 :

- Contrôle ext4 et comparaison des fichiers, droits et propriétaires.
- Emballage et réextraction avec les outils natifs de RKDevTool 2.69 ; contrôles du conteneur Rockchip, de ses partitions et de l’en-tête de démarrage.
- Connexion SSH avec le mot de passe public, `sudo`, refus de root en SSH et refus d’un mauvais mot de passe dans **QEMU ARMv7**.

**La P11 elle-même n’a pas encore été flashée ni démarrée sur la carte physique.** QEMU teste ici les logiciels ARM et l’authentification ; il n’émule pas les périphériques RK3188 du Radxa.

## English quick start

P11 is a community **Alpine Linux ARMv7** server image for the **Radxa Rock Pro RK3188 internal NAND**. It retains P10’s kernel and hardware configuration while replacing the personal account password with public initial credentials.

First extract the P11 `.img.zip` archive to obtain the unchanged `.img` firmware. The image SHA-256 below refers to the extracted IMG.

1. Extract all of `RKDevTool-2.69.zip` and launch `RKDevTool-2.69\AndroidTool.exe` on Windows with the Rockusb driver installed.
2. Connect a USB **data** cable to the board’s OTG port and enter LOADER mode using RECOVERY, with RESET if the board is already powered.
3. In **Upgrade Firmware**, select the complete P11 IMG with **Firmware**, then click **Upgrade**. This replaces the existing NAND contents.
4. Boot normally with the **5 V / 2 A DC barrel supply**, RECOVERY released and the programming OTG cable disconnected. Obtain the board’s DHCP address from your router.
5. SSH user: **`radaxrock2026`**. Initial password: **`radaxrock2026`**. Port: **22**. Change the public password with `passwd`. Use `sudo -i` for root privileges; direct root SSH is disabled.

The initial firewall permits SSH from **192.168.1.0/24**, **192.168.137.0/24** and IPv6 link-local addresses. Other subnets require a local-console firewall adjustment.

Run `sudo radxa-affichage` to select **1024×768**, **1280×720** (default), **1600×900** or **1920×1080**, all at **60 Hz**, applied after reboot. Display compatibility depends on the monitor.

NAND storage, a 512 MiB swap mechanism and 1 GiB RAM filesystem limits are inherited from P10. Python, Node.js, PostgreSQL and Nakama are installed through separate scripts. Recent server-only Ethernet and USB/Wi-Fi fixes remain in script 1, rather than being embedded in this IMG.

Native Rockchip packaging and ARM QEMU password/sudo checks passed. **A physical boot test of P11 remains outstanding**, as do complete microSD/Bluetooth usage checks and Ethernet FIFO investigation. The retained P10 banner can be distinguished from the image variant using `cat /etc/radxa/p11`.

## Références et attribution

- [Compte du projet](https://github.com/mehdielfi)
- [Spécifications Radxa Rock Pro](https://wiki.radxa.com/Rock/specification)
- [Mode LOADER](https://wiki.radxa.com/Rock/Loader_mode)
- [Chargement d’une image Rockchip](https://wiki.radxa.com/Rock/flash_the_image)

RKDevTool/AndroidTool et ses manuels sont des logiciels et documents de Rockchip. Le ZIP préparé conserve les exécutables et les PDF d’origine ; seuls le choix de langue et l’ajout d’une notice diffèrent, et les anciens journaux/dumps sont exclus. Cette préparation ne constitue pas une concession de droits sur ces éléments. Les composants conservent leurs licences respectives. Ce dépôt publie à ce stade l’image P11, sa documentation et le chargeur Windows ; les sources complètes du portage ne sont pas encore publiées dans ce dépôt.
