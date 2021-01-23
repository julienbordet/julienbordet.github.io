---
layout: post
title: "macOS vs malware : Fonctionnement de Gatekeeper"
date: 2021-01-23
category: [mac]
tags: [mac, ssv, sécurité]
---

Apple apporte un soin particulier à la sécurisation de son système d'exploitation, désigné d'abord sous le nom de MacOS X, puis macOS 10 et macOS 11. Cette suite d'articles a pour vocation à réaliser un tour d'horizon des solutions que l'éditeur a implémenté, progressivement, pour prévenir la compromission du système par un logiciel malveillant (*malware*).

Pour commencer, une petite présentation de Gatekeeper.

1. Gatekeeper : signature électronique des applications

<!--more-->

## Présentation

Remontant à Mac OS X Lion et introduite pour la première fois en (2011-12), Gatekeeper est, de manière très synthétique, une solution de signature électronique et de contrôle des applications téléchargées et exécutées par l'utilisateur.  Une application non signée est alors considérée comme moins sûre qu'une application non signée, et nécessitera une action supplémentaire de l'utilisateur pour être installée.

Mais qu'apporte la **signature** d'une application pour l'utilisateur ? Deux éléments principaux :

- Tout d'abord, elle identifie l'utilisateur qui a signé l'application, en lui associant *a minima* un nom, une adresse email et un numéro de téléphone (utilisé par Apple pour valider l'identité du détenteur du compte) . Même si l'identification de l'utilisateur n'est pas parfaite (il est toujours possible de créer une fausse adresse email, et d'avoir un numéro de téléphone anonyme), cela permet en principe de faciliter la recherche de l'auteur d'un programme malveillant. Par ailleurs, le certificat associé à la signature peut être révoqué en cas de problème par Apple.
- Ensuite, elle garantit que l'application en elle-même n'a pas été modifiée, et donc que l'élément distribué est bien du  développeur. On se protège ainsi contre une éventuelle compromission du site Web permettant de récupérer le logiciel et sa modification pour y insérer du code malveillant.

Du point de vue du développeur, la signature de l'application est aussi nécessaire pour pouvoir bénéficier de l'API CloudKit (accès à iCloud pour les applications et la synchronisation entre les appareils) et du système de notification Apple Push Notification (pour iOS).

### Le processus de validation

Le principe est alors le suivant :

- Le développeur s'inscrit sur le site [developper.apple.com](https://developer.apple.com/) et doit fournir une adresse email et un numéro de téléphone, qui sont tous les deux validés.
- Il génère un "Developer ID Certificate", le certificat qui va lui permettre de signer l'ensemble des applications. Ce certificat est généré dans Xcode, et stocké dans les Trousseaux d'accès (Keychain).
- Le développeur développe son application (ou simplement une nouvelle version) dans Xcode (ou non).
- Au moment de la compilation, Xcode va signer l'application à l'aide du "Developer ID Certificate". Si le développement n'est pas réalisé dans Xcode, la [signature peut être réalisée en ligne de commande](https://medium.com/better-programming/writing-ios-apps-without-xcode-89450d0de21a), même si Xcode doit de toute façon être installé sur la machine.
- L'application signée peut être distribuée, avec à la clé une authentification (relative) de l'auteur, et une garantie d'intégrité des fichiers. A ce stade, depuis 2019 et macOS Mojave, les applications doivent aussi être validées par le "notaire" d'Apple (processus de *notarisation* en bon franglais...) ; on y reviendra dans un prochain article. 

### Le contrôle des signatures au sein de macOS

Lorsqu'un utilisateur lance pour la première fois une application, qu'il l'ait récupérée via l'Apple Store, par Internet, une clé USB ou tout autre moyen, macOS vérifie sa provenance :

#### 1/ App Store

Si elle a été obtenue via l'App Store, c'est qu'elle est dotée d'une signature (et de plusieurs autres caractéristiques) : elle est autorisée par défaut.

#### 2/ Application signée hors App Store

Si elle a été signée par un développeur, mais qu'elle n'a pas été téléchargée via l'App Store, le comportement dépend de la configuration du système dans **Préférences Système** > **Sécurité et confidentialité** > **Général**, comme on peut le voir sur la capture d'écran ci-dessous :

![macOS-SecConf-Gnal](/images/blog/macOS-SecConf-Gnal.png)

En fonction de sa configuration, les applications signées par un développeur identifiée seront, ou non, installées sans avertissement supplémentaire.

#### Application non signée

Si l'application n'a pas été signée, le comportement sera différent sous Mac Intel et sous Mac M1

Sous <u>Mac Intel</u>, un message d'avertissement sera affichée en indiquant que l'application n'est pas fiable. Il sera alors possible de retourner dans **Préférences Système** > **Sécurité et confidentialité** > **Général**, et cliquer sur le bouton **Ouvrir quand même** pour lancer l'application. Elle est alors enregistrée comme une application reconnue, et aucune validation ne sera plus demandée.

Sous <u>Mac M1</u>, l'application ne sera tout simplement pas lancée.

**NB** : comme indiqué un peu plus haut, le comportement de macOS face aux applications issues de macOS ou non, signées ou non, a sensiblement été modifié depuis Mojave. Les informations données ici sont donc incomplètes au regard de la fonction de "notarisation".

## Fonctionnement détaillé

La vérification réalisée sur les systèmes macOS depuis Catalina suit la trame suivante :

- Vérification du statut de quarantaine 
- Dans le cas d'une application en quarantaine :
  1. Validation du hachage de l'exécutable
  2. Validation de la signature et du certificat
  3. Validations complémentaires
  4. Exécution de l'application

TODO : suivre la trame ci-dessous

### Vérification du statut de quarantaine

Quelle est la condition pour déclencher la vérification d'une signature ? Lorsque son statut est "sous quarantaine". Ce statut est stocké directement dans les métadonnées de l'application elle-même. Il est possible de le visualiser à l'aide de la commande `xattr`: 

```
$ xattr -l vlc-3.0.12-intel64.dmg
[...]
com.apple.quarantine: 0083;600b4574;Safari;06262AFD-1B64-405F-8FD7-60017E821B78
```

La chaîne `com.apple.quarantine`atteste de l'état du fichier du point de vue de macOS. Il a bien été récupéré via Safari. Une fois l'archive ouverte et l'application copiée dans `/Applications`, on observe qu'il en est de même pour l'application elle-même :

```
$ xattr -l VLC.app
com.apple.quarantine: 0183;600b4574;Safari;06262AFD-1B64-405F-8FD7-60017E821B78
```

### Validation du hachage de l'exécutable

On a vu plus haut que lorsque le développeur signe son application, un hachage est calculé pour l'ensemble de l'application distribuée. Le calcul de l'intégrité est réalisé de manière récursive : le hachage de l'application est calculé à l'aide de ceux des fichiers présents dans les répertoires et les sous-répertoires de l'application. 

Le hachage est comparé à ceux déjà connus par macOS dans la base de données locale de sécurité :

- Si c'est le cas, on passe à l'étape suivante.
- Si ce n'est pas le cas, macOS contacte les serveurs d'Apple pour vérifier que le hachage ne correspond pas à celui d'un malware connu. 
  - Si c'est un malware, l'exécution est alors arrêtée immédiatement.
  - Sinon, le hachage de l'application est ajouté à la base de données de sécurité, et macOS poursuit l'exécution.

La validation du hachage est réalisée soit au lancement de l'application si l'utilisateur a téléchargé l'application, soit même au moment de l'extraction de l'archive.

### Validation de la signature et du certificat

Lors du premier lancement de l'application, Gatekeeper continue son travail par la vérification du certificat.

Si le certificat est celui d'un développeur signé par un certificat racine Apple, les vérifications suivantes sont effectués. Dans le cas contraire, on passe directement à l'étape d'exécution de l'application. Notons qu'il est toujours possible de signer une application avec un certicat généré "en interne", sans faire intervenir Apple.

#### Validité de la signature

Bien entendu, la première vérification consiste à s'assurer que la signature est bien valable

#### Validité du certificat

En complément de l'ensemble des tests indiqués jusqu'ici, macOS va aussi vérifier que le certificat du développeur qui a signé l'application n'a pas expiré. 

macOS réalise l'opération  en se connectant au site `ocsp.apple.com`. Ceci permet notamment à Apple de révoquer les certificats d'applications malveillantes, et notamment lorsque certaines d'entre elles parviennent à intégrer le Mac App Store, comme c'est arrivé à plusieurs reprises et notamment [récemment](https://www.macg.co/logiciels/2020/10/apple-notarise-de-nouveau-un-malware-117279).

Ainsi, et pour être très clair, à chaque fois que macOS rencontre une nouvelle application, ou s'il n'a pas vérifié l'expiration du certificat depuis un certain temps, le système se connecte sur les serveurs d'Apple. Bien sûr, si le Mac n'est pas connecté à Internet, il ne pourra valider le certificat.

Lors de la sortie de macOS Big Sur en novembre dernier, le service ocsp.apple.com avait été fortement perturbée pendant plusieurs heures : partout dans le monde, de très nombreux utilisateurs de macOS ont constaté une extrême lenteur dans le lancement de leurs applications... En revanche, si le Mac n'est pas connecté à Internet, cette étape est sautée.

La validité du certificat n'est pas vérifié à chaque lancement d'une application. macOS met en place un cache, et y stocke les informations sur les certificats. A l'heure actuelle, les informations sont stockées dans le cache pendant 12 heures, contre 5 minutes il y a quelques mois. Il semble qu'Apple puisse changer à distance la durée de validité dans le cache.

### Validations complémentaires réalisées par Gatekeeper

Au fur et à mesure des versions, Apple a rajouté de nouvelles fonctionnalités à Gatekeeper, qui ne se contente pas uniquement de vérifier l'intégrité de l'application et le signataire, mais réalise des validations complémentaires

- Vérification que les librairies chargées par le programme *via* `RPATH` ou un lien à l'aide d'un chemin absolu soient bien soit stockées soit dans l'application, soit dans les chemins standards des librairies sous macOS (`/System/Library/`, `/Library`, `/usr/lib/`...)
- Validation que l'application ne contient pas des liens symboliques qui ne pointent nulle part, vers des chemins "anormaux" au regard de l'application ou ailleurs que vers `/System` ou `/Library`.

NB : c'est à ce stade que d'autres vérifications spécifiées par le développeur sont réalisées, désignées par Apple sous la forme de *Designated Requirements*. Ces fonctionnalités sont hors du périmètre du présent document, mais peuvent être consultées sur la [page  consacrée](https://developer.apple.com/library/archive/technotes/tn2206/_index.html) au sujet sur developer.apple.com

### Exécution de l'application

L'application peut maintenu être exécutée. Avant cela, macOS affiche la fenêtre suivante à l'utilisateur, pour obtenir son accord.

![macOS-Gatekeeper-1](/images/blog/macOS-Gatekeeper-1.png)

Ceci est aussi appliqué lorsqu'on utilise la commande `open` en ligne de commande, qui fait appel au bundle `LaunchServices`,  comme le fait le Finder lorsqu'on double clique sur une application.

L'application est finalement lancée.

Une fois l'application lancée et que l'utilisateur a validé qu'il est bien OK pour l'ouvrir, la métadonnée de l'application est changée pour  stocker le fait que Gatekeeper a validé l'application :

```
$ xattr -l VLC.app
com.apple.quarantine: 01c3;600b4574;Safari;06262AFD-1B64-405F-8FD7-60017E821B78
```

## Visualisation en ligne de commande

 La visualisation de la signature se fait à l'aide de la commande `codesign`:

```
$ cd /Applications
$ codesign -dvv iTerm.app
Executable=/Applications/iTerm.app/Contents/MacOS/iTerm2
Identifier=com.googlecode.iterm2
Format=app bundle with Mach-O universal (x86_64 arm64)
CodeDirectory v=20500 size=135009 flags=0x10000(runtime) hashes=4208+7 location=embedded
Signature size=8979
Authority=Developer ID Application: GEORGE NACHMAN (H7V7XYVQ7D)
Authority=Developer ID Certification Authority
Authority=Apple Root CA
Timestamp=7 Dec 2020 at 05:59:36
Info.plist entries=51
TeamIdentifier=H7V7XYVQ7D
Runtime Version=11.0.0
Sealed Resources version=2 rules=13 files=316
Internal requirements count=1 size=216
```

On voit ici un certain nombre d'information identifiant l'application (`Executable`, `Identifier`) et ses caractéristiques (`Format` : ici l'application peut s'exécuter sur les Mac Intel et M1). On note aussi :

- `Authority` : le développeur qui a signé l'application, ici George Nachman
- `'Authority` (2ème et 3ème occurence : la chaîne des certificats jusqu'au certificat racine, ici `Apple Root`, le certificat racine, conformitant que la signature a bien été effectuée *via* Apple
- `Timestamp` : la date de la signature
- `TeamIdentifier` : l'équipe de développeur (ici identique au développeur lui-même, comme le montre la chaîne `H7V7XYVQ7D`
- `Sealed Resources version` : le nombre de fichiers inclus dans la signature (`316`) et ainsi que le nombre de règles devant être vérifiées pour que la signature soit bien validée. On y revient plus bas.

Mais il est aussi possible d'avoir des informations sur l'intégrité et les informations de hachage, en demandant plus de détail (`-vvv`):

```
$ codesign -dvvv iTerm.app
Executable=/Applications/iTerm.app/Contents/MacOS/iTerm2
[...]
Hash type=sha256 size=32
CandidateCDHash sha256=cdb59caa11ae37c45537e33beff454156f5d90ad
CandidateCDHashFull sha256=cdb59caa11ae37c45537e33beff454156f5d90ad4d5dc00d393a47e4c6c28f91
Hash choices=sha256
CMSDigest=cdb59caa11ae37c45537e33beff454156f5d90ad4d5dc00d393a47e4c6c28f91
CMSDigestType=2
CDHash=cdb59caa11ae37c45537e33beff454156f5d90ad
[...]
```

L'intégrité est validée à l'aide d'un hachage (`CandidateCDHashFull`), calculé avec un SHA256. 

La commande `codesign -d` ne réalise pas la vérification de la signature, elle se contente d'afficher ce qui est disponible au niveau de la signature de l'application.  Pour visualiser l'ensemble des tests réalisés par Gatekeeper, on peut  utiliser la commande `codesign --verify`:

```
$ codesign --verify --deep --strict --verbose=2 iTerm.app
--prepared:/Applications/iTerm.app/Contents/MacOS/iTermServer
--validated:/Applications/iTerm.app/Contents/MacOS/iTermServer
--prepared:/Applications/iTerm.app/Contents/MacOS/image_decoder
--validated:/Applications/iTerm.app/Contents/MacOS/image_decoder
--prepared:/Applications/iTerm.app/Contents/Frameworks/libswiftAppKit.dylib
--validated:/Applications/iTerm.app/Contents/Frameworks/libswiftAppKit.dylib
--prepared:/Applications/iTerm.app/Contents/Frameworks/libswiftObjectiveC.dylib
--validated:/Applications/iTerm.app/Contents/Frameworks/libswiftObjectiveC.dylib
--prepared:/Applications/iTerm.app/Contents/Frameworks/libswiftXPC.dylib
--validated:/Applications/iTerm.app/Contents/Frameworks/libswiftXPC.dylib
[...]
--prepared:/Applications/iTerm.app/Contents/Frameworks/CoreParse.framework/Versions/Current/.
--validated:/Applications/iTerm.app/Contents/Frameworks/CoreParse.framework/Versions/Current/.
--prepared:/Applications/iTerm.app/Contents/Frameworks/libswiftDarwin.dylib
--prepared:/Applications/iTerm.app/Contents/Frameworks/libswiftCloudKit.dylib
[...]
iTerm.app: valid on disk
iTerm.app: satisfies its Designated Requirement
```

On voit la vérification de l'ensemble des fichiers qui composent l'application, jusqu'à la conclusion `valid on disk` qui indique que la signature est valide.

La commande suivante de même le test :

```
$ spctl -a -t exec -vv VLC.app
/Applications/VLC.app: accepted
source=Notarized Developer ID
origin=Developer ID Application: VideoLAN (75GAHG3SZQ)
```

## Les limites de Gatekeeper

### Un système de quarantine totalement optionnel

La fonction quarantine a été instaurée sous Mac OS X 10.5 pour differencier les fichiers considérés comme potentiellement dangereux (récupéré en pièce jointe d'un mail, via AirDrop ou récupéré depuis un autre moyen comme une clé USB...) des fichiers déjà validés. Étonnament, il est de la responsabilité de l'application qui récupère l'application (Safari, Mail, Firefox...) de positionner ce drapeau `com.apple.quarantine`. Si elle ne le fait pas, alors l'application ne sera pas analysée par Gatekeeper. Ceci permet notamment à un auteur de malware de récupérer des applications sans devoir les soumettre à l'analyse de Gatekeeper.

```
$ wget https://videolan.mirror.liteserver.nl/vlc/3.0.12/macosx/vlc-3.0.12-intel64.dmg
[...]
$ xattr -l vlc-3.0.12-intel64.dmg
$
```

Pas de `com.apple.quarantine` ici, et il en sera de même pour l'application extraite de l'archive. Le fichier ne sera donc pas vérifié par Gatekeeper.

### Contourner la protection contre le lancement en ligne de commande d'une application en quarantaine 

Le système qui empêche l'exécution *via* `open VLC.app` vu ci-dessus est particulièrement imparfait. Même si une application et l'ensemble de son contenu sont sous quarantaine, et il est tout de même possible de lancer directement le binaire contenu dans l'arborescence de l'application : 

```
$ /VLC.app/Contents/MacOS/VLC
VLC media player 3.0.12 Vetinari (revision 3.0.12-1-0-gd147bb5e7e)
[00007ff4f7c24e10] main libvlc: Lancement de vlc avec l'interface par défaut. Utiliser « cvlc » pour démarrer VLC sans interface.
[...]
```

L'effet pour l'attaquant est alors nul...

Par ailleurs, un tel lancement a aussi pour effet de rendre inopérant la validation de l'utilisateur exigée lors de l'ouverture normale de l'application. Après avoir réalisé l'opération ci-dessous, il est possible de faire tout simplement :

```
$ open VLC.app
```

ou lancer simplement l'application depuis le Finder

Ceci n'est de toute façon que peu surprenant : l'utilisateur courant est le propriétaire des applications installées, comme on peut le voir ici :

```
$ ls -l /Applications/VLC.app
total 0
drwxr-xr-x@ 8 julien  admin  256 16 déc 19:06 Contents
```

### Contourner la protection en supprimant le drapeau de quarantaine

De manière plus simple, si un attaquant a la capacité d'exécuter des commandes arbitraires telles que ci-dessus sous l'identité de l'utilisation, il lui suffirait, plus simplement, de supprimer le flag quarantaine :

```
$  attr -d com.apple.quarantine  /Applications/VLC.app
```

et ainsi Gatekeeper ne s'activerait simplement pas.

## Notes

[^1] : Rappelons qu'une application sous macOS n'est pas un binaire exécutable, comme sous Windows ou Linux, mais un ensemble de répertoire contenant binaire(s), librairies, script et différentes ressources.

### Sources

- [Apple - macOS Code Signing In Depth](https://developer.apple.com/library/archive/technotes/tn2206/_index.html)
- https://eclecticlight.co/2019/05/27/is-a-mac-os-x-gatekeeper-bypass-what-it-says/
- https://eclecticlight.co/2020/07/08/will-macos-protect-you-from-ransomware-like-thiefquest/
- https://eclecticlight.co/2020/11/16/checks-on-executable-code-in-catalina-and-big-sur-a-first-draft/






XProtect : https://www.sentinelone.com/blog/malware-can-easily-defeat-apples-macos-security/
MRT : https://www.sentinelone.com/blog/apples-malware-removal-mrt-tool-update/
Notarization :

- https://www.sentinelone.com/blog/maco-notarization-security-hardening-or-security-theater/
- https://www.macg.co/os-x/2018/06/securite-dans-macos-mojave-un-notaire-pour-arreter-les-frais-102584
- https://developer.apple.com/documentation/xcode/notarizing_macos_software_before_distribution


Yara : 
- https://yara.readthedocs.io/en/latest/
- https://labs.sentinelone.com/detecting-macos-gmera-malware-through-behavioral-inspection/
- https://www.sentinelone.com/blog/macos-security-updates-part-3-apples-whitelists-blacklists-and-yara-rules/



Gatekeeper

- [Safely open apps on your Mac](https://support.apple.com/en-us/HT202491)
- https://developer.apple.com/library/archive/technotes/tn2206/_index.html
- https://www.sentinelone.com/blog/how-malware-bypass-macos-gatekeeper/





Ce que Apple e fait pas : surveiller les ressources en temps réel.
