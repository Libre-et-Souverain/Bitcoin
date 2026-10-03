# Construisez votre nœud Bitcoin "from scratch *" puis devenez un gardien souverain

**\* construire soi-même : compiler le code source des applications critiques et paramétrer le tout pour une sécurité et une confidentialité satisfaisantes.**


# Matériel dédié

Afin d'éviter toute déconvenue il est souhaitable de commencer par là.

## Requis machine de base

* Microprocesseur suffisamment véloce, le strict minimum étant le Raspberry Pi4
* Mémoire vive 8 Gio, fonctionne avec 4 Gio mais limitant dans les fonctionnalités annexes.
* Mémoire de masse 2 To de type SSD ou NVMe
* Interface réseau filaire type RJ45 reliée à internet et à votre réseau local.
* Système d'exploitation : Open Source
* Installation et administration par accès ssh (interface en ligne de commande)

dans les détails

* Un nœud Bitcoin doit tourner sans jamais discontinuer et a priori pendant des années
* En neuf il y a les mini PC avec des CPU frugaux comme le N100
* D'occasion avec les PC de taille réduite comme la série des DELL Optiplex SFF
* Deux périphériques stockage de masse seront un véritable plus, il est opportun de séparer le système d'exploitation de la copie de la blockchain Bitcoin.
* Pour les données blockchain utiliser un SSD ou mieux un NVMe.M2
* Éviter toute connexion au réseau par ondes hertziennes, les fils c'est bien plus fiable !
* Les choix doivent être dictés par l'usage, la simplicité et la sobriété sont bien souvent une garantie de silence, de fiabilité et de longévité.
* À prévoir le cas échéant, une alimentation secteur secourue en cas de coupure.

## Machine minimale

Raspberry Pi 5

* cpu Arm Cortex-A76 / 4 Cores / 2.4 Ghz / TDP 12 W (gravure 16 nm )
* 8 Gio de Ram / LPDDR4X 4267 MHz
* Connecté au réseau par RJ45
* OS + données blockchain sur : HAT+ Geekworm X1001 avec NVMe.M2 de 2 To (¹)
* Debian GNU/Linux arm64 version "console-only"
* Le système démarre directement sur le NVMe.M2 pour une fiabilité accrue
* Consommation 4.5 W (moyenne relevée sur 20 jours)
* Alimentation secourue :
  * à condition que la consommation des périphériques soit < 600 mA (hdd usb interdits) il est possible d'utiliser une "Power Bank" 5V / 3A, vérifier que celle-ci soit capable de charge bidirectionnelle (alimenter le Raspberry Pi en même temps qu'elle se recharge).
  * si le lieu est exotique et qu'il n'y a pas de secteur 230v, possibilité d'alimentation par batterie avec un convertisseur DC / DC comme entrée 9-36V / sortie 5V 5A, disque USB3 supporté si consommation < 1.6A, tapez `pinctrl | grep MAX` si 'hi' est affiché dans la ligne c'est ok jusqu'à 1.6A.

(¹) pourquoi un seul périphérique de stockage :

* la carte SD est un poison dans le temps pour le système d'exploitation, à éviter absolument !
* jusqu'à présent il est impossible de booter sur NVMe si plus d'un sur la carte d'extension, à vérifier dans le temps bien sûr.
* booter sur usb est nul vu le concept de cette mini-machine, autant passer sur un système plus classique et parfois moins cher si acquis d'occasion.

## Machine classique

Dell Optiplex 5050 SFF

* cpu intel i3-6100 / 2 Cores / 4 Threads / 3.70 Ghz / TDP 51W (gravure 14 nm )
* 8 Gio de Ram / DDR4 2133 MHz
* Connecté au réseau par RJ45
* Operating System sur SSD sata 240 Go
* Données blockchain sur NVMe.M2 de 2 To (le connecteur M2 2280 est sur la carte mère)
* Debian GNU/Linux PC 64 bits installée "console-only"
* Consommation 11.2 W (moyenne relevée sur 15 jours)
* Alimentation secourue : onduleur. Si l'onduleur est capable de dialoguer avec le PC, vous avez upsmon pour arrêter le système proprement en cas de coupure prolongée. Et puis oui les onduleurs c'est pénible et coûteux dans le temps. Soit on change les batteries au plomb tous les 4 à 5 ans, soit on s'aperçoit qu'il faut changer la batterie lorsque y a une coupure puisque l'onduleur n'a pas fait son boulot et que le système s'est arrêté en vrac ! Suivant le camp ou vous vous situez ne mettez pas d'onduleur … ou bien regardez du côté des stations d'énergie à batterie LiFePO4 dont la durée de vie est en principe > 10 ans.

## Installation du système d'exploitation

Vous trouverez sur le net de bons tutoriels pour installer le système d'exploitation. L'Open Source n'est pas une idéologie ou une méthode, c'est un rempart pour l'individu dans le cyberespace face à la centralisation et au contrôle. Le système d'exploitation est une des premières briques, donc tout comme le code source de Bitcoin il est plus que souhaitable d'utiliser un OS Open Source, Linux est le choix le plus pratique et le plus répandu. Ici également visez la simplicité, pas d'interface graphique, pas de fioritures inutiles, juste l'essentiel. La machine sera entièrement dédiée à la fonctionnalité de nœud. Par choix à l'installation j'ai créé un seul utilisateur que j'ai appelé 'btc-node'. Pour finir, juste 3 "tips" sur PC si vous avez choisi Debian la mère des distributions Linux :

* si vous préférez utiliser sudo : à "root password" laissez le champ vide, cela installe sudo et l'utilisateur que vous allez renseigner ensuite se verra attribuer les droits.
* Ext4 est par défaut le type de partition utilisé à l'installation de l'OS, ok pour moi.
* à sélection des logiciels, cochez uniquement 'serveur SSH' et 'utilitaires usuels du système'.

Installation de l'OS bouclée, pour découvrir l'adresse ip de la machine faites `ip a`, puis à partir de la vous pouvez débrancher écran et clavier. Accédez à votre machine dédiée par ssh sur le même réseau local avec un PC de bureau.

```bash
ssh nom_utilisateur@IP_machine_distante
```

Tout ce qui suit est décrit sous Debian Linux, adaptez si vous n'utilisez pas cette distribution.

## Performance stockage de masse

```bash
sudo apt-get update
sudo apt-get install hdparm
# Identifier le périphérique et ou la partition
lsblk
# Remplacer le /dev par celui de votre machine
sudo hdparm -Tt /dev/nvme0n1
```

| Raspberry Pi 5 / NVMe.M2 2To: |
|----|
| Timing cached reads:   6992 MB in  2.00 seconds = 3498.29 MB/sec |
| Timing buffered disk reads: 1320 MB in  3.00 seconds = **439.73 MB/sec** |

| **Dell OptiPlex 5050 / NVMe.M2 2To (le NVMe est identique à celui du Rpi) :** |
|----|
| Timing cached reads:   15686 MB in  1.98 seconds = 7906.21 MB/sec |
| Timing buffered disk reads: 7534 MB in  3.00 seconds = **2511.01 MB/sec** |

| **A titre de comparaison** **/ Disque dur 7200 tr/mn sur port sata :** |
|----|
| Timing cached reads:   32956 MB in  1.97 seconds = 16706.10 MB/sec |
| Timing buffered disk reads: 568 MB in  3.00 seconds = **189.05 MB/sec** |

La première ligne est une indication sur le débit obtenu en lecture à partir du cache tampon du système d'exploitation sans accès au disque. *Cette mesure est essentiellement une indication du débit du processeur, du cache et de la mémoire du système testé.*

La deuxième ligne est une indication du débit que le périphérique supporte en lecture de données séquentielles sans surcharge du système de fichier. C'est cette ligne qu'il convient d'apprécier puisque dans le cas d'usage d'un nœud Bitcoin certains processus vont demander la lecture d'une très grande quantité de données.

Avant d'écrire des centaines de Gio sur un stockage de masse qui a un passé inconnu utilisez `smartctl` du paquet "smartmontools" pour apprécier son état de santé :

```bash
# Si besoin installer ce package
sudo apt-get update
sudo apt-get install smartmontools

# Identifier le périphérique
lsblk -o NAME,MODEL,SIZE

# Test sur mon NVMe de 2To
sudo smartctl -A /dev/nvme1n1

=== START OF SMART DATA SECTION ===
Data Units Written:                 44 888 895 [22,9 TB]
Power On Hours:                     14 044
```

14 044 heures soit 585 jours ON avec 23 TB d'écriture. Donc environ 0,04 TB jour.  Si le TBW (Tera Bytes Write) annoncé par le constructeur est de 110, mon NVMe sera cuit en 2750 jours soit 7,5 ans d'usage. Le TBW du matériel est primordial pour la apprécier la durabilité, les chiffres annoncés par les fabricants varient dans de grandes proportions (bas de gamme ≈ 100 et haut de gamme > 1000). A matériel équivalent, plus le NVMe est de grande capacité et plus le TBW sera grand.

## Performance mémoire vive

```bash
sudo apt-get install sysbench
sysbench memory run
```

| Raspberry Pi 5 / Ram 8 Go | **Dell OptiPlex 5050 / Ram 8 Go** |
|----|----|
| 3634.62 MiB/sec | 6690.11 MiB/sec |

Le débit indiqué ici est à rapprocher de la première ligne de la performance du stockage de masse, les chiffres doivent être approximativement similaires.

## Outils complémentaires éventuels

* Monitoring : `sudo apt-get install btop`
* Descriptif de la machine : `sudo apt-get install fastfetch`, pour une information à l'ouverture du shell ajoutez `fastfetch` à la fin de `.bashrc`
* Depuis une éternité il existe sous Debian un gestionnaire de fichier en mode texte, un clone de Norton Commander, que j'affectionne bien et avec lequel vous pouvez utiliser la souris en mode texte dans un terminal : "Midnight Commander" alias `mc` si cela vous tente c'est `sudo apt-get install mc` .

# Logiciel Bitcoin

## **Mettre à jour le système d'exploitation**

```bash
sudo apt-get update
sudo apt-get upgrade
```

## **Installer les outils et les librairies**

Installer les dépendances nécessaires à la construction

```bash
sudo apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3 libevent-dev

sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-test-dev libboost-thread-dev

sudo apt-get install libsqlite3-dev

sudo apt-get install libzmq3-dev

# Optionnel, uniquement si vous choisissez l'architecture multiprocess (IPC)
# disponible a partir de Bitcoin Core v30 (voir Options de compilation)
# sudo apt-get install libcapnp-dev capnproto
```

Voir les fichiers installés par un package `sudo dpkg -L nom_du_paquet`

Voir les versions des librairies installées `apt show nom_de_la_librarie`

Si besoin de dé-installer une librairie, c'est `sudo apt-get remove nom_de_la_librarie`

Faire le ménage après des suppressions de paquets `sudo apt-get autoremove`

## Compiler Bitcoin

D'après ce que l'on sait, durant les années 2007 à 2008 une (ou plusieurs) personne a pondu pas mal de lignes de code C++, puis les a lâchées dans le cyber-espace en 2009, un outil fonctionnel et potentiellement révolutionnaire est né. [Au fil du temps le code est ré-écrit](https://bitcoin.fr/au-coeur-du-code/), fiabilisé et amélioré, au début par quelques individus puis par une équipe connue depuis 2011 sous le nom de "core developers". Cela peut paraître perturbant mais tout code informatique doit être amélioré, fiabilisé et sécurisé. L'idée de départ parait parfaite mais sa transcription exacte en code informatique est délicate, le résultat est donc sûrement qualifiable d'imparfait. Qui plus est : le 3 janvier 2009 le jour du bloc genesis, le comité était réduit à une poignée d'individus voire moins, face à des milliers de lignes de code mettant en oeuvre un concept jusque-là jamais atteint, une équipe sera ensuite bien plus forte qu'un ou quelques individus. Plus le temps passe et plus nous pouvons dire que cela devient moins imparfait ***à condition*** que l'équipe en charge du code conserve l'idéologie et la philosophie initiale. A souligner également, l'environnement autour de Bitcoin n'est pas figé, il peut changer ou évoluer. Ici également il faudra que les "core developers" effectuent ce qu'il faut. En tant qu'individu et gardien du registre distribué vous ferez respecter le consensus, face au code vous avez votre mot à dire, vous pouvez le choisir, avoir le choix est important et primordial, c'est le fondement même de toute véritable démocratie. Votre voix compte 1:1 comme tous les autres, vous participerez à la gouvernance avec un modèle de décision strictement horizontal. Si l'époque le demande, renseignez-vous, puis choisissez, ensuite vous allez mettre en oeuvre le processus de compilation de 80K lignes de C++ qu'est devenu Bitcoin. Et fait important, **vous obtiendrez un binaire que vous avez construit vous même à partir du code source.**

### **Télécharger le code source de Bitcoin**

à partir des endroits où il est officiellement maintenu :

```bash
sudo apt-get install git
mkdir ~/code
cd ~/code

# Non exhaustif ... c'est a vous de chercher et de choisir ...
git clone https://github.com/bitcoin/bitcoin.git      # Pour Bitcoin Core

# ou

git clone https://github.com/bitcoinknots/bitcoin.git # Pour Bitcoin Knots
# /!\ Octobre 2026 : jusqu'a preuve du contraire Knots a quitte la chaine BTC
# /!\ (BIP110 puis hard fork Blake2b), lire "Choix du code" ci-dessous

# ou encore
?
```

### Choix du code

- [ ] **Octobre 2024** : après avoir cloné un repository, choisissez la version logicielle du nœud que vous souhaitez construire, faites `git -C ~/code/bitcoin tag` ou allez voir les [releases sur le repository Bitcoin](https://github.com/bitcoin/bitcoin/releases), nous sommes loin de la "Guerre des blocs" de 2015 à 2017, la période est apparemment calme et sereine, donc se limiter à deux options me semble raisonnable, cela a donné pour moi :

* 'v28.0' Core, pour la dernière "final" qui intègre des améliorations et des correctifs.
* 'v27.1' Core, l'avant-dernière "final", contient des correctifs pour l'essentiel.

- [ ] **Décembre 2025** : l'année 2025 a vu apparaître une crise du spam avec OP_RETURN. OP_RETURN est un opcode ou une instruction dans le langage de script de Bitcoin qui permet d'intégrer des données arbitraires dans une sortie de transaction que l'on ne peut pas dépenser. Cela signifie que ces données sont stockées sur la blockchain sans créer une sortie valide qui pourrait être réclamée plus tard. Historiquement OP_RETURN existe depuis 2009, Bitcoin v0.9 de 2014 l'a seulement rendu standard avec une limite de 40 octets, passée à 80 octets en 2015, cela permet l'ajout de petites quantités de données dans les transactions. Ces données non directement monétaires ont des cas réels d'usage : des preuves de timestamp, des hachages pour des applications comme le notaire virtuel, des métadonnées comme la payload de la Tx0 Whirlpool, etc ... Avant la sortie de Bitcoin Core 30.0 d'octobre 2025 un seul OP_RETURN est autorisé par transaction avec une limite standard de relais (relay policy) de 80 octets pour empêcher une surcharge de la blockchain avec des données inutiles et qualifiables de spam. Depuis la V30 de Core cette limite historique a été levé en autorisant de multiples OP_RETURN par transaction et en les limitant à 100 000 octets au lieu de 80 (voir OP_RETURN dans le glossaire pour plus de détail) ; pour résumer, à partir de la v30 de Core la limitation des données non financières dans un bloc est tenue par celle de la taille des blocs à 4 000 000 WU (BIP141 / SegWit adopté en 2017). Les débats autour d'OP_RETURN portent souvent sur l'équilibre entre nouvelles fonctionnalités et la protection contre le spam, qui pourrait gonfler la blockchain, augmenter les coûts pour les utilisateurs ordinaires, et ainsi diminuer la dé-centralisation. A l'opposé de cette permissivité amenée par Bitcoin Core v30 qui semble laisser la porte grande ouverte aux usages non monétaires les plus divers, ce qui au final éloigne Bitcoin de sa mission initiale consistant à transférer sans aucune censure de la valeur en pair à pair, il y a [Bitcoin Knots](https://github.com/bitcoinknots/bitcoin) un fork de Bitcoin Core orienté décentralisation et filtrage des transactions considérées comme abusives. Par défaut, un nœud Knots ne relaie par transaction qu'un seul OP_RETURN limité par défaut à 83 octets. Sur [The Bitcoin Portal](https://thebitcoinportal.com/onchain/spam-analysis/overview) figure un dashboard qui distingue les transactions financières des non financières dans le temps. Pour plus d'information sur comment est arrivé la crise du Spam voir [ici](https://www.citadel21.com/the-merge).
- [ ] 8 août 2026 : les nœuds Knots qui avaient activé BIP110 ont échoués ! Un fork de chaîne s'est produit après le bloc 961 631. Ce soft fork temporaire (Reduced Data Temporary Softfork) initié autour de Bitcoin Knots, de Luke Dashjr et de Dathon Ohm (pseudonyme de l'auteur de BIP110) prévu pour limiter pendant environ un an les données non financières stockées dans les blocs en réponse à Bitcoin Core v30 est devenu un hard fork. Après le bloc 961 631 la chaîne BIP110 est restée quasiment figée puisqu'elle a hérité de la difficulté de minage de la chaîne principale. Du 8/8/26 au 28/8/26 seuls 8 blocs seront produits sous SHA-256d. A partir du 15 août Luke Dashjr a décidé qu'il fallait changer l'algorithme de preuve de travail de SHA-256d à Blake2b pour contourner les ASIC utilisés par Bicoin. Le 30/08/26 le n°961640 est le 1er bloc produit avec Blake2b. Ce n'est plus BTC c'est BTCB2, un alt-coin jusqu'à preuve du contraire. /!\\ *Les deux chaînes ont le même historique jusqu'au bloc 961 631. N'importez jamais vos clés privées ni votre seed dans un outil lié au fork. Si vous voulez manipuler des jetons BIP110, séparez d'abord vos fonds, sinon une de vos transactions pourrait être rejouée sur Bitcoin et vous faire perdre de vrais BTC* /!\\"
- [ ] **Octobre 2026** : **jusqu'à preuve du contraire, Bitcoin Knots a pris une autre direction que Bitcoin.** Avec leur configuration par défaut, les versions publiées depuis mai 2026 sur le [dépôt Knots](https://github.com/bitcoinknots/bitcoin/releases) ne suivent plus la chaîne principale BTC :
  * `v29.3.knots20260507` (8 mai 2026) : **dernière version de Knots qui suit la chaîne principale** (SHA-256d, sans BIP110/RDTS). Elle refuse même de démarrer si `consensusrules=rdts` est présent dans `bitcoin.conf`. Mais elle n'est plus maintenue, aucun correctif de sécurité ne viendra.
  * `v29.3.knots20260508` (9 mai 2026) et `v29.4.knots20260508` (7 août 2026) : appliquent les règles BIP110 (RDTS). Les binaires officiels les appliquent même sans `consensusrules=rdts`, avec un simple avertissement dans les logs.
  * `v29.4.1.knots20260508` (2 septembre 2026) : hard fork vers la preuve de travail Blake2b.
  * `v29.4.2.knots20260508` (21 septembre 2026, marquée "Latest") : poursuit sur la chaîne Blake2b.
  * Conséquence : faire `git tag` puis prendre la dernière version de Knots construit aujourd'hui un nœud d'une autre chaîne. Si vous avez un nœud Knots, contrôlez sa version par `bitcoin-cli -version` et comparez le résultat de `bitcoin-cli getbestblockhash` avec un explorateur indépendant (mempool.space par exemple).

**Mon choix d'octobre 2026 : Bitcoin Core** `v31.1` **avec OP_RETURN limité à 83 octets.** Mes arguments :

* Maintenu : Core maintient ses trois dernières versions majeures (31.x, 30.x et 29.x) et la v31.1 du 8 juillet 2026 est la dernière stable. La v32.0 n'est qu'en release candidate, j'attends sa version finale.
* Sur la bonne chaîne : mêmes règles de consensus que le réseau BTC.
* Les options `datacarrier` et `datacarriersize` existent toujours en v31 et ne sont plus dépréciées. Remettre `datacarriersize=83` restaure la politique de relais d'avant la v30 : 83 octets de script, soit 80 octets de données utiles. Depuis la v30 la limite s'applique à la somme des scripts de toutes les sorties OP_RETURN d'une transaction : un OP_RETURN de 80 octets de données passe, deux OP_RETURN de 40 octets (2 x 42 = 84 octets) sont refusés.
* `permitbaremultisig=0` refuse de relayer les sorties multisig "nues" (bare multisig), détournées notamment par les "Stamps" pour stocker des données dans l'ensemble des UTXO. C'est pire qu'un OP_RETURN car ces sorties ne sont jamais dépensées ni élaguées. C'est le défaut de Knots, celui de Core est `permitbaremultisig=1`.
* Compatible avec Electrs v0.12.0 qui exige Bitcoin Core 31 ou plus (voir Logiciel serveur Electrum).

Limites, pour être honnête : ces réglages sont la politique de relais (policy) de **mon** nœud, pas du consensus. Ma mempool filtre et mon nœud ne relaie pas ces transactions, mais un bloc miné contenant de gros OP_RETURN reste valide et mon nœud l'accepte, heureusement, sinon il quitterait la chaîne comme les nœuds BIP110 en août 2026. Core ne propose pas les autres filtres de Knots (inscriptions dans le witness, `datacarrierfullcount` ...). A l'arrivée d'un nouveau bloc, les transactions filtrées peuvent devoir être téléchargées en plus (compact block relay moins efficace), et l'estimation des frais se base sur une mempool un peu moins complète.

Paramétrage correspondant dans `bitcoin.conf` (déjà intégré plus bas dans Paramétrage de Bitcoin) :

```bash
# Politique OP_RETURN facon "avant v30" (Bitcoin Core >= 30)
datacarrier=1
datacarriersize=83
# Pas de relais des sorties multisig nues (bare multisig)
permitbaremultisig=0
```

- [ ] **le futur** : personne ne le connait ! Après la guerre des blocs, les ordinals, la crise du spam … sur qu'il y en aura bien d'autres … comme les Bitcoin de Satoshi et le quantique. Quoi qu'il en soit il est sain que des frictions existent. La résilience d'un réseau c'est aussi sa diversification, avant cette crise du spam seulement 1 à 2 % des [nœuds du réseau](https://coin.dance/nodes) n'utilisaient pas Bitcoin Core, mi 2026 il y a eu jusqu'à 20 % de nœuds Bitcoin Knots sur le réseau. Et justement cette opposition a contribué à ce que Core renonce à déprécier `datacarrier` et `datacarriersize` juste avant la sortie de la v30 ! Mais tout ceci est de la gouvernance, les divergences rendent cela vivant ! Faites vos propres recherches, forgez-vous votre propre avis puis participez !

Le terme "Bitcoin Core" associé aux "Core Developers", peut sembler tendancieux ou chargé d'une connotation centralisatrice, comme si cette implémentation et ce groupe de développeurs étaient le "cœur" officiel et incontestable de Bitcoin, au détriment d'autres visions. Le renommage du logiciel Bitcoin en "Bitcoin Core" date de 2014. La raison officielle avancée par les développeurs de l'époque était de distinguer clairement l'implémentation logicielle du réseau Bitcoin lui-même, autrement dit pour éviter la confusion entre le protocole/réseau Bitcoin décentralisé et sans propriétaire, et son implémentation logicielle qui peut varier.

### Options de compilation

`gui` ou `bitcoin-qt (GUI)` signifie Graphical User Interface et `qt` est un framework de développement multi-plateforme pour la création d'applications et d'interfaces utilisateur graphiques avec C++. Nécessite une interface graphique, à tester c'est sympa, mais off dans ce cas d'usage.

`zmq` ou `ZeroMQ` est une interface de notification qui permet à des applications externes de recevoir des mises à jour en temps réel sur les événements du réseau Bitcoin. Zéro c'est pour 0 intermédiaire, 0 latence, 0 coût, 0 administration. Cool, zéro admin :) MQ c'est pour "Message Queue" ou file de messages. Activez, peut être utile par la suite.

A propos du terme USDT, cela n'a rien à voir avec les "stablecoins", c'est utilisé pour activer les traces utilisateur statiquement définies, ce sont des outils de traçage pour l'analyse des performances et le débogage. C'est très intéressant mais hors du "scope" ici.

`IPC` ou architecture multiprocess : depuis Bitcoin Core v30.0 (octobre 2025) l'option de compilation `ENABLE_IPC` est activée par défaut (sauf sous Windows). Elle construit le binaire `bitcoin-node` qui permet de découper le nœud en plusieurs processus communiquant entre eux par IPC (Inter-Process Communication) avec la bibliothèque Cap'n Proto. Son premier usage est une interface de minage expérimentale destinée à Stratum v2 (`bitcoin -m node -ipcbind=unix`) : un logiciel de pool ou de mineur externe vient y chercher ses modèles de bloc. En v29 l'équivalent `WITH_MULTIPROCESS` existait déjà, mais expérimental et désactivé par défaut. Le nœud décrit ici ne mine pas, il sert un portefeuille et Electrs : l'IPC est inutile, d'où le flag `-DENABLE_IPC=OFF`. Sans ce flag et sans les paquets Cap'n Proto, `cmake` s'arrête en erreur à partir de la v30. Si vous choisissez l'architecture multiprocess (par exemple pour miner en Stratum v2 avec votre nœud), installez `sudo apt-get install libcapnp-dev capnproto` et n'utilisez pas ce flag.

Il existe d'autres options de compilation non décrites ici, non pertinentes pour l'usage visé ici.

### Avant la v29

Ci-dessous je choisis la  v28.0 sans les options de test de débogage et de performance :

```bash
# Valable si la version construite precede la v29
cd ~/code/bitcoin
git tag                    # liste les versions disponibles
git checkout tags/v28.0    # pour la la v28 Core

# Si reconstruction, faire un clean prealable
#make clean

./autogen.sh
./configure --disable-tests --disable-fuzz-binary --disable-bench --disable-usdt
```

Relancer `./configure` tant que vous n'obtenez pas ce que vous désirez, pour de l'aide faites `./configure --help` ou `./configure --help | grep -A1 "je cherche ce terme"`

Sortie Autotools que j'obtiens :

```ini
Options used to compile and link:
  external signer = yes
  multiprocess    = no
  with wallet     = yes
    with sqlite   = yes
    with bdb      = no
  with gui / qt   = no
  with zmq        = yes
  with test       = no
  with fuzz binary = no
  with bench      = no
  with upnp       = no
  with natpmp     = no
  USDT tracing    = no
  sanitizers      =
  debug enabled   = no
  werror          = no

  target os       = linux-gnu
  build os        = linux-gnu
```

Si vous avez paramétré une option mais qu'à l'arrivée elle est absente, regardez si vous n'avez pas une dépendance manquante. Pour vérifier si c'est le cas effectuer ceci avec les logs générés par `configure` : `grep -i <option_particuliere> config.log` comme par exemple `grep -i bench config.log`

Construction et installation du binaire :

```bash
make                # Compilation
sudo make install   # Installation des binaires, par défaut dans /usr/local/bin
```

La compilation (la ligne make) dure 1 à 2 heures sur un Raspberry Pi 5 / NVMe.M2 en fonction des options (si tests, fuzz binary et bench sont actives cela rallonge le temps de compilation) et environ 20mn sur le Dell Optiplex 5050. Pendant ce temps vous pouvez ouvrir un nouveau terminal et installer Tor, I2P … ou continuer à flâner dans le code source ou la documentation de Bitcoin. N'oubliez pas de taper la deuxième ligne !

### A partir de la v29

Le système de **build Bitcoin a migré** d'Autotools vers CMake.

Si CMake n'est pas installé, effectuer `sudo apt-get install cmake`

```bash
cd ~/code/bitcoin
git tag                                  # liste les versions disponibles
git checkout tags/v31.1                  # Octobre 2026 : v31.1 Core (voir Choix du code)

# Verifier la signature du tag AVANT de compiler (indispensable)
# Les cles des mainteneurs sont publiees dans un depot distinct du code : guix.sigs
git clone https://github.com/bitcoin-core/guix.sigs ~/code/guix.sigs
gpg --import ~/code/guix.sigs/builder-keys/*.gpg
git verify-tag v31.1
# Sortie attendue pour la v31.1, signee par Michael Ford (fanquake) :
#   Good signature from "Michael Ford (bitcoin-otc) <fanquake@gmail.com>"
#   Primary key fingerprint: E777 299F C265 DD04 7930  70EB 944D 35F9 AC3D B76A
# (la v30.3 est signee par Ava Chow : 1528 1230 0785 C964 44D3  334D 1756 5732 E08E 5E41)
# L'avertissement "This key is not certified with a trusted signature" est normal.
# "BAD signature" ou "No public key" : NE PAS compiler, chercher pourquoi.

mkdir build                              # Creation du repertoire build
# ou si reconstruction, supprimer tout ce qui est dans build
# rm -rf ~/code/bitcoin/build/*

cmake -B build -LH # liste complete et à jour des options disponibles avec leurs valeurs par defaut
cmake -B build -DWITH_ZMQ=ON -DBUILD_TESTS=OFF -DENABLE_IPC=OFF # Selection des options avant compilation
```

Sortie CMake que j'obtiens (ici avec Knots v29.2, les lignes varient un peu selon la version) :

```ini
Configure summary 
Executables: 
  bitcoind ............................ ON 
  bitcoin-qt (GUI) .................... OFF 
  bitcoin-cli ......................... ON 
  libbitcoinconsensus ................. OFF 
  bitcoin-tx .......................... ON 
  bitcoin-util ........................ ON 
  bitcoin-wallet ...................... ON 
  bitcoin-chainstate (experimental) ... OFF 
  libbitcoinkernel (experimental) ..... OFF 
Optional features: 
  wallet support ...................... ON 
   - descriptor wallets (SQLite) ...... ON 
   - legacy wallets (Berkeley DB) ..... OFF 
  external signer ..................... ON 
  tor subprocess ...................... ON 
  port mapping using UPnP ............. OFF 
  ZeroMQ .............................. ON 
  USDT tracing ........................ OFF 
  QR code (GUI) ....................... OFF 
  DBus (GUI) .......................... OFF 
Tests: 
  test_bitcoin ........................ OFF 
  test_bitcoin-qt ..................... OFF 
  bench_bitcoin ....................... OFF 
  fuzz binary ......................... OFF
```

Construction et installation du binaire :

```bash
cmake --build build -j $(nproc) # compile en utilisant tous les coeurs CPU
# duree du build sur les 40 cores un Xeon E5-2698 v4 : 109s !

sudo cmake --install build      # installation 
```

### Binaires installés

* `bitcoin-cli` client RPC qui permet d'interagir avec `bitcoind` via des commandes
* `bitcoin-tx` utilitaire pour créer, modifier, signer et décoder des transactions brutes. Mode offline possible.
* `bitcoin-util` utilitaire pour développeur. Comme effectuer des tests pour des tâches Bitcoin qui ne nécessitent pas un nœud en cours d'exécution. Utilisé principalement pour les opérateurs/maintainers de Signet un réseau de test Bitcoin plus stable et fiable que le traditionnel testnet.
* `bitcoin-wallet` outil pour gérer les fichiers wallet offline, opérations wallet sécurisées hors node.
* `bitcoind` daemon principal qui exécute un nœud Bitcoin

## **Installer Tor et I2P**

Ces deux réseaux se superposent à internet et apportent une couche d'anonymisation. Tous deux procurent un espace de liberté et de souveraineté individuelles. Par ailleurs, ils procurent à Bitcoin une résilience et une résistance à la censure accrues par complémentarité avec clearnet (IPV4 et 6). Commençons par Tor:

```bash
sudo apt-get install tor
```

Editer le fichier de configuration de Tor par :

```bash
sudo nano /etc/tor/torrc
```

```bash
#Dé-commenter les deux lignes suivantes:
ControlPort 9051
CookieAuthentication 1

#Rajouter les deux lignes suivantes:
CookieAuthFileGroupReadable 1
DataDirectoryGroupReadable 1
```

Re-démarrer Tor par:

```bash
sudo systemctl restart tor
```

Détecter le groupe Tor utilisé dans la distribution :

```bash
sudo ls -al /run/tor/control.authcookie
```

Pour la Debian la réponse est "debian-tor", et dans mon cas c'est l'utilisateur 'btc-node' (que j'ai créé à l'installation de ma distribution) qui lancera le programme `bitcoind`, cela donne pour moi :

```bash
sudo usermod -a -G debian-tor btc-node
```

Se déconnecter puis se reconnecter du système par :

```bash
exit
ssh nom_utilisateur@IP
```

Passons à I2P

```bash
sudo apt-get install i2pd
```

Pour plus tard, si vous voulez voir les logs I2P :

```bash
sudo tail -f /var/log/i2pd/i2pd.log
```

Les adresses `.onion` et `.i2p` attribuées à `bitcoind` sont persistantes tant que les fichiers clés restent dans le 'datadir' `~/.bitcoin`, il s'agit de `onion_v3_private_key` pour Tor et `i2p_private_key` pour I2P. Si ces fichiers sont supprimés, de nouvelles adresses sont générées. Particularité pour I2P, si `-i2pacceptincoming=0` est défini ainsi dans `bitcoin.conf`, `bitcoind` n'accepte plus les connexions entrantes I2P et utilise une adresse I2P transitoire différente à chaque connexion sortante, ceci afin de limiter la corrélation et le fingerprinting.

CJDNS est une autre solution d'anonymisation supportée par Bitcoin, sa mise en oeuvre est relativement compliquée et elle me semble moins utilisée que Tor ou I2P, je l'ai explorée, je reviendrai mettre à jour ici si je trouve que cela est concluant.

## **SSH et la sécurité**

Votre futur nœud \[Bitcoin\] est connecté au réseau internet 7j/7, les mots de passe ne sont pas d'une sécurité absolue et sont pénibles à l'usage. Pour se passer du mot de passe et améliorer la sécurité prenez le temps d'installer une clé d'authentification sur le ou les postes clients de votre réseau local devant avoir un accès Secure Shell (ssh) à votre nœud. Tout est basé sur la cryptographie asymétrique, la clé publique sera sur \[Bitcoin\] et la clé privée sur \[PC\].

\[PC\] désigne le poste client (pour moi c'est Linux avec interface graphique)

\[PC\] lancer un terminal et ouvrir une session sur votre nœud par `ssh username@IP`

\[Bitcoin\] si le répertoire `.ssh` est non présent enchaîner `mkdir ~/.ssh` puis `chmod 700 ~/.ssh`

\[Bitcoin\] saisir : `nano ~/.ssh/authorized_keys` qui contiendra la clé publique, laissez ouvert.

\[PC\] ouvrir un 2ème terminal pour générer des clés par `ssh-keygen -t rsa` ? mais attendez … quelqu'un me souffle dans l'oreille, utilise plutôt `ssh-keygen -t ed25519` c'est plus moderne. (ok … cela change juste dans le texte qui suit `_rsa` en `_ed25519`)

* sauvegarder la paire de clés dans le home de l'utilisateur, saisir `/home/[PC]_user/id_ed25519`
* par compromis sécurité/praticité la passphrase sera vide, taper `Enter` 2 fois

\[PC\] visualiser le contenu de la clé publique `cat id_ed25519.pub`

\[PC\] copier intégralement la sortie

*Tip : pour copier coller du texte d'un terminal Linux à un autre terminal Linux, sélectionnez le simplement à la souris dans le 1er, positionnez le curseur de la souris dans le 2ème puis faites un click molette.*

\[Bitcoin\] coller le contenu dans `~/.ssh/authorized_keys` précédemment ouvert, enregistrer, quitter nano, faire `chmod 600 ~/.ssh/authorized_keys`. Ne pas fermer la session.

\[Bitcoin\] Faites : `sudo nano /etc/ssh/sshd_config`

Lisez puis rajoutez ceci à la fin du fichier de configuration de ssh

```bash
# Acces SSH avec cle d'authentification et sans mot de passe
#
# Si les parametres definis ci-dessous sont presents plus haut dans le fichier
# de configuration, alors commentez les !

# ChallengeResponseAuthentication no # Pour Debian 11 avec OpenSSH 8.4
KbdInteractiveAuthentication no      # A partir de Debian 12 et d'OpenSSH 8.7

PasswordAuthentication no
UsePAM no
```

\[Bitcoin\] sauvegarder et quitter l'éditeur nano, toujours laisser la session ouverte.

\[Bitcoin\] redémarrer le service ssh par `sudo service ssh reload`

\[PC\] ouvrir un nouveau terminal pour se connecter à \[Bitcoin\] par `ssh username@IP`

\[PC\] vérifier qu'il est impossible de se connecter à \[Bitcoin\]

\[PC\] Déplacer les clés dans `~/.ssh` par :

```bash
mv ~/id_ed25519.pub ~/.ssh/id_ed25519.pub
mv ~/id_ed25519 ~/.ssh/id_ed25519
```

\[PC\] se re-connecter à \[Bitcoin\] par `ssh username@IP`, cela doit fonctionner sans mot de passe.

En cas d'échec vous avez toujours accès à \[Bitcoin\] par l'ouverture de la première session pour remédier au problème ... Si c'est OK, fermer tous les terminaux par 'exit'.

\[PC\] S'il est nécessaire d'effacer un "host" en particulier dans `.ssh/known_hosts`, la commande est : `ssh-keygen -R <hostname>` , `hostname` désigne une IP ou un nom de machine sur le réseau local.

\[PC\] avec ssh vous pouvez accéder à de multiples hôtes à condition d'utiliser la même clé d'authentification. Si vous devez utiliser des clés différentes, il faudra définir un fichier `config` dans `~/.ssh` répertoriant chaque hôte / clé.

## Séparer les données Bitcoin de l'OS

**A effectuer seulement si vous avez 2 unités de stockage distinctes.**

Les données blockchain d'un nœud complet sont imposantes (650 Gio en octobre 2024), si vous ne pouvez plus mettre à jour l'Operating System ou rencontrez un problème avec celui-ci ou avec la machine elle-même, il est judicieux de séparer les données du nœud Bitcoin du reste.

Si le deuxième stockage de masse n'est pas déjà monté au démarrage de la machine, utiliser la commande `lsblk -o +PTTYPE,MODEL` pour obtenir le "NAME" de tous les périphériques branchés, si c'est partitionné "TYPE" indiquera `part` , s'ils sont montés ce sera indiqué à la colonne "MOUNTPOINTS", "PTTYPE" indique le type de table de partition dos ou gpt :

* l'indicateur `dos` reporté par `lsblk` veut dire MBR (Master Boot Record), la taille de la partition sera limitée à 2 To, et c'est 4 partitions maximum, outil `fdisk`.
* `gpt` c'est GPT (GUID Partition Table), les partitions peuvent excéder 2 To, et il est possible de créer jusqu'à 128 partitions, outil `gdisk` ou `parted`.
* pour résumer, préférez GPT pour la table des partitions.

**/!\\ vérifiez bien ce que vous faites et adaptez en fonction de votre cas /!\\**

Si `gdisk` pas installé : `sudo apt-get update` suivi de `sudo apt-get install gdisk`

Sur mon Dell Optiplex c'est `nvme0n1` de 2To, il n'est pas monté au démarrage, c'est donc :

`sudo gdisk /dev/nvme0n1` pour créer la table des partitions sur `nvme0n1`, puis saisir :

* `p` pour information et ainsi vérifier que c'est le bon périphérique
* `n` pour créer une nouvelle partition, entrée par défaut.
* Premier secteur, entrée par défaut.
* Dernier secteur, entrée par défaut pour tout l'espace restant.
* Type de partition, entrée pour "Linux filesystem" par défaut.
* `p` pour vérifier la partition avant de l'écrire
* `w` pour écrire les changements et `Y` pour confirmer.

**Si la machine est limitée**, formater en `ext4` comme ceci :

```bash
# Obtenir le nom de la nouvelle partition (pour moi la reponse est 'nvme0n1p1')
lsblk -o +PTTYPE

# Créer le systeme de fichier, ici ext4
sudo mkfs -t ext4 /dev/nvme0n1p1

# Facultatif : donner un Label à la partition, maxi 16 caractères
sudo e2label /dev/nvme0n1p1 Bitcoin-Part

# Créer un point de montage, j'ai choisi 'nvme' dans mnt
sudo mkdir /mnt/nvme

# D'abord monter le périphérique
sudo mount /dev/nvme0n1p1 /mnt/nvme

# Ensuite donner les droits à l'utilisateur qui va lancer le noeud
# Desormais cette partition appartiendra a btc-node
sudo chown -R btc-node:btc-node /mnt/nvme

# Creer le repertoire bitcoin
mkdir /mnt/nvme/bitcoin

# Verifier l'utilisateur, le groupe et les droits d'acces
ls -al /mnt/nvme

# Identifier le peripherique par
sudo blkid

# Editer fstab par :
sudo nano /etc/fstab

# Puis rajouter à la fin de fstab ces 2 lignes ci-dessous  :

# Stockage de masse dedie a Bitcoin
UUID=uuid-device /mnt/nvme   ext4    defaults        0       2
```

**Si la machine est plus véloce et que le périphérique est SSD ou NVMe** vous pouvez utiliser BTRFS (B-tree File System), comme ceci :

```bash
# Si besoin est, installer BTRFS
sudo apt-get install btrfs-progs

# Obtenir le nom de la nouvelle partition (pour moi la reponse est 'nvme0n1p1')
lsblk -o +PTTYPE

# Creer le systeme de fichier, ici BTRFS (-f force si besoin)
sudo mkfs.btrfs -f --metadata single --data single /dev/nvme0n1p1

# Créer un point de montage, j'ai choisi 'btrfs' dans mnt
sudo mkdir /mnt/btrfs

# Monter temporairement le périphérique
sudo mount /dev/nvme0n1p1 /mnt/btrfs

# Creer des subvolumes qui permettront de separer logiquement les donnees
# et de faire des snapshots independants.
sudo btrfs subvolume create /mnt/btrfs/@bitcoin      # la blockchain bitcoin
sudo btrfs subvolume create /mnt/btrfs/@electrs      # les donnees d'Electrum
sudo btrfs subvolume create /mnt/btrfs/@snapshots    # les snapshots

# Demonter
sudo umount /mnt/btrfs

# Creer les sous-repertoires dans l'arborescence (montages a venir)
sudo mkdir /mnt/btrfs/bitcoin
sudo mkdir /mnt/btrfs/electrs
sudo mkdir /mnt/btrfs/snapshots

# Recuperer l'UUID du peripherique
sudo blkid /dev/nvme0n1p1

# Editer fstab par :
sudo nano /etc/fstab

# Puis rajouter à la fin de fstab les 7 lignes ci-dessous  :

# Montage des subvolumes BTRFS. Sur un NVMe le niveau de compression 3 standard
# est un bon compromis entre gain de place / performances / ressources CPU
# Un point de montage distinct pour chaque subvolume permet des options de montage
# differentes si besoin est par la suite
UUID=uuid-device   /mnt/btrfs/bitcoin   btrfs   defaults,noatime,compress=zstd:3,subvol=@bitcoin   0   2
UUID=uuid-device   /mnt/btrfs/electrs   btrfs   defaults,noatime,compress=zstd:3,subvol=@electrs   0   2
UUID=uuid-device   /mnt/btrfs/snapshots btrfs   defaults,noatime,compress=zstd:3,subvol=@snapshots   0   2

# Tester le montage
sudo mount -a
sudo systemctl daemon-reload

# Ensuite donner les droits à l'utilisateur qui va lancer le noeud
sudo chown -R btc-node:btc-node /mnt/btrfs/bitcoin
sudo chown -R btc-node:btc-node /mnt/btrfs/electrs
sudo chown -R btc-node:btc-node /mnt/btrfs/snapshots

# Verifier l'utilisateur, le groupe et les droits d'acces
ls -al /mnt/btrfs

# Voici l'equivalent de disk free (df) avec les particularites de BTRFS
sudo btrfs filesystem df /mnt/btrfs/bitcoin
```

Voir le système de fichier : `lsblk -o +FSTYPE`

**Inconvénients de BTRFS sur Ext4** : consomme des ressources CPU et de la Ram. Est moins optimal sur les disques durs mécaniques qu'il ne l'est sur les non mécaniques.

**Avantages de BTRFS sur Ext4** : très bien adapté au stockage de masse hors rotatif comme les SSD et les NVMe. BTRFS intègre une fonctionnalité très puissante, les snapshots : ils sont très rapides et économes en espace car seul les changements sont stockés ce qui est parfait pour des backups avec snapshot read-only ou des tests avec snapshot writable (modifier la copie sans affecter l'original grâce au Copy-on-Write). Effectuer un snapshot de toute la blockchain Bitcoin peut s'avérer utile lors d'une mise à jour de version de `bitcoind`. Autres avantages : compression transparente, checksums pour détecter les corruptions, subvolumes, optimisé pour SSD / NVMe avec maintien des performances et prolongation de la durée de vie. Avec le kernel 6.12 de Debian 13 (Trixie) toutes les options SSD / NVMe utiles pour l'usage d'un nœud Bitcoin sont activées par défaut sauf `noatime` (Debian monte les BTRFS avec `relatime` par défaut). Pour vérifier ces options, faire `mount | grep btrfs`.

Redémarrer la machine par `sudo shutdown -r now` et vérifier que le périphérique est bien monté automatiquement.

Pour finir, si le répertoire `~/.bitcoin` est présent vérifiez qu'il soit vide, ensuite supprimez le. Créez un lien symbolique appelé `.bitcoin` dans le home de l'utilisateur qui lancera le nœud Bitcoin, et qui pointera vers votre unité de stockage dédié à cela.

```bash
# Adaptez le chemin à votre peripherique
ln -s /mnt/nvme/bitcoin ~/.bitcoin          # pour ext4
ou
ln -s /mnt/btrfs/bitcoin ~/.bitcoin         # pour BTRFS
```

## **Paramétrage de Bitcoin**

`bitcoin.conf` est le fichier de configuration de `bitcoind` le logiciel nœud Bitcoin, remarquez que sans fichier de configuration cela fonctionne, mais lisez bien la suite …

`bitcoind` le d à la fin est pour "daemon", un démon logiciel est un processus qui tourne en arrière-plan. Pour l'arrêter tapez "CTRL-C" dans la console quand il n'est pas explicitement lancé en mode démon. S'il est en mode démon utiliser le programme `bitcoin-cli` (acronyme de Bitcoin Command Line Interface) qui communique avec lui, la commande est `bitcoin-cli stop` .

**Sachez que dès son lancement**, vu que vous avez zéro bloc dans votre nœud, `bitcoind` va **obsessionnellement** aller les chercher auprès d'autres nœuds sans utiliser de surcouche d'anonymisation. Donc effectuez ce qui suit puis choisissez.

Créer le répertoire `.bitcoin` qui va héberger le fichier de configuration :

```bash
mkdir ~/.bitcoin # oubliez cette ligne si vous avez une unite de stockage separee et dediee a la blockchain
touch ~/.bitcoin/debug.log # pour etre en mesure de suivre les logs des le lancement de bitcoind
nano ~/.bitcoin/bitcoin.conf
```

Copier / coller ceci dans le fichier `bitcoin.conf` :

```bash
# Configuration de Bitcoin (Core ou Knots)
# ~/.bitcoin/bitcoin.conf
#
# Pour plus d'informations visitez 
# https://jlopp.github.io/bitcoin-core-config-generator/

# Pour 'bitcoin-qt' (version GUI - hors propos ici):le noeud acceptera les connexions
# commandes RPC (Remote Procedure Call) et notamment les commandes de "bitcoin-cli"
# Pour 'bitcoind' : bien que non necessaire, laisser ainsi egalement pour bitcoind 
server=1 

# bitcoind est lance en tache de fond comme un demon et accepte les commandes
# Avec ce parametre a 1 pour le stopper utiliser la commande 'bitcoin-cli stop'
daemon=1

# Nombre de threads de verification des scripts (signatures).
# par=1 aucun parallelisme
# par=0 automatique, le bon choix la plupart du temps.
# par < 0 laisse n coeur(s) libre(s). Ex par=-1 laisse 1 coeur libre
# La commande 'nproc' retourne le nombre de coeurs logiques de votre machine.
# Le parallelisme est TRES utile pendant l'IBD, et encore plus avec assumevalid=0.
par=0

# Force le noeud a valider chaque transaction et bloc a partir du bloc Genesis.
# La suppression de cette optimisation rallonge le telechargement initial (IBD)
# de la blockchain sur les machines peu veloces. Choix strict et conservateur. 
assumevalid=0

# Elague la blockchain au fur et a mesure afin de reduire la place occupee.
# Vous perdez les transactions les plus anciennes afin de limiter ici à 550 Mo
# de fichiers de blocs 'blocks' auquels il faut ajouter le 'chainstate' avec un peu plus
# de 10 Gio, soit un total environ 15Gio.
# Apres activation il n'est pas possible de revenir en arriere, il faudra
# a nouveau telecharger la totalite de la blockchain.
#prune=550

# Taille du cache en RAM de l'ensemble des UTXO en Mio. Default is 450
# Avant modifier le default, verifiez la memoire vive disponible avec "free -mh"
# Pour une synchronisation plus rapide, definissez le parametre en fonction de la
# memoire disponible. Par exemple, avec 8 Gio de memoire, quelque chose comme
# 'dbcache= 4000' fait sens.
# Contrepartie : l'arret du noeud prend plus de temps, car tout le cache est ecrit sur
# le disque et apres un crash, il faut recalculer plus de données au redemarrage.
# Pour une utilisation reduite de la memoire, ce parametre peut etre diminue ou voire
# supprime une fois que la synchronisation initiale (IBD) est terminee.
dbcache=4000

# Plafond en Mo de la mempool en RAM. Default is 300.
# Avant modifier le default, verifiez la memoire vive disponible avec "free -mh"
# A 800 le noeud garde plus de transactions en attente quand le reseau est
# congestionne. Moins de transactions a frais bas sont supprimees.
# L'estimation des frais sera meilleure et si des applications comme mempool.space
# sont installees sur le noeud, elles auront une vue plus complete.
# Note : pendant l'IBD, la part inutilisee de la mempool sert de cache supplementaire
# a dbcache.
maxmempool=800

# /!\ Brouillage de la blockchain /!\
# Depuis Bitcoin Core 28, les fichiers de blocs (blk*.dat) sont brouilles par defaut
# avec une clef XOR aleatoire (default blocksxor=1). Le but est d'eviter qu'un
# antivirus supprime ces fichiers parce qu'il y trouve des donnees suspectes inscrites
# dans la blockchain !!! Comme du spam ?
# Une blockchain sans brouillage peut etre lue directement avec des outils simples,
# pour moi ce sera sans brouillage.
# /!\ Desactiver /!\ avant le premier lancement de bitcoind puisque l'option n'agit
# qu'a la creation des fichiers de blocs. On ne peut pas passer à 0 sur des blocs
# deja brouilles sans tout resynchroniser !!!
# Pas brouillee = "od -An -tx1 ~\.bitcoin/blocks/xor.dat" => " 00 00 00 00 00 00 00 00"
blocksxor=0

# Index de transaction etendu optionnel, cela prend un peu plus d'espace.
# Requis pour les explorateurs self-hosted comme btc-rpc-explorer et mempool
# Cela permet egalement d'activer certaines fonctionnalites pour les "wallets"
# se connectant par RPC directement a bitcoind comme la recherche
# des entrees sur les transactions.
# Si vous ne voulez pas de ces fonctionnalites dans un premier temps vous
# pouvez commenter, il est possible d'activer par la suite, bitcoind
# construira l'index la ou il s'est arrete.
# Non compatible avec les noeuds elagues (Pruned nodes)
# Electrs n'en a pas besoin.
txindex=1

# Politique OP_RETURN facon "avant v30" (Bitcoin Core >= 30, voir Choix du code)
# 83 octets de script au total pour toutes les sorties OP_RETURN d'une transaction,
# soit 80 octets de donnees utiles. Ce n'est pas du consensus : uniquement ce que
# votre noeud accepte dans sa mempool et relaie.
datacarrier=1
datacarriersize=83
# Pas de relais des sorties multisig nues (bare multisig) utilisees pour stocker
# des donnees dans l'ensemble des UTXO
permitbaremultisig=0

# Serveur REST (lecture seule, sans authentification) sur les memes adresses que
# le RPC. Indispensable pour Electrs v0.12 et plus.
rest=1

# Si listen=0 les autres noeuds ne peuvent pas se connecter au votre. Vous ne servez
# donc pas de source de blocs pour les noeuds en IBD, et vous ne fournissez aucun
# creneau entrant aux noeuds qui cherchent des pairs. Cela de-active 'listeonion' et
# la découverte d'adresses 'discover', vous n'aurez donc plus d'adresses Tor et I2P
# entrantes. Vos transactions sont neamoins toujours diffusees puisque les connexions
# sont bi-directionnelles.   
# Si listen=1 accepte les connexions entrantes avec d'autres pairs : vous permettez aux
# autres noeuds d'acceder au votre. 
listen=1

# Options debug, il y en a tout un tas, vous pouvez rendre bavard
# bitcoind dans differents domaines, soit au lancement en ligne de commande par
# 'bitcoind -debug=tor -debug=i2p' ou ci-dessous dans les reseaux en de-commentant
# Une seule categorie par ligne, sinon bitcoind refuse de demarrer
#debug=net
#debug=i2p
#debug=tor

# Parametres du journal de debogage ou "Debug log settings"
# Commentez si vous voulez voir tous les logs au debut, c'est interressant :)
# Ensuite de-commentez car cela peut occuper beaucoup de place
shrinkdebugfile=1

# Si IPV4 est inactif vous pouvez laisser le commentaire de la ligne rpcallowip,
# sinon de-commentez pour limiter l'acces RPC bitcoind a votre reseau local (LAN)
# afin de ne pas l'exposer a des reseaux non fiables tels que l'internet (WAN).
# Adaptez l'adresse par celle de votre reseau local, exemple : 192.168.0.0/24
# autorise 256 adresses de (192.168.0.0 à 192.168.0.255)
# rpcallowip=192.168.X.0/24

# de-active le wallet de bitcoind
#disablewallet=1

# Ouverture automatique du port 8333 sur votre box : perso, je n'aime pas
# Depuis Core v30 (et dans Knots 29.x) natpmp=1 est le defaut : sans proxy Tor,
# par exemple pendant un IBD en clearnet, la box ouvrirait le port 8333 et
# exposerait votre IP. Ligne a laisser de-commentee.
natpmp=0
# Si version ≤ 28.x / de-activer UPnP
#upnp=0


############################################################
# IPV4 inactif / TOR et I2P active / Confidentialite avancee
############################################################

# Actif par defaut : demande d'adresses de pairs par consultation de la graine
# DNS que si le nombre d'adresses est insuffisant.
# Mettre 0 pour confidentialite avancee
dnsseed=0

# Actif par defaut, autoriser les recherches DNS pour les valeurs -addnode,
# -seednode et -connect.
# Mettre 0 pour confidentialite avancee
dns=0

# bitcoind se lie a l'adresse donnee et l'ecoute toujours
# si c'est '127.0.0.1' (localhost) bitcoind sera dans l'impossiblite
# d'ecouter sur les adresses 'Clearnet'
bind=127.0.0.1

# TOR
proxy=127.0.0.1:9050
onlynet=onion

# I2P
i2psam=127.0.0.1:7656
onlynet=i2p

# Fixe le total des connexions P2P (entrantes + sortantes). Default is 125.
# Chaque connexion Tor ou I2P coute plus cher qu'en clearnet (chiffrement et
# demons TOR/I2P)
# La bande passante disponible sur ces réseaux est faible et la latence elevee.
# Beaucoup de pairs entrants ne rendent pas le noeud plus utile, mais le ralentissent.
# 40 est un bon choix ici.
maxconnections=40

# Si vous connaissez des noeuds de confiance, les ajouter ici renforce votre
# protection contre les attaques par eclipse. Le fait d'echanger avec d'autres noeuds
# à la fois sur TOR ET I2P et deja une bonne protection. 
# Les connexions addnode ont leur propre limite, jusqu'à 8 en plus, et ne comptent pas
# dans le total de maxconnections.
# Le noeud se reconnectera automatiquement a ces pairs s'ils se deconnectent.
# addnode=xxx.onion
# addnode=xxx.b32.i2p
```

Enregistrez puis réglez les permissions par `chmod 600 ~/.bitcoin/bitcoin.conf`

Pour une aide exhaustive sur toutes les commandes : `bitcoind -help`

Lisez attentivement votre fichier `bitcoin.conf`, ensuite si vous souhaitez télécharger la blockchain assez rapidement (de quelques heures à 3 jours en fonction de votre débit internet), vous pouvez faire l'IBD sur clearnet. Votre fournisseur d'accès et vos pairs verront alors que votre IP fait tourner un nœud Bitcoin pendant ce temps. **Mais attention à ne pas relier durablement votre IP à vos futures adresses Tor et I2P** : avec `listen=1`, même sans les lignes Tor et I2P, `bitcoind` crée quand même son adresse `.onion` par le port de contrôle de Tor (`listenonion`). Votre nœud est alors joignable à la fois par votre IP et par cette adresse `.onion`, et un observateur qui se connecte aux deux peut les corréler (mêmes réponses, même version, même durée de fonctionnement …). La documentation de Bitcoin Core le dit : un adversaire qui peut se connecter à votre nœud sur plusieurs réseaux peut être capable de corréler ces identités. Comme la clé `onion_v3_private_key` est conservée, ce lien survivrait au passage à Tor/I2P. Procédez donc ainsi :


1. Pendant l'IBD sur clearnet : commentez toutes les lignes après "IPV4 inactif / TOR et I2P active / Confidentialité avancée" et mettez `listen=0` (votre nœud ne fait que des connexions sortantes, ce qui suffit pour télécharger, sans adresse `.onion` ni port ouvert). Gardez `natpmp=0`.
2. Lorsque c'est synchronisé avec les autres nœuds, stoppez `bitcoind`, remettez `listen=1`, dé-commentez les lignes précédemment commentées.
3. Par précaution, si une adresse a pu être créée pendant la phase clearnet (par exemple si vous aviez déjà lancé `bitcoind` avec `listen=1`), supprimez les fichiers d'identité et de pairs d'ancrage, de nouvelles adresses seront générées : `rm -f ~/.bitcoin/onion_v3_private_key ~/.bitcoin/i2p_private_key ~/.bitcoin/anchors.dat`
4. Relancez `bitcoind`.

Si vous tenez à votre confidentialité dès le départ, laissez comme c'est, le téléchargement de la blockchain sera plus long, cela peut prendre jusqu'à une semaine.

## Lancement de `bitcoind`

Visualisez les logs dans un premier terminal :

```bash
tail -f ~/.bitcoin/debug.log
```

Dans un deuxième terminal, lancez votre nœud avec le debug de TOR et I2P si besoin :

```bash
bitcoind -debug=tor -debug=i2p
```

Observer les logs de quelques minutes à plusieurs heures étalées, documentez-vous et décodez-les, c'est intéressant. Vérifiez que votre nœud télécharge les blocs.

Pour arrêter : `bitcoin-cli stop` et attendre l'apparition dans les log de `Shutdown: done` (cela prend parfois plusieurs minutes). Relancez `bitcoind`, il reprendra de l'endroit où il s'était arrêté.

Savoir où en est votre nœud : `bitcoin-cli getblockchaininfo`

* `blocks` : Nombre de blocs validés par le nœud. (1 bloc = jusqu'à environ 4 Mo)
* `headers` : Nombre d'entêtes de blocs connues par le nœud. ( 1 header = 80 octets)
* `initialblockdownload` : téléchargement initial des blocs, vrai ou faux
* Le nœud est synchronisé si `blocks == headers` et que `initialblockdownload = false`

Informations réseau : `bitcoin-cli getnetworkinfo | less`

Voir le nombre d'adresses connues par votre nœud : `bitcoin-cli -addrinfo`

Évaluer le trafic réseau : `bitcoin-cli getnettotals`

Depuis Bitcoin v0.21, aucun portefeuille n'est créé par défaut. A ce stade ne pas s'en soucier.

## Détail des connexions réseau

Voici un aperçu des pairs entrants et sortants après plusieurs jours de fonctionnement sans interruption. Ne soyez pas pressés, cela prend du temps.

Tapez dans le shell `bitcoin-cli -netinfo 4`, ici la sortie de Bitcoin Core 27.1.0 :

```bash
Bitcoin Core client v27.1 - server 70016/Satoshi:27.1.0/

<->   type   net  v  mping   ping send recv  txn  blk  hb addrp addrl  age    id address                        version
 in          npr  1    156    221   11   11    *              .        261 18175 127.0.0.1:48326                70016
 in          npr  2    177    233    2    2                  37         43 18445 127.0.0.1:60608                70016/Satoshi:27.1.0/
 in          npr  1    187    648    5    5    *        *     .       2395 15748 127.0.0.1:54910                70016/Satoshi:25.0.0/
 in          npr  1    196    371    2    4  240           3982       2016 16151 127.0.0.1:60686                70016/Satoshi:26.0.0/
 in          npr  2    229    349    2    2   46   38  .   1227        638 17732 127.0.0.1:46094                70016/Satoshi:27.0.0/
 in          npr  1    245    319   49   49    *  338         .       1167 17113 127.0.0.1:55826                70016/Satoshi:26.0.0/
 in          npr  1    300    362    2    2   49            574     1  293 18137 127.0.0.1:59070                70016/Satoshi:25.0.0/
 in          npr  1    301    412    2    6   51           3232       1799 16400 127.0.0.1:57090                70016/Satoshi:26.0.0/
 in          npr  1    346  13863   30   16                  20        960 17359 127.0.0.1:49610                70016/Satoshi:23.0.0/
 in          i2p  2    560   1480   72   72    *              .        758 17598 pt-%<--cut-->%-.b32.i2p:0      70016/Satoshi:27.0.0/
 in          i2p  2    572   1906    2    5   52            509        301 18122 lm-%<--cut-->%-.b32.i2p:0      70016/Satoshi:27.1.0/
 in          i2p  2    606   4995    2    2   51           1004        496 17892 jw-%<--cut-->%-.b32.i2p:0      70016/Satoshi:27.1.0/
out   full onion  1    117    130    0    1    0   40  .   7901     2 3734 14307 i2-%<--cut-->%-.onion:8333     70016/Satoshi:26.0.0/
out  block onion  2    197    241   28   28    *              .         18 18480 dj-%<--cut-->%-.onion:8333     70016/Satoshi:27.1.0/
out   full onion  2    202    325    0    1    0           7653       3151 14939 c6-%<--cut-->%-.onion:8333     70016/Satoshi:27.1.0/
out   full onion  1    231   2633    1    1    0           7267       3210 14876 n3-%<--cut-->%-.onion:8333     70016/Satoshi:27.1.0/
out   full onion  1    232   1799    0    0    0   20  .  13617       6480 11445 q4-%<--cut-->%-.onion:8333     70016/Satoshi:27.1.0/
out   full onion  2    254    298    0    0    0           3286    18 1002 17308 e3-%<--cut-->%-.onion:8333     70016/Satoshi:27.0.0/
out   full onion  2    288   4042    2    7    0           6074       2994 15099 e2-%<--cut-->%-.onion:8333     70016/Satoshi:27.1.0/
out   full onion  2    319    535    3   31    0  943      7848       3199 14889 ed-%<--cut-->%-.onion:8333     70016/Satoshi:27.1.0/
out   full   i2p  2    589   2040    2    2    2           1254        112 18355 sk-%<--cut-->%-.b32.i2p:0      70016/Satoshi:27.1.0/
out  block   i2p  2    829  18705    0  102    *              .        312 18103 wf-%<--cut-->%-.b32.i2p:0      70016/Satoshi:27.1.0/
                        ms     ms  sec  sec  min  min                  min

        onion     i2p   cjdns     npr   total   block
in          0       3       0       9      12
out         8       2       0       0      10       2
total       8       5       0       9      22
```

Evolution avec une version plus récente de Core et l'apparition de la colonne `serv` :

```bash
Bitcoin Core client v29.3.0 - server 70016/Satoshi:29.3.0/

<->   type   net   serv  v  mping   ping send recv  txn  blk  hb addrp addrl  age  id address                  version
 in          npr         1      0      0    4    4    *              .        564   6 127.0.0.1:51848          70001/electrs:0.10.9/
 in        onion   nwl2  2    190    429    0    0    4           1604     3  350 160 127.0.0.1:46546          70016/Satoshi:29.3.0/Knots:20260507/
 in        onion   nwl2  2    251    489    6    6                            518  45 127.0.0.1:42772          70016/dsn.kastel.kit.edu/bitcoin:28.0.0/
 in        onion         1    265   5294   94   89    *              .        384 138 127.0.0.1:59036          70016
 in        onion         1    468    571    3    3    *              .        512  50 127.0.0.1:49694          70016
 in          i2p   nwl2  2   1444   3741    0    0                  12          2 422 a6%<--cut-->%.b32.i2p:0  70016/Satoshi:29.3.0/Knots:20260507/
out  block onion  nbwl2  2    147    240    1    1    *              .        564   1 d7%<--cut-->%.onion:8333 70016/Satoshi:29.0.0/
out   full onion   nbwl  1    167    308    1    0    0           3728        562  11 pc%<--cut-->%.onion:8333 70016/Satoshi:29.3.0/
out   full onion  nbwl2  2    235    604    0    0    0           3975     1  564   5 kw%<--cut-->%.onion:8333 70016/Satoshi:30.1.0/
out   full onion    nwl  1    289    411    0    0    0    6  .   3809        562  12 ez%<--cut-->%.onion:8333 70016/Satoshi:27.1.0/
out   full onion  nwcl2  2    318   4136    0    0    0  266      4123    23  557  19 ns%<--cut-->%.onion:8333 70016/Satoshi:29.2.0/
out   full onion   nwl2  2    344    601    0    0    0   58  .   2679    15  306 193 hx%<--cut-->%.onion:8333 70016/Satoshi:28.1.0/
out   full onion    nwl  1    476    831    0    0    0  128  .   4098    10  562  14 sb%<--cut-->%.onion:8333 70016/Satoshi:29.3.0/
out   full onion    wl2  2    551   1166    0    0    0           3647     3  539  32 uv%<--cut-->%.onion:8333 70016/Satoshi:31.1.0/
out  block   i2p  nwcl2  2    925   1221  113  112    *              .        111 346 bv%<--cut-->%.b32.i2p:0  70016/Satoshi:30.0.0/
out   full   i2p    nwl  1   1060   1239    0    0                1037     1    8 416 u4%<--cut-->%.b32.i2p:0  70016/Satoshi:24.0.1/
                               ms     ms  sec  sec  min  min                  min

        onion     i2p     npr   total   block
in          4       1       1       6
out         8       2       0      10       2
total      12       3       1      16
```

Signification des en-têtes de colonnes dans Bitcoin Core Client, cela évolue avec les versions taper : `bitcoin-cli -netinfo help` pour une description à jour.

< - > Sens de la connexion

* in(bound) : connexion entrante, un pair s'est connecté à mon nœud, par cette connexion bidirectionnelle ils s'échangent des données.
* out(bound) : connexion sortante, mon nœud s'est connecté à ce pair, par cette connexion bi-directionnelle ils s'échangent des données.

type, indique le type de chaque connexion :

* **vide**, rien est normal, c'est une connection entrante `inbound`.
* **full**, `outbound-full-relay` relais complet, le type par défaut. Mon nœud choisit ce pair au hasard dans son carnet d'adresses et échange tout avec lui : blocs, transactions, adresses. Son rôle est remplir mon mempool, de diffuser mes transactions, de découvrir des pairs.
* **block**, `block-relay-only` relais de blocs uniquement\*\*.  Le\*\* choix effectué au hasard comme `full` sauf que cette connexion échange uniquement des blocs. Environ toutes les 5 minutes, une connexion `block`supplémentaire est ouverte pour vérifier qu'aucun bloc ne manque, puis refermée. C'est une protection discrète contre l'attaque éclipse (voir plus tard le glossaire), puisque contrairement à `full` ce type est invisible aux techniques d'analyse du trafic. Lors de son arrêt,`bitcoind` écrit dans le fichier `anchors.dat` les adresses de ses connexions `block-relay-only`, au maximum 2. Lors de son démarrage, il lit le fichier, se reconnecte en priorité à ces 2 adresses puis le supprime le fichier.
* **feeler**, `feeler` connexion de courte durée pour tester les adresses du carnet. Si le pair répond, l'adresse passe de la table '"new" à la table "tried" et la connexion est fermée tout de suite. Son rôle est de garder le carnet d'adresses sain, avec des adresses testées qui fonctionnent.
* addr, `addr-fetch`, connexion de courte durée pour demander des adresses. Elle est utilisée au démarrage quand le carnet est vide, ou quand un réseau manque de pairs connus. Avec Tor et i2p only les DNS seeds ne sont pas utilisables directement, donc ce mécanisme compte davantage.
* **manual**, `manual` pair ajouté manuellement. Définie soit par `addnode` ou `connect` dans `bitcoin.conf`, soit en ligne de commande par `bitcoin-cli addnode <addr:port> add`, `addr` peut être une adresse IPV4/6, Tor, I2P, suivi du port facultatif (default 8333). `connect` déactive toutes les connexions automatiques `full`, `block`, `feeler` et les anchors, à éviter sauf pour des configurations particulières. Voici les subtilités de la commande `addnode` :
  * `bitcoin-cli addnode <addr> add`                         ajoute le nœud dans la liste et connecte toi
  * `bitcoin-cli addnode <addr> remove`                   retire le nœud de la liste
  * `bitcoin-cli disconnectnode <addr> ou <id>`    coupe la connexion avec le pair
  * `bitcoin-cli addnode <addr> onetry`                    tente une connexion sans persister, sans ajouter le nœud dans la liste.
  * `bitcoin-cli getaddednodeinfo` pour lister les nœuds ajoutés manuellement.

net, Réseau par lequel le pair est connecté, ipv4, ipv6, onion, i2p, cjdns ou npr

* 'npr' pour "Not Publicly Routable", mon nœud Bitcoin est connecté à un pair dont l'adresse ip n'est pas publiquement routable. Chaque nœud du réseau partage également les informations des autres et un nœud peut conserver les informations dans sa propre base de données locale de nœuds connus, de sorte que d'autres ou vous, pouvez vous connecter à l'autre même si l'ip n'est pas publiquement routable. Dans ce cas `bitcoind` crée des ports locaux (127.0.0.1:xxxxx) éphémères pour pouvoir échanger avec ces pairs.
* onion : la connexion au pair s'effectue par TOR
* i2p : la connexion au pair s'effectue par I2P

v, version utilisée pour la connexion

* v1 correspond à un mode de connexion ou sous-protocole de base, le plus ancien.
* v2 correspond à mode de connexion ou sous-protocole plus récent, offrant des améliorations en termes de performances et de sécurité surtout pour le clearnet IPv4/IPv6.
* en règle générale la version du protocole le plus ancien est utilisé par les versions de Bitcoin Core plus anciennes, mais pas que : pour certaines raisons (configuration réseau, rétrocompatibilité, mises à jour progressives) une même version de Bitcoin Core peut utiliser soit v1 soit v2. `bitcoind` choisit automatiquement la version du sous-protocole en fonction des configurations et capacités du pair avec lequel il se connecte. `v2transport=1` est le défaut depuis Bitcoin Core 27.0. V2 a été introduit désactivé dans Bitcoin Core v26.0. Cela ne présente aucun intérêt mais V2 est dé-activable dans `bitcoin.conf` par `v2transport=0`.

**Colonne serv** abréviation de "services", implémentée pour améliorer le monitoring réseau. Elle concatène des lettres minuscules et des chiffres correspondant aux flags actifs supportés par chaque pair.

* n : NETWORK, peut servir la blockchain complète, full node standard.
* b : BLOOM, supporte les filtres Bloom (BIP37, wallets SPV / légers), **déprécié pour privacy**.
* w : WITNESS, support des données witness (BIP144 SegWit compatible)
* c : COMPACT_FILTERS (BIP157/158) → `blockfilterindex=1` et `peerblockfilters=1` dans `bitcoin.conf`
* l : NETWORK_LIMITED, indique que le nœud est au moins capable de servir 288 blocs. 288 x 10 mn = **2** days, ce qui laisse une marge suffisante pour que les pairs puissent rattraper un retard de synchronisation raisonnable. Indique également un nœud pruned si flag NETWORK absent.
* 2 : P2P_V2 (BIP324) protocole de transport avec chiffrement opportuniste des connexions entre nœuds.
* "u" - UNKNOWN : indicateur non reconnu

`bitcoin-cli getnetworkinfo | jq '.localservicesnames'` pour lister les flags de son propre nœud.

La commande `getpeerinfo` est plus détaillée que `bitcoin-cli -netinfo 4`, sa sortie est un JSON de tous les pairs, pour visualiser les flags "servicename" d'un pair en particulier, ici par l'id 1857, tapez ceci :

```bash
# Si jq pas installé (jq = processeur JSON en ligne de commande) 
sudo apt-get update && sudo apt-get install jq

# Commande proprement dite
bitcoin-cli getpeerinfo | jq '.[] | select(.id == 1857)'

# Visualiser uniquement les flags servicenames pour le pair 1857
bitcoin-cli getpeerinfo | jq '.[] | select(.id == 1857) | .servicesnames'

# Lister tous les pairs par leur id et afficher leurs flags "servicesnames"
bitcoin-cli getpeerinfo | jq -r '.[] | "\(.id): \(.servicesnames | join(", "))"'
```

mping

* ping minimal observé, en millisecondes (ms)

ping

* dernier ping observé, en millisecondes (ms)

send

* nombre de secondes écoulées depuis le dernier message envoyés au pair, pouvant inclure des transactions, blocs, requêtes, réponses, etc… le compteur est remis à zéro après chaque message.

recv

* nombre de secondes écoulées depuis le dernier message reçu du pair, pouvant inclure des transactions, blocs, requêtes, réponses, etc… le compteur est remis à zéro après chaque message.

txn

* temps écoulé depuis la dernière transaction nouvelle reçue de ce pair et acceptée dans le mempool de notre noeud, en minutes.
* `*` indique que nous ne relayons pas de transactions vers ce pair ("relaytxes" est false dans getpeerinfo)

blk

* temps écoulé depuis le dernier bloc nouveau reçu de ce pair et ayant passé les premières vérifications, en minutes.

hb

* signifie "high bandwidth compact block relay"
* Relais haute bande passante des compact blocks (BIP152)
* `.` (vers) mon nœud a choisi ce pair comme pair haute bande passante
* `*` (de) ce pair a choisi mon nœud comme pair haute bande passante
* Voir Compact block relay dans le glossaire.

addrp

* nombre total d'adresses traitées, sans compter celles rejetées par la limitation de débit.
* `.` indique que nous ne relayons pas d'adresses vers ce pair (`addr_relay_enabled` est false dans getpeerinfo)

addrl

* nombre total d'adresses rejetées par la limitation de débit

age

* durée de la connexion dans le temps spécifié en bas de la colonne

id

* numéro du pair, attribué dans l'ordre croissant des connexions depuis le démarrage du nœud. Chaque pair se voit attribuer un identifiant unique lorsqu'il se connecte à mon nœud. Cet identifiant est temporaire et spécifique à la session de connexion. L'id permet à `bitcoind` de gérer les connexions, les envois et les réceptions de messages, ainsi que d'effectuer le suivi des statistiques et des performances de chaque pair individuel. L'identifiant unique est utile pour diagnostiquer et résoudre des problèmes de connectivité ou de performance en permettant de suivre les interactions spécifiques avec chaque pair. Le pair est identifié dans les logs par cette id.

address

* Désigne l'autre extrémité de la connexion, vue par votre nœud.
* Pour un pair entrant `in` , c'est l'adresse d'ou provient la connexion :
  * IPv4/IPv6 affichent l'IP réelle du pair et I2P sa destination `.b32.i2p`
  * \[localhost:port_éphémère\] comme \[127.0.0.1:xxxxx\] désigne soit un pair Tor relayé par le démon Tor local (son .onion reste inconnu, c'est voulu par Tor), soit un programme local comme Electrs.
* Pour un pair sortant `out`, cela indique la destination, c'est-à-dire l'adresse d'écoute du pair que votre nœud a appelée.
* Ne pas utiliser le flag `externalip=adresse` insérable dans `bitcoin.conf`, **sauf si vous savez ce que vous faites**. Ce flag indique manuellement à `bitcoind` sa propre adresse publique, celle qu'il va annoncer au réseau pour que d'autres nœuds puissent se connecter à lui. **Ne concerne donc que les connexions entrantes.**

version

* "Protocol version" suivi de "subversion" du logiciel Bitcoin Core utilisé par le pair. Pour déconnecter un pair qui ne vous plait pas car il n'annonce pas sa subversion, la chaîne de caractère qu'il renvoie est vide : `bitcoin-cli disconnectnode "" id_du_pair` .

Ci dessous la sortie de [Bitcoin Knots](https://bitcoinknots.org/) en v29.1 : celle-ci comporte une nouvelle colonne par rapport à Bitcoin Core pour un monitoring avancé des nœuds connectés :

```bash
Bitcoin Knots client v29.1.knots20250903 - server 70016/Satoshi:29.1.0/Knots:20250903/ - services nwl2r

<->   type   net   serv  v  mping   ping send recv  txn  blk  hb addrp addrl cpu  age   id address                                                             version
 in          npr         1      0      0    3    3    *              .           7086    4 127.0.0.1:57536            70001/electrs:0.10.9/
 in          npr   nbwl  1    133    364    0    0    5           7105         1 3680 4089 127.0.0.1:60458            70016/Satoshi:26.0.0/
 in          npr   nbwl  1    140    187    0    0    6            475         1  241 8247 127.0.0.1:34008            70016/Satoshi:25.0.0/
 in          npr  nwcl2  2    151    197    0    0    1           6609         1 3080 4837 127.0.0.1:48622            70016/Satoshi:28.0.0/
 in          npr   nwl2  2    153    366    0  107                               1091 7186 127.0.0.1:36508            70016/dsn.kastel.kit.edu/bitcoin:28.0.0/
 in          npr   nwl2  2    160    193    0    0    0           1692            936 7374 127.0.0.1:38648            70016/Satoshi:29.2.0/Knots:20251010/
 in          npr  nwcl2  2    169    218  105  105    *              .            686 7711 127.0.0.1:37356            70016/Satoshi:29.0.0/
 in          npr   nbwl  1    213    347    0    1   18           7287         1 4164 3491 127.0.0.1:53982            70016/Satoshi:26.0.0/
 in          npr nbwcl2  2    236    495    0    0   12        *  5643         1 3056 4873 127.0.0.1:45188            70016/Satoshi:29.0.0/
 in          npr   nwl2  2    276    445    0    0    1           6244         1 3514 4331 127.0.0.1:39260            70016/Satoshi:28.1.0/
 in          npr  nwcl2  2    323    459    0    0  104           1283         1  482 7959 127.0.0.1:50226            70016/Satoshi:29.0.0/
 in          npr         1    328    381  108  109    *              .            523 7898 127.0.0.1:58212            70016
 in          npr   nwl2  2    335    443   40   40    *              .            142 8349 127.0.0.1:53610            70016/Satoshi:27.1.0/
 in          npr   nbwl  1    340    925    0    3  103            365         1  178 8307 127.0.0.1:42082            70016/Satoshi:26.0.0/
 in          npr   nwl2  2    381    425    0    0  113           2355         1 1316 6920 127.0.0.1:43998            70016/Satoshi:28.1.0/
 in          npr  nwcl2  2    385    385   31   30                                  0 8565 127.0.0.1:44520            70016/Satoshi:29.2.0/Knots:20251010/
 in          npr   nwl2  2    392    432    0    0                  23         1   23 8533 127.0.0.1:58232            70016/Satoshi:29.2.0/Knots:20251010/
 in          npr   nwl2  2    392    660    9    9    *              .            482 7958 127.0.0.1:45958            70016/Satoshi:29.2.0/Knots:20251010/
 in          npr    nwl  1    398    799    0    0    0            804            569 7849 127.0.0.1:51486            70016/Satoshi:29.2.0/Knots:20251010/
 in          npr         1    543    639    0    0                   .         1  281 8201 127.0.0.1:37238            70016/Satoshi:27.0.0/
out   full onion  nwl2r  2    102    271    1    0    0    1  .   2304         1  605 7813 %<--cut-->%.onion:8333     70016/Satoshi:29.1.0/Knots:20250903/
out   full onion  nwcl2  2    169   3395    2    2    2    9  .   2266           1093 7183 %<--cut-->%.onion:8333     70016/Satoshi:29.1.0/
out   full onion    nwl  1    203    203    0    6    0           1014             11 8548 %<--cut-->%.onion:8333     70016/Satoshi:29.1.0/
out  block onion  nwl2r  2    274    298   94   94    *              .            321 8162 %<--cut-->%.onion:8333     70016/Satoshi:29.1.0/Knots:20250903/
out   full onion   nwl2  2    287    529    1    4    1   18  .  12313           6826  290 %<--cut-->%.onion:8333     70016/Satoshi:29.1.0/
out   full onion   nwl2  2    296    317    1    0    0           3450         1 2529 5485 %<--cut-->%.onion:8333     70016/Satoshi:28.1.0/
out  block onion   nwl2  2    309    362   14   14    *              .           3095 4813 %<--cut-->%.onion:8333     70016/Satoshi:29.2.0/Knots:20251010/
out   full onion   nwl2  2    323   1036    0    0    0 4800      8829         3 5545 1857 %<--cut-->%.onion:8333     70016/Satoshi:29.0.0(@wiz)/
out   full onion   nwl2  2    384    503    5    2    0           2116            743 7646 %<--cut-->%.onion:8333     70016/Satoshi:28.0.0/
out   full onion   nbwl  1    390    452    0    3   94           7969           3961 3759 %<--cut-->%.onion:8333     70016/Satoshi:26.0.0/
                               ms     ms  sec  sec  min  min                   ‰  min

        onion     i2p     npr   total   block
in          0       0      20      20
out        10       0       0      10       2
total      10       0      20      30
```

**Colonne cpu** pour "cpu_load", cela mesure le temps CPU que le thread de traitement des messages consacre à chaque pair. Exprimée en permilles (‰) de la durée totale de la connexion. C'est-à-dire : le temps CPU (utilisateur + système) passé à traiter les messages reçus du pair et à préparer les réponses, divisé par la durée de la connexion (multiplié par 1000 pour obtenir des ‰). Exemple par l'absurde : 500 pour 500‰, donc 50% de temps CPU pour un pair connecté, cette valeur (fictive) très élevée indique que ce pair est en train de vous spammer ou DDOS :( → déconnectez le avec `disconnectnode` :) . C'est la PR #31672 de Bitcoin Core ouverte en janvier 2025 et toujours pas fusionnée car plusieurs réserves à éclaircir la bloquent pour l'instant. Luke Dashjr a intégré cette PR alors qu'elle n'était pas encore acceptée dans Core. Elle est présente depuis Knots 28.1.knots20250305. Bien ou mal ? aucune idée !

## Mise en place du service `bitcoind`

Votre nœud est maintenant synchronisé, si vous avez une partition BTRFS effectuez un Snapshot de `@bitcoin`. Ensuite passez à mise en place du service permettant d'automatiser le lancement et l'arrêt du nœud Bitcoin au lancement et à l'arrêt de la machine. Dans un premier temps, créer le fichier `bitcoin.service`

```none
sudo nano /etc/systemd/system/bitcoin.service
```

```bash
# /etc/systemd/system/bitcoin.service
# Adaptez l'utilisateur, ici btc-node
# Inspire de : https://github.com/bitcoin/bitcoin/tree/master/contrib/init

[Unit]
Description=Bitcoin daemon
# Attendre le reseau, Tor et I2P (noeud Tor/I2P uniquement)
Wants=network-online.target tor.service i2pd.service
After=network-online.target tor.service i2pd.service

[Service]
# -daemon=0 : bitcoind reste au premier plan, gere par systemd
# (prime sur le daemon=1 de bitcoin.conf, utile pour les lancements manuels)
ExecStart=/usr/local/bin/bitcoind -daemon=0 \
                                  -conf=/home/btc-node/.bitcoin/bitcoin.conf \
                                  -datadir=/home/btc-node/.bitcoin \
                                  -startupnotify='systemd-notify --ready' \
                                  -shutdownnotify='systemd-notify --stopping'
Type=notify
NotifyAccess=all
Restart=on-failure
TimeoutStartSec=infinity
# Temps laisse a bitcoind pour ecrire son cache (dbcache) sur disque a l'arret
TimeoutStopSec=600

User=btc-node
Group=btc-node

# Durcissement : limite les degats si bitcoind etait compromis
# /tmp prive, invisible des autres processus
PrivateTmp=true

# Tout le systeme de fichiers en lecture seule, sauf ReadWritePaths
ProtectSystem=strict

# /home en lecture seule : btc-node est aussi l'administrateur (sudo), un
# bitcoind compromis ne pourra pas modifier ~/.ssh/authorized_keys, ~/.bashrc ...
ProtectHome=read-only

# Seul le repertoire de donnees reste modifiable. Le "-" ignore un chemin absent :
# le lien ~/.bitcoin et ses cibles possibles (ext4 ou BTRFS), adaptez si besoin
ReadWritePaths=-/home/btc-node/.bitcoin -/mnt/nvme/bitcoin -/mnt/btrfs/bitcoin

# Aucune elevation de droits (sudo, setuid)
NoNewPrivileges=true

# Pas d'acces aux peripheriques (disques, USB)
PrivateDevices=true

# Pas de memoire a la fois ecrivable et executable
MemoryDenyWriteExecute=true

# Appels systeme de l'architecture native uniquement
SystemCallArchitectures=native

[Install]
WantedBy=multi-user.target
```

Pourquoi ce durcissement ? L'utilisateur `btc-node` qui lance `bitcoind` est aussi le compte d'administration avec `sudo`. Sans protection, une faille exploitée dans `bitcoind` permettrait d'écrire dans le home de cet utilisateur, par exemple d'ajouter une clé dans `~/.ssh/authorized_keys` ou une commande dans `~/.bashrc`, puis de récupérer les droits `sudo` à votre prochaine connexion. `ProtectSystem=strict`, `ProtectHome=read-only` et `ReadWritePaths` limitent l'écriture au seul répertoire de données, et `NoNewPrivileges=true` empêche tout `sudo` depuis le service. La solution consacrée reste un utilisateur système dédié sans `sudo` par service, mais on perd la simplicité d'un seul utilisateur !

Recharger la configuration de `systemd` après chaque création ou modification du fichier : `sudo systemctl daemon-reload`

Vérifier la syntaxe par : `sudo systemd-analyze verify /etc/systemd/system/bitcoin.service` vide:ok

Voir le niveau d'exposition du service : `systemd-analyze security bitcoin.service`

Afin de vérifier le déroulement, ouvrir un nouveau terminal puis observer les logs par :

`tail -f ~/.bitcoin/debug.log`

Si `bitcoind` est lancé l'arrêter par `bitcoin-cli stop` , attendre l'apparition dans les log de `Shutdown: done`

Activer le service `sudo systemctl enable bitcoin.service`

Démarrer le service `sudo systemctl start bitcoin.service` et attendre le retour du prompt.

Vérifier que le service fonctionne correctement `sudo systemctl status bitcoin.service`

Tout en observant les logs dans un terminal, lancez sur un autre `sudo reboot` vous devez voir `Shutdown: done` dans les logs juste avant que la machine s'arrête, cela signifie que le nœud a stoppé proprement. Démarrez la machine, `bitcoind` doit se lancer automatiquement. Bien sûr, si le service est actif ne plus utiliser `bitcoin-cli stop`, dorénavant pour l'arrêter faire : `sudo systemctl stop bitcoin.service`

Pour info si vous souhaitez désactiver ce service `sudo systemctl disable bitcoin.service`

# Choix de connexion portefeuille à nœud personnel

Souveraineté individuelle ou ne dépendre d'aucune entité, c'est beau mais surtout important. Vous voulez effectuer un transfert de valeur : votre portefeuille est connecté à votre propre nœud, vous initiez une nouvelle transaction (origination) qui sera signée avec vos clés privées signifiant l'autorisation de dépenser les fonds référencés. Elle sera vérifiée puis annoncée par votre nœud pour être ensuite diffusée à tous les pairs. Chaque nœud vérifiera la validité de cette transaction, quand c'est fait les nœuds stockent celle-ci dans une mémoire temporaire appelée "mempool", les nœuds propagent également le "mempool" aux autres nœuds afin que chaque nœud dispose des mêmes transactions en attente. Les mineurs sélectionnent les transactions à partir du "mempool", puis ils assemblent celles-ci dans un potentiel nouveau bloc, ils sont en compétition entre eux pour résoudre un problème cryptographique. Le premier qui a trouvé gagne la récompense de bloc et tous les frais de transaction. Il propose le nouveau bloc aux nœuds qui le vérifient afin de l'intégrer dans leur copie de la "blockchain". Les nœuds mettent à jour la "mempool" en retirant toutes les transactions qui ont été intégrées dans le nouveau bloc. Là il s'est écoulé approximativement 10mn … et cela dure depuis plus de 15 ans, oui **Bitcoin EST un ovni monétaire**. *Propos d'Andreas Antonopoulos : nous assistons aux balbutiements d'une technologie fondamentalement déstabilisatrice et révolutionnaire qui est en train de transformer une technologie des plus anciennes de cette planète, la technologie de l'argent. Aujourd'hui elle est difficile d'utilisation, compliquée, comprise par une infime fraction de la population humaine. Mais elle va changer le monde !*

Pour gérer un portefeuille Bitcoin qui interagit avec son nœud, il y a différentes solutions qui peuvent demander des données et des ressources supplémentaires. En octobre 2024 j'ai comparé ces données supplémentaires avec la taille de la blockchain Bitcoin elle-même.

| Données | Taille | Information |
|----|----|----|
| blockchain | 656Gio | La référence d'octobre 2024 pour le calcul en % des autres données. |
| txindex | 58Gio\[8%\] | `txindex=1` dans `bitcoin.conf`, ce filtre répond à "dans quel bloc se trouve ce txid quelconque ? " Il permet de retrouver n'importe quelle transaction à partir de son seul identifiant. Sans ces données le nœud ne peut retrouver une transaction que si elle est dans le mempool ou si elle appartient à un des wallets qu'il intègre. Requis pour toute utilisation didactique ou d'investigation sur la blockchain. Requis également par les explorateurs de blocs self hosted (btc-rpc-explorer / mempool.space) et le serveur SPV Fulcrum, recommandé pour Electrs). |
| blockfilter/basic | 11Gio\[1.7%\] | `blockfilterindex=1` dans `bitcoin.conf`, venu avec BIP158. Ce filtre répond à "quels blocs concernent mes adresses". Ces données accélèrent x5 à x10 l'analyse des blocs s'il est fait usage de la fonctionnalité wallet de `bitcoind`, soit directement ou indirectement avec des portefeuilles logiciels tiers comme Specter ou Sparrow. Ce filtre est compatible avec les nœuds prunés contrairement à `txindex`. *Choix de contribution "pour les autres que vous" : si ce filtre est activé vous pouvez activer un autre filtre* `peerblockfilters=1` *qui ne nécessite pas d'espace disque supplémentaire et sert les clients légers du réseau (avec ces 2 filtres activés votre nœud annoncera aux autres* COMPACT_FILTERS) . |
| electrum | 47Gio\[7.2%\] | Bien qu'il soit possible de s'en passer, le standard SPV d'Electrum est très utilisé. Les wallets SPV n'utilisent pas la fonctionnalité wallet de `bitcoind` mais passent par une couche intermédiaire composée d'un serveur Electrum et de sa propre base de donnée. |

Bien qu'Il soit possible de créer un wallet directement avec votre nœud Bitcoin pour effectuer des transactions, il est plus que souhaitable d'utiliser un portefeuille tiers qui va vous faciliter la tâche. Effectuer des transactions en ligne de commande n'est pas évident, il faut effectuer régulièrement des sauvegardes du wallet puisqu'il n'y a pas de phrase de récupération. La conservation de ce minimalisme dans le code est cependant une excellente chose !

Ensuite gardez à l'esprit que votre nœud est constamment connecté à internet et donc exposé à des risques potentiels. Si le wallet intégré à `bitcoind` est utilisé (directement ou avec la librairie Cormorant), cela peut faire de vous une cible si votre solde est découvert.

Afin d'élargir le choix des portefeuilles utilisables, il est opportun d'installer un serveur Electrum sur le nœud Bitcoin. Il sera ensuite possible d'utiliser le portefeuille Electrum ou un autre pourvu qu'il soit compatible avec le format SPV d'Electrum. Lorsque la date de naissance du portefeuille est inconnue (date de la 1 ère transaction) toute la blockchain doit être parcourue depuis l'origine. Le format SPV permet d'obtenir le solde d'un wallet Bitcoin bien plus rapidement que l'utilisation directe ou indirecte du wallet de Bitcoin et ce même avec `blockfilterindex` activé.

Avant de se lancer dans d'autres installations, et que l'un n'empêche pas l'autre, ci-dessous la configuration `bitcoin.conf` pour une connexion directe portefeuille au nœud Bitcoin par RPC (Remote Procedure Call ou appel de procédure distante).

## Portefeuille connecté directement à Bitcoin

Si vous préférez le standard SPV d'Electrum, sautez donc ce paragraphe !

Vérifiez d'abord si le portefeuille que vous voulez utiliser est capable de se connecter directement au nœud Bitcoin. [Point de départ pour évaluer les wallets](https://bitcoin.org/en/choose-your-wallet?step=5), sélectionnez ensuite votre matériel / OS.

Éditez et rajoutez ces lignes à la fin de `bitcoin.conf` par `nano ~/.bitcoin/bitcoin.conf`

```bash
# Config pour Sparrow wallet ou autre utilisant l'accès rpc de bitcoind
#
# Commentez txindex dans bitcoin.conf si vous n'en avez aucun besoin par ailleurs
#txindex=1

# BIP158 : Index de filtres de blocs
# Apres avoir redemarre le noeud, bitcoind va creer cet index de filtrage, cela
# prend du temps, observer les logs ... "Syncing basic block filter index"
# Ensuite le wallet de bitcoind parcours environ 5x plus rapidement la blockchain
# a la recherche de transactions sur une adresse (rescanblockchain)
# Les logs indiqueront "fast variant using block filters" a l'oppose de
# "slow variant inspecting all blocks" si l'index n'est pas construit
blockfilterindex=1

# rpcauth permet de se connecter à bitcoind avec authentification
# Comment generer votre clé ? voir (1)
rpcauth=btc-node:creer-votre-propre-authentification-voir-plus-bas

# Ferme la porte a toute connexion externe
rpcbind=127.0.0.1

# C'est l'IP de votre noeud sur votre resau local
# Attention : le serveur REST (rest=1, sans authentification) sera lui aussi
# accessible sur cette adresse depuis votre reseau local
rpcbind=192.168.X.XXX

# Ferme la porte a toute connexion externe
rpcallowip=127.0.0.1

# Si non défini dans bitcoin.conf !
# Adaptez l'adresse par celle de votre réseau local, exemple : 192.168.0.0/24
# autorise 256 adresses de (192.168.0.0 à 192.168.0.255) 
rpcallowip=192.168.X.0/24
```

(1) Afin de sécuriser l'accès RPC à `bitcoind` en utilisant une méthode plus robuste qu'un simple mot de passe en texte clair (tandem rpcuser / rpcpassword) dans `bitcoin.conf`, il faut utiliser 'RPC Auth'. Pour cela, exécuter sur votre nœud ceci :

```bash
# En fin de ligne, adaptez l'utilisateur et le mot de passe
# Mieux : ne donnez pas de mot de passe, rpcauth.py en generera un aleatoire
# et il ne restera pas dans l'historique du shell
python3 ~/code/bitcoin/share/rpcauth/rpcauth.py btc-node who-are-you-satoshi?
```

voici la sortie :

```none
String to be appended to bitcoin.conf:
rpcauth=btc-node:5a8257571875d0d6ee80df0741c45e5b$ceb75c8e6bde720896714a9981f0a149c068db8d7a273f66e35f70e13484e5b2
Your password:
who-are-you-satoshi?
```

En fonction de vos choix, modifiez la ligne `rpcauth` de `bitcoin.conf`, redémarrez `bitcoind`, puis installez sur votre ordinateur personnel un portefeuille capable de se connecter directement à votre nœud Bitcoin, par exemple Sparrow wallet qui utilise la librairie Cormorant, puis allez dans :

`File/Preferences puis Server`

* Activer Bitcoin Core
* URL: <ip de votre noeud> / 8332
* User / Pass: btc-node / who-are-you-satoshi?
* Faire `Test Connection`

# Logiciel serveur Electrum

## La construction

Le serveur et le portefeuille Electrum sont apparus en 2011, ils ont été des précurseurs dans le domaine. A l'heure actuelle il existe plusieurs variantes du serveur Electrum, les plus connues sont Fulcrum et Electrs. Fulcrum offre des temps de réponse rapide même sous forte charge mais en contrepartie sa base de données est environ 5 fois plus importante que celle d'Electrs. J'ai choisi de mettre en oeuvre Electrum Rust Server (en condensé `electrs`) qui est plus efficient en terme de ressources pour l'usage recherché. L'installation s'effectue sur le nœud par :

**Compatibilité Electrs / nœud (octobre 2026)** : Electrs `v0.12.0` (13 septembre 2026) ne lit plus les blocs par le protocole P2P mais par de nouvelles fonctions REST de Bitcoin Core, ajoutées en v30 (PR #32540) et v31 (PR #33657), deux PR écrites par Roman Zeyde, l'auteur d'Electrs. Elle exige donc **Bitcoin Core 31 ou plus avec** `rest=1` et Rust 1.85 ou plus. Avec un nœud Core 30 ou moins, ou avec Knots 29.x qui n'a pas ces fonctions, restez sur Electrs `v0.11.1`. Le nouvel index est plus compact (environ 7 % de la taille des blocs au lieu de 10 %) et les requêtes sont plus rapides. `txindex` n'est pas nécessaire à Electrs.

```bash
sudo apt-get update
# Dependances de construction (doc officielle Electrs pour Debian 13)
sudo apt-get install build-essential libclang-dev git cargo
# Rust 1.85 minimum : c'est la version de Debian 13. Sous Debian 12 (Rust 1.63)
# installer un Rust recent avec https://rustup.rs
rustc --version

# Aller sur https://github.com/romanz/electrs/blob/master/doc/install.md
# et faire un choix pour librocksdb, link statique ou dynamique ?
# J'ai choisi link dynamique : RocksDB 9.10.0 exige, c'est celle de Debian 13
sudo apt-get install librocksdb-dev
apt-cache policy librocksdb-dev   # verifier que la version installee est 9.10.0-...

cd ~/code
git clone https://github.com/romanz/electrs
cd electrs

# Obtenir la version de la dernière release
git tag | sort --version-sort | tail -n 1
# Octobre 2026 : "v0.12.0", qui exige Bitcoin Core 31+ (voir ci-dessus)

# Verification OBLIGATOIRE de la signature de la release
# Les tags jusqu'a v0.11.0 sont signes avec la cle GPG de Roman Zeyde :
curl https://romanzey.de/pgp.txt | gpg --import
gpg --fingerprint 87CAE5FA46917CBB
# Empreinte attendue : 15C8 C357 4AE4 F1E2 5F3F  35C5 87CA E5FA 4691 7CBB

# A partir de v0.11.1 les tags sont signes avec une cle SSH, git verify-tag
# echoue si on ne lui indique pas la cle autorisee :
echo "me@romanzey.de ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAZVq/3fgkildjN/MqEnhrP5550sDpFzGxMwevr5q/9w" > ~/.ssh/allowed_signers_electrs
git -c gpg.ssh.allowedSignersFile=$HOME/.ssh/allowed_signers_electrs verify-tag "v0.12.0"
# Sortie attendue :
#   Good "git" signature for me@romanzey.de with ED25519 key SHA256:GifMn7F2swVKyn6MewbQHrYCs4i/bPK7gnwxhuPz/YA
# Seconde source pour cette cle : https://api.github.com/users/romanz/ssh_signing_keys
# (et la mention "Verified" sur la page des tags GitHub)

# Choix de la release
git checkout "v0.12.0"

# Construction avec link dynamique de librocksdb
ROCKSDB_INCLUDE_DIR=/usr/include ROCKSDB_LIB_DIR=/usr/lib cargo build --locked --release
# Sans ces deux variables, cargo compile RocksDB en statique (plus long, ok aussi)
```

Patienter cela prend du temps ... poursuivre avec ceci :

```bash
# Installation du binaire en mode manuel
sudo cp ~/code/electrs/target/release/electrs /usr/local/bin/electrs
sudo chown root /usr/local/bin/electrs
sudo chgrp root /usr/local/bin/electrs
```

## Séparer les données Electrs de l'OS

**A effectuer seulement si vous avez 2 unités de stockage distinctes.**

Comme pour Bitcoin il est souhaitable de séparer les données `electrs` de l'Operating System. Si le répertoire `~/.electrs` est présent vérifiez qu'il soit vide, ensuite supprimez le.

```bash
# Creer le repertoire electrs sur l'unite de stockage separee de l'OS
# Adaptez le début du chemin à vos choix précédents
mkdir /mnt/nvme/electrs

# Créez le lien
# Adaptez le chemin à votre peripherique
ln -s /mnt/nvme/electrs ~/.electrs        # pour ext4
# ou
ln -s /mnt/btrfs/electrs ~/.electrs       # pour BTRFS
```

## Configuration d'Electrs

```bash
mkdir ~/.electrs # si une seule unité de stockage
nano ~/.electrs/electrs.toml
```

Copier-coller ceci (adaptez si l'utilisateur est différent de 'btc-node')

```bash
# Ce fichier de configuration doit etre dans le chemin
# ~/.electrs/electrs.toml
#
# btc-node est l'utilisateur qui lance le noeud, remplacer si besoin.
# 
# Fichier dans lequel bitcoind stocke le cookie (authentification d'Electrs)
cookie_file = "/home/btc-node/.bitcoin/.cookie"

# L'adresse RPC d'ecoute de bitcoind, le port est generalement 8332.
daemon_rpc_addr = "127.0.0.1:8332"

# Electrs v0.12+ lit les blocs par l'API REST de bitcoind, sur le port RPC ci-dessus.
# L'option daemon_p2p_addr n'existe plus en v0.12, a supprimer.
# Pour Electrs v0.11.x et avant seulement :
# daemon_p2p_addr = "127.0.0.1:8333"

# Repertoire dans lequel l'index Electrum doit etre stocke
db_dir = "/home/btc-node/.electrs/db"

# bitcoin signifie ici mainnet
network = "bitcoin"

# Nombre de transactions a consulter avant de renvoyer une erreur,
# ceci afin d'eviter que des adresses trop populaires ne monopolisent
# le serveur RPC electrs.
# Le defaut est 0, c'est-a-dire sans limite (depuis electrs 0.9.4).
# Mettre une valeur AJOUTE une limite : une adresse qui depasse ce nombre de
# transactions renvoie une erreur. Option sans effet a partir d'Electrs v0.12.
# index_lookup_limit = 1000

# L'adresse qu'Electrs ecoute / votre wallet va s'y connecter
# Avec 0.0.0.0 c'est en clair et c'est une mauvaise idee
# Laissez sous forme de commentaire
# electrum_rpc_addr = "0.0.0.0:50001"

# L'adresse qu'Electrs ecoute / votre wallet va s'y connecter
# Le "tunneling" ssl est hautement recommande pour acceder a electrs
electrum_rpc_addr = "127.0.0.1:50001"

# Sort seulement les informations de base dans les logs
log_filters = "INFO"
```

Régler les permissions par `chmod 600 ~/.electrs/electrs.toml`

## Lancer Electrs une première fois

Le démon `bitcoind` est lancé et c'est synchronisé à 100% ?

* Non → faire le nécessaire
* Oui  → poursuivre

Dans un premier temps lancer manuellement Electrum server par :

```none
electrs --conf ~/.electrs/electrs.toml
```

Attendre qu'il effectue la synchronisation, c'est long ... genre 8h sur mon RPi5 et 3h15 sur le Dell Optiplex 5050.

Electrs indexe à coup de 2000 blocs, à la fin il effectuera un compactage des données (exemple de logs obtenu avec la v0.10, les messages diffèrent à partir de la v0.12) :

```none
[2024-10-11T21:29:02.323Z INFO  electrs::db] starting config compaction
[2024-10-11T21:29:02.324Z INFO  electrs::db] starting headers compaction
[2024-10-11T21:29:02.326Z INFO  electrs::db] starting txid compaction
[2024-10-11T21:44:24.408Z INFO  electrs::db] starting funding compaction
[2024-10-11T22:55:31.464Z INFO  electrs::db] finished full compaction
```

Pour l'arrêter faire `CTRL-C` et si ce n'est pas achevé vous pouvez reprendre par la suite.

## Mise en place du service "electrs"

Comme pour `bitcoind` il faut créer un service avec le fichier `electrs.service` en y copiant le contenu décrit plus bas.

```none
sudo nano /etc/systemd/system/electrs.service
```

```bash
# /etc/systemd/system/electrs.service
# Adaptez l'utilisateur, ici btc-node

[Unit]
Description=Electrs
# Demarrer apres bitcoind, dont electrs a besoin
Wants=bitcoin.service
After=bitcoin.service

[Service]
Type=simple
WorkingDirectory=/home/btc-node/.electrs
ExecStart=/usr/local/bin/electrs --conf /home/btc-node/.electrs/electrs.toml
User=btc-node
Group=btc-node

# Relance en cas d'arret, par exemple si bitcoind n'est pas encore pret
Restart=always
RestartSec=60

# Delai maximal pour demarrer ou s'arreter avant arret force
TimeoutSec=300

# Priorite CPU legerement inferieure a celle de bitcoind
Nice=5

# Journalisation dans le journal systemd (remplace l'obsolete "syslog")
StandardOutput=journal
StandardError=journal
SyslogIdentifier=electrs

# Durcissement : limite les degats si electrs etait compromis
# /tmp prive, invisible des autres processus
PrivateTmp=true
# Tout le systeme de fichiers en lecture seule, sauf ReadWritePaths
ProtectSystem=strict
# /home en lecture seule (btc-node est aussi l'administrateur)
ProtectHome=read-only
# Seul l'index electrs reste modifiable, adaptez si besoin
ReadWritePaths=-/home/btc-node/.electrs -/mnt/nvme/electrs -/mnt/btrfs/electrs
# Aucune elevation de droits (sudo, setuid)
NoNewPrivileges=true
# Pas d'acces aux peripheriques (disques, USB)
PrivateDevices=true
# Pas de memoire a la fois ecrivable et executable
MemoryDenyWriteExecute=true
# Appels systeme de l'architecture native uniquement
SystemCallArchitectures=native

[Install]
WantedBy=multi-user.target
```

Afin de vérifier le déroulement, ouvrir un nouveau terminal puis observer les logs par :

`sudo journalctl -f | grep electrs`

Recharger la configuration de systemd : `sudo systemctl daemon-reload`

Vérifier la syntaxe : `sudo systemd-analyze verify /etc/systemd/system/electrs.service` (ok si vide)

Activer le service `sudo systemctl enable electrs.service`

Démarrer le service `sudo systemctl start electrs.service`

Vérifier que le service fonctionne correctement `sudo systemctl status electrs.service`

Observer les logs dans le terminal, c'est synchronisé avec le dernier bloc, tout est ok ? oui, alors lancez `sudo reboot` vous devez voir `Stopped electrs.service - Electrs` dans les logs juste avant que la machine s'arrête, cela signifie que `electrs` a stoppé proprement. Au redémarrage de la machine, `bitcoind` doit se lancer automatiquement suivi par `electrs`. Pour l'arrêter, par exemple pour une mise à jour de `electrs.toml` faire :

`sudo systemctl stop electrs.service`

Si plus tard vous effectuez de l'exploration didactique avec le wallet Electrum sur des portefeuilles en lecture seule avec des adresses de mineurs, il est possible que le serveur `electrs` ne réponde plus pendant un certain temps. Si cela se produit, fermez le portefeuille qui a engendré le time-out, observez les logs d'`electrs`, puis choisissez une des 2 options :


1. Lancez `sudo systemctl stop electrs.service` , ensuite il est nécessaire d'attendre patiemment 5mn pour retrouver la main, c'est le temps du `TimeoutSec=300` paramétré dans le fichier `electrs.service` , relancez ensuite le service.
2. Ne faites rien et généralement en moins de 10mn quand vous verrez dans les logs `disconnecting due to failed to send response` le serveur `electrs` sera de nouveau opérationnel.

## Chiffrer la connexion

Afin d'établir une connexion chiffrée entre le portefeuille qui sera présent sur votre ordinateur personnel et votre nœud, il faut installer et configurer Nginx.

Prérequis : créer un certificat auto signé. A chaque champ attendu appuyez sur la touche `Enter`, sauf à `Common Name`, mettez `Nakamoto` ou autre chose sans laisser le champ vide.

```bash
openssl req -x509 -nodes -days 10227 -newkey rsa:2048 -keyout ~/.electrs/electrs.local.key -out ~/.electrs/electrs.local.crt
```

Cette commande a généré la clé privée RSA `electrs.local.key` utilisée pour signer le certificat `electrs.local.crt`, le certificat lui même est valide pour 28 ans. D'ici là je serai crevé.

Voir le certificat en clair : `openssl x509 -in ~/.electrs/electrs.local.crt -text -noout`

Installer Nginx et vérifier que le service est actif :

```bash
sudo apt-get update
sudo apt install nginx libnginx-mod-stream
systemctl status nginx
```

Précédemment, dans `electrs.toml` figure la ligne `electrum_rpc_addr = '127.0.0.1:50001` ce sera la connexion amont de Ngnix, la connexion aval sera sur le port 50002.

Electrs n'utilise pas http, mais un flux tcp brut (mod-stream). Il faut configurer Nginx spécialement pour cet usage. Arrêter le service `sudo systemctl stop nginx`

Par `sudo cat /etc/nginx/nginx.conf | less` , visualiser la configuration de Nginx pour vérifier que la ligne `include /etc/nginx/modules-enabled/*.conf;` soit bien présente.

Créer la configuration Nginx pour electrs

`sudo nano /etc/nginx/modules-enabled/99-electrs.conf`

```bash
# configuration Nginx pour electrs
# Ce fichier de configuration doit etre dans le chemin
# /etc/nginx/modules-enabled
#
# Si besoin adaptez l'utilisateur defini, ici c'est btc-node

stream {
    # proxy SSL pour electrs
    upstream electrs {
        server 127.0.0.1:50001;
    }
    server {
        listen 50002 ssl;
        proxy_pass electrs;

        ssl_certificate /home/btc-node/.electrs/electrs.local.crt;
        ssl_certificate_key /home/btc-node/.electrs/electrs.local.key;

        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;

        ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:\
        ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:\
        ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305';
        
        ssl_session_cache shared:SSL:50m; # Verifier si pas defini ailleurs
        ssl_session_timeout 1d;
        ssl_session_tickets off;
        ssl_ecdh_curve X25519:sect571r1:secp521r1:secp384r1;
    }
}
```

Vérifiez la configuration par `sudo nginx -t` la réponse doit être `syntax is ok`, démarrer par `sudo systemctl start nginx.service`

Pour information, tester la clé puis voir le certificat généré :

`sudo openssl rsa -in ~/.electrs/electrs.local.key -check`

`openssl x509 -in ~/.electrs/electrs.local.crt -text -noout`

Afin de vérifier le fonctionnement, installez un portefeuille compatible SPV Electrum sur votre ordinateur personnel , puis paramétrez "Network" à Server: `IP_de_votre_noeud:50002`, si l'option est disponible activez SSL (si elle ne l'est pas, à priori c'est déjà SSL)

*Si vous n'utilisez que ce type de portefeuille, vous pouvez désactiver le wallet de* `bitcoind` *en vous assurant que la ligne* `disablewallet=1` *est bien présente dans* `bitcoin.conf`

## Utiliser Tor

Où que vous soyez sur la planète, si vous avez accès à internet vous aurez également accès à votre serveur Electrum et pourrez interagir avec la blockchain Bitcoin à travers votre propre nœud en toute confidentialité. A travers Tor c'est plus lent tout en étant parfaitement utilisable.

Editer le fichier de configuration `sudo nano /etc/tor/torrc` et rajouter ces lignes à la fin :

```bash
# Configuration pour electrs
HiddenServiceDir /var/lib/tor/electrs_hidden_service/
HiddenServiceVersion 3
HiddenServicePort 50001 127.0.0.1:50001
```

* Stopper `bitcoind` : `sudo systemctl stop bitcoin.service`
* Re-démarrer Tor : `sudo systemctl restart tor`
* Relancer `bitcoind` : `sudo systemctl start bitcoin.service`
* Obtenir l'adresse Tor pour connecter son portefeuille à son nœud :

  `sudo cat /var/lib/tor/electrs_hidden_service/hostname`
* Si vous souhaitez conserver cette adresse onion, effectuez une sauvegarde du répertoire `electrs_hidden_service` sur un support sécurisé et sûr. Pour l'accès il est nécessaire d'utiliser `sudo` sur le nœud puis de passer par la copie intermédiaire dans le home de l'user, changez récursivement le propriétaire puis transférez par `scp -p -r utilisateur@NOM_DE_LA_MACHINE_ou_IP:~/electrs_hidden_service .` Pour finir n'oubliez pas de supprimer le répertoire dans le home de l'utilisateur.

# Les portefeuilles

En interagissant avec la blockchain Bitcoin un portefeuille permet de visualiser son solde, de visualiser ses transactions passées, d'initier de nouvelles transactions puis de les signer et de les propager, de visualiser ses adresses de réception qui ont été générées avec ses clés elles mêmes générées avec sa seed phrase. Un portefeuille est aussi un coffre-fort contenant ses clés. Afin d'appréhender correctement ce qu'est un portefeuille il est nécessaire de comprendre certaines notions. Cela permettra également de faire des choix en connaissance de cause.

## Seed phrase

Également connue sous le nom de phrase de récupération ou phrase mnémonique, cette suite de mots est générée lors de la création du portefeuille. Elle est généralement composée de 12 à 24 mots choisis aléatoirement dans une liste prédéfinie de 2048 mots anglais, elle permet de restaurer l'accès à son portefeuille en cas de perte ou destruction des clés. Bien évidemment il est très important de conserver cette liste en lieu sûr; mais également puisque qu'à partir de celle-ci n'importe qui peut effectuer des transactions en créant un doublon du portefeuille. De plus gardez en mémoire qu'il est toujours utilisé des fonctions à sens unique pour passer de la seed phrase à la clé privée maîtresse ainsi qu'à toutes les autres clés et ce jusque aux adresses publiques Bitcoin.

La seed phrase a grandement simplifié et sécurisé la gestion des clés en permettant de dériver hiérarchiquement toutes celles-ci à partir d'un ensemble de mots. Les portefeuilles utilisant la seed phrase sont qualifiés de déterministes (ou HD Wallet, pour Hierarchical Deterministic Wallet). Cela permet également une interopérabilité sécurisée entre différents portefeuilles (exemple : re-créer à l'identique avec un autre logiciel et ou matériel)

Le concept de seed phrase a été officiellement standardisé en 2013 avec le protocole **BIP-39** (Bitcoin Improvement Proposal). BIP-39 a été proposé par Marek Palatinus (alias Slush), Pavol Rusnak (alias Stick), et d'autres membres de la communauté Bitcoin.

La norme de seed phrase BIP39 est la plus utilisée, et donc la plus universelle pour le moment.

## Choix du "Pay to"

Le terme "Pay to" fait référence aux mécanismes d'adressage utilisés pour envoyer des Bitcoins à une adresse spécifique, chacun étant basé sur une logique cryptographique différente. Ces mécanismes déterminent comment les fonds sont verrouillés et déverrouillés, influençant la sécurité, les frais de transaction, la compatibilité avec les wallets et les cas d'utilisation.

Le choix du `P2` ou du type de script n'est pas anodin ! Commençons un tour d'horizon avec une traduction des conseils lus dans les [sources](https://github.com/bitcoinknots/bitcoin) de [Bitcoin Knots](https://blockdyor.com/bitcoin-knots/) de Luke Dashjr :

* Base58 (Legacy) : présente la compatibilité la plus élevée. *Le meilleur choix pour la santé du réseau Bitcoin.* Entraîne des frais plus élevés. Recommandé. `P2PKH`
* Base58 (P2SH SegWit) : compatible avec la plupart des anciens portefeuilles, il peut entraîner des frais inférieurs à ceux de Legacy. `P2SH-P2WPKH`
* Native SegWit (Bech32) : Frais inférieurs à ceux de Base58, mais certains portefeuilles anciens ne le prennent pas en charge. `P2WPKH`
* Taproot (Bech32m) : Frais les plus bas, mais la prise en charge des portefeuilles est encore limitée. `P2TR`

Ces recommandations datent un peu, mais c'est une première approche.

J'ai obtenu cela en téléchargeant les sources de Bitcoin Knots et en effectuant cette recherche :

`find ~/code/bitcoin -type f -exec grep -H 'Widest compatibility and best' {} \;`


[Tableau d'utilisation des "Pay to"](https://unchained.com/blog/bitcoin-address-types-compared/?utm_campaign=btcmag-launch) depuis l'origine de Bitcoin

| **Type** | **1 ère vue** | **En % de l'offre BTC ¹** | **Utilisation ¹** | Appellation courante | **Encodage** | **Préfixe d'adresse** | Nbre de caractères |
|----|----|----|----|----|----|----|----|
| P2PK | Jan 2009 | 9% (1.7M) | Obsolète | Pay to Public Key | Aucun | Aucun |    |
| P2PKH | Jan 2009 | 43% (8.3M) | Diminue | Adresse Legacy | Base58 | 1 | 26-34 |
| P2MS | Jan 2012 | Négligeable | Obsolète | Multisig nu | Aucun | Aucun |    |
| P2SH | Apr 2012 | 24% (4.6M) | Diminue | Adresse script | Base58 | 3 | 34 |
| P2WPKH | Aug 2017 | 20% (3.8M) | Augmente | SegWit natif | Bech32 | bc1q | 42 |
| P2WSH | Aug 2017 | 4% (0.8M) | Augmente | SegWit natif script | Bech32 | bc1q | 62 |
| P2TR | Nov 2021 | 0.1% (0.02M) | Augmente | Taproot | Bech32m | bc1p | 62 |

¹ valable pour l'année 2024, ces données sont sujettes au changement


Description des `Pay to` :

* `P2PK` `Pay-to-Public-Key` (paiement à la clé publique)
* `P2PKH` `Pay-to-Public-Key-Hash`(paiement à l'empreinte de clé publique)
* `P2MS` `Pay-to-Multisig` (paiement à des signatures multiples → obsolète utiliser `P2SH` )
* `P2SH` `Pay-to-Script-Hash`(paiement à l'empreinte de script)
* `P2WPKH` `Pay-to-Witness-Public-Key-Hash`(paiement à l'empreinte de clé publique témoin)
* `P2WSH` `Pay-to-Witness-Script-Hash`(paiement à l'empreinte de script témoin)
* `P2TR` `Pay-to-Taproot` (paiement à Taproot, "racine pivotante ou principale")


Dans les faits, il y a encore deux `Pay to` avec `Nested ou Wrapped ou Legacy SegWit` qui encapsule SegWit dans le script de P2SH :

* `P2SH-P2WPKH Nested SegWit` enveloppe une adresse "Native SegWit" `P2WPKH` dans une adresse `P2SH`, 3 est leur préfixe et l'encodage est Base58. Appelé aussi `p2sh-segwit`
* `P2SH-P2WSH Nested SegWit` enveloppe une adresse "Native SegWit" `P2WSH` dans une adresse `P2SH`, 3 est leur préfixe et l'encodage est Base58.

Cela a constitué une solution de transition permettant de bénéficier des améliorations de SegWit tout en restant compatible avec les systèmes qui n'étaient pas à jour ou SegWit Natif, les nœuds entre autres. À première vue, les adresses SegWit imbriquées (nested) ne se distinguent pas des autres adresses P2SH.

### SegWit

a été activé au bloc 481 824 le 24 août 2017 via un soft fork du protocole Bitcoin. SegWit est la contraction de Segregated Witness. Dans les faits cela sépare (segregate) les données de signature (witness data) des données de transaction principale. C'est par la [BIP 141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) qu'il a été proposé le 21 décembre 2015 à la communauté pour améliorer la capacité transactionnelle du réseau et corriger la malléabilité des transactions.

* Correction de la malléabilité des transactions : précédemment à SegWit un attaquant était en mesure de changer les identifiants de transaction (TXID) avant que la transaction ne soit confirmée. Cela pouvait entraîner des complications, notamment dans les transactions à signatures multiples. Cette correction a également facilité l'implémentation des canaux de paiement qui forment la base du réseau Lightning.
* En séparant les données de transaction des signatures et en introduisant une nouvelle métrique, SegWit a amélioré la capacité transactionnelle du réseau Bitcoin. Cette métrique est nommée WU (Weight Unit ou Unité de Poids), elle quantifie virtuellement la taille des transactions dans le bloc comme ceci :
  * la limite est fixée à 4 MWU (Mega Weight Unit = 1 million de WU) par bloc avec
    * `WU = 4 x taille transaction hors witness + taille du witness`(¹)
  * Un bloc de transactions unique Legacy occuperait environ 1Mo puisque
    * `4 x 1Mo + 0 = approximativement 4MWU`
  * Un bloc de transactions uniquement SegWit augmenterait la taille réelle du bloc d'un facteur inférieur à 4. En résumé, le maximum théorique approche 4 Mo sans jamais l'attendre.
  * En réalité la [taille des blocs](https://www.blockchain.com/fr/explorer/charts/blocks-size) est variable en fonction des types de transactions inclues dans celui-ci, de l'activité et de l'usage du réseau, la moyenne constatée en début d'année 2025 se situe à 1,85Mo par bloc avec un MWU proche de 4.
  * Avant SegWit, la taille des blocs était mesurée en octets et plafonnée à 1 Mo, une limite ajoutée par Satoshi en 2010 (auparavant seule la taille maximale des messages réseau, 32 Mo, bornait implicitement les blocs). SegWit a levé cette barrière à sa façon.
* SegWit permet également une réduction conséquente des frais de transaction évalués en `sat/vB` (1 satoshi = 0,00000001 BTC et vB = virtual byte ou octet virtuel. La définition du vB est WU de la transaction divisé par 4 (vB=WU÷4), cela permet de comparer les transactions Legacy et SegWit sur une base commune. Avant SegWit les frais étaient mesurés en fonction de la taille en octets de la transaction, avec SegWit c'est en fonction du WU ÷ 4 soit `vB = taille hors witness + (taille du witness ÷ 4)`. Donc au point de vue des frais SegWit est avantagé par rapport à Legacy, de plus vu que le bloc est limité à 4MWU, les mineurs sont incités à inclure les transactions possédant un bon ratio entre ses frais et son poids en WU.

(¹) Pour éviter toute confusion entre x3 et x4 due à la définition officielle [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) où WU est présenté ainsi : `WU = taille TX hors witness x 3 + (taille TX hors witness + taille du witness)` **est identique à** `WU = 4 x taille TX hors witness + taille du witness`

Informations complémentaires

* les tailles sont évaluées en octets, donc :
  * Chaque octet de la partie non-témoin d'une transaction compte pour 1 octet virtuel.
  * Chaque octet de la partie témoin d'une transaction compte pour 1/4 d'un octet virtuel.
* Le poids d'un bloc est simplement la somme des poids de toutes les transactions qu'il contient, il n'y a donc qu'une seule et même formule WU pour les transactions et les blocs.
* Les données hors witness incluent : version, nombre d'entrées, entrées elles-mêmes (sans scripts témoins), nombre de sorties, sorties, locktime.
* Les données witness incluent : signatures et scripts de déverrouillage.

### **Taproot**

a été activé au bloc 709 632 le 14 novembre 2021, au même titre que SegWit c'est un soft fork majeur. Taproot est la dernière évolution du protocole. Il a été proposé initialement pour améliorer la confidentialité et l'efficacité des transactions Bitcoin. 3 BIPs en sont l'origine, ces propositions d'améliorations ont commencé en janvier 2018.

* BIP340 : Introduit les signatures Schnorr permettant l'agrégation de plusieurs signatures en une seule. Cela améliore l'efficacité, la confidentialité et la taille des transactions, diminue les frais des transactions multi-signatures ou utilisant des scripts complexes.
* BIP341 : mise à jour Taproot. En n'exposant que les détails de la transaction exécutée, Taproot offre une plus grande confidentialité aux utilisateurs. En effet les informations de transactions non exécutées, qui peuvent contenir des informations privées sensibles, ne sont plus enregistrées sur la blockchain.
* BIP342 : introduit Tapscript, cela dote Bitcoin d'un langage de programmation de transactions amélioré, axé sur la technologie Schnorr et Taproot. Permet des scripts de transactions plus flexibles et puissants.

En résumé, Taproot optimise l'espace dans les blocs et a introduit des fonctionnalités avancées. Cela facilite ou permet de nouvelles choses, mais permet également de s'éloigner de l'usage initial de Bitcoin. Les Ordinals en sont l'exemple type par le déguisement de données arbitraires en code de programme au sein de l'espace témoin tapscript. Pour information, les Ordinals sont un protocole qui permet d'inscrire des données (comme des images, des textes ou des vidéos) directement sur des satoshis individuels dans la blockchain. Ainsi un satoshi peut devenir un actif numérique unique, similaire à un NFT (Non-Fungible Token).

Certains diront que l'on s'éloigne vraiment de la philosophie initiale et de la simplicité originale de Bitcoin, **dans le fond ils n'ont pas tort !**

A l'opposé des NFT, des "stablecoins" ou de l'émission de jetons BRC-20, des contrats intelligents de la finance décentralisée, ces évolutions facilitent également la réalisation des couches L2 sensées effectuer des micro-transactions quasi instantanées pour des frais réduits, ce dont la couche L1 est incapable avec 7 transactions par seconde. A bien y réfléchir : les sommes importantes n'ont nullement besoin d'être déplacées à la vitesse de la lumière, donc Bitcoin L1 convient parfaitement offrant une sécurité optimale. Les petits montants n'ont pas ce besoin de sécurité poussée donc les L2 sont tout à fait adaptées pour de nombreuses transactions quasi-instantanées. Cela permet de couvrir un large spectre d'utilisation sans dénaturer le concept initial de Bitcoin et c'est tant mieux !

***Retour à l'essentiel et conclusion pour le long cours*** *: sur les différents matériels et logiciels, le standard qui se dégage est SegWit native, **et même si Legacy est "vieillissant" il est très bien supporté*** ! *Ceci étant faites vos propres recherches et choisissez en connaissance de cause.*

Pour information voici les "Pay to" possibles avec le portefeuille Electrum :

* legacy `P2PKH`
* p2sh-segwit ou wrapped segwit `P2SH-P2WPKH`
* segwit `P2WPKH`
* taproot `P2TR`, Electrum est capable de payer vers une adresse `bc1p` Taproot, mais il ne permet pas de créer un portefeuille Taproot à ce jour (vérifié avec v 4.7.0 début 2026).

## Inter-opérabilité des `Pay To`

**Les scripts** `Pay To` **sont tous interopérables au niveau du protocole Bitcoin et ce de manière transparente pour l'utilisateur.** Si vous rencontrez un problème dans lequel un type d'adresse ne peut pas être envoyé à un autre, il ne s'agit pas d'une limitation du code Bitcoin, mais d'une limitation du portefeuille utilisé.

Parfois il est dit : "Les adresses Legacy (1..) ne sont pas compatibles avec SegWit (bc1q..)", cela signifie simplement qu'une adresse Legacy ne peut pas envoyer de transaction SegWit. Le type de transaction et la possibilité pour l'adresse de réception d'utiliser pleinement tous les avantages de la transaction dépendent de l'adresse d'envoi. Une adresse Legacy P2PKH peut transférer les droits de propriété BTC sur une adresse SegWit, cependant la transaction sera P2PKH et pas SegWit. Dans le détail cela donne :

* Depuis une adresse Legacy (1..) vers une adresse SegWit (bc1q..)
  * L'UTXO source sera au format Legacy
  * L'UTXO de destination sera au format SegWit
  * Les frais seront basés sur la taille Legacy
* Depuis une adresse SegWit (bc1q..) vers une adresse Legacy (1..)
  * L'UTXO source sera au format SegWit
  * L'UTXO de destination sera au format Legacy
  * Les frais seront basés sur une transaction mixte (entrée SegWit, sortie Legacy)

## La collision

Les adresses Bitcoin sont donc générées en aveugle et au hasard. A la création de mon portefeuille est-il possible que je tombe sur une adresse ne m'appartenant pas ou qu'un jour un individu lambda tombe sur la mienne ?

**NON,** ce phénomène aussi appelé collision d'adresse est statistiquement négligeable !

Si la dose de hasard (l'entropie) utilisée pour sélectionner l'adresse dans un ensemble titanesque(¹) d'adresses est suffisante, alors c'est statistiquement impossible.

Avec un hasard bien construit et vu la taille du nombre (¹)

1 460 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000

cela n'arrivera jamais.

Bilan : j'ai fait mes propres recherches, je peux générer un gros paquet d'adresses tranquillement afin de ne jamais les réutiliser et ainsi préserver ma confidentialité. C'est d'ailleurs ce dont les portefeuilles modernes déterministes sont capables, le nombre théorique d'adresses Bitcoin par seed phrase et par "account" du chemin de dérivation est de 2³¹ × 2, soit 4.29 milliards. Comme il n'est pas souhaitable d'afficher des milliers d'adresses vides, les portefeuilles n'affichent en règle générale que les premières. Par défaut le portefeuille Electrum ainsi que d'autres arrêtent de les lister lorsqu'ils trouvent 20 adresses consécutives sans transactions.

*Le génie de ce système est qu'il ne nécessite aucune coordination centrale pour l'attribution des adresses. Chaque utilisateur peut générer ses adresses de manière totalement indépendante, sans avoir besoin de vérifier si elles existent déjà. Ce principe simple mais puissant est l'un des aspects qui préserve la décentralisation de Bitcoin.*

(¹) Une adresse Bitcoin P2PKH/P2WPKH est dérivée d'un hachage `RIPEMD-160(SHA-256(clé_publique))`, ce qui donne un espace de 2¹⁶⁰ possibilités, soit environ 1.46 × 10⁴⁸ adresses uniques. Pour une adresse P2WSH ou Taproot, c'est 2²⁵⁶ donc encore plus !

## Le portefeuille logiciel Electrum

Comme dit précédemment un portefeuille est aussi, si on les lui a confiées, un coffre sécurisé contenant ses clés, le logiciel Electrum peut les générer lui même et les encrypter par un mot de passe sur le disque dur du poste de travail : *c'est le Hot Wallet*. *Remarque importante : si vous créez la seed phrase avec Electrum elle sera incompatible BIP39 (voir glossaire), les développeurs d'Electrum estiment qu'elles ne répondent pas à leurs normes de sécurité car les semences BIP39 ne comportent pas de numéro de version, ce qui compromet la compatibilité avec les futurs logiciels. Pour plus de détail voir [Electrum Seed Version System](https://electrum.readthedocs.io/en/latest/seedphrase.html). Si vous souhaitez du BIP39 utilisez un portefeuille matériel comme décrit ci-dessous ou à des fins didactiques générez avec "BIP39 tools" la seed phrase "BIP39 Mnemonic" puis restaurez le portefeuille avec Electrum, faites "Options" et cochez BIP39 seed.*

Electrum est également capable d'interagir avec un portefeuille matériel du type Trezor, Ledger, BitBox02, etc … les clés seront alors générées et stockées dans le dispositif, dans ce cas aucune clé privée ne sera stockée sur le poste de travail : *c'est le Cold Wallet*. A la création de votre cold wallet avec Electrum si vous avez choisi de l'encrypter avec le dispositif vous ne pourrez l'ouvrir que si le dispositif est branché et déverrouillé. Si votre choix a été de ne pas l'encrypter avec le dispositif vous pourrez l'ouvrir sans celui ci en mode spectateur uniquement, cela sous entend que certaines clés et adresses publiques seront stockées en clair sur votre disque dur.

### Installation

Pour un "Desktop Linux", je décris ici la méthode d'installation `AppImage`.

**Dans tous les cas vérifiez toujours que l'exécutable est original !**

Sur <https://electrum.org/> à la rubrique `Download` téléchargez l'Appimage dans un sous répertoire du home de l'utilisateur.

Ensuite il est primordial de vérifier le fichier `electrum-X.x.x-x86_64.AppImage` avec la signature d'un des concepteurs, ici Thomas Voegtlin. A partir de la même page téléchargez dans le même répertoire la clé publique `ThomasV.asc` pour moi cela donne ceci dans un terminal :

`wget https://raw.githubusercontent.com/spesmilo/electrum/master/pubkeys/ThomasV.asc`

Obtenez sa signature RSA par `gpg --show-keys ThomasV.asc` puis vérifiez la avec un événement, comme cette [vidéo tournée le 7 septembre 2016](https://www.youtube.com/watch?v=hjYCXOyDy7Y) ou apparaît sa signature publique au début, si cela correspond vous avez vérifié sa clé !

Importez la clé dans votre système par `gpg --import ThomasV.asc`

Vérifiez l'intégrité de l' `Appimage` par `gpg --verify electrum-4.5.8-x86_64.AppImage.asc` vous devez obtenir en sortie `Bonne signature de Thomas Voegtlin` (les noms de `*.appimage` *et* `*.appimage.asc` doivent être strictement identiques, sensible à la casse)

Par `chmod u+x appimage_file` donner le droit d'exécution, créez un raccourci sur le bureau puis lancez Electrum.

### Paramétrage

* par le réseau local en ssl
  * Par Tools / Networks / Server
  * Connection mode → Connect only to a single server
  * Server : `ip_ou_hostname-de_votre_noeud:50002:s`
  * Rond état du réseau en bas à droite = vert
* par le réseau Tor (\*)
  * Par Tools / Networks / Proxy
  * Cochez Use Proxy avec Socks5/TOR puis faites Detect Tor Proxy
  * Par Tools / Networks / Server
  * Connection mode → Connect only to a single server
  * Server : `votre_adresse.onion:50001:t`
  * Rond état du réseau en bas à droite = bleu

La syntaxe de connexion à un serveur `electrs` est une URL du format  `host:port:[t|s]` (t pour TCP en clair, s pour SSL/TLS). *Le suffixe :t convient pour Tor puisque Tor chiffre déjà le traffic.*

(\*) Sur votre ordinateur personnel si vous avez besoin d'activer Tor, lancez Tor Browser par exemple, connectez vous et laissez ouvert.

Il est possible interdire au portefeuille Electrum de scanner d'autres serveurs que le votre en éditant manuellement sa configuration par `nano ~/.electrum/config` chercher `"oneserver": false` puis remplacez `false` par `true` , relancez Electrum allez à network, il ne teste plus les adresses différentes de la votre.

### Tests

A des fins didactiques avec Electrum il est possible d'importer une adresse Bitcoin pour visualiser toutes les transactions effectuées sur celle ci, bien sûr c'est en lecture seule (mode spectateur). Attention si vous tapez une adresse avec des milliers de transactions cela risque d'être long et risque de provoquer un time-out d'electrs surtout si la machine est peu véloce comme c'est le cas avec un Raspberry Pi. Si votre version d'electrs est inférieure à 0.12 vérifiez la valeur du paramètre `index-lookup-limit` dans `electrs.toml` qui fixe le plafond du nombre de transactions. Ci-dessous quelques adresses testées avec succès sur mon Dell Optiplex :

* Bloc 210 000 / Bloc du 1ᵉʳ halving adresse du mineur / last tx 22.02.2013 / 5206 transactions
  * `1NEU779yvLaFk39k4Q3QdLjwpWTdWCbzqL`
* Bloc 873 630 / adresse du mineur / actif en 2025 / environ 1230 transactions :
  * `bc1qrpp7g75sx3ejclvsfdw2uahzchtyu7vumkuadu`
* Bloc 873 652 / adresse du mineur / actif en 2025 / environ 2300 transactions
  * `3Awm3FNpmwrbvAFVThRUFqgpbVuqWisni9`

Les autres portefeuilles capables de créer un wallet spectateur d'après une adresse publique sont Bitcoin et BlueWallet qui accepte la connexion SPV à un serveur Electrum et également la connexion RPC à `bitcoind` (exécutables pour Android, IOS et macOS).

Le portefeuille Electrum dispose également d'une console Python intégrée, cela en fait un outil extrêmement puissant qui donne accès direct au cœur du wallet, il est possible de lire, écrire, modifier, automatiser, débugger presque tout ; à condition de savoir ce que l'on fait !

### **Electrum Android**

* installez d'abord l'application `Orbot`, c'est elle qui procurera le réseau TOR à Electrum.
* installez Electrum ou téléchargez l'apk sur le site electrum.org (pour vérifier la signature procédez comme l'Appimage)
* lancez Orbot, démarrez le PRV, dans choisir les application sélectionnez Electrum.
* lancez Electrum, faites Réseau
* à Proxy Settings, cochez `Enable Proxy` avec `SOCKS5/TOR`
  * Adresse `localhost`
  * Port `9050`
* à Réglages serveur, ne pas cocher `Sélectionner un serveur automatiquement`
  * Serveur `votre_adresse.onion:50001:t`
* Notez qu'Electrum Android ne supporte aucun wallet matériel jusqu'à présent.

## Portefeuilles froids

Ce qu'ils sont censés apporter :

* une sécurité accrue avec génération et stockage de la master seed dans une puce spécialisée
* la master seed ne quitte jamais le dispositif même lors de la signature d'une transaction
* ils affichent les détails de la transaction sur leur écran afin que vous puissiez la vérifier
* la transaction est physiquement approuvée ou rejetée sur le dispositif lui même
* ils procurent une couche de sécurité complémentaire aux portefeuilles logiciels

Cette séparation des rôles est fondamentale pour la sécurité : même si votre ordinateur est compromis, un attaquant ne peut pas signer de transactions sans avoir accès physique au portefeuille matériel et votre approbation active.

**Les 6 règles d'or du cold wallet :**


1. L'achat s'effectue directement chez le fabricant, jamais de tiers entre vous et le concepteur.
2. A réception, toujours inspecter l'emballage qui doit être scellé à un ou plusieurs niveaux.
3. Vérifiez l'authenticité du dispositif et installez vous-même le firmware original du wallet.
4. Seulement deux supports doivent reporter les mots de votre seed phrase : l'écran de votre portefeuille matériel qui listera les mots en principe une seule et unique fois lors de la création du wallet et le support physique où seront inscrit les mots de votre main.
5. Après avoir crée votre portefeuille et avant de l'utiliser toujours tester votre sauvegarde en ré-initialisant le device et en ressaisissant les mots sur le portefeuille matériel lui même.
6. Si votre cold wallet est fonctionnel mais que la seed phrase est perdue ou compromise, créez un nouveau portefeuille avec une nouvelle seed phrase puis transférez les fonds de l'ancien vers le nouveau portefeuille. (exception faite de BitBox02 si la seed est perdue il est possible de la re-visualiser)

**Comparatif de 4 cold wallets :**

| Critère | Trezor Safe 3 Bitcoin-only edition | Ledger Nano S Plus | BitBox02 Bitcoin-only edition | **Tangem** |
|:---|----|----|----|----|
| **Format** | Clé usb | Clé usb | Clé usb | Carte de crédit ou anneau |
| **Matériel et fabricant du secure element (¹)** | Open source sauf le secure element EAL6+ `Infineon Technologies` | Partiellement open source, et non open source pour le secure element EAL5+ `STMicroelectronics` ou EAL6+ `Infineon Technologies` suivant l'année de fabrication. | Qualifiable d'open source car le secure element `Microchip Technology` ne stocke pas les clés privées. | Propriétaire, Secure element EAL6+ `Samsung` |
| **Vérification, installation du firmware et de l'app Bitcoin sur le dispositif - OS supportés** | Utilisation **obligatoire** de [Trezor Suite](https://trezor.io) (logiciel open source, supporte Tor) - Windows / macOS / Linux / Android / iOS (portefeuille en lecture seule) | Utilisation **obligatoire** (²) de [Ledger Live](https://www.ledger.com) (logiciel open source, ne supporte pas Tor) - Windows / macOS / Linux(³) / Android / iOS non compatible | Utilisation **obligatoire** de BitBoxApp (logiciel open source, supporte Tor) - Windows / macOS / Linux(⁴) / Android | Utilisation **obligatoire** de Tangem Crypto Wallet (ios / android) application mobile pour interaction avec la carte ou le ring par NFC. |
| **Support d'un nœud personnel par la suite logicielle du fabricant** | Oui avec le format Electrum SPV en clair, SSL ou TOR. Supporte également Blockbook personnel. | Oui par une connexion RPC à Bitcoin dont le wallet doit être actif. Il est nécessaire d'installer Ledger SatStack, logiciel open-source qui est une passerelle entre Ledger Live et le nœud Bitcoin complet. | Oui avec le format Electrum SPV en clair, SSL ou TOR | Non |
| **Création du wallet et initialisation du PIN** | Electrum, Rabby, BlueWallet ... ou Trezor Suite | Uniquement sur le dispositif. | BitBoxApp obligatoire | Tangem Crypto wallet |
| **Phrase de récupération Single Seed BIP39** | Trezor Suite avec "anciennes sauvegardes" en 12 ou 24 mots mais capacité à restaurer en 18mots. Avec Electrum 12,18 ou 24 mots. | Création en 24 mots uniquement mais capacité à restaurer un portefeuille avec une phrase de récupération de 12, 18 ou 24 mots. | Création en 12 ou 24 mots avec capacité à restaurer un portefeuille avec une phrase de récupération de 12, 18 ou 24 mots. | Le fabricant privilégie un backup dans plusieurs exemplaires du dispositif sans phrase de récupération, il propose néanmoins 12 ou 24 mots. |
| **Vérifier sur le dispositif la phrase de récupération** | Oui | Oui en installant au préalable sur le device l'app Recovery Check à partir de Ledger Live | Oui | Non testé |
| **Re-visualiser la phrase de récupération sur le dispositif (⁵)** | Non | Non | Oui | Non testé |
| **Option portefeuille caché et protégé par une passphrase** | Oui | Oui | Oui | Non |
| **Phrase de récupération autre** | Oui (⁶) | Non | Non | Non |
| **"Pay to" par défaut** | SegWit | SegWit | SegWit | SegWit |
| **Autres "Pay to" supportés par la suite logicielle du fabricant** | Legacy / P2SH-SegWit / Taproot | Legacy / P2SH-SegWit / Taproot | P2SH-SegWit / Taproot (⁷) | Legacy |
| **Wallets tiers supportés** | Electrum, Sparrow, Specter … | Electrum, Sparrow, Specter … | Electrum, Sparrow, Specter … | Aucun |
| **Wallets mobiles supportés (système)** | Trezor Suite Lite (Android) ou version Web de Trezor Suite par WebUSB - Mycelium (iOS / Android) | Ledger Live (Android) | BitBoxApp avec les mêmes fonctionnalités que la version desktop (Android) | Tangem Crypto Wallet (iOS / Android) |
| **Autres fonctionnalités** | Personnalisation écran d'accueil avec image n/b 128x64 pixels | Possibilité de créer un deuxième code PIN lié au portefeuille caché et protégé par une passphrase | Sauvegarde de la seed phrase et des notes de transactions sur microSD. Brouillage du signal usb entre le dispositif et l'appareil auquel il est raccordé. | Non testé |
| **Réinitialisation au paramètres d'usine (effacement de toutes les données utilisateur)** | par Trezor Suite ou sur le dispositif après saisie de 16 PIN erronés ou par saisie d'un PIN dédié (wipe code) à la place du PIN de déverrouillage | Réinitialisation aux paramètres d'usine (⁸) par le Control Center de Ledger Live, sur le dispositif par appui long des 2 boutons (settings) ou après la saisie de 3 PIN erronés. | Réinitialisation aux paramètres d'usine avec BitBoxApp ou après la saisie sur le dispositif de 10 PIN erronés | Uniquement avec Tangem Crypto Wallet |
| **Multi-portefeuille** | Oui (⁹) | Oui (⁹) | Oui (⁹) | Oui (⁹) |
| **Entreprise - Pays d'origine** | [SatoshiLabs](https://trezor.io) - République tchèque | [Ledger SAS](https://www.ledger.com) - France | **[Shift Crypto](https://bitbox.swiss/) -** Suisse | [Tangem AG](https://tangem.com) -Suisse |
| **Achat du dispositif** | Fiat ou Bitcoin | Fiat | Fiat ou Bitcoin | Fiat ou Bitcoin |

(¹) à l'heure actuelle il n'existe pas de "secure element" open source, en principe c'est dans cette puce sécurisée que les clés privées sont stockées.

(²) Pas tout à fait, il existe une alternative avec [Bacca](https://github.com/darosior/ledger_installer).

(³) Après avoir déverrouillé le dispositif et lancé l'app Bitcoin, sous Linux il sera parfois nécessaire de fixer les règles UDEV par : `wget -q -O - https://raw.githubusercontent.com/LedgerHQ/udev-rules/master/add_udev_rules.sh | sudo bash` (comme indiqué sur le portail Ledger)

(⁴) **Bien que non explicite sur le site du fabricant**, sous Linux il sera parfois nécessaire de fixer les règles UDEV avec un script téléchargeable [ici](https://github.com/BitBoxSwiss/bitbox-wallet-app/blob/master/frontends/qt/resources/deb-afterinstall.sh), si vous utilisez seulement BitBox02 commentez les deux lignes sous `# BitBox V1 udev rules` rendez le exécutable par `chmod u+x deb-afterinstall.sh` puis lancez le par `sudo ./deb-afterinstall.sh`. Heureusement que certains se sont donné la peine avec [ceci](https://github.com/spesmilo/electrum/tree/master/contrib/udev) ou est répertorié les règles UDEV pour la majorité des hardwares wallets. Pour lister toutes les règles que vous avez mis en place : `sudo ls /etc/udev/rules.d/`

(⁵) BitBox02 est le seul dispositif que j'ai testé qui permet de voir plus d'une fois la seed phrase :

* avantage, si la seed phrase est perdue il est possible de la ré-écrire sur un support physique ou de la sauvegarder à nouveau sur micro SD.
* inconvénient, si une personne mal intentionnée à accès au dispositif et qu'elle connait le code PIN, en dehors du vol direct comme sur tous les autres dispositifs il faut considérer l'effet pervers du temps que cela peut engendrer. En effet cette personne sera en mesure de créer son backup personnel et de déplacer les fonds plus tard …

(⁶) Autres phrases de récupération sur Trezor wallet :

* Trezor Suite propose SLIP39 Shamir en 20 mots uniquement, avec Single-Share Backup (1 seul fragment) ou du Multi-Share Backup (2 à 16 fragments), il est possible de produire le Backup Multi-Share avec Single-Share. Les deux seront valables, à l'utilisateur de détruire ensuite le Single-Backup.
* Electrum est capable de gérer un portefeuille avec un dispositif initialisé (vérification, installation du firmware et du micrologiciel Bitcoin avec Trezor Suite car à la livraison le matériel est en "bootloader mode"). Il peut initier la procédure pour nommer l'appareil et activer son code Pin. En plus de BIP39 et avec "Show expert settings" Electrum propose la seed SLIP39 Shamir ou Super Shamir avec 20 ou 33 mots. Si vous êtes vraiment un expert, est proposé en plus : avec la phrase supplémentaire (hidden wallet) ou le Seedless Mode (master seed sans backup). Une fois le portefeuille ouvert dans Electrum vous aurez accès à presque toutes les fonctionnalités fournies par le dispositif en cliquant sur l'icône dans le coin inférieur d'Electrum.

(⁷) Les autres "Pay to" sont supportés avec un portefeuille tiers tels que Sparrow ou Electrum.

(⁸) Effacement du Ledger Nano S+ : la ré-initialisation efface tout sauf le firmware, il est ensuite nécessaire de ré-installer l'app Bitcoin avec Ledger Live.

(⁹) En utilisant des chemins de dérivation ou des `Pay To` différents, il est possible de créer de multiples portefeuilles avec la même graine maîtresse (master seed), donc la même liste de mots. Attention il est préférable de noter les choix effectués puisque cela complexifie la gestion et la récupération en cas de problèmes.

*Sauf pour Tangem, il est possible de restreindre l'usage des suites logicielles fournies par les fabricants à la vérification, la mise à jour du firmware et à l'installation du micro logiciel. Avec Trezor Safe 3 ou Ledger Nano S Plus vous n'avez pas besoin de créer de compte (création du portefeuille et génération des adresses) dans la suite logicielle. BitBox02 nécessite l'utilisation de BitboxApp pour restaurer ou créer le portefeuille en générant la phrase de récupération, ensuite il est possible d'utiliser le dispositif avec le portefeuille logiciel de votre choix pourvu qu'il soit compatible.*

# Annexes

## 10mn : 1 bloc

La blockchain augmente immuablement de 144 blocs par jour et de 52596 en moyenne par an, la [taille des blocs](https://www.blockchain.com/fr/explorer/charts/blocks-size) est variable et fonction des types de transactions inclues dans celui-ci, de l'activité et de l'usage du réseau. Si je prends 2 Mio par bloc cela donne 103 Gio de + par an. Mon NVMe.M2 de 2 To sera approximativement blindé en 2036. Les capacités et les vitesses de transfert des stockages de masse n'auront de cesse de s'améliorer dans le futur, il est envisageable que le code de Bitcoin s'adaptera à nouveau.

## Copie directe de la blockchain

Si vous avez un accès physique à un autre nœud de confiance vous pouvez copier sa blockchain au lieu de la télécharger des pairs. Normalement située dans `/home/bitcoin_user/.bitcoin` , vous devrez copier récursivement les répertoires `blocks` et `chainstate` (`indexes` est facultatif car le nœud les re-construira à partir de ce que vous avez paramétré dans `bitcoin.conf`). Le préalable est bien sûr de stopper `bitcoind` le temps de la sauvegarde (ou si btrfs sans arrêter le daemon avec un snapshot).

Copier 700 Gio de données sur un support externe n'est pas anodin, par un port usb 3.0 j'ai eu deux expériences : une de 5h avec un débit de 38 Mo/s sur je sais plus quoi et une autre de 30 minutes avec un débit de 385 Mo/s sur un bon adaptateur usb 3 pour NVMe.

IMPORTANT : il est indispensable de noter la version de Bitcoin du nœud qui a fourni les blocs. Par exemple une sauvegarde de blockchain Bitcoin Core v28.0 ne peut pas être restaurée sur un nœud Bitcoin Core v27.2, il faudra d'abord passer le nœud en V28.0 afin d'éviter une longue ré-indexation causé par le message "Corrupted block database detected. Please restart with -reindex or -reindex-chainstate to recover".

Une fois copié, si besoin régler les droits d'accès utilisateur et groupe au user qui lance `bitcoind` et les permissions à `-rw-------` en effectuant :

```bash
sudo find ~/.bitcoin/chainstate -type d -exec chmod 700 {} \;
sudo find ~/.bitcoin/chainstate -type f -exec chmod 600 {} \;
sudo chown -R btc-node ~/.bitcoin/chainstate
sudo chgrp -R btc-node ~/.bitcoin/chainstate
```

```bash
sudo find ~/.bitcoin/blocks -type d -exec chmod 700 {} \;
sudo find ~/.bitcoin/blocks -type f -exec chmod 600 {} \;
sudo chown -R btc-node ~/.bitcoin/blocks
sudo chgrp -R btc-node ~/.bitcoin/blocks
```

Faire de même sur `indexes` si vous avez copié toutes les données de ce répertoire.

Démarrez le nœud et observez les logs, après certaines vérifications, si tout se passe bien, il va aller chercher les blocs manquants pour se synchroniser avec les autres pairs.

Plus d'information sur les fichiers et répertoires du nœud Bitcoin : [toute la doc](https://github.com/bitcoin/bitcoin/blob/master/doc/files.md) du "file system".

## Nœud public / privé

Un nœud est opérationnel seulement s'il est à jour, si l'on souhaite effectuer des tests tout azimut, un nœud privé peut s'avérer utile. En effet cela permet d'isoler le nœud de test des autres, il ne divulguera pas d'informations à ses pairs, vous pourrez le redémarrer à volonté et effectuer des choses sensibles en toute discrétion. C'est très bien pour apprendre.

* le public est connecté avec ses pairs à travers une couche d'anonymisation (Tor / I2P ) , il fonctionne 7j / 7 en continu. IPv4 n'est utilisé que sur le réseau local pour mettre à jour le nœud privé.
* le privé fonctionne sur une machine distincte reliée au nœud public par réseau local, c'est le seul pair avec qui il dialogue. En résumé, il ne fait que mettre à jour sa copie de la blockchain afin de rester opérationnel.

Afin de mener cela à bien, j'ai compilé puis installé sur mon Desktop Linux Bitcoin avec interface graphique, j'ai ensuite restauré une copie de la blockchain que j'avais sous la main avec les deux répertoires `blocks` `chainstate`. J'ai paramétré `bitcoin.conf` pour que ce nœud soit privé. Au lancement, `bitcoin-qt` se connecte au nœud public pour mettre à jour la copie de la blockchain. N'espérez pas que le nœud public envoie les blocs vers le privé à travers le réseau local à la vitesse grand V, les nœuds sont codés pour échanger les blocs entre eux à une cadence suffisante pour diffuser l'information correctement. Si vous souhaitez télécharger rapidement la blockchain d'un nœud à l'autre par votre réseau local, stoppez les nœuds et utilisez `scp` dans un terminal. Pour information un réseau local 1 Gb/s donne un transfert d'environ 115 Mio/s (700 Gio = 1h45).

Configuration du nœud public **par ajout de ceci à la fin** de `bitcoin.conf`

```bash
# Configuration du noeud public
#
# Le noeud est deja parametre pour echanger avec ses pairs par TOR ou I2P EXCLUSIVEMENT
# Correspond a tous les parametres actifs de "Confidentialite avancee" 
# Il permet la mise a jour du ou d'autres noeuds connectes au reseau local IPV4

# Remplacez X par la valeur que vous utilisez sur votre LAN
# Exemple : 192.168.0.0/24 autorise 256 adresses de (192.168.0.0 à 192.168.0.255)
whitelist=192.168.X.0/24        # Autorise seulement la plage d'IP(v4) du reseau local
rpcallowip=192.168.X.0/24       # Limite l'acces RPC bitcoind au reseau local

# Commentez le "bind" defini plus haut pour n'avoir que celui ci
bind=0.0.0.0                    # Ecoute sur toutes les interfaces reseau
```

Configuration du privé

```bash
# Ce fichier de configuration doit etre dans le chemin
# ~/.bitcoin/bitcoin.conf
#
# Configuration en noeud prive sur reseau local (LAN)
# Met à jour sa blockchain uniquement avec le noeud public
# Ne dialogue pas avec les pairs d'internet
#

listenonion=0    # De-active Tor

# Commentez si utilisation exclusive de la console de "bitcoin-qt"
server=1  # Accepte les connexions et commandes RPC de "bitcoin-cli"

# Remplacez X par la valeur que vous utilisez sur votre LAN
# Exemple : 192.168.0.0/24 autorise 256 adresses de (192.168.0.0 à 192.168.0.255)
rpcallowip=192.168.X.0/24       # Limite l'acces RPC bitcoind au reseau local

# Adresse IP du noeud public a laquelle se connecter et
# tenter de maintenir la connexion ouverte.
# Cette option peut être définie plusieurs fois si vous avez plusieurs noeuds publics
# Il n'y aura pas de connexions à d'autres noeuds que ceux définis ici.
connect=192.168.X.X

# Pour information la commande "addnode" ajoute la connexion au noeud de
# maniere permanente tout en maintenant les connexions sortantes automatiques.
# Ne pas utiliser cette commande dans ce cas d'usage

# BIP158 : Index de filtres de blocs
# bitcoind va creer cet index s'il est absent, observer les logs
# Ensuite le wallet de bitcoind parcours environ 5 a 10x plus rapidement la blockchain
# a la recherche de transactions sur une adresse.
# Les logs indiqueront "fast variant using block filters" a l'oppose de
# "slow variant inspecting all blocks" si l'index n'est pas construit
blockfilterindex=1

# Pour information, le filtre txindex ne sert a rien dans ce cas de figure.
```

*Comment créer un portefeuille en lecture seule avec une adresse de mineur sur le nœud privé ?*

Bien que tout soit réalisable avec `bitcoin-cli` en ligne de commande, ici j'utilise la console de `bitcoin-qt` pour ses fonctionnalités supplémentaires, comme l'horodatage et la répétition des commandes.

**Créer un wallet "miner_wallet" dans le répertoire Test avec "disabled private keys"**

(équivalent de `bitcoin-cli createwallet "Test/miner_wallet" true false ""` )

Un chemin est présent dans le nom, ce portefeuille sera dans `~/.bitcoin/Test/miner_wallet`

Ensuite dans "Window / Console", saisir :

* `getdescriptorinfo "addr(3Awm3FNpmwrbvAFVThRUFqgpbVuqWisni9)"`
* copier la somme de contrôle "checksum", ici `j8r6vh28`, puis saisir :
* `importdescriptors '[{"desc": "addr(3Awm3FNpmwrbvAFVThRUFqgpbVuqWisni9)#j8r6vh28", "timestamp": "now", "label": "mineur"}]'` (champ label facultatif)
* attendre que la sortie indique `[{"success": true}]`
* `timestamp : now` indique que la blockchain sera parcourue à la recherche de transactions sur cette adresse au lancement de cette commande sur environ 2 heures soit 12 blocs et cela avec ou sans index défini (`blockfilterindex`).
* pour voir toutes les transactions du mineur à cette adresse, effectuer ceci :
  * l'adresse a été utilisée pour la première fois le 30-06-2023 au bloc `796537`
  * nous sommes le 27 janvier 2025 à 22h00, le dernier bloc est le `881098`
  * lancer la recherche par : `rescanblockchain 796537 881098`
* observer les log du nœud, 84 561 blocs seront parcourus entre 1min et 2min et jusqu'à une heure en fonction du débit du support de masse qui héberge la blockchain et de la présence de l'index `blockfilter`
* par l'ajout de `walletnotify` dans `bitcoin.conf` toute nouvelle transaction exécutera un script ou une commande : `walletnotify=xmessage "Funds had been moved: %s" -center`, `%s` sera remplacé par l'id de la transaction.

pour supprimer le portefeuille "miner_wallet"

* décharger le portefeuille, saisir dans la console `unloadwallet miner_wallet`
* fermez `bitcoin-qt`, puis dans `~/.bitcoin`
  * supprimez le répertoire portant le nom `miner_wallet`
  * éditez le fichier `settings.json` et supprimez la référence à `miner_wallet`

Pour aller plus loin vous trouverez sur le net tout ce qu'il faut pour créer un wallet avec clé privée sur votre nœud privé (lui aussi). Vous pouvez générer avec un navigateur en mode hors ligne clés privées et adresses en chargeant la page "BIP39 tool" à partir de votre disque dur.

## Seed phrase 12 ou 24 mots

| **Face à** | **Phrase de 12 mots** | **Phrase de 24 mots** |
|----|----|----|
| Sécurité face au quantique | Hors de portée, donc suffisante en l'état actuel des perspectives. | 24 mots donnent une marge sans discussion. |
| Praticité | Moins de mots c'est moins d'erreurs lors d'une restauration, moins d'efforts pour graver sur support métallique. | 2 fois plus longue à graver et à restaurer. |
| Confidentialité | Discrète à stocker ou partager en cas de besoin, exemple gravure sur métal compacte. | Difficile à brute-forcer si partiellement compromise, 24 mots protègent mieux que 12 contre les fuites partielles comme une courte exposition visuelle. |
| Échappatoire | Mémorisable par un individu | Plus difficilement mémorisable par un individu |
| Résistance à un oppresseur | Si forcée à divulgation, une phrase de 12 mots est vulnérable à une reconstruction partielle. | Plus complexe à retenir, potentiellement plus long à la divulgation augmentant le risque de capitulation de l'oppresseur. Reconstruction partielle plus délicate. |

# Les mises à jour

## maj du système d'exploitation

Ouvrez un terminal pour accéder au nœud, puis :

```bash
sudo apt-get update
sudo apt-get upgrade
```

Si nécessaire redémarrez le système par `sudo reboot`

## Snapshot des données

Si les données blockchain sont hébergées sur un stockage de masse formaté BTRFS, effectuer un snapshot peut s'avérer utile avant une mise à jour de `bitcoind`. Un snapshot read-only permet de sauvegarder l'état de la blockchain avant chaque mise à jour et d'effectuer un rollback si nécessaire. Il est possible de faire pas mal de choses avec les snapshots, comme copier la blockchain jusqu'à la date du snapshot sur un autre support de stockage sans arrêter le daemon `bitcoind` ; ou encore utiliser la copie blockchain de votre nœud avec un snapshot writable destiné à faire tourner un autre nœud par exemple en mode graphique sur votre PC. (montage dans le filesystem de votre PC avec SSHFS (SSH File System) `sshfs utilisateur@hote_distant:/chemin/distant ~/remote_mount`)

Voici quelques commandes et cas d'usages :

```bash
# Afficher l'espace libre / utilise
sudo btrfs filesystem usage /mnt/btrfs/bitcoin

# Lister les subvolumes et les snapshots
sudo btrfs subvolume list -t /mnt/btrfs/bitcoin

# Creer un READ-ONLY snapshot de la blockchain Bitcoin (subvolume snapshots deja existant)
sudo systemctl stop bitcoin.service   # Stoppez le demon bitcoind
# Attendre obligatoirement l'invite !
# afin que bitcoind flush (parfois plusieurs Go) du cache UTXO en RAM vers le disque
sudo btrfs subvolume snapshot -r /mnt/btrfs/bitcoin /mnt/btrfs/snapshots/bitcoin_2025-12-14_RO
sudo systemctl start bitcoin.service   # Demarrer le demon bitcoind

# Voir le contenu du snapshot
ls -lh /mnt/btrfs/snapshots/bitcoin_2025-12-14_RO

# Copier un snapshot, c'est copier les données de l'image figee d'un sous-volume
# a une date donnee sur un autre support de masse au format BTRFS
# par exemple une clef USB montee dans le systeme de fichier, ici en /mnt/usb
# Pas besoin de stopper bitcoind vu que c'est un snapshot
sudo btrfs send /mnt/btrfs/snapshots/nom-du_snapshot_date | sudo btrfs receive /mnt/usb

# Afficher l'espace libre / utilise sur la clef usb
sudo btrfs filesystem usage /mnt/usb

# Effacer un snapshot
sudo btrfs subvolume delete /mnt/btrfs/snapshots/nom-du-snapshot
```

A partir d'un snapshot **read-only** revenir à un état précédent (rollback)

```bash
# /!\ ATTENTION /!\ c'est destructif pour les changements post-snapshot
# Ne pas oublier le wallet du noeud s'il est utilisé

# Arreter TOUS les sevices qui utilisent des sous volumes btrfs  
sudo systemctl stop electrs.service            # Stoppez electrs
sudo systemctl stop bitcoin.service            # Stoppez le demon bitcoind
sudo btrfs subvolume list /mnt/btrfs/bitcoin   # Identifier le snapshot

# Demonter TOUS les sous volumes
sudo btrfs subvolume list -t /mnt/btrfs/bitcoin    # Liste d'abord tous les subvolumes
sudo umount /mnt/btrfs/bitcoin
sudo umount /mnt/btrfs/electrs
sudo umount /mnt/btrfs/snapshots

# Montez temporairement le volume RACINE btrfs sans l'option subvol 
sudo mkdir /mnt/btrfs_root                           # si pas deja cree
sudo mount /dev/nvme0n1p1 /mnt/btrfs_root            # Pour moi c'est nvme0n1p1
sudo btrfs subvolume delete /mnt/btrfs_root/@bitcoin # Efface ... puis remplace ci-dessous
sudo btrfs subvolume snapshot /mnt/btrfs_root/@snapshots/bitcoin_2025-12-14_RO /mnt/btrfs_root/@bitcoin
sudo umount /mnt/btrfs_root    # Demonter        
sudo mount -a                  # Remonter les volumes listés dans fstab
```

## maj de `bitcoind`

Si vous avez utilisé le portefeuille de votre nœud, il est recommandé de sauvegarder le répertoire contenant le ou les wallets avant de mettre à jour. Si btrfs effectuer un snaphot read-only de la blockchain est aussi une bonne idée. Ouvrir un terminal afin d'accéder au nœud puis afficher la version actuelle avec la commande `bitcoin-cli -version` . Ensuite consulter la page des releases sur le repository, il est utile de lire les notes de la nouvelle version vers laquelle vous mettez à jour (et celles des versions que vous avez sautées) s'il y a des fonctionnalités / options qui vous intéressent ou que vous allez devoir gérer. Il est bon de préciser que l'on va toujours de l'avant avec les versions. Si vous devez aller en arrière lisez impérativement les releases notes. Ainsi si vous avez construit votre nœud avec Bitcoin Core v28.0 sans `blocksxor=0` dans `bitcoin.conf` puis téléchargé toute la blockchain n'installez pas une v27.x car elle ne saura pas lire la blockchain puisque le brouillage est venu avec la v28.0 ! 

Si vous lisez la documentation concernant le cycle de vie de Bitcoin Core, vous constaterez qu'il est le suivant :

* 2 versions majeures par an
* des versions mineures qui reportent d'importantes corrections de bogues des versions majeures récentes
* les corrections de bogues ne seront reportées que sur les 18 derniers mois de version car cela nécessite trop de ressources de maintenance

Ainsi, il n'est pas considéré comme sûr d'utiliser des versions de Bitcoin Core datant de plus de 18 mois, car elles peuvent contenir des vulnérabilités non corrigées. Si un nombre important de nœuds utilisent des logiciels très anciens et non corrigés, cela peut représenter un risque systémique pour le réseau.

Un aspect fondamental du modèle de sécurité de Bitcoin est que le logiciel du nœud n'est PAS mis à jour automatiquement. Les mises à jour automatiques créeraient un point de défaillance unique qui pourrait permettre à une mise à jour malveillante de se propager rapidement à une super-majorité de nœuds sur le réseau.

Bon, tout cela est de la belle littérature : en définitive faites ce que bon vous semble, le joyeux bazar est aussi un modèle de sécurité. [Historique des mises à jour logicielles des nœuds.](https://blog.lopp.net/when-do-bitcoin-node-operators-upgrade/)

```bash
# Se positionner dans le répertoire du code source de bitcoin
cd ~/code/bitcoin

# Si vous passez a autre chose que Bitcoin Core ou decidiez d'y revenir
# renommez le repertoire actuel. Construire toujours dans le repertoire
# bitcoin et une bonne maniere d'operer.

# Mettre à jour le code source
git pull

# Afficher les versions
git tag
```

**Ici pour l'exemple, je choisis la "Core v29.0"**

Indispensable : vérifier que la "release" que vous avez sélectionnée est signée, avant de compiler :

```bash
# Cles des mainteneurs : depot guix.sigs, distinct du depot du code (voir compilation initiale)
git -C ~/code/guix.sigs pull
gpg --import ~/code/guix.sigs/builder-keys/*.gpg
git verify-tag "v29.0"
# Vous devez obtenir en sortie : Good signature from ... puis comparer l'empreinte
# "Primary key fingerprint" avec celle du mainteneur (voir compilation initiale)
# "BAD signature" ou "No public key" : NE PAS compiler.
```

Poursuivre avec

```bash
git checkout tags/v29.0

Pour la construction voir compilation initiale plus haut, faire le distinguo version.
A partir de la v29 ce n'est plus Autotools mais CMake qui est utilisé
Compiler ici ...
```

Le nouveau binaire est construit, on continue

```bash
# Afin de vérifier l'arrêt complet de bitcoind qui est parfois long
# ouvrir un terminal pour observer les logs
tail -f ~/.bitcoin/debug.log

# Dans un autre terminal stopper electrs si lancé et bitcoind
sudo systemctl stop electrs.service
sudo systemctl stop bitcoin.service

# Pour bitcoin vous avez vu "Shutdown: done" dans les logs ? Poursuivez
# sinon attendez parfois qq min

# Se positionner dans le répertoire du code source de Bitcoin
cd ~/code/bitcoin

# Installer la nouvelle version selon le cas :
sudo make install            # Avant la v29
sudo cmake --install build   # A partir de la v29

# Lancer bitcoind au premier plan
bitcoind -daemon=0

# Regarder ce qui se passe en visualisant la sortie ... ok ou pas ?
# Quoi qu'il en soit arreter bitcoind en tapant CTRL-C 

# Ce n'est pas bon ...
# si besoin de re-démarrer la machine il faudra préalablement arrêter les services par 
sudo systemctl disable electrs.service
sudo systemctl disable bitcoin.service
# investiguez le problème ... puis faites la commande inverse avec enable

# Tout est correct ? Ok, relancer bitcoind avec le service 
sudo systemctl start bitcoin.service

# bitcoind est synchronisé avec ses pairs ? Oui alors relancer electrs
sudo systemctl start electrs.service
```

## **maj d'Electrs**

Ouvrir un terminal afin d'accéder à votre nœud puis afficher la version actuelle avec la commande `electrs --version` ensuite consulter la [page de publication d'Electrs](https://github.com/romanz/electrs/releases) pour voir si une version plus récente est disponible.

Une nouvelle version est disponible et vous voulez effectuer la mise à jour, suivez ceci :

```bash
# Se positionner dans le répertoire du code source d'Electrs
cd ~/code/electrs

# Afficher l'actuelle dernière étiqutte de version
git tag | sort --version-sort | tail -n 1

# Nettoyer le code source local
git clean -xfd

# Mettre à jour
git fetch

# Après mise à jour, afficher la dernière étiquette de version
git tag | sort --version-sort | tail -n 1
# Octobre 2026 : "v0.12.0", qui exige Bitcoin Core 31+ et rest=1 dans bitcoin.conf
# Sinon (Core 30 ou moins, Knots) choisissez "v0.11.1"

# OBLIGATOIRE : verifier que la release que vous allez compiler est originale
# v0.11.1 et plus : signature SSH (fichier allowed_signers cree a l'installation)
git -c gpg.ssh.allowedSignersFile=$HOME/.ssh/allowed_signers_electrs verify-tag "v0.12.0"
# Sortie attendue : Good "git" signature for me@romanzey.de with ED25519 key SHA256:GifMn7F2swVKyn6MewbQHrYCs4i/bPK7gnwxhuPz/YA
# Jusqu'a v0.11.0 : signature GPG, git verify-tag "v0.11.0"
# Sortie attendue : Good signature from "Roman Zeyde <me@romanzey.de>"

# Choix de la release
git checkout "v0.12.0"

# Nettoyage
cargo clean

# Construction (link dynamique de librocksdb 9.10.0)
ROCKSDB_INCLUDE_DIR=/usr/include ROCKSDB_LIB_DIR=/usr/lib cargo build --locked --release
```

**Passage à la v0.11 ou à la v0.12** : chacune change le format de l'index, Electrs réindexe tout au premier lancement (plusieurs heures pendant lesquelles il ne répond plus). Prévoir au moins 120 Go libres pour la v0.12. Avant d'installer la v0.12 : mettre à jour le nœud en Bitcoin Core 31 ou plus, ajouter `rest=1` dans `bitcoin.conf` puis redémarrer `bitcoind`, et supprimer `daemon_p2p_addr` de `electrs.toml`. L'ancien binaire `electrs-old` ne sait pas relire le nouvel index : un retour en arrière impose une nouvelle réindexation (ou la restauration d'un snapshot BTRFS de `@electrs`).

Attendre que le nouveau binaire soit disponible puis afin de vérifier le déroulement, ouvrir un nouveau terminal puis observer les logs par : `sudo journalctl -f | grep electrs`

Arrêter `electrs` par `sudo systemctl stop electrs.service`

Sauvegarder l'ancien binaire et installer le nouveau en mode manuel :

```bash
sudo cp /usr/local/bin/electrs /usr/local/bin/electrs-old
sudo cp ~/code/electrs/target/release/electrs /usr/local/bin/electrs
sudo chown root /usr/local/bin/electrs
sudo chgrp root /usr/local/bin/electrs
```

Relancer `electrs` par `sudo systemctl start electrs.service`

Observer les logs, tout est correct ? alors fermer les terminaux.

## maj Appimage sur le Desktop Linux

Effectuez une copie de sauvegarde de l'Appimage avant une mise à jour. Si la fonctionnalité de mise à jour est disponible dans l'application elle même, procédez et bien qu'en principe il n'y ai plus besoin de vérifier l'authenticité et l'intégrité de l'Appimage, il est quand même souhaitable de vérifier au moins l'intégrité. Sinon téléchargez la nouvelle version, vérifiez son authenticité, puis installez comme pour la première fois, pour finir mettez à jour le nom dans le lanceur si vous en avez crée un la première fois. N'oubliez pas de rendre la nouvelle Appimage exécutable. Les paramètres utilisateur sont en principe à l'abri puisque séparés dans des répertoires `~/.nom_d-appimage` ou dans `~/.config/nom_d-appimage`

Exemple avec Trezor-suite :

```typescript
# Telechargez d'abord la signature
wget https://trezor.io/security/satoshilabs-2021-signing-key.asc

Importez la signature
gpg --import satoshilabs-2021-signing-key.asc

# Telechargez Trezor-Suite-X.x.x-linux-x86_64.appimage.asc
# Telechargez Trezor-Suite-X.x.x-linux-x86_64.appimage
# Les noms doivent correspondre exactement !
# En sortie vous devez obtenir : Bonne signature de « SatoshiLabs 2021 Signing Key »
gpg --verify Trezor-Suite-X.x.x-linux-x86_64.appimage.asc

# Rendre executable
chmod +x Trezor-Suite-X.x.x-linux-x86_64.appimage

# Tester en visualisant les logs dans le terminal
./Trezor-Suite-X.x.x-linux-x86_64.appimage

# Extraire le contenu de l'appimage dans un répertoire à partir duquel vous obtiendrez
# l'icone de l'app entre autres ...
./Trezor-Suite-X.x.x-linux-x86_64.appimage --appimage-extract
```

# Glossaire

## 1Mio

Le mébioctet permet de quantifier la taille que des données occupent sur un support de stockage. Le bit, qui est contraction de **binary digit,** est la plus petite unité de données dans un ordinateur, qui peut avoir une valeur de 0 ou 1, éteint ou allumé, off / on. Ensuite vient l'**octet** (`oct` pour 8, `et` pour petit) qui rassemble 8 bits. A partir de là, **le passage d'une grandeur à l'autre est toujours x 1024** :

* 1 Kio (KiB ) (kibioctet) : 1024 octets / 2¹⁰ octets ou de 00 0000 0000 à 11 1111 1111
* 1 Mio (MiB) (mébioctet): 1024 Kio / 2²⁰ octets
* 1 Gio (GiB) (gibioctet): 1024 Mio / 2³⁰ octets
* 1 Tio (TiB) (tébioctet) : 1024 Gio / 2⁴⁰ octets

La question qui tue, pourquoi x 1024 et pas x 1000 ?

* Les ordinateurs fonctionnent sur un système binaire, qui utilise des puissances de 2.
* 1024 est une puissance de 2, ce qui en fait une valeur naturelle dans le contexte binaire.
* choisir 1000 aurait cassé la facilité de compréhension et de manipulation de ces ^2.

*En informatique, l'usage historique, enseigné partout et toujours courant, est **1 Ko = 1 024 octets**. Mais « kilo » signifie 1 000 dans le Système international, et les fabricants de disques l'utilisent dans ce sens. Pour lever l'ambiguïté, la **CEI a créé en 1998** des préfixes binaires : Kio, Mio, Gio, Tio (KiB, MiB, GiB, TiB en anglais) valent 1 024, 1 024², 1 024³, 1 024⁴ octets, tandis que ko, Mo, Go, To valent 1 000, 1 000², 1 000³, 1 000⁴. Pour résumer : avant 1998 1Ko = 1024 octets, après 1998 1Ko = 1000 octets !*

## Base 16

Appelé plus souvent hexadécimal ou hexa, il est défini par 16 symboles différents, de 0 à 9 et de A à F. Représenter des 0 1 est très vite fastidieux, commençons par cela :

| **je compte en binaire sur 8 bits ou 1 octet** | **je compte en hexadécimal** |
|----|----|
| 0000 0000 | 0 |
| 0000 0001 | 1 |
| 0000 0010 | 2 |
| 0000 0011 | 3 |
| 0000 0100 | 4 |
| 0000 0101 | 5 |
| 0000 0110 | 6 |
| 0000 0111 | 7 |
| 0000 1000 | 8 |
| 0000 1001 | 9 |
| 0000 1010 | A |
| 0000 1011 | B |
| 0000 1100 | C |
| 0000 1101 | D |
| 0000 1110 | E |
| 0000 1111 | F |
| 0001 0000 | 10 |
| 0001 0001 | 11 |
| … cut | … le tableau serait trop long |
| 1111 1111 | FF |

Pour l'exemple prenons une donnée de 32 bits de large, si tous les bits sont "On" donc à 1 et que je découpe en paquets de 4 bits, cela donne `1111 1111 1111 1111 1111 1111 1111 1111` et en hexa ? c'est juste trop facile puisque 1111 = F donc cela donne FFFF FFFF .

L'index de transaction txid de Bitcoin est codé en hexa, exemple :

`277944f4de6401f06bd38a39e2b9bae6240e6d8e1a0da1f3206bc3b8bbdc797e`

## Base58

Est un système de représentation efficace et surtout compact de données binaires (base 2 qui utilise 0 et 1) en une chaîne de caractères alphanumériques. Avec base 58 il est nécessaire de définir 58 caractères différents, par convention est employé les chiffres de base 10 + l'alphabet majuscule + l'alphabet minuscule - les caractères ambigus `0 O l I`soit : `123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz`

Base58 est utilisée pour les adresses Bitcoin et les clés privées.

La variante Base58Check inclut une somme de contrôle pour vérifier l'intégrité des données afin de prévenir les erreurs de saisie.

Le cerveau humain n'est pas habitué à manipuler des données en base 58, une machine programmée par un cerveau humain elle en est capable.

Exemple d'adresse : `1NEU779yvLaFk39k4Q3QdLjwpWTdWCbzqL`

## Bech32

Afin d'améliorer les points faibles de base58 (jeu de caractères, algorithme de somme de contrôle, production de QR codes), un autre système d'encodage des adresses a été développé ([BIP173](https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki)). Il utilise 32 caractères, les chiffres base 10 + l'alphabet minuscule - les caractères ambigus `1 b i o` soit :

`023456789acdefghjklmnpqrstuvwxyz`

Bech32 est apparu et est utilisé pour les adresses Bitcoin SegWit native.

Exemple d'adresse : `bc1qc7slrfxkknqcq2jevvvkdgvrt8080852dfjewde450xdlk4ugp7szw5tk9`

## Bech32m

Utilise les mêmes caractères que bech32 pour sa représentation mais sous une organisation différente qui [offre une meilleure sécurité que le schéma original de bech32](https://github.com/bitcoin/bips/blob/master/bip-0350.mediawiki).

Bech32m est utilisé pour les adresses Bitcoin Taproot.

Exemple d'adresse : `bc1p0xlxvlhemja6c4dqv22uapctqupfhlxm9h8z3k2e72q4k9hcz7vqzk5jj0`

/!\\ **N'envoyez jamais de fonds à cette adresse, ni à aucune adresse ou seed phrase d'exemple de ce document.** Cette adresse est un vecteur de test de la [BIP350](https://github.com/bitcoin/bips/blob/master/bip-0350.mediawiki) avec une particularité : une fois décodée, la clé publique qu'elle contient (`79be667e...16f81798`) est la coordonnée x du point générateur G de la courbe secp256k1. Or la clé publique d'une clé privée k vaut k x G : celle de cette adresse correspond donc à la clé privée **1**, connue de tous. Des robots surveillent ce genre d'adresses et vident immédiatement tout fonds qui y arrive. /!\\

## BIP

Une BIP (Bitcoin Improvement Proposal) est une proposition d'amélioration de Bitcoin, cela se traduit par un document de conception fournissant des informations à la communauté ou décrivant une nouvelle fonctionnalité, ses processus ou son environnement. Liste des BIP [ici](https://github.com/bitcoin/bips) ou [là](https://bips.dev).

## BIP39

Le cerveau humain n'étant pas à l'aise face à des clés cryptographiques des individus se sont penchés sur le problème. Cela a donné BIP 39 qui définit une méthode pour passer de 12, 15, 18, 21 ou 24 mots extraits d'une liste prédéfinie de 2048 mots anglais (¹) à une graine cryptographique unique. Cette fonction est à sens unique, il n'est pas possible de retrouver les mots d'après la graine. Si je génère aléatoirement avec 12 mots j'obtiens `party duck tribe color model help bulk ghost vehicle loud sketch visual` , la graine BIP39 correspondante est :

`f0c594301c50e67b02dc14b34e69b7d070a2d524878528f4a3777d0a266291a70f3bdad261c571ef93997e3eb9d38ab6aeca814a3533575b1575ad10f404e05c`

(¹) cette liste de 2048 mots existe dans d'autres langues mais leur utilisation est déconseillée pour éviter des problèmes d'incompatibilités.

## BIP32 / BIP44

A partir de la graine BIP39, BIP32 permet de dériver les adresses ainsi que les clés privées de manière déterministe. BIP44 étend cette architecture en définissant une structure hiérarchique standard permettant de gérer actifs cryptographiques et comptes à partir d'une seule phrase mnémonique. Pour résumer, BIP39 fournit la graine via une phrase mnémonique, BIP32 permet la dérivation hiérarchique des clés, et BIP44 organise cette dérivation pour une utilisation multi-actifs et multi-comptes. Ensemble, ils forment le standard des portefeuilles déterministes modernes. Toutes ces fonctions sont à sens unique.

## BIP32 Root fingerprint

Ce terme est parfois rapporté par les portefeuilles, explications :

* Root fingerprint est une partie de l'empreinte de la clé publique maîtresse.
* BIP32 Root fingerprint agit comme un identifiant partiel qui est suffisant pour distinguer les portefeuilles dans de nombreux contextes, sans pour autant révéler la clé publique complète permettant de voir l'intégralité de l'historique des transactions.

## Blockbook

Est un service backend utilisé principalement par Trezor Suite pour fournir des informations sur les transactions et les adresses des blockchains. Le [serveur Blockbook](https://github.com/trezor/blockbook) est Open Source et est principalement développé et maintenu par l'équipe de Trezor (SatoshiLabs), qui l'utilise pour leur suite logicielle. La plate forme officiellement supportée est Debian Linux, la machine doit être équipée de 32Gio de Ram et d'un SSD > 200Gio. Elle supporte les actifs cryptographiques Bitcoin, Bitcoin Cash, Zcash, Dash, Litecoin, Bitcoin Gold, Ethereum, Ethereum Classic, Dogecoin, Namecoin, Vertcoin, DigiByte, Liquid, ainsi que d'autres qui ont été mis en place par la communauté. Supporte les testnets également.

## Chemin de dérivation

Les portefeuilles matériels ou logiciels modernes permettent de générer des clés et des adresses de manière hiérarchique et déterministe. Ils facilitent la gestion et l'organisation des adresses Bitcoin pour différents usages ou comptes. A partir d'une simple phrase de récupération ils sont capables de gérer plusieurs portefeuilles avec des **chemins de dérivation** différents. La structure typique d'un chemin de dérivation est la suivante : `m/purpose/coin_type/account/change/address_index`

* **m** → Master seed, c'est la clé maître, le point de départ de toutes les dérivations.
* **purpose** → c'est le but, et informe sur la norme du chemin de dérivation. Venu avec la [BIP43](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki)
  * `44` en référence aux adresses Legacy `1` `P2PKH` [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)
  * `45` en référence aux portefeuilles multisigs anciens `3` `P2SH` [BIP45](https://github.com/bitcoin/bips/blob/master/bip-0045.mediawiki)
  * `47` en référence aux codes de paiement réutilisables [BIP47](https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki)
  * `48` en référence aux portefeuilles multisigs matériels [BIP48](https://github.com/bitcoin/bips/blob/master/bip-0048.mediawiki)
  * `49` en référence aux adresses nested-SegWit `3` `P2SH-P2WPKH` [BIP49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki)
  * `84` en référence aux adresses native SegWit `bc1q` `P2WPKH` [BIP84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki)
  * `86` en référence aux adresses Taproot `bc1p` `P2TR` [BIP86](https://github.com/bitcoin/bips/blob/master/bip-0086.mediawiki)
  * l'utilisation de l'apostrophe `'` ou du `h` après le paramètre signifie "hardened derivation" ou dérivation renforcée, comme `44h` ou `44'`. Attention, il s'agit de l'apostrophe droite du clavier (ASCII 0x27) : les apostrophes typographiques `'` `'` qu'insèrent les traitements de texte ne sont pas reconnues par les logiciels, en cas de doute utilisez `h`. Un algorithme supplémentaire est mis en oeuvre, il devient presque impossible d'accéder à la clé maître même si une clé privée dérivée est compromise. Cela constitue une couche supplémentaire de sécurité.
* **coin type** → renseigne sur l'actif
  * `0` est Bitcoin (`'` ou `h` sont utilisables)
  * pour reste c'est [ici](https://github.com/satoshilabs/slips/blob/master/slip-0044.md).
* **account** → indique l'identité ou la collection d'adresses, la première est notée `0`. Dans un portefeuille multi-comptes cela permet aux utilisateurs de séparer les fonds pour différentes choses comme hold, dépense, dons, etc… (`'` ou `h` sont utilisables)
* **change** → renseigne sur la transaction
  * `0` indique les adresses de `chaîne externe`, régulières.
  * `1` indique les adresses de `chaîne interne`, c'est le change, le rendu monnaie.
  * pour plus de détail, voir [BIP44 pour la chaîne de change](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki#user-content-Change).
* **address index** → index séquentiel pour générer des adresses spécifiques à l'intérieur d'un compte, la première valeur est notée `0` ou adresse de rang 1.

Par exemple `m/44h/0h/0h/1/1` indique :

* `44h` adresse Legacy renforcée
* `0h` le type d'actif est renforcé, ici Bitcoin.
* `0h` le compte est renforcé, ici le premier, le suivant sera noté `1h`.
* `1` une adresse de rendu monnaie
* `1` une adresse de rang 2, le rang 1 (la première adresse) étant notée `0` .

## Compact block relay

Quand un mineur trouve un nouveau bloc, il doit être propagé le plus vite possible à tout le réseau. Sachant que la majorité des transactions qu'il contient sont déjà connues des nœuds car elles sont dans leur mempool. Compact block relay (BIP 152, déployé avec Bitcoin Core 0.13.0 en 2016) exploite cet état de fait, donc au lieu d'envoyer le bloc complet il envoie un résumé compact contenant :

* le header du bloc
* les identifiants raccourcis (6 octets) de chaque transaction
* les transactions que l'émetteur pense inconnues du pair (typiquement la coinbase)

Le nœud récepteur reconstitue le bloc à partir de son propre mempool. S'il lui manque une transaction, il la demande individuellement. Le gain est considérable, la transmission n'est que de quelques Kio au lieu de plusieurs Mio.

Au chapitre "Détail des connexions réseau" avec la commande `bitcoin-cli -netinfo 4` la colonne `hb` indique le statut "high bandwidth compact block relay", en voici une explication détaillé :

* Low bandwidth (mode par défaut signalé par un `hb` vide) :
  * Le nœud annonce qu'il a un nouveau bloc par `inv` / `headers`
  * Le pair le demande ensuite par `cmpctblock`
  * Le bloc compact est envoyé
* High bandwidth (ce mode est signalé par un `hb` = `.` ou `*`)
  * Le nœud envoie le bloc compact immédiatement, sans demande préalable.
  * Pas d'aller-retour, latence minimale.
  * Un nœud ne sélectionne que quelques pairs en mode high bandwidth (typiquement 3)
  * `.` signifie que le pair est en mode high bandwidth, il vous envoie les blocs compacts sans attendre que vous les demandiez.
  * `*` signifie que le pair est en mode high bandwidth et qu'il est un de vos pairs pour recevoir vos blocs en high bandwidth.

## Consensus vs Standarness

**Règles de consensus**

| Caractéristique | Détail |
|----|----|
| **Portée** | Validité des blocs et des transactions confirmées |
| **Qui doit être d'accord** | Tout le réseau à l'unanimité |
| **Conséquence si violée** | Bloc rejeté → fork de chaîne potentiel |
| **Modifiable comment** | Soft fork / hard fork (changement coordonné, lourd) |
| **Exemples** | Poids du bloc ≤ 4 000 000 WU · plafond de 21 M de BTC · pas de double-dépense · validité des signatures · règles de script à l'exécution |

Changer une règle de consensus de façon non coordonnée scinde la chaîne en deux. Si mon nœud applique des règles de consensus différentes des autres, je suis potentiellement une autre chaîne.

**Règles standardness** (règle de politique)

| Caractéristique | Détail |
|----|----|
| **Portée** | Relais, mempool, construction des templates de bloc. |
| **Qui doit être d'accord** | Personne - c'est local, propre à chaque nœud. |
| **Conséquence si violée** | Transaction non relayée / non minée par ce nœud ; reste valide en bloc |
| **Modifiable comment** | Une ligne dans bitcoin.conf, puis redémarrage du nœud, aucun accord requis. |
| **Exemples** | datacarrier / datacarriersize · nombre d'OP_RETURN · MAX_STANDARD_TX_WEIGHT (100 000 vbytes) · permitbaremultisig · minrelaytxfee · types de script « standards » |

Une transaction non standard n'est pas invalide. Elle est juste refusée à la propagation par les nœuds qui appliquent cette politique. Un mineur qui ne l'applique pas peut tout à fait l'inclure dans un bloc ; et ce bloc sera accepté par tout le monde, y compris les nœuds qui auraient refusé de relayer la transaction.

## CPU

Central Processor Unit (CPU), aussi appelé microprocesseur ou processeur central, est la puce qui exécute le code binaire produit par le compilateur à partir du code source d'un programme informatique. Le CPU est un circuit intégré généraliste à logique non câblée, ce qui signifie qu'il n'est pas spécialisé dans un domaine particulier et qu'il peut effectuer une grande variété de tâches. Cette polyvalence lui permet de "savoir tout faire", mais il n'est pas optimisé pour des opérations spécifiques.

À l'opposé, les puces spécialisées à logique câblée appelés ASIC (Application Specific Integrated Circuit), sont conçues pour exécuter très rapidement un nombre restreint de tâches spécifiques. Les ASIC destinés au minage de Bitcoin sont spécialement prévus pour résoudre l'algorithme de hachage sécurisé SHA-256 (Secure Hash Algorithm-256) utilisé dans le mécanisme de preuve de travail (Proof of Work) de Bitcoin. Ils surpassent largement les CPU et même les GPU en termes d'efficacité et de vitesse. Ces matériels sont devenus incontournables pour miner et ainsi espérer être à l'origine de la création d'un nouveau bloc.

## Eclipse

Votre nœud ne connaît l'état du réseau Bitcoin que par ses pairs. Dans une attaque par éclipse, l'attaquant prend le contrôle de toutes les connexions de votre nœud. Vous ne parlez plus qu'à lui, et il décide de ce que vous voyez : blocs, transactions, adresses d'autres nœuds.

**Méthode** : l'attaquant rempli votre carnet d'adresses (`addrman` présent dans `peers.dat`) en occupant les places entrantes et en ouvrant de nombreuses connexions vers votre nœud, il attend ensuite que vous redémarriez celui-ci, ou bien il cherchera à provoquer une coupure de connexion. En se reconnectant, votre nœud choisit ses nouveaux pairs dans le carnet d'adresses qui est désormais plein de nœuds de l'attaquant.

**Pouvoir de nuisance de l'attaquant :**

* Retarder ou cacher des blocs et des transactions. C'est le danger principal pour un nœud Lightning connecté à votre `bitcoind`. Si l'attaquant vous cache ce bloc assez longtemps et si vous ne réagissez pas avant l'expiration du timelock vous perdez les fonds.
* Vous montrer une fausse chaîne pour réaliser une double dépense contre vous : vous voyez un paiement confirmé qui n'existe pas sur la vraie chaîne. Il doit quand même miner des blocs valides, ce qui coûte cher, et ne vaut donc le coup que pour de gros montants.
* Censurer vos transactions, c'est-à-dire ne jamais les relayer.
* Vous désanonymiser : en utilisant votre nœud pour "transacter" vous serez le premier à émettre sur le réseau, il saura qu'elle vient de vous.

**Protections intégrées à Bitcoin :**

* Les connexions block-relay-only, plus difficiles à repérer pour un attaquant.
* Les anchors (du fichier `anchors.dat` qui n'existe que lorsque le noeud est arrêté) au redémarrage, votre nœud se reconnecte d'abord à ses 2 pairs block-relay-only précédents.
* Les connexions feeler et le mécanisme « test-before-evict », qui protègent le carnet d'adresses.
* La protection contre l'éviction des bons pairs : ceux qui vous envoient des blocs rapidement, ceux qui sont connectés depuis longtemps…
* La détection d'une chaîne figée : si aucun nouveau bloc n'arrive pendant environ 30 minutes, le nœud ouvre des connexions sortantes supplémentaires.

**Pourquoi Tor et I2P rendent l'attaque plus facile :**

En clearnet, Bitcoin Core répartit ses pairs entre différents opérateurs (par plage d'IP ou par AS avec asmap). Un attaquant doit donc contrôler des IP dans beaucoup de réseaux différents, ce qui est difficilement réalisable ou coûte très cher. Sur Tor et I2P, créer une adresse ne coûte rien. Un attaquant peut en générer des milliers et les regroupements par origine n'ont plus vraiment de sens. Un nœud qui n'utilise que Tor et I2P est donc plus exposé. **Avoir les deux réseaux plutôt qu'un seul réduit déjà le risque, parce que l'attaquant doit couvrir les deux.**

**Si vous connaissez un ou des pairs de confiance ce sera une excellente protection que de se connecter à eux !** Il suffit qu'un seul pair honnête reste connecté pour que l'attaque éclipse échoue. Ajouter `addnode`  dans `bitcoin.conf` comme décrit dans "Paramétrage de Bitcoin" ou en ligne de commande afin de tester sans redémarrer :

```bash
bitcoin-cli addnode xxxx.onion add
bitcoin-cli addnode xxxx.b32.i2p add

# verifier que la connexion est établie (patienter c'est plus long avec i2p)
bitcoin-cli getaddednodeinfo
```

**Les connexions croisées entre pairs de confiance (2 à 4 maxi) sont une bonne idée** pour ce prémunir mutuellement de cette attaque, et de préférence vers des nœuds hébergés ailleurs et sur d'autres réseaux que le vôtre (pas le même réseau local, pas sur la même box internet, en clearnet si vous êtes TOR/I2P only). **Le seul vrai prix à payer est la vie privée vis-à-vis de ce ou ces pairs.**

Afficher le nombre d'adresses connues par réseau : `bitcoin-cli -addrinfo`

Voir la répartition entre les tables `new` (adresses que d'autres nœuds ont communiqué au votre mais auxquelles votre nœud ne s'est jamais connecté) et `tried` (les adresses auxquelles votre nœud s'est déjà connecté avec succès, en principe les plus sures) : `bitcoin-cli getaddrmaninfo`  .

## Fingerprinting

C'est l'art de reconnaître un nœud unique à partir de ses caractéristiques, même si son IP est masquée. Exemples typiques :

* Version logicielle, services annoncés, réglages réseau.
* Horaires et rythme de connexion.
* Modèle de propagation des transactions.
* Adresse I2P / Tor persistante ou réutilisée.

## Fulcrum

Fulcrum est un serveur SPV (Simple Payment Verification) multithread haute performance pour Bitcoin (BTC), Bitcoin Cash (BCH) et Litecoin (LTC), écrit en C++20. En lançant 3 instances différentes sur une machine qui héberge les 3 nœuds complets (BTC, BCH et LTC) il sera capable de dialoguer avec le wallet Electrum et ses forks Electrum Cash et Electrum-LTC pour "transacter" sur les 3 blockchains\*\*.\*\* La synchronisation initiale de sa base de donnée (RocksDB) avec un full node Bitcoin est bien plus longue et bien plus consommatrice de RAM (4 à 8 Gio / instance conseillé) qu'avec Elecrs. Une fois opérationnel Fullcrum est nettement plus rapide à l'usage, là ou Elecrs est plus lent, mais se distingue par sa légèreté. (© Loïc Morel - Dictionnaire de Bitcoin que vous devriez acquérir :)

## GPG

GNU Privacy Guard est un logiciel open source de chiffrement et de signature numérique.

* Obtenir la liste des clés `gpg --list-keys`
* Effacer une clé en utilisant la `KeyID` : `gpg --delete-key KeyID`

## I2P

S'intéresser à Bitcoin permet de parfaire ses connaissances, avant je ne connaissais pas "Invisible Internet Project". C'est un réseau anonyme offrant une couche logicielle de type réseau overlay que les applications emploient pour envoyer de façon anonyme et sécurisée des informations entre elles. La communication est chiffrée de bout en bout.

Au total, quatre couches de chiffrement sont utilisées pour envoyer un message. L'anonymat est assuré par le concept de « mix network », qui consiste à supprimer les connexions directes entre les pairs qui souhaitent échanger de l'information. À la place, le trafic passe par une série d'autres pairs de façon qu'un observateur ne puisse identifier ni l'expéditeur ni le destinataire de l'information. Chaque pair peut, à sa décharge, dire que les données ne lui étaient pas destinées, un déni plausible.

Sur Internet, on identifie un destinataire par une adresse IP et un port. Cette adresse IP correspond à une interface physique (modem ou routeur, serveur, etc.). Sur I2P on identifie un destinataire par une clef cryptographique.

Contrairement à l'adressage IP, on ne peut pas désigner la machine propriétaire de cette clef. Du fait que la clé est publique, la relation entre la clé et l'interface qui en est propriétaire n'est pas divulguée.

*En résumé les participants ne révèlent pas leur véritable adresse IP*.

## IBD

L'Initial Block Download est la phase de première synchronisation d'un nœud, il télécharge les blocs depuis le genesis et vérifie chaque bloc/transaction jusqu'à rattraper la chaîne la plus récente.

## OP_RETURN

Dans le cadre de la crise du spam il est fait état de 80 ou parfois de 83 octets ? Les deux chiffres décrivent la même limite, 80 c'est la taille des données utiles, et 83 c'est la taille du script qui utilise 3 octets d'entête ou d'enrobage. Si `datacarriersize=83` est déclaré dans `bitcoin.conf` cela fait référence à la taille du script, ce qui laisse 80 octets utiles pour la donnée dans ce cas comme ceci :

```bash
6a   4c   50   <80 octets de données>
│    │    │
│    │    └─ 0x50 = 80 en décimal : la longueur des données qui suivent
│    └────── 0x4c = OP_PUSHDATA1 : annonce une donnée à empiler jusqu'à 255 octets
└─────────── 0x6a = OP_RETURN : marqueur de données qui indique sortie non dépensable
```

`OP_PUSHDATA1` n'est utilisé que pour la plage de données utiles de 76 à 255 octets.

En dessous de 76 octets, on prend le push direct car il est plus court et est conforme aux règles *standardness* de Bitcoin qui est d'utiliser l'encodage le plus compact possible. Cela économise un octet, exemple de push direct avec la Tx0 Whirlpool qui nécessite 46 octets de data (46 = 0x2e en hexadécimal), le script occupe donc 48 octets (46+2)  :

```bash
6a   2e   <46 octets de données>
│    │
│    └─ 0x2e = pousse les 46 octets qui suivent (l'opcode EST la longueur)
└────── 0x6a = OP_RETURN
```

Depuis Bitcoin Core 30 le défaut est actuellement de 100 000 octets ce qui permet théoriquement d'inclure jusqu'à 99 994 octets de données. En effet il faut utiliser 6 octets d'enrobage comme ceci :

```bash
6a   4e   9a 86 01 00   <99 994 octets de données>
│    │    └─────┬─────┘
│    │          └─ longueur = 99994 (4 octets)
│    └──────────── 0x4e = OP_PUSHDATA4 (1 octet)
└───────────────── 0x6a = OP_RETURN (1 octet)
```

Les "Core Developers" ont subtilement adopté la valeur `datacarriersize=100000` par défaut puisque calée pile sur la limite de poids de transaction. En effet les données contenues dans l'OP_RETURN sont hors témoin est pèsent 4WU, ce qui donne 400 000 WU soit la limite de la règle standarness (ou règle politique) `MAX_STANDARD_TX_WEIGHT`. En résumé on ne peut jamais réellement atteindre les 100 000 octets dans une transaction standard. Le maximum par transaction effectivement relayable de données OP_RETURN se situe en dessous 100 000 en fonction de la taille des entrées/sorties accompagnant la sortie data. C'est pourquoi la limite est dite « effectivement sans effet »,  c'est le poids qui tranche. Pour rappel, au dessus de la règle `MAX_STANDARD_TX_WEIGHT` s'appliquant à une transaction, il y a celle s'appliquant au bloc de transactions avec la taille limite fixée à 4 000 000 WU, soit 10 fois plus. Cette dernière n'est pas politique mais fait partie des règles du consensus Bitcoin.

Commandes à loger dans `bitcoin.conf` concernant OP_RETURN policies :

* `datacarrier=1`  le nœud Core relaye le ou les OP_RETURN par transaction, c'est le défaut.
* `datacarrier=1`  le nœud Knots relaye un seul OP_RETURN par transaction, c'est le défaut.
* `datacarriersize=100000` défaut pour Core, depuis la v30 une transaction peut contenir plus d'un OP_RETURN, en conséquence la limite s'applique à la somme s'il y en a plusieurs.
* `datacarriersize=83` défaut pour Knots
* Depuis Bitcoin Core V30 il n'est pas possible de relayer un seul OP_RETURN par transaction, soit c'est zéro avec `datacarrier=0` , soit le nombre d'OP_RETURN est limité par le poids. Avec `datacarriersize=83` la somme des scripts OP_RETURN d'une transaction est limitée à 83 octets : cela laisse passer un seul OP_RETURN de 80 octets de données, ou plusieurs petits dont la somme ne dépasse pas 83 octets (voir Choix du code, Octobre 2026).
* Il est a remarquer que plan v30 initial dépréciait cette configurabilité par suppression des options `datacarrier` et `datacarriersize` ; cette dépréciation a été annulée une semaine avant la sortie (PR #33453, intégrée à la branche 30.x le 6 octobre 2025, sortie de la v30.0 le 13 octobre), après la levée de boucliers de la communauté.
* Le flag `corepolicy=1` bascule Knots vers les défauts façon Core 29.x en une seule ligne. Cela supprime tous les filtres particuliers à Knots et relaie quasiment tout ce que Core 29.x relaierait.

## PSBT

Partially Signed Bitcoin Transactions. Une transaction Bitcoin partiellement signée est un format de données qui permet aux portefeuilles et à d'autres outils d'échanger des informations sur une transaction Bitcoin et les signatures nécessaires pour la finaliser. Une PSBT peut être créée en identifiant un ensemble d'UTXO à dépenser et un ensemble de sorties pour recevoir la valeur dépensée. Des informations sur chaque UTXO nécessaires pour générer une signature peuvent ensuite être ajoutées, éventuellement par un outil distinct, comme le script de l'UTXO ou sa valeur précise en Bitcoin. La PSBT peut ensuite être copiée par n'importe quel moyen vers un programme capable de la signer. Pour les portefeuilles à plusieurs signatures ou dans les cas où différents portefeuilles contrôlent différentes entrées, cette dernière étape peut être répétée plusieurs fois par différents programmes sur différentes copies de la PSBT. Plusieurs PSBT comportant chacune une ou plusieurs signatures nécessaires peuvent être intégrées ultérieurement dans une PSBT unique. Enfin, cette PSBT entièrement signée peut être convertie en une transaction complète prête à être diffusée.

## Secure element et norme EAL

La norme EAL est utilisée pour quantifier la résistance à diverses formes d'attaques (physiques ou logicielles) des puces appelées "secure element" utilisées entre autre dans les "non-custodial hardwares wallets" (portefeuilles matériels non dépositaires). Les clés privées sont à l'intérieur de ces puces sécurisées. Les niveaux EAL (Evaluation Assurance Level) vont de 1 à 7, chaque niveau offrant une évaluation plus rigoureuse et des exigences de sécurité plus élevées.

## SLIP39

La norme Shamir's Secret Sharing Scheme a été proposé et adopté par Trezor en 2019. Cette norme est supportée par un nombre bien plus réduit de matériels et de logiciels que BIP39.

La phrase de récupération de la graine est de 20 ou de 33 mots et est divisible de 1 à 16 fragments (ou shards), ensuite un seuil de récupération (threshold) est fixé comme 3/3 ou 2/3 ou 3/5 … mais jamais 1 si plus d'un fragment. Exemple si 20 mots en 2/3 il y aura 3 listes séparées de 20 mots chacune (60 mots au total), la restauration fonctionnera avec 2 listes de 20 mots.

Liste des portefeuilles matériels SLIP39 compatibles en 2024 :

* Trezor (Model T à partir de la version 2.7.2, Safe 3, Safe 5)
* Keystone Pro

Shamir Backup est de la sauvegarde distribuée de phrase de récupération. Multisig propose cela et procure en plus une signature distribuée, par contre c'est beaucoup plus compliqué à mettre en oeuvre.

## SPV

Simplified Payment Verification ou vérification simplifiée des paiements. Cette technique permet de vérifier des paiements sans directement utiliser un nœud complet (Full node) avec l'intégralité de la blockchain. Le client SPV n'a besoin que des en-têtes des blocs, ce qui est bien plus petit que les blocs complets. Pour vérifier qu'une transaction figure dans un bloc, le client SPV demande une preuve d'inclusion, sous la forme d'une branche de Merkle. Notez que cette technique est décrite dans le papier de Satoshi Nakamoto au paragraphe 8.

## SSH

Secure Shell désigne un protocole réseau crypté qui permet d'établir, sur un réseau supposé non fiable, une connexion sécurisée avec le shell d'un ordinateur distant.

Shell (coquille) comme Bash ou Zsh désigne l'interpréteur de commandes, c'est-à-dire le programme qui sert d'interface entre l'utilisateur et le système. Il est nommé ainsi car il agit comme une couche externe ou une « coquille » qui entoure et protège le système, permettant aux utilisateurs de saisir des commandes textuelles pour interagir avec le celui-ci sans avoir à manipuler directement du code de bas niveau. Il existe un paquet d'interpréteurs différents comme Bash (Bourne-Again Shell / défaut Linux), Zsh (Z Shell / Default sur macOS), sh (Bourne Shell / l'original), Fish (Friendly Interactive Shell / moderne avec auto suggestions …).  Pour voir la version en cours d'exécution `echo $0`. Pour le shell par défaut de l'utilisateur `echo $SHELL`.

Secure implique que toutes les communications sont chiffrées y compris les identifiants, les commandes, les données échangées. Chaque message transmis inclut un code de vérification pour détecter toute modification pendant le transit.

Par défaut, SSH utilise le **port TCP 22** et suit une architecture client-serveur. SSH est incontournable pour l'administration système, il permet d'exécuter des commandes à distance, de transférer des fichiers de manière sécurisée (via SCP ou SFTP) et de créer des tunnels pour d'autres services réseau.

## UTXO

Unspent Transaction Output ou sortie de transaction non dépensée. Une transaction consomme un ou plusieurs UTXO existants comme entrée et crée de nouveaux UTXO comme sortie, le total des fonds entrants est toujours égal au total des fonds sortants y compris les frais de transaction. Ce concept fondamental, *qui de fait est un déplacement des droits de propriété*, a été implémenté à la création de Bitcoin. Exemple : vous possédez 1 BTC sur un seul UTXO et vous devez payer 0.5 BTC à un tiers. Votre transaction utilisera votre UTXO de 1 BTC comme entrée et créera deux nouvelles sorties : 0.5 BTC vers le tiers, 0.49999 BTC de retour vers vous sur une de vos adresses de change, les frais (0.00001 BTC ici) représentent la différence entre les entrées et les sorties. Vous avez maintenant un nouvel UTXO de 0.49999 BTC. Les entrées consomment un UTXO existant tandis que les sorties créent un nouvel UTXO, en résumé seuls les produits non dépensés peuvent être utilisés dans de nouvelles transactions, cela évite la double dépense et la fraude. En règle générale les portefeuilles gèrent les UTXO de manière transparente pour les utilisateurs selon différentes stratégies en fonction des choix implémentés par les développeurs du portefeuille.

Un "full node" conserve l'UTXO set (ensemble de toutes les sorties de transactions non dépensées) + l'historique complet des blocs. A un moment donné l'ensemble des UTXO peut être additionné pour calculer l'offre totale en circulation, la commande `bitcoin-cli gettxoutsetinfo` donne 19 992 626 BTC au 21-02-2026.

# Liens

* Documentation sur le github de [Bitcoin Core](https://github.com/bitcoin/bitcoin/tree/master/doc) et de [Bitcoin Knots](https://github.com/bitcoinknots/bitcoin/tree/29.x-knots/doc) - parcourez suivant ce que vous recherchez.
* [le nombre de noeuds by Luke Dashjr](https://luke.dashjr.org/programs/bitcoin/files/charts/historical.html) - historique graphique depuis 2017
* [jon atack](https://jonatack.github.io/articles) - documentation et ressources autour de Bitcoin
* [Bitcoin Forum](https://bitcointalk.org/)
* [BIP 39 tool](https://github.com/iancoleman/bip39) - de Iancoleman, un outil pour convertir les phrases mnémoniques BIP39 en adresses et clés privées. Pour plus de sécurité activer le mode hors ligne avec le navigateur. Avec Firefox faire F10 puis puis cocher "Travailler hors connection".
* [Ur₿an.T丰ch21](https://x.com/urbantech21) - le narrateur et explorateur des archives sur Bitcoin
* [BTC TouchPoint](https://btctouchpoint.com) - le parcours en vidéos et podcasts de la chute dans le Bitcoin Rabbit Hole
* [DSN Bitcoin monitoring](https://www.dsn.kastel.kit.edu/bitcoin/) - les vidéos de propagation des blocs à diverses époques !!!
* Ludovic Lars [décortique ici](https://viresinnumeris.fr/limite-21-millions-but-originel-bitcoin/) le parcours de Satoshi Nakamoto pour établir l'adéquation économique de départ. Sinon le . de départ de ses nombreux articles [ici](https://viresinnumeris.fr/liste-articles/).
* [BitcoinStrings](https://bitcoinstrings.com/) / [Bitaddress.org](https://www.bitaddress.org/) / [LearnMeBitcoin](https://learnmeabitcoin.com/)

# Littérature

[Traduction française du white paper](https://viresinnumeris.fr/bitcoin/)

[Livre "Mastering Bitcoin" traduction française incomplète qui a le mérite d'exister](https://bitcoin.fr/wp-content/uploads/2020/08/Mastering-Bitcoin.pdf)

[Le livre blanc d'Ethereum en français contient des informations intéressantes … sur Bitcoin](https://ethereum.org/fr/whitepaper/)

[Le dictionnaire de Bitcoin par Loic … LA mine d'informations](https://github.com/LoicPandul/Dictionnaire-de-Bitcoin)


# Conclusion

## of Satoshi Nakamoto's paper

We have proposed a system for electronic transactions without relying on trust. We started with the usual framework of coins made from digital signatures, which provides strong control of ownership, but is incomplete without a way to prevent double-spending. To solve this, we proposed a peer-to-peer network using proof-of-work to record a public history of transactions that quickly becomes computationally impractical for an attacker to change if honest nodes control a majority of CPU power. The network is robust in its unstructured simplicity. Nodes work all at once with little coordination. They do not need to be identified, since messages are not routed to any particular place and only need to be delivered on a best effort basis. Nodes can leave and rejoin the network at will, accepting the proof-of-work chain as proof of what happened while they were gone. They vote with their CPU power, expressing their acceptance of valid blocks by working on extending them and rejecting invalid blocks by refusing to work on them. Any needed rules and incentives can be enforced with this consensus mechanism.

## du papier de Satoshi Nakamoto

Nous avons proposé un système de transactions électroniques qui ne repose pas sur la confiance. Nous avons commencé par le cadre habituel des pièces de monnaie fabriquées à partir de signatures numériques, qui permet un contrôle solide de la propriété, mais qui est incomplet s'il n'y a pas de moyen d'empêcher la double dépense. Pour résoudre cela, nous avons proposé un réseau pair à pair utilisant la preuve de travail pour enregistrer un historique public des transactions qui devient rapidement impossible à modifier pour un attaquant si les nœuds honnêtes contrôlent la majorité de la puissance CPU (¹). Le réseau est robuste dans sa simplicité non structurée. Les nœuds travaillent tous en même temps avec peu de coordination. Ils n'ont pas besoin d'être identifiés, puisque les messages ne sont pas acheminés vers un endroit particulier et qu'ils ne doivent être délivrés que dans la mesure du possible. Les nœuds peuvent quitter et rejoindre le réseau à volonté, en acceptant la chaîne de preuve de travail comme preuve de ce qui s'est passé pendant leur absence. Ils votent avec la puissance de leur CPU (¹), exprimant leur acceptation des blocs valides en travaillant à leur extension et au rejet des blocs non valides en refusant de travailler dessus. Toutes les règles et récompenses nécessaires peuvent être imposées avec ce mécanisme de consensus.


(¹) Nous sommes fin 2008, il est fait référence à un équipement tout en un où le nœud utilise la puissance de traitement de son CPU (²) pour la création des nouveaux blocs avec la preuve de travail. Depuis 2013 \~ 2014 cette preuve de travail (Proof of Work ou POW), *qui est la base du consensus*, est devenue une activité industrielle nécessitant des investissements colossaux, par conséquent l'individu n'a quasiment plus sa place. Le hashrate, qui quantifie la puissance de traitement SHA256 disponible sur le réseau Bitcoin, s'est externalisé et professionnalisé. Autrement dit, pour avoir une chance raisonnable de propager de nouveaux blocs avec un nœud Bitcoin, il est nécessaire de déléguer (³) la puissance de calcul à des milliers de puces spécialisées appelées ASIC qui sont groupées dans ce que l'on nomme des pools de minage.

(²) Le minage CPU via `bitcoind` n'est plus supporté depuis Bitcoin core 0.13.0 sorti en 2016.

(³) Pour des raisons de scalabilité cette délégation s'effectue majoritairement par Stratum, un protocole de communication qui sert d'intermédiaire entre les pools d'ASIC spécialisés SHA256 et le nœud Bitcoin, il permet d'agréger la puissance de calcul du minage en pool avec optimisation de la charge et de la latence réseau. Mais pas que puisque c'est au niveau du serveur de pool que le bloc est crée. En simplifiant : le mineur ne fait qu'apporter sa puissance de calcul à Stratum qui va coordonner les pools de mineurs. Si un bloc est trouvé, Stratum le transmet au nœud qui va le diffuser ensuite à tous ses pairs.

Au chapitre 4 de son papier, Satoshi Nakamoto avait intelligemment prévu et anticipé que la puissance de calcul du matériel augmenterait dans le temps et dès le départ il a conçu Bitcoin avec une difficulté ajustable pour s'adapter à cette augmentation de puissance de calcul et à l'intérêt variable des nœuds qui minent les nouveaux blocs.

## Qu'engendre la professionnalisation de la puissance de calcul dans la gouvernance de Bitcoin ?

**Les nœuds** ne créant pas de nouveaux blocs, appelés aussi nœuds témoins, ils sont gérables par les individus, ils

* sont les gardiens de toutes les transactions passées avec l'enchaînement de tous les blocs
* assurent la collecte et la diffusion des nouvelles transactions
* valident ou invalident les transactions ou les blocs qui leur sont proposés
* font respecter les règles du consensus
* sont reliés à leurs pairs le plus souvent par une couche d'anonymisation réseau

**Les pools de minage**, les industriels créant des nouveaux blocs, ils

* mettent en oeuvre des nœuds optimisés (matériel et configuration) pour le minage
* sélectionnent les transactions en attente et les ajoutent au bloc
* créent et proposent les nouveaux blocs en résolvant l'algorithme de preuve de travail
* sont à l'initiative du consensus
* sont les percepteurs de l'inflation programmée avec la récompense de bloc jusqu'en 2140
* sont les percepteurs des frais de transaction
* sont reliés à leurs pairs le plus souvent sans couche d'anonymisation pour raison de latence réseau

En admettant que ce que j'ai écrit est suffisamment vrai et exhaustif, l'on pourrait en déduire que les pools de minage proposent et les nœuds disposent. Cependant la réalité doit être bien plus subtile que cela : les pools proposent sous contrainte des nœuds et les nœuds valident avec des règles que les pools leur soumettent ? Ensuite il y a un critère important, celui de la répartition:

* du nombre de nœuds sous gestion d'individus ou d'industriels
* de la puissance de calcul entre les différents acteurs industriels du minage
* de la diversité des pools de minage
* géographique de tout cela

Réponse : *j'essaie de la trouver … mais en fait j'en sais rien !* Et puis quand je regarde la courbe à tendance parabolique du [hashrate](https://www.coinwarz.com/mining/bitcoin/hashrate-chart) dans le temps, les "incentives" fonctionnent à plein régime ! A se demander si ce n'est pas trop en regard de l'industrialisation effrénée qui tend à concentrer la puissance de calcul entre trop peu de mains. Cette concentration amène aussi une certaine opacité dans le code informatique de ce qui gravite autour du nœud Bitcoin (serveur de pool et ASIC).

Au fait, pour connaitre le hashrate à partir du nœud c'est `bitcoin-cli getnetworkhashps`

* aujourd'hui cela me répond 7.465207827981607e+20 (746 EH/s)
* au point bas de 2009, le 24 août c'était 7.208467e+5 (720 KH/s soit le hash de 10 NerdMiner)
* la différence est approximativement de e+15 soit 1 000 000 000 000 000 de fois plus

## Ma conclusion

Faites donc tourner un nœud, le vôtre, soyez indépendant, existez en tant qu'individu !