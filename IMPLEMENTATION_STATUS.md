# Quant-Engine / OpenClaw - État d'implémentation

## ✅ PHASE 1 - COMPLÈTE (OPÉRATIONNELLE)

### Infrastructure
- ✅ VPS Hostinger avec Ubuntu
- ✅ Environnement Python (.venv)
- ✅ Arborescence canonique respectée

### Données
- ✅ Connexion Bybit via CCXT (linear perps)
- ✅ Fetch 72h avec pagination
- ✅ Stockage Parquet par symbole : `datasets/raw/ccxt/<SYMBOL>:USDT/1m.parquet`
- ✅ Univers data-driven (basé sur fichiers présents)
- ✅ 650+ paires disponibles

### Screener 1m
- ✅ `engine/screener/run_screener.py` opérationnel
- ✅ Calcul des scores : compression, ignition, opportunité
- ✅ Outputs : `scan_latest.parquet`, `top_20.parquet`

### Registry & Backtest
- ✅ Génération registry d'events LTF
- ✅ Ancrage canonique T-5 minutes
- ✅ Backtesting event-based fonctionnel
- ✅ Métriques : MFE, MAE, time_to_peak

## 🔄 PHASE 2 v1.1 - EN COURS

### État actuel
- ✅ Spécification canonique v1.1 finalisée (scoring formel, early warning, pyramiding)
- ✅ Structure GitHub documentaire en place
- ⏳ **PROCHAINE ÉTAPE : Module 1 - EventScore v1**
  - Agrégation non-linéaire (moyenne géométrique)
  - Normalisation par rang centile
  - EarlyWarningScore (pré-détection)
  - Variables à mémoire (dérivées temporelles)

### Code existant (dans le dépôt privé original)
- `/engine/screener/` → screener_1m.py, run_screener.py
- `/engine/features/` → regime_shift.py, etc.
- `/engine/universe/` → alt_sampling_plan.py
- Données Parquet : des milliers de fichiers (non publics)
