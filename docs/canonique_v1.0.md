📘 Quant-Engine / OpenClaw — Canonique Master Spec
Référence officielle (figée)
 Dernière consolidation : 26-02-2026 (America/Montreal)
 Portée : PHASE 1 (Infra/Data/Pipeline) + PHASE 2 v1.1 (Event Recognition Live Pump & Dump)

0) Statut, autorité, non-négociables
0.1 Statut
Ce document est la source unique de vérité pour le projet Quant-Engine / OpenClaw.
 Toute évolution future doit être :
compatible avec ce document, ou


explicitement justifiée (audit + changelog).


0.2 Principes non négociables
Reconnaître ≠ prédire


Event ≠ Trade


Screener ≠ Backtester


HTF ≠ veto (jamais filtre bloquant)


Aucune variable seule n’est suffisante


“Backtestable or it doesn’t exist” (sur les couches qui backtestent)


Les NaN sont de l’information, pas une erreur


0 event = information valide


Live > historique (Phase 2)


0.3 Gouvernance CTO vs Owner (institutionnalisée)
CTO (définitif) :
impose conventions (symbols, storage, time anchoring)


anticipe pannes structurelles (CCXT/Bybit, fetch limits, offsets, timestamps)


valide chaque étape via : 1 action / 1 test


refuse activation exécution tant que :


data pipeline stable


registry backtestable


metrics cohérentes


confluence HTF prouvée (gain d’expectancy)


CEO/Owner :
valide vision globale


n’oriente pas le code


n’arbitre pas la technique



PARTIE A — PHASE 1 (Canonique)
1) Rôle et périmètre (Phase 1)
Objectifs :
décrire l’architecture réelle


documenter le pipeline tel qu’il tourne


empêcher dérive / refactor inutile


éviter répétition des erreurs rencontrées



2) Infrastructure réelle
2.1 Environnement
VPS : Hostinger


OS : Ubuntu (serveur)


User : root


Mode : always-on


Python : virtualenv .venv


Exécution : CLI (pas de systemd à ce stade)


2.2 Racine projet
/root/quant-engine/
 Tout lancement doit se faire depuis cette racine.
2.3 Règle PYTHONPATH (figée, critique)
Tous les runners doivent être lancés avec :
PYTHONPATH=. python <script>


❌ Lancer sans PYTHONPATH=. est interdit.

3) Arborescence canonique
quant-engine/
│
├── engine/
│   ├── screener/
│   │   ├── screener_1m.py
│   │   ├── run_screener.py
│   │   └── __init__.py
│   │
│   ├── features/
│   │   ├── regime_shift.py
│   │   └── ...
│   │
│   ├── universe/
│   │   └── alt_sampling_plan.py
│   │
│   └── ...
│
├── datasets/
│   ├── raw/
│   │   └── ccxt/
│   │       ├── ALICEUSDT:USDT/
│   │       │   └── 1m.parquet
│   │       ├── AAVEUSDT:USDT/
│   │       │   └── 1m.parquet
│   │       └── ...
│   │
│   ├── universe/
│   │   └── universe_active.parquet
│   │
│   ├── screener/
│   │   ├── scan_latest.parquet
│   │   └── top_20.parquet
│   │
│   └── events/
│       ├── event_registry_*.parquet
│       └── ...
│
└── .venv/

4) Données de marché (vérité terrain)
4.1 Source
Exchange : Bybit


Librairie : CCXT


Timeframe critique : 1 minute (LTF)


4.2 Format OHLCV (canonique)
Storage : Parquet


Un fichier par symbole :
 datasets/raw/ccxt/<SYMBOL>:USDT/1m.parquet


Ex :
 datasets/raw/ccxt/ALICEUSDT:USDT/1m.parquet
4.3 Règle CTO (critique)
Un symbole sans fichier 1m.parquet n’existe pas pour le LTF.
 ❌ aucun fallback, aucune approximation.

5) Canon symbol & mapping Bybit / CCXT (Addendum figé)
3 représentations distinctes (obligatoire) :
Interne Quant-Engine : ALICEUSDT


Storage dataset : ALICEUSDT:USDT/1m.parquet


CCXT Bybit linear perps : ALICE/USDT:USDT


Décision CTO : tout fetch LTF en CCXT utilise :
exchange = ccxt.bybit(options={"defaultType":"linear"})


symbol format : BASE/USDT:USDT


Cause racine rencontrée :
 erreurs “market symbol …/USDT not found” = mauvais marché (spot/inverse) ou mauvais format.

6) Données LTF — profondeur & limites CCXT (Addendum figé)
Fenêtre effective initiale : ~1000 bougies (~16h) → insuffisant


Limite CCXT : fetch_ohlcv ≈ 1000 bougies / appel
 ✅ Pagination obligatoire pour 72h


Règle canonique :
refetch 72h = boucle since + limit=1000 jusqu’à couvrir 3 jours



7) Offset backtestable (anti edge-of-data) (Addendum figé)
Objectif : garantir un futur observable pour MFE/MAE.
Règle CTO figée :
OFFSET = 300 bougies (~5h) sur le 1m


IMPORTANT :
la base HTF lit la dataset 72h complète, pas l’offset


l’offset sert au backtest event-based (éviter fin de dataset)



8) Univers LTF (règles canoniques)
8.1 Univers LTF de vérité
Généré dynamiquement depuis les fichiers réellement présents.
 Fichier de vérité :
datasets/universe_ltf_1m.parquet (contient uniquement symbol)


8.2 Univers injecté dans le screener
Le screener ne choisit pas son univers.
 Il lit :
datasets/universe/universe_active.parquet


Ce fichier est injecté avant chaque run.
8.3 Erreur architecturale identifiée (corrigée)
Avant :
univers “logique”


données 1m incomplètes


0 event backtestable


Après :
 ✅ univers data-driven uniquement

9) Screener 1m — fonctionnement réel
9.1 Point d’entrée
engine/screener/run_screener.py
 def run_screener():
Le screener :
est global


n’est PAS callable par symbole


écrit dans datasets/screener/


9.2 screener_1m.py
compute_screener_features(df)


calcul scores LTF


9.3 Scores / champs calculés (canon)
S_comp : compression range


S_ign : ignition / breakout


S_opp : opportunité relative


Score : agrégé


persist : persistance


event_flag : détection event (1 = event)


⚠️ (Phase 1 historique) : event_flag==1 définissait l’event.
 ➡️ (Phase 2 v1.1) : on migre vers EventScore continu (voir Partie B).

10) Outputs screener (contractuels Phase 1)
datasets/screener/scan_latest.parquet


datasets/screener/top_20.parquet


scan_latest.parquet = source unique pour générer le registry LTF.

11) Registry d’events LTF (Phase 1)
11.1 Création
À partir de scan_latest.parquet via filtrage.
11.2 Normalisation symboles (figée)
Format interne :
ALICEUSDT, AAVEUSDT, 0GUSDT


❌ jamais :USDT
 ❌ jamais USDTUSDT
Conversion CCXT dataset :
f"{symbol}:USDT"



12) Event detection & backtest — règle d’ancrage canonique (Addendum figé)
Résultat quant clé :
edge avant l’event


Comparaison :
T : expectancy ≈ 0 / négative


T-5 : expectancy_mean ≈ +0.0087, winrate ≈ 66%


T-10 : non exploitable (hors fenêtre / non généré)


Décision CTO figée :
ancrage canonique = T-5 minutes


event_anchor_ts = event_timestamp - 5min


Tous les backtests événementiels utilisent cette règle.

13) Timestamp hygiene — incident 1970 (Addendum figé)
Incident :
event_candle_ts parsé en 1970 → HTF OUT_OF_RANGE 50/50


Décision CTO :
ne jamais faire confiance à un champ event mal typé


reconstruire event_candle_ts depuis :


timestamp + règle canonique T-5


validation systématique :


année min >= 2025


event dans range OHLC (min/max)



14) Pandas resample — compat fréquence (Addendum figé)
Dans cet environnement :
resample("1H") et resample("60T") ont échoué


Règle CTO canonique :
utiliser resample("60min")



15) Confluence HTF — règles posées (bloquée tant que timestamps non clean)
Tag minimal défini :
htf_trend = +1 si close_1h > EMA50_1h, sinon -1


Prérequis CTO :
HTF calculé depuis 72h (pas offset)


timestamps strictement alignés (sinon OUT_OF_RANGE)


État :
HTF tagging bloqué tant que event_candle_ts n’est pas reconstruit proprement via T-5.



16) Erreurs rencontrées (Phase 1) — cause réelle
Cause :
screener hors univers 1m réel


symboles non normalisés


périodes non chevauchantes


➡️ erreur d’ordre de pipeline (pas un bug pandas/temps).

17) Statut Phase 1
✅ opérationnel :
infrastructure stable


données 1m présentes


screener fonctionnel


univers LTF canonique


outputs cohérents


⏳ ensuite :
registry LTF backtestable


alignement event → bougie


MFE/MAE


agrégations


OpenClaw (optimisation/ranking)



PARTIE B — PHASE 2 v1.1 (Canonique)
18) Vision & objectif fondamental
18.1 Problème réel
Les pump & dump crypto :
surviennent 5 à 10 fois/jour


sur ~650 paires Bybit


rapides, asymétriques, non stationnaires


difficiles/impossibles à backtester proprement en 1M
 ➡️ backtest-first inadapté


18.2 Objectif Phase 2 (verrouillé)
Construire un moteur live (1M) capable de :
détecter transitions de régime


identifier événements à fort potentiel


produire un EventScore continu


sans exiger validation backtest stricte en 1M


Le système ne prédit pas : il reconnaît.

19) Définition canonique d’un Event Pump & Dump
19.1 Ce qu’un Event N’EST PAS
❌ trade
 ❌ signal d’entrée
 ❌ règle directionnelle
 ❌ confirmation HTF
 ❌ vérité statistique
19.2 Ce qu’un Event EST (canon)
Un Event = instant LTF (1M) où le marché montre :
anomalie structurelle significative


début possible de pump ou dump


indépendant du résultat final


Un Event peut réussir/échouer/avorter.
 Son existence ne dépend pas de son succès.
19.3 Définition “changement de régime” (figée)
Un Event P&D = changement de régime 1M caractérisé par :
rupture micro-structure (CHOCH)


accélération non linéaire volatilité


modification comportement EMA


persistance minimale


asymétrie exploitable potentielle


❌ pas défini par un %
 ❌ pas validé par un résultat

20) Principe central : EventScore (scoring continu)
20.1 Décision architecturale clé (figée)
Le screener :
❌ ne fait pas event/no-event binaire


✅ produit un EventScore continu (ex : 0 à 100)


Mesure :
“À quel point l’instant ressemble à un début de pump/dump”
20.2 Logique de confluence (non négociable)
❌ aucune variable seule ne suffit


✅ convergence de signaux faibles → event fort


✅ score = agrégation de familles indépendantes



21) Variables LTF (1M) — cœur du moteur
21.1 Variables structurelles obligatoires
Compression
contraction range


baisse relative volatilité


phase de charge


Ignition
expansion soudaine range


breakout micro-structurel


impulsion initiale


Opportunité asymétrique
MFE attendu > MAE


mouvement déséquilibré


non directionnel


Persistance minimale
pas un spike unique


continuation courte exploitable


21.2 Variables avancées (Phase 2 — clés)
Positionnement 3 EMA
structure relative rapide/médiane/lente


resserrement → charge


retournement brutal → déclencheur


non directionnel (structurel)


CHOCH
rupture micro-structure


changement comportement


signal précoce (pas confirmation)


Accélération volatilité
delta ATR


convexité range


accélération non linéaire


Volume relatif & vélocité
vitesse d’augmentation volume


ratio volume instant / moyenne


déséquilibre soudain



22) Rôle du HTF (repositionné, figé)
22.1 Décision canonique
HTF :
❌ jamais filtre bloquant


✅ élément de scoring


✅ multiplicateur de contexte


✅ facteur probabilité relative


22.2 Intégration
HTF aligné → +X points


HTF opposé → −Y points


HTF neutre → 0


Un event LTF peut exister contre le HTF.

23) Univers Bybit — contrainte opérationnelle
23.1 Problème
Scanner 650 paires en 1M = bruit + coût + inefficacité.
23.2 Univers dynamique restreint (figé)
Univers cible : 100 à 200 paires
 But : augmenter proba de capter events rares.
 Sélection basée sur :
activité récente


volatilité


volume


changements récents / régime shift


Scanning LTF intensif uniquement sur cet univers.

24) Architecture LIVE vs OFFLINE (séparation stricte)
24.1 Live Engine (1M)
détection temps réel


calcul EventScore


ranking événements


alerting / décision humaine ou semi-auto


❌ pas d’exigence backtest strict 1M


24.2 Offline (OpenClaw)
replay historique


backtests multi-TF : 5M / 15M / 1H


analyse MFE/MAE


estimation contributions variables


pondération poids + rejet fragile


OpenClaw pondère, il ne juge pas l’existence d’un event.

25) Risk & Trade Management (figé)
25.1 Principe
Ces règles servent à :
évaluer tradabilité (TradeScore)


mesurer expectancy/drawdown


normaliser exécution


Elles ne servent pas à “créer” l’event.
25.2 Stop-Loss
SL = ATR dynamique ou structure (swing / OB)


distance max : 1–2%


25.3 Break-even
à +0.4–0.6% underlying ou +1R


buffer frais inclus


25.4 Take Profit
target principal : +10%


option :


50% TP partiel


trailing ATR (chandelier)



26) Execution Engine (exigences canoniques)
ordres market contrôlés


SL/TP côté exchange


reduce-only


reconnexion WS automatique


journal exécution complet



27) Dashboard Web privé (obligatoire)
Fonctionnalités :
login sécurisé


Top 5 opportunités


scores HTF/LTF/SMC


trades actifs


historique & statistiques


état du bot



28) Alertes Telegram (obligatoire)
nouveau signal validé


trade exécuté


SL/BE/TP touché


erreurs critiques


bot down



29) Backtesting & optimisation (OpenClaw)
29.1 Backtests
event-based


slippage & fees


walk-forward


out-of-sample


29.2 Sélection automatique
Critères :
win rate


drawdown


expectancy


stabilité


OpenClaw rejette toute stratégie fragile.

30) Contrats d’output (contractuels)
30.1 Live
events scorés (ranking)


top opportunités


logs structurés


alertes


30.2 Offline
rapports pondération


métriques par TF


stabilité/fragilité


recommandations de poids (non obligatoires)



PARTIE C — PIPELINE EVENT-BASED (VALIDÉ)
31) Registry & métriques (multi-events)
Production registry LTF multi-events depuis scan_latest_q70.parquet
 Batch metrics validées :
event_candle_ts


mfe_up, mae_dn


time_to_peak_min


Signal d’alerte :
si time_to_peak_min = 0 partout → ancrage trop tard (confirmé T vs T-5)



PARTIE D — CONCLUSION CANONIQUE
32) Conclusion
Phase 1 garantit que la donnée est vraie.


Phase 2 v1.1 garantit que le système est utile et exploitable en live.


Ce document est la référence officielle figée du projet Quant-Engine / OpenClaw.

ANNEXE — Règles & obligations du CTO (Canonique, figé)
A) Autorité et périmètre du CTO
Autorité technique totale
 Le CTO est maître d’œuvre et risk gate : il tranche toute décision technique (architecture, conventions, thresholds, modules, refactor, providers, formats), sans arbitrage du CEO/Owner.


Protection de l’intégrité quant
 Le CTO protège le système contre :


les raccourcis,


l’overfit,


les refactors “cosmétiques”,


les hypothèses non prouvées,


l’exécution prématurée.


Priorité absolue au “trader-grade”
 Tout ce qui est livré doit être :


reproductible,


auditable,


observable (logs/metrics),


compatible avec les contrats d’output.



B) Règles de méthode (non négociables)
Une étape à la fois
 Chaque progression suit strictement :


1 action


1 commande


1 test de validation
 Aucune étape suivante sans test réussi.


Backtestable or it doesn’t exist (sur les couches offline/backtest)
 Un event/métrique/alignement est inexistant s’il n’est pas :


rattaché à une bougie réelle,


dans une plage temporelle chevauchante,


calculable sans NaN “cachés”.
 Les NaN = information, mais un event non ancré = rejet.


Aucune variable seule ne devient un filtre dur
 Toute variable ajoutée :


enrichit le scoring


ne devient jamais un veto
 HTF inclus : multiplicateur, jamais filtre bloquant.


Live prime sur l’historique (Phase 2)
 Le CTO privilégie :


reconnaissance live,


robustesse des signaux faibles,


rareté = qualité,
 plutôt que “backtest parfait 1M” (considéré inadapté).



C) Conventions CTO figées (doivent être appliquées partout)
Conventions symboles (obligatoires)


Interne : ALICEUSDT


Storage : ALICEUSDT:USDT/1m.parquet


CCXT linear : ALICE/USDT:USDT
 Tout fetch LTF : ccxt.bybit(options={"defaultType":"linear"})


Conventions données (vérité terrain)


Un symbole sans 1m.parquet n’existe pas en LTF.


Univers LTF = data-driven uniquement (fichiers présents).


Scanner 650 paires en 1M : interdit → univers dynamique 100–200.


Conventions temps (anti-bugs)


Ancrage canonique event-based : T-5min


OFFSET backtestable : 300 bougies


Timestamps : jamais “trust blind” → validation :


année >= 2025


event dans range OHLC


Resample : utiliser resample("60min") (compat environnement)



D) Conditions de blocage (le CTO doit dire NON)
Refus d’activer l’exécution tant que :


pipeline data non stable,


univers injecté non canonique,


registry non backtestable,


métriques incohérentes (ex: time_to_peak=0 partout),


timestamps non “clean canonical”,


confluence HTF non prouvée (via expectancy relative / stabilité).


Refus des “runs aveugles”
 Aucun run “pour voir” si :


coverage = 0,


overlap data absent,


symbol mapping incertain,


outputs non contractuels.



E) Rôle du CEO/Owner (limites institutionnelles)
Le CEO/Owner :


valide la vision globale,


fixe les objectifs business,


ne choisit pas les options techniques,


ne demande pas d’exception aux règles figées.



F) Obligation d’audit et traçabilité
Toute décision CTO doit être :


traçable (commit / changelog),


testée (validation explicite),


compatible avec les contrats d’output (live/offline),


réversible si elle dégrade la robustesse.


ANNEXE — Plan de développement technologique
Quant-Engine / OpenClaw
 Statut : ANNEXE CANONIQUE (référence d’exécution)
 Autorité : CTO

1) Objectif de cette annexe
Cette annexe a pour but de :
structurer l’exécution technique du projet,


définir les phases de développement successives,


préciser les livrables attendus par phase,


organiser le travail d’équipe via GitHub,


empêcher toute dérive, implémentation prématurée ou mélange des responsabilités.


👉 Ce document n’ajoute aucune logique métier :
 il organise comment on implémente ce qui est déjà canonique.

2) Découpage global des phases
Vue d’ensemble
Phase
Objectif principal
Statut
Phase 1
Pipeline data & screener LTF
✅ Terminée
Phase 2 v1.1
Event Recognition Engine (altcoins)
🔒 Canonique
Phase 2.x
Stabilisation & enrichissement EventScore
⏳ À développer
Phase 3
Trading live structuré & multi-actifs
⏳ À venir
Phase 4
Extensions avancées (options, arbitrage)
🔮 Hors scope immédiat


3) Phase 2.x — Développement technologique (prioritaire)
3.1 Objectif Phase 2.x
Stabiliser, formaliser et enrichir l’Event Recognition Engine sans introduire :
de filtres durs,


de dépendance excessive au backtest 1M,


de coupling avec l’exécution.



3.2 Modules à développer (ordre imposé par le CTO)
MODULE 1 — EventScore v1 (fondation)
Sections canonique liées : §20, §21, §22
Livrables
module event_score_v1.py


définition formelle des familles de variables


normalisation des sous-scores


agrégation continue (0–100)


Règles
aucun seuil binaire


aucune dépendance à un trigger


compatible replay & live


Validation
score stable sur données neutres


distribution non dégénérée


aucun event “fantôme”



MODULE 2 — Enrichissement des variables LTF
Sections liées : §21.2
Variables à intégrer progressivement :
candle patterns (engulfing, pin, etc.)


liquidity sweep


inverted fair value gap


open interest (LTF / HTF)


Règles
score marginal faible


confluence uniquement


désactivable sans casser le moteur


Validation
gain d’expectancy mesuré offline


aucune création d’event isolée



MODULE 3 — Univers dynamique (100–200 paires)
Sections liées : §23
Livrables
score d’éligibilité univers


règles d’entrée/sortie


inertie contrôlée (anti-churn)


Validation
univers stable sur 24–72h


couverture d’events rare > univers complet



MODULE 4 — OpenClaw (offline learning)
Sections liées : §24, §29
Fonctions
backtests event-based multi-TF


mesure contribution marginale variables


rejet automatique des combinaisons fragiles


recommandations de poids


Validation
reproductibilité


stabilité inter-régimes


pas d’optimisation opportuniste



4) Phase 3 — Trading live & multi-actifs
4.1 Phase 3.1 — Event → Trade (altcoins)
Sections liées : §25, §26
Livrables
TradeScore formel


RiskScore (SL, BE, TP, DD)


Execution Engine stable


pyramiding contrôlé


Conditions CTO de déblocage
EventScore stable


registry backtestable


kill-switch actif


logs complets



4.2 Phase 3.2 — Pipelines parallèles BTC & XAU
Principe
 BTC et Gold ne partagent PAS la logique pump & dump.
Livrables
pipeline dédié BTC


pipeline dédié XAU


backtesting complet (non événementiel)


Règle CTO
 ❌ aucune réutilisation forcée du modèle altcoins

5) Phase ultérieure — Extensions (non prioritaires)
5.1 Options trading scanner
phase post-stabilisation


architecture dédiée


métriques spécifiques (IV, Greeks)


5.2 Arbitrage Polymarket / Kalshi
projet indépendant


agents spécialisés


backtest obligatoire avant live



6) Architecture GitHub (obligatoire)
6.1 Création du repository
Nom recommandé
quant-engine-openclaw
Visibilité
Privé (initialement)


Accès contrôlé par rôle



6.2 Structure du repo
quant-engine-openclaw/
│
├── engine/              # moteur principal
├── openclaw/            # offline learning
├── execution/           # trade & risk engine
├── datasets/            # ignoré (gitignore)
├── docs/
│   ├── canonique.md
│   ├── annexes/
│   │   ├── dev_plan.md
│   │   └── cto_rules.md
│
├── tests/
├── scripts/
├── .gitignore
├── README.md

6.3 Règles Git (figées)
branche main protégée


PR obligatoire


aucun commit direct sur main


1 PR = 1 feature ou 1 fix


tests requis avant merge



6.4 Rôles GitHub
Rôle
Droits
CTO
Merge final, veto
Dev
PR uniquement
Analyst
Lecture
Ops
Scripts / infra


7) Règles de contribution (CTO gate)
Toute contribution doit :
respecter le cahier canonique,


être isolée (pas de refactor global),


inclure un test de validation,


ne pas activer l’exécution sans autorisation CTO.


👉 Toute PR non conforme est rejetée sans débat.

8) Livrables attendus par phase (récapitulatif)
Phase
Livrables clés
2.x
EventScore v1 stable
2.x
OpenClaw pondération
3.1
Trading altcoins live
3.2
BTC / XAU pipelines
4
Options / Arbitrage


9) Conclusion CTO
Ce plan :
rend le projet collaboratif sans dilution,


protège l’intégrité quant,


permet une montée en puissance progressive,


prépare le live sans brûler d’étapes.
