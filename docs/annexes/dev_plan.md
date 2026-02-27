# Plan de développement technologique
## Quant-Engine / OpenClaw
Statut : ANNEXE CANONIQUE (référence d'exécution)
Autorité : CTO

## 1) Objectif de cette annexe
Cette annexe a pour but de :
- structurer l'exécution technique du projet
- définir les phases de développement successives
- préciser les livrables attendus par phase
- organiser le travail d'équipe via GitHub
- empêcher toute dérive, implémentation prématurée ou mélange des responsabilités

👉 Ce document n'ajoute aucune logique métier : il organise comment on implémente ce qui est déjà canonique.

## 2) Découpage global des phases
Vue d'ensemble
| Phase | Objectif principal | Statut |
|-------|-------------------|--------|
| Phase 1 | Pipeline data & screener LTF | ✅ Terminée |
| Phase 2 v1.1 | Event Recognition Engine (altcoins) | 🔒 Canonique |
| Phase 2.x | Stabilisation & enrichissement EventScore | ⏳ À développer |
| Phase 3 | Trading live structuré & multi-actifs | ⏳ À venir |
| Phase 4 | Extensions avancées (options, arbitrage) | 🔮 Hors scope immédiat |

## 3) Phase 2.x — Développement technologique (prioritaire)
### 3.1 Objectif Phase 2.x
Stabiliser, formaliser et enrichir l'Event Recognition Engine sans introduire :
- de filtres durs
- de dépendance excessive au backtest 1M
- de coupling avec l'exécution

### 3.2 Modules à développer (ordre imposé par le CTO)
**MODULE 1 — EventScore v1 (fondation)**
Sections canonique liées : §20, §21, §22
Livrables :
- module event_score_v1.py
- définition formelle des familles de variables
- normalisation des sous-scores
- agrégation continue (0–100)

Règles :
- aucun seuil binaire
- aucune dépendance à un trigger
- compatible replay & live

Validation :
- score stable sur données neutres
- distribution non dégénérée
- aucun event "fantôme"

**MODULE 2 — Enrichissement des variables LTF**
Sections liées : §21.2
Variables à intégrer progressivement :
- candle patterns (engulfing, pin, etc.)
- liquidity sweep
- inverted fair value gap
- open interest (LTF / HTF)

Règles :
- score marginal faible
- confluence uniquement
- désactivable sans casser le moteur

Validation :
- gain d'expectancy mesuré offline
- aucune création d'event isolée

**MODULE 3 — Univers dynamique (100–200 paires)**
Sections liées : §23
Livrables :
- score d'éligibilité univers
- règles d'entrée/sortie
- inertie contrôlée (anti-churn)

Validation :
- univers stable sur 24–72h
- couverture d'events rare > univers complet

**MODULE 4 — OpenClaw (offline learning)**
Sections liées : §24, §29
Fonctions :
- backtests event-based multi-TF
- mesure contribution marginale variables
- rejet automatique des combinaisons fragiles
- recommandations de poids

Validation :
- reproductibilité
- stabilité inter-régimes
- pas d'optimisation opportuniste

## 4) Phase 3 — Trading live & multi-actifs
### 4.1 Phase 3.1 — Event → Trade (altcoins)
Sections liées : §25, §26
Livrables :
- TradeScore formel
- RiskScore (SL, BE, TP, DD)
- Execution Engine stable
- pyramiding contrôlé

Conditions CTO de déblocage :
- EventScore stable
- registry backtestable
- kill-switch actif
- logs complets

### 4.2 Phase 3.2 — Pipelines parallèles BTC & XAU
Principe :
BTC et Gold ne partagent PAS la logique pump & dump.
Livrables :
- pipeline dédié BTC
- pipeline dédié XAU
- backtesting complet (non événementiel)

Règle CTO :
❌ aucune réutilisation forcée du modèle altcoins

## 5) Phase ultérieure — Extensions (non prioritaires)
### 5.1 Options trading scanner
- phase post-stabilisation
- architecture dédiée
- métriques spécifiques (IV, Greeks)

### 5.2 Arbitrage Polymarket / Kalshi
- projet indépendant
- agents spécialisés
- backtest obligatoire avant live

## 6) Architecture GitHub (obligatoire)
### 6.1 Création du repository
Nom recommandé : quant-engine-openclaw
Visibilité : Privé (initialement), accès contrôlé par rôle

### 6.2 Structure du repo
quant-engine-openclaw/
│
├── engine/              # moteur principal
├── openclaw/            # offline learning
├── execution/           # trade & risk engine
├── datasets/            # ignoré (gitignore)
├── docs/
│   ├── canonique.md
│   └── annexes/
│       ├── dev_plan.md
│       └── cto_rules.md
│
├── tests/
├── scripts/
├── .gitignore
└── README.md


### 6.3 Règles Git (figées)
- branche main protégée
- PR obligatoire
- aucun commit direct sur main
- 1 PR = 1 feature ou 1 fix
- tests requis avant merge

### 6.4 Rôles GitHub
| Rôle | Droits |
|------|--------|
| CTO | Merge final, veto |
| Dev | PR uniquement |
| Analyst | Lecture |
| Ops | Scripts / infra |

## 7) Règles de contribution (CTO gate)
Toute contribution doit :
- respecter le cahier canonique
- être isolée (pas de refactor global)
- inclure un test de validation
- ne pas activer l'exécution sans autorisation CTO

👉 Toute PR non conforme est rejetée sans débat.

## 8) Livrables attendus par phase (récapitulatif)
| Phase | Livrables clés |
|-------|-----------------|
| 2.x | EventScore v1 stable |
| 2.x | OpenClaw pondération |
| 3.1 | Trading altcoins live |
| 3.2 | BTC / XAU pipelines |
| 4 | Options / Arbitrage |

## 9) Conclusion CTO
Ce plan :
- rend le projet collaboratif sans dilution
- protège l'intégrité quant
- permet une montée en puissance progressive
- prépare le live sans brûler d'étapes
