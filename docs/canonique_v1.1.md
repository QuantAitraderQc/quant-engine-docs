1. Vision d'ensemble : Vers Canonique v1.1
1.1 Ce que nous avons construit ensemble
Nos discussions ont enrichi le projet sur 4 piliers fondamentaux :

Pilier	Sujets abordés	Impact
Scoring & Détection	Agrégation non-linéaire, normalisation, variables à mémoire, EventState vectoriel	Précision + interprétabilité
Filtrage univers	Volume, spread, âge listing, memecoins, univers dynamique 150-200	Efficacité + qualité
Timeframe	1M vs 5M, confirmation multi-bougies, approche hybride	Réactivité + robustesse
Risk Management	Pyramiding, stops par tranche, réduction risque initial	Survie + optimisation
1.2 Structure proposée pour Canonique v1.1
text
PARTIE A — PHASE 1 (Canonique) [inchangée]
PARTIE B — PHASE 2 v1.1 (Existante)
    ↓ Ajout de sections
PARTIE B+ — PHASE 2 v1.1 enrichie (Nouvelle)
    §32. Spécification formelle de l'EventScore
    §33. Normalisation et agrégation
    §34. Variables à mémoire et détection précoce
    §35. Gestion de l'univers dynamique
    §36. Timeframe hybride 1M/5M
    §37. Pyramiding et gestion du risque
PARTIE C — PIPELINE EVENT-BASED [inchangé]
PARTIE D — CONCLUSION [inchangée]
ANNEXES — mises à jour
2. Synthèse détaillée des recommandations
2.1 Système de scoring (EventScore)
Problèmes identifiés
Agrégation sous-spécifiée (risque de moyenne simple)

Normalisation non définie (échelles hétérogènes)

Absence de mémoire temporelle

Recommandations
R1. Agrégation non-linéaire obligatoire

python
EventScore = (S_comp * S_ign * S_opp * S_persist)^(1/4) * (1 + α*HTF)
La moyenne géométrique force la confluence

R2. Normalisation par rang centile glissant

python
S_comp_norm = percentile(S_comp_raw, window=1000) / 100
Robuste aux changements de régime

R3. EventState vectoriel (optionnel)

python
EventState = {
    "regime": "compression|ignition|expansion",
    "confidence": 0.85,
    "asymmetry": float,  # positif = biais haussier
    "htf_context": "aligned|neutral|opposing"
}
2.2 Filtrage de l'univers
Problèmes identifiés
650 paires = bruit + coût calcul

Memecoins toxiques (préfixes numériques)

Paires non tradables (spot only, faible volume)

Recommandations
R4. Filtres bloquants obligatoires

python
UNIVERSE_MANDATORY_FILTERS = {
    "min_volume_24h_usd": 500000,
    "max_spread_percent": 0.1,
    "has_perpetual": True,
    "min_listing_age_days": 30,
    "exclude_numeric_prefix": True
}
R5. Score d'éligibilité pour classement

Volume (40%), spread (30%), âge (20%), corrélation BTC (10%)

Univers final : 120-150 paires

R6. Univers dynamique à deux vitesses

Core (100 paires) : rotation hebdomadaire

Satellite (50 paires) : rotation quotidienne

Watchlist (50 paires) : surveillance sans scan intensif

2.3 Timeframe hybride 1M/5M
Problèmes identifiés
1M pure = bruit microstructurel

5M pure = perte de la fenêtre d'entrée

Recommandations
R7. Détection primaire en 1M avec confirmation

python
CONFIRMATION_RULES = {
    "min_breakout_candles": 3,  # 3 bougies 1M au-dessus range
    "volume_confirm": "volume > MA20 * 2",
    "no_retest": "pas de retour sous range avant 5min"
}
R8. Filtrage secondaire en 5M

Validation que la cassure tient en 5M

Utilisé pour confirmer avant tranche 2 de pyramiding

R9. Délai d'entrée standardisé

Early warning : T+3min (surveillance)

Confirmation action : T+6min (entrée possible)

Validation structurelle : T+10-15min (pyramiding)

2.4 Early Warning System
R10. Score de pré-détection (EarlyWarningScore)

python
EARLY_WARNING_COMPONENTS = {
    "compression_extreme": 0.25,  # range/ATR < 0.5
    "absorption_orders": 0.20,    # bids > asks
    "divergence_ema": 0.20,       # EMA9/EMA20 croisé
    "volume_silencieux": 0.20,     # ordres passifs en hausse
    "volatilité_condensée": 0.15   # ATR(5)/ATR(50) < 0.4
}
R11. Surveillance prioritaire

Top 20 EarlyWarningScore → scan toutes les 15-30 secondes

Reste univers → scan toutes les 1-5 minutes

2.5 Pyramiding et gestion du risque
R12. Règles de pyramiding

python
PYRAMIDING_SPEC = {
    "max_tiers": 3,
    "sizing": [0.4, 0.3, 0.3],  # 40%, 30%, 30%
    "min_gain_between_tiers_pct": 1.5,
    "min_time_between_tiers_min": 5,
    "max_total_position_pct": 2.0  # du capital
}
R13. Gestion des stops par tranche

Chaque tranche a son propre stop initial (-2%)

Quand gain global > +4%, tous les stops au BE moyen

Stop suiveur pour la première tranche

R14. Calcul du Risk/Reward avec pyramiding

text
Risque initial = taille_tranche1 * stop_initial
Gain potentiel = position_moyenne * prix_cible
RR_effectif = gain_potentiel / risque_initial
3. Intégration au document canonique existant
3.1 Modifications section par section
PARTIE B — PHASE 2 v1.1 (existante)
§20. EventScore → Remplacer par :

20) EventScore : spécification formelle

20.1 Principe d'agrégation
L'EventScore est calculé par moyenne géométrique des composantes pour forcer la confluence :
EventScore = (∏_{i=1}^n S_i)^(1/n) * (1 + β*HTF_context)

20.2 Normalisation
Chaque variable brute est transformée en score [0,1] via normalisation par rang centile sur fenêtre glissante de 1000 périodes.

20.3 Composantes obligatoires

S_comp : compression relative

S_ign : ignition/breakout

S_opp : opportunité asymétrique

S_persist : persistance minimale

Nouveau §21 bis → Ajouter :

21 bis) Early Warning System

21 bis.1 Score de pré-détection
Un EarlyWarningScore (0-100) est calculé toutes les 5 minutes sur l'ensemble de l'univers pour identifier les phases de charge.

21 bis.2 Surveillance prioritaire
Les 20 paires avec le meilleur EarlyWarningScore sont scannées en continu (intervalle 15-30s).

NOUVELLE SECTION — PARTIE B+ (après §31)
PARTIE B+ — PHASE 2 v1.1 enrichie

32) Gestion de l'univers dynamique

32.1 Filtres bloquants
Toute paire doit satisfaire : volume > 500k USD/jour, spread < 0.1%, perp existant, âge > 30 jours, pas de préfixe numérique.

32.2 Score d'éligibilité
Les paires sont classées selon : volume (40%), spread (30%), âge (20%), corrélation BTC (10%).

32.3 Univers cible
L'univers actif est limité à 120-150 paires, renouvelées hebdomadairement pour le core (100 paires) et quotidiennement pour le satellite (50 paires).

33) Timeframe hybride 1M/5M

33.1 Principe
La détection primaire reste en 1M (timeframe native du phénomène). La 5M est utilisée comme filtre de confirmation secondaire.

33.2 Règles de confirmation

Minimum 3 bougies 1M consécutives au-dessus du range

Volume > moyenne 20 * 2

Pas de retour sous le range avant 5 minutes

33.3 Délais canoniques

Early warning : T+3min

Confirmation action : T+6min

Validation structurelle : T+10-15min

34) Pyramiding et gestion du risque

34.1 Justification
Le pyramiding réduit le risque initial de 50-60% tout en permettant de capturer l'essentiel du mouvement.

34.2 Règles

Maximum 3 tranches : 40%, 30%, 30%

Distance minimale entre tranches : +1.5%

Délai minimal : 5 minutes

Taille maximale position : 2% du capital

34.3 Gestion des stops

Stops individuels par tranche (-2% initial)

Passage en BE global à +4%

Stop suiveur pour première tranche

3.2 Mise à jour des annexes
ANNEXE — Règles & obligations du CTO → Ajouter :

G) Nouvelles obligations issues de Canonique v1.1

G.1 Validation du scoring
Le CTO doit valider que l'agrégation est non-linéaire et que la normalisation est robuste aux changements de régime.

G.2 Contrôle de l'univers
Le CTO s'assure que l'univers actif respecte les filtres bloquants et que sa composition est auditée hebdomadairement.

G.3 Surveillance du pyramiding
Le CTO vérifie que les règles de pyramiding sont appliquées et que le risque global reste dans les limites définies.

ANNEXE — Plan de développement → Ajouter au Module 1 :

Module 1 enrichi — EventScore v1.1

Implémentation de l'agrégation non-linéaire

Normalisation par rang centile

Variables à mémoire (dérivées temporelles)

EarlyWarningScore (pré-détection)

4. Implémentation sur GitHub
4.1 Structure de fichiers recommandée
text
quant-engine-openclaw/
│
├── docs/
│   ├── canonique_v1.0.md          # Version originale (archivée)
│   ├── canonique_v1.1.md          # Nouvelle version (active)
│   └── annexes/
│       ├── cto_rules.md
│       ├── dev_plan.md
│       └── changelog.md           # NOUVEAU
│
├── engine/
│   ├── scoring/
│   │   ├── event_score.py         # NOUVEAU (agrégation non-linéaire)
│   │   ├── normalization.py       # NOUVEAU (rang centile)
│   │   └── early_warning.py       # NOUVEAU
│   │
│   ├── universe/
│   │   ├── pair_filter.py         # NOUVEAU
│   │   ├── eligibility_scorer.py  # NOUVEAU
│   │   └── dynamic_universe.py    # NOUVEAU
│   │
│   ├── risk/
│   │   ├── pyramiding.py          # NOUVEAU
│   │   └── position_sizing.py     # NOUVEAU
│   │
│   └── timeframe/
│       └── hybrid_confirmation.py # NOUVEAU
│
└── tests/
    ├── test_scoring.py
    ├── test_universe.py
    └── test_pyramiding.py
4.2 Processus de mise à jour
Étape 1 : Archive de la version actuelle

bash
cp docs/canonique.md docs/canonique_v1.0.md
git add docs/canonique_v1.0.md
git commit -m "docs: Archive version canonique v1.0"
Étape 2 : Création de la nouvelle version

bash
# Éditer docs/canonique_v1.1.md avec les modifications ci-dessus
git add docs/canonique_v1.1.md
git commit -m "docs: Ajoute canonique v1.1 avec scoring formel, early warning, pyramiding"
Étape 3 : Changelog

markdown
# Changelog

## v1.1 (2026-02-26)
### Ajouts majeurs
- Spécification formelle de l'EventScore (agrégation non-linéaire, normalisation par rang)
- Early Warning System pour détection précoce des phases de charge
- Filtrage avancé de l'univers (volume, spread, âge listing)
- Timeframe hybride 1M/5M avec règles de confirmation
- Pyramiding avec gestion de risque par tranche

### Modifications
- §20 : EventScore maintenant formellement spécifié
- Nouvelle §21 bis : Early Warning System
- Nouvelle PARTIE B+ : Gestion univers, timeframe hybride, pyramiding
- Annexes mises à jour avec obligations CTO supplémentaires
Étape 4 : Issues GitHub pour suivi
Créez des issues pour chaque module :

[FEATURE] EventScore v1.1 - Agrégation non-linéaire

[FEATURE] Early Warning System

[FEATURE] Filtrage univers dynamique

[FEATURE] Pyramiding engine

5. Calendrier d'intégration recommandé
Phase	Contenu	Délai
J+1	Mise à jour documentation (canonique v1.1)	1 jour
J+2	Création des issues GitHub	1 jour
Semaine 1	Implémentation EventScore + normalisation	5 jours
Semaine 2	Early Warning System + tests	5 jours
Semaine 3	Filtrage univers + intégration	5 jours
Semaine 4	Pyramiding + validation	5 jours
Semaine 5	Tests intégration & déploiement	5 jours
6. Conclusion
Cette synthèse représente l'évolution naturelle de votre document canonique. Elle :

Résout les zones de sous-spécification (scoring, normalisation)

Ajoute des fonctionnalités critiques (early warning, pyramiding)

Maintient la rigueur épistémique du document original

Structure l'implémentation sur GitHub

La version v1.1 devient ainsi un document opérationnellement complet tout en restant fidèle à la philosophie "reconnaître ≠ prédire" qui fait la force du projet.

Prochaine action : Créer le fichier docs/canonique_v1.1.md et initier les premières issues GitHub pour le scoring.
