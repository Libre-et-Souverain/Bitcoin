# Construisez votre nœud Bitcoin "from scratch *" puis devenez un gardien souverain

**\* construire soit-même : compiler le code source de toutes les applications critiques et paramétrer le tout pour une sécurité et une confidentialité satisfaisantes.**


# Matériel dédié

Afin d'éviter toute déconvenue il est souhaitable de commencer par là.

## Requis machine de base

* Microprocesseur suffisamment véloce (minimum Raspberry Pi4)
* Mémoire vive 8 Go (minimum 4Go)
* Mémoire de masse 2 To de type SSD ou NVMe
* Interface réseau filaire type RJ45 reliée à internet et à votre réseau local.
* Système d'exploitation : Open Source
* Installation et administration par accès ssh (interface en ligne de commande)

dans les détails

* Un nœud Bitcoin doit tourner 24h/24  7j sur 7 et à priori pendant des années
* En neuf il y a les mini PC avec des CPU frugaux comme le N100
* D'occasion avec les PC de taille réduite comme la série des DELL Optiplex SFF
* Deux périphériques stockage de masse seront un véritable plus, il est opportun de séparer le système d'exploitation de la copie de la blockchain Bitcoin.
* Pour les données blockchain utiliser un SSD ou mieux un NVMe.M2
* Éviter toute connexion au réseau par ondes hertziennes, les fils c'est fiable !
* Les choix doivent être dictés par l'usage, la simplicité et la sobriété sont bien souvent une garantie de silence, de fiabilité et de longévité.
* A prévoir le cas échéant, une alimentation secteur secourue en cas de coupure.

## Machine minimale

Raspberry Pi 5

* cpu Arm Cortex-A76 / 4 Cores / 2.4Ghz / TDP 12 W (gravure 16 nm )
* 8 Go de Ram / LPDDR4X 4267 MHz
* Connecté au réseau par RJ45
* OS + données blockchain sur : HAT+ Geekworm X1001 avec NVMe.M2 de 2 To (¹)
* Debian GNU/Linux arm64 version "console-only"
* Le système démarre directement sur le NVMe.M2 pour une fiabilité accrue
* Consommation 4.5 watts / heure (en situation sur une moyenne de 20 jours)
* Alimentation secourue :
  * à condition que la consommation des périphériques soit < 600 mA (hdd usb interdits) il est possible d'utiliser une "Power Bank"  5V / 3A, vérifier que celle-ci soit capable de charge bidirectionnelle (alimenter le Raspberry Pi en même temps qu'elle se recharge).
  * si le lieu est exotique et qu'il n'y a pas de secteur 230v, possibilité d'alimentation par batterie avec un convertisseur DC / DC comme entrée 9-36V / sortie 5V 5A, disque USB3  supporté si consommation < 1.6A, tapez `pinctrl | grep MAX` si 'hi' est affiché dans la ligne c'est ok jusqu'à 1.6A.

(¹) pourquoi un seul périphérique de stockage :

* la sd-card est un poison dans le temps pour le système d'exploitation, à éviter absolument !
* jusqu'à présent il est impossible de booter sur NVME si plus d'un sur la carte d'extension, à vérifier dans le temps bien sûr. 
* booter sur usb est nul vu le concept de cette mini-machine, autant passer sur un système plus classique et parfois moins cher si acquis d'occasion.

## Machine classique

Dell Optiplex 5050 SFF

* cpu intel i3-6100 / 2 Cores / 4 Threads / 3.70 Ghz / TDP 51W (gravure 14 nm )
* 8 Go de Ram / DDR4 2133 MHz
* Connecté au réseau par RJ45
* Operating System sur SSD sata 240 Go
* Données blockchain sur NVMe.M2 de 2 To (le connecteur M2 2280 est sur la carte mère)
* Debian GNU/Linux PC 64 bits installée  "console-only"
* Consommation 11.2 watts / heure (en situation sur une moyenne de 15 jours)
* Alimentation secourue : onduleur. Si l'onduleur est capable de dialoguer avec le PC, vous avez  upsmon pour arrêter le système proprement en cas de coupure prolongée. Et puis oui les onduleurs c'est pénible et coûteux dans le temps. Soit on change les batteries au plomb tous les 4 à 5 ans, soit on s'aperçoit qu'il faut changer la batterie lorsque y a une coupure puisque l'onduleur n'a pas fait son boulot et que le système s'est arrêté en vrac ! Suivant le camp ou vous situez ne mettez pas d'onduleur … ou bien regardez du côté des stations d'énergie à batterie LiFePO4 dont la durée de vie est en principe > 10 ans.

## Installation du système d'exploitation

Vous trouverez sur le net de bons tutoriels pour installer le système d'exploitation. L'Open Source n'est pas une idéologie ou une méthode, c'est un rempart pour l'individu dans le cyberespace face à la centralisation et au contrôle. Le système d'exploitation est une des premières briques, donc tout comme le code source de Bitcoin il est plus que souhaitable d'utiliser un OS Open Source, Linux est le choix le plus pratique et le plus répandu. Ici également visez la simplicité, pas d'interface graphique, pas de fioritures inutiles, juste l'essentiel. La machine sera entièrement dédiée à la fonctionnalité de nœud. Par choix à l'installation j'ai créé un seul utilisateur que j'ai appelé  'btc-node'. Pour finir, juste 3 "tips" sur PC si vous avez choisi Debian la mère des distributions Linux  :

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
| Timing buffered disk reads: 1320 MB in  3.00 seconds = 439.73 MB/sec |

| **Dell OptiPlex 5050 / NVMe.M2 2To (le nvme est identique à celui du Rpi) :** |
|----|
| Timing cached reads:   15686 MB in  1.98 seconds = 7906.21 MB/sec |
| Timing buffered disk reads: 7534 MB in  3.00 seconds = 2511.01 MB/sec |

| **Desktop PC** i5-6600K **/ Disque dur 7200 tr/mn sur port sata :** |
|----|
| Timing cached reads:   32956 MB in  1.97 seconds = 16706.10 MB/sec |
| Timing buffered disk reads: 568 MB in  3.00 seconds = 189.05 MB/sec |

La première ligne est une indication sur le débit obtenu en lecture à partir du cache tampon du système d'exploitation sans accès au disque. *Cette mesure est essentiellement une indication du débit du processeur, du cache et de la mémoire du système testé.*

La deuxième ligne est une indication du débit que le périphérique supporte en lecture de données séquentielles sans surcharge du système de fichier.

Dans le cas d'usage d'un nœud Bitcoin certains processus vont demander la lecture d'une très grande quantité de données, **pour le choix du stockage de masse considérer uniquement la 2ème ligne.** 

## Performance mémoire vive

```bash
sudo apt-get install sysbench
sysbench memory run
```

| Raspberry Pi 5 / Ram 8 Go | **Dell OptiPlex 5050 / Ram 8 Go** |
|----|----|
| 3634.62 MiB/sec | 6690.11 MiB/sec |

Le débit indiqué ici est à rapprocher de la première ligne de la performance du stockage de masse, les chiffres doivent être approximativement similaires.

## Outils complémentaires à installer éventuellement

* Monitoring : `sudo apt-get install btop`  
* Descriptif hardware de la machine : `sudo apt-get install fastfetch`, puis si voulez l'info à l'ouverture du shell ajoutez une ligne `fastfetch` à la fin de `.bashrc`

# Logiciel Bitcoin

## **Mettre à jour le système d'exploitation**

```bash
sudo apt-get update
sudo apt-get upgrade
```

## **Télécharger avec git le code source de Bitcoin**

à partir des l'endroits où il est officiellement maintenu :

```bash
sudo apt-get install git
mkdir ~/code
cd ~/code

# Non exaustif ... et pour le choix voir plus bas "Compiler Bitcoin"
git clone https://github.com/bitcoin/bitcoin.git      # Pour Bitcoin Core

# ou

git clone https://github.com/bitcoinknots/bitcoin.git # Pour Bitcoin Knots
```

## **Installer les outils et les librairies**

Installer les dépendances nécessaires à la construction de [Bitcoin Core](https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md) ou de [Bitcoin Knots](https://github.com/bitcoinknots/bitcoin/)

```bash
sudo apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3 libevent-dev

sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-test-dev libboost-thread-dev

sudo apt-get install libsqlite3-dev

sudo apt-get install libminiupnpc-dev
sudo apt-get install libzmq3-dev
```

Voir les versions des librairies installées `apt show nom_de_la_librarie`

Si besoin de dé-installer une librairie, c'est `sudo apt-get remove nom_de_la_librarie`

Faire le ménage après des suppressions de paquets `sudo apt-get autoremove`

La dépendance à la base de données Berkeley ne se pose plus puisque depuis la version 0.21, Bitcoin utilise la base de données SQLite. 

## Compiler Bitcoin

D'après ce que l'on sait, durant les années 2007 à 2008 une (ou plusieurs) personne a pondu pas mal de lignes de code C++, puis les a lâchées dans le cyber-espace en 2009, un outil fonctionnel et potentiellement révolutionnaire est né. [Au fil du temps le code est ré-écrit](https://bitcoin.fr/au-coeur-du-code/), fiabilisé et amélioré par une équipe connue depuis 2010 \~ 2011 sous le nom de "core developers". Cela peut paraître perturbant mais tout code informatique doit être amélioré, fiabilisé et sécurisé. L'idée de départ parait parfaite mais sa transcription exacte en code informatique est très délicate, le résultat est donc surement qualifiable d'imparfait. Qui plus est : le 3 janvier 2009 le jour du bloc genesis, le comité était réduit à une poignée d'individus voire moins, face à des milliers de lignes de code Open Source mettant en oeuvre un concept jusque-là jamais atteint, une équipe sera ensuite bien plus forte qu'un ou quelques individus. Plus le temps passe et plus nous pouvons dire que cela se rapproche de la perfection ***à condition*** que l'équipe en charge du code conserve l'idéologie et la philosophie initiale. A souligner également, l'environnement autour de Bitcoin n'est pas figé, il peut changer ou évoluer. Ici également il faudra que les "core developers" effectuent ce qu'il faut. En tant qu'individu et gardien du registre distribué vous ferez respecter le consensus, face au code vous avez votre mot à dire, vous pouvez le choisir, avoir le choix est important et primordial, c'est le fondement même de toute véritable démocratie. Votre voix compte 1:1 comme tous les autres, ici la gouvernance s'opère par démocratie directe avec un modèle de décision strictement horizontal. Si l'époque le demande, renseignez vous, puis choisissez, ensuite vous allez mettre en oeuvre le processus de compilation de 80K lignes de C++ qu'est devenu Bitcoin et obtenir un binaire spécialement conçu pour s'exécuter sur votre machine.

### Choix du code

**Octobre 2024** : pour choisir la version de client que vous souhaitez construire, faites `git -C ~/code/bitcoin tag` ou allez voir les [releases sur le repository bitcoin](https://github.com/bitcoin/bitcoin/releases), nous sommes loin de la "Guerre des blocs" de 2015 à 2017, la période est apparemment calme et sereine, donc se limiter à deux options me semble raisonnable, cela a donné pour moi :

* 'v28.0' pour la dernière "final" qui intègre des améliorations et des correctifs.
* 'v27.1' l'avant-dernière "final", contient des correctifs pour l'essentiel.

**Décembre 2025** : l'année 2025 a vu apparaître une crise du spam avec notamment "la guerre des OP_RETURN". OP_RETURN est un opcode ou une instruction dans le langage de script de Bitcoin qui permet d'intégrer des données arbitraires dans une sortie de transaction que l'on ne peut pas dépenser. Cela signifie que ces données sont stockées sur la blockchain sans créer une sortie valide qui pourrait être réclamée plus tard. Historiquement OP_RETURN a été introduit en 2014 dans Bitcoin Core 0.9 pour permettre l'ajout de petites quantités de données comme des preuves de timestamp, des métadonnées ou des hachages pour des applications comme le notaire virtuel, tout en évitant le spam sur le réseau. La limite standard de relais (relay policy) était de 80 octets par OP_RETURN pour empêcher une surcharge de la blockchain avec des données inutiles.  La version Bitcoin Core 30.0 sortie en octobre 2025 lève cette limite de 80 octets, ne laissant plus que la limite de taille des blocs de 4 Mo. Les débats autour d'OP_RETURN portent souvent sur l'équilibre entre nouvelles fonctionnalités (les inscriptions, les tokens, finance décentralisée, etc … ) et la protection contre le spam, qui pourrait gonfler la blockchain et augmenter les coûts pour les utilisateurs ordinaires. A l'opposé de Bitcoin Core v 30.0 il y a [Bitcoin Knots](https://github.com/bitcoinknots/bitcoin) v 29.2 20251110 qui est conçu entre autres pour renforcer la décentralisation et filtrer les transactions considérées comme abusives, par défaut OP_RETURN est limité à 83 octets. 

Il est sain que des frictions existent. La résilience d'un réseau c'est aussi sa diversification : avant ce débat seulement 1 à 2 % des [nœuds du réseau](https://coin.dance/nodes) n'utilisaient pas Bitcoin Core, en décembre 2025 c'est 21.5 % qui tournent avec Bitcoin Knots. Sur [The Bitcoin Portal](https://thebitcoinportal.com/onchain/spam-analysis/overview) figure un dashboard qui distingue les transactions financières des non financières dans le temps. Ceci étant faites vos propres recherches et forgez vous votre propre avis.

### Options de compilation

`gui` ou `bitcoin-qt (GUI)` signifie Graphical User Interface et `qt` est un framework de développement multi-plateforme pour la création d'applications et d'interfaces utilisateur graphiques avec C++. Nécessite une interface graphique, à tester c'est sympa, mais off dans ce cas d'usage.

`zmq` ou `ZeroMQ` est une interface de notification qui permet à des applications externes de recevoir des mises à jour en temps réel sur les événements du réseau Bitcoin. Zéro c'est pour 0 intermédiaire, 0 latence, 0 coût, 0 administration. Cool zéro admin :) MQ c'est pour "Message Queue" ou file de messages. Activez, peut être utile par la suite.

A propos du terme USDT, cela n'a rien à voir avec les "stablecoins", c'est utilisé pour activer les traces utilisateur statiquement définies, ce sont des outils de traçage pour l'analyse des performances et le débogage. C'est très intéressant mais hors du "scope" ici.

Il existe d'autres options de compilation non décrites ici, non utiles pour l'usage visé ici.

### Si version < v29

Ci dessous je choisi la  v28.0 sans les options de test de débogage et de performance :

```bash
# Si version choisie < v29, utiliser Autotools
cd ~/code/bitcoin
git checkout tags/v28.0
./autogen.sh
./configure --disable-tests --disable-fuzz-binary --disable-bench --disable-usdt
```

Relancer `.configure` tant que vous n'obtenez pas ce que vous désirez, pour de l'aide faites `./configure --help` ou `./configure --help | grep -A1 "je cherche ce terme"`

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
  with upnp       = yes
  with natpmp     = no
  USDT tracing    = no
  sanitizers      =
  debug enabled   = no
  werror          = no

  target os       = linux-gnu
  build os        = linux-gnu
```

Si vous avez paramétré une option mais qu'à l'arrivée elle est absente, regardez si vous n'avez pas une dépendance manquante. Des logs sont générés par `configure`, pour vérifier si une option activée au départ n'est pas ensuite dé-activée effectuer : `grep -i <option_particuliere> config.log` comme par exemple `grep -i bench config.log`

Construction et installation du binaire :

```bash
make                # Compilation
sudo make install   # Installation des binaires, par défaut dans /usr/local/bin
```

La compilation (la ligne make) dure 1 à 2 heures sur un Raspberry Pi 5 / NVMe.M2 en fonction des options (si tests, fuzz binary et bench sont actives cela rallonge le temps de compilation) et environ 20mn sur le Dell Optiplex 5050. Pendant ce temps vous pouvez ouvrir un nouveau terminal et installer Tor, I2P,  … ou continuer à flâner dans le code source ou la documentation de Bitcoin. N'oubliez pas de taper la deuxième ligne !

### Si version ≥ v29

A partir de la v29 de Bitcoin, le système de build a migré d'Autotools vers CMake.

 Si CMake non installé, effectuer `sudo apt-get install cmake`

```bash
# Si version choisie > v28
cd ~/code/bitcoin
git checkout tags/v29.2.knots20251110
mkdir build

cmake -B build -LH # liste complete et à jour des options disponibles avec leurs valeurs par defaut
cmake -B build -DWITH_ZMQ=ON -DBUILD_TESTS=OFF # Selection des options avant compilation
```

Sortie CMake que j'obtiens :

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
cmake --build build -j $(nproc) # compile en utilisant tous les cœurs CPU
# duree du buid avec tous les cœurs d'un Xeon E5-2698 v4 : 109s !

sudo cmake --install build      # installation 
```

### Binaires installés

* `bitcoin-cli` client RPC qui permet d'interagir avec `bitcoind` via des commandes
* `bitcoin-tx` utilitaire pour créer, modifier, signer et décoder des transactions brutes. Mode offline possible.
* `bitcoin-util` utilitaire pour développeur. Comme effectuer des tests pour des tâches Bitcoin qui ne nécessitent pas un nœud en cours d'exécution. Utilisé principalement pour les opérateurs/maintainers de Signet un réseau de test Bitcoin plus stable et fiable que le traditionnel testnet. 
* `bitcoin-wallet` outil pour gérer les fichiers wallet offline, opérations wallet sécurisées hors node.


* `bitcoind` daemon principal qui exécute un nœud Bitcoin

## **Installer Tor et I2P**

Ces deux réseaux se superposent à internet et apportent une couche d'anonymisation. Tous deux procurent un espace de liberté et de souveraineté individuelles, d'autre part ils procurent à Bitcoin une résilience et une résistance à la censure accrues. Commençons par Tor:

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

Pour la Debian la réponse est "debian-tor", et dans mon cas c'est l'utilisateur "btc-node" (que j'ai créé à l'installation de ma distribution) qui lancera le programme `bitcoind`, cela donne pour moi  :

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

CJDNS est une autre solution d'anonymisation supportée par Bitcoin, sa mise en oeuvre est relativement compliquée et elle me semble moins utilisé que Tor ou I2P, je l'ai explorée, je reviendrais mettre à jour ici si je trouve que cela est concluant.

## **SSH et la sécurité**

Votre futur nœud \[Bitcoin\] est connecté au réseau internet 7j/7, les mots de passe ne sont pas d'une sécurité absolue et sont pénibles à l'usage. Pour se passer du mot de passe et améliorer la sécurité installez une clé d'authentification sur le ou les postes clients de votre réseau local devant avoir un accès Secure Shell (ssh) à votre nœud.

\[PC\] désigne le poste client (pour moi c'est Manjaro Linux)

\[PC\] lancer un terminal et ouvrir une session sur votre nœud par `ssh username@IP`

\[Bitcoin\] si le répertoire `.ssh` est non présent enchaîner `mkdir ~/.ssh` puis `chmod 700 ~/.ssh`

\[Bitcoin\] saisir : `nano ~/.ssh/authorized_keys` et laissez ouvert.

\[PC\] ouvrir un 2ème terminal pour générer des clés par `ssh-keygen -t rsa`

* sauvegarder la paire de clés dans le home de l'utilisateur, saisir `/home/[PC]_user/id_rsa`
* passphrase doit être vide, taper `Enter` 2 fois

\[PC\] visualiser le contenu de la clé publique  `cat id_rsa.pub`

\[PC\] copier intégralement la sortie

*Tip : pour copier coller du texte d'un terminal Linux à un autre terminal Linux, sélectionnez le simplement à la souris dans le 1er, positionnez le curseur de la souris dans le 2ème puis faites un click molette.*

\[Bitcoin\] coller le contenu dans `~/.ssh/authorized_keys` précédemment ouvert, enregistrer, quitter nano, faire `chmod 600 ~/.ssh/authorized_keys`. Ne pas fermer la session.

\[Bitcoin\] Faites :  `sudo nano /etc/ssh/sshd_config` 

Lisez puis rajoutez ceci à la fin du fichier de configuration de ssh

```bash
# Acces SSH avec cle d'authentification et sans mot de passe
#
# Si les parametres definis ci-dessous sont presents plus haut dans le fichier
# de configuration, alors commentez les !
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
```

\[Bitcoin\] sauvegarder et quitter l'éditeur nano, toujours laisser la session ouverte.

\[Bitcoin\] redémarrer le service ssh par  `sudo service ssh reload`

\[PC\] ouvrir un nouveau terminal pour se connecter à \[bitcoin\] par  `ssh username@IP` 

\[PC\] vérifier qu'il est impossible de se connecter à \[bitcoin\]

\[PC\] Déplacer les clés dans `~/.ssh` par :

```bash
mv ~/id_rsa.pub ~/.ssh/id_rsa.pub
mv ~/id_rsa ~/.ssh/id_rsa
```

\[PC\] se re-connecter à \[bitcoin\] par `ssh username@IP`, cela doit fonctionner sans mot de passe.

En cas d'échec vous avez toujours accès à \[bitcoin\] par l'ouverture de la première session pour remédier au problème ... Si c'est OK, fermer tous les terminaux par 'exit'.

\[PC\] S'il est nécessaire d'effacer un "host" en particulier dans `.ssh/known_hosts`, la commande est : `ssh-keygen -R <hostname>` , `hostname` désigne une IP ou un nom de machine sur le réseau local.

\[PC\] vous pouvez accéder par ssh à plus de multiples hôtes à condition d'utiliser la même clé d'authentification. Si vous devez utiliser des clés différentes, il faudra définir un fichier `config` dans `~/.ssh` répertoriant chaque hôte / clé.

## Séparer les données Bitcoin de l'OS

 **A effectuer seulement si vous avez 2 unités de stockage distinctes.**

Les données blockchain d'un nœud complet sont imposantes (650 Go en octobre 2024), si vous ne pouvez plus mettre à jour l'Operating System ou rencontrez un problème avec celui-ci ou avec la machine elle-même, il est judicieux de séparer les données du nœud Bitcoin du reste.

Si le deuxième stockage de masse n'est pas déjà monté au démarrage de la machine, utiliser la commande `lsblk -o +PTTYPE` pour obtenir le "NAME" de tous les périphériques branchés, si c'est partitionné "TYPE" indiquera `part` , s'ils sont montés ce sera indiqué à la colonne "MOUNTPOINTS", "PTTYPE" indique le type de table de partition dos ou gpt :

* l'indicateur `dos` reporté par `lsblk` veut dire MBR (Master Boot Record), la taille de la partition sera limitée à 2 To, et c'est 4 partitions maximum, outil `fdisk`.
* `gpt` c'est GPT (GUID Partition Table), les partitions peuvent excéder 2 To, et il est possible de créer jusqu'à 128 partitions, outil `gdisk` ou `parted`.
* pour résumé, préférez GPT pour la table des partitions.

Pour la suite **attention /!\\ vérifiez bien ce que vous faites et adaptez en fonction de votre cas /!\\** 

Si `gdisk` pas installé : `sudo apt-get update` suivi de `sudo apt-get install gdisk`

Sur mon Dell Optiplex c'est `nvme0n1` de 2To, il n'est pas monté au démarrage, c'est donc :

 `sudo gdisk /dev/nvme0n1` pour créer la table des partitions sur `nvme0n1`, puis saisir :

*  p pour information et ainsi vérifier que c'est le bon périphérique
* `n` pour créer une nouvelle partition primaire, entrée par défaut.
* Premier secteur, entrée par défaut.
* Dernier secteur, entrée par défaut pour tout l'espace restant.
* Type de partition, entrée pour "Linux filesystem" par défaut.
* `p` pour vérifier la partition avant de l'écrire
* `w` pour écrire les changements et `Y` pour confirmer\n

**Si la machine est limitée**, formater en `ext4` comme ceci :

```bash
# Obtenir le nom de la nouvelle partition (pour moi la reponse est 'nvme0n1p1')
lsblk -o +PTTYPE

# Creer le systeme de fichier, ici ext4
sudo mkfs -t ext4 /dev/nvme0n1p1

# Créer un point de montage, j'ai choisi 'nvme' dans mnt
sudo mkdir /mnt/nvme

# D'abord monter le périphérique
sudo mount /dev/nvme0n1p1 /mnt/nvme

# Ensuite donner les droits à l'utilisateur qui va lancer le noeud
# Desormait cette partition appartiendra ici a btc-node
sudo chown -R btc-node:btc-node /mnt/nvme

# Creer le repertoire bitoin
mkdir /mnt/nvme/bitcoin

# Verifier l'utilisateur, le groupe et les droits d'acces
ls -al /mnt/nvme

# Identifier le peripherique par
sudo blkid

# Editer fstab par :
sudo nano /etc/fstab

# Puis rajouter à la fin de fstab ces 2 lignes ci dessous  :

# Stockage de masse dedie a bitcoin
UUID=uuid-device /mnt/nvme   ext4    defaults        0       2
```

**Si  la machine est plus véloce et que le périphérique est ssd ou nvme** vous pouvez utiliser BTRFS (B-tree File System), comme ceci :

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

# Creer des subvolumes qui permettentront de separer logiquement les donnees
# et de faire des snapshots independants.
sudo btrfs subvolume create /mnt/btrfs/@bitcoin      # la blockchain bitcoin
sudo btrfs subvolume create /mnt/btrfs/@electrs      # les donnees d'Electrum
sudo btrfs subvolume create /mnt/btrfs/@snapshots    # les snapshots

# Demonter
sudo umount /mnt/btrfs

# Creer les sous repertoires dans l'arborescence (montages a venir)
sudo mkdir /mnt/btrfs/bitcoin
sudo mkdir /mnt/btrfs/electrs
sudo mkdir /mnt/btrfs/snapshots

# Recuperer l'UUID du peripherique
sudo blkid /dev/nvme0n1p1

# Editer fstab par :
sudo nano /etc/fstab

# Puis rajouter à la fin de fstab les 7 lignes ci dessous  :

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
ls -al /mnt/brtfs

# Voici l'equivalent de disk free (df) avec les particularites de BTRFS
sudo btrfs filesystem df /mnt/btrfs/bitcoin
```

**Inconvénients de BTRFS sur Ext4** : consomme des ressources CPU et de la Ram, pas adapté aux disques durs classiques. 

**Avantages de BTRFS sur Ext4** : très bien adapté au stockage de masse hors rotatif comme les ssd et les NVMe. BTRFS intègre une fonctionnalité très puissante, les snapshots : ils sont très rapides et économes en espace car seul les changements sont stockés ce qui est parfait pour des backups avec snapshot read-only ou des tests avec snapshot writable (modifier la copie sans affecter l'original grace au Copy-on-Write). Effectuer un snapshot de toute la blockchain bitcoin peut s'avérer utile lors d'une mise à jour de version de `bitcoind`. Autres avantages : compression transparente, checksums pour détecter les corruptions, subvolumes, optimisé pour ssd / nvme avec maintien des performances et prolongation de la durée de vie. Avec le kernel 6.12 de Debian 13 (Trixie) toutes les options ssd / nvme utiles pour l'usage d'un nœud bitcoin sont activées par défaut sauf `noatime` (Debian monte les Btrfs avec `relatime` par défaut). Pour vérifier ces options, faire `mount | grep btrfs`.\n

Redémarrer la machine par `sudo shutdown -r now` et vérifier que le périphérique est bien monté automatiquement. Pour copier des fichiers et faire tout un tas de choses (y compris tout massacrer), depuis une éternité il existe sous Debian un gestionnaire de fichier en mode texte, un clone de Norton Commander, que j'affectionne bien et avec lequel vous pouvez utiliser la souris en mode texte dans un terminal : "Midnight Commander" alias `mc` si cela vous tente c'est `sudo apt-get install mc` .

Pour finir, si le répertoire `~/.bitcoin` est présent vérifiez qu'il soit vide, ensuite supprimez le. Créez un lien symbolique appelé `.bitcoin` dans le home de l'utilisateur qui lancera le nœud Bitcoin, et qui pointera vers votre unité de stockage dédié à cela.

```bash
# Adaptez le chemin à votre peripherique si ext4 ou BTRFS
ln -s /mnt/nvme/bitcoin ~/.bitcoin
ln -s /mnt/btrfs/bitcoin ~/.bitcoin
```


## **Paramétrage de Bitcoin**

`bitcoin.conf` est le fichier de configuration de `bitcoind` le logiciel nœud Bitcoin.

`bitcoind` (d à la fin pour "daemon", un démon logiciel est un processus qui tourne en arrière plan) lancé sans fichier de configuration fonctionne très bien. Pour l'arrêter tapez "CTRL-C" quand il n'est pas explicitement lancé en mode démon. S'il est en mode démon utiliser le programme `bitcoin-cli`  (acronyme de Bitcoin Command Line Interface) qui communique avec lui, pour arrêter le démon c'est `bitcoin-cli stop`

**Sachez que dès son lancement**, vu que vous avez zéro bloc dans votre nœud, `bitcoind` va **obsessionnellement** aller les chercher auprès d'autres nœuds sans utiliser de surcouche d'anonymisation. Donc effectuez ce qui suit puis choisissez.

Créer le répertoire `.bitcoin` qui va héberger le fichier de configuration :

```bash
mkdir ~/.bitcoin # oubliez cette ligne si vous avez une unite de stockage séparée et dediée à la blockchain
touch ~/.bitcoin/debug.log # pour être en mesure de suivre les logs dès le lancement de bitoind
nano ~/.bitcoin/bitcoin.conf
```

Copier / coller ceci dans le fichier `bitcoin.conf` :

```bash
# Ce fichier de configuration doit etre dans le chemin
# ~/.bitcoin/bitcoin.conf
#
# Pour plus d'informations visitez 
# https://jlopp.github.io/bitcoin-core-config-generator/

# En activant ce mode, le noeud accepte les connexions et commandes RPC (Remote Procedure Call) 
# et notamment les commandes de "bitcoin-cli"
server=1 # Laisser actif si daemon=1

# bitcoind est lance en tache de fond comme un demon et accepte les commandes
# Avec ce parametre a 1 pour le stopper utiliser la commande 'bitcoin-cli stop'
daemon=1

# 1 = CPU_CORES, 0 = automatique, moins de zero = laisse ce nombre de coeur CPU libre(s)
# Pour information, la commande 'nproc' retourne le nombre de CPU_CORES de la machine.
par=-1

# Force votre noeud a valider chaque transaction et bloc a partir du bloc Genesis.
assumevalid=0

# Elague la blockchain au fur et a mesure afin de reduire la place occupee
# Vous perdez les transactions les plus anciennes afin de limiter ici à 550Mo
# Apres activation il n'est pas possible de revenir en arriere, il faudra
# a nouveau telecharger la totalite de la blockchain.
#prune=550

# Pour une synchronisation plus rapide, definissez le parametre en fonction de la
# memoire disponible. Par exemple, avec 8 Go de memoire, quelque chose comme
# 'dbcache=5000' fait sens. Verifiez la memoire vive totale avec 'free -mh'.
# Pour une utilisation reduite de la memoire, ce parametre peut etre diminue ou voire
# supprime une fois que la synchronisation initiale est terminee.
# La valeur par defaut est 300 (mb).
dbcache=5000

# Index de transaction etendu optionnel, cela prend un peu plus d'espace.
# Requis pour btc-rpc-explorer, Electrum server et le reseau Lightning.
# Cela permet egalement d'activer certaines fonctionnalites pour les "wallets"
# se connectant par RPC directement a bitcoind comme la recherche
# des entrees sur les transactions.
# Si vous ne voulez pas de ces fonctionnalites dans un premier temps vous
# pouvez commenter, il est possible d'activer par la suite, bitcoind
# construira l'index la ou il s'est arrete.
# Non compatible avec les noeuds elagues (Pruned nodes)
txindex=1

# Accepte les connections entrantes avec d'autres pairs : vous permettez aux
# autres noeuds d'acceder au votre.
listen=1

# Options debug, il y en a tout un tas, vous pouvez rendre bavard
# bitcoind dans differents domaines, soit au lancement en ligne de commande par
# 'bitcoind -debug=tor -debug=i2p' ou ci-dessous dans les reseaux en de-commentant
#debug=net i2p tor

# Parametres du journal de debogage ou "Debug log settings"
# Commentez si vous voulez voir tous les logs au debut, c'est interressant :)
# Ensuite de-commentez car cela peut occuper beaucoup de place
#shrinkdebugfile=1

# Si IPV4 est inactif vous pouvez laisse commente la ligne rpcalowip, sinon
# de-commentez pour limiter l'acces RPC bitcoind a votre reseau local (LAN)
# afin de ne pas l'exposer a des reseaux non fiables tels que l'internet (WAN).
# Adaptez l'adresse par celle de votre reseau local, exemple : 192.168.0.0/24
# autorise 256 adresses de (192.168.0.0 à 192.168.0.255)
# rpcallowip=192.168.X.0/24

# de-active le wallet de bitcoind
#disablewallet=1
 


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
# d'ecouter sur les adresses IP autres et notamment exterieures
bind=127.0.0.1

# TOR
proxy=127.0.0.1:9050
onlynet=onion

# I2P
i2psam=127.0.0.1:7656
onlynet=i2p
```

Enregistrez puis réglez les permissions par `chmod 600 ~/.bitcoin/bitcoin.conf`

Lisez attentivement votre fichier `bitcoin.conf`, ensuite si vous souhaitez télécharger la blockchain assez rapidement (de quelques heures à 3 jours en fonction de votre débit internet) commentez toutes les lignes après "IPV4 inactif / TOR et I2P active / Confidentialité avancée", vous perdrez la confidentialité le temps du téléchargement initial. Lorsque c'est synchronisé avec les autres nœuds, stoppez bitcoind, dé-commentez les lignes précédemment commentées, relancez bitcoind.

Si vous tenez à votre confidentialité dès le départ, laissez comme c'est, le téléchargement de la blockchain sera plus long, cela peut prendre jusqu'à une semaine.

## Lancement de bitcoind

Visualisez les logs dans un premier terminal :

```bash
tail -f ~/.bitcoin/debug.log
```

Dans un deuxième terminal, lancez votre nœud avec le debug de TOR et I2P si besoin :

```bash
bitcoind -debug=tor -debug=i2p
```

Observer les logs de quelques minutes à plusieurs heures étalées, documentez-vous et décodez-les, c'est intéressant. Vérifiez que votre nœud télécharge les blocs. 

Pour arrêter :  `bitcoin-cli stop` et attendre l'apparition dans les log de `Shutdown: done` (cela prend parfois plusieurs minutes). Relancez `bitcoind`, il reprendra de l'endroit où il s'était arrêté.

Savoir où en est votre nœud : `bitcoin-cli getblockchaininfo`

Informations réseau : `bitcoin-cli getnetworkinfo | less`

Voir le nombre d'adresses connues par votre nœud : `bitcoin-cli -addrinfo`

Évaluer le trafic réseau : `bitcoin-cli getnettotals`

Informations sur le wallet qui vient avec bitcoind : `bitcoin-cli getwalletinfo`

## Détail des connexions réseau

Voici un aperçu des pairs entrants et sortants après plusieurs jours de fonctionnement sans interruption. Ne soyez pas pressés, cela prends du temps.

Tapez dans le shell : `bitcoin-cli -netinfo 4`

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

Signification des entêtes de colonnes dans Bitcoin Core Client

< - >

* in(bound) : connexion entrante, le pair est connecté à mon nœud,  je lui envoie mes données.
* out(bound) : connexion sortante, je suis connecté à un pair, il m'envoie ses données.

type, niveau de service fourni par le pair

* **vide**, normal c'est in.
* **full**, le pair peut fournir l'intégralité de la blockchain, il la conserve en totalité, offrant ainsi une validation complète des transactions.
* **block**, pour économiser de la place le pair ne conserve pas les blocs les plus anciens (pruned node ou nœud élagué) , il ne peut fournir que les blocs les plus récents.
* **feeler**, connexions sortantes utilisées par les nœuds Bitcoin pour vérifier l'état du réseau et trouver de nouveaux pairs. Elles sont temporaires et permettent de s'assurer que le nœud reste connecté à un ensemble diversifié de pairs pour maintenir la robustesse du réseau.
* **terrible**, utilisé lorsqu'une adresse de pair échoue à se connecter suffisamment de fois, elle est qualifiée de « terrible » pour indiquer qu'elle est peu fiable. Si une nouvelle adresse entrante entre en collision avec une adresse « terrible », cette dernière est expulsée de la table des nouvelles connexions, ce qui signifie qu'elle ne sera plus considérée pour de nouvelles connexions. Cela aide à maintenir la qualité et la fiabilité des connexions dans le réseau Bitcoin en éliminant les pairs problématiques.
* **manual**, connexion à un nœud défini par `addnode` ou `connect` dans `bitcoin.conf` 

net, informations sur les connexions réseau du nœud

* 'npr' pour "Not Publicly Routable", mon nœud Bitcoin est connecté à un pair dont l'adresse ip n'est pas publiquement routable. Chaque nœud du réseau partage également les informations des autres et un nœud peut conserver les informations dans sa propre base de données locale de nœuds connus, de sorte que d'autres ou vous, pouvez vous connecter à l'autre même si l'ip n'est pas publiquement routable. Dans ce cas `bitcoind` crée des ports locaux (127.0.0.1:xxxxx) dynamiques pour pouvoir échanger avec ces pairs.
* onion : la connexion au pair s'effectue par TOR
* i2p : la connexion au pair s'effectue par I2P

v, version utilisé pour la connexion

* v1 correspond à un mode de connexion ou sous-protocole plus ancien ou simplifié
* v2 correspond à mode de connexion ou sous-protocole plus récent, offrant des améliorations en termes de sécurité et de performance.
* en règle générale la version du protocole le plus ancien est utilisé par les versions de Bitcoin Core plus anciennes, mais pas que : pour certaines raisons (configuration réseau, rétrocompatibilité, mises à jour progressives) une même version de Bitcoin Core peut utiliser soit v1 soit v2.  L'utilisateur n'a pas la main là-dessus, c'est `bitcoind` qui choisit automatiquement la version du sous-protocole en fonction des configurations et capacités du pair avec lequel il se connecte.

mping (mean ping)

* représente le temps moyen de réponse pour envoyer un message "ping" au pair. Il s'agit d'une moyenne de plusieurs essais, donnant une indication générale de la latence réseau entre les nœuds.

ping

* représente le temps de réponse instantané pour un message "ping" spécifique. C'est une mesure plus directe et immédiate de la latence au moment précis de l'envoi.

send 

* nombre total de messages envoyés au pair, pouvant inclure des transactions, blocs, requêtes, réponses, etc… 

rcv

* nombre total de messages reçus du pair, pouvant inclure des transactions, blocs, requêtes, réponses, etc…

txn, c'est le nombre de transactions pour cette connexion, cela comprend :

* la propagation des transactions : lorsqu'un utilisateur initie une transaction, elle est d'abord envoyée à un nœud. Ce nœud vérifie la validité de la transaction puis la propage à ses pairs via des messages "txn".
* la vérification et validation : tous les nœuds qui reçoivent le message "txn" vérifient la validité de la transaction en utilisant la blockchain complète qu'ils détiennent. S'ils jugent la transaction valide, ils la propagent à leur tour.
* l'inclusion dans les blocs : les mineurs récupèrent ces transactions vérifiées pour les inclure dans de nouveaux blocs.

blk

* nombre de blocs transférés pour cette connexion

hb

* l'état du "heartbeat", c'est le métrique de la disponibilité de la connexion.

addrp (address requests peer)

* c'est une demande d'adresse au pair. Mon nœud demande des informations sur d'autres nœuds du réseau Bitcoin à ce pair spécifique. C'est essentiel pour découvrir et se connecter à davantage de nœuds pour une meilleure connectivité et résilience du réseau.

addrl (address replies peer)

* indique le nombre de réponses aux requêtes d'adresses reçues du pair. En d'autres termes, cela indique combien de fois ce pair a répondu à mes demandes en fournissant les adresses d'autres nœuds du réseau Bitcoin.

age

* l'âge de la connexion dans le temps spécifié en bas de la colonne

id

* chaque pair se voit attribuer un identifiant unique lorsqu'il se connecte à mon nœud. Cet identifiant est temporaire et spécifique à la session de connexion. L'id permet à Bitcoin Core" de gérer les connexions, les envois et les réceptions de messages, ainsi que d'effectuer le suivi des statistiques et des performances de chaque pair individuel. L'identifiant unique est utile pour diagnostiquer et résoudre des problèmes de connectivité ou de performance en permettant de suivre les interactions spécifiques avec chaque pair. Le pair est identifié dans les logs par cette id.

address

* adresse IP ou TOR(onion:8333) ou I2P(b32.i2p) du pair
* si l'adresse est de la forme \[localhost:port_dynamique\] comme \[127.0.0.1:xxxxx\], le pair ne divulgue pas son adresse externe (dans `bitcoin.conf` le paramètre `externalip` est non configuré) ou c'est une IP npr.

version

* "Protocol version" suivi de "subversion" du logiciel Bitcoin Core utilisé par le pair

Ci dessous la sortie de [Bitcoin Knots](https://bitcoinknots.org/) en v29.1 : celle-ci comporte deux colonnes supplémentaires par rapport à Bitcoin Core pour une meilleure visibilité et un monitoring avancé des nœuds connectés :

```javascript
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

**Colonne serv** abréviation de "services", implémentée pour améliorer le monitoring réseau. Elle concatène des lettres minuscules et des chiffres correspondant aux flags actifs supportés par chaque pair. Le nœud local l'indique dans son en-tête, ici en fin première ligne c'est "nwl2r"

* n : NETWORK, si seul c'est un pair basique, legacy.
* b : BLOOM, supporte les filtres Bloom (pour wallets SPV / légers)
* w : WITNESS, supporte SegWit
* c : CSV_BLOCKS, support des blocs avec CheckSequenceVerify (CSV, BIP 68) pour timelocks relatifs.
* l : NETWORK_LIMITED
* 2 : NETWORK_LIMITED_2, réduit la bande passante pour pairs lents, entre autres.
* r : NOTRANS, ne relaie pas les transactions non confirmées, anti-spam.
* vide : aucun service spécial n'est actif, affiche une valeur vide.

La commande `getpeerinfo` est plus détaillée que `bitcoin-cli -netinfo 4`, sa sortie est un JSON de tous les pairs, pour visualiser les flags "servicename" d'un pair en particulier, ici par l'id 1857, tapez ceci :

```javascript
# Si jq pas installé (jq = processeur JSON en ligne de commande) 
sudo apt-get update && sudo apt-get install jq

# Commande proprement dite
bitcoin-cli getpeerinfo | jq '.[] | select(.id == 1857)'

# Visualiser uniquement les flags servicenames pour le pair 1857
bitcoin-cli getpeerinfo | jq '.[] | select(.id == 1857) | .servicesnames'

# Lister tous les pairs par leur id et afficher leurs flags "servicesnames"
bitcoin-cli getpeerinfo | jq -r '.[] | "\(.id): \(.servicesnames | join(", "))"'
```

**Colonne cpu** pour "cpu_load", c'est la charge cpu imposée par chaque pair, exprimée en permilles (‰) de la durée totale de la connexion. C'est-à-dire : le temps CPU (utilisateur + système) passé à traiter les messages reçus du pair et à préparer les réponses, divisé par la durée de la connexion (multiplié par 1000 pour obtenir des ‰). Exemple par l'absurde : 500 pour 500‰, donc 50% de temps CPU pour un pair connecté, cette valeur (fictive) très élevée indique  que ce pair est en train de vous spammer ou DDOS :( → déconnectez le avec `disconnectnode` :) Bitcoin Core n'a pas ce champ dans `netinfo` ni dans `getpeerinfo`. Il faut des outils externes pour "monitorer" le CPU par pair, ce qui est moins pratique. Dans Knots, c'est natif pour un diagnostic en temps réel.

## Mise en place du service `bitcoind`

Votre nœud est maintenant synchronisé, si vous avez une partition BRTFS effectuez un Snapshot de `@bitcoin`. Ensuite passez à mise en place du service permettant d'automatiser le lancement et l'arrêt du nœud Bitcoin au lancement et à l'arrêt de la machine. Dans un premier temps, créer le fichier `bitcoin.service`

```none
sudo nano /etc/systemd/system/bitcoin.service
```

```bash
# bitcoin service
# /etc/systemd/system/bitcoin.service
##### Adaptez l'utilisateur defini, ici c'est 'btc-node' #####
#
# Ce service est minimaliste, pour plus d'informations visitez :
# https://github.com/bitcoin/bitcoin/tree/master/contrib/init

[Unit]
Description=Bitcoin daemon
After=network.target

[Service]
ExecStart=/usr/local/bin/bitcoind -daemon \
                                  -pid=/home/btc-node/.bitcoin/bitcoind.pid \
                                  -conf=/home/btc-node/.bitcoin/bitcoin.conf \
                                  -datadir=/home/btc-node/.bitcoin \
                                  -startupnotify='systemd-notify --ready' \
                                  -shutdownnotify='systemd-notify --stopping'

Type=notify
NotifyAccess=all
PIDFile=/home/btc-node/.bitcoin/bitcoind.pid

Restart=on-failure
TimeoutStartSec=infinity
TimeoutStopSec=600

User=btc-node
Group=btc-node

[Install]
WantedBy=multi-user.target
```

Afin de vérifier le déroulement, ouvrir un nouveau terminal puis observer les logs par :

`tail -f ~/.bitcoin/debug.log`

Si `bitcoind` est lancé l'arrêter par `bitcoin-cli stop` , attendre l'apparition dans les log de `Shutdown: done`

Activer le service `sudo systemctl enable bitcoin.service` 

Démarrer le service `sudo systemctl start bitcoin.service` et attendre le retour du prompt.

Vérifier que le service fonctionne correctement `sudo systemctl status bitcoin.service`

Tout en observant les logs dans un terminal, lancez sur un autre `sudo reboot` vous devez voir `Shutdown: done` dans les logs juste avant que la machine s'arrête, cela signifie que le nœud a stoppé proprement. Démarrez la machine, `bitcoind` doit se lancer automatiquement. Bien sur, si le service est actif ne plus utiliser`bitcoin-cli stop,`dorénavant pour l'arrêter faire :

 `sudo systemctl stop bitcoin.service` 

Pour info si vous souhaitez désactiver ce service  `sudo systemctl disable bitcoin.service`


# Choix de connexion portefeuille à nœud personnel

Souveraineté individuelle ou ne dépendre d'aucune entité, c'est beau mais surtout important. Vous voulez effectuer un transfert de valeur : votre portefeuille est connecté à votre propre nœud, vous initiez une nouvelle transaction (origination) qui sera signée avec vos clés privées signifiant l'autorisation de dépenser les fonds référencés. Elle sera vérifiée puis annoncée par votre nœud pour être ensuite diffusée à tous les pairs. Chaque nœud vérifiera la validité de cette transaction, quand c'est fait les nœuds stockent celle-ci dans une mémoire temporaire appelée "mempool", les nœuds propagent également le "mempool" aux autres nœuds afin que chaque nœud dispose des mêmes transactions en attente. Les mineurs sélectionnent les transactions à partir du "mempool", puis ils assemblent celles-ci dans un potentiel nouveau bloc, ils sont en compétition entre eux pour résoudre un problème cryptographique. Le premier qui a trouvé gagne la récompense de bloc et tous les frais de transaction. Il propose le nouveau bloc aux nœuds qui le vérifient afin de l'intégrer dans leur copie de la "blockchain". Les nœuds mettent à jour la "mempool" en retirant toutes les transactions qui ont été intégrées dans le nouveau bloc. Là il s'est écoulé approximativement 10mn … et cela dure depuis plus de 15 ans, oui **Bitcoin EST un ovni monétaire**.

Pour connecter un portefeuille à un nœud il y a différentes solutions, cela peut demander des données et des ressources supplémentaires. En octobre 2024 j'ai comparé ces données supplémentaires avec la taille de la blockchain Bitcoin elle-même.

| Données | Taille | Information |
|----|----|----|
| blockchain | 656Go | La référence pour le calcul en % des autres données. |
| txindex | 58Go\[8%\] | `txindex=1` dans `bitcoin.conf`, ces données sont obligatoires pour peu que l'on utilise son propre nœud. |
| blockfilter/basic | 11Go\[1.7%\] | `blockfilterindex=1` dans `bitcoin.conf`, venu avec BIP158. Ces données accélèrent x5 à x10 l'analyse des blocs s'il est fait usage de la fonctionnalité wallet de `bitcoind`, soit directement ou indirectement avec des portefeuilles logiciels tiers comme Specter ou Sparrow. |
| electrum | 47Go\[7.2%\] | Bien qu'il soit possible de s'en passer, le standard SPV d'Electrum est très utilisé. Les wallets SPV n'utilisent pas la fonctionnalité wallet de `bitcoind` mais passent par une couche intermédiaire composée d'un serveur Electrum et de sa propre base de donnée.  |

Donc `bitcoind` intègre un wallet avec lequel vous pouvez interagir, les clés privées seront stockées sur le nœud bien protégées par un mécanisme de "timelock". Vous débloquez l'accès pendant un temps suffisant pour effectuer une transaction, puis c'est automatiquement re-bloqué. Tout ce passe en ligne de commande, et il est souhaitable d'effectuer régulièrement des sauvegardes. La conservation de ce minimalisme dans le code est une bonne chose mais c'est "super hard" pour les premiers venus.

Ensuite gardez à l'esprit que votre nœud est constamment connecté à Internet et donc exposé à des risques potentiels. Si le wallet intégré à `bitcoind` est utilisé (directement ou avec la librairie Cormorant), cela peut faire de vous une cible si votre solde est découvert. Pour voir le solde, les clés publiques et si les clés privées sont actives, tapez : `bitcoin-cli getwalletinfo`. Si la sortie indique bien `private_keys_enabled : false`, il sera donc impossible de dépenser à partir de là.

Afin d'élargir le choix des portefeuilles utilisables, il est opportun d'installer un serveur Electrum sur le nœud Bitcoin. Il sera ensuite possible d'utiliser le portefeuille Electrum ou un autre pourvu qu'il soit compatible avec le format SPV d'Electrum. Lorsque la date de naissance du portefeuille est inconnue (date de la 1 ère transaction) toute la blockchain doit être parcourue depuis l'origine. Le format SPV permet d'obtenir le solde bitcoin bien plus rapidement que l'utilisation directe ou indirecte du wallet de Bitcoin et ce même avec `blockfilterindex` activé.

Avant de se lancer dans d'autres installations, et que l'un n'empêche pas l'autre, ci-dessous la configuration `bitcoin.conf`  pour une connexion directe portefeuille à Bitcoin par RPC (Remote Procedure Call ou appel de procédure distante).

## Portefeuille connecté directement à Bitcoin

Si vous préférez le standard SPV d'Electrum, sautez donc ce paragraphe !

Vérifiez d'abord si le portefeuille que vous voulez utiliser est capable de se connecter directement au nœud Bitcoin. [Point de départ pour évaluer les wallets](https://bitcoin.org/en/choose-your-wallet?step=5), sélectionnez ensuite votre matériel / OS.

Rajoutez ces lignes à la fin de `bitcoin.conf` par `nano ~/.bitcoin/bitcoin.conf`

```bash
# Config pour Sparrow wallet ou autre utilisant l'accès rpc de bitcoind
#
# BIP158 : Index de filtres de blocs
# Apres avoir redemarre le noeud, bitcoind va creer cet index de filtrage, cela
# prends du temps, observer les logs ... "Syncing basic block filter index"
# Ensuite le wallet de bitcoind parcours environ 5x plus rapidement la blockchain
# a la recherche de transactions sur une adresse (rescanblockchain)
# Les logs indiqueront "fast variant using block filters" a l'oppose de
# "slow variant inspecting all blocks" si l'index n'est pas construit
blockfilterindex=1

# rpcauth permet de se connecter à Bitcoin avec authentification
# Comment generer votre clé ? voir (1)
rpcauth=btc-node:5a8257571875d0d6ee80df0741c45e5b$ceb75c8e6bde720896714a9981f0a149c068db8d7a273f66e35f70e13484e5b2

# Ferme la porte a toute connection externe
rpcbind=127.0.0.1

# C'est l'IP de votre noeud sur votre resau local
rpcbind=192.168.X.XXX

# Ferme la porte a toute connection externe
rpcallowip=127.0.0.1

# Si non défini precedemment !
# Adaptez l'adresse par celle de votre réseau local, exemple : 192.168.0.0/24
# autorise 256 adresses de (192.168.0.0 à 192.168.0.255) 
rpcallowip=192.168.X.0/24
```

(1) Afin de sécuriser l'accès RPC à `bitcoind` en utilisant une méthode plus robuste qu'un simple mot de passe en texte clair (tandem rpcuser / rpcpassword) dans `bitcoin.conf`, il faut utiliser 'RPC Auth'. Pour cela, exécuter sur votre nœud ceci : 

```bash
# En fin de ligne, adaptez l'utilisateur et le mot de passe
python3 ~/code/bitcoin/share/rpcauth/rpcauth.py btc-node who-are-you-satoshi?
```

voici la sortie :

```none
String to be appended to bitcoin.conf:
rpcauth=btc-node:5a8257571875d0d6ee80df0741c45e5b$ceb75c8e6bde720896714a9981f0a149c068db8d7a273f66e35f70e13484e5b2
Your password:
who-are-you-satoshi?
```

En fonction de vos choix, modifiez la ligne `rpcauth` de `bitcoin.conf`, redémarrez `bitcoind`, puis installez sur votre ordinateur personnel un portefeuille capable de se connecter directement à votre nœud Bitcoin, par exemple Sparrow wallet qui utilise la librairie Cormorant, puis  allez dans :

`File/Preferences puis Server`

* Activer Bitcoin Core
* URL: <ip de votre noeud> / 8332
* User / Pass: btc-node /  who-are-you-satoshi?
* Faire `Test Connection`


# Logiciel serveur Electrum

## La construction

Le serveur et le portefeuille Electrum sont apparus en 2011, ils ont été des précurseurs dans le domaine. A l'heure actuelle il existe plusieurs variantes du serveur Electrum, j'ai choisi celle qui me semble la meilleure et la plus efficiente en terme de ressources, à savoir Electrum Rust Server (en condensé `electrs`), l'installation s'effectue sur le nœud par :

```bash
sudo apt-get update
sudo apt-get install cargo clang cmake
sudo apt-get install libclang-dev

# Aller sur https://github.com/romanz/electrs/blob/master/doc/install.md
# et faire un choix pour librocksdb, link statique ou dymamique ?
# J'ai choisi link dynamique, ok quelle version est candidate ? 
apt-cache policy librocksdb-dev
# Pour Debian 12 > librocksdb-dev=7.8.3-2
# Pour Debian 13 > librocksdb-dev=9.10.0-1+b1
sudo apt-get install librocksdb-dev=9.10.0-1+b1 # ici c'est Debian 13

cd ~/code
git clone https://github.com/romanz/electrs
cd electrs

# Obtenir la version de la dernière realease
git tag | sort --version-sort | tail -n 1
# Pour moi c'est "v0.10.6"

# Facutatif : importer une première fois la signature du concepteur
curl https://romanzey.de/pgp.txt | gpg --import

# Facultatif : Vérifier que la realease que vous allez compiler est originale
git verify-tag "v0.10.6"
# Vous devez obtenir en sortie : Good signature from "Roman Zeyde <me@romanzey.de>"

# Choix de la realease
git checkout "v0.10.6"

# Construction
cargo build --locked --release
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

Comme pour Bitcoin il est souhaitable de séparer les données `electrs` de l'Operating System. Si le répertoire `~/.electrs`est présent vérifiez qu'il soit vide, ensuite supprimez le.

```bash
# Creer le repertoire electrs sur l'unite de stockage separee de l'OS
# Adaptez le début du chemin à vos choix précédents
mkdir /mnt/nvme/electrs

# Créez le lien
# Adaptez le chemin à votre peripherique si ext4 ou BTRFS
ln -s /mnt/nvme/electrs ~/.electrs
# ou
ln -s /mnt/btrfs/electrs ~/.electrs
```


## Configuration d'Electrs

```bash
mkdir ~/.electrs # si une seule unité de stockage
nano ~/.electrs/electrs.toml
```

Copier coller ceci (adaptez si l'utilisateur est différent de 'btc-node')

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

# L'adresse P2P d'ecoute de bitcoind, le port est generalement 8333.
daemon_p2p_addr = "127.0.0.1:8333"

# Repertoire dans lequel l'index Electrum doit etre stocke
db_dir = "/home/btc-node/.electrs/db"

# bitcoin signifie ici mainnet
network = "bitcoin"

# Nombre de transactions a consulter avant de renvoyer une erreur,
# ceci afin d'eviter que des adresses trop populaires ne monopolisent
# le serveur RPC electrs.
# C'est fonction de la velocite de la machine hebergeant electrs. Le defaut est 200.
# Si l'usage est de multiples et simultanees connections RPC, laisse commente.
# A contrario si l'usage est la consultation d'adresses de portefeuilles avec de
# nombreuses transactions et peu voire une seule connection RPC alors de-commentez.
# index-lookup-limit = 1000

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

## Lancer Electrum une première fois

Le démon `bitcoind` est lancé et c'est synchronisé à 100% ?

* Non → faire le nécessaire
* Oui  → poursuivre

Dans un premier temps lancer manuellement Electrum server par :

```none
electrs --conf ~/.electrs/electrs.toml
```

Attendre qu'il effectue la synchronisation, c'est long ... genre 8h sur mon RPi5 et 3h15 sur le Dell Optiplex 5050.

Electrs indexe à coup de 2000 blocs, à la fin il effectuera un compactage des données :

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
# electrs service
# /etc/systemd/system/electrs.service
# Si besoin adaptez l'utilisateur defini, ici c'est 'btc-node'

[Unit]
Description=Electrs
Wants=bitcoind.service
After=bitcoind.service

[Service]
Type=simple
KillMode=process
WorkingDirectory=/home/btc-node/.electrs
ExecStart=/usr/local/bin/electrs --conf /home/btc-node/.electrs/electrs.toml
User=btc-node
Group=btc-node
TimeoutSec=300
Restart=always
RestartSec=60
Nice=5
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=electrs

[Install]
WantedBy=multi-user.target
```

Afin de vérifier le déroulement, ouvrir un nouveau terminal puis observer les logs par :

`sudo journalctl -f | grep electrs`

Activer le service `sudo systemctl enable electrs.service`

Démarrer le service `sudo systemctl start electrs.service`

Vérifier que le service fonctionne correctement`sudo systemctl status electrs.service`

Observer les logs dans le terminal, c'est synchronisé avec le dernier bloc, tout est ok ? oui, alors lancez `sudo reboot` vous devez voir `Stopped electrs.service - Electrs` dans les logs juste avant que la machine s'arrête, cela signifie que `electrs` a stoppé proprement. Au redémarrage de la machine, `bitcoind` doit se lancer automatiquement suivi par `electr`. Pour l'arrêter, par exemple pour une mise à jour de `electrs.toml` faire :

`sudo systemctl stop electrs.service`

Si plus tard vous effectuez de l'exploration didactique avec le wallet Electrum sur des portefeuilles en lecture seule avec des adresses de mineurs, il est possible que le serveur `electrs` ne réponde plus pendant un certain temps. Si cela se produit, fermez le portefeuille qui a engendré le time-out, observez les logs d'`electrs`,  puis choisissez une des 2 options :


1. Lancez `sudo systemctl stop electrs.service` , ensuite il est nécessaire d'attendre patiemment 5mn pour retrouver la main, c'est le temps du `TimeoutSec=300` paramétré dans le fichier `electrs.service` , relancez ensuite le service.
2. Ne faites rien et généralement en moins de 10mn quand vous verrez dans les logs `disconnecting due to failed to send response` le serveur `electrs` sera de nouveau opérationnel.

## Chiffrer la connexion

Afin d'établir une connexion chiffrée entre le portefeuille (que vous devrez installer sur votre ordinateur personnel) et votre nœud, il faut installer et configurer Ngnix.

Prérequis : créer un certificat auto signé. A chaque champ attendu appuyez sur la touche `Enter`, sauf à `Common Name`, mettez `Nakamoto` ou autre chose sans laisser le champ vide.

```bash
openssl req -x509 -nodes -days 10227 -newkey rsa:2048 -keyout ~/.electrs/electrs.local.key -out ~/.electrs/electrs.local.crt
```

Cette commande a généré la clé privée RSA `electrs.local.key` utilisée pour signer le certificat `electrs.local.crt`, le certificat lui même est valide pour 28 ans. D'ici là je serais crevé.

Voir le certificat en clair : `openssl x509 -in ~/.electrs/electrs.local.crt -text -noout`

Ceci est non obligatoire mais c'est le "secret parfait" : générer un certificat encodé PEM groupe Diffie-Hellman. Des clés de session éphémères seront utilisées afin de garantir que les communications passées ne peuvent pas être déchiffrées si wxcvbn

la clé de session est compromise.

`time sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096`

C'est long comme la mort … 46mn sur mon RPi5 et infiniment plus rapide sur le Dell … 3mn23s Pour voir si ok : `cat /etc/ssl/certs/dhparam.pem`

Installer Ngnix et vérifier que le service est actif :

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
# configuration Ngnix pour electrs
# Ce fichier de configuration doit etre dans le chemin
# /etc/nginx/modules-enabled
#
# Si besoin adaptez l'utilisateur defini, ici c'est 'btc-node'

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

        ssl_ciphers 'TLS13+AESGCM+AES128:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';

        ssl_session_cache shared:SSL:50m; # Verif si pas defini ailleurs
        ssl_session_timeout 1d;
        ssl_session_tickets off;
        ssl_ecdh_curve X25519:sect571r1:secp521r1:secp384r1;
        
        # Securite supplementaire avec le groupe 'Diffie-Hellman'
        #ssl_dhparam /etc/ssl/certs/dhparam.pem;
    }
}
```

Si vous avez installé `dhparam.pem`dé-commentez `ssl_dhparam /etc/ssl/certs/dhparam.pem;` 

Vérifiez la configuration par `sudo nginx -t` la réponse doit être `syntax is ok`, démarrer par `sudo systemctl start nginx.service`

Pour information, tester la clé puis voir le certificat généré :

`sudo openssl rsa -in ~/.electrs/electrs.local.key -check`

`openssl x509 -in ~/.electrs/electrs.local.crt -text -noout`

Afin de vérifier le fonctionnement, installez un portefeuille compatible SPV Electrum sur votre ordinateur personnel , puis paramétrez "Network" à  Server: `IP_de_votre_noeud:50002`,  si l'option est disponible activez SSL (si elle ne l'est pas, à priori c'est déjà SSL)

*Si vous n'utilisez que ce type de portefeuille, vous pouvez désactiver le wallet de bitcoind en rajoutant la ligne* `disablewallet=1` *à la fin de* `bitcoin.conf`

## Utiliser Tor

Ou que vous soyez sur la planète, si vous avez accès à internet vous aurez également accès à votre serveur Electrum et pourrez interagir avec la blockchain Bitcoin à travers votre propre nœud en toute confidentialité. A travers Tor c'est plus lent tout en étant parfaitement utilisable.

Editer le fichier de configuration `sudo nano /etc/tor/torrc` et rajouter ces lignes à la fin :

```bash
# Configuration pour electrs
HiddenServiceDir /var/lib/tor/electrs_hidden_service/
HiddenServiceVersion 3
HiddenServicePort 50001 127.0.0.1:50001
```

* Stopper bitcoind : `sudo systemctl stop bitcoin.service`
* Re-démarrer Tor : `sudo systemctl restart tor`
* Relancer bitcoind : `sudo systemctl start bitcoin.service`
* Obtenir l'adresse Tor pour connecter son portefeuille à son nœud :

  `sudo cat /var/lib/tor/electrs_hidden_service/hostname`

Sur votre ordinateur personnel si vous avez besoin d'activer Tor lancez Tor Browser par exemple, connectez vous et laissez ouvert. Sur votre portefeuille paramétrez "Network" à  Server: `votre_adresse.onion:50001`, si l'option est disponible dé-activez SSL et activez Tor.

Sur certains portefeuilles comme Electrum mettez un t à la fin comme `votre_adresse.onion:50001:t`, puis à l'onglet Proxy cochez Use Tor Proxy.


# Les portefeuilles

En interagissant avec la blockchain Bitcoin un portefeuille permet de visualiser son solde, de visualiser ses transactions passées, d'initier de nouvelles transactions puis de les signer et de les propager, de visualiser ses adresses de réception qui ont été générées avec ses clés elles mêmes générées avec sa seed phrase. Un portefeuille est aussi un coffre-fort contenant ses clés. Afin d'appréhender correctement ce qu'est un portefeuille il est nécessaire de comprendre certaines notions. Cela permettra également de faire des choix en connaissance de cause.

## Seed phrase

Également connue sous le nom de phrase de récupération ou phrase mnémonique, cette suite de mots est générée lors de la création du portefeuille. Elle est généralement composée de 12, 18 ou 24 mots choisis aléatoirement dans une liste prédéfinie de 2048 mots anglais, elle permet de restaurer l'accès à son portefeuille en cas de perte ou destruction des clés. Bien évidemment il est très important de conserver cette liste en lieu sûr; mais également puisque qu'à partir de celle-ci n'importe qui peut effectuer des transactions en créant un doublon du portefeuille. De plus gardez en mémoire qu'il est toujours utilisé des fonctions à sens unique pour passer de la seed phrase à la clé privée maîtresse ainsi qu'à toutes les autres clés et ce jusque aux adresses publiques Bitcoin.

La seed phrase a grandement simplifié et sécurisé la gestion des clés en permettant de dériver hiérarchiquement toutes celles-ci à partir d'un ensemble de mots. Les portefeuilles utilisant la seed phrase sont qualifiés de déterministes (ou HD Wallet, pour Hierarchical Deterministic Wallet). Cela permet également une interopérabilité sécurisée entre différents portefeuilles (exemple : re-créer à l'identique avec un autre logiciel et ou matériel)

Le concept de seed phrase a été officiellement standardisé en 2013 avec le protocole **BIP-39** (Bitcoin Improvement Proposal). BIP-39 a été proposé par Marek Palatinus (alias Slush), Pavol Rusnak (alias Stick), et d'autres membres de la communauté Bitcoin.

La norme de seed phrase BIP39 est la plus utilisée, et donc la plus universelle pour le moment.

## Choix du "Pay to"

Le terme "Pay to" fait référence aux mécanismes d'adressage utilisés pour envoyer des Bitcoins à une adresse spécifique, chacun étant basé sur une logique cryptographique différente. Ces mécanismes déterminent comment les fonds sont verrouillés et déverrouillés, influençant la sécurité, les frais de transaction, la compatibilité et les cas d'utilisation.

Le choix du `P2` ou du type de script n'est pas anodin ! Commençons un tour d'horizon avec une traduction des conseils lus dans les [sources](https://github.com/bitcoinknots/bitcoin) de [Bitcoin Knots](https://blockdyor.com/bitcoin-knots/) de Luke Dashjr :

* Base58 (Legacy) : présente la compatibilité la plus élevée. *Le meilleur choix pour la santé du réseau Bitcoin.*  Entraîne des frais plus élevés. Recommandé. `P2PKH`
* Base58 (P2SH Segwit) : compatible avec la plupart des anciens portefeuilles, il peut entraîner des frais inférieurs à ceux de Legacy. `P2SH-P2WPKH`
* Native Segwit (Bech32) : Frais inférieurs à ceux de Base58, mais certains portefeuilles anciens ne le prennent pas en charge. `P2WPKH`
* Taproot (Bech32m) : Frais les plus bas, mais la prise en charge des portefeuilles est encore limitée. `P2TR`

J'ai obtenu cela en téléchargeant les sources de Bitcoin Knots et en effectuant cette recherche :

 `find  ~/code/bitcoin -type f -exec grep -H 'Widest compatibility and best' {} \;` 


[Tableau d'utilisation des "Pay to"](https://unchained.com/blog/bitcoin-address-types-compared/?utm_campaign=btcmag-launch) depuis l'origine de Bitcoin

| **Type** | **1 ère vue** | **En % de l'offre BTC ¹** | **Utilisation ¹** | Appellation courante | **Encodage** | **Préfixe addr** |
|----|----|----|----|----|----|----|
| P2PK | Jan 2009 | 9% (1.7M) | Obsolète | aucune c'est P2PK | Base16 | 1 |
| P2PKH | Jan 2009 | 43% (8.3M) | Diminue | Legacy | Base58 | 1 |
| P2SH | Apr 2012 | 24% (4.6M) | Diminue | aucune c'est P2SH | Base58 | 3 |
| P2WPKH | Aug 2017 | 20% (3.8M) | Augmente | Segwit ² | Bech32 | bc1q |
| P2WSH | Aug 2017 | 4% (0.8M) | Augmente | Segwit ² (P2WSH) | Bech32 | bc1q |
| P2TR | Nov 2021 | 0.1% (0.02M) | Augmente | Taproot | Bech32m | bc1p |

¹ valable pour l'année 2024, ces données sont sujettes au changement

² Segwit employé tout seul veut dire ici Native Segwit


Description des `Pay to` :

* `P2PK` `Pay-to-Public-Key` (paiement à la clé publique)
* `P2PKH` `Pay-to-Public-Key-Hash`(paiement à l'empreinte de clé publique)
* `P2MS` `Pay-to-Multisig` (paiement à des signatures multiples → obsolète utiliser `P2SH` )
* `P2SH` `Pay-to-Script-Hash`(paiement à l'empreinte de script)
* `P2WPKH` `Pay-to-Witness-Public-Key-Hash`(paiement à l'empreinte de clé publique témoin)
* `P2WSH` `Pay-to-Witness-Script-Hash`(paiement à l'empreinte de script témoin)
* `P2TR` `Pay-to-Taproot` (paiement à Taproot, "racine pivotante ou principale")


Dans les faits Il y a encore deux `Pay to` avec `Nested ou Wrapped ou Legacy Segwit` qui encapsule SegWit dans le script de P2SH :

* `P2SH-P2WPKH Nested segwit` enveloppe une adresse "Native Segwit" `P2WPKH` dans une adresse `P2SH`,  3 est leur préfixe et l'encodage est Base58. Appelé aussi `p2sh-segwit`


* `P2SH-P2WSH Nested segwit` enveloppe une adresse "Native Segwit" `P2WSH` dans une adresse `P2SH`,  3 est leur préfixe  et l'encodage est Base58.

Cela a constitué une solution de transition permettant de bénéficier des améliorations de SegWit tout en restant compatible avec les systèmes qui n'étaient pas à jour ou Segwit Natif, les nœuds entre autres. À première vue, les adresses Segwit imbriquées (nested) ne se distinguent pas des autres adresses P2SH. 

### Segwit

a été activé au bloc 481 824 le 24 août 2017, **c'est le premier soft fork du protocole Bitcoin**. SegWit est la contraction de Segregated Witness. Dans les faits cela sépare (segregate) les données de signature (witness data) des données de transaction principale. C'est par la [BIP 141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) qu'il a été proposé le 21 décembre 2015 à la communauté pour améliorer la capacité transactionnelle du réseau et corriger la malléabilité des transactions.

* Correction de la malléabilité des transactions : précédemment à Segwit un attaquant était en mesure de changer les identifiants de transaction (TXID) avant que la transaction ne soit confirmée. Cela pouvait entraîner des complications, notamment dans les transactions à signatures multiples. Cette correction a également facilité l'implémentation des canaux de paiement qui forment la base du réseau Lightning.
* En séparant les données de transaction des signatures et en introduisant une nouvelle métrique, Segwit a amélioré la capacité transactionnelle du réseau Bitcoin. Cette métrique est nommée WU (Weight Unit ou Unité de Poids), elle quantifie virtuellement la taille des transactions dans le bloc comme ceci :
  * la limite est fixée à 4 MWU (Mega Weight Unit = 1 million de WU) par bloc avec
    * `WU = 4 x taille transaction hors witness + taille du witness`(¹)
  * Un bloc de transactions unique Legacy occuperait environ 1Mo puisque
    * `4 x 1Mo + 0 = approximativement 4MWU`
  * Un bloc de transactions uniquement Segwit augmenterait la taille réelle du bloc d'un facteur inférieur à 4. Certains calculs théoriques indiquent que dans ces conditions la taille absolue qu'un bloc peut atteindre est quelque chose comme 3,7 Mo. 
  * En réalité la [taille des blocs](https://www.blockchain.com/fr/explorer/charts/blocks-size) est variable en fonction des types de transactions inclues dans celui-ci, de l'activité et de l'usage du réseau, la moyenne constatée en début d'année 2025 se situe à 1,85Mo par bloc avec un MWU proche de 4.
  * Avant Segwit la taille des transactions était réellement mesurée en octet et limitée depuis l'origine de Bitcoin à 1Mo, à sa façon Segwit a donc levé cette barrière.
* Segwit permet également une réduction conséquente des frais de transaction évalués en `sat/vB` (1 satoshi = 0,00000001 BTC et vB = virtual byte ou octet virtuel. La définition du vB est WU de la transaction divisé par 4 (vB=WU÷4), cela permet de comparer les transactions Legacy et SegWit sur une base commune.  Avant Segwit les frais étaient mesurés en fonction de la taille en octets de la transaction, avec Segwit c'est en fonction du WU ÷ 4  soit `vB = taille transaction hors witness + taille du witness ÷ 4`. Donc au point de vue des frais Segwit est avantagé par rapport à Legacy, de plus vu que le bloc est limité à 4MWU, les mineurs sont incités à inclure les transactions possédant un bon ratio entre ses frais et son poids en WU.

(¹) Pour éviter toute confusion entre x3 et x4 due à la définition officielle [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) où WU est présenté ainsi : `WU = taille TX hors witness x 3 + (taille TX hors witness + taille du witness)` **est identique à** `WU = 4 x taille TX hors witness + taille du witness` 

Informations complémentaires

* les tailles sont évaluées en octets, donc :
  * Chaque octet de la partie non-témoin d'une transaction compte pour 1 octet virtuel.
  * Chaque octet de la partie témoin d'une transaction compte pour 1/4 d'un octet virtuel.
* Le poids d'un bloc est simplement la somme des poids de toutes les transactions qu'il contient, il n'y a donc qu'une seule et même formule WU pour les transactions et les blocs.
* Les données hors witness incluent : version, nombre d'entrées, entrées elles-mêmes (sans scripts témoins), nombre de sorties, sorties, locktime.
* Les données witness incluent : signatures et scripts de déverrouillage.


### **Taproot**

a été activé au bloc 709 632 le 14 novembre 2021, c'est le deuxième soft fork et la dernière évolution du protocole. Il a été proposé initialement pour améliorer la confidentialité et l'efficacité des transactions Bitcoin. 3 BIPs en sont l'origine, ces propositions d'améliorations ont commencé en janvier 2018.

* BIP340 : Introduit les signatures Schnorr permettant de combiner plusieurs signatures en une seule. Les adresses et les transactions multi-signatures sont indiscernables des simples. Cela améliore l'efficacité, la confidentialité et la taille des transactions, diminue les frais des transactions multi-signatures ou utilisant des scripts complexes.
* BIP341 : mise à jour Taproot. En n'exposant que les détails de la transaction exécutée, Taproot offre une plus grande confidentialité aux utilisateurs. En effet les informations de transactions non exécutées, qui peuvent contenir des informations privées sensibles, ne sont plus enregistrées sur la blockchain.
* BIP342 : introduit Tapscript, cela dote Bitcoin d'un langage de programmation de transactions amélioré, axé sur la technologie Schnorr et Taproot. Permet des scripts de transactions plus flexibles et puissants.

En résumé, Taproot optimise l'espace dans les blocs et a introduit des fonctionnalités avancées comme les contrats intelligents. Cela facilite ou permet de nouvelles choses, comme par exemple l'introduction des Ordinals en optimisant la capacité de stockage des blocs et en simplifiant le stockage de données arbitraires. Les Ordinals sont un protocole qui permet d'inscrire des données (comme des images, des textes ou des vidéos) directement sur des satoshis individuels dans la blockchain. Ainsi un satoshi peut devenir un actif numérique unique, similaire à un NFT (Non-Fungible Token). 

Certains diront que l'on s'éloigne vraiment de la philosophie initiale et de la simplicité originale de Bitcoin, **dans le fond ils n'ont pas tord !**

A l'opposé des NFT, des contrats intelligents de la finance décentralisée et des "stablecoins", ces nouveautés facilitent également la réalisation des couches L2 sensées effectuer des micro-transactions quasi instantanées pour des frais réduits, ce dont la couche L1 est incapable avec 7 transactions par seconde. A bien y réfléchir : les sommes importantes n'ont nullement besoin d'être déplacées à la vitesse de la lumière, donc Bitcoin L1 convient parfaitement offrant une sécurité optimale. Les petits montants n'ont pas ce besoin de sécurité poussée donc les L2 sont tout à fait adaptées pour de nombreuses transactions quasi-instantanées. Cela permet de couvrir un large spectre d'utilisation sans dénaturer le concept initial de Bitcoin et c'est tant mieux !

***Conclusion pour le long court*** *: sur les différents matériels et logiciels, le standard qui se dégage pour le moment est Segwit native, **et même si Legacy est "vieillissant" il très bien supporté*** ! *Ceci étant faites vos propres recherches et choisissez en connaissance de cause.*

Pour information voici les "Pay to" possibles avec le portefeuille Electrum :

* legacy `P2PKH`
* p2sh-segwit ou wapped segwit `P2SH-P2WPKH`
* segwit `P2WPKH`
* ne supporte pas Taproot fin 2024

## Inter-opérabilité des `Pay To`

**Les scripts** `Pay To` **sont tous interopérables au niveau du protocole Bitcoin et ce de manière transparente pour l'utilisateur.** Si vous rencontrez un problème dans lequel un type d'adresse ne peut pas être envoyé à un autre, il ne s'agit pas d'une limitation du code Bitcoin, mais d'une limitation du portefeuille utilisé.

Parfois il est dit : "Les adresses Legacy (1..) ne sont pas compatibles avec Segwit (bc1q..)", cela signifie simplement qu'une adresse Legacy ne peut pas envoyer de transaction Segwit. Le type de transaction et la possibilité pour l'adresse de réception d'utiliser pleinement tous les avantages de la transaction dépendent de l'adresse d'envoi. Une adresse Legacy P2PKH peut transférer les droits de propriété BTC sur une adresse Segwit, cependant la transaction sera P2PKH et pas Segwit. Dans le détail cela donne :

* Depuis une adresse Legacy (1..) vers une adresse Segwit (bc1q..)
  * L'UTXO source sera au format Legacy
  * L'UTXO de destination sera au format Segwit
  * Les frais seront basés sur la taille Legacy
* Depuis une adresse Segwit (bc1q..) vers une adresse Legacy (1..)
  * L'UTXO source sera au format Segwit
  * L'UTXO de destination sera au format Legacy
  * Les frais seront basés sur une transaction mixte (entrée Segwit, sortie Legacy)

## La collision

Les adresses Bitcoin sont donc générées en aveugle et au hasard. A la création de mon portefeuille est t'il possible que je tombe sur une adresse ne m'appartenant pas ou qu'un jour un individu lambda tombe sur la mienne ? 

**NON,** ce phénomène aussi appelé collision d'adresse est impossible !

Si la dose de hasard (l'entropie) utilisée pour sélectionner l'adresse dans un ensemble titanesque(¹) d'adresses est suffisante, alors c'est statistiquement impossible.

Avec un hasard bien construit et vu la taille du nombre (¹)

1 460 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000

cela n'arrivera jamais.

Bilan : j'ai fait mes propres recherches, je peux générer un gros paquet d'adresses tranquillement afin de ne jamais les réutiliser et ainsi préserver ma confidentialité. C'est d'ailleurs ce dont les portefeuilles modernes déterministes sont capables, le nombre théorique d'adresses Bitcoin par seed phrase et par "account" du chemin de dérivation est de 2³¹ × 2, soit 4.29 milliards. Comme il n'est pas souhaitable d'afficher des milliers d'adresses vides, les portefeuilles n'affichent en règle générale que les premières. Par défaut le portefeuille Electrum ainsi que d'autres arrêtent de les lister lorsqu'ils trouvent 20 adresses consécutives sans transactions.

*Le génie de ce système est qu'il ne nécessite aucune coordination centrale pour l'attribution des adresses. Chaque utilisateur peut générer ses adresses de manière totalement indépendante, sans avoir besoin de vérifier si elles existent déjà. Ce principe simple mais puissant est l'un des aspects qui préserve la décentralisation de Bitcoin.*

(¹) Une adresse Bitcoin est dérivée d'un hachage `RIPEMD-160(SHA-256(clé_publique))`, ce qui donne un espace de 2¹⁶⁰ possibilités, soit environ 1.46 × 10⁴⁸ adresses uniques.

## Le portefeuille logiciel Electrum

Comme dit précédemment, un portefeuille est aussi un coffre sécurisé contenant ses clés, le logiciel Electrum peut les générer lui même et les encrypter par un mot de passe sur le disque dur du poste de travail : *c'est le Hot Wallet*. *Remarque importante : si vous créez la seed phrase avec Electrum elle sera incompatible BIP39, les développeurs d'Electrum estiment qu'elles ne répondent pas à leurs normes de sécurité car les semences BIP39 ne comportent pas de numéro de version, ce qui compromet la compatibilité avec les futurs logiciels. Pour plus de détail voir [Electrum Seed Version System](https://electrum.readthedocs.io/en/latest/seedphrase.html). Si vous souhaitez du BIP39 utilisez un portefeuille matériel comme décrit ci-dessous ou à des fins de tests générez avec BIP39 tools la seed phrase "BIP39 Mnemonic" puis créez un nouveau portefeuille avec Electrum, faites "Options" et cochez BIP39 seed.*

Electrum est également capable d'interagir avec un portefeuille matériel du type Trezor, Ledger, BitBox02, etc … les clés seront alors générées et stockées dans le dispositif, dans ce cas aucune clé privée ne sera stockée sur le poste de travail : *c'est le Cold Wallet*. A la création de votre cold wallet avec Electrum si vous avez choisi de l'encrypter avec le dispositif vous ne pourrez l'ouvrir que si le dispositif est branché et déverrouillé. Si votre choix a été de ne pas l'encrypter avec le dispositif vous pourrez l'ouvrir sans celui ci en mode spectateur uniquement, cela sous entend que certaines clés et adresses publiques seront stockées en clair sur votre disque dur.

Ci-dessous, je décris ici la méthode d'installation `Appimage` pour un "Desktop Linux".

**Dans tous les cas vérifiez toujours que l'exécutable est original !**

Sur <https://electrum.org/> à la rubrique `Download` téléchargez l'Appimage dans un sous répertoire du home de l'utilisateur.

Ensuite il est primordial de vérifier le fichier `electrum-X.x.x-x86_64.AppImage` avec la signature d'un des concepteurs, ici Thomas Voegtlin. A partir de la même page téléchargez dans le même répertoire la clé publique `ThomasV.asc` pour moi cela donne ceci dans un terminal :

`wget https://raw.githubusercontent.com/spesmilo/electrum/master/pubkeys/ThomasV.asc`

Obtenez sa signature RSA par `gpg --show-keys ThomasV.asc` puis vérifiez la avec un événement, comme cette [vidéo tournée le 7 septembre 2016](https://www.youtube.com/watch?v=hjYCXOyDy7Y) ou apparaît sa signature publique au début, si cela correspond vous avez vérifié sa clé !

Importez la clé dans votre système par `gpg --import ThomasV.asc` 

Vérifiez le fichier `Appimage` par  `gpg --verify electrum-4.5.8-x86_64.AppImage.asc` vous devez obtenir en sortie `Bonne signature de Thomas Voegtlin`

Par `chmod u+x appimage_file` donner le droit d'exécution, créez un raccourci sur le bureau puis lancez Electrum.

Cocher `Advanced network settings` et `Select Server` puis à `Serveur` renseigner : `ip_ou_hostname-de_votre_noeud:50002` ensuite décochez Select `server automaticaly`

La syntaxe complète de connexion à un serveur comme `elects` est une URL du format  `host:port:[t|s]` (**t** pour tor, **s** pour ssl mais pas obligatoire pour le portefeuille Electrum)

Vous pouvez ensuite interdire au portefeuille Electrum de scanner d'autres serveurs que le votre en éditant manuellement sa configuration par `nano ~/.electrum/config` chercher `"oneserver": false` puis remplacez `false` par `true` , relancez Electrum allez à network, il ne teste  plus les adresses différentes de la votre.

A des fins didactiques avec Electrum il est possible d'importer une adresse Bitcoin pour visualiser toutes les transactions effectuées sur celle ci, bien sur c'est en lecture seule (mode spectateur). Attention si vous tapez une adresse avec des milliers de transactions cela risque d'être long et risque de provoquer un time-out d'electrs si la machine est peu véloce. Adresses que j'ai testées avec succès sur mon nœud Dell Optiplex en laissant `index-lookup-limit` au paramétrage par défaut, par contre avec mon Raspberry Pi5 j'ai du modifier `electrs.toml` en rajoutant `index-lookup-limit = 1000`  :

* Bloc 210 000 / Bloc du 1ᵉʳ halving adresse du mineur / last tx 22.02.2013 / 5206 transactions
  * `1NEU779yvLaFk39k4Q3QdLjwpWTdWCbzqL`
* Bloc 873 630 /  adresse du mineur / actif en 2025 / environ 1230 transactions :
  * `bc1qrpp7g75sx3ejclvsfdw2uahzchtyu7vumkuadu`
* Bloc 873 652 / adresse du mineur / actif en 2025 / environ 2300 transactions
  * `3Awm3FNpmwrbvAFVThRUFqgpbVuqWisni9`

Les autres portefeuilles capables de créer un wallet spectateur d'après une adresse publique sont Bitcoin et BlueWallet qui accepte la connection SPV à un serveur Electrum et également la connexion RPC à `bitcoind` (exécutables pour Android, IOS et macOS).

Le portefeuille Electrum dispose également d'une console Python intégrée, cela en fait un outil extrêmement puissant qui donne accès direct au cœur du wallet, il est possible de lire, écrire, modifier, automatiser, débugger presque tout ; à condition de savoir ce que l'on fait !

**Pour connecter Electrum Android à votre nœud via TOR :**

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
* il procurent une couche de sécurité complémentaire aux portefeuilles logiciels

Cette séparation des rôles est fondamentale pour la sécurité : même si votre ordinateur est compromis, un attaquant ne peut pas signer de transactions sans avoir accès physique au portefeuille matériel et votre approbation active.

**Les 6 règles d'or du cold wallet :**


1. L'achat s'effectue directement chez le fabricant, jamais de tiers entre vous et le concepteur.
2. A réception, toujours inspecter l'emballage qui doit être scellé à un ou plusieurs niveaux.
3. Vérifiez l'authenticité du dispositif et installez vous même le firmware original du wallet.
4. Seulement deux supports doivent reporter les mots de votre seed phrase : l'écran de votre portefeuille matériel qui listera les mots en principe une seule et unique fois lors de la création du wallet et le support physique où seront inscrit les mots de votre main. 
5. Après avoir crée votre portefeuille et avant de l'utiliser toujours tester votre sauvegarde en ré-initialisant le device et en ressaisissant les mots sur le portefeuille matériel lui même.
6. Si votre cold wallet est fonctionnel mais que la seed phrase est perdue ou compromise, créez un nouveau portefeuille avec une nouvelle seed phrase puis transférez les fonds de l'ancien vers le nouveau portefeuille. (exception faite de BitBox02 si la seed est perdue il est possible de la re-visualiser)

**Comparatif de 4 cold wallets :**

| Critère | Trezor Safe 3 Bitcoin-only edition | Ledger Nano S Plus | BitBox02 Bitcoin-only edition | **Tangem** |
|:---|----|----|----|----|
| **Format** | Clé usb | Clé usb | Clé usb | Carte de crédit ou anneau |
| **Matériel et fabricant du secure element (¹)** | Open source sauf le secure element EAL6+ `Infineon Technologies` | Partiellement open source, et non open source pour le secure element EAL5+ `STMicroelectronics` ou EAL6+ `Infineon Technologies` suivant l'année de fabrication. | Qualifiable d'open source car le secure element `Microchip Technology` ne stocke pas les clés privées. | Propriétaire, Secure element EAL6+ `Samsung` |
| **Vérification, installation du firmware et de l'app Bitcoin sur le dispositif - OS supportés** | Utilisation **obligatoire** de [Trezor Suite](https://trezor.io) (logiciel open source, supporte Tor)  - Windows / macOS / Linux / Android / iOS (portefeuille en lecture seule) | Utilisation **obligatoire** (²) de [Ledger Live](https://www.ledger.com) (logiciel open source, ne supporte pas Tor) - Windows / macOS / Linux(³) / Android / iOS non compatible | Utilisation **obligatoire** de BitBoxApp (logiciel open source, supporte Tor) - Windows / macOS / Linux(⁴) / Android | Utilisation **obligatoire** de Tangem Crypto Wallet (ios / android) application mobile pour interaction avec la carte ou le ring par NFC. |
| **Support d'un nœud personnel par la suite logicielle du fabricant** | Oui avec le format Electrum SPV en clair, SSL ou TOR. Supporte également Blockbook personnel. | Oui par une connection RPC à Bitcoin dont le wallet doit être actif. Il est nécessaire d'installer Ledger SatStack, logiciel open-source qui est une passerelle entre Ledger Live et le nœud Bitcoin complet. | Oui avec le format Electrum SPV en clair, SSL ou TOR | Non |
| **Création du wallet et initialisation du PIN** | Electrum, Rabby, BlueWallet ... ou Trezor Suite | Uniquement sur le dispositif. | BitBoxApp  obligatoire | Tangem Crypto wallet |
| **Phrase de récupération Single Seed BIP39** | Trezor Suite avec "anciennes sauvegardes" en 12 ou 24 mots mais capacité à restaurer en 18mots. Avec Electrum 12,18 ou 24 mots. | Création en 24 mots uniquement mais capacité à restaurer un portefeuille avec une phrase de récupération de 12, 18 ou 24 mots. | Création en 12 ou 24 mots avec capacité à restaurer un portefeuille avec une phrase de récupération de 12, 18 ou 24 mots. | Le fabricant privilégie un backup dans plusieurs exemplaires du dispositif sans phrase de récupération, il propose néanmoins 12 ou 24 mots. |
| **Vérifier sur le dispositif  la phrase de récupération** | Oui | Oui en installant au préalable sur le device l'app Recovery Check à partir de Ledger Live | Oui | Non testé |
| **Re-visualiser la phrase de récupération sur le dispositif (⁵)** | Non | Non | Oui | Non testé |
| **Option portefeuille caché et protégé par une passphrase** | Oui | Oui | Oui | Non |
| **Phrase de récupération autre** | Oui (⁶) | Non | Non | Non |
| **"Pay to" par défaut** | Segwit | Segwit | Segwit | Segwit |
| **Autres "Pay to" supportés par la suite logicielle du fabricant** | Legacy / P2SH-Segwit / Taproot | Legacy / P2SH-Segwit / Taproot | P2SH-Segwit / Taproot (⁷) | Legacy |
| **Wallets tiers supportés** | Electrum, Sparrow, Specter … | Electrum, Sparrow, Specter … | Electrum, Sparrow, Specter … | Aucun |
| **Wallets mobiles supportés (système)** | Trezor Suite Lite (Android) ou version Web de Trezor Suite par WebUSB - Mycelium (iOS / Android) | Ledger Live (Android) | BitBoxApp avec les mêmes fonctionnalités que la version desktop (Android) | Tangem Crypto Wallet (iOS / Android) |
| **Autres fonctionnalités** | Personnalisation écran d'accueil avec image n/b 128x64 pixels | Possibilité de créer un deuxième code PIN lié au portefeuille caché et protégé par une passphrase | Sauvegarde de la seed phrase et des notes de transactions sur microSD. Brouillage du signal usb entre le dispositif et l'appareil auquel il est raccordé. | Non testé |
| **Réinitialisation au paramètres d'usine (effacement de toutes les données utilisateur)** | par Trezor Suite ou sur le dispositif après saisie de 16 PIN erronés ou par saisie d'un PIN dédié (wipe code) à la place du PIN de déverrouillage | Réinitialisation aux paramètres d'usine (⁸) par le Control Center de Ledger Live, sur le dispositif par appui long des 2 boutons (settings) ou après la saisie de 3 PIN erronés. | Réinitialisation aux paramètres d'usine avec BitBoxApp ou après la saisie sur le dispositif de 10 PIN erronés  | Uniquement avec Tangem Crypto Wallet |
| **Multi-portefeuill**e | Oui (⁹) | Oui (⁹) | Oui (⁹) | Oui (⁹) |
| **Entreprise - Pays d'origine** | [SatoshiLabs](https://trezor.io) - République tchèque | [Ledger SAS](https://www.ledger.com) - France | **[Shift Crypto](https://bitbox.swiss/) -** Suisse | [Tangem AG](https://tangem.com) -Suisse |
| **Achat du dispositif** | Fiat ou bitcoin | Fiat | Fiat ou bitcoin | Fiat ou bitcoin |

(¹) à l'heure actuelle il n'existe pas de "secure element" open source, en principe c'est dans cette puce sécurisée que les clés privés sont stockées.

(²) Pas tout à fait, il existe une alternative avec [Bacca](https://github.com/darosior/ledger_installer).

(³) Comme indiqué sur le portail Ledger, sous Linux il est nécessaire de fixer les règles UDEV par : `wget -q -O - https://raw.githubusercontent.com/LedgerHQ/udev-rules/master/add_udev_rules.sh | sudo bash`

(⁴) **Bien que non expliqué sur le site du fabricant**, sur mon Desktop Linux il a été nécessaire de fixer les règles UDEV avec un script téléchargeable [ici](https://github.com/BitBoxSwiss/bitbox-wallet-app/blob/master/frontends/qt/resources/deb-afterinstall.sh), si vous utilisez seulement BitBox02 commentez les deux lignes sous `# BitBox V1 udev rules` rendez le exécutable par `chmod u+x deb-afterinstall.sh` puis lancez le par `sudo ./deb-afterinstall.sh`. Heureusement que certains se sont donné la peine avec [ceci](https://github.com/spesmilo/electrum/tree/master/contrib/udev) ou est répertorié les règles UDEV pour la majorité des hardwares wallets. Pour lister toutes les règles que vous avez mis en place : `sudo ls /etc/udev/rules.d/`

(⁵) BitBox02 est le seul dispositif que j'ai testé qui permet de voir plus d'une fois la seed phrase :

* avantage, si la seed phrase est perdue il est possible de la ré-écrire sur un support physique ou de la sauvegarder à nouveau sur micro SD.
* inconvénient, si une personne mal intentionnée à accès au dispositif et qu'elle connait le code PIN, en dehors du vol direct comme sur tous les autres dispositifs il faut considérer l'effet pervers du temps que cela peut engendrer. En effet cette personne sera en mesure de créer son backup personnel et de déplacer les fonds plus tard …

(⁶) Autres phrases de récupération sur Trezor wallet :

* Trezor Suite propose SLIP39 Shamir en 20 mots uniquement, avec Single-Share Backup (1 seul fragment) ou du Multi-Share Backup (2 à 16 fragments), il est possible de produire le Backup Multi-Share avec Single-Share. Les deux seront valables, à l'utilisateur de détruire ensuite le Single-Backup.


* Electrum est capable de créer un portefeuille avec un dispositif initialisé (vérification, installation du firmware et du micrologiciel Bitcoin avec Trezor Suite car à la livraison le matériel est en "bootloader mode"). Il peut initier la procédure pour nommer l'appareil et activer son code Pin. En plus de BIP39 et avec "Show expert settings" Electrum propose la seed SLIP39 Shamir ou Super Shamir avec 20 ou 33 mots. Si vous êtes vraiment un expert, est proposé en plus : avec la phrase supplémentaire (hidden wallet) ou le Seedless Mode (master seed sans backup). Une fois le portefeuille ouvert dans Electrum vous aurez accès à presque toutes les fonctionnalités fournies par le dispositif en cliquant sur l'icône dans le coin inférieur d'Electrum.

(⁷) Les autres "Pay to" sont supportés avec un portefeuille tier tels que Sparrow ou Electrum. 

(⁸) Effacement du Ledger Nano S+ : la ré-initialisation efface tout sauf le firmware, il est ensuite nécessaire de ré-installer l'app Bitcoin avec Ledger Live.

(⁹) En utilisant des chemins de dérivation ou des `Pay To` différents, il est possible de créer de multiples portefeuilles avec la même graine maîtresse (master seed), donc la même liste de mots. Attention il est préférable de noter les choix effectués puisque cela complexifie la gestion et la récupération en cas de problèmes.

*Sauf pour Tangem, il est possible de restreindre l'usage des suites logicielles fournies par les fabricants à la vérification, la mise à jour du firmware et à l'installation du micro logiciel. Avec Trezor Safe 3 ou Ledger Nano S Plus vous n'avez pas besoin de créer de compte (création du portefeuille et génération des adresses) dans la suite logicielle. BitBox02 nécessite l'utilisation de BitboxApp pour restaurer ou créer le portefeuille en générant la phrase de récupération, ensuite il est possible d'utiliser le dispositif avec le portefeuille logiciel de votre choix pourvu qu'il soit compatible.*


# Annexes

## 10mn : 1 bloc

La blockchain augmente immuablement de 144 blocs par jour et de 52596 en moyenne par an, la [taille des blocs](https://www.blockchain.com/fr/explorer/charts/blocks-size) est variable et fonction des types de transactions inclues dans celui-ci, de l'activité et de l'usage du réseau. Si je prends 2 Mo par bloc cela donne 103 Go de + par an. Mon NVMe.M2 de 2 To sera approximativement blindé en 2036. Les capacités et les vitesses de transfert des stockages de masse n'aurons de cesse de s'améliorer dans le futur, il est envisageable que le code de Bitcoin s'adaptera à nouveau.

## Copie directe de la blockchain

Si vous avez un accès physique à un autre nœud de confiance vous pouvez copier sa blockchain au lieu de la télécharger des pairs. Normalement située dans `/home/bitcoin_user/.bitcoin` , vous devrez copier récursivement les répertoires `blocks` et `chainstate` (`indexes` est facultatif car le nœud les re-construira à partir de ce que vous avez paramétré dans `bitcoin.conf`). Le préalable est bien sûr de stopper `bitcoind` le temps de la sauvegarde (ou utilisez un sapshot si btrfs).

Copier 700 Go de données sur un support externe n'est pas anodin, par un port usb 3.0 j'ai eu deux expériences : une de 5h avec un débit de 38 Mo/s sur je sais plus quoi et une autre de 30 minutes avec un débit de 385 Mo/s sur un bon adaptateur usb 3 pour nvme.e M2.  

IMPORTANT : il est indispensable de noter la version de Bitcoin du nœud qui a fourni les blocs. Par exemple une sauvegarde de blockchain Bitcoin Core v28.0 ne peut pas être restauré sur un nœud Bitcoin Core v27.2, il faudra d'abord passer le nœud en V28.0 afin d'éviter une longue ré-indexation causé par le message "Corrupted block database detected.  Please restart with -reindex or -reindex-chainstate to recover".

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

* le public est connecté avec ses pairs à travers une couche d'anonymisation (Tor / I2P ) , il fonctionne 7j / 7 en continu. ipv4 n'est utilisé que sur le réseau local pour mettre à jour le nœud privé.
* le privé fonctionne sur une machine distincte reliée au nœud public par réseau local, c'est le seul pair avec qui il dialogue. En résumé, il ne fait que mettre à jour sa copie de la blockchain afin de rester opérationnel.

Afin de mener cela à bien, j'ai compilé puis installé sur mon Desktop Linux "Bitcoin Knots" avec interface graphique, j'ai ensuite restauré une copie de la blockchain avec les deux répertoires `blocks` `chainstate`. J'ai paramétré  `bitcoin.conf` pour que ce nœud soit privé. Au lancement, `bitcoin-qt` se connecte au nœud public pour mettre à jour la copie de la blockchain. N'espérez pas que le nœud public envoie les blocs vers le privé à travers le réseau local à la vitesse grand V, les nœuds sont codés pour échanger les blocs entre eux à une cadence suffisante pour diffuser l'information correctement. Si vous souhaitez télécharger rapidement la blockchain d'un nœud à l'autre par votre réseau local, stoppez les nœuds et utilisez `scp` dans un terminal.

Configuration du nœud public **par ajout de  ceci à la fin** de `bitcoin.conf` 

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

# Accepte les connexions et commandes RPC de "bitcoin-cli" entre autres
server=1    # Commentez si utilisation exclusive de la console de "bitcoin-qt"

# Remplacez X par la valeur que vous utilisez sur votre LAN
# Exemple : 192.168.0.0/24 autorise 256 adresses de (192.168.0.0 à 192.168.0.255)
rpcallowip=192.168.X.0/24       # Limite l'acces RPC bitcoind au reseau local

# Adresse IP du noeud public a laquelle se connecter et
# tenter de maintenir la connexion ouverte.
# Cette option peut être définie plusieurs fois si vous avez plusieurs noeuds publics
# Il n'y aura pas de connexions à d'autres noeuds que ceux définis ici.
connect=192.168.X.X

# Pour information la commande "addnode" ajoute la connexion au noeud de
# maniere permanente tout en continuant a recevoir des connexions d'autres noeuds
# extérieurs. Cette commande n'est pas adapté au cas d'usage ici.

# BIP158 : Index de filtres de blocs
# bitcoind va creer cet index s'il est absent, observer les logs
# Ensuite le wallet de bitcoind parcours environ 5x plus rapidement la blockchain
# a la recherche de transactions sur une adresse
# Les logs indiqueront "fast variant using block filters" a l'oppose de
# "slow variant inspecting all blocks" si l'index n'est pas construit
blockfilterindex=1
```

*Comment créer un portefeuille en lecture seule avec une adresse de mineur sur le nœud privé ?*

Bien que tout soit réalisable avec `bitcoin-cli` en ligne de commande,  ici j'utilise la console de `bitcoin-qt` pour ses fonctionnalités supplémentaires,  comme l'horodatage et la répétition des commandes.

**Créer un wallet avec "disabled private keys", nommez le "miner_wallet"**

(équivalent de `bitcoin-cli createwallet "Test/miner_wallet" true false false ""` )

Un chemin est présent dans le nom, ce portefeuille sera dans `~/.bitcoin/Test/miner_wallet`

Ensuite dans "Window / Console", saisir :

* `getdescriptorinfo "addr(3Awm3FNpmwrbvAFVThRUFqgpbVuqWisni9)"`
* copier la somme de contrôle "checksum", ici `j8r6vh28`, puis saisir :
* `importdescriptors '[{"desc": "addr(3Awm3FNpmwrbvAFVThRUFqgpbVuqWisni9)#j8r6vh28", "timestamp": "now", "watchonly": true}]'`
* attendre que la sortie indique `[{"success": true}]`
* `timestamp : now` indique que la blockchain ne sera pas parcourue à la recherche de transactions sur cette adresse au lancement de cette commande si aucun index est défini (`txindex=0` et `blockfilterindex=0`),  si un index est défini un peu moins de 20 blocs seront parcourus rapidement.
* pour voir toutes les transactions du mineur à cette adresse, effectuer ceci :
  * l'adresse a été utilisée pour la première fois le 30-06-2023 au bloc `796537`
  * nous sommes le 27 janvier 2025 à 22h00, le dernier bloc est le `881098`
  * lancer la recherche par : `rescanblockchain 796537 881098`
* observer les log du nœud, 84 561 blocs seront parcourus entre 1mn à 2mn et jusqu'à une heure en fonction du débit du support de masse qui héberge la blockchain et de la présence de l'index `blockfilter`
* par l'ajout de `walletnotify` dans `bitcoin.conf` toute nouvelle transaction exécutera un script ou une commande : `walletnotify=xmessage "Funds had been moved: %s" -center`, %s sera remplacé par l'id de la transaction.

pour supprimer le portefeuille "miner_wallet"

* décharger le portefeuille, saisir dans la console `unloadwallet miner_wallet`
* fermez le programme, puis dans `~/.bitcoin` 
  * supprimez le répertoire portant le nom `miner_wallet`
  * éditez le fichier `settings.json` et supprimez la référence à `miner_wallet` 

Pour aller plus loin vous trouverez sur le net tout ce qu'il faut pour créer un wallet avec clé privée sur votre nœud privé (lui aussi). Vous pouvez générer avec un navigateur en mode hors ligne clés privées et adresses en chargeant à partir de votre disque dur la page "BIP39 tool".


## Seed phrase 12 ou 24 mots

| **Face à** | **Phrase de 12 mots** | **Phrase de 24 mots** |
|----|----|----|
| Sécurité quantique | Suffisamment sécurisée pour le présent et l'avenir proche (2025 + 10 ans environ)  | Sécurisée pour l'avenir lointain |
|    |    |    |
| Praticité | Moins de mots c'est moins d'erreurs lors d'une restauration, moins d'efforts pour graver sur support métallique. | 2 fois plus longue à graver et à restaurer. |
| Confidentialité | Discrète à stocker ou partager en cas de besoin, exemple gravure sur métal compacte. | Difficile à brute-forcer si partiellement compromise, 24 mots protègent mieux contre les fuites partielles comme une courte exposition visuelle. |
| Échappatoire | Mémorisable par un individu | Difficilement mémorisable par un individu |
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

Si le périphérique de masse qui héberge les données blockchain est formaté BTRFS un snapshot peut s'avérer utile avant une mise à jour de `bitcoind`. Le snapshot permet de sauvegarder l'état de la blockchain avant chaque mise à jour, d'effectuer un roolback si besoin est, mais également d'être en mesure de le copier sur un autre support de stockage la blockchain entière sans arrêter le daemon `bitcoind` .

 Voici quelques exemples de commandes :

```bash
# Afficher l'espace libre / utilise
sudo btrfs filesystem usage /mnt/btrfs/bitcoin

# Lister les subvolumes et les snapshots
sudo btrfs subvolume list -t /mnt/btrfs/bitcoin

# Creer un READ-ONLY snapshot de la blockcain bitcoin (subvolume snapshots deja existant)
sudo systemctl stop bitcoin.service   # Stoppez le demon bitoind, attendre l'invite.
sudo btrfs subvolume snapshot -r /mnt/btrfs/bitcoin /mnt/btrfs/snapshots/bitcoin_2025-12-14_RO
sudo systemctl start bitcoin.service   # Demarrer le demon bitoind

# Voir le contenu du snapshot
ls -lh /mnt/btrfs/snapshots/bitcoin_2025-12-14_RO

# Copier un snapshot, c'est copier les données de l'image figee d'un sous-volume
# a une date donnee sur un autre support de masse au format BRTFS
# par exemple une clef USB montee dans le systeme de fichier, ici en /mnt/usb
# Pas besoin de stopper bitcoind vu que c'est un snapshot
sudo btrfs send /mnt/btrfs/snapshots/nom-du_snapshot_date | btrfs receive /mnt/usb

# Afficher l'espace libre / utilise sur la clef usb
sudo btrfs filesystem usage /mnt/usb

# Effacer un snapshot
sudo btrfs subvolume delete /mnt/btrfs/snapshots/nom-du-snapshot_date
```

A partir d'un snapshot **read-only** revenir à un état précédent (rollback)

```bash
# /!\ ATTENTION  /!\ c'est destructif pour les changements post-snapshot
sudo systemctl stop electrs.service            # Stoppez electrs
sudo systemctl stop bitcoin.service            # Stoppez le demon bitoind
sudo btrfs subvolume list /mnt/btrfs/bitcoin   # Identifier le snapshot

# Demonter TOUS les sous volumes
sudo umount /mnt/btrfs/bitcoin
sudo umount /mnt/btrfs/electrs
sudo umount /mnt/btrfs/snapshots

# Montez temporairement le volume RACINE btrfs sans l'option subvol 
sudo mkdir /mnt/btrfs_root                           # si pas deja cree
sudo mount /dev/nvme0n1p1 /mnt/btrfs_root            # Pour moi c'est nvme0n1p1
sudo btrfs subvolume delete /mnt/btrfs_root/@bitcoin # Efface ... puis remplace ci dessous
sudo btrfs subvolume snapshot /mnt/btrfs_root/@snapshots/bitcoin_2025-12-14_RO /mnt/btrfs_root/@bitcoin
sudo umount /mnt/btrfs_root    # Demonter        
sudo mount -a                  # Remontez les volumes listes fstab
```

## maj de bitcoind

Ouvrir un terminal afin d'accéder au nœud puis afficher la version actuelle avec la commande `bitcoin-cli -version` . Ensuite consulter la page des [releases sur le repository bitcoin](https://github.com/bitcoin/bitcoin/releases), il est utile de lire les notes de la nouvelle version vers laquelle vous mettez à jour (et celles des versions que vous avez sautées) s'il y a des fonctionnalités / options qui vous intéressent ou que vous allez devoir gérer. Il est ici bon de préciser que l'on va toujours de l'avant, jamais en arrière. Ainsi si vous avez construit votre nœud avec Bitcoin Core v28.0 n'installez pas une v27.x car cela va mal se passer avec les données de blockchain.

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

# Mettre à jour le code source
git pull

# Afficher les versions
git tag
```

**Ici pour l'exemple, je choisi la "v29.0"**

Facultatif : vérifier que la "release" que vous avez sélectionnée est signée :

```bash
# A faire une fois : ajouter les clés PGP approuvées à votre machine, voir avec 
# https://github.com/bitcoin/bitcoin/blob/master/contrib/verify-commits/README.md
gpg --keyserver hkps://keys.openpgp.org --recv-keys $(<contrib/verify-commits/trusted-keys)
git verify-tag "v29.0"
# Vous devez obtenir en sortie : Good signature from ...
```

Poursuivre avec

```bash
git checkout "v29.0"
make clean
./autogen.sh
./configure --disable-tests --disable-fuzz-binary --disable-bench
# Vérifier que la sortie du .configure soit dans vos attentes

# C'est ok ? alors construisez et attendez que cela se passe ...
make

# OU
# bien si vous avez autre chose en vue, lancez cette commande en tâche de fond
make > build-bitcoind.log &

# Vérifiez quand même que cela se construit en visualisant la sortie capturée
tail -f build-bitcoind.log
# Ok, je ferme le terminal, j'arrête mon PC desktop et je me barre
# je rependrais plus tard par cette commande pour voir si tout est correct
cat ~/code/bitcoin/build-bitcoind.log
```

Le nouveau binaire est construit, on continue

```bash
# Afin de vérifier l'arrêt complet de bitcoind qui est parfois long
# ouvrir un terminal pour observer les logs
tail -f ~/.bitcoin/debug.log

# Dans un autre terminal stopper electrs si lancé et bitcoind
sudo systemctl stop electrs.service
sudo systemctl stop bitcoin.service

# Pour bitcoind vous avez vu "Shutdown: done" dans les logs ? Poursuivez
# sinon attendez parfois qq min

# Ne pas lancer la nouvelle version de bitcoind en mode démon, éditer bitcoin.conf
nano ~/.bitcoin/bitcoin.conf
# Commenter #daemon=1 dans le fichier de conf, sauvegarder.

# Se positionner dans le répertoire du code source de bitcoin
cd ~/code/bitcoin

# Installer la nouvelle version
sudo make install

# Lancer bitcoind, 
bitcoind

# Regarder ce qui se passe en visualisant la sortie ... ok ou pas ?
# Arreter bitcoind en tapant CTRL-C 

# Ce n'est pas bon ...
# si besoin de re-démarrer la machine il faut arrêter les services 
sudo systemctl disable electrs.service
sudo systemctl disable bitcoin.service
# investiguez le problème ...

# Tout est correct, retour du démon, dé-commenter daemon=1 dans le fichier de conf
nano ~/.bitcoin/bitcoin.conf

# Observer les logs dans un terminal
# Dans un autre terminal, relancer bitcoin avec le service 
sudo systemctl start bitcoin.service

# bitoind est synchronisé avec ses pairs ? Oui alors relancer electrs
sudo systemctl start electrs.service
```

## **maj d' Electrs**

Ouvrir un terminal afin d'accéder à votre nœud puis afficher la version actuelle avec la commande `electrs --version` ensuite consulter la [page de publication d'Electrs](https://github.com/romanz/electrs/releases) pour voir si une version plus récente est disponible.

Une nouvelle version est disponible et vous voulez effectuer la mise à jour, suivez ceci :

```bash
# Se positionner dans le répertoire du code source d'Electrs
cd ~/code/electrs

# Nettoyer le code source local
git clean -xfd

# Mettre à jour
git fetch

# Afficher la dernière étiquette de version
git tag | sort --version-sort | tail -n 1
# La dernière version pour moi est "v0.10.8"

# Facultatif : Vérifier que la realease que vous allez compiler est originale
git verify-tag "v0.10.8"
# Vous devez obtenir en sortie : Good signature from "Roman Zeyde <me@romanzey.de>"

# Choix de la realease
git checkout "v0.10.8"

# Nettoyage
cargo clean

# Construction
cargo build --locked --release
```

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

Rien de plus simple, si la fonctionnalité de mise à jour est disponible dans l'application elle même, activez là et en principe plus besoin de vérifier l'authenticité.  Sinon téléchargez la nouvelle version, vérifiez son authenticité, renommez l'ancienne en `.old` puis installez comme pour la première fois, pour finir mettez à jour le nom dans le lanceur si vous en avez crée un la première fois. Les paramètres utilisateur sont en principe à l'abri puisque séparés dans des répertoires `~/.nom_du_programme` ou dans `~/.config/nom_du_programme`

# Glossaire

## 1Mo

Le Méga Octet (ou Mio le [mébioctet pour les puristes](https://fr.wikipedia.org/wiki/Octet) et dans tout ce document Mo=Mio) permet de quantifier la taille que des données occupent sur support de stockage donné. Le bit, qui est contraction de **binary digit,** est la plus petite unité de données dans un ordinateur, qui peut avoir une valeur de 1 ou 0, allumé ou éteint, on / off. Ensuite viens l'octet (`oct` pour 8, `et` pour petit) qui rassemble 8 bits. A partir de là, le passage d'une grandeur à l'autre est toujours x 1024 :

* 1 Ko (Kio) (Kilo Octet) : 1024 octets / 2¹⁰ octets ou de 00 0000 0000 à 11 1111 1111
* 1 Mo (Mio) (**Méga Octet**): 1024 Ko / 2²⁰ octets
* 1 Go (Gio) (Giga Octet): 1024 Mo / 2³⁰ octets
* 1 To (Tio) (Tera Octet) : 1024 Go / 2⁴⁰ octets

La question qui tue, pourquoi x 1024 et pas x 1000 ?

* Les ordinateurs fonctionnent sur un système binaire, qui utilise des puissances de 2.
* 1024 est une puissance de 2, ce qui en fait une valeur naturelle dans le contexte binaire.
* choisir 1000 aurait cassé la facilité de compréhension et de manipulation de ces ^2.

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

Un CPU est capable de manipuler des données de 64 bits de large, si tous les bits sont "On" donc à 1 et que je découpe en paquets de 4 bits, cela donne `1111 1111 1111 1111 1111 1111 1111 1111` et en hexa ? c'est juste trop facile puisque 1111 = F donc cela donne FFFF FFFF .

L'index de transaction txid de Bitcoin est codé en hexa, exemple :

`277944f4de6401f06bd38a39e2b9bae6240e6d8e1a0da1f3206bc3b8bbdc797e`

## Base58

Est un système de représentation efficace et surtout compact de données binaires (base 2 qui utilise 0 et 1)  en une chaîne de caractères alphanumériques. Avec base 58 il est nécessaire de définir 58 caractères différents, par convention est employé les chiffres de base 10 + l'alphabet majuscule + l'alphabet minuscule - les caractères ambigus `0 O l I`soit : `123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefjhijkmnopqrstuvwxyz`

Base58 est utilisée pour les adresses Bitcoin et les clés privées.

La variante Base58Check inclut une somme de contrôle pour vérifier l'intégrité des données afin de prévenir les erreurs de saisie.

Le cerveau humain n'est pas habitué à manipuler des données en base 58, une machine programmée par un cerveau humain elle en est capable.

Exemple d'adresse : `1NEU779yvLaFk39k4Q3QdLjwpWTdWCbzqL`

## Bech32

Afin d'améliorer les points faibles de base58, qui [sont](https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki) le jeu de caractères, l'algorithme de somme de contrôle et la production de QR codes, un autre système d'encodage des adresses a été développé. Il utilise 32 caractères, les chiffres base 10 +  l'alphabet minuscule  - les caractères ambigus `0 i l o` soit :

`123456789abcdefghjkmnpqrstuvwxyz` 

Bech32 est apparu et est utilisé pour les adresses Bitcoin segwit native.

Exemple d'adresse : `bc1qc7slrfxkknqcq2jevvvkdgvrt8080852dfjewde450xdlk4ugp7szw5tk9`

## Bech32m

Utilise les mêmes caractères que bech32 pour sa représentation mais sous une organisation différente qui [offre une meilleure sécurité que le schéma original de bech32](https://github.com/bitcoin/bips/blob/master/bip-0350.mediawiki).

Bech32m est utilisé pour les adresses Bitcoin Taproot.

Exemple d'adresse : `bc1p0xlxvlhemja6c4dqv22uapctqupfhlxm9h8z3k2e72q4k9hcz7vqzk5jj0`

## BIP

Une BIP (Bitcoin Improvement Proposal) est une proposition d'amélioration de Bitcoin, cela se traduit par un document de conception fournissant des informations à la communauté ou décrivant une nouvelle fonctionnalité, ses processus ou son environnement. Liste des BIP [ici](https://github.com/bitcoin/bips) ou [là](https://bips.dev).

## BIP32 Root fingerprint

Ce terme est parfois rapporté par les portefeuilles, explications :

* BIP32 est la référence au protocole qui permet la création de portefeuilles déterministes hiérarchiques (les portefeuilles HD générant une seed phrase).
* Root fingerprint est une partie de l'empreinte de la clé publique maîtresse.
* BIP32 Root fingerprint agit comme un identifiant partiel qui est suffisant pour distinguer les portefeuilles dans de nombreux contextes, sans pour autant révéler la clé publique complète ce qui permet de voir l'intégralité de l'historique des transactions.

## Blockbook

Est un service backend utilisé principalement par Trezor Suite pour fournir des informations sur les transactions et les adresses des blockchains. Le [serveur Blockbook](https://github.com/trezor/blockbook) est Open Source et est principalement développé et maintenu par l'équipe de Trezor (SatoshiLabs), qui l'utilise pour leur suite logicielle. La plate forme officiellement supportée est Debian Linux, la machine doit être équipée de 32Go de Ram et d'un SSD > 200Go. Elle supporte les actifs cryptographiques Bitcoin, Bitcoin Cash, Zcash, Dash, Litecoin, Bitcoin Gold, Ethereum, Ethereum Classic, Dogecoin, Namecoin, Vertcoin, DigiByte, Liquid, ainsi que d'autres qui ont été mis en place par la communauté. Supporte les testnets également.

## Chemin de dérivation

Les portefeuilles matériels ou logiciels modernes permettent de générer des clés et des adresses de manière hiérarchique et déterministe. Ils facilitent la gestion et l'organisation des adresses Bitcoin pour différents usages ou comptes. A partir d'une simple phrase de récupération ils sont capables de gérer plusieurs portefeuilles avec des **chemins de dérivation** différents. La structure typique d'un chemin de dérivation est la suivante : `m/purpose/coin_type/account/change/address_index`

* **m** → Master seed, c'est la clé maître, le point de départ de toutes les dérivations.
* **purpose** → c'est le but, et informe sur la norme du chemin de dérivation. Venu avec la [BIP43](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki)
  * `0` ou `44` en référence aux adresses Legacy `1`  `P2PKH`  [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)
  * `45` en référence aux portefeuilles multisigs anciens `3`  `P2SH`  [BIP45](https://github.com/bitcoin/bips/blob/master/bip-0045.mediawiki)
  * `47` en référence aux codes de paiement réutilisables [BIP47](https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki)
  * `48` en référence aux portefeuilles multisigs matériels [BIP48](https://github.com/bitcoin/bips/blob/master/bip-0048.mediawiki)
  * `49` en référence aux adresses nested-Segwit `3`  `P2SH-P2WPKH`  [BIP49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki)
  * `84` en référence aux adresses native SegWit `bc1q`  `P2WPKH`   [BIP84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki)
  * `86` en référence aux adresses Taproot `bc1p`  `P2TR`  [BIP86](https://github.com/bitcoin/bips/blob/master/bip-0086.mediawiki)
  * l'utilisation de l'apostrophe `'` ou du `h` après le paramètre signifie "hardened derivation" ou dérivation renforcée, comme `44h` ou `44'`. Un algorithme supplémentaire est mis en oeuvre, il devient presque impossible d'accéder à la clé maître même si une clé privée dérivée est compromise. Cela constitue une une couche supplémentaire de sécurité.
* **coin type** → renseigne sur l'actif
  * `0` est bitcoin (`'`  ou `h` sont utilisables)
  * pour reste c'est [ici](https://github.com/satoshilabs/slips/blob/master/slip-0044.md).
* **account** → indique l'identité ou la collection d'adresses, la première est notée `0`. Dans un portefeuille multi-comptes cela permet aux utilisateurs de séparer les fonds pour différentes choses comme hold, dépense, dons, etc… (`'`  ou `h` sont utilisables)
* **change** → renseigne sur la transaction
  * `0` indique les adresses de `chaîne externe`, régulières.
  * `1` indique les adresses de `chaîne interne`, c'est le change, le rendu monnaie.
  * pour plus de détail, voir [BIP47](https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki)
* **address index** → index séquentiel pour générer des adresses spécifiques à l'intérieur d'un compte, la première valeur est notée `0` ou adresse de rang 1.

Par exemple `m/44h/0h/0h/1/1` indique :

* `44h` adresse Legacy renforcée
* `0h` le type d'actif est renforcé, ici bitcoin.
* `0h` le compte est renforcé, ici le premier, le suivant sera noté `1'`.
* `1` une adresse de rendu monnaie
* `1` une adresse de rang 2, le rang 1 (la première adresse) étant notée `0` .

## CPU

Central Processor Unit (CPU), aussi appelé microprocesseur ou processeur central, est la puce responsable de l'exécution du code binaire produit par un compilateur à partir du code source d'un programme informatique. Le CPU est un circuit intégré généraliste à logique non câblée, ce qui signifie qu'il n'est pas spécialisé dans un domaine particulier et qu'il peut effectuer une grande variété de tâches. Cette polyvalence lui permet de "savoir tout faire", mais il n'est pas optimisé pour des opérations spécifiques.

À l'opposé, les puces spécialisées à logique câblée, telles que les ASIC (Application Specific Integrated Circuit), sont conçues pour exécuter très rapidement un nombre restreint de tâches spécifiques. Par exemple, les ASIC destinés au minage de Bitcoin sont spécialement conçus pour résoudre l'algorithme de hachage sécurisé SHA-256 (Secure Hash Algorithm-256), qui est l'algorithme utilisé dans le mécanisme de preuve de travail (Proof of Work) de Bitcoin. Ces ASIC surpassent largement les CPU et même les GPU en termes d'efficacité et de vitesse pour le minage de Bitcoin, ce qui les rend indispensables dans les opérations de minage à grande échelle.

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

## PSBT

Partially Signed Bitcoin Transactions. Une transaction Bitcoin partiellement signée est un format de données qui permet aux portefeuilles et à d'autres outils d'échanger des informations sur une transaction Bitcoin et les signatures nécessaires pour la finaliser. Une PSBT peut être créée en identifiant un ensemble d'UTXO à dépenser et un ensemble de sorties pour recevoir la valeur dépensée. Des informations sur chaque UTXO nécessaires pour générer une signature peuvent ensuite être ajoutées, éventuellement par un outil distinct, comme le script de l'UTXO ou sa valeur précise en bitcoin. La PSBT peut ensuite être copiée par n'importe quel moyen vers un programme capable de la signer. Pour les portefeuilles à plusieurs signatures ou dans les cas où différents portefeuilles contrôlent différentes entrées, cette dernière étape peut être répétée plusieurs fois par différents programmes sur différentes copies de la PSBT. Plusieurs PSBT comportant chacune une ou plusieurs signatures nécessaires peuvent être intégrées ultérieurement dans une PSBT unique. Enfin, cette PSBT entièrement signée peut être convertie en une transaction complète prête à être diffusée.

## Secure element et norme EAL

La norme EAL est utilisée pour quantifier la résistance à diverses formes d'attaques (physiques ou  logicielles) des puces appelées "secure element" utilisées entre autre dans les "non-custodial hardwares wallets" (portefeuilles matériels non dépositaires). Les clés privées sont à l'intérieur de ces puces sécurisées. Les niveaux EAL (Evaluation Assurance Level) vont de 1 à 7, chaque niveau offrant une évaluation plus rigoureuse et des exigences de sécurité plus élevées.

## SLIP39

La norme Shamir's Secret Sharing Scheme a été proposé et adopté par Trezor en 2019. Cette norme est supporté par un nombre bien plus réduit de matériels et de logiciels que BIP39.

La phrase de récupération de la graine est de 20 ou de 33 mots et est divisible de 1 à 16 fragments (ou shards), ensuite un seuil de récupération (threshold) est fixé comme 3/3 ou 2/3 ou 3/5 … mais jamais 1 si plus d'un fragment. Exemple si 20 mots en 2/3 il y aura 3 listes séparées de 20 mots chacune (60 mots au total), la restauration fonctionnera avec 2 listes de 20 mots.

Liste des portefeuilles matériels SLIP39 compatibles en 2024 :

* Trezor (Model T à partir de la version 2.7.2, Safe 3, Safe 5)
* Keystone Pro

Shamir Backup est de la sauvegarde distribuée de phrase de récupération. Multisig propose cela et procure en plus une signature distribuée, par contre c'est beaucoup plus compliqué à mettre en oeuvre.

## SPV

Simplified Payment Verification ou vérification simplifiée des paiements. Cette technique permet de vérifier des paiements sans directement utiliser un nœud complet (Full node) avec l'intégralité de la blockchain. Le client SPV n'a besoin que des en-têtes des blocs, ce qui est bien plus petit que les blocs complets. Pour vérifier qu'une transaction figure dans un bloc, le client SPV demande une preuve d'inclusion, sous la forme d'une branche de Merkle. Notez que cette technique est décrite dans le papier de Satoshi Nakamoto au paragraphe 8. 

## UTXO

Unspent Transaction Output ou sortie de transaction non dépensé. Une transaction consomme un ou plusieurs UTXO existants comme entrée et crée de nouveaux UTXO comme sortie, le total des fonds entrants est toujours égal au total des fonds sortants y compris les frais de transaction.  Ce concept fondamental, *qui de fait est un déplacement des droits de propriété*, a été implémenté à la création de Bitcoin. Exemple : vous possédez 1 BTC sur un seul UTXO et vous devez payer 0.5 BTC à un tiers. Votre transaction utilisera votre UTXO de 1 BTC comme entrée et créera trois nouvelles sorties : 0.5 BTC vers le tiers, 0.00001 BTC de frais de transaction, 0.49999 BTC de retour vers vous sur une de vos adresses de change. Vous avez maintenant un nouvel UTXO de 0.49999 BTC. Les entrées consomment un UTXO existant tandis que les sorties créent un nouvel UTXO, en résumé seuls les produits non dépensés peuvent être utilisés dans de nouvelles transactions, cela évite la double dépense et la fraude. En règle générale les portefeuilles gèrent les UTXO de manière transparente pour les utilisateurs selon différentes stratégies en fonction des choix implémentés par les développeurs du portefeuille.

Un "full node" possède l'historique de la totalité des UTXO dans son stockage local. A un moment donné l'ensemble des UTXO peut être additionné pour calculer l'offre totale en circulation,  la commande `bitcoin-cli gettxoutsetinfo` donne  19 817 079 BTC au 30-01-2025.

# Liens

* Documentation sur le github de [Bitcoin Core](https://github.com/bitcoin/bitcoin/tree/master/doc) et de [Bitcoin Knots](https://github.com/bitcoinknots/bitcoin/tree/29.x-knots/doc) - parcourez suivant ce que vous recherchez.


* [le nombre de noeuds by Luke Dashjr](https://luke.dashjr.org/programs/bitcoin/files/charts/historical.html) - historique graphique depuis 2017


* [jon atack](https://jonatack.github.io/articles) - documentation et ressources autour de Bitcoin
* [Bitcoin Forum](https://bitcointalk.org/)


* [BIP 39 tool](https://github.com/iancoleman/bip39) - outil pour convertir les phrases mnémoniques BIP39 en adresses et clés privées


* [Ur₿an.T丰ch21](https://x.com/urbantech21) - le narrateur et explorateur des archives sur Bitcoin


* [BTC TouchPoint](https://btctouchpoint.com) - le parcours en vidéos et podcasts de la chute dans le Bitcoin Rabbit Hole
* [DSN Bitcoin monitoring](https://www.dsn.kastel.kit.edu/bitcoin/) - les vidéos de propagation des blocs à diverses époques !!!
* [BitcoinStrings](https://bitcoinstrings.com/)  /  [Bitaddress.org](https://www.bitaddress.org/)

# Littérature

[Livre "Mastering Bitcoin" traduction française incomplète qui a le mérite d'exister](https://bitcoin.fr/wp-content/uploads/2020/08/Mastering-Bitcoin.pdf)

[Le livre blanc d'Ethereum en français contient des informations intéressantes … sur Bitcoin](https://ethereum.org/fr/whitepaper/)

[Le dictionnaire de Bitcoin par Loic … LA mine d'informations](https://github.com/LoicPandul/Dictionnaire-de-Bitcoin)


# Conclusion

## of Satoshi Nakamoto's paper

We have proposed a system for electronic transactions without relying on trust. We started with the usual framework of coins made from digital signatures, which provides strong control of ownership, but is incomplete without a way to prevent double-spending. To solve this, we proposed a peer-to-peer network using proof-of-work to record a public history of transactions that quickly becomes computationally impractical for an attacker to change if honest nodes control a majority of CPU power. The network is robust in its unstructured simplicity. Nodes work all at once with little coordination. They do not need to be identified, since messages are not routed to any particular place and only need to be delivered on a best effort basis. Nodes can leave and rejoin the network at will, accepting the proof-of-work chain as proof of what happened while they were gone. They vote with their CPU power, expressing their acceptance of valid blocks by working on extending them and rejecting invalid blocks by refusing to work on them. Any needed rules and incentives can be enforced with this consensus mechanism.


## du papier de Satoshi Nakamoto

Nous avons proposé un système de transactions électroniques qui ne repose pas sur la confiance. Nous avons commencé par le cadre habituel des pièces de monnaie fabriquées à partir de signatures numériques, qui permet un contrôle solide de la propriété, mais qui est incomplet s'il n'y a pas de moyen d'empêcher la double dépense. Pour résoudre cela, nous avons proposé un réseau pair à pair utilisant la preuve de travail pour enregistrer un historique public des transactions qui devient rapidement impossible à modifier pour un attaquant si les nœuds honnêtes contrôlent la majorité de la puissance CPU (¹). Le réseau est robuste dans sa simplicité non structurée. Les nœuds travaillent tous en même temps avec peu de coordination. Ils n'ont pas besoin d'être identifiés, puisque les messages ne sont pas acheminés vers un endroit particulier et qu'ils ne doivent être délivrés que dans la mesure du possible. Les nœuds peuvent quitter et rejoindre le réseau à volonté, en acceptant la chaîne de preuve de travail comme preuve de ce qui s'est passé pendant leur absence. Ils votent avec la puissance de leur CPU (¹), exprimant leur acceptation des blocs valides en travaillant à leur extension et au rejet des blocs non valides en refusant de travailler dessus. Toutes les règles et récompenses nécessaires peuvent être imposées avec ce mécanisme de consensus.


(¹) Nous sommes fin 2008, il est fait référence à un équipement tout en un où le nœud utilise la puissance de traitement de son CPU (²) pour la création des nouveaux blocs avec la preuve de travail. Depuis 2013 \~ 2014 cette preuve de travail (Proof of Work ou POW), *qui est la base du consensus*,  est devenu une activité industrielle nécessitant des investissements colossaux et donc hors de portée de l'individu. Le hashrate, qui quantifie la puissance de traitement SHA256 disponible sur le réseau Bitcoin, s'est externalisé et professionnalisé.  Autrement dit, pour avoir une chance raisonnable de propager de nouveaux blocs avec un nœud Bitcoin, il est nécessaire de déléguer (³) la puissance de calcul à des milliers de puces spécialisées appelées ASIC qui sont groupées dans ce que l'on nomme des pools de minage.

(²)  Le minage CPU via `bitcoind` n'est plus supporté depuis `bitcoin core 0.13.0` sorti en 2016.

(³) Pour des raisons de scalabilité cette délégation s'effectue majoritairement par Stratum, un protocole de communication qui sert d'intermédiaire entre les pools d'ASIC spécialisés SHA256 et le nœud Bitcoin, il permet d'agréger la puissance de calcul du minage en pool avec optimisation de la charge et de la latence réseau. Mais pas que puisque c'est au niveau de Stratum que le bloc est crée. En simplifiant : le mineur ne fait qu'apporter sa puissance de calcul à Stratum qui va coordonner les pools de mineurs. Si un bloc est trouvé, Stratum le transmet au nœud qui va le diffuser ensuite à tous ses pairs.

Au chapitre 4 de son papier, Satoshi Nakamoto avait intelligemment prévu et anticipé que la puissance de calcul du matériel augmenterait dans le temps et dès le départ il a conçu Bitcoin avec une difficulté ajustable pour s'adapter à cette augmentation de puissance de calcul et à l'intérêt variable des nœuds qui minent les nouveaux blocs.

## Q'engendre la professionnalisation de la puissance de calcul dans la gouvernance de Bitcoin ?

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

* aujourd'hui cela me réponds 7.465207827981607e+20 (746 EH/s)
* au point bas de 2009, le 24 août c'était 7.208467e+5 (720 KH/s soit le hash de 10 NerdMiner)
* la différence est approximativement de e+15 soit 1 000 000 000 000 000 de fois plus

## Ma conclusion

Faites donc tourner un nœud, le vôtre, soyez indépendant, existez en tant qu'individu !