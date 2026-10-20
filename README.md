# Prévision de la demande d'électricité en Allemagne (2015–2025)

Modélisation économétrique de séries temporelles et prévision à 6 mois de la demande mensuelle d'électricité en Allemagne, avec R.

Projet réalisé dans le cadre du Master Ingénierie des Risques Économiques et Financiers – Economic Risk and Data Sciences (Université de Bordeaux).

## Objectif

Identifier le processus générateur de la demande d'électricité allemande, une série marquée par une forte saisonnalité et par plusieurs chocs (Covid-19, crise énergétique de 2022, sortie du nucléaire en 2023), puis produire une prévision à 6 mois avec un intervalle de confiance à 95 %.

## Données

- **Source :** Ember, *Monthly Electricity Data* (voir `data/README.md`).
- **Série :** demande d'électricité mensuelle en Allemagne, en TWh.
- **Période :** janvier 2015 – novembre 2025, soit 131 observations.
- **Contrôles qualité :** aucune valeur manquante, aucun doublon de date, aucun mois manquant.

## Méthodologie

1. **Analyse exploratoire :** chronogramme, décomposition multiplicative, monthplot. La série présente une saisonnalité annuelle stable, avec une demande plus élevée en hiver.
2. **Différenciation saisonnière** d'ordre 12 pour retirer la saisonnalité.
3. **Tests de racine unitaire :**
   - Dickey-Fuller et Dickey-Fuller augmenté, avec un nombre de retards choisi par le critère MAIC ;
   - Zivot-Andrews, avec une rupture endogène détectée en août 2022 (crise énergétique) ;
   - Lee-Strazicich (2003), avec une rupture détectée en mars 2024. La statistique de test vaut −5,57 pour une valeur critique de −4,51 à 5 % : la série est stationnaire avec rupture.
4. **Identification et estimation :** ACF, PACF, EACF et test de Ljung-Box, puis élimination itérative des coefficients non significatifs.
   - Série différenciée : ARMA(4,12) contraint, dont seuls la constante, le terme AR(1) et le terme MA(12) sont conservés.
   - Série brute : SARIMA(4,0,12)(2,1,1)₁₂ contraint.
5. **Validation des résidus :** Ljung-Box (autocorrélation), Student (moyenne nulle), ARCH-LM d'Engle de 1 à 30 retards (hétéroscédasticité), Jarque-Bera (normalité).
6. **Prévision à 6 mois** (décembre 2025 – mai 2026) avec un intervalle de confiance à 95 %.

## Résultats

| Modèle | Ljung-Box (p) | ARCH-LM | Jarque-Bera (p) | Conclusion |
|---|---|---|---|---|
| ARMA contraint, série différenciée | 0,60 | aucun effet ARCH | 0,25 | Résidus assimilables à un bruit blanc gaussien |
| SARIMA(4,0,12)(2,1,1)₁₂, série brute | 0,46 | effets ARCH aux retards 1 à 3 | 0,50 | Moyenne et saisonnalité bien captées, variance non modélisée |

- La prévision SARIMA donne une demande comprise entre environ 42 et 49 TWh par mois, avec un pic en janvier 2026 puis une baisse progressive jusqu'en mai 2026.
- Les ruptures détectées correspondent à la crise énergétique de 2022 et au recul de la demande industrielle en 2024.

## Limites et pistes

- L'hétéroscédasticité conditionnelle des résidus du SARIMA appellerait une modélisation de la variance (ARCH/GARCH). Les intervalles de prévision sont donc à interpréter avec prudence.
- La performance de prévision n'est pas évaluée hors échantillon. Une piste serait de réserver les 12 derniers mois et de comparer les modèles par RMSE et MAPE.

## Reproduire l'analyse

1. Installer R (version 4.2 ou plus récente) et RStudio.
2. Installer les packages :
   ```r
   install.packages(c("tidyverse", "urca", "CADFtest", "forecast", "lmtest", "zoo",
                      "foreach", "doSNOW", "TSA", "FinTS"))
   ```
3. Placer les données dans `data/` (voir `data/README.md`) et la fonction du test de Lee-Strazicich dans `R/LeeStrazicichUnitRootTest.R`.
4. Ouvrir `demande_electricite_allemagne.Rmd` dans RStudio et cliquer sur **Knit**.

## Structure du dépôt

```
├── demande_electricite_allemagne.Rmd   # Analyse complète (R Markdown)
├── projetST1.css                       # Mise en forme du rapport HTML
├── R/
│   └── LeeStrazicichUnitRootTest.R     # Test de Lee-Strazicich (2003)
└── data/
    └── README.md                       # Instructions de téléchargement des données
```

## Outils

R, R Markdown, tidyverse, forecast, urca, CADFtest, lmtest, TSA, FinTS.

## Auteur

**Jules Marin** – [LinkedIn](https://www.linkedin.com/in/jules-marin-64611035a/)
