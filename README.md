# Analyse de l'effet du changement climatique

Mini‑projet de statistique/épidémiologie visant à étudier l'effet d'événements climatiques extrêmes sur la santé d'une population urbaine et rurale, à partir du jeu de données **Effect_Climate_Change.csv**.

Le projet est implémenté en R (via un document Quarto/R Markdown) et comporte :

- une **phase de préparation** et nettoyage des données,
- des **statistiques descriptives**,
- des **analyses de corrélation**,
- des **visualisations exploratoires**,
- des **tests statistiques** (χ², t‑test, ANOVA),
- des **modèles de régression** (linéaire simple, multiple, logistique).

Ce dépôt est destiné à la **validation d’un mini‑projet**.

---

## 1. Objectifs du projet

À partir du fichier `Effect_Climate_Change.csv`, le projet vise à :

- Décrire les caractéristiques socio‑démographiques, climatiques et sanitaires de la population (âge, sexe, région, BMI, maladies, etc.).
- Explorer les liens entre événements climatiques (vagues de chaleur, inondations, contamination de l’eau, froid, etc.) et présence de maladie.
- Estimer des modèles simples pour identifier les principaux facteurs associés à la **présence** et au **degré** de maladie.

---

## 2. Fonctionnement général

L’analyse est décrite dans un fichier Quarto/R Markdown ( `climate_change_analysis.qmd`) structuré comme suit :

1. **Préparation de l’environnement**
   - Installation des packages manquants.
   - Chargement des librairies nécessaires (`tidyverse`, `ggplot2`, `dplyr`, `corrplot`, `psych`, `car`).

2. **Chargement et exploration des données**
   - Lecture du fichier `Effect_Climate_Change.csv`.
   - Aperçu des données (`head`, `dim`, `str`, `summary`).
   - Vérification des valeurs manquantes.

3. **Nettoyage des données**
   - Suppression de la colonne de série (`SNO`).
   - Standardisation des noms de variables via `make.names()`.
   - Conversion des variables qualitatives en facteurs (région, sexe, statut marital, éducation, occupation, BMI, maladie, type de maladie, inondations, contamination de l’eau, vagues de froid, ville, etc.).

4. **Statistiques descriptives**
   - Description des variables numériques (âge, poids, taille, BMI, vagues de chaleur, degré de maladie).
   - Tableaux de fréquences pour les variables catégorielles (région, sexe, maladie, inondations, ville).

5. **Analyses de corrélation**
   - Construction d’une matrice de corrélation pour les variables numériques pertinentes.
   - Visualisation de la matrice grâce à `corrplot`.
   - Analyse spécifique de la corrélation entre **vagues de chaleur** et **degré de maladie**.

6. **Visualisations**
   - Boxplots de l’âge par région.
   - Graphiques en barres de la proportion de maladies selon la catégorie de BMI.
   - Violons/boxplots des vagues de chaleur par ville.
   - Graphiques liant contamination de l’eau et présence de maladies.
   - Nuages de points liant vagues de chaleur, BMI et maladie.

7. **Tests statistiques**
   - Test du χ² : inondations vs maladie.
   - Test du χ² : contamination de l’eau vs maladie.
   - Test t : différence de BMI selon l’exposition aux inondations.
   - ANOVA : ville et vagues de chaleur (avec post‑hoc Tukey).
   - ANOVA : catégorie de BMI et degré de maladie (chez les sujets malades).

8. **Modèles de régression**
   - Régression linéaire simple : degré de maladie ~ vagues de chaleur.
   - Régression linéaire multiple : degré de maladie ~ vagues de chaleur + BMI + âge.
   - Régression logistique (binaire) pour la présence de maladie en fonction de :
     - vagues de chaleur,
     - BMI,
     - âge,
     - exposition aux inondations,
     - contamination de l’eau.

Chaque section du document génère à la fois :
- du **code R**,
- des **tableaux** (statistiques, tests),
- et des **graphiques** pour l’interprétation.

---

## 3. Dépendances (packages R)

Les packages utilisés dans le projet sont :

```r
packages_needed <- c(
  "tidyverse",   # Manipulation + visualisation de données
  "ggplot2",     # Graphiques
  "dplyr",       # Manipulation des données
  "corrplot",    # Matrices de corrélation
  "psych",       # Statistiques descriptives avancées
  "car"          # Diagnostics de régression, VIF, etc.
)
```

L’installation automatique (si nécessaire) est gérée dans le code :

```r
new_packages <- packages_needed[!(packages_needed %in% installed.packages()[, "Package"])]

if (length(new_packages) > 0) {
  install.packages(new_packages)
}

lapply(packages_needed, library, character.only = TRUE)
```

---

## 4. Données

- Fichier principal : **`Effect_Climate_Change.csv`**
- Nombre d’observations : **800**
- Nombre de variables : **31**

Les variables couvrent :
- Données **démographiques** : âge, sexe, région, statut marital, éducation, occupation.
- Données **économiques** : revenu mensuel.
- Données de **santé** : poids, taille, BMI, présence de maladie, type de maladie, degré de maladie.
- Données **climatiques/environnementales** : vagues de chaleur, inondations, contamination de l’eau, vagues de froid, tempêtes hivernales, ville, etc.

---

## 5. Exemples de code / d’utilisation

### 5.1 Chargement des données

```r
data <- read.csv(
  "Effect_Climate_Change.csv",
  header = TRUE,
  stringsAsFactors = TRUE
)

head(data)
dim(data)
str(data)
summary(data)
```

### 5.2 Nettoyage des données

```r
library(dplyr)

# Suppression de l'identifiant
data_clean <- data %>% select(-SNO)

# Standardisation des noms
names(data_clean) <- make.names(names(data_clean))

# Variables catégorielles à passer en facteur
categorical_vars <- c(
  "REGION.", "SEX", "MARITAL.STATUS", "EDUCATION",
  "OCCUPATION", "BMI.CATEGORY", "DISEASE", "DISEASE.TYPE",
  "FLOODS", "CONTAMINATION.OF.WATER", "COLD.WAVES",
  "WINTER.STORMS", "CITY"
)

for (var in categorical_vars) {
  if (var %in% names(data_clean)) {
    data_clean[[var]] <- as.factor(data_clean[[var]])
  }
}
```

### 5.3 Statistiques descriptives (numériques)

```r
library(psych)

numeric_vars <- c("AGE", "WEIGHT", "HEIGHT", "BMI",
                  "HEAT.WAVES", "DEGREE.OF.DISEASE")

desc_stats <- describe(data_clean[, numeric_vars])
desc_stats
```

### 5.4 Matrice de corrélation

```r
library(corrplot)

numeric_data <- data_clean %>%
  dplyr::select(
    AGE, WEIGHT, HEIGHT, BMI, HEAT.WAVES, DEGREE.OF.DISEASE,
    FLOODS.descrete, DISEASE.descrete2,
    CONTAMINATION.OF.WATER.descrete
  )

cor_matrix <- cor(numeric_data, use = "complete.obs")
corrplot(cor_matrix,
         method = "color",
         type = "upper",
         tl.col = "black",
         tl.srt = 45,
         title = "Matrice de Corrélation - Variables Climatiques et Santé")
```

### 5.5 Exemple de modèle de régression linéaire

```r
data_disease_only <- data_clean %>% dplyr::filter(DISEASE == "Yes")

model_1 <- lm(DEGREE.OF.DISEASE ~ HEAT.WAVES, data = data_disease_only)
summary(model_1)
```

### 5.6 Exemple de modèle de régression logistique

```r
data_clean$DISEASE_binary <- ifelse(data_clean$DISEASE == "Yes", 1, 0)

model_logistic <- glm(
  DISEASE_binary ~ HEAT.WAVES + BMI + AGE +
    FLOODS.descrete + CONTAMINATION.OF.WATER.descrete,
  data = data_clean,
  family = binomial
)

summary(model_logistic)
exp(coef(model_logistic))  # Odds ratios
```

---




