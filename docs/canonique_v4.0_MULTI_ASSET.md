START
# 📘 **CANONIQUE v4.0 - MULTI-ASSET FRAMEWORK**

## EXTENSION DU SYSTÈME QUANT-ENGINE / OPENCLAW

**Date** : 2026-02-28  
**Auteur** : CTO  
**Statut** : SPÉCIFICATION FIGÉE - VALIDÉE POUR DÉVELOPPEMENT

---

## 📋 **TABLE DES MATIÈRES**

1. [Principes fondateurs](#1-principes-fondateurs)
2. [Architecture générale](#2-architecture-générale)
3. [Pipeline data multi-actifs](#3-pipeline-data-multi-actifs)
4. [BaseEngine - Classe abstraite](#4-baseengine---classe-abstraite)
5. [Moteurs par actif](#5-moteurs-par-actif)
6. [Backtesting multi-actifs (OpenClaw)](#6-backtesting-multi-actifs-openclaw)
7. [Exécution live & Risk Management](#7-exécution-live--risk-management)
8. [Dashboard & Alertes](#8-dashboard--alertes)
9. [Plan de développement](#9-plan-de-développement)
10. [Métriques de succès](#10-métriques-de-succès)
11. [Annexes](#11-annexes)

---

## 1. PRINCIPES FONDATEURS

### 1.1 Héritage du Canonique
| Principe | Application multi-actifs |
|----------|-------------------------|
| Un actif ≠ un autre | Chaque actif a son propre moteur d'analyse |
| Pas de forcing | Aucune variable imposée d'un actif à l'autre |
| Backtestable or it doesn't exist | 100% des stratégies backtestables |
| Live prime sur historique | Altcoins uniquement (P&D) |
| Backtest prime sur live | BTC & Gold (approche classique) |
| HTF = contexte | Jamais filtre bloquant |
| Chaque actif a son contrat d'output | Métriques, délais, fréquence spécifiques |

### 1.2 Gouvernance
- **CTO** : Valide l'architecture multi-actifs, les conventions, les thresholds
- **CEO/Owner** : Valide la vision et l'allocation des ressources entre actifs

---

## 2. ARCHITECTURE GÉNÉRALE

### 2.1 Structure des dossiers

quant-engine/
│
├── engine/
│ ├── core/ # Mutualisé TOUS actifs
│ │ ├── data/ # Pipeline data unifié
│ │ ├── backtest/ # Moteur backtest générique
│ │ ├── registry/ # Registry multi-actifs
│ │ └── utils/ # Conventions, time, logging
│ │
│ ├── assets/ # Dossier des moteurs par actif
│ │ ├── altcoins/ # Moteur existant (Phase 2 v1.1)
│ │ │ ├── altcoin_screener.py
│ │ │ ├── altcoin_features.py
│ │ │ └── run_altcoin.py
│ │ │
│ │ ├── btc/ # Moteur Bitcoin
│ │ │ ├── btc_screener.py
│ │ │ ├── btc_features.py
│ │ │ ├── btc_strategies.py
│ │ │ └── run_btc.py
│ │ │
│ │ └── gold/ # Moteur Gold
│ │ ├── gold_screener.py
│ │ ├── gold_features.py
│ │ ├── gold_strategies.py
│ │ └── run_gold.py
│ │
│ └── orchestrator/ # Optionnel (Phase 3+)
│ ├── schedule_manager.py
│ └── multi_asset_dashboard.py
│
├── datasets/
│ ├── raw/ # Données brutes par actif
│ │ ├── ccxt/ # Altcoins (existant)
│ │ ├── btc/ # BTC
│ │ │ ├── 1m.parquet
│ │ │ ├── 1h.parquet
│ │ │ └── onchain/ # Optionnel
│ │ │
│ │ └── gold/ # Gold
│ │ ├── 1m.parquet
│ │ ├── 1h.parquet
│ │ ├── 4h.parquet
│ │ └── daily.parquet
│ │
│ ├── features/ # Features pré-calculées par actif
│ ├── events/ # Registry multi-actifs
│ └── backtests/ # Résultats de backtest
│
├── openclaw/ # Offline learning (multi-actifs)
│ ├── backtest_runner.py
│ ├── variable_importance.py
│ ├── strategy_optimizer.py
│ └── reports/
│
└── execution/ # Phase 3
├── risk_manager.py
├── order_executor.py
└── telegram_alerts.py


### 2.2 Règles d'interaction
- **Altcoins → BTC** : Influence possible (corrélation), pas de dépendance
- **BTC → Altcoins** : Contexte uniquement (ex: BTC haussier = favorable altcoins)
- **Gold ↔ BTC** : Faible corrélation directe
- **Cross-asset trading** : Interdit en Phase 1 multi-actifs

---

## 3. PIPELINE DATA MULTI-ACTIFS

### 3.1 Sources par actif

| Actif | Source principale | Timeframes | Sources secondaires |
|-------|------------------|------------|---------------------|
| **Altcoins** | CCXT (Bybit) | 1m, 1h, 4h | - |
| **BTC** | CCXT (Bybit/Binance) | 1m, 5m, 1h, 4h, 1d | Funding rates, Open Interest |
| **Gold** | **yfinance** (historique) + **Oanda** (live) | 1m, 5m, 1h, 4h, 1d | DXY, Taux réels (TIPS) |

### 3.2 Spécification Gold - Source de données

```yaml
# config/gold_data.yaml
historical:
  primary: "yfinance"
  symbol: "GC=F"              # Gold futures
  timeframes: 
    1m: true                  # Nécessite aggregation
    5m: true
    1h: true
    4h: true
    1d: true
  backup: "dukascopy"         # Fallback si yfinance indisponible

live:
  primary: "oanda"
  api_key: "${OANDA_API_KEY}"
  account_id: "${OANDA_ACCOUNT_ID}"
  environment: "practice"      # "practice" ou "live"
  instruments: ["XAU_USD"]
  
macro_data:
  dxy: "DX-Y.NYB"              # Dollar Index via yfinance
  tips_10y: "^TIP"             # TIPS 10 ans via yfinance
  usd_rates: "US10Y"           # Taux US 10 ans

3.3 Conventions de nommage (extension)
Format	Exemple
Interne	BTCUSDT, XAUUSD
Storage raw	datasets/raw/btc/1m.parquet
Storage raw	datasets/raw/gold/1h.parquet
CCXT	BTC/USDT:USDT
Yahoo Finance	GC=F, DX-Y.NYB
3.4 Règle CTO
Un actif sans données brutes validées n'existe pas dans le système.
Gold nécessite une validation spécifique des sources avant tout développement.

4. BASEENGINE - CLASSE ABSTRAITE
4.1 Interface obligatoire
python
# engine/assets/base_engine.py
from abc import ABC, abstractmethod
import pandas as pd
from typing import Dict, List, Optional, Tuple

class BaseAssetEngine(ABC):
    """
    Classe de base pour tous les moteurs d'actifs.
    Toute implémentation doit respecter ce contrat.
    """
    
    def __init__(self, asset_name: str):
        self.asset = asset_name
        self.data: Dict[str, pd.DataFrame] = {}
        self.features: Dict[str, pd.Series] = {}
        self.scores: Dict[str, float] = {}
        self.events: List[Dict] = []
    
    @abstractmethod
    def load_data(self, 
                  timeframe: str, 
                  start_date: str, 
                  end_date: str) -> bool:
        """
        Charge les données depuis le dataset raw.
        Retourne True si succès, False sinon.
        """
        pass
    
    @abstractmethod
    def compute_features(self) -> Dict[str, pd.Series]:
        """
        Calcule les features spécifiques à l'actif.
        Retourne un dictionnaire {nom_feature: série}
        """
        pass
    
    @abstractmethod
    def compute_score(self) -> float:
        """
        Produit un score normalisé.
        Altcoins: EventScore (0-100)
        BTC: TrendScore (-100 à +100)
        Gold: MacroScore (0-100)
        """
        pass
    
    @abstractmethod
    def generate_events(self) -> List[Dict]:
        """
        Génère les événements pour le registry.
        Format: {timestamp, type, score, metadata}
        """
        pass
    
    @abstractmethod
    def backtest_strategy(self, 
                          strategy_params: Dict,
                          start_date: str,
                          end_date: str) -> Dict:
        """
        Lance un backtest sur une stratégie spécifique.
        Retourne les métriques de performance.
        """
        pass
    
    def validate_data(self) -> bool:
        """Vérifie l'intégrité des données chargées."""
        if not self.data:
            return False
        for tf, df in self.data.items():
            if df.empty:
                return False
            if 'timestamp' not in df.columns:
                return False
        return True
4.2 Contrat de sortie par actif
Actif	Score	Plage	Interprétation
Altcoins	EventScore	0-100	Probabilité de pump
BTC	TrendScore	-100 à +100	Direction (-) / (+) + force
Gold	MacroScore	0-100	Confiance dans configuration
5. MOTEURS PAR ACTIF
5.1 Altcoins (existant - Phase 2)
Objectif : Détection P&D

Timeframe critique : 1m avec confirmation 5m

Variables clés : Compression, Ignition, CHOCH, EMA position

HTF : Contexte (jamais veto)

5.2 Bitcoin (BTC)
python
# engine/assets/btc/btc_features.py
class BTCFeatures:
    """
    Variables spécifiques Bitcoin
    """
    @staticmethod
    def funding_impact(funding_rates: pd.Series) -> pd.Series:
        """Impact du funding rate sur le prix"""
        pass
    
    @staticmethod
    def cvd_divergence(price: pd.Series, cvd: pd.Series) -> pd.Series:
        """Divergence prix / CVD"""
        pass
    
    @staticmethod
    def ema_trend(price: pd.Series, periods: List[int] = [9, 21, 50, 200]) -> Dict:
        """Structure des EMA"""
        pass
    
    @staticmethod
    def range_breakout(price: pd.Series, lookback: int = 20) -> pd.Series:
        """Détection de cassure de range"""
        pass
Hypothèses de trading BTC :

BTC a une mémoire (trend persistence 3-7 jours)

Funding rates sont mean-reverting

Les breakouts de range sont significatifs

Volume confirme la force du mouvement

5.3 Gold (XAUUSD)
python
# engine/assets/gold/gold_features.py
class GoldFeatures:
    """
    Variables spécifiques Gold
    """
    @staticmethod
    def real_rates_correlation(price: pd.Series, tips_10y: pd.Series) -> float:
        """Corrélation avec taux réels"""
        pass
    
    @staticmethod
    def dxy_impact(price: pd.Series, dxy: pd.Series, lag: int = 3) -> pd.Series:
        """Impact décalé du DXY"""
        pass
    
    @staticmethod
    def macro_event_impact(calendar: pd.DataFrame) -> pd.Series:
        """Impact des événements macro (FED, NFP, CPI)"""
        pass
    
    @staticmethod
    def technical_zones(price: pd.Series, levels: List[float] = [1800, 1900, 2000]) -> Dict:
        """Détection de zones techniques"""
        pass
Hypothèses de trading Gold :

Gold corrélé négativement aux taux réels (TIPS)

Les niveaux psychologiques (1800, 1900, 2000) sont importants

Le DXY a une influence avec délai de 1-3 jours

Les annonces FED créent des breakouts durables

6. BACKTESTING MULTI-ACTIFS (OPENCLAW)
6.1 Métriques communes
Métrique	Formule	Applicable
Sharpe ratio	mean(returns)/std(returns)*sqrt(252)	Tous
Sortino ratio	mean(returns)/std(negative_returns)*sqrt(252)	Tous
Win rate	wins / total_trades	Tous
Profit factor	gross_profit / gross_loss	Tous
Max drawdown	max(peak - trough)	Tous
Expectancy	avg_win * win_rate - avg_loss * (1-win_rate)	Tous
6.2 Métriques spécifiques
Actif	Métrique	Calcul
Altcoins	Time-to-peak	Minutes entre entrée et pic
MFE/MAE ratio	max_favorable_excursion / max_adverse_excursion
Event survival rate	events_confirmes / events_detectes
BTC	Trend duration	Heures/jours de trend continue
Pullback depth	Retracement % pendant trend
Funding impact	PnL ajusté du funding
Gold	Macro alignment	% du temps aligné avec macro
Hold time distribution	Distribution des durées de position
Session performance	Performance par session (Asie, Londres, NY)
6.3 Walk-forward validation
python
class WalkForwardValidator:
    """
    Validation walk-forward pour chaque actif
    """
    def __init__(self, n_splits: int = 5):
        self.n_splits = n_splits
    
    def validate(self, 
                 asset_engine: BaseAssetEngine,
                 strategy_params: Dict,
                 full_date_range: Tuple[str, str]) -> Dict:
        """
        Valide une stratégie sur périodes glissantes
        
        Returns:
            stability_score: ratio test_sharpe / train_sharpe
            min_stability > 0.7 requis pour validation
        """
        pass
6.4 Registre des hypothèses
markdown
# docs/asset_hypotheses.md

## Altcoins
- H1: Les pumps sont des événements rares mais asymétriques
- H2: Une phase de compression précède 80% des pumps
- H3: Le volume confirme la légitimité du mouvement

## Bitcoin
- H1: Les trends ont une persistance de 3-7 jours
- H2: Les funding rates extrêmes précèdent les retournements
- H3: Les ranges de 20+ bougies sont significatifs

## Gold
- H1: Les décisions FED créent des breakouts durables
- H2: La corrélation avec les taux réels est stable > 0.7
- H3: Les niveaux psychologiques agissent comme support/résistance
7. EXÉCUTION LIVE & RISK MANAGEMENT
7.1 Paramètres par actif
Paramètre	Altcoins	BTC	Gold
Taille position	% du capital / ATR(20)	% fixe (1-2%)	% du capital / ATR(20)
Stop loss	ATR(14) × 1.5	Swing low/high (structurel)	ATR(14) × 1.5
Take profit	10% fixe	Trailing (activation 1.5%, distance 1.0%)	Structurel (support/résistance)
Pyramiding	Non	Max 2 positions	Non
Max DD journalier	3%	2%	1.5%
Max trades/jour	5	2	2
7.2 Execution Engine unifié
python
# execution/order_executor.py
class OrderExecutor:
    """
    Exécution unifiée pour tous les actifs
    """
    def __init__(self):
        self.connections = {}  # Une connexion par broker
        self.risk_manager = RiskManager()
    
    def execute(self, signal: Dict) -> Dict:
        """
        Exécute un signal (vérification risque + ordre)
        signal = {
            'asset': 'btc',
            'direction': 'long',
            'size': 0.01,
            'entry_price': 65000,
            'sl': 64000,
            'tp': 67000
        }
        """
        # Vérification risk management
        if not self.risk_manager.check(signal):
            return {'status': 'rejected', 'reason': 'risk_limit'}
        
        # Exécution selon l'actif
        if signal['asset'] == 'btc':
            return self._execute_btc(signal)
        elif signal['asset'] == 'gold':
            return self._execute_gold(signal)
        else:
            return self._execute_altcoin(signal)
8. DASHBOARD & ALERTES
8.1 Canaux Telegram
Canal	Contenu	Fréquence
#altcoins-signals	Top opportunités P&D	Temps réel
#btc-signals	Signaux trend BTC	Horaire
#gold-signals	Configurations Gold	Quotidien
#risk-alerts	Drawdown, limites atteintes	Temps réel
#system-alerts	Erreurs techniques, reconnexions	Temps réel
8.2 Dashboard multi-actifs
python
# orchestrator/multi_asset_dashboard.py
class MultiAssetDashboard:
    """
    Vue consolidée de tous les actifs
    """
    def get_summary(self) -> Dict:
        return {
            'altcoins': {
                'top_opportunities': [...],
                'active_events': 3,
                'daily_pnl': 124.50
            },
            'btc': {
                'trend': 'bullish',
                'strength': 78,
                'position': 'long_0.5%'
            },
            'gold': {
                'macro_score': 65,
                'setup': 'breakout_1900',
                'position': 'none'
            }
        }
9. PLAN DE DÉVELOPPEMENT
Phase 1 (4 semaines) — Infrastructure multi-actifs
Semaine	Module	Livrable
S1	Source Gold	Implémentation yfinance + tests
S2	Data BTC	Funding rates, Open Interest
S3	Structure assets/	BaseEngine + dossiers
S4	Validation data	Tests intégrité
Phase 2 (6 semaines) — Moteurs BTC & Gold v1
Semaine	Module	Livrable
S5	BTC features v1	EMA, RSI, Volume
S6	BTC screener v1	Signaux simples
S7	BTC backtest	6 mois de données
S8	Gold features v1	DXY, Taux, Macro
S9	Gold screener v1	Confluence macro/technique
S10	Gold backtest	6 mois de données
Phase 3 (4 semaines) — OpenClaw multi-actifs
Semaine	Module	Livrable
S11	Backtest runner multi	Adaptation OpenClaw
S12	Variable importance BTC	Analyse contribution
S13	Variable importance Gold	Analyse contribution
S14	Rapports comparatifs	Documentation
Phase 4 (6 semaines) — Optimisation & Live
Semaine	Module	Livrable
S15-16	Optimisation BTC	50 générations
S17-18	Optimisation Gold	50 générations
S19	Risk Manager multi	Implémentation
S20	Execution Engine	Unifié
S21	Dashboard v1	Streamlit
S22	Alertes Telegram	Tous canaux
10. MÉTRIQUES DE SUCCÈS
Métrique	Objectif	Mesure
Extensibilité	< 2 semaines	Temps ajout nouvel actif
Isolation	Bug isolé	Impact limité à 1 actif
Backtestabilité	100%	Toute stratégie backtestable
Observabilité	Dashboard	Métriques visibles par actif
Robustesse	Walk-forward > 0.7	Stabilité test/train
Couverture data	99%	Disponibilité sources
Latence exécution	< 1s	Signal → ordre
11. ANNEXES
11.1 Glossaire
P&D : Pump & Dump

CVD : Cumulative Volume Delta

DXY : Dollar Index

TIPS : Treasury Inflation-Protected Securities

HTF : Higher Timeframe

11.2 Références
Document canonique v1.2 (Phase 2)

Document canonique v3.0 (Phase 3)

Mission OpenClaw

Session 28-02-2026

11.3 Prochaines étapes
✅ CE DOCUMENT - Canonique v4.0 validée

🔜 Analyse fondamentale BTC & Gold

🔜 Décomposition Phase 1 en micro-étapes

*Document maintenu par le CTO - Extension multi-actifs - 28 février 2026* 🚀
END
