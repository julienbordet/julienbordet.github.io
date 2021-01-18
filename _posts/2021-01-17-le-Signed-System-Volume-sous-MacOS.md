---
layout: post
title: "Le Signed System Volume sous macOS Big Sur : intégrité du File System garantie"
date: 2021-01-19
category: [mac]
tags: [mac, ssv, sécurité]
---

Depuis plusieurs années, Apple a mis une énergie considérable dans le développement d'un macOS plus sécurisé, avec notamment GateKeeper, Kernel Integrity Protection ou encore System Integrity Protection (SIP).

macOS Big Sur amène, du point de vue de la sécurité, un nombre intéressant de nouveautés, comme par exemple la "suppression" des extensions Kernel hors de l'espace noyau ou des améliorations de la notarisaton des applications. Avec l'arrivée des puces M1, l'intégrité du processus de boot a été amélioré, et d'autres améliorations sont déployées progressivement, notamment, l'authentification des pointeurs.

L'objectif de cet article est de présenter l'une des nouveautés de macOS Big Sur : l'introduction du SSV, le système de sécurisation de l'intégrité du volume Système.

<!--more-->

## Etat des lieux

Depuis macOS Mojave (10.14), les systèmes d'exploitation Mac ont fait l'objet d'évolutions significatives concernent le système de fichiers.

Avec **Mojave**, il n'y a pas de différences entre les données du système et les données de l'utilisateur. Tout est mélanger dans un seul volume APFS (Apple File System). Les données sont protégées à l'aide des autorisations UNIX classiques, auxquels macOS rajoute System Integrity Protection (SIP).

Lors de l'introduction de **Catalina**, le volume de boot est séparé en deux : le volume système (*System*) et le volume données (*Data*), au sein d'un même *Volume Group*. L'ensemble continue à bénéficier des protections du SIP, mais le volume *System* est monté en mode lecture seule.

## Signed System Volume : un sceau d'intégrité pour l'ensemble du système de fichiers 

Big Sur continue à s'appuyer sur la séparation du *Volume Group* en *System* et *Data*, mais il rajoute un système de protection supplémentaire : SSV, pour *Signed System Volume*.

L'idée en est simple : pour chaque fichier du volume, un hachage cryptographique (SHA-256) est calculé et stocké dans les métadonnées. A chaque accès au fichier, on calcule à nouveau le hash et on le compare à la valeur stockée, pour s'assurer que le fichier n'a pas été modifié. 

Mais l'utilisation des hachages ne s'arrête pas aux fichiers : dans chaque répertoire, un hachage est calculé pour l'ensemble des métadonnées. Ce hachage de chaque sous-répertoire `A1`, `A2`, `A3` d'un répertoire `A`est alors ajouté aux métadonnées de 'A', qui font elles-mêmes l'objet d'un hachage intégré dans les métadonnées du répertoire parent, et ainsi de suite jusqu'au répertoire racine. Le hachage du répertoire racine est appelé le *sceau* (*seal*). L'intégrité cryptographique peut alors être vérifiée non seulement au niveau de chaque fichier, mais au niveau du volume dans son ensemble, incluant les méta-données et la structure des répertoires.

Ce sceau est vérifié à chaque démarrage du système, par le boot loader (géré par la puce Apple T2 et dont le code est stocké en mémoire NVRAM) juste avant le chargement du noyau, et lors de la mise à jour du système. En cas d'erreur dans la vérification du sceau, l'opération s'arrête et il est demandé à l'utilisateur de réinstaller le système.

## Quelles sont les données concernées par le SSV ?

D'après le document *Apple File System Reference*, seul le volume *System* peut être protégé à l'aide du SSV. Il ne semble donc pas possible de protéger aussi les données utilisateurs.

Howard Noakley ([@howardnoakley](https://twitter.com/howardnoakley)) a fait une excellente synthèse des données qui sont stockées sur le volume *System* et dans le volume *Data* dans Big Sur :

![Illustration](https://eclecticlightdotcom.files.wordpress.com/2021/01/bigsurintsimple.jpg?w=940)

*Copyright @Howard Noakley (2021)*

Ce qui est particulièrement intéressant à noter, c'est qu'avec APFS il est possible de monter *plusieurs systèmes de fichiers* derrière un **même répertoire** : ainsi, le répertoire `/Library` contient à la fois des bibliothèques Apple issues du volume *System* **et** les bibliothèques ajoutées par l'utilisateur et stockées dans le volume *Data*.

## SSV et Snapshots

Depuis l'introduction d'APFS (Apple File System), il est possible de faire des "snapshots" du système de fichier : il s'agit d'un "instantané" de l'état du système de fichier à un moment donné. Ceci est souvent particulièrement utile pour les sauvegardes notamment, ou pour prévenir les problèmes lors des mises à jour système : il sera toujours possible de revenir à l'état *stable* précédent.

Pour réaliser la sceau du volume Système, macOS Big Sur ne travaille pas au niveau du Volume directement : il commence d'abord par réaliser un snapshot du Volume, et c'est lui qu'il signe. Ceci peut être vu à l'aide de la comme `diskutil apfs list`.



## FAQ

### 1/ Est-il possible de retirer le sceau du volume *System* ?

Oui, c'est possible, en redémarrant en mode *Recovery*, et en utilisant la commande :

```
$ csrutil authenticated-root disable
```

Il est ensuite possible de monter le volume *System* en mode lecture-écriture et de faire les modifications souhaitées. Pour rendre le volume à nouveau bootable, il est nécessaire de créer un nouveau snapshot à l'aide de la commande :

```
$ sudo bless --folder /[mountpath]/System/Library/CoreServices --bootefi --create-snapshot
```

Néanmoins, le volume *System* n'est alors plus éligible à la vérification d'intégrité SSV : il n'est pas possible de revenir en arrière.

### 2/ Est-il possible d'utiliser le SSV sur d'autres volumes que *System* ?

Il semblerait que non. Le document **Apple File System Reference** indique que pour qu'un volume soit doté d'un sceau, il faut que le rôle de ce dernier soit `APFS_VOL_ROLE_SYSTEM`, associé au répertoire racine du système. C'est d'ailleurs indiqué dans le nom : *Signed <u>System</u> Volume*.

### 3/ Comment vérifier si mon système utilise le SSV ?

Le plus simple est d'utiliser la commande `crsutil`, celle qui permettait de désactiver le SSV, comme indiqué plus haut :

```
$ csrutil authenticated-root status
Authenticated Root status: enabled
```

A l'heure actuelle, les commandes fournit par Apple ne sont pas cohérentes. La commande `diskutil apfs list` montre bien l'état du snapshot  :

```
$ diskutil list
/dev/disk0 (internal):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                         500.3 GB   disk0
   1:             Apple_APFS_ISC ⁨⁩                        524.3 MB   disk0s1
   2:                 Apple_APFS ⁨Container disk3⁩         494.4 GB   disk0s2
   3:        Apple_APFS_Recovery ⁨⁩                        5.4 GB     disk0s3

/dev/disk3 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +494.4 GB   disk3
                                 Physical Store disk0s2
   1:                APFS Volume ⁨Macintosh HD⁩            15.1 GB    disk3s1
   2:              APFS Snapshot ⁨com.apple.os.update-...⁩ 15.1 GB    disk3s1s1
   3:                APFS Volume ⁨Preboot⁩                 305.0 MB   disk3s2
   4:                APFS Volume ⁨Recovery⁩                1.1 GB     disk3s3
   5:                APFS Volume ⁨Data⁩                    94.4 GB    disk3s5
   6:                APFS Volume ⁨VM⁩                      3.2 GB     disk3s6

$ diskutil apfs list
APFS Containers (3 found)
|
+-- Container disk3 BB9AF583-95CA-4B9F-88A6-DEFDB1A5543B
    ====================================================
    APFS Container Reference:     disk3
    Size (Capacity Ceiling):      494384795648 B (494.4 GB)
    Capacity In Use By Volumes:   110459219968 B (110.5 GB) (22.3% used)
    Capacity Not Allocated:       383925575680 B (383.9 GB) (77.7% free)
    |
    +-< Physical Store disk0s2 7895F813-BDC9-4264-98D3-31E5BFCE4ADC
    |   -----------------------------------------------------------
    |   APFS Physical Store Disk:   disk0s2
    |   Size:                       494384795648 B (494.4 GB)
    |
    +-> Volume disk3s1 0BCFB63E-8457-4D20-8908-0B71AD347DEB
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s1 (System)
    |   Name:                      Macintosh HD (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         15053811712 B (15.1 GB)
    |   Sealed:                    Broken
    |   FileVault:                 Yes (Unlocked)
    |   Encrypted:                 No
    |   |
    |   Snapshot:                  C2BAF138-4B0A-48EB-A65E-7996454D08A5
    |   Snapshot Disk:             disk3s1s1
    |   Snapshot Mount Point:      /
    |   Snapshot Sealed:           Yes
    
[...]
```

La volume Système est  `disk3s1` , et le snapshot associé est `disk3s1s1`. Ce dernier est indiqué comme scellé (`Snapshot Sealed: Yes`).

Néanmoins, le résultat de la commande `diskutil info`n'est, lui, pas cohérent

```
$ diskutil info  disk1s1
[...]

   This disk is an APFS Volume Snapshot.  APFS Information:
   APFS Snapshot Name:        com.apple.os.update-5523D8E63431315F9F949CCDD0274BF797F5CEE4EAF616D4C66A01B8D6A83C7B
   APFS Snapshot UUID:        C2BAF138-4B0A-48EB-A65E-7996454D08A5
   APFS Container:            disk3
   APFS Physical Store:       disk0s2
   Fusion Drive:              No
   APFS Volume Group:         18CB59E0-D1AD-4056-A756-8AAB902F2938
   EFI Driver In macOS:       1677060023000000
   Encrypted:                 No
   FileVault:                 Yes
   Sealed:                    Broken
   Locked:                    No
```

Le SSV est indiqué comme `Broken`, ce qui n'est évidemment pas le cas... Il s'agit d'un bug de `diskutil`.

## Sources

- L'[article](https://developer.apple.com/news/?id=3xpv8r2m) sur le site d'Apple.
- The Eclectic Light Company - [Big Sur’s Signed System Volume: added security protection](https://eclecticlight.co/2020/06/25/big-surs-signed-system-volume-added-security-protection/)
- [Apple File System Reference](https://developer.apple.com/support/downloads/Apple-File-System-Reference.pdf)
