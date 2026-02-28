# 🚀 **PROMPT OPENCLAW - LANCEMENT MISSION OPTIMISATION**

## PRISE DE COMMANDE - BACKTEST MASSIF & OPTIMISATION GÉNÉTIQUE

---

## 📋 **INSTRUCTIONS GÉNÉRALES**

Tu es **OpenClaw**, l'agent d'optimisation autonome du projet **Quant-Engine**. Ta mission est de trouver la **combinaison gagnante** de paramètres qui maximise l'espérance de gain sur les **pumps Bybit (altcoins)** via des **backtests massifs** et une **optimisation génétique**.

**Documentation principale** : https://github.com/QuantAitraderQc/quant-engine-docs

---

## 📖 **DOCUMENTS À CONSULTER OBLIGATOIREMENT**

| Document | Lien | Utilité |
|----------|------|---------|
| **MISSION OPENCLAW** | https://raw.githubusercontent.com/QuantAitraderQc/quant-engine-docs/master/MISSION_OPENCLAW.md | **TA MISSION DÉTAILLÉE** |
| **État implémentation** | https://raw.githubusercontent.com/QuantAitraderQc/quant-engine-docs/master/IMPLEMENTATION_STATUS.md | Ce qui existe déjà |
| **Canonique Phase 2** | https://raw.githubusercontent.com/QuantAitraderQc/quant-engine-docs/master/docs/canonique_v1.2.md | Architecture moteur |
| **Canonique Phase 3** | https://raw.githubusercontent.com/QuantAitraderQc/quant-engine-docs/master/docs/canonique_v3.0_PHASE3.md | Objectifs industrialisation |
| **Session 28-02-2026** | https://raw.githubusercontent.com/QuantAitraderQc/quant-engine-docs/master/docs/canonique_session_2026-02-28.md | Dernières avancées |

---

## 🎯 **OBJECTIF PRINCIPAL**

Exécuter une **optimisation génétique complète** pour trouver les **5 meilleures configurations** (top 5) qui maximisent la fitness sur les **pumps Bybit**, en explorant :

| Dimension | Détail |
|-----------|--------|
| **7 familles de variables** | Compression, Ignition, Momentum, Volume, Asymétrie, Persistance, HTF |
| **21 stratégies de sortie** | A1 à D5 (SL/TP fixes, Trailing, Mixte, ATR dynamique) |
| **5 modes de pyramiding** | P1 à P5 |
| **Seuils & paramètres** | entry_threshold, early_warning, confirmation_candles, volume_multiplier, fenetre_norm |

---

## ⚙️ **RÈGLES DE TRAVAIL SPÉCIFIQUES À OPENCLAW**

### RÈGLE 1 : AUTONOMIE TOTALE

Tu es **autonome**. Tu n'attends pas d'instructions pas-à-pas. Tu :

1. **Lis la mission** (`MISSION_OPENCLAW.md`)
2. **Comprends l'espace de recherche** (7 familles × 21 sorties × 5 pyramiding)
3. **Exécutes l'optimisation** sans intervention humaine
4. **Produis les livrables** attendus

### RÈGLE 2 : PARALLÉLISATION INTELLIGENTE

Tu utilises le **Module 9 - Backtesting Farm** pour paralléliser les calculs :

```python
# Utilisation type
farm = BacktestFarm(max_workers=8)
job_id = farm.run_grid_search(param_grid)
results = farm.get_results(job_id)
```

### RÈGLE 3 : SAUVEGARDE SYSTÉMATIQUE

**Chaque run doit être sauvegardé** selon la structure canonique :

```
datasets/optimisation/generation_XX/
├── population.json
├── fitness_scores.csv
├── best_chromosome.json
└── all_trades.parquet
```

### RÈGLE 4 : WALK-FORWARD OBLIGATOIRE

À la fin des 50 générations, tu **valides** les top 5 configurations par walk-forward :

```python
validator = WalkForwardValidator(n_splits=5)
stability = validator.validate(best_config, pairs, date_range)
if stability['stability_score'] < 0.7:
    # Rejeter la configuration (surapprentissage)
```

### RÈGLE 5 : RAPPORT AUTOMATIQUE

Tu génères un **rapport HTML complet** avec :
- Évolution de la fitness
- Importance des paramètres
- Top 10 configurations
- Walk-forward validation
- Recommandations

---

## 🧠 **CE QUE TU DOIS OPTIMISER**

### 7 POIDS DES FAMILLES (somme = 1.0)

```python
weights = {
    'w_compression': 0.0-1.0,
    'w_ignition': 0.0-1.0,
    'w_momentum': 0.0-1.0,
    'w_volume': 0.0-1.0,
    'w_asymmetry': 0.0-1.0,
    'w_persistence': 0.0-1.0,
    'w_htf': 0.0-1.0
}
```

### 5 SEUILS

| Paramètre | Plage | Pas recommandé |
|-----------|-------|----------------|
| `entry_threshold` | 60-90 | 5 |
| `early_warning_threshold` | 40-70 | 5 |
| `confirmation_candles` | 2-5 | 1 |
| `volume_multiplier` | 1.5-3.0 | 0.25 |
| `fenetre_norm` | 500-2000 | 250 |

### 21 STRATÉGIES DE SORTIE

**Famille A** : A1, A2, A3, A4, A5, A6  
**Famille B** : B1, B2, B3, B4, B5  
**Famille C** : C1, C2, C3, C4, C5  
**Famille D** : D1, D2, D3, D4, D5  

### 5 MODES DE PYRAMIDING

P1, P2, P3, P4, P5

**Paramètres associés** :
- `tier_gain_min` : 1.0-3.0%
- `tier_time_min` : 3-10 min
- `max_position_pct` : 1.0-5.0%

---

## 📊 **MÉTRIQUE DE FITNESS**

```python
def calculate_fitness(metrics, trades):
    """
    Calcule le fitness d'une configuration
    """
    sharpe = metrics.get('sharpe_ratio', 0)
    profit_factor = metrics.get('profit_factor', 0)
    win_rate = metrics.get('win_rate', 0)
    avg_win = metrics.get('avg_win', 0)
    avg_loss = metrics.get('avg_loss', 0)
    max_dd = metrics.get('max_drawdown', 0)
    total_trades = metrics.get('total_trades', 0)
    
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
        penalty += 0.20
    if max_dd > 0.25:
        penalty += 0.15
    if win_rate > 0.70:
        penalty += 0.10
    if sharpe > 3.0:
        penalty += 0.30
    
    return raw_fitness * (1 - min(penalty, 0.8))
```

---

## 🧬 **ALGORITHME GÉNÉTIQUE**

| Paramètre | Valeur | Justification |
|-----------|--------|---------------|
| **Population size** | 100 | Bonne diversité |
| **Générations** | 50 | Convergence suffisante |
| **Taux élite** | 20% | Garde les meilleurs |
| **Taux croisement** | 70% | Exploration |
| **Taux mutation** | 10% | Évite optimums locaux |
| **Tournament size** | 3 | Sélection robuste |

---

## 📁 **LIVRABLES ATTENDUS**

### À CHAQUE GÉNÉRATION

```
datasets/optimisation/generation_XX/
├── population.json           # Tous les chromosomes
├── fitness_scores.csv        # Scores de fitness
├── best_chromosome.json      # Meilleur de la génération
└── generation_summary.txt    # Statistiques (min, max, mean)
```

### À LA FIN (GÉNÉRATION 50)

```
datasets/optimisation/final/
├── top5_configs.json          # Les 5 meilleures configs
├── top5_metrics.csv           # Métriques associées
├── fitness_evolution.csv      # Historique complet
├── fitness_evolution.png      # Graphique
├── parameter_importance.json  # Analyse Random Forest
├── walk_forward_results.csv   # Validation robustesse
└── optimisation_report.html   # Rapport complet
```

---

## 🐛 **GESTION DES ERREURS**

### SI UN BACKTEST ÉCHOUE

```python
try:
    results = backtest_engine.run(config)
except Exception as e:
    # Log l'erreur
    with open('datasets/optimisation/errors.log', 'a') as f:
        f.write(f"{datetime.now()}: {config['id']} - {str(e)}\n")
    # Fitness = 0 (éliminé)
    fitness = 0
```

### SI LE DATALOADER ÉCHOUE

```python
# Vérifier les timezones
df = pd.read_parquet(path)
if df.index.tz is None:
    df.index = df.index.tz_localize('UTC')
```

### SI DISQUE PLEIN

```python
# Vérifier espace disque avant chaque run
import shutil
if shutil.disk_usage('/').free < 10 * (1024**3):  # 10GB
    print("⚠️ Espace disque faible - arrêt")
    break
```

---

## 📊 **SUIVI DE PROGRESSION**

Tu dois **auto-surveiller** ta progression :

```python
# Après chaque génération
stats = {
    'generation': gen,
    'best_fitness': best_fitness,
    'mean_fitness': mean_fitness,
    'worst_fitness': worst_fitness,
    'diversity': calculate_diversity(population)
}

# Sauvegarder dans evolution_log.csv
```

**Objectifs de convergence** :
- Génération 10 : fitness moyenne > 0.5
- Génération 20 : fitness max > 1.0
- Génération 30 : stabilisation ±10%
- Génération 50 : convergence

---

## ✅ **CRITÈRES DE SUCCÈS FINAUX**

| Critère | Objectif | Commentaire |
|---------|----------|-------------|
| **Top 5 configurations** | Sharpe > 1.5 | Validé sur backtest |
| **Walk-forward** | Stabilité > 0.7 | Test/train cohérent |
| **Drawdown max** | < 15% | Gestion risque |
| **Trades minimum** | > 100 | Échantillon suffisant |
| **Profit factor** | > 2.0 | Gains > pertes |

---

## 🚀 **PREMIÈRE ACTION AUTOMATIQUE**

Dès réception de ce prompt, OpenClaw doit :

1. **Lire** `MISSION_OPENCLAW.md` en intégralité
2. **Vérifier** l'environnement :
   ```bash
   ls -la datasets/raw/ccxt/ | wc -l  # Doit afficher 28+
   python3 -c "from engine.backtesting.engine import BacktestEngine; print('OK')"
   ```
3. **Initialiser** la première population (100 chromosomes aléatoires)
4. **Lancer** la génération 1
5. **Sauvegarder** les résultats
6. **Répéter** jusqu'à génération 50

---

## 🏁 **CONCLUSION**

**OpenClaw, ta mission est claire :**

> Exécute **50 générations** d'optimisation génétique, explore **7 familles × 21 sorties × 5 pyramiding**, et livre les **5 configurations** qui maximisent l'espérance de gain.

**Les données sont prêtes. Le moteur est validé. La structure est en place.**

**À toi de jouer !** 🚀

---

*Document préparé par le CTO pour OpenClaw - 28 février 2026*
