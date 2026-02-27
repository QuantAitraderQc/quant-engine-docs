# Quant-Engine / OpenClaw - État du projet

## 📌 Identité du projet
- **Focus** : Détection pump & dump sur Bybit (altcoins uniquement)
- **Timeframe** : Live 1M avec confirmation 5M
- **Stack** : Python, CCXT, Parquet, Pandas
- **Philosophie** : Reconnaître ≠ prédire, confluence obligatoire

## ✅ Ce qui est terminé (Phase 1)
- Pipeline data stable (fetch 72h, pagination CCXT)
- Screener 1m opérationnel
- Univers data-driven
- Registry d'events LTF
- Backtesting event-based (T-5)

## 🔄 Phase 2 v1.1 - En cours
- ✅ Document canonique v1.1 finalisé (scoring formel, early warning, pyramiding)
- ✅ Structure GitHub mise en place
- ⏳ **Prochaine étape : Module 1 - EventScore v1**
  - Agrégation non-linéaire (moyenne géométrique)
  - Normalisation par rang centile
  - EarlyWarningScore (pré-détection)
  - Variables à mémoire

## 📂 Où trouver quoi
- `docs/canonique_v1.1.md` → Spécification complète
- `docs/annexes/dev_plan.md` → Roadmap détaillée
- `docs/annexes/changelog.md` → Historique des modifications
- `docs/annexes/cto_rules.md` → Gouvernance technique

## 🎯 Objectif immédiat
Implémenter le Module 1 du plan de développement.
