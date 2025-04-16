---
layout: default
title: "ZkLighter Protocol : Point de vue Technique"
parent: Rapport
categories: "rapport"
nav_order: 1
author: GuruTanuki_
license: CC BY-ND 4.0
---

# ZkLighter Protocol : Point de vue Technique

license: CC BY-ND 4.0
Auteur : GuruTanuki_  
X : [@GuruTanuki_](https://x.com/GuruTanuki_)

---

## Introduction et Problématique

Le paysage du trading est confronté à des défis majeurs en matière de sécurité, de transparence et d'efficacité. Le 16/04/2025, Binance annonce avoir des soucis à cause de problèmes chez AWS. Les systèmes traditionnels centralisés souffrent aussi de risques de contrepartie, de manque de transparence et de vulnérabilité aux opérateurs malveillants.

Bien que la technologie blockchain offre une décentralisation de la confiance et une auditabilité publique, les plateformes existantes comme Ethereum présentent des limitations importantes pour les carnets d'ordres (order books) en raison des frais de transaction élevés et des temps de bloc longs, rendant les opérations à faible latence et haute fréquence économiquement non viables, en particulier pour les marchés volatils (crypto, actions, matières premières).

Cela a conduit à la popularité des Teneurs de Marché Automatisés (AMM), qui, malgré leurs avantages, souffrent d'inefficacités :

- Dépendance de l'arbitrage pour l'ajustement des prix, besoin de liquidité important, risque de slippage élevé (surtout pour les marchés peu liquides ou volatils) et risque de perte impermanente pour les fournisseurs de liquidité, souvent négligé.
- Les solutions de Layer 2 généralistes (comme Arbitrum) ont réduit les coûts, mais leur nature non spécialisée entraîne des inefficacités de performance et de coût par rapport aux rollups spécifiques à une application.
- Les solutions hybrides (comme 0x Limit Order Protocol) déplacent le calcul hors chaîne mais conservent un carnet d'ordres centralisé, ce qui empêche la vérification on-chain de la priorité prix-temps et crée un risque de censure par l'opérateur centralisé.
- D'autres solutions comme StarkEx améliorent la scalabilité avec des preuves de validité mais gardent les moteurs de correspondance hors chaîne et centralisés, limitant la vérification aux ordres croisés et introduisant un risque de censure et de favoritisme.
- Enfin, de nouvelles solutions Layer 1 (comme dYdX v4, Hyperliquid) visent une correspondance vérifiable mais sont intrinsèquement plus centralisées qu'Ethereum, reposant sur leur propre consensus, ce qui peut compromettre la sécurité. Leurs temps de bloc plus longs augmentent la latence et les opportunités de MEV (Maximal Extractable Value). De plus, étant hors de l'écosystème Ethereum, elles manquent de composabilité et dépendent de ponts (bridges) pour les transferts d'actifs.

---

## La Solution zkLighter

ZkLighter propose un zk-rollup spécifique à une application, conçu pour les applications financières scalables, sécurisées, transparentes, non dépositaires (non-custodial) et équitables. Opérant sur des blockchains compatibles EVM (comme Ethereum), zkLighter tire parti de la sécurité et de la transparence de la blockchain sous-jacente tout en offrant une scalabilité.

L'innovation clé de zkLighter réside dans ses moteurs de correspondance (matching) et de liquidation vérifiables, capables de générer des preuves succinctes (zk-SNARKs) pour l'exécution d'ordres selon la priorité prix-temps et pour les liquidations. Cela garantit non seulement l'exactitude des transactions mais aussi l'équité dans le traitement des ordres, éliminant le risque de censure ou de réorganisation malveillante par l'opérateur du rollup (le Sequencer).

Le moteur de correspondance est hautement optimisé, capable (d’après le whitepaper) d'exécuter des milliers d'opérations par seconde avec une latence extrêmement faible, rivalisant avec les bourses traditionnelles. En garantissant qu'un ordre limite placé ne peut être ni censuré ni voir sa priorité modifiée, zkLighter réduit considérablement le risque de MEV. Une entité malveillante devrait acheter toute la liquidité disponible jusqu'au niveau de prix souhaité pour exploiter un ordre, ce qui serait probablement non rentable sur un marché liquide.

zkLighter se présente comme une primitive pour construire de nouvelles plateformes de trading numérique haute performance et sécurisées, en commençant par une bourse de contrats à terme perpétuels (perpetual futures).

![Revolution zkLighter](/assets/zkLighter/Revolution%20zkLighter.png)

---

## Architecture du Protocole zkLighter

Le protocole zkLighter repose sur trois composants principaux :

### Le Sequencer

- Interface primaire pour les interactions utilisateur (transactions de rollup).
- Organise les transactions utilisateur dans des blocs structurés selon un principe de premier arrivé, premier servi.
- Intègre des oracles décentralisés (non spécifiés) pour obtenir les prix d'indice nécessaires aux calculs de marge et de financement.
- Publie les données de ces blocs sur la chaîne (Smart Contracts sur Ethereum) pour un stockage immuable et génère les données "témoins" nécessaires au Prover.
- Fournit une API et un service WebSocket pour l'accès aux données en temps réel.
- Son rôle est limité à l'ordonnancement des transactions. L'exécution et la correspondance sont vérifiées par le Prover, ce qui facilite la décentralisation future du Sequencer et limite son pouvoir de censure.

ZkLighter utilise un schéma de pré-engagement pour garantir la finalité instantanée et l'intégrité des données publiées par le Sequencer via un flux de données.

### Le Prover

- Génère des preuves zk-SNARK succinctes pour l'exécution des blocs zkLighter et les transitions d'état résultantes.
- Utilise des circuits arithmétiques personnalisés pour vérifier les changements d'état (dépôts, retraits, trades, liquidations).
- Les entrées des circuits sont l'entrée publique (hash composite du bloc zkLighter et de la racine d'état précédente) et le témoin (données d'état supplémentaires fournies par le Sequencer).
- La preuve générée, vérifiée par les Smart Contracts avec l'entrée publique, confirme l'exécution correcte des transactions et permet la mise à jour de l'état du rollup.
- Utilise des structures de données innovantes (notamment l'Order Book Tree) pour vérifier la correspondance d'ordres selon la priorité prix-temps.

### Les Smart Contracts

- Déployés sur la blockchain de base (ex : Ethereum), ils maintiennent l'intégrité du système.
- Stockent la racine d'état actuelle (State Root) du rollup, archivent les données des blocs précédents pour assurer la disponibilité des données, et vérifient les preuves soumises par le Prover pour autoriser les mises à jour d'état.
- Gèrent la file d'attente des transactions prioritaires (ex : retraits résistants à la censure), forçant leur inclusion par le Sequencer dans un délai imparti. Si le Sequencer échoue, les Smart Contracts activent le mécanisme Exit Hatch.
- Implémentent la logique de vérification pour le flux de pré-engagement assurant la finalité instantanée.

![Transaction zkLighter](/assets/zkLighter/Transaction%20zkLighter.png)

---

## Structure de l'État zkLighter

L'état complet de zkLighter est encapsulé dans un ensemble de structures de données, dont la racine globale est le zkLighter State Root, un hash composite de deux éléments principaux :

- **Account Tree Root** : Racine d'un arbre de Merkle représentant l'état des comptes utilisateurs (collatéral, positions). [Thread explicatif](https://x.com/GuruTanuki_/status/1910201365747679485)
- **Validium Root** : Racine d'un hash composite de 4 éléments d'état non entièrement récupérables à partir des données publiques (pour optimiser les coûts on-chain), comme l'état du carnet d'ordres.

### Éléments du Validium Root

- **Market Tree Root** : Racine d'un arbre de Merkle contenant la configuration et l'état de chaque marché (y compris son carnet d'ordres).
- **Account Metadata Tree Root** : Racine d'un arbre de Merkle stockant des métadonnées de compte (nonce, clé publique, taille totale des ordres ouverts).
- **Transaction Tree Root** : Racine d'un arbre de Merkle stockant les transactions pour prouver leur exécution individuelle et pour le mécanisme de pré-engagement.
- **Hash de l'Instruction Register** : Registre stockant l'état des opérations multi-cycles en cours.

### Structures de données clés

- **Account Tree** : Arbre de Merkle où chaque feuille contient l'adresse L1, le collatéral principal, les positions sur chaque marché, et la racine d'un Sub-Account Tree. Permet des sous-comptes avec des signataires distincts.
- **Account Metadata Tree** : Stocke des informations comme le nonce du compte, la clé publique du sous-compte, et la taille totale des ordres ouverts par type (Ask/Bid).
- **Transaction Tree** : Chaque transaction reçoit un index unique, permettant de prouver son exécution via une preuve de Merkle.
- **Market Tree** : Chaque feuille représente un marché et contient sa configuration et la racine de son Order Book Tree.
- **Order Book Tree (OBT)** : Arbre binaire parfait spécialisé et optimisé pour les circuits zkLighter. Stocke dynamiquement les ordres actifs et supporte l'insertion, l'annulation, la récupération de cotations et l'exécution du meilleur trade avec une taille de circuit constante (complexité logarithmique en nombre d'ordres, O(log N)).
- **Indexation Prix-Temps** : L'index de la feuille d'un ordre est calculé à partir de son prix (P bits) et d'un nonce unique (O bits) représentant le temps.  
  - Pour les asks : `index = prix * 2^O + nonce`
  - Pour les bids : `index = prix * 2^O + (2^O - 1 - nonce_inverse)`
- **Données Internes** : Chaque nœud interne stocke des sommes agrégées des ordres actifs dans son sous-arbre (AskSizeSum, BidSizeSum, AskQuoteSum, BidQuoteSum).
- **Instruction Register** : Registre pour gérer les opérations nécessitant de modifier plusieurs feuilles d'arbres de Merkle en un seul "événement" logique.
- **Market Metadata** : Informations cruciales sur chaque marché (prix d'indice, prix de marque, configuration de marge, données pour le financement) incluses dans les données de chaque bloc publié on-chain.

![Architecture zkLighter](/assets/zkLighter/Architecture%20zkLighter.png)

---

## Mécanismes et Processus Clés

- **Transactions** :  
  - Rollup Transactions (gestion d'ordres/positions, via Sequencer, sans frais L1 directs)  
  - Priority Transactions (ex : retraits résistants à la censure, via Smart Contracts, avec frais L1 Exit Hash)

- **Correspondance d'Ordres Vérifiable** :  
  Les circuits zkLighter vérifient que les ordres sont appariés strictement selon la priorité prix-temps en utilisant l'Order Book Tree et ses données internes. Lors de la création d'un ordre, les circuits calculent la taille totale correspondante avec les ordres existants (en utilisant les sommes des nœuds internes de l'OBT), mettent à jour les soldes et insèrent l'ordre restant (si partiel) dans l'OBT. Si plusieurs trades sont nécessaires, l'Instruction Register est utilisé pour dérouler l'exécution sur plusieurs cycles.

- **Contrats à Terme Perpétuels** :
  - **Prix de Marque (Mark Price)** : Calculé pour représenter le prix "juste" du contrat, basé sur le prix d'indice, le taux de financement précédent et un TWAP dérivé de la liquidité du carnet d'ordres.
  - **Marges et Liquidation** : Niveaux de marge multiples (Initial, Maintenance, Close-out) et cascade de liquidation (Waterfall).
  - **Mécanisme de Financement (Funding)** : Assure la convergence du prix du contrat vers le prix spot, calculé périodiquement et échangé entre les positions longues et courtes. Un mécanisme de cache est utilisé pour appliquer le financement aux comptes uniquement lorsque nécessaire.

- **Circuits zkLighter** :
  - **Pré-Exécution** : Met à jour l'état des marchés perpétuels.
  - **Cycle d'Exécution** : Exécute une nouvelle transaction ou continue une opération multi-cycles via l'Instruction Register.

---

## Sécurité et Vivacité

- **Exit Hatch** : Mécanisme de sécurité activé par les Smart Contracts si le Sequencer échoue. Gèle l'état du rollup et permet aux utilisateurs de retirer leurs fonds directement via les Smart Contracts en fournissant des preuves de Merkle contre l'état gelé.
- **Finalité et Pré-engagements** : La finalité L1 est atteinte lorsque la preuve du bloc est vérifiée on-chain. zkLighter offre une finalité quasi-instantanée via un flux de données où le Sequencer signe et publie les transactions en attente.
- **Déploiement et Limitation de Taux** : Prévu sur Ethereum. Contrats initialement améliorables via timelock, puis rendus immuables. Des limiteurs de taux dynamiques sont mis en place au niveau du Sequencer pour prévenir les attaques DDoS et le spamming.
- **Résistance au MEV** : L'architecture garantit la priorité prix-temps, limitant le MEV par réorganisation. Des recherches sont en cours pour minimiser davantage le MEV (séquençage équitable, chiffrement temporel).

![Protocol zkLighter](/assets/zkLighter/Protocol%20zkLighter.png)

---

## Conclusion et Travaux Futurs

zkLighter apporte des avancées techniques pour le trading efficace et transparent. En intégrant des moteurs de correspondance et de liquidation vérifiables via zk-SNARKs et des structures de données innovantes comme l'Order Book Tree, il garantit la priorité prix-temps, réduit le risque de censure et devrait offrir un faible slippage.

Les travaux futurs se concentrent sur l'extension de la plateforme à d'autres instruments financiers (spot, options, marchés de prêt), la minimisation continue du MEV potentiel via des techniques de séquençage équitable et de chiffrement, et la décentralisation complète du protocole tout en préservant la faible latence et la sécurité des utilisateurs.

L'objectif est de réinventer l'infrastructure financière traditionnelle avec une transparence et des garanties de sécurité accrues.

---

## Avis personnel

Bon, on a gratté dans le sens du poil, voilà maintenant mon avis sans filtre.

- **Dépendance à ETH** : Bien que le projet semble pouvoir se connecter à tous les EVM, il semble que la solution (logique) soit ETH. Très bien au niveau de la sécurité, cela a aussi des désavantages avec les congestions et les frais élevés que peut apporter ETH. À voir dans le futur si cela change.
- **Centralisation du séquenceur** : J'ai vu dans ce projet une volonté forte de vouloir décentraliser le séquenceur, mais mon avis là-dessus restera toujours le même : quelle est la rentabilité ?
- **Surcouche complexe** : circuits zkSnarks, arbre du carnet d'ordres, OBT... Il y a beaucoup de nouvelles technologies introduites dans ce projet, parfois même uniques, ce qui pose le même problème : la sécurité ! Complexité = N*risk !
- **Contrats améliorables** : Bon, ce n'est pas nouveau, mais c'est toujours embêtant de devoir attendre l'abandon des clés administrateur...
- **Dépendance aux oracles de prix** : Nous n'avons toujours pas la liste des oracles qui seront utilisés.

Par contre, je suis ravi de voir enfin un projet vouloir apporter des solutions au problème du MEV !

En bref, on manque d'historique ! Le projet, sur le papier, est vraiment cool, mais il reste du chemin avant de voir leur réelle avancée.
