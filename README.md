# Radxa ROCK Pro Linux — P12 Ethernet (2026)

♻️ Une nouvelle vie pour un ordinateur monocarte ARMv7 de 2014 : Alpine Linux et Linux 6.18.59-radxa1 sur sa NAND.

## Téléchargement / Download

**[P12 — fichiers et notes de version](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/tag/p12)** · **[ZIP de l'IMG](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/download/p12/alpine-rock-pro-nand-p12-linux618-ethernet-rk269.img.zip)** · **[IMG brute](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/download/p12/alpine-rock-pro-nand-p12-linux618-ethernet-rk269.img)** · **[RKDevTool 2.69 Windows](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/download/p11/RKDevTool-2.69.zip)**

Extraire le ZIP avant de choisir l'IMG dans **Upgrade Firmware** de RKDevTool 2.69. Le flash remplace le système et les données NAND : sauvegarder les données à conserver. Matériel ciblé uniquement : **Radxa ROCK Pro / RK3188 / ARMv7 / NAND**, pas ROCK Pi.

**P12 est une Pre-release : son assemblage complet reste à tester sur la carte physique.** Les correctifs ont été mesurés sur le serveur réel ; le format et les tests logiciels ARM/QEMU de l'image ont réussi. Ne pas confondre ces périmètres.

## Ce qui change dans P12

- Correction RX/TX et synchronisation des anneaux du contrôleur.
- Reprises FIFO bornées, avec compteurs matériels conservés.
- Priorité d'accès mémoire PERI 3/3 qualifiée par comparaison sur le vrai Radxa.
- MAC propre à chaque carte enregistrée au premier démarrage, puis conservée.
- Python 3 inclus pour activer ces correctifs avant DHCP, sans installation supplémentaire.

Noyau/DTB, chargeur NAND, géométrie, affichage HDMI 720p60 et menu des autres résolutions 60 Hz, console, USB/Wi-Fi/Bluetooth et comptes de P11 conservés par comparaison d'empreintes. La bannière historique peut rester celle de P10 ; `cat /etc/radxa/p12` identifie cette version.

**[Détail technique complet FR/EN, mesures et limites](CHANGELOG-P12.md)** · **[Guide P12](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/download/p12/README-P12.md)** · **[Sources des corrections Ethernet](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/download/p12/Ethernet-P12-Sources.tar.gz)**

## Connexion initiale

Utilisateur et mot de passe publics : `radaxrock2026` / `radaxrock2026`. Root est interdit en SSH ; utiliser `sudo`. **Changer le mot de passe avec `passwd` avant toute exposition Internet.** Ethernet démarre en DHCP ; aucun Wi-Fi privé ou IP fixe personnelle n'est embarqué.

Le pare-feu initial de P11 reste en place : SSH est autorisé depuis `192.168.1.0/24`, `192.168.137.0/24` et les adresses IPv6 de lien local. Sur un autre sous-réseau, adapter ses règles depuis la console locale ; un mot de passe correct ne suffit pas à franchir ce filtrage.

Le gestionnaire de paquets est **APK** : `sudo apk update`. Le mécanisme de swap de 512 Mio et les plafonds de 1 Gio pour `/dev`, `/run` et `/tmp` sont conservés de P11 ; ces derniers sont des systèmes de fichiers en RAM et ne retirent pas d'espace à la NAND. Vérifier l'état réel après démarrage avec `free -m`, `cat /proc/swaps` et `df -h /`.

```sh
sudo radxa-ethernet-qos status
sudo radxa-ethernet-fifo status
cat /etc/radxa/ethernet-mac
df -h /
sudo radxa-affichage
```

## P11 conservée

[Ancienne release P11](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/tag/p11). Elle ne contient pas les correctifs Ethernet intégrés dans P12. Les sources automatiques GitHub représentent le contenu du dépôt, pas une distribution complète des sources Linux/Alpine ; le paquet Ethernet joint couvre les corrections décrites.

## English

**[Download P12](https://github.com/mehdielfi/radxa-rock-pro-linux/releases/tag/p12)**: extract the ZIP and flash the IMG with RKDevTool 2.69. Flashing replaces NAND contents. This image targets only the original **Radxa ROCK Pro RK3188 ARMv7 NAND**.

P12 adds RX/TX recovery, bounded FIFO retries, physically qualified PERI bus priority 3/3, a persistent per-board MAC and Python 3 for early boot helpers. P11 kernel/DTB, loader/layout and HDMI/peripheral/account files are retained. Public SSH user/password: `radaxrock2026` / `radaxrock2026`; change it before Internet exposure. Root SSH is disabled; use `sudo`.

Native Rockchip packaging and ARM QEMU tests passed. **Physical boot of this complete P12 image is still outstanding.** Prior physical driver tests do not prove the new assembled image. Read [CHANGELOG-P12.md](CHANGELOG-P12.md) for measurements, WSL receiver failures, validation scope and limitations. No guarantee of future error-free operation.
