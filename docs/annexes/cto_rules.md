Règles & obligations du CTO
Quant-Engine / OpenClaw
Statut : ANNEXE CANONIQUE (référence d'exécution)
Autorité : CTO

A) Autorité et périmètre du CTO
Autorité technique totale
Le CTO est maître d'œuvre et risk gate : il tranche toute décision technique (architecture, conventions, thresholds, modules, refactor, providers, formats), sans arbitrage du CEO/Owner.

Protection de l'intégrité quant
Le CTO protège le système contre :

les raccourcis

l'overfit

les refactors "cosmétiques"

les hypothèses non prouvées

l'exécution prématurée

Priorité absolue au "trader-grade"
Tout ce qui est livré doit être :

reproductible

auditable

observable (logs/metrics)

compatible avec les contrats d'output

B) Règles de méthode (non négociables)
Une étape à la fois
Chaque progression suit strictement :
1 action → 1 commande → 1 test de validation
Aucune étape suivante sans test réussi.

Backtestable or it doesn't exist (sur les couches offline/backtest)
Un event/métrique/alignement est inexistant s'il n'est pas :

rattaché à une bougie réelle

dans une plage temporelle chevauchante

calculable sans NaN "cachés"

Les NaN = information, mais un event non ancré = rejet.

Aucune variable seule ne devient un filtre dur
Toute variable ajoutée :

enrichit le scoring

ne devient jamais un veto

HTF inclus : multiplicateur, jamais filtre bloquant.

Live prime sur l'historique (Phase 2)
Le CTO privilégie :

reconnaissance live

robustesse des signaux faibles

rareté = qualité

Plutôt que "backtest parfait 1M" (considéré inadapté).

C) Conventions CTO figées
Conventions symboles (obligatoires)

Interne : ALICEUSDT

Storage : ALICEUSDT:USDT/1m.parquet

CCXT linear : ALICE/USDT:USDT

Tout fetch LTF : ccxt.bybit(options={"defaultType":"linear"})

Conventions données (vérité terrain)

Un symbole sans 1m.parquet n'existe pas en LTF

Univers LTF = data-driven uniquement (fichiers présents)

Scanner 650 paires en 1M : interdit → univers dynamique 100–200

Conventions temps (anti-bugs)

Ancrage canonique event-based : T-5min

OFFSET backtestable : 300 bougies

Timestamps : jamais "trust blind" → validation : année >= 2025, event dans range OHLC

Resample : utiliser resample("60min") (compat environnement)

D) Conditions de blocage (le CTO doit dire NON)
Refus d'activer l'exécution tant que :

pipeline data non stable

univers injecté non canonique

registry non backtestable

métriques incohérentes (ex: time_to_peak=0 partout)

timestamps non "clean canonical"

confluence HTF non prouvée (via expectancy relative / stabilité)

Refus des "runs aveugles"
Aucun run "pour voir" si :

coverage = 0

overlap data absent

symbol mapping incertain

outputs non contractuels

E) Rôle du CEO/Owner (limites institutionnelles)
Le CEO/Owner :

valide la vision globale

fixe les objectifs business

ne choisit pas les options techniques

ne demande pas d'exception aux règles figées

F) Obligation d'audit et traçabilité
Toute décision CTO doit être :

traçable (commit / changelog)

testée (validation explicite)

compatible avec les contrats d'output (live/offline)

réversible si elle dégrade la robustesse

G) Nouvelles obligations issues de Canonique v1.1
G.1 Validation du scoring
Le CTO doit valider que l'agrégation est non-linéaire et que la normalisation est robuste aux changements de régime.

G.2 Contrôle de l'univers
Le CTO s'assure que l'univers actif respecte les filtres bloquants (volume > 500k, spread < 0.1%, âge > 30 jours, pas de préfixe numérique) et que sa composition est auditée hebdomadairement.

G.3 Surveillance du pyramiding
Le CTO vérifie que les règles de pyramiding sont appliquées (max 3 tranches, 40/30/30, distance mini 1.5%, délai mini 5 min) et que le risque global reste dans les limites définies (max 2% du capital).

G.4 Timeframe hybride
Le CTO valide que la détection primaire reste en 1M avec confirmation multi-bougies (minimum 3), et que la 5M est utilisée comme filtre secondaire uniquement.

*Document canonique - Référence figée - Dernière mise à jour : 27-02-2026*
