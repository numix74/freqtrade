# Guide Complet Claude pour Freqtrade

## Table des matières

1. [Vue d'ensemble](#vue-densemble)
2. [Architecture du projet](#architecture-du-projet)
3. [Structure détaillée des modules](#structure-détaillée-des-modules)
4. [Concepts clés](#concepts-clés)
5. [Développement et contribution](#développement-et-contribution)
6. [Commandes détaillées](#commandes-détaillées)
7. [Configuration](#configuration)
8. [API et RPC](#api-et-rpc)
9. [Tests et qualité](#tests-et-qualité)
10. [Ressources](#ressources)

---

## Vue d'ensemble

**Freqtrade** est un bot de trading de crypto-monnaies gratuit et open source écrit en Python. Conçu pour être flexible et extensible, il supporte toutes les principales plateformes d'échange et peut être contrôlé via Telegram, Discord ou une interface web (FreqUI).

### Caractéristiques principales

- **Trading automatisé** : Exécution de stratégies personnalisées 24/7
- **Backtesting avancé** : Test de stratégies sur données historiques
- **Hyperoptimisation** : Optimisation automatique des paramètres de stratégie
- **FreqAI** : Intégration d'apprentissage automatique pour stratégies adaptatives
- **Mode Dry-run** : Simulation sans risque financier
- **Multi-exchange** : Support de nombreuses plateformes (Binance, Bybit, OKX, etc.)
- **Trading Spot et Futures** : Support du trading au comptant et à effet de levier
- **Extensibilité** : Architecture pluggable pour stratégies, pairlists, protections

### Avertissement critique

⚠️ **CE LOGICIEL EST À DES FINS ÉDUCATIVES UNIQUEMENT**

- Ne risquez **JAMAIS** d'argent que vous ne pouvez pas vous permettre de perdre
- Utilisez le logiciel **À VOS PROPRES RISQUES**
- Les auteurs n'assument **AUCUNE RESPONSABILITÉ** pour vos résultats de trading
- **Commencez TOUJOURS en mode Dry-run**
- **Comprenez le code source** avant d'engager de l'argent réel
- Des **connaissances en Python** sont fortement recommandées

---

## Architecture du projet

### Structure des répertoires

```
freqtrade/
├── freqtrade/              # Code source principal
│   ├── commands/           # Commandes CLI (trade, backtesting, hyperopt, etc.)
│   ├── configuration/      # Gestion de la configuration
│   ├── data/              # Gestion des données (historique, conversion)
│   ├── enums/             # Énumérations (SignalType, ExitType, etc.)
│   ├── exchange/          # Intégration des exchanges (via ccxt)
│   ├── freqai/            # Module d'apprentissage automatique
│   ├── leverage/          # Calculs pour trading avec levier
│   ├── optimize/          # Backtesting, hyperoptimisation
│   ├── persistence/       # Modèles de base de données (SQLAlchemy)
│   ├── plugins/           # Pairlists, Protections
│   ├── resolvers/         # Chargement dynamique de stratégies/plugins
│   ├── rpc/               # API REST, Telegram, Discord, Webhooks
│   ├── strategy/          # Interface et helpers pour stratégies
│   ├── templates/         # Templates de stratégies
│   ├── util/              # Utilitaires divers
│   ├── freqtradebot.py    # Cœur du bot de trading
│   ├── main.py            # Point d'entrée principal
│   └── constants.py       # Constantes globales
│
├── docs/                  # Documentation complète (MkDocs)
├── tests/                 # Tests unitaires et d'intégration (pytest)
├── scripts/               # Scripts utilitaires
├── user_data/             # Données utilisateur (non versionné)
│   ├── strategies/        # Stratégies personnalisées
│   ├── data/              # Données de marché téléchargées
│   ├── notebooks/         # Jupyter notebooks pour analyse
│   └── backtest_results/  # Résultats de backtests
└── config.json            # Configuration principale
```

### Technologies et dépendances clés

| Composant | Technologie | Usage |
|-----------|-------------|-------|
| **Langage** | Python 3.11+ | Développement principal |
| **Échange** | ccxt | Connexion aux exchanges |
| **Base de données** | SQLAlchemy + SQLite/PostgreSQL | Persistance des trades |
| **API Web** | FastAPI | Interface REST |
| **WebSocket** | FastAPI WebSocket | Streaming temps réel |
| **Interface Web** | FreqUI (Vue.js) | Interface utilisateur |
| **Messaging** | python-telegram-bot | Contrôle Telegram |
| **Analyse technique** | pandas, TA-Lib, pandas-ta | Indicateurs techniques |
| **ML/AI** | scikit-learn, catboost, lightgbm | FreqAI |
| **Backtesting** | numba (optionnel) | Accélération calculs |
| **Graphiques** | plotly | Visualisation |
| **Tests** | pytest, pytest-cov | Tests unitaires |
| **Linting** | ruff, mypy | Qualité du code |

---

## Structure détaillée des modules

### 1. `freqtrade/freqtradebot.py`

**Cœur du système de trading** (3000+ lignes)

Responsabilités principales :
- Gestion du cycle de vie des trades (ouverture, mise à jour, fermeture)
- Exécution des signaux de stratégie
- Gestion des ordres (placement, annulation, vérification)
- Application du stoploss et ROI
- Ajustement de position (DCA/Position averaging)
- Gestion du wallet et du stake amount

Classes principales :
- `FreqtradeBot` : Contrôleur principal du bot

### 2. `freqtrade/strategy/`

**Système de stratégies**

Fichiers clés :
- `interface.py` : Classe abstraite `IStrategy` - base de toutes les stratégies
- `hyper.py` : `HyperStrategyMixin` - support hyperoptimisation
- `informative_decorator.py` : Décorateurs pour données multi-timeframe
- `parameters.py` : Paramètres hyperopt (Integer, Decimal, Categorical, etc.)

Une stratégie doit implémenter :
```python
class MyStrategy(IStrategy):
    # Configuration
    minimal_roi = {"0": 0.10}
    stoploss = -0.05
    timeframe = '5m'

    # Méthodes obligatoires
    def populate_indicators(self, dataframe, metadata):
        # Calcul des indicateurs
        return dataframe

    def populate_entry_trend(self, dataframe, metadata):
        # Signaux d'entrée (long/short)
        return dataframe

    def populate_exit_trend(self, dataframe, metadata):
        # Signaux de sortie
        return dataframe
```

### 3. `freqtrade/exchange/`

**Abstraction des exchanges**

- `exchange.py` : Classe de base `Exchange`
- Sous-classes spécifiques : `binance.py`, `bybit.py`, `okx.py`, etc.
- Gestion des particularités de chaque exchange
- Wrapper autour de ccxt avec retry logic et gestion d'erreurs
- Support des marchés Spot et Futures

Fonctionnalités :
- Récupération de données OHLCV
- Placement/annulation d'ordres
- Gestion du leverage et margin
- Récupération des tickers, balances
- Validation des paires de trading

### 4. `freqtrade/optimize/`

**Backtesting et Hyperoptimisation**

Fichiers principaux :
- `backtesting.py` : Moteur de backtesting vectorisé
- `hyperopt_tools.py` : Outils pour hyperoptimisation
- `hyperopt_epoch_filters.py` : Filtres pour résultats hyperopt

Le backtesting simule l'exécution de stratégies :
- Support des données tick (trades) pour précision
- Simulation des frais, slippage
- Calcul des métriques (Sharpe, Sortino, Max Drawdown, etc.)
- Export des résultats pour analyse

### 5. `freqtrade/rpc/`

**Interfaces de contrôle à distance**

Modules :
- `rpc.py` : Logique RPC commune
- `telegram.py` : Bot Telegram
- `discord.py` : Bot Discord
- `api_server/` : API REST (FastAPI)
  - `api_v1.py` : Endpoints v1
  - `api_backtest.py` : Endpoints backtesting
  - `api_ws.py` : WebSocket pour streaming
  - `webserver.py` : Serveur principal

Commandes RPC disponibles :
- Status des trades ouverts
- Performance, profit/loss
- Balance du wallet
- Forcer achat/vente
- Gestion des blacklists
- Statistiques détaillées

### 6. `freqtrade/data/`

**Gestion des données**

- `history/` : Téléchargement et stockage des données historiques
- `converter/` : Conversion entre formats (JSON, Parquet, Feather)
- `dataprovider.py` : Fournisseur de données pour stratégies
- `btanalysis.py` : Analyse des résultats de backtest

Formats supportés :
- JSON (gzippé ou non)
- Parquet (recommandé pour performance)
- Feather
- HDF5 (legacy)

### 7. `freqtrade/freqai/`

**Intelligence artificielle**

Composants :
- `freqai_interface.py` : Interface principale FreqAI
- `data_kitchen.py` : Préparation des features
- `data_drawer.py` : Stockage des prédictions
- `prediction_models/` : Modèles ML (Classifiers, Regressors, RL)

Modèles supportés :
- LightGBM, CatBoost, XGBoost
- PyTorch (réseaux de neurones)
- Reinforcement Learning (PPO, A2C, DQN)

### 8. `freqtrade/persistence/`

**Persistance et base de données**

Modèles SQLAlchemy :
- `Trade` : Représente un trade (ouvert ou fermé)
- `Order` : Ordres associés aux trades
- `PairLock` : Verrouillages de paires
- `KeyValueStore` : Stockage clé-valeur générique

Bases de données supportées :
- SQLite (par défaut, bon pour dev/test)
- PostgreSQL (recommandé pour production)
- MySQL/MariaDB

---

## Concepts clés

### Stratégies de trading

Une stratégie définit :
1. **Indicateurs** : Calculs techniques (RSI, MACD, Bollinger, etc.)
2. **Signaux d'entrée** : Conditions pour ouvrir un trade
3. **Signaux de sortie** : Conditions pour fermer un trade
4. **Gestion du risque** : Stoploss, ROI, trailing stop

**Version d'interface** : `INTERFACE_VERSION = 3` (support short + leverage)

### Pairlists

Système pluggable pour sélectionner les paires de trading :

- **StaticPairList** : Liste statique dans config
- **VolumePairList** : Par volume de trading
- **PercentChangePairList** : Par variation de prix
- **MarketCapPairList** : Par capitalisation
- **AgeFilter**, **PriceFilter**, **SpreadFilter**, etc. : Filtres

Configuration en cascade :
```json
"pairlists": [
    {"method": "VolumePairList", "number_assets": 20},
    {"method": "AgeFilter", "min_days_listed": 10},
    {"method": "PrecisionFilter"},
    {"method": "PriceFilter", "low_price_ratio": 0.01}
]
```

### Protections

Mécanismes de protection contre les pertes :

- **StopLossGuard** : Pause après X stoploss
- **MaxDrawdown** : Pause si drawdown > seuil
- **LowProfitPairs** : Désactive paires non rentables
- **CooldownPeriod** : Délai avant ré-entrée

### Modes de trading

1. **Spot** : Trading au comptant (par défaut)
2. **Margin** : Trading avec marge
3. **Futures** : Contrats à terme (leverage)

### Dry-run vs Live

- **Dry-run** : Simulation sans ordres réels (wallet virtuel 1000 USDT)
- **Live** : Trading réel avec argent réel

---

## Développement et contribution

### Workflow de développement

1. **Fork** et clone du repository
2. Créer une **branche** depuis `develop` :
   ```bash
   git checkout develop
   git checkout -b feat/ma-nouvelle-fonctionnalite
   ```

3. **Installation environnement dev** :
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -e .
   pip install -r requirements-dev.txt
   ```

4. **Installer pre-commit hooks** :
   ```bash
   pre-commit install
   ```

5. **Développer** avec tests :
   ```bash
   # Lancer tests
   pytest tests/

   # Test spécifique
   pytest tests/test_freqtradebot.py::test_process_maybe_execute_entry

   # Avec couverture
   pytest --cov=freqtrade tests/
   ```

6. **Vérifier qualité** :
   ```bash
   # Linting et formatting
   ruff check .
   ruff format .

   # Type checking
   mypy freqtrade

   # Ou tout en un
   pre-commit run -a
   ```

7. **Commit et push** :
   ```bash
   git add .
   git commit -m "feat: ajouter support pour indicateur XYZ"
   git push origin feat/ma-nouvelle-fonctionnalite
   ```

8. **Pull Request** vers `develop` (JAMAIS vers `stable`)

### Standards de code

#### Style Python

- **PEP 8** via ruff
- **Type hints** partout (vérifiés par mypy)
- **Docstrings** sur toutes les méthodes publiques (format reST)
  ```python
  def my_function(param1: str, param2: int) -> bool:
      """
      Description courte.

      Description longue optionnelle.

      :param param1: Description du paramètre
      :param param2: Description du paramètre
      :return: Description du retour
      :raises ValueError: Quand la valeur est invalide
      """
      pass
  ```

- **Double quotes** pour les docstrings
- **Anglais** pour code, commentaires, commits, PR

#### Tests

- **Couverture** : Nouveaux fichiers doivent avoir > 90% couverture
- **Tests unitaires** : Un test par méthode minimum
- **Fixtures pytest** : Utiliser conftest.py
- **Mocking** : Utiliser pytest-mock pour services externes

### Branches et releases

- **`develop`** : Branche de développement active (peut être instable)
- **`stable`** : Dernière version stable (utilisée en production)
- **`feat/*`** : Branches de fonctionnalités
- **Tags** : Versions sémantiques (ex: v2024.1)

### Process de contribution

1. **Issues** : Toujours vérifier si une issue existe
2. **Nouvelles features** : Ouvrir une issue d'abord ou discuter sur Discord (#dev)
3. **Bug fixes** : Priorité haute, peuvent être mergées rapidement
4. **Documentation** : Bienvenue, idéal pour premiers contributeurs
5. **Labels "good first issue"** : Bon point de départ

### Committers

Devenir **Committer** :
- Contributions régulières de qualité
- Connaissance de la codebase
- Reviews de PR
- Participation communautaire

Devenir **Core Committer** :
- Committer expérimenté
- Accès complet au repository
- Responsabilité sur sécurité (gestion des API keys)

---

## Commandes détaillées

### Installation et configuration

```bash
# Créer répertoire utilisateur
freqtrade create-userdir --userdir user_data/

# Créer nouvelle configuration interactive
freqtrade new-config --config user_data/config.json

# Afficher configuration résolue (avec valeurs par défaut)
freqtrade show-config --config user_data/config.json
```

### Gestion des données

```bash
# Télécharger données historiques
freqtrade download-data \
    --exchange binance \
    --pairs BTC/USDT ETH/USDT \
    --timeframes 5m 1h \
    --days 30

# Lister données disponibles
freqtrade list-data --userdir user_data/

# Convertir format de données
freqtrade convert-data \
    --format-from json \
    --format-to parquet \
    --datadir user_data/data/binance

# Convertir trades en OHLCV
freqtrade trades-to-ohlcv \
    --exchange binance \
    --pairs BTC/USDT \
    --timeframes 1m 5m
```

### Stratégies

```bash
# Créer nouvelle stratégie
freqtrade new-strategy --strategy MyAwesomeStrategy

# Lister stratégies disponibles
freqtrade list-strategies --userdir user_data/

# Mettre à jour stratégie vers nouvelle version
freqtrade strategy-updater \
    --strategy-path user_data/strategies/my_strategy.py
```

### Backtesting

```bash
# Backtest simple
freqtrade backtesting \
    --strategy SampleStrategy \
    --timerange 20230101-20231231

# Backtest avec paramètres avancés
freqtrade backtesting \
    --strategy MyStrategy \
    --timerange 20230101-20231231 \
    --timeframe 5m \
    --stake-amount 100 \
    --enable-position-stacking \
    --max-open-trades 5 \
    --breakdown day

# Afficher résultats précédents
freqtrade backtesting-show

# Analyse détaillée des résultats
freqtrade backtesting-analysis \
    --analysis-groups 0 1 2 \
    --enter-reason-list
```

### Hyperoptimisation

```bash
# Lancer hyperopt (500 epochs)
freqtrade hyperopt \
    --strategy MyStrategy \
    --hyperopt-loss SharpeHyperOptLoss \
    --epochs 500 \
    --spaces buy sell roi stoploss

# Lister résultats hyperopt
freqtrade hyperopt-list --best 10

# Afficher détails d'une epoch
freqtrade hyperopt-show -n 42
```

### Analyses avancées

```bash
# Détecter lookahead bias
freqtrade lookahead-analysis \
    --strategy MyStrategy

# Détecter problèmes récursifs
freqtrade recursive-analysis \
    --strategy MyStrategy
```

### Trading live

```bash
# Dry-run (simulation)
freqtrade trade \
    --strategy MyStrategy \
    --config user_data/config.json \
    --dry-run

# Live trading (ATTENTION!)
freqtrade trade \
    --strategy MyStrategy \
    --config user_data/config.json
```

### API et interface web

```bash
# Installer FreqUI
freqtrade install-ui

# Lancer serveur web
freqtrade webserver --config user_data/config.json

# API accessible sur http://127.0.0.1:8080
```

### Utilitaires

```bash
# Lister exchanges supportés
freqtrade list-exchanges

# Lister paires sur un exchange
freqtrade list-pairs --exchange binance --quote USDT

# Lister timeframes disponibles
freqtrade list-timeframes --exchange binance

# Tester configuration pairlist
freqtrade test-pairlist --config user_data/config.json

# Afficher trades de la DB
freqtrade show-trades --db-url sqlite:///tradesv3.sqlite

# Plotting
freqtrade plot-dataframe \
    --strategy MyStrategy \
    --pairs BTC/USDT \
    --indicators1 ema50 ema200 \
    --indicators2 rsi

freqtrade plot-profit \
    --trade-source file \
    --exportfilename user_data/backtest_results/backtest-result.json
```

---

## Configuration

### Fichier de configuration (`config.json`)

Structure minimale :
```json
{
    "max_open_trades": 3,
    "stake_currency": "USDT",
    "stake_amount": 100,
    "tradable_balance_ratio": 0.99,
    "fiat_display_currency": "USD",
    "dry_run": true,
    "cancel_open_orders_on_exit": false,

    "exchange": {
        "name": "binance",
        "key": "your_api_key",
        "secret": "your_api_secret",
        "ccxt_config": {},
        "ccxt_async_config": {},
        "pair_whitelist": ["BTC/USDT", "ETH/USDT"],
        "pair_blacklist": ["BNB/.*"]
    },

    "entry_pricing": {
        "price_side": "same",
        "use_order_book": true,
        "order_book_top": 1,
        "check_depth_of_market": {
            "enabled": false,
            "bids_to_ask_delta": 1
        }
    },

    "exit_pricing": {
        "price_side": "same",
        "use_order_book": true,
        "order_book_top": 1
    },

    "pairlists": [
        {"method": "StaticPairList"}
    ],

    "telegram": {
        "enabled": true,
        "token": "your_telegram_token",
        "chat_id": "your_chat_id"
    },

    "api_server": {
        "enabled": true,
        "listen_ip_address": "127.0.0.1",
        "listen_port": 8080,
        "username": "user",
        "password": "pass"
    }
}
```

### Configuration avancée

**Trading avec leverage** :
```json
{
    "trading_mode": "futures",
    "margin_mode": "isolated",
    "liquidation_buffer": 0.05
}
```

**Protections** :
```json
{
    "protections": [
        {
            "method": "StopLossGuard",
            "lookback_period_candles": 60,
            "trade_limit": 4,
            "stop_duration_candles": 20,
            "required_profit": 0.0
        },
        {
            "method": "MaxDrawdown",
            "lookback_period_candles": 200,
            "trade_limit": 20,
            "stop_duration_candles": 10,
            "max_allowed_drawdown": 0.2
        }
    ]
}
```

**FreqAI** :
```json
{
    "freqai": {
        "enabled": true,
        "purge_old_models": 2,
        "train_period_days": 30,
        "backtest_period_days": 7,
        "identifier": "unique_id",
        "feature_parameters": {
            "include_timeframes": ["5m", "15m", "1h"],
            "include_corr_pairlist": ["BTC/USDT", "ETH/USDT"],
            "label_period_candles": 20,
            "include_shifted_candles": 2,
            "DI_threshold": 0.9
        },
        "data_split_parameters": {
            "test_size": 0.33,
            "random_state": 1
        },
        "model_training_parameters": {
            "n_estimators": 1000
        }
    }
}
```

---

## API et RPC

### API REST

**Base URL** : `http://localhost:8080/api/v1`

**Authentification** : Basic Auth ou JWT

#### Endpoints principaux

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/ping` | GET | Vérifier si API répond |
| `/start` | POST | Démarrer le bot |
| `/stop` | POST | Arrêter le bot |
| `/status` | GET | Status des trades ouverts |
| `/balance` | GET | Balance du wallet |
| `/performance` | GET | Performance par paire |
| `/profit` | GET | Profit/loss global |
| `/trades` | GET | Historique des trades |
| `/trades/<trade_id>` | GET/DELETE | Trade spécifique |
| `/whitelist` | GET | Whitelist actuelle |
| `/blacklist` | GET/POST | Gérer blacklist |
| `/forcebuy` | POST | Forcer achat |
| `/forceexit` | POST | Forcer vente |
| `/strategies` | GET | Lister stratégies |
| `/strategy/<strategy>` | GET | Détails stratégie |
| `/available_pairs` | GET | Paires disponibles |
| `/pair_candles` | GET | Données OHLCV |
| `/sysinfo` | GET | Informations système |

#### Exemple d'utilisation (curl)

```bash
# Obtenir status
curl -X GET http://localhost:8080/api/v1/status \
  -u user:pass

# Forcer vente d'un trade
curl -X POST http://localhost:8080/api/v1/forceexit \
  -u user:pass \
  -H "Content-Type: application/json" \
  -d '{"tradeid": 123}'

# Ajouter à blacklist
curl -X POST http://localhost:8080/api/v1/blacklist \
  -u user:pass \
  -H "Content-Type: application/json" \
  -d '{"blacklist": ["BTC/USDT"]}'
```

### WebSocket

**URL** : `ws://localhost:8080/api/v1/ws`

Channels disponibles :
- `whitelist` : Mises à jour de la whitelist
- `analyzed_df` : Dataframes analysés
- `trades` : Trades ouverts/fermés
- `candles` : Streaming de chandeliers

### Telegram

Commandes principales :
- `/start` : Démarrer le bot
- `/stop` : Arrêter le bot (trades restent ouverts)
- `/stopbuy` : Arrêter les nouvelles entrées
- `/status` : Lister trades ouverts
- `/profit [n]` : Profit sur n derniers jours
- `/balance` : Balance actuelle
- `/daily [n]` : Profit quotidien
- `/performance` : Performance par paire
- `/forceexit <id>|all` : Forcer sortie
- `/forcelong <pair> [rate]` : Forcer entrée long
- `/forceshort <pair> [rate]` : Forcer entrée short
- `/delete <id>` : Supprimer trade de DB
- `/whitelist` : Voir whitelist
- `/blacklist [pair]` : Gérer blacklist
- `/edge` : Afficher edge (si activé)
- `/help` : Aide
- `/version` : Version

---

## Tests et qualité

### Lancer les tests

```bash
# Tous les tests
pytest

# Avec couverture
pytest --cov=freqtrade --cov-report=html

# Tests spécifiques
pytest tests/test_freqtradebot.py

# Parallélisation (plus rapide)
pytest -n auto

# Verbose
pytest -vv

# Arrêt au premier échec
pytest -x

# Voir print statements
pytest -s
```

### Structure des tests

```
tests/
├── conftest.py               # Fixtures globales
├── test_freqtradebot.py      # Tests du bot principal
├── test_wallets.py           # Tests wallets
├── commands/                 # Tests commandes CLI
├── data/                     # Tests gestion données
├── exchange/                 # Tests exchanges
├── optimize/                 # Tests backtesting/hyperopt
├── rpc/                      # Tests RPC/API
├── strategy/                 # Tests stratégies
└── plugins/                  # Tests plugins
```

### Fixtures utiles

```python
def test_something(default_conf, mocker, caplog):
    """
    - default_conf: Configuration par défaut
    - mocker: pytest-mock pour mocking
    - caplog: Capture des logs
    """
    pass
```

### Linting et formatage

```bash
# Ruff (linter + formatter)
ruff check .               # Vérifier
ruff check --fix .         # Auto-fix
ruff format .              # Formatter

# Mypy (type checking)
mypy freqtrade

# Pre-commit (tout en un)
pre-commit run -a
```

---

## Ressources

### Documentation officielle

- **Site principal** : [www.freqtrade.io](https://www.freqtrade.io)
- **Documentation** : [docs.freqtrade.io](https://www.freqtrade.io/en/stable/)
- **API Docs** : [docs.freqtrade.io/rest-api](https://www.freqtrade.io/en/stable/rest-api/)
- **Stratégie 101** : [docs.freqtrade.io/strategy-101](https://www.freqtrade.io/en/stable/strategy-101/)

### Communauté

- **Discord** : [discord.gg/p7nuUNVfP7](https://discord.gg/p7nuUNVfP7)
  - `#general` : Discussion générale
  - `#strategy-development` : Développement de stratégies
  - `#dev` : Développement du bot
  - `#freqai` : Machine learning
  - `#support` : Support technique

- **GitHub** :
  - Repository : [github.com/freqtrade/freqtrade](https://github.com/freqtrade/freqtrade)
  - Issues : [github.com/freqtrade/freqtrade/issues](https://github.com/freqtrade/freqtrade/issues)
  - Discussions : [github.com/freqtrade/freqtrade/discussions](https://github.com/freqtrade/freqtrade/discussions)

### Exchanges supportés

#### Spot Trading (Stable)
- [Binance](https://www.binance.com/)
- [BingX](https://bingx.com/)
- [Bitget](https://www.bitget.com/)
- [Bitmart](https://bitmart.com/)
- [Bybit](https://bybit.com/)
- [Gate.io](https://www.gate.io/)
- [HTX](https://www.htx.com/)
- [Hyperliquid](https://hyperliquid.xyz/) (DEX)
- [Kraken](https://kraken.com/)
- [OKX](https://okx.com/)

#### Futures Trading (Expérimental)
- Binance, Bitget, Gate.io, Hyperliquid, OKX, Bybit

**Note** : Vérifier [docs/exchanges.md](docs/exchanges.md) pour configurations spécifiques

### Configuration système

#### Hardware minimum
- **RAM** : 2GB (4GB+ recommandé pour FreqAI)
- **Disque** : 1GB (plus pour données historiques)
- **CPU** : 2 vCPU (plus pour backtesting parallèle)
- **Horloge** : Synchronisation NTP critique!

#### Software requis
- **Python** : 3.11, 3.12 ou 3.13
- **pip** : Gestionnaire de paquets Python
- **git** : Contrôle de version
- **TA-Lib** : Bibliothèque d'analyse technique
  ```bash
  # Ubuntu/Debian
  sudo apt-get install libta-lib0-dev

  # macOS
  brew install ta-lib

  # Windows
  # Télécharger depuis https://github.com/cgohlke/talib-build/releases
  ```
- **virtualenv** (recommandé) : Environnements isolés
- **Docker** (recommandé) : Déploiement simplifié

### Démarrage rapide

#### Installation native

```bash
# Cloner repository
git clone https://github.com/freqtrade/freqtrade.git
cd freqtrade

# Créer environnement virtuel
python3 -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .venv\Scripts\activate   # Windows

# Installer
pip install -e .

# Créer configuration
freqtrade create-userdir --userdir user_data
freqtrade new-config --config user_data/config.json

# Télécharger données
freqtrade download-data --exchange binance --pairs BTC/USDT ETH/USDT --days 30

# Lancer en dry-run
freqtrade trade --strategy SampleStrategy --config user_data/config.json
```

#### Installation Docker

```bash
# Créer structure
mkdir ft_userdata
cd ft_userdata

# Télécharger docker-compose
curl https://raw.githubusercontent.com/freqtrade/freqtrade/stable/docker-compose.yml -o docker-compose.yml

# Créer config
docker compose run --rm freqtrade create-userdir --userdir user_data
docker compose run --rm freqtrade new-config --config user_data/config.json

# Lancer
docker compose up -d
```

### Liens utiles

- **Publication scientifique** : [JOSS Paper](https://doi.org/10.21105/joss.04864)
- **TA-Lib Documentation** : [ta-lib.org](https://ta-lib.org/)
- **ccxt** : [github.com/ccxt/ccxt](https://github.com/ccxt/ccxt)
- **Pandas** : [pandas.pydata.org](https://pandas.pydata.org/)
- **Stratégies communautaires** : [github.com/freqtrade/freqtrade-strategies](https://github.com/freqtrade/freqtrade-strategies)

---

## Glossaire

- **Backtesting** : Test d'une stratégie sur données historiques
- **Dry-run** : Mode simulation sans argent réel
- **Hyperopt** : Optimisation automatique de paramètres
- **Pairlist** : Liste de paires de trading
- **ROI** : Return On Investment (retour sur investissement)
- **Stake amount** : Montant investi par trade
- **Stoploss** : Limite de perte par trade
- **Timeframe** : Période de chandelier (1m, 5m, 1h, etc.)
- **Trade** : Position ouverte ou fermée
- **Whitelist** : Paires autorisées au trading
- **Blacklist** : Paires exclues du trading

---

**Version** : Ce document correspond à Freqtrade v2024.x
**Dernière mise à jour** : 2025-01-18
**Auteur** : Documentation générée pour Claude AI Assistant
