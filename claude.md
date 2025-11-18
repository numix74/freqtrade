# Guide Claude pour Freqtrade

## À propos du projet

Freqtrade est un bot de trading de crypto-monnaies gratuit et open source écrit en Python. Il est conçu pour prendre en charge toutes les principales plateformes d'échange et peut être contrôlé via Telegram ou une interface web. Il comprend des outils de backtesting, de visualisation et de gestion de l'argent, ainsi que l'optimisation de stratégies par apprentissage automatique.

## Structure du projet

### Répertoires principaux

- **docs/** - Documentation complète du projet
- **freqtrade/** - Code source principal du bot
- **tests/** - Tests unitaires et d'intégration
- **user_data/** - Données utilisateur (stratégies, configurations)

### Technologies utilisées

- Python 3.11+
- SQLite pour la persistance
- ccxt pour l'intégration des exchanges
- Telegram pour le contrôle à distance
- FreqAI pour l'apprentissage automatique

## Conventions de développement

### Branches

- `develop` - Branche de développement (peut contenir des changements de rupture)
- `stable` - Dernière version stable
- `feat/*` - Branches de fonctionnalités en cours de développement

### Contribution

- Toujours créer les PR contre la branche `develop`, jamais contre `stable`
- Lire [CONTRIBUTING.md](CONTRIBUTING.md) avant de contribuer
- Pour les nouvelles fonctionnalités majeures, ouvrir d'abord une issue ou discuter sur Discord

## Commandes principales

### Bot

- `freqtrade trade` - Lancer le bot en mode trading
- `freqtrade backtesting` - Effectuer un backtest
- `freqtrade hyperopt` - Optimiser les paramètres de stratégie
- `freqtrade download-data` - Télécharger les données historiques

### Développement

- Tests : `pytest`
- Linter : Utilisation de pre-commit hooks
- Documentation : Disponible sur [www.freqtrade.io](https://www.freqtrade.io)

## Avertissement important

Ce logiciel est destiné à des fins éducatives uniquement. Ne risquez pas d'argent que vous avez peur de perdre. UTILISEZ LE LOGICIEL À VOS PROPRES RISQUES.

- Toujours commencer en mode Dry-run
- Avoir des connaissances en Python est fortement recommandé
- Lire et comprendre le code source avant utilisation

## Exchanges supportés

### Spot Trading
- Binance, BingX, Bitget, Bitmart, Bybit
- Gate.io, HTX, Hyperliquid
- Kraken, OKX, et potentiellement beaucoup d'autres via ccxt

### Futures Trading (expérimental)
- Binance, Bitget, Gate.io
- Hyperliquid, OKX, Bybit

Consulter [docs/exchanges.md](docs/exchanges.md) pour les notes spécifiques à chaque exchange.

## Fonctionnalités clés

- Mode Dry-run pour tester sans risque financier
- Backtesting avec données historiques
- Optimisation de stratégie par machine learning (FreqAI)
- Gestion de whitelist/blacklist de crypto-monnaies
- Interface web intégrée
- Contrôle via Telegram
- Affichage des profits/pertes en monnaie fiat

## Support

- Documentation : [www.freqtrade.io](https://www.freqtrade.io)
- Discord : [discord.gg/p7nuUNVfP7](https://discord.gg/p7nuUNVfP7)
- Issues : [github.com/freqtrade/freqtrade/issues](https://github.com/freqtrade/freqtrade/issues)

## Configuration minimale

### Hardware
- 2GB RAM minimum
- 1GB d'espace disque
- 2 vCPU

### Software
- Python >= 3.11
- pip, git
- TA-Lib
- virtualenv (recommandé)
- Docker (recommandé)

### Important
L'horloge système doit être précise et synchronisée avec un serveur NTP pour éviter les problèmes de communication avec les exchanges.
