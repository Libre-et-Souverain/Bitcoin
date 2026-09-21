# Bitcoin et le risque quantique

Ce texte traite du risque de vol individuel : peut-on vous prendre vos bitcoins ? Il ne traite pas du risque systémique (que vaut un bitcoin le jour où quelqu'un vide les coins dormants), question qui relève du débat « geler ou laisser voler » ouvert par BIP-361.

## Le minage n'est pas cassé

SHA-256 n'est pas cassé par une machine quantique. Grover offre un gain quadratique théorique sur la recherche de nonce, mais les portes quantiques sont des millions de fois plus lentes qu'un ASIC et la correction d'erreurs absorbe le gain : aucun avantage pratique prévisible. Production des blocs, consensus et fonctionnement du réseau tiennent. Le seul risque résiduel, théorique, est une centralisation du minage si un acteur obtenait un avantage quantique — pas une rupture du consensus.

## La propriété des UTXO : une question de clés, pas d'adresses

La menace est l'algorithme de Shor, qui remonte de la clé publique secp256k1 à la clé privée. Une seule règle : **une clé publique est compromise dès qu'elle apparaît en clair, quelle que soit l'adresse qui l'a révélée.**

Exposés par conception, la clé étant dans la sortie elle-même :

* **P2PK** — pas de format d'adresse : ce sont les coinbases de 2009-2010, \~1,7 M BTC sur \~20 000 clés.
* **P2TR** (`bc1p`) — la clé de sortie est dans le scriptPubKey ; Shor en déduit la clé privée.

Non exposés tant qu'ils n'ont jamais dépensé : **P2PKH** (`1`), **P2SH** (`3`), **P2WPKH** et **P2WSH** (`bc1q`). Le paiement va à une empreinte HASH160 (160 bits) dont la préimage reste hors de portée en pratique, même avec Grover (\~2^80 opérations quantiques).

Le stock exposé ne se résume pas à P2PK + Taproot. Les estimations 2026 convergent vers 6 à 7 M BTC (30-34 % de l'offre), et l'essentiel vient des adresses **réutilisées après une dépense**. L'écart entre les chiffres tient au périmètre (Taproot, rendu de monnaie, exchanges inclus ou non), pas à un désaccord factuel.

Trois cas que l'on oublie :

* **Multisig P2SH/P2WSH** : la dépense révèle le script complet, donc les clés de *tous* les cosignataires, y compris ceux qui n'ont pas signé. Si ces clés (mêmes xpubs, mêmes hardware wallets) servent ailleurs, les autres wallets sont exposés par ricochet.
* **Fuites hors chaîne** : un **xpub** partagé (watch-only, explorateur, logiciel fiscal, coordinateur coinjoin) expose toutes les clés enfants non durcies, quel que soit le type d'adresse et sans jamais dépenser. Idem, à moindre échelle, pour les canaux Lightning, la signature de message (BIP-137/322) et les PSBT partagés en multisig.
* **Custodians** : not your keys, not your threat model. Vous héritez de l'hygiène de clés de l'exchange, que vous ne pouvez ni auditer ni corriger. Glassnode (mai 2026) : 1,66 M BTC exposés sur exchanges — Bitfinex 100 %, Binance 85 %, Coinbase 5 %.

Taproot, donc : pour un stockage long terme, un P2WPKH vierge vaut mieux qu'un P2TR. Pour des dépenses courantes, la différence est marginale puisque toute dépense expose la clé. Le compromis de Taproot était connu et assumé ; le correctif est un soft fork post-quantique, pas un retour à `bc1q`.

La seed n'est pas concernée par Shor (BIP-39/BIP-32 : HMAC-SHA512, pas de courbe elliptique). 12 mots (128 bits, \~2^64 sous Grover) restent hors de portée ; 24 mots ne se discutent pas.

## Le mempool

Toute dépense révèle la clé publique de l'entrée, pour tous les types d'adresse sans exception. Mais le mempool n'apprend rien à un attaquant patient : cette clé finit dans un bloc, définitivement, et l'UTXO qu'elle contrôlait est consommé. Écouter le mempool pendant des années ne sert à rien — la blockchain stocke déjà pour lui.

Le mempool compte pour un attaquant **rapide**. Sur une architecture à cycle rapide (supraconducteur, photonique), Google Quantum AI estime (2026) le cassage d'une clé secp256k1 à quelques minutes avec < 1 200 qubits logiques et < 500 000 qubits physiques. L'attaquant dérive la clé privée, signe une transaction concurrente vers lui-même avec des frais supérieurs et la diffuse : avec full-RBF (Bitcoin Core ≥ 28), il n'a besoin d'aucun hashpower. La fenêtre court de la diffusion jusqu'à N confirmations — pas jusqu'à une « inclusion définitive » qui n'existe pas — et une transaction à frais faibles peut traîner des heures. Seule mitigation individuelle : des frais suffisants. Les architectures à cycle lent (ions piégés : \~26 jours par clé) ne permettent que l'attaque à longue exposition.

Un vecteur propre au mempool subsiste : la transaction diffusée puis jamais minée (évincée, expirée, en conflit). La clé a fuité, et l'UTXO est toujours dépensable.

Enfin, la migration vers un schéma post-quantique sera le moment de plus grande exposition : chacun devra dépenser ses UTXO ECDSA/Schnorr, donc passer par le mempool avec sa clé en clair, précisément quand la menace est crédible. C'est pour cela que la fenêtre mempool, marginale aujourd'hui, est la question centrale de la conception du soft fork (schémas commit-reveal).

## La réutilisation d'adresse

Pour un type d'adresse qui ne divulgue pas la clé publique, recevoir plusieurs fois sur la même adresse n'est pas une faille quantique. Elle le devient dès qu'il y a eu une dépense depuis cette adresse : la clé est en clair on-chain et tout ce qui y revient est exposé. Le rendu de monnaie n'est pas concerné s'il va sur une adresse neuve à chaque fois, comportement par défaut de tout wallet moderne. Ce n'est pas l'adresse qui compte, c'est la clé qu'elle a révélée.

## Que faire

Mesure conservatoire, dès maintenant, au coût d'une transaction :


1. Déplacer vers des P2WPKH vierges tout ce qui est exposé : Taproot, adresses réutilisées après dépense, P2PK, tout wallet dont un xpub a circulé.
2. Ne plus jamais réutiliser une adresse après dépense.
3. Ne jamais partager un xpub.
4. Dépenser avec des frais suffisants.

Ce que cela ne couvre pas : la course mempool le jour où vous migrerez vers un schéma post-quantique. Cette mesure retire vos clés du stock déjà exposé ; ce n'est pas une protection définitive.

Quand ? Les chiffres ci-dessus sont des estimations de ressources, pas des démonstrations : aucune machine n'en approche. Mais l'estimation a été divisée par \~20 en cinq ans, et la mesure coûte une transaction. La date ne change pas la décision.


---

**Sources** : Google Quantum AI, *Securing Elliptic Curve Cryptocurrencies against Quantum Vulnerabilities*, PRX Quantum 7, 031001 (2026) ; Glassnode Research, *Quantum exposure of the Bitcoin supply*, mai 2026 ; BIP-360 (P2MR) et BIP-361, bips.dev.


# Glossaire — Bitcoin et le risque quantique

Annexe au texte « Bitcoin et le risque quantique ». Chaque entrée donne la définition, puis ce qu'elle implique pour le sujet quantique.

## Cryptographie et algorithmes quantiques

### SHA-256

Fonction de hachage cryptographique produisant une empreinte de 256 bits, conçue par la NSA et publiée par le NIST en 2001. Bitcoin l'utilise partout : preuve de travail (double SHA-256 de l'en-tête de bloc), identifiants de transaction, arbre de Merkle, et comme première étape de HASH160 = RIPEMD160(SHA256(clé publique)).

**Implication quantique** : non cassée. Une fonction de hachage n'a pas de structure mathématique exploitable par Shor. Seul Grover s'applique, avec un gain insuffisant pour menacer quoi que ce soit. C'est pourquoi le minage et les adresses non dépensées tiennent.

### Grover (algorithme de, 1996)

Algorithme quantique de recherche dans un espace non structuré. Là où un ordinateur classique doit essayer en moyenne N/2 possibilités, Grover en essaie environ √N.

**Implication quantique** : c'est la seule attaque quantique connue contre les fonctions de hachage. Elle ramène la sécurité de SHA-256 de 2^256 à 2^128, et celle d'une empreinte HASH160 de 2^160 à 2^80 — deux chiffres qui restent hors de portée. Grover se parallélise mal : répartir la recherche sur M machines ne divise le temps que par √M, ce qui annule l'intérêt des fermes de calcul.

### Gain quadratique

Passage d'un coût N à un coût √N. C'est la nature du gain offert par Grover, par opposition au gain **exponentiel** de Shor.

**Implication quantique** : c'est toute la différence entre « affaibli » et « cassé ». Un gain quadratique sur 256 bits laisse 128 bits de sécurité, soit une marge confortable : on compense en doublant la taille. Le gain de Shor, lui, fait passer un problème de 2^128 opérations à quelques millions d'opérations — aucune augmentation de taille raisonnable ne le compense, il faut changer de famille mathématique.

### Shor (algorithme de, 1994)

Algorithme quantique qui résout en temps polynomial la factorisation d'entiers et le logarithme discret, y compris sur courbe elliptique (ECDLP). C'est-à-dire : retrouver la clé privée *d* à partir de la clé publique *P = d·G*.

**Implication quantique** : c'est la menace réelle. Elle ne concerne que la cryptographie à clé publique — RSA, Diffie-Hellman, ECDSA, Schnorr. Dans Bitcoin, elle ne mord que sur les clés publiques *visibles en clair*. D'où la règle centrale : une clé publique exposée est une clé privée à terme.

### secp256k1

La courbe elliptique utilisée par Bitcoin, choisie par Satoshi : y² = x³ + 7 sur le corps fini de caractéristique p = 2^256 − 2^32 − 977. C'est une courbe de Koblitz, standardisée par le SECG, dont l'ordre du groupe avoisine 2^256, offrant \~128 bits de sécurité classique.

**Implication quantique** : les estimations de ressources quantiques visent précisément cette courbe. Google Quantum AI (2026) : moins de 1 200 qubits logiques et moins de 500 000 qubits physiques pour casser une clé en quelques minutes sur architecture à cycle rapide. La sécurité de secp256k1 contre un attaquant quantique est nulle une fois la machine disponible ; sa robustesse actuelle ne vient que de l'absence de cette machine.

### ASIC

*Application-Specific Integrated Circuit* : puce gravée pour une seule tâche. Dans Bitcoin, les ASIC de minage ne savent faire qu'une chose — du double SHA-256 — et le font à des centaines de térahashs par seconde, soit plus de 10^14 opérations par seconde et par machine.

**Implication quantique** : c'est l'argument massue contre l'idée d'un « minage quantique ». Les portes quantiques s'exécutent à des cadences de l'ordre du microseconde à la milliseconde, et chaque opération logique tolérante aux fautes mobilise des milliers de qubits physiques pour la correction d'erreurs. Le gain quadratique de Grover est écrasé par cet écart de vitesse de plusieurs ordres de grandeur.

## Structures et protocole Bitcoin

### UTXO

*Unspent Transaction Output* : une sortie de transaction non encore dépensée. Bitcoin n'a pas de comptes ni de soldes ; il n'a qu'un ensemble d'UTXO, chacun verrouillé par un script de verrouillage (*scriptPubKey*). Votre « solde » est la somme des UTXO que votre wallet peut déverrouiller. Un UTXO se dépense en totalité : le surplus revient en rendu de monnaie sur une nouvelle sortie.

**Implication quantique** : l'unité de risque est l'UTXO, pas l'adresse ni le wallet. La question à poser pour chaque UTXO est : « la clé publique qui le déverrouille est-elle visible quelque part ? ». Un même wallet peut contenir des UTXO exposés et d'autres non.

### ECDSA / Schnorr

Les deux schémas de signature de Bitcoin, tous deux construits sur secp256k1. **ECDSA** est le schéma d'origine (2009), utilisé par P2PK, P2PKH, P2SH, P2WPKH et P2WSH. **Schnorr** (BIP-340) est arrivé avec Taproot en 2021 : signatures plus courtes, vérifiables par lot, et surtout linéaires, ce qui permet l'agrégation de clés (MuSig, FROST).

**Implication quantique** : la distinction n'a aucune importance ici. La vulnérabilité ne vient pas du schéma de signature mais de la courbe sous-jacente, commune aux deux. Shor casse l'un comme l'autre. Les propositions de sortie (BIP-360, SHRINCS, ML-DSA, Falcon) ne consistent donc pas à corriger ECDSA ou Schnorr, mais à les remplacer par des primitives d'une autre famille mathématique — réseaux euclidiens ou constructions fondées sur le hachage.

### PSBT

*Partially Signed Bitcoin Transaction* (BIP-174) : format standard d'échange d'une transaction en cours de construction entre plusieurs logiciels ou signataires — typiquement un wallet watch-only, un hardware wallet, et les cosignataires d'un multisig.

**Implication quantique** : un PSBT contient les clés publiques et les chemins de dérivation des entrées concernées. Le transmettre par e-mail, cloud ou clé USB partagée, ou le stocker sans précaution, revient à divulguer ces clés publiques hors de la blockchain — y compris pour des UTXO qui n'ont jamais été dépensés et que l'on croit protégés par leur empreinte.

### full-RBF

*Replace-By-Fee* : mécanisme permettant de remplacer une transaction non confirmée par une transaction conflictuelle payant des frais supérieurs. Historiquement optionnel (BIP-125, il fallait le signaler dans la transaction), le remplacement est devenu **inconditionnel par défaut** depuis Bitcoin Core 28 (octobre 2024) : toute transaction non confirmée est remplaçable, qu'elle l'ait signalé ou non.

**Implication quantique** : c'est ce qui rend l'attaque mempool réalisable sans puissance de calcul de minage. L'attaquant qui casse la clé publique d'une transaction en attente n'a pas besoin de miner un bloc ni de corrompre un mineur : il construit une transaction concurrente dépensant les mêmes entrées vers lui-même, y met des frais plus élevés, et laisse la politique de relais standard faire le reste. La seule contre-mesure individuelle est de payer d'emblée des frais suffisants pour raccourcir la fenêtre.

### xpub

*Extended Public Key* (BIP-32) : une clé publique **étendue**, c'est-à-dire la concaténation d'une clé publique et d'un *chain code* de 32 octets. Elle occupe un nœud de l'arbre de dérivation hiérarchique d'un wallet (HD wallet) et sert à générer les adresses de réception sans détenir la moindre clé privée. C'est ce qu'on fournit à un wallet watch-only, à un explorateur de solde, à un logiciel de comptabilité ou de fiscalité, à un coordinateur de coinjoin, ou ce que l'on retrouve dans un descripteur de wallet multisig.

**Ce qu'implique sa divulgation** — trois niveaux, de gênant à catastrophique :


1. **Perte totale de confidentialité.** Le détenteur de l'xpub dérive toutes vos adresses passées et futures sur cette branche, et reconstitue l'intégralité de votre historique, de vos soldes et de vos habitudes de dépense. Ce risque est immédiat et n'a rien de quantique.
2. **Exposition de toutes vos clés publiques, sans qu'aucune dépense n'ait eu lieu.** C'est le point décisif pour le sujet qui nous occupe. La dérivation non durcie (*non-hardened*, chemins de réception et de change usuels) permet de calculer les clés publiques **enfants** à partir de l'xpub parent seul. Toutes les clés publiques de la branche sont donc connues — y compris celles d'adresses P2PKH ou P2WPKH neuves, jamais dépensées, dont on croyait la clé protégée derrière une empreinte HASH160. Le raisonnement « je n'ai jamais dépensé depuis cette adresse, donc ma clé n'est pas exposée » tombe intégralement. Un xpub collé dans un explorateur en 2019 expose aujourd'hui l'ensemble du wallet aussi sûrement que des sorties Taproot.
3. **Compromission totale, sans quantique, si une seule clé privée enfant fuite.** Propriété structurelle de la dérivation non durcie de BIP-32 : **xpub parent + une clé privée enfant non durcie ⇒ clé privée parente**, donc l'ensemble du wallet. Un xpub partagé transforme la fuite d'une clé privée isolée en perte complète. C'est la raison pour laquelle les niveaux supérieurs d'un chemin de dérivation (`m/84'/0'/0'`) utilisent la dérivation durcie.

**À retenir** : ne partagez un xpub que si vous acceptez ces trois conséquences ; préférez un xpub de compte dédié, isolé du reste, et considérez comme exposé tout wallet dont l'xpub a circulé, y compris ses adresses vierges.

## Propositions de protocole

### BIP-361 — *Post Quantum Migration and Legacy Signature Sunset*

Proposition rédigée par Jameson Lopp, Christian Papathanasiou, Ian Smith, Joe Ross, Steve Vaile et Pierre-Luc Dallaire-Demers, assignée en février 2026. Statut : **Draft**, type *Informational*, soft fork. Elle dépend d'une proposition préalable définissant un schéma de signature post-quantique — elle organise la migration, elle ne fournit pas la cryptographie.

Calendrier proposé, en deux phases : **Phase A**, environ 160 000 blocs (\~3 ans) après activation, il devient interdit d'*envoyer* des fonds vers des scripts quantiquement vulnérables ; **Phase B**, deux ans plus tard (jour-charnière à cinq ans), la validation des signatures ECDSA et Schnorr est durcie, ce qui revient à neutraliser les dépenses legacy. Le texte actuel prévoit une voie de récupération pour les fonds bloqués, par preuve à divulgation nulle de connaissance de la seed (ZK-STARK sur dérivation durcie BIP-32) ou schéma commit-reveal — le cadrage est passé du « gel pur » au « gel assorti d'un sauvetage ».

**Enjeu** : c'est la formulation la plus aboutie du débat **freeze vs steal**. Trois positions s'affrontent. Laisser quiconque voler les coins vulnérables — le premier détenteur d'une machine rafle 6 à 7 millions de BTC. Limiter le débit de vol (proposition *Hourglass*, restreignant le rythme de dépense des P2PK) — les auteurs de BIP-361 la soutiennent pour les seuls P2PK, où aucune preuve de possession n'est possible puisque ces sorties sont antérieures à BIP-32. Ou n'autoriser personne à voler, position défendue par le BIP lui-même, résumée par ses auteurs : *« Quantum recovered coins only make everyone else's coins worth less. »* À noter que BIP-360, le volet technique, refuse explicitement de trancher cette question politique.

## Tableau des types de PAY TO

| Type | Appellation courante | Encodage | Préfixe d'adresse |
|----|----|----|----|
| **P2PK** | *Pay to Public Key* — sortie coinbase historique | Aucun : la clé publique brute figure dans le scriptPubKey | *Aucun* — ces sorties n'ont pas de format d'adresse |
| **P2MS** | *Bare multisig* — multisig nu | Aucun : les clés publiques figurent en clair dans le scriptPubKey | *Aucun* |
| **P2PKH** | Adresse *legacy* | Base58Check, octet de version `0x00` | `1` |
| **P2SH** | Adresse script, dite *legacy compatible* | Base58Check, octet de version `0x05` | `3` |
| **P2SH-P2WPKH** | SegWit imbriqué (*nested* / *wrapped*) | Base58Check `0x05` — un P2SH encapsulant un témoin v0 | `3` (indistinguable d'un P2SH ordinaire) |
| **P2WPKH** | SegWit natif, *bech32* | Bech32, HRP `bc`, témoin v0, programme de 20 octets | `bc1q` (42 caractères) |
| **P2WSH** | SegWit natif script | Bech32, HRP `bc`, témoin v0, programme de 32 octets | `bc1q` (62 caractères) |
| **P2TR** | Taproot | Bech32m, HRP `bc`, témoin v1, programme de 32 octets | `bc1p` |
| **P2MR** *(BIP-360, Draft)* | *Pay to Merkle Root* — ex-P2QRH, ex-P2TSH | Bech32m, HRP `bc`, témoin v2 | `bc1z` *(non activé)* |

> Le caractère qui suit `bc1` n'est pas décoratif : il encode la version du témoin — `q` pour v0, `p` pour v1, `z` pour v2. Et le code correcteur change avec elle : Bech32 pour le témoin v0, Bech32m (BIP-350) à partir du v1.

**Exposition de la clé publique, par type :**

* Exposée **par conception**, avant toute dépense : P2PK, P2MS, P2TR.
* Exposée **seulement après une dépense** : P2PKH, P2SH, P2SH-P2WPKH, P2WPKH, P2WSH — et, pour les variantes à script, ce sont les clés de *tous* les cosignataires qui sont révélées, signataires ou non.
* Exposée **hors chaîne**, quel que soit le type et sans aucune dépense : tout wallet dont l'xpub a circulé.