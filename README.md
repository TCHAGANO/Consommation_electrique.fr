# Analyse et prévision de la consommation électrique en France

Étude de série temporelle quotidienne — approche SARIMA
Rapport d'analyse statistique et de modélisation
Source : Open Data RTE — Réseau de Transport d'Électricité
Période étudiée : 2018 – 2026

---

## Table des matières

- Résumé
- Contexte et objectifs
- Données
- Analyse exploratoire
- Stationnarité
- Modèle retenu
- Protocole de prévision et évaluation
- Diagnostics des résidus et limites
- Pistes d'amélioration
- Conclusion
- Technologies et outils
- Structure du projet
- Reproduire l'analyse
- Références
- Note de traçabilité

---

## Résumé

Dans ce travail, j'étudie l'évolution de la consommation électrique en France à partir des données ouvertes publiées par RTE. Les données initiales, disponibles à une fréquence infra-journalière de 30 minutes et comprenant plusieurs indicateurs du système électrique, ont été restreintes à la date et à la consommation afin de construire une série univariée. J'ai ensuite agrégé les observations à une fréquence quotidienne en calculant la consommation moyenne journalière.

Mon analyse exploratoire met en évidence :
- des variations de niveau ;
- une saisonnalité annuelle marquée ;
- un effet hebdomadaire.

La stationnarité de la série brute est remise en question par les tests ADF et KPSS. J'ai étudié une différenciation saisonnière de période 7, ce qui m'a conduit à retenir une structure SARIMA(2,0,2)(0,1,1)₇. Les résultats montrent une erreur de prévision plus faible à l'horizon de 7 jours qu'à 28 jours. Les diagnostics signalent toutefois une autocorrélation résiduelle et une non-normalité : le modèle constitue donc une base de prévision utile, mais perfectible.

---

## Contexte et objectifs

La consommation électrique varie selon les conditions météorologiques, les rythmes d'activité, les jours de la semaine, les périodes de vacances et les saisons. Sa prévision à court terme constitue un enjeu important pour l'équilibre entre production et consommation du système électrique.

Cette étude m'a permis de :
- décrire les principales caractéristiques temporelles de la consommation quotidienne ;
- évaluer sa stationnarité ;
- construire un modèle de prévision univarié ;
- analyser ses performances sur une période de test chronologique.

Objectifs détaillés :
- Étudier l'évolution de la consommation et ses composantes temporelles.
- Comparer les niveaux de consommation selon les jours de la semaine et les périodes de l'année.
- Identifier une transformation permettant de stabiliser la série.
- Estimer un modèle SARIMA et évaluer ses prévisions à court terme.

---

## Données

### Source

J'ai utilisé les données du portail Open Data de RTE (Réseau de Transport d'Électricité) : https://opendata.rte-france.com/.

Le jeu de données initial comporte plusieurs variables relatives au système électrique, notamment la consommation, la production et les échanges.

### Sélection des variables

Pour cette étude, je n'ai conservé que les informations de date et de consommation. Ce choix est volontaire : il me permet de mener une modélisation univariée et d'étudier la dynamique propre de la consommation, sans introduire de variables explicatives externes.

### Nettoyage et agrégation

Les données initiales sont observées toutes les 30 minutes. Les changements d'heure liés au passage à l'heure d'été et à l'heure d'hiver nécessitent un traitement spécifique :

- les intervalles dupliqués en octobre sont agrégés ;
- les créneaux manquants de mars sont interpolés linéairement ;

afin de garantir 48 observations par journée. J'ai ensuite agrégé les observations infra-journalières par date et calculé la moyenne quotidienne.

La série finale couvre la période 2018–2026 et comporte environ 3 000 observations quotidiennes. La variable que j'étudie est la consommation moyenne journalière, exprimée en MW.

$$
Y_t = \frac{1}{n_t} \sum_{i=1}^{n_t} y_{t,i}
$$

où \(y_{t,i}\) désigne la consommation mesurée au créneau infra-journalier \(i\) du jour \(t\), et \(n_t\) le nombre de créneaux retenus pour cette journée.

| Caractéristique | Valeur |
|---|---|
| Source | Open Data RTE |
| Période | 2018 – 2026 |
| Fréquence initiale | 30 minutes |
| Fréquence finale | Journalière |
| Volume | ≈ 3 000 observations |
| Variable cible | Consommation moyenne journalière (MW) |
| Approche | Univariée |

---

## Analyse exploratoire

### Évolution temporelle et décomposition

La représentation de la série et sa décomposition additive font apparaître :

- une tendance évolutive, marquée par plusieurs phases de baisse et de reprise ;
- une composante saisonnière avec un cycle annuel prononcé ;
- des résidus fluctuant autour de zéro, avec des variations importantes et quelques valeurs extrêmes.

$$
Y_t = T_t + S_t + R_t
$$

Cette décomposition distingue la tendance \(T_t\), la composante saisonnière \(S_t\) et le résidu \(R_t\). Elle reste descriptive : elle me permet de visualiser les structures présentes, mais ne suffit pas à établir la stationnarité.

### Effet du jour de la semaine

La comparaison des distributions journalières indique une consommation généralement plus élevée du lundi au vendredi et plus faible pendant le week-end.

| Groupe | Moyenne (MW) | Médiane (MW) | Écart-type | Minimum | Maximum |
|---|---:|---:|---:|---:|---:|
| Hors week-end | 53 275,16 | 49 297,58 | 9 920,15 | 36 955,00 | 89 600,08 |
| Week-end | 47 724,75 | 43 738,62 | 9 654,53 | 35 618,71 | 75 460,00 |

La différence relative entre les moyennes est d'environ 11,63 % : la moyenne hors week-end est supérieure à celle du week-end.

$$
\text{Écart relatif} = \frac{53\,275{,}16 - 47\,724{,}75}{47\,724{,}75} \times 100 \approx 11{,}63\,\%
$$

Cette comparaison confirme un effet hebdomadaire, sans pour autant démontrer à elle seule une causalité.

### Effet saisonnier annuel

L'analyse par mois met en évidence :
- une consommation plus élevée en hiver ;
- une consommation plus faible en été ;
- une reprise progressive à partir de septembre ;
- des valeurs atypiques en février, mars et octobre.

J'ai pu associer ces observations à des épisodes de consommation exceptionnels, mais leur cause exacte ne peut pas être établie à partir de la seule série de consommation.

---

## Stationnarité

La stationnarité est une propriété importante des modèles ARMA/SARIMA. Une série stationnaire au sens faible possède notamment une espérance et une variance constantes dans le temps, ainsi qu'une autocovariance qui dépend uniquement du décalage temporel.

$$
E(Y_t) = \mu \quad ; \quad \text{Var}(Y_t) = \sigma^2 \quad ; \quad \text{Cov}(Y_t, Y_{t-h}) = \gamma(h)
$$

Les graphiques exploratoires suggèrent que la série brute n'est pas stationnaire en moyenne, notamment en raison de ses structures saisonnières et de l'évolution de son niveau.

### Tests ADF et KPSS

- ADF (Augmented Dickey-Fuller) : hypothèse nulle de présence d'une racine unitaire. Une petite p-value conduit à rejeter cette hypothèse.
- KPSS : hypothèse nulle de stationnarité autour d'un niveau constant.

Résultats sur la série brute :

| Test | Statistique | p-value | Conclusion |
|---|---:|---:|---|
| KPSS | 0,93127 | 0,01 | Stationnarité rejetée |
| ADF | — | < 0,01 | Racine unitaire rejetée |

J'ai noté que ces résultats apparemment contradictoires ne doivent pas être interprétés comme une preuve que la série est stationnaire. Les tests ne ciblent pas exactement les mêmes hypothèses et peuvent réagir différemment en présence de saisonnalité, de ruptures ou d'une spécification inadéquate.

### Différenciation saisonnière hebdomadaire

La fonction nsdiffs() indique une différenciation saisonnière nécessaire (D = 1). J'ai privilégié la période hebdomadaire s = 7, car les comparaisons exploratoires montrent une différence nette entre jours ouvrés et week-end.

$$
\nabla_7 Y_t = Y_t - Y_{t-7}
$$

Résultats après différenciation saisonnière (période 7) :

| Test | Statistique | p-value | Conclusion |
|---|---:|---:|---|
| ADF | — | < 0,01 | Stationnarité en moyenne |
| KPSS | 0,014031 | 0,1 | Stationnarité non rejetée |

J'ai également utilisé la fréquence 365 pour construire une représentation annuelle de la série. Le modèle que je présente finalement utilise toutefois une saisonnalité hebdomadaire de période 7. Une saisonnalité annuelle résiduelle peut donc subsister et mérite d'être examinée dans les diagnostics.

---

## Modèle retenu

### Rappels théoriques

Un modèle SARIMA(p,d,q)(P,D,Q)ₛ combine :
- une partie autorégressive (AR) ;
- une différenciation ordinaire ;
- une partie moyenne mobile (MA) ;
- des composantes saisonnières de période s.

$$
\Phi(B^s)\,\varphi(B)\,(1-B)^d\,(1-B^s)^D\,Y_t = c + \Theta(B^s)\,\theta(B)\,\varepsilon_t
$$

où B est l'opérateur retard (B Y_t = Y_{t-1}), et ε_t représente une innovation aléatoire.

### Spécification retenue

$$
\boxed{\text{SARIMA}(2,0,2)(0,1,1)_7}
$$

La spécification comprend :
- 2 termes autorégressifs et 2 termes de moyenne mobile dans la partie non saisonnière ;
- une différenciation saisonnière hebdomadaire ;
- un terme de moyenne mobile saisonnier.

J'ai estimé le modèle à l'aide de la fonction Arima() du package forecast.

J'ai justifié la sélection finale par l'examen conjoint des ACF/PACF, des critères d'information (AIC/BIC) et des diagnostics résiduels.

---

## Protocole de prévision et évaluation

J'ai réservé les 28 dernières observations pour le test et utilisé les observations antérieures pour l'apprentissage. Cette séparation chronologique est adaptée aux séries temporelles : elle évite d'utiliser des observations futures pour entraîner le modèle.

$$
n_{\text{test}} = 28 \quad ; \quad n_{\text{train}} = n - 28
$$

### Résultats d'évaluation

| Échantillon / horizon | MAPE | MAE (MW) | RMSE (MW) | MASE |
|:---|---:|---:|---:|---:|
| Entraînement | 2,17 % | 1 147 | 1 774 | 0,34 |
| Test — 7 jours | 0,89 % | 398 | 519 | 0,12 |
| Test — 28 jours | 4,74 % | 2 021 | 2 275 | 0,59 |

### Définition des métriques

- MAPE — erreur absolue en pourcentage :
$$
\text{MAPE} = \frac{100}{n} \sum_{t} \left| \frac{Y_t - \hat{Y}_t}{Y_t} \right|
$$
- MAE — erreur absolue moyenne.
- RMSE — pénalise davantage les erreurs importantes.
- MASE — compare l'erreur du modèle à celle d'une prévision naïve de référence.

### Interprétation

- Bonne précision à 7 jours (MAPE ≈ 0,89 %, MASE ≈ 0,12).
- Dégradation à 28 jours (MAPE ≈ 4,74 %, MASE ≈ 0,59) — fréquent lorsque l'horizon s'allonge.
- Je souligne que le MAPE de 0,89 % ne doit pas être présenté comme une « précision de 99,1 % » : le MAPE est une mesure d'erreur, et non une métrique de classification ou une précision au sens strict.

---

## Diagnostics des résidus et limites

### Autocorrélation résiduelle

Le test de Ljung-Box donne une p-value de 5,23 × 10⁻¹⁰. L'hypothèse nulle d'absence d'autocorrélation résiduelle est donc rejetée.

J'en conclus que le modèle ne capture pas toute la dépendance temporelle présente dans les données. Les résidus ne peuvent pas être considérés comme un bruit blanc.

### Normalité

Le test de Shapiro-Wilk donne W = 0,912 et p < 2,2 × 10⁻¹⁶. L'hypothèse de normalité des résidus est rejetée. Le graphique Q-Q montre également des écarts à la droite théorique, notamment dans les queues de distribution.

J'ai noté que la non-normalité n'annule pas automatiquement l'utilité des prévisions ponctuelles. Elle m'invite toutefois à interpréter avec prudence les intervalles de prédiction construits sous des hypothèses distributionnelles, notamment les intervalles à 95 %.

### Limites méthodologiques

- Le modèle est univarié : il n'intègre pas directement la température, les jours fériés ou d'autres facteurs explicatifs.
- La différenciation hebdomadaire ne garantit pas la disparition de la saisonnalité annuelle.
- L'autocorrélation résiduelle montre que la dynamique n'est pas entièrement modélisée.
- Les performances à 28 jours sont moins bonnes qu'à 7 jours.
- Les valeurs extrêmes et les événements exceptionnels peuvent dégrader les prévisions.

---

## Pistes d'amélioration

- SARIMAX : intégrer la température, les jours fériés et éventuellement les calendriers de vacances.
- Termes de Fourier : représenter les cycles annuels et améliorer la prise en compte de plusieurs saisonnalités.
- Validation à origine glissante (rolling-origin) : évaluer la stabilité des performances sur plusieurs périodes.
- Comparaison AIC/BIC : comparer plusieurs spécifications SARIMA tout en privilégiant des résidus proches d'un bruit blanc.
- Analyse des valeurs extrêmes : examiner les observations extrêmes et vérifier leur correspondance avec des événements réels avant toute correction ou suppression.

---

## Conclusion

Cette étude m'a permis de construire une chaîne d'analyse et de prévision de la consommation électrique quotidienne à partir des données ouvertes de RTE. L'agrégation des mesures de 30 minutes à une fréquence journalière a rendu possible l'étude d'une série d'environ 3 000 observations couvrant 2018–2026. Mon analyse exploratoire a mis en évidence une saisonnalité annuelle, un effet hebdomadaire et des variations importantes du niveau de consommation.

La différenciation saisonnière de période 7 m'a conduit à estimer un modèle SARIMA(2,0,2)(0,1,1)₇. Les résultats que j'ai obtenus sont encourageants à très court terme, avec un MAPE de 0,89 % sur 7 jours, mais se dégradent à 28 jours. Les tests de diagnostic révèlent néanmoins une autocorrélation résiduelle significative et une non-normalité.

J'en conclus que le modèle doit être considéré comme une première solution de prévision pertinente, mais non comme un modèle dont toutes les hypothèses seraient parfaitement satisfaites.

Une extension vers SARIMAX, l'ajout de termes de Fourier et une validation chronologique répétée constituent, à mon sens, des pistes naturelles pour améliorer la robustesse et la qualité des prévisions.

---

## Technologies et outils

| Domaine | Outils / Packages |
|---|---|
| Langage | R |
| Modélisation temporelle | forecast — Arima(), forecast(), nsdiffs() |
| Tests statistiques | tseries — ADF, KPSS |
| Manipulation des données | dplyr, lubridate |
| Visualisation | ggplot2 |
| Environnement | Notebook R (RStudio / VS Code) |

---

## Structure du projet
