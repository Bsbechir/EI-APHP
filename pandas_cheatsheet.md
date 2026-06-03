# 🐼 Pandas — Référence complète

> Cheatsheet personnel · Exploration · Nettoyage · Transformation · Visualisation

---

## 📦 Import standard

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```

---

## 1. Création de structures

```python
# Depuis un dict
df = pd.DataFrame({
    'nom': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'score': [88.5, 92.0, 79.3]
})

# Depuis une liste de listes
df = pd.DataFrame([[1, 2], [3, 4]], columns=['a', 'b'])

# Series
s = pd.Series([10, 20, 30], index=['x', 'y', 'z'])

# DataFrame vide
df = pd.DataFrame(columns=['col1', 'col2'])
```

---

## 2. Lecture / Écriture

```python
# CSV
df = pd.read_csv('fichier.csv', sep=',', encoding='utf-8', index_col=0)
df.to_csv('sortie.csv', index=False)

# Excel
df = pd.read_excel('fichier.xlsx', sheet_name='Sheet1')
df.to_excel('sortie.xlsx', index=False, sheet_name='Résultats')

# JSON
df = pd.read_json('data.json')
df.to_json('sortie.json', orient='records', indent=2)

# Parquet (rapide, compressé)
df = pd.read_parquet('data.parquet')
df.to_parquet('sortie.parquet')

# SQL
import sqlite3
conn = sqlite3.connect('base.db')
df = pd.read_sql('SELECT * FROM table', conn)
df.to_sql('table', conn, if_exists='replace', index=False)

# Depuis le presse-papier
df = pd.read_clipboard(sep='\t')

# Options utiles pour read_csv
df = pd.read_csv(
    'data.csv',
    sep=';',
    decimal=',',         # virgule comme séparateur décimal
    thousands='.',
    na_values=['N/A', '-', ''],
    parse_dates=['date'],
    dtype={'id': str},
    nrows=1000,          # lire seulement les 1000 premières lignes
    skiprows=2,
    usecols=['col1', 'col2']
)
```

---

## 3. Exploration initiale

```python
df.shape          # (lignes, colonnes)
df.dtypes         # types de chaque colonne
df.info()         # résumé complet (types + nulls + mémoire)
df.head(10)       # 10 premières lignes
df.tail(5)        # 5 dernières lignes
df.sample(5)      # 5 lignes aléatoires

df.describe()                    # stats pour colonnes numériques
df.describe(include='all')       # inclut catégorielles
df.describe(include=['object'])  # seulement texte

df.columns.tolist()   # liste des noms de colonnes
df.index              # index
df.values             # array numpy sous-jacent

# Valeurs uniques
df['col'].unique()
df['col'].nunique()
df['col'].value_counts()
df['col'].value_counts(normalize=True)  # proportions

# Mémoire utilisée
df.memory_usage(deep=True)
df.memory_usage(deep=True).sum() / 1e6  # en MB
```

---

## 4. Sélection et accès aux données

```python
# Colonnes
df['col']               # Series
df[['col1', 'col2']]    # DataFrame

# Lignes par index label
df.loc[0]               # ligne 0
df.loc[0:5]             # lignes 0 à 5 inclus
df.loc[:, 'col']        # toute une colonne
df.loc[0:5, ['a', 'b']] # lignes 0-5, colonnes a et b

# Lignes par position entière
df.iloc[0]
df.iloc[0:5]
df.iloc[:, 0]           # première colonne
df.iloc[0:5, 0:3]       # sous-tableau

# Conditions booléennes
df[df['age'] > 25]
df[(df['age'] > 25) & (df['score'] >= 90)]
df[(df['pays'] == 'FR') | (df['pays'] == 'DE')]
df[~df['col'].isna()]

# isin / between
df[df['pays'].isin(['FR', 'MA', 'TN'])]
df[df['age'].between(20, 35)]

# query (syntaxe string)
df.query("age > 25 and score >= 90")
df.query("pays in ['FR', 'MA']")

# nlargest / nsmallest
df.nlargest(5, 'score')
df.nsmallest(3, 'age')

# where / mask
df['col'].where(df['col'] > 0, other=0)   # garde si True, remplace sinon
df['col'].mask(df['col'] < 0, other=np.nan)
```

---

## 5. Valeurs manquantes

```python
# Détecter
df.isna()                    # DataFrame booléen
df.isna().sum()              # nombre de NaN par colonne
df.isna().sum() / len(df)    # proportion
df.isna().any()              # True si au moins un NaN dans la colonne
df[df['col'].isna()]         # lignes où col est NaN

# Supprimer
df.dropna()                          # lignes avec au moins un NaN
df.dropna(axis=1)                    # colonnes avec au moins un NaN
df.dropna(subset=['col1', 'col2'])   # seulement si NaN dans ces colonnes
df.dropna(thresh=3)                  # garder si au moins 3 valeurs non-NaN

# Remplacer / imputer
df.fillna(0)
df.fillna({'col1': 0, 'col2': 'inconnu'})
df['col'].fillna(df['col'].mean())
df['col'].fillna(df['col'].median())
df['col'].fillna(method='ffill')     # forward fill
df['col'].fillna(method='bfill')     # backward fill

# Interpolation
df['col'].interpolate(method='linear')
df['col'].interpolate(method='time')  # si index datetime
```

---

## 6. Types de données

```python
# Convertir
df['age'] = df['age'].astype(int)
df['score'] = df['score'].astype(float)
df['actif'] = df['actif'].astype(bool)
df['code'] = df['code'].astype(str)

# Catégoriel (économise de la mémoire)
df['pays'] = df['pays'].astype('category')
df['pays'].cat.categories
df['pays'].cat.codes           # entiers sous-jacents

# Dates
df['date'] = pd.to_datetime(df['date'])
df['date'] = pd.to_datetime(df['date'], format='%d/%m/%Y')
df['date'] = pd.to_datetime(df['date'], errors='coerce')  # NaT si erreur

# Numérique (gère les erreurs)
df['val'] = pd.to_numeric(df['val'], errors='coerce')
```

---

## 7. Manipulation des colonnes

```python
# Renommer
df.rename(columns={'ancien': 'nouveau', 'a': 'b'}, inplace=True)
df.columns = ['col1', 'col2', 'col3']  # renommer tout

# Ajouter / modifier
df['nouvelle'] = df['a'] + df['b']
df['flag'] = df['score'] > 90

# Supprimer
df.drop(columns=['col1', 'col2'], inplace=True)
df.drop('col1', axis=1, inplace=True)

# Réordonner
df = df[['col3', 'col1', 'col2']]

# Dupliquer une colonne
df['copie'] = df['original'].copy()

# apply sur une colonne
df['maj'] = df['nom'].apply(lambda x: x.upper())
df['categorie'] = df['score'].apply(lambda x: 'A' if x >= 90 else 'B')

# apply sur plusieurs colonnes (axis=1 = par ligne)
df['total'] = df.apply(lambda row: row['a'] + row['b'], axis=1)

# map (depuis un dict)
mapping = {'M': 'Homme', 'F': 'Femme'}
df['genre_label'] = df['genre'].map(mapping)
```

---

## 8. Chaînes de caractères (str)

```python
df['nom'].str.upper()
df['nom'].str.lower()
df['nom'].str.strip()               # supprimer espaces
df['nom'].str.replace(' ', '_')
df['nom'].str.contains('Ali')       # booléen
df['nom'].str.startswith('A')
df['nom'].str.len()
df['nom'].str.split(' ')            # séparer en liste
df['nom'].str.split(' ', expand=True)  # séparer en colonnes
df['email'].str.extract(r'(\w+)@')     # regex
df['code'].str.zfill(5)             # padding avec zéros
df['val'].str.replace(',', '.').astype(float)  # virgule → point
```

---

## 9. Dates et temps

```python
# Accéder aux composantes
df['date'].dt.year
df['date'].dt.month
df['date'].dt.day
df['date'].dt.hour
df['date'].dt.minute
df['date'].dt.dayofweek   # 0=lundi, 6=dimanche
df['date'].dt.day_name()
df['date'].dt.month_name()
df['date'].dt.quarter
df['date'].dt.is_month_end
df['date'].dt.is_weekend   # depuis pandas 2.2+

# Différences
df['duree'] = df['fin'] - df['debut']         # Timedelta
df['jours'] = df['duree'].dt.days
df['heures'] = df['duree'].dt.total_seconds() / 3600

# Décalage (offset)
df['date'] + pd.DateOffset(months=1)
df['date'] + pd.Timedelta(days=7)

# Filtrer par date
df[df['date'] >= '2024-01-01']
df[df['date'].between('2024-01-01', '2024-12-31')]

# Resampling (avec index datetime)
df.set_index('date', inplace=True)
df.resample('M').mean()    # mensuel
df.resample('W').sum()     # hebdo
df.resample('Q').agg({'ventes': 'sum', 'prix': 'mean'})
```

---

## 10. Tri et rang

```python
df.sort_values('score', ascending=False)
df.sort_values(['pays', 'score'], ascending=[True, False])
df.sort_index()

# Rang
df['rang'] = df['score'].rank(ascending=False, method='dense')
```

---

## 11. Dédoublonnage

```python
df.duplicated()                          # booléen par ligne
df.duplicated(subset=['nom', 'date'])    # sur colonnes spécifiques
df.drop_duplicates()
df.drop_duplicates(subset=['nom'], keep='last')
```

---

## 12. Groupby (agrégations)

```python
# Agrégation simple
df.groupby('pays')['score'].mean()
df.groupby('pays')['score'].agg(['mean', 'std', 'count'])

# Plusieurs colonnes
df.groupby(['pays', 'annee'])['ventes'].sum()

# agg avec dict
df.groupby('pays').agg(
    score_moyen=('score', 'mean'),
    score_max=('score', 'max'),
    nb_individus=('nom', 'count')
)

# transform (garde la forme originale)
df['score_normalise'] = df.groupby('pays')['score'].transform(
    lambda x: (x - x.mean()) / x.std()
)
df['rang_dans_pays'] = df.groupby('pays')['score'].rank(ascending=False)

# filter (filtre des groupes entiers)
df.groupby('pays').filter(lambda x: len(x) >= 5)

# apply (custom par groupe)
df.groupby('pays').apply(lambda g: g.nlargest(3, 'score'))
```

---

## 13. Merge / Join

```python
# Inner join (défaut)
merged = pd.merge(df1, df2, on='id')

# Types de join
pd.merge(df1, df2, on='id', how='left')
pd.merge(df1, df2, on='id', how='right')
pd.merge(df1, df2, on='id', how='outer')
pd.merge(df1, df2, on='id', how='inner')

# Sur plusieurs colonnes
pd.merge(df1, df2, on=['id', 'date'])

# Clés différentes
pd.merge(df1, df2, left_on='user_id', right_on='id')

# Suffixes en cas de collision de noms
pd.merge(df1, df2, on='id', suffixes=('_gauche', '_droite'))

# join sur index
df1.join(df2, how='left')

# Concaténation
pd.concat([df1, df2], ignore_index=True)           # empiler verticalement
pd.concat([df1, df2], axis=1)                      # coller côte à côte
pd.concat([df1, df2], ignore_index=True, keys=['A', 'B'])
```

---

## 14. Pivot et reshape

```python
# Pivot table
pivot = df.pivot_table(
    values='ventes',
    index='mois',
    columns='produit',
    aggfunc='sum',
    fill_value=0,
    margins=True    # ajoute totaux
)

# Pivot simple (sans agrégation)
df.pivot(index='date', columns='produit', values='ventes')

# Melt (wide → long)
df_long = df.melt(
    id_vars=['id', 'nom'],
    value_vars=['jan', 'fev', 'mar'],
    var_name='mois',
    value_name='ventes'
)

# stack / unstack
df.set_index(['pays', 'mois']).stack()
df.unstack(level='mois')

# crosstab
pd.crosstab(df['pays'], df['genre'])
pd.crosstab(df['pays'], df['genre'], normalize='index')  # proportions
```

---

## 15. Fenêtres glissantes (rolling / expanding)

```python
# Rolling
df['moy_mobile_7j'] = df['valeur'].rolling(window=7).mean()
df['somme_30j'] = df['valeur'].rolling(window=30).sum()
df['std_14j'] = df['valeur'].rolling(window=14).std()

# Expanding (depuis le début)
df['cumul'] = df['valeur'].expanding().sum()
df['max_hist'] = df['valeur'].expanding().max()

# Cumsum / cumprod
df['cumul'] = df['valeur'].cumsum()
df['produit_cumul'] = df['valeur'].cumprod()

# Lag / décalage
df['valeur_j1'] = df['valeur'].shift(1)      # valeur d'hier
df['valeur_j7'] = df['valeur'].shift(7)
df['variation'] = df['valeur'].diff(1)        # différence
df['pct_change'] = df['valeur'].pct_change()  # variation en %
```

---

## 16. Discrétisation / Binning

```python
# Intervalles égaux
df['tranche'] = pd.cut(df['age'], bins=5)
df['tranche'] = pd.cut(df['age'], bins=[0, 18, 35, 60, 100],
                        labels=['enfant', 'jeune', 'adulte', 'senior'])

# Quantiles égaux
df['quartile'] = pd.qcut(df['score'], q=4, labels=['Q1', 'Q2', 'Q3', 'Q4'])
```

---

## 17. Index

```python
df.set_index('id', inplace=True)
df.reset_index(inplace=True)
df.reset_index(drop=True, inplace=True)   # supprimer l'ancien index

# Index multi-niveau (MultiIndex)
df.set_index(['pays', 'date'], inplace=True)
df.loc[('FR', '2024-01-01')]
df.xs('FR', level='pays')

# Réindexer
df.reindex([0, 1, 2, 3, 4])   # ajoute NaN pour les index manquants
```

---

## 18. Performances et optimisation

```python
# Éviter apply quand possible — préférer les opérations vectorisées
df['total'] = df['a'] + df['b']          # ✅ rapide
df['total'] = df.apply(lambda r: r.a + r.b, axis=1)  # ❌ lent

# np.where (rapide pour conditions simples)
df['label'] = np.where(df['score'] >= 90, 'excellent', 'standard')

# np.select (plusieurs conditions)
conditions = [df['score'] >= 90, df['score'] >= 75, df['score'] >= 60]
choices = ['A', 'B', 'C']
df['grade'] = np.select(conditions, choices, default='D')

# Réduire la mémoire
df['code'] = df['code'].astype('category')
df['age'] = df['age'].astype('int8')       # si petits entiers
df['score'] = df['score'].astype('float32')

# Chunk reading pour gros fichiers
for chunk in pd.read_csv('gros_fichier.csv', chunksize=10000):
    # traiter chunk
    pass

# eval (rapide pour calculs)
df.eval("nouveau = a + b * 2", inplace=True)
```

---

## 19. Visualisation rapide

```python
# Directement via pandas
df['score'].hist(bins=30)
df['score'].plot(kind='hist', bins=30, figsize=(10, 5))
df.plot(x='date', y='valeur', kind='line')
df.plot(x='pays', y='score', kind='bar')
df.boxplot(column='score', by='pays')
df.plot(kind='scatter', x='age', y='score')

# Matrice de corrélation
corr = df.corr(numeric_only=True)
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm')
plt.show()

# Distribution
df['score'].plot(kind='kde')          # courbe de densité

# Pandas Profiling (rapport complet)
# pip install ydata-profiling
from ydata_profiling import ProfileReport
report = ProfileReport(df)
report.to_file('rapport.html')
```

---

## 20. Fonctions utiles diverses

```python
# Appliquer une fonction à tout le DataFrame
df.applymap(lambda x: round(x, 2))   # (pandas < 2.1)
df.map(lambda x: round(x, 2))        # (pandas >= 2.1)

# Pipe (chaîner des opérations)
df = (
    df
    .pipe(lambda d: d.dropna())
    .pipe(lambda d: d[d['score'] > 50])
    .pipe(lambda d: d.reset_index(drop=True))
)

# Afficher sans troncature
pd.set_option('display.max_rows', 100)
pd.set_option('display.max_columns', 50)
pd.set_option('display.float_format', '{:.2f}'.format)
pd.set_option('display.max_colwidth', 200)
pd.reset_option('all')   # reset

# Copie vs vue
df_copie = df.copy()         # copie indépendante
df_vue = df[['a', 'b']]      # peut être une vue (attention!)

# Itérer (lent — éviter si possible)
for idx, row in df.iterrows():
    print(row['nom'])

# Assign (retourne nouveau df, utile en pipe)
df = df.assign(
    total=lambda d: d['a'] + d['b'],
    flag=lambda d: d['total'] > 100
)

# Clip (borner les valeurs)
df['score'] = df['score'].clip(lower=0, upper=100)

# Abs, round
df['ecart'].abs()
df['score'].round(2)

# Swap levels dans MultiIndex
df.swaplevel()
df.sort_index(level=0)
```

---

## 21. Checklist exploration rapide

```python
def explore(df):
    print("=== SHAPE ===")
    print(df.shape)
    
    print("\n=== TYPES ===")
    print(df.dtypes)
    
    print("\n=== VALEURS MANQUANTES ===")
    missing = df.isna().sum()
    print(missing[missing > 0])
    
    print("\n=== DOUBLONS ===")
    print(f"{df.duplicated().sum()} doublons")
    
    print("\n=== STATISTIQUES ===")
    print(df.describe(include='all'))
    
    print("\n=== APERÇU ===")
    return df.head()

explore(df)
```

---

*Maintenu par Bechir · mis à jour selon les besoins*
``

