# P12 — Correction Ethernet intégrée / Integrated Ethernet correction

## Français — 3 novembre 2026

Cette P12 publique reprend exactement la P11 comme base. Elle intègre les correctifs Ethernet auparavant appliqués au serveur de test, pour les charger dès le démarrage, avant le DHCP. Elle reste une image **Alpine Linux ARMv7** pour la **NAND du Radxa ROCK Pro RK3188**, avec le noyau **6.18.59-radxa1 #8**. Ce n'est ni Debian ni une image pour les modèles ROCK Pi.

### 1. Récupération RX/TX et synchronisation du pilote

Le correctif RX/TX conserve une seule gestion de la lecture du compteur matériel RX à remise à zéro et de la détection de blocage dans NAPI. La récupération des anneaux se termine avant de réactiver les interruptions. Le chemin d'émission, sa complétion et sa réinitialisation partagent un verrou. Cela évite les courses constatées pendant le diagnostic. Le module d'origine reste dans le noyau ; les modules corrigés sont ajoutés séparément et le contrôleur est rattaché au démarrage.

Module de récupération : `radxa_emac_recovery.ko`, SHA256 `60084fe6bc9997e9c55be2a34a27e5be86c7a754f7c778d4c88305a0a266813a`.

### 2. Reprise bornée des trames abandonnées après sous-alimentation FIFO TX

Le module `radxa_emac_fifo.ko` reprend les trames explicitement signalées comme perdues par le contrôleur (UFLO). Les reprises sont bornées : limite 32, délai 200 microsecondes, émission sérialisée et tampon DMA de copie. Les compteurs distinguent les erreurs matérielles, les reprises réussies et leur éventuel épuisement. **Les erreurs matérielles ne sont pas masquées et les compteurs ne sont pas remis à zéro pour annoncer un résultat.**

Module FIFO : SHA256 `d6a1c371880f1fe3570b7e3118d186c25a01ee4a540369d28cf3fcc4f059c061`.

### 3. Correction de la priorité d'accès au bus mémoire

La récupération ne suffisait pas à expliquer les sous-alimentations FIFO. Les essais comparatifs sur le vrai Radxa ont identifié une influence du réglage de priorité PERI du RK3188. La P12 applique **3/3**, encodé **15**, au registre de priorité `0x1012c008`, au lieu de 1/0 (valeur 4). Les autres paramètres restent identiques : mode 3, bande 32, fenêtre 64. Aucune modification de PLL, de PHY, de MTU ou des priorités LCDC/CPU/GPU.

L'encodage vient du BSP constructeur. Sa valeur PERI 2/2 (10) n'avait pas suffi dans nos essais ; **3/3 est la valeur qualifiée par nos mesures**, pas une prétention que le constructeur la configure par défaut. L'écriture est un accès C `volatile uint32_t` aligné, protégé par des contrôles de modèle RK3188, de version du noyau, de permissions, d'empreinte du binaire et d'état attendu du registre.

Exemple de comparaison réelle, 8 Mio TCP vérifiés à chaque phase :

| Priorité PERI | 4 (1/0) | 15 (3/3) | 15 (3/3) | 4 (1/0) |
|---|---:|---:|---:|---:|
| Augmentation des erreurs FIFO TX | 173 | 0 | 0 | 73 |

Sous charge mémoire, les phases 15/4/15 ont donné 0/159/0 erreurs FIFO sur TCP, puis 0/3/0 sur UDP. Après installation persistante et redémarrage réel, le récepteur Windows natif a vérifié 8 Mio dans chaque sens et reçu 10 000/10 000 datagrammes de 1 400 octets, sans corruption ni doublon. FIFO/RX/TX et reprises étaient à zéro dans ces fenêtres. Deux essais avec le récepteur WSL ont perdu des datagrammes malgré des compteurs Ethernet à zéro ; la cause de cette différence de chemin n'est pas établie et ces essais ne sont pas présentés comme des réussites.

Ces résultats établissent une influence du réglage d'accès mémoire dans les conditions testées. Ils ne constituent ni une mesure interne des cycles AHB ni une garantie d'absence future d'erreur.

### 4. MAC persistante et dépendances disponibles dès le démarrage

Au premier démarrage de l'image, la MAC unicast propre à la carte est enregistrée dans `/etc/radxa/ethernet-mac`, puis conservée aux redémarrages. Aucune MAC ou configuration IP privée du serveur de développement n'est copiée dans l'image. Un nouveau flash remet cette identité enregistrée à zéro.

Python 3 et ses dépendances signées Alpine sont inclus pour que les helpers fonctionnent **avant l'installation d'outils supplémentaires**. Ordre : console d'origine, récupération RX/TX et identité si absente, FIFO, priorité mémoire, application MAC, puis DHCP/services de P11. La racine NAND conserve son extension automatique ; les plafonds de `/dev`, `/run` et `/tmp` restent en RAM.

### 5. Conservation des fonctionnalités de P11

Le noyau, le DTB, le chargeur, la géométrie NAND et le code d'amorçage NAND sont identiques à P11. La longueur de l'initramfs et les sommes de contrôle du conteneur sont recalculées. Les empreintes de la configuration HDMI 720p60, de son menu 1024×768 / 1280×720 / 1600×900 / 1920×1080 à 60 Hz, de la console, du clavier USB, du Wi-Fi, du Bluetooth et des comptes sont conservées. Cela prouve la conservation de ces fichiers, pas une nouvelle observation visuelle HDMI.

### Validation de la nouvelle image et limites

- Ext4 neuf, contrôle e2fsck et relecture des contenus/modes/propriétaires.
- Correspondance de 1 414 entrées ajoutées/modifiées entre initramfs et racine NAND.
- Emballage/extraction exacts avec les outils natifs de RKDevTool 2.69, capacités des partitions, MD5 RKFW et SHA1 Rockchip contrôlés.
- QEMU ARMv7 : connexion SSH avec le compte public, sudo, refus root et mauvais mot de passe, runtime Python, menu des résolutions, chargement/retrait des deux modules avec ce noyau.
- Identité MAC : création unique, conservation et rejet des adresses multicast testés sur des fichiers isolés.

**L'assemblage complet P12 n'a pas encore été démarré sur la carte physique.** Les mesures réelles ci-dessus qualifient les correctifs sur le serveur en service. QEMU n'émule pas le RK3188, son HDMI, sa NAND, son PHY Ethernet, son Wi-Fi ou son contrôleur USB. Le test après flashage de cette P12 reste nécessaire. La release est donc marquée *Pre-release*.

## English

P12 is based on the exact public P11 image and embeds the previously qualified Ethernet corrections before DHCP: RX/TX recovery and ring synchronization, bounded FIFO retries (32 attempts, 200 µs, serialized TX and DMA bounce storage), RK3188 PERI memory-bus priority 3/3, and a per-board MAC saved on first boot. Python 3 is included so the helpers are available immediately.

The priority change only writes the documented PERI priority field (15 instead of 4). Mode/bandwidth/saturation stay 3/32/64; PLL, PHY, MTU, LCDC/CPU/GPU priorities and HDMI configuration are unchanged. A physical 8 MiB-per-phase TCP comparison at priorities 4/15/15/4 produced 173/0/0/73 TX FIFO errors. After a real reboot of the test server, native Windows received all 10,000 UDP datagrams and verified 8 MiB each way with zero FIFO/RX/TX errors or retries in those windows. Separate WSL receiver tests lost datagrams and remain recorded as failures; their exact cause is unknown. Hardware counters are neither hidden nor reset.

The P11 kernel/DTB, NAND loader and partition layout, HDMI defaults and menu, USB/Wi-Fi/Bluetooth scripts and account data are preserved. Software validation covers fresh ext4, RAM/NAND content consistency, native RKDevTool 2.69 pack/unpack and checksums, plus ARM QEMU SSH/sudo and module load/unload. **This complete P12 image has not yet been booted on the physical board.** Prior on-board correction tests and QEMU checks do not prove a physical P12 boot or every peripheral. No future error-free guarantee is made.

The attached Ethernet source bundle contains the C driver sources/headers, helper code and build notes for these corrections. It is not a full Linux/Alpine source distribution. P11 remains available as an earlier release. No personal image or private configuration is part of this release.
