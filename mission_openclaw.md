🎯 MISSION_OPENCLAW.md
PRISE DE COMMANDE - OPTIMISATION STRATÉGIQUE
Date : 2026-02-28
Auteur : CTO
Statut : 🟢 ACTIF - TRANSFERT DE MISSION

📋 SOMMAIRE
État des lieux

Objectif de mission

Espace de recherche complet

Stratégies de sortie (SL/TP/TTP)

Pyramiding & gestion position

Paramètres réalistes

Algorithme génétique

Workflow détaillé

Métriques de fitness

Structure de sortie

Livrables attendus

Annexes techniques

✅ 1. ÉTAT DES LIEUX
Ce qui a été complété par le CTO
text
PHASE 1 (Infrastructure) : ✅ COMPLÈTE
├── Pipeline data Bybit (fetch 72h, pagination)
├── Stockage Parquet par symbole
└── Univers data-driven (150 paires)

PHASE 2 (Moteur événementiel) : ✅ COMPLÈTE
├── Module 1: EventScore v1 (normalisation rang centile)
├── Module 2: Early Warning System
├── Module 3: Pyramiding
├── Module 4: Universe Manager (filtrage + scoring)
├── Module 5: Timeframe Hybride (1M/5M)
├── Module 6: Backtesting Engine
├── Module 7: Genetic Optimizer
└── Module 8: Persistence & Comparison

PHASE 3 (Industrialisation) : ✅ STRUCTURE PRÊTE
├── Module 9: Backtesting Farm (parallélisation)
├── Module 10: Hyperparameter Database
├── Module 11: Performance Analytics
└── Module 12: Trading Connector

DOCUMENTS CANONIQUES : ✅ TOUS DISPONIBLES
├── canonique_v1.0.md (Phase 1 - archive)
├── canonique_v1.1.md (Phase 2 - archive)
├── canonique_v1.2.md (Phase 2 complète - actif)
├── canonique_v3.0_PHASE3.md (Phase 3 - actif)
└── annexes/ (dev_plan, cto_rules, changelog)
🎯 2. OBJECTIF DE MISSION
Trouver la combinaison gagnante de paramètres qui maximise l'espérance de gain sur les pumps Bybit, en explorant systématiquement toutes les familles de variables et stratégies de sortie.

Périmètre
Paires : 150 altcoins (univers filtré Bybit)

Période : 90 jours glissants (janvier-mars 2026)

Timeframe : 1M avec confirmation 5M

Capital simulé : 10,000 USDT

Taille position max : 2% du capital

Livrable final
Top 5 configurations validées, prêtes pour paper trading, avec :

Fichiers JSON complets

Métriques de performance

Walk-forward validation

Analyse de robustesse

🔬 3. ESPACE DE RECHERCHE COMPLET
3.1 Familles de variables fondamentales (7)
Famille	Variables	Plage	Description
🧊 Compression	range_ratio, bb_width, atr_compression	0.1-0.5	Détection phase de charge
🔥 Ignition	breakout_strength, candle_impulse	1.5-5.0	Cassure initiale
⚡ Momentum	roc_5, ema_cross, slope_angle	0.3-0.9	Accélération
📊 Volume	vol_ratio, vol_velocity, vol_delta	1.5-5.0	Confirmation liquidité
⚖️ Asymétrie	close_position, order_imbalance	0.4-0.9	Biais directionnel
⏱️ Persistance	consecutive_candles, hold_time	2-8	Durabilité
📈 HTF Context	trend_1h, ema_1h_position	-1 à +1	Contexte tendance
3.2 Poids des familles (à optimiser)
python
weights = {
    'w_compression': 0.0-1.0,   # Importance de la compression
    'w_ignition': 0.0-1.0,       # Importance de la cassure
    'w_momentum': 0.0-1.0,       # Importance de l'accélération
    'w_volume': 0.0-1.0,         # Importance du volume
    'w_asymmetry': 0.0-1.0,      # Importance du biais directionnel
    'w_persistence': 0.0-1.0,    # Importance de la durabilité
    'w_htf': 0.0-1.0             # Importance du contexte HTF
}
# CONTRAINTE: somme des poids = 1.0 (normalisation automatique)
3.3 Paramètres de scoring & seuils
Paramètre	Plage	Pas recommandé	Description
entry_threshold	60-90	5	Seuil EventScore pour entrée
early_warning_threshold	40-70	5	Seuil EarlyWarning pour surveillance
confirmation_candles	2-5	1	Bougies 1M de confirmation
volume_confirm_multiplier	1.5-3.0	0.25	Ratio volume pour confirmation
fenetre_norm	500-2000	250	Période normalisation rang
💰 4. STRATÉGIES DE SORTIE (SL/TP/TTP)
4.1 Famille A : SL/TP Fixes
ID	SL %	TP %	Ratio	Profil
A1	1.5	3.0	1:2	Conservateur
A2	2.0	4.0	1:2	Conservateur+
A3	2.0	6.0	1:3	Équilibré
A4	2.5	7.5	1:3	Agressif
A5	3.0	6.0	1:2	Large SL
A6	3.0	9.0	1:3	Très agressif
4.2 Famille B : Trailing Stop Only (TTP)
ID	Activation %	Distance %	Step %	Profil
B1	2.0	1.0	0.3	Trail serré
B2	2.5	1.2	0.4	Trail modéré
B3	3.0	1.5	0.5	Trail standard
B4	3.5	1.8	0.6	Trail large
B5	4.0	2.0	0.7	Trail très large
4.3 Famille C : Mixte (TP partiel + Trailing)
ID	TP1 %	% size	Activation TTP	Distance	Profil
C1	3.0	30%	2.5	1.2	Prudent
C2	4.0	30%	3.0	1.5	Modéré
C3	3.0	50%	2.5	1.2	Équilibré
C4	4.0	50%	3.0	1.5	Agressif
C5	5.0	30%	3.5	1.8	Très agressif
4.4 Famille D : ATR Dynamique
ID	SL (×ATR)	TP (×ATR)	ATR période	Profil
D1	1.2	3.0	14	Conservateur
D2	1.5	3.5	14	Modéré
D3	1.2	4.0	20	Équilibré
D4	1.5	4.5	20	Agressif
D5	2.0	5.0	20	Très agressif
4.5 Espace combinatoire des sorties
text
TOTAL = 6 (A) + 5 (B) + 5 (C) + 5 (D) = 21 STRATÉGIES DE SORTIE
📐 5. PYRAMIDING & GESTION POSITION
5.1 Modes de répartition
ID	Tranche 1	Tranche 2	Tranche 3	Description
P1	40%	30%	30%	Équilibré
P2	50%	30%	20%	Front-loaded
P3	60%	40%	-	2 tranches
P4	33%	33%	34%	Égalitaire
P5	50%	25%	25%	Progressif
5.2 Paramètres associés
Paramètre	Plage	Pas	Description
min_gain_between_tiers	1.0-3.0%	0.5%	Gain mini entre tranches
min_time_between_tiers	3-10 min	1 min	Temps mini entre tranches
max_total_position	1.0-5.0%	0.5%	% max du capital
5.3 Règles de gestion
python
PYRAMIDING_RULES = {
    "activation": "Chaque tranche ne s'active que si la précédente est en gain",
    "stop_loss": "Stop individuel par tranche (-2% de son prix d'entrée)",
    "breakeven": "Quand gain global > 4%, tous les stops au BE moyen",
    "trailing": "Première tranche bénéficie d'un trailing après activation"
}
💸 6. PARAMÈTRES RÉALISTES
6.1 Configuration fees & slippage
python
FEES_SLIPPAGE_CONFIG = {
    'fees': {
        'maker': 0.001,      # 0.1% (Bybit standard)
        'taker': 0.001,      # 0.1% (identique pour simplification)
        'use_discount': False, # True si volume > 1M$
    },
    'slippage': {
        'model': 'fixed',     # "fixed", "volume_based", "historical"
        'fixed_pct': 0.05,    # 0.05% slippage fixe (valeur conservative)
        'volume_factor': 0.1, # Pour modèle basé sur volume
        'min_pct': 0.02,      # Slippage minimum
        'max_pct': 0.2,       # Slippage maximum
        'liquidity_threshold': 500000  # Seuil liquidité (USD)
    }
}
6.2 Impact calcul PnL
python
def calculate_net_pnl(entry_price, exit_price, size, side='long'):
    """
    Calcule le PnL net après fees et slippage
    """
    # Slippage à l'entrée
    if side == 'long':
        effective_entry = entry_price * (1 + slippage_pct/100)
        effective_exit = exit_price * (1 - slippage_pct/100)
    else:  # short
        effective_entry = entry_price * (1 - slippage_pct/100)
        effective_exit = exit_price * (1 + slippage_pct/100)
    
    # PnL brut
    gross_pnl = (effective_exit - effective_entry) * size
    
    # Frais (taker sur entrée ET sortie)
    fee_cost = (effective_entry * size * fees['taker']) + \
               (effective_exit * size * fees['taker'])
    
    # PnL net
    net_pnl = gross_pnl - fee_cost
    
    return net_pnl
🧬 7. ALGORITHME GÉNÉTIQUE
7.1 Chromosome complet
python
chromosome = {
    # POIDS DES FAMILLES (7 variables)
    'w_compression': 0.25,      # 0.0-1.0
    'w_ignition': 0.30,          # 0.0-1.0
    'w_momentum': 0.15,          # 0.0-1.0
    'w_volume': 0.20,            # 0.0-1.0
    'w_asymmetry': 0.10,         # 0.0-1.0
    'w_persistence': 0.15,       # 0.0-1.0
    'w_htf': 0.10,               # 0.0-1.0
    
    # SEUILS (5 variables)
    'entry_threshold': 75,       # 60-90
    'early_warning_threshold': 45, # 40-70
    'confirmation_candles': 3,   # 2-5
    'volume_multiplier': 2.2,    # 1.5-3.0
    'fenetre_norm': 1000,        # 500-2000
    
    # STRATÉGIE DE SORTIE (1 variable catégorielle)
    'exit_strategy': 'C3',       # A1..D5 (21 options)
    
    # PYRAMIDING (4 variables)
    'pyramiding_mode': 'P1',     # P1..P5 (5 options)
    'tier_gain_min': 1.8,        # 1.0-3.0%
    'tier_time_min': 5,          # 3-10 min
    'max_position_pct': 2.0,     # 1.0-5.0%
    
    # FEES & SLIPPAGE (fixes - pas optimisés)
    'fees_taker': 0.001,
    'slippage_pct': 0.05
}
7.2 Paramètres génétiques
Paramètre	Valeur	Justification
Population size	100	Bon équilibre diversité/temps
Générations	50	Convergence suffisante
Taux élite	20%	Garde les meilleurs
Taux croisement	70%	Exploration
Taux mutation	10%	Évite optimums locaux
Mutation amplitude	5-15%	Variation réaliste
Tournament size	3	Sélection robuste
7.3 Initialisation population
python
def initialize_population(size=100):
    population = []
    for _ in range(size):
        # Poids aléatoires (normalisés)
        raw_weights = np.random.uniform(0, 1, 7)
        normalized_weights = raw_weights / raw_weights.sum()
        
        chromosome = {
            # Poids normalisés
            'w_compression': normalized_weights[0],
            'w_ignition': normalized_weights[1],
            'w_momentum': normalized_weights[2],
            'w_volume': normalized_weights[3],
            'w_asymmetry': normalized_weights[4],
            'w_persistence': normalized_weights[5],
            'w_htf': normalized_weights[6],
            
            # Seuils aléatoires
            'entry_threshold': np.random.randint(60, 91),
            'early_warning_threshold': np.random.randint(40, 71),
            'confirmation_candles': np.random.randint(2, 6),
            'volume_multiplier': np.random.uniform(1.5, 3.0),
            'fenetre_norm': np.random.choice([500,750,1000,1250,1500,1750,2000]),
            
            # Stratégie aléatoire
            'exit_strategy': np.random.choice([
                'A1','A2','A3','A4','A5','A6',
                'B1','B2','B3','B4','B5',
                'C1','C2','C3','C4','C5',
                'D1','D2','D3','D4','D5'
            ]),
            
            # Pyramiding aléatoire
            'pyramiding_mode': np.random.choice(['P1','P2','P3','P4','P5']),
            'tier_gain_min': np.random.uniform(1.0, 3.0),
            'tier_time_min': np.random.randint(3, 11),
            'max_position_pct': np.random.uniform(1.0, 5.0),
            
            # Fixes
            'fees_taker': 0.001,
            'slippage_pct': 0.05
        }
        population.append(chromosome)
    
    return population
🔄 8. WORKFLOW DÉTAILLÉ
text
┌─────────────────────────────────────────────────────────────────┐
│                    WORKFLOW OPENCLAW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  GÉNÉRATION 1                                                    │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ Population initiale (100 chromosomes aléatoires)        │     │
│  └─────────────────────────────────────────────────────────┘     │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ BACKTEST PARALLÈLE                                        │     │
│  │ ┌─────────────────────────────────────────────────────┐ │     │
│  │ │ Worker 1: chromosome 1 → métriques                   │ │     │
│  │ │ Worker 2: chromosome 2 → métriques                   │ │     │
│  │ │ ... (8 workers en parallèle)                         │ │     │
│  │ └─────────────────────────────────────────────────────┘ │     │
│  └─────────────────────────────────────────────────────────┘     │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ ÉVALUATION                                               │     │
│  │ • Calcul fitness pour chaque chromosome                 │     │
│  │ • Classement                                             │     │
│  │ • Sauvegarde génération                                 │     │
│  └─────────────────────────────────────────────────────────┘     │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ SÉLECTION                                                │     │
│  │ • Élite: top 20% conservés                              │     │
│  │ • Reste: sélection par tournoi                          │     │
│  └─────────────────────────────────────────────────────────┘     │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ CROISEMENT & MUTATION                                    │     │
│  │ • 70% croisement (uniform crossover)                    │     │
│  │ • 10% mutation aléatoire                                │     │
│  │ • Création nouvelle population                          │     │
│  └─────────────────────────────────────────────────────────┘     │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ GÉNÉRATION SUIVANTE (×50)                                │     │
│  └─────────────────────────────────────────────────────────┘     │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ ANALYSE FINALE                                           │     │
│  │ • Évolution fitness                                      │     │
│  │ • Importance paramètres                                  │     │
│  │ • Top 10 configurations                                  │     │
│  │ • Walk-forward validation                                │     │
│  └─────────────────────────────────────────────────────────┘     │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ RAPPORT FINAL                                            │     │
│  │ • Top 5 configurations (JSON)                           │     │
│  │ • Métriques détaillées                                   │     │
│  │ • Recommandation paper trading                          │     │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
📊 9. MÉTRIQUES DE FITNESS
9.1 Formule principale
python
def calculate_fitness(metrics, trades):
    """
    Calcule le fitness d'un chromosome
    """
    # Métriques de base
    sharpe = metrics.get('sharpe_ratio', 0)
    profit_factor = metrics.get('profit_factor', 0)
    win_rate = metrics.get('win_rate', 0)
    avg_win = metrics.get('avg_win', 0)
    avg_loss = metrics.get('avg_loss', 0)
    max_dd = metrics.get('max_drawdown', 0)
    total_trades = metrics.get('total_trades', 0)
    
    # Ratio win/loss
    win_loss_ratio = avg_win / avg_loss if avg_loss != 0 else 0
    
    # Fitness brut
    raw_fitness = (
        sharpe * 0.35 +
        profit_factor * 0.25 +
        win_rate * 0.15 +
        win_loss_ratio * 0.15 -
        max_dd * 0.10
    )
    
    # Pénalités de surapprentissage
    penalty = 0
    
    if total_trades < 30:
        penalty += 0.20  # Pas assez de trades
    elif total_trades < 50:
        penalty += 0.10
    
    if max_dd > 0.25:
        penalty += 0.15  # Drawdown trop élevé
    
    if win_rate > 0.70:
        penalty += 0.10  # Suspect (trop beau)
    
    if sharpe > 3.0:
        penalty += 0.30  # Très suspect
    
    if profit_factor > 5.0:
        penalty += 0.20  # Probablement surappris
    
    # Fitness final
    fitness = raw_fitness * (1 - min(penalty, 0.8))
    
    return max(0, fitness)  # Non négatif
9.2 Métriques complémentaires
Métrique	Seuil acceptable	Seuil cible
Total trades	> 50	> 100
Win rate	45-60%	50-65%
Profit factor	> 1.5	> 2.0
Sharpe ratio	> 1.0	> 1.5
Max drawdown	< 20%	< 15%
Avg trade duration	5-60 min	10-45 min
Recovery factor	> 2.0	> 3.0
📁 10. STRUCTURE DE SORTIE
text
datasets/optimisation/
│
├── runs/
│   ├── 2026-02-28_10h30_gen1_pop100/
│   │   ├── config.json
│   │   ├── population.json
│   │   ├── fitness_scores.csv
│   │   └── best_chromosome.json
│   ├── 2026-02-28_12h15_gen2_pop100/
│   └── ...
│
├── generations/
│   ├── generation_01/
│   │   ├── population.json
│   │   ├── fitness_stats.json
│   │   └── top5.json
│   ├── generation_02/
│   └── ...
│   └── generation_50/
│
├── analysis/
│   ├── fitness_evolution.csv
│   ├── fitness_evolution.png
│   ├── parameter_importance.csv
│   ├── parameter_correlation.csv
│   ├── top10_configs.json
│   ├── top10_metrics.csv
│   └── walk_forward_results.csv
│
├── validation/
│   ├── walk_forward_periods.json
│   ├── stability_scores.csv
│   └── recommended_configs.json
│
└── reports/
    ├── optimisation_report_2026-03-01.html
    ├── optimisation_report_2026-03-01.pdf
    └── presentation_cto.md
📦 11. LIVRABLES ATTENDUS
11.1 Livrables obligatoires
text
📦 DOSSIER DE LIVRAISON OPENCLAW/
├── 1_TOP5_CONFIGS/
│   ├── config_01.json
│   ├── config_02.json
│   ├── config_03.json
│   ├── config_04.json
│   ├── config_05.json
│   └── config_summary.csv
│
├── 2_METRICS/
│   ├── backtest_results.parquet
│   ├── metrics_comparison.csv
│   └── equity_curves.png
│
├── 3_VALIDATION/
│   ├── walk_forward_results.json
│   ├── stability_analysis.md
│   └── out_of_sample_test.csv
│
├── 4_ANALYSIS/
│   ├── parameter_importance.json
│   ├── correlation_matrix.png
│   └── fitness_evolution.png
│
└── 5_REPORT/
    ├── optimisation_report.html
    ├── recommandation_cto.md
    └── presentation.pptx
11.2 Format des configurations
json
{
  "config_id": "openclaw_opt_001",
  "timestamp": "2026-03-01T10:30:00Z",
  "fitness": 1.24,
  "parameters": {
    "weights": {
      "compression": 0.28,
      "ignition": 0.32,
      "momentum": 0.12,
      "volume": 0.18,
      "asymmetry": 0.05,
      "persistence": 0.03,
      "htf": 0.02
    },
    "thresholds": {
      "entry": 78,
      "early_warning": 52,
      "confirmation_candles": 3,
      "volume_multiplier": 2.4,
      "fenetre_norm": 1000
    },
    "exit_strategy": {
      "family": "C",
      "id": "C3",
      "description": "TP partiel 3.0% (50%) + trailing 2.5/1.2"
    },
    "pyramiding": {
      "mode": "P1",
      "tiers": [0.4, 0.3, 0.3],
      "min_gain_pct": 1.8,
      "min_time_min": 5,
      "max_position_pct": 2.0
    }
  },
  "metrics": {
    "sharpe": 1.86,
    "profit_factor": 2.34,
    "win_rate": 0.58,
    "total_trades": 342,
    "max_drawdown": 0.12,
    "avg_win": 4.2,
    "avg_loss": 1.8,
    "expectancy": 1.68
  }
}
🧪 12. ANNEXES TECHNIQUES
12.1 BacktestEngine - Interface
python
class BacktestEngine:
    """
    Moteur de backtest pour évaluer une configuration
    """
    
    def __init__(self, config: Dict):
        self.config = config
        self.fees = FEES_SLIPPAGE_CONFIG['fees']
        self.slippage = FEES_SLIPPAGE_CONFIG['slippage']
    
    def run(self, 
            pairs: List[str], 
            start_date: str, 
            end_date: str) -> Dict:
        """
        Exécute un backtest complet
        
        Returns:
            {
                'trades': List[Dict],
                'metrics': Dict,
                'equity_curve': List[float]
            }
        """
        pass
    
    def _calculate_event_score(self, df: pd.DataFrame) -> float:
        """Calcule EventScore selon configuration"""
        pass
    
    def _apply_exit_strategy(self, trade: Dict, price_data: pd.Series) -> Dict:
        """Applique la stratégie de sortie configurée"""
        pass
12.2 GeneticOptimizer - Interface
python
class GeneticOptimizer:
    """
    Optimiseur génétique pour explorer l'espace des paramètres
    """
    
    def __init__(self, 
                 population_size: int = 100,
                 generations: int = 50,
                 elite_rate: float = 0.2,
                 crossover_rate: float = 0.7,
                 mutation_rate: float = 0.1):
        
        self.population = []
        self.fitness_history = []
        self.best_chromosomes = []
    
    def initialize_population(self):
        """Crée population initiale"""
        pass
    
    def evaluate_population(self, backtest_engine: BacktestEngine):
        """Évalue tous les chromosomes"""
        pass
    
    def selection(self):
        """Sélectionne les meilleurs"""
        pass
    
    def crossover(self):
        """Croise les chromosomes"""
        pass
    
    def mutation(self):
        """Mute certains chromosomes"""
        pass
    
    def run(self, 
            backtest_engine: BacktestEngine,
            pairs: List[str],
            date_range: Tuple[str, str]) -> Dict:
        """
        Lance l'optimisation complète
        
        Returns:
            {
                'best_chromosomes': List[Dict],
                'fitness_history': List[float],
                'generation_stats': List[Dict]
            }
        """
        pass
12.3 WalkForwardValidator - Interface
python
class WalkForwardValidator:
    """
    Valide la robustesse d'une configuration
    """
    
    def __init__(self, n_splits: int = 5):
        self.n_splits = n_splits
    
    def validate(self, 
                 config: Dict,
                 pairs: List[str],
                 full_date_range: Tuple[str, str]) -> Dict:
        """
        Validation walk-forward sur périodes glissantes
        
        Returns:
            {
                'stability_score': float,
                'period_results': List[Dict],
                'avg_sharpe_train': float,
                'avg_sharpe_test': float,
                'sharpe_decay': float
            }
        """
        # Divise en n_splits périodes
        # Pour chaque split: train sur k-1, test sur 1
        # Calcule stabilité = sharpe_test / sharpe_train
        pass
🏁 13. COMMANDES DE LANCEMENT
bash
# Lancer une optimisation complète (50 générations)
python -m openclaw.optimizer.run \
    --population 100 \
    --generations 50 \
    --pairs 150 \
    --start-date 2026-01-01 \
    --end-date 2026-03-01 \
    --output-dir datasets/optimisation/run_$(date +%Y%m%d_%H%M)

# Visualiser l'évolution
python -m openclaw.analysis.plot_fitness \
    --input datasets/optimisation/run_*/generations/

# Générer rapport final
python -m openclaw.report.generate \
    --input datasets/optimisation/run_*/ \
    --output reports/optimisation_report.html
✅ 14. CHECKLIST DE MISSION
Semaine 1: 10 générations → vérifier convergence

Semaine 2: 20 générations → premières tendances

Semaine 3: 40 générations → stabilisation

Semaine 4: 50 générations → configuration finale

Semaine 5: Walk-forward validation

Semaine 6: Rapport final et recommandations

🎯 15. CONCLUSION
OpenClaw, la mission est claire :

Explore 21 stratégies de sortie, 7 familles de variables, 5 modes de pyramiding, et trouve les 5 configurations qui maximisent l'espérance de gain sur les pumps Bybit.

GESTION DES HYPOTHÈSES
    Les hypothèses de trading (ex: "les funding rates extrêmes précèdent les retournements") 
    sont des guides, pas des vérités absolues.
    
    Si une hypothèse n'est PAS validée par les backtests :
    - Ce n'est PAS un échec du projet
    - C'est une INFORMATION précieuse
    - On ajuste, on modifie, on itère
    - On documente la leçon dans le registre des hypothèses
    
    La robustesse du système vient de sa capacité à rejeter les mauvaises hypothèses.

Tous les outils sont prêts. Les données sont là. Les métriques sont définies.

La balle est dans ton camp. Bonne chasse ! 🚀

*Document transféré par le CTO - 28 février 2026 - Prise de commande effective*
