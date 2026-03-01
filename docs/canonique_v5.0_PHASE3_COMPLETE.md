📘 CANONIQUE v5.0 - PHASE 3 MULTI-ASSET COMPLÈTE
TRADING 1M → EXTENSION TIMEFRAMES → HYPEROPTIMISATION OPENCLAW
Date : 2026-03-01
Auteur : CTO
Statut : SPÉCIFICATION FIGÉE - VALIDÉE POUR DÉVELOPPEMENT

📋 TABLE DES MATIÈRES
Vision stratégique

Infrastructure 1M multi-actifs

Extension multi-timeframes

Familles SL/TP & gestion risque

Sauvegarde des runs & analyse comparative

Hyperoptimisation OpenClaw

Architecture finale

Plan de développement détaillé

Métriques de succès

Annexes

1. VISION STRATÉGIQUE
1.1 Principes fondateurs
Principe	Application
1M d'abord	Tous les actifs doivent pouvoir trader en 1m
Extension progressive	5m → 15m → 1h → 4h par la suite
Sauvegarde systématique	Chaque run est persistant et comparable
Familles SL/TP universelles	5 familles implémentées pour tous
OpenClaw multi-actifs	Optimisation séparée par actif
1.2 Gouvernance
CTO : Valide l'architecture, les conventions, les thresholds

CEO : Valide la vision et l'allocation des ressources

2. INFRASTRUCTURE 1M MULTI-ACTIFS
2.1 Sources de données 1m par actif
Actif	Source 1m	Statut	Problème
Altcoins	CCXT (Bybit)	✅ OK	-
BTC	CCXT (Bybit)	✅ OK	-
Gold	À DÉFINIR	⚠️ Critique	Yahoo ne fournit pas 1m
2.2 Solutions pour Gold 1m
Option	Source	Implémentation	Complexité
A	Dukascopy	API REST	Moyenne
B	Oanda	API v20	Faible
C	MT5 bridge	Python-MT5	Élevée
D	Synthèse ticks	Calcul depuis ticks 1s	Élevée
Décision : À arbitrer en début de Phase 3.1

2.3 Format de stockage unifié
text
datasets/raw/[actif]/[timeframe].parquet
datasets/raw/btc/1m.parquet
datasets/raw/gold/1m.parquet
datasets/raw/altcoins/AAVEUSDT/1m.parquet
Contrat :

Index datetime (timezone-naive, UTC)

Colonnes: open, high, low, close, volume

Pas de NaN sur les colonnes critiques

3. EXTENSION MULTI-TIMEFRAMES
3.1 Timeframes supportées par actif
Actif	1m	5m	15m	1h	4h	1d
Altcoins	✅	⏳	⏳	✅	✅	✅
BTC	✅	⏳	⏳	✅	✅	✅
Gold	⏳	⏳	⏳	✅	✅	✅
3.2 Architecture multi-timeframe
python
# Dans chaque AssetEngine
def load_data(self, timeframes: List[str], start_date: str, end_date: str):
    """Charge plusieurs timeframes simultanément"""
    for tf in timeframes:
        filename = f"{self.data_path}{self.asset}_{tf}.parquet"
        df = pd.read_parquet(filename)
        self.data[tf] = df

def compute_features(self, timeframe: str = "1m"):
    """Calcule features sur un timeframe spécifique"""
    df = self.data[timeframe]
    # ... calculs
3.3 Règles de confirmation inter-TF
Règle	Description
R1	Signal 1m doit être confirmé par 5m
R2	Tendance 1h donne le contexte
R3	Pas de trade contre la tendance 4h
4. FAMILLES SL/TP & GESTION RISQUE
4.1 Les 5 familles universelles
F1 - Fixe
python
sl_pct = 2.0      # Stop loss fixe 2%
tp_pct = 6.0      # Take profit fixe 6%
ratio = tp_pct / sl_pct  # 1:3
F2 - ATR Dynamique
python
sl_atr = 1.5      # Stop loss = 1.5 × ATR(14)
tp_atr = 4.0      # Take profit = 4.0 × ATR(14)
atr = compute_atr(14)
sl_price = entry_price - (atr * sl_atr)
tp_price = entry_price + (atr * tp_atr)
F3 - Trailing Stop
python
activation_pct = 3.0    # S'active après +3%
trail_distance = 1.5    # Trail à 1.5% du plus haut
step_pct = 0.5          # Réactivation tous les 0.5%
F4 - Mixte (TP partiel + Trailing)
python
tp_partial_pct = 3.0    # Premier TP à +3%
partial_size = 0.5      # 50% de la position
trail_remainder = True  # Trail sur le reste
activation_pct = 2.5    # Activation trail après +2.5%
F5 - Structurel
python
# Basé sur niveaux techniques
support = previous_swing_low
resistance = previous_swing_high
sl_price = support - buffer
tp_price = resistance
4.2 Fees & Slippage unifiés
yaml
fees:
  bybit:
    maker: 0.001    # 0.1%
    taker: 0.001    # 0.1%
  oanda:            # Pour Gold
    commission: 0.0005  # 0.05%
    
slippage:
  model: "dynamic"  # Basé sur volatilité
  base_pct: 0.05    # 0.05% de base
  volatility_multiplier: 0.5
  min_pct: 0.02
  max_pct: 0.20
4.3 Intégration dans backtest
python
def apply_fees_and_slippage(self, trade):
    """Applique fees et slippage à un trade"""
    # Slippage à l'entrée
    effective_entry = trade.entry_price * (1 + self.slippage_pct/100)
    
    # Slippage à la sortie
    effective_exit = trade.exit_price * (1 - self.slippage_pct/100)
    
    # Frais (taker sur entrée ET sortie)
    fee_cost = (effective_entry * trade.size * self.fees['taker']) + \
               (effective_exit * trade.size * self.fees['taker'])
    
    trade.net_pnl = trade.gross_pnl - fee_cost
    return trade
5. SAUVEGARDE DES RUNS & ANALYSE COMPARATIVE
5.1 Structure canonique des runs
text
datasets/backtest_runs/
├── YYYY-MM-DD_HHMM_description/
│   ├── config.json                # Paramètres complets
│   ├── trades.parquet              # Tous les trades
│   ├── metrics.csv                  # Métriques agrégées
│   ├── summary.txt                   # Résumé lisible
│   └── metadata.json                 # Environnement, versions
│
├── comparisons/
│   ├── all_runs_metrics.csv         # Comparatif global
│   ├── top10_configs.json            # Top performances
│   └── parameter_importance.csv      # Analyse RF
│
└── reports/
    └── optimisation_report_YYYY-MM-DD.html
5.2 Contrats d'output
Fichier	Format	Contenu obligatoire
config.json	JSON	Actif, timeframes, SL/TP famille, paramètres
trades.parquet	Parquet	timestamp, pair, direction, entry, exit, pnl, fees, slippage
metrics.csv	CSV	total_trades, win_rate, sharpe, profit_factor, max_dd, expectancy
metadata.json	JSON	git_commit, python_version, timestamp
5.3 Métriques de comparaison
Métrique	Formule	Poids comparaison
Sharpe ratio	mean(returns)/std(returns)*sqrt(252)	0.35
Profit factor	gains_total / pertes_total	0.25
Win rate	gagnants / total_trades	0.15
Avg win / avg loss	avg_win / avg_loss	0.15
Max drawdown	max(peak - trough)	-0.10
5.4 Comparateur multi-runs
python
class RunComparator:
    """
    Compare plusieurs runs de backtest
    """
    def __init__(self, runs_path: str = "datasets/backtest_runs/"):
        self.runs_path = runs_path
    
    def load_all_metrics(self) -> pd.DataFrame:
        """Charge toutes les métriques de tous les runs"""
        all_metrics = []
        for run_dir in os.listdir(self.runs_path):
            metrics_file = f"{self.runs_path}/{run_dir}/metrics.csv"
            if os.path.exists(metrics_file):
                df = pd.read_csv(metrics_file)
                df['run_id'] = run_dir
                all_metrics.append(df)
        return pd.concat(all_metrics)
    
    def get_top_configs(self, metric: str = "sharpe", n: int = 10) -> pd.DataFrame:
        """Retourne les top N configurations selon une métrique"""
        df = self.load_all_metrics()
        return df.nlargest(n, metric)
    
    def parameter_importance(self) -> pd.DataFrame:
        """Analyse d'importance des paramètres (Random Forest)"""
        # Implémentation à venir
        pass
6. HYPEROPTIMISATION OPENCLAW
6.1 Espace de recherche par actif
Altcoins
Catégorie	Variables	Plage
Poids familles	compression, ignition, momentum, volume, asymétrie	0.0-1.0
Seuils	entry_threshold (60-90), early_warning (40-70)	Continu
Fenêtres	atr_period (10-20), ema_short (5-20), ema_long (20-50)	Discret
SL/TP	sl_pct (1-3%), tp_pct (3-10%)	Continu
BTC
Catégorie	Variables	Plage
Tendance	ema_short (9-21), ema_long (50-200)	Discret
Momentum	rsi_period (10-20), rsi_threshold (30-70)	Continu
Funding	funding_threshold (0.02-0.1), lookback (8-24h)	Continu
SL/TP	sl_atr (1.0-2.5), tp_atr (2.0-5.0)	Continu
Gold
Catégorie	Variables	Plage
Macro	tips_threshold (1.0-2.5%), dxy_corr_threshold (-0.7 à -0.3)	Continu
Momentum	rsi_period (10-20), macd_fast (12-26), macd_slow (26-52)	Discret
Volume	volume_threshold (1.5-3.0)	Continu
SL/TP	sl_pct (1.0-2.5%), tp_pct (2.0-5.0%)	Continu
6.2 OpenClaw multi-actifs
python
class OpenClawMultiAsset:
    """
    OpenClaw adapté pour l'optimisation multi-actifs
    """
    def __init__(self):
        self.optimizers = {
            'altcoins': GeneticOptimizer(),
            'btc': GeneticOptimizer(),
            'gold': GeneticOptimizer()
        }
        self.results = {}
    
    def run_all(self, generations: int = 50):
        """Lance l'optimisation pour tous les actifs"""
        for asset, optimizer in self.optimizers.items():
            print(f"🚀 Optimisation {asset}...")
            self.results[asset] = optimizer.run(generations)
    
    def get_best_configs(self, n: int = 5) -> Dict:
        """Retourne les top n configs par actif"""
        best = {}
        for asset, results in self.results.items():
            best[asset] = results['top_configs'][:n]
        return best
    
    def generate_report(self):
        """Génère un rapport comparatif multi-actifs"""
        # Implémentation à venir
        pass
6.3 Workflow d'optimisation
text
1. Charger les données 1m pour chaque actif (période 6 mois)
2. Définir l'espace de recherche (JSON par actif)
3. Lancer optimisation génétique (50 générations)
   → Population 100
   → Élite 20%
   → Croisement 70%
   → Mutation 10%
4. Sauvegarder chaque génération
5. À la fin, analyser:
   - Évolution fitness
   - Top 10 configurations
   - Importance des paramètres
6. Générer rapport comparatif
7. Sauvegarder dans openclaw_runs/
7. ARCHITECTURE FINALE
text
quant-engine/
├── engine/
│   ├── core/
│   │   ├── data/
│   │   │   ├── base_loader.py
│   │   │   ├── altcoin_loader.py
│   │   │   ├── btc_loader.py
│   │   │   └── gold_loader.py
│   │   ├── backtest/
│   │   │   ├── engine.py
│   │   │   ├── metrics.py
│   │   │   └── comparator.py
│   │   ├── risk/
│   │   │   ├── sl_tp_families.py
│   │   │   ├── fees.py
│   │   │   └── slippage.py
│   │   └── utils/
│   │
│   ├── assets/
│   │   ├── altcoins/
│   │   │   ├── altcoin_engine.py
│   │   │   └── altcoin_features.py
│   │   ├── btc/
│   │   │   ├── btc_engine.py
│   │   │   └── btc_features.py
│   │   └── gold/
│   │       ├── gold_engine.py
│   │       └── gold_features.py
│   │
│   └── orchestrator/
│
├── openclaw/
│   ├── optimizers/
│   │   ├── genetic_optimizer.py
│   │   └── bayesian_optimizer.py
│   ├── multi_asset.py
│   ├── backtest_runner.py
│   └── reports/
│
├── datasets/
│   ├── raw/
│   │   ├── altcoins/
│   │   ├── btc/
│   │   └── gold/
│   ├── backtest_runs/
│   │   ├── YYYY-MM-DD_HHMM_description/
│   │   └── comparisons/
│   └── openclaw_runs/
│       ├── altcoins/
│       ├── btc/
│       ├── gold/
│       └── reports/
│
└── execution/
    ├── risk_manager.py
    ├── order_executor.py
    └── telegram_alerts.py
8. PLAN DE DÉVELOPPEMENT DÉTAILLÉ
PHASE 3.1 - Infrastructure 1m (4 semaines)
Semaine	Module	Livrables
S1	Gold 1m	Choix source + loader opérationnel
S2	BTC 1m	Finalisation BTCEngine
S3	Tests 1m	Backtests validation sur tous actifs
S4	Sauvegarde runs	Structure + premier run comparatif
PHASE 3.2 - Multi-timeframes (6 semaines)
Semaine	Module	Livrables
S5	Loaders multi-TF	5m,15m,1h,4h pour tous
S6	Features par TF	Altcoins, BTC, Gold
S7	Règles confirmation	Implémentation R1,R2,R3
S8	Backtests comparatifs	Performance par TF
S9	Validation croisée	Choix TF optimal
S10	Documentation	Guide multi-timeframes
PHASE 3.3 - SL/TP & Risk (4 semaines)
Semaine	Module	Livrables
S11	Familles SL/TP	5 familles implémentées
S12	Fees & slippage	Intégration backtest
S13	Risk Manager	Par actif + global
S14	Tests validation	Backtests avec frais réels
PHASE 3.4 - OpenClaw multi-actifs (6 semaines)
Semaine	Module	Livrables
S15	Espace recherche	JSON par actif
S16	Adaptation OpenClaw	Support multi-actifs
S17	Optimisation Altcoins	50 générations
S18	Optimisation BTC	50 générations
S19	Optimisation Gold	50 générations
S20	Rapport final	Top 5 configs par actif
9. MÉTRIQUES DE SUCCÈS
Phase	Métrique	Objectif	Mesure
3.1	Tous actifs 1m	✅ S4	ls datasets/raw/*/1m.parquet
3.2	Multi-TF chargées	✅ S10	Tests unitaires
3.3	5 familles SL/TP	✅ S14	Implémentation validée
3.4	Top 5 configs	✅ S20	Fichiers JSON produits
Global	Runs sauvegardés	> 50	ls datasets/backtest_runs/ | wc -l
Global	Comparateur actif	✅	Dashboard opérationnel
Global	Sharpe backtest	> 1.2	Par actif
10. ANNEXES
10.1 Commandes utiles
bash
# Lancer un backtest avec sauvegarde
python3 run_backtest.py --asset btc --timeframe 1m --sl_family F2 --save

# Comparer tous les runs
python3 compare_runs.py --metric sharpe --top 10

# Lancer optimisation OpenClaw
python3 -m openclaw.multi_asset --generations 50 --save

# Générer rapport
python3 generate_report.py --output docs/reports/phase3_complete.md
10.2 Références
Canonique v1.2 (Phase 2 - Altcoins)

Canonique v3.0 (Phase 3 - Industrialisation)

Canonique v4.0 (Multi-Asset Framework)

Ce document : v5.0 (Phase 3 complète)

🏁 CONCLUSION
Ce document canonique v5.0 intègre :

✅ Vision 1M d'abord pour tous les actifs
✅ Extension multi-timeframes progressive
✅ 5 familles SL/TP universelles
✅ Fees & slippage réalistes
✅ Sauvegarde systématique des runs
✅ Analyse comparative multi-runs
✅ OpenClaw multi-actifs pour hyperoptimisation
✅ Plan détaillé sur 20 semaines

La feuille de route est claire. Prêt pour l'exécution. 🚀

*Document maintenu par le CTO - Version canonique v5.0 - 1er mars 2026*
