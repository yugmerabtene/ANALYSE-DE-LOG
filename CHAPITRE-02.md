# CHAPITRE 2 — STRUCTURE ET EXTRACTION DES LOGS AVEC PYTHON

**Durée totale :** 2 h 30
**Niveau :** Bachelor 3 – Data & Business Intelligence
**Compétences visées :**

* Comprendre la structure détaillée d’un log web (Apache, Nginx).
* Savoir extraire, filtrer et structurer des logs bruts.
* Manipuler les logs avec **Python et Pandas**.
* Produire des rapports statistiques (trafic, erreurs, IP, codes HTTP).
* Automatiser les premières analyses répétitives.

---

## 2.1. Rappel : structure d’un log web

Les fichiers de logs des serveurs web suivent un format dit **Common Log Format (CLF)**, ou sa variante **Combined Log Format**, utilisée par Apache et Nginx.

### Exemple de ligne CLF :

```
192.168.0.10 - - [18/Oct/2025:09:45:12 +0200] "GET /index.html HTTP/1.1" 200 512 "-" "Mozilla/5.0"
```

### Exemple de ligne Nginx (format JSON) :

```
{"time":"2025-10-18T09:45:12+02:00","remote_addr":"192.168.0.10","method":"GET","uri":"/index.html","status":200,"bytes":512}
```

### Différence :

| Format           | Avantages                      | Inconvénients           |
| ---------------- | ------------------------------ | ----------------------- |
| CLF (texte brut) | Universel, léger               | Plus difficile à parser |
| JSON logs        | Lisible par machines, flexible | Plus volumineux         |

---

## 2.2. Extraction des logs : principes et méthodes

Pour exploiter les logs dans un environnement de **Data Analysis**, il faut :

1. **Charger le fichier** dans un outil d’analyse (Python, Excel, SQL, ELK).
2. **Extraire les champs pertinents** (IP, date, méthode, code, etc.).
3. **Nettoyer les données** (format de date, conversion numérique).
4. **Analyser et visualiser** les résultats.

### Outils possibles :

| Outil                  | Usage principal                        |
| ---------------------- | -------------------------------------- |
| `grep`, `awk`, `sed`   | Extraction rapide en ligne de commande |
| **Python (Pandas)**    | Analyse statistique et filtrage        |
| **GoAccess / AWStats** | Visualisation rapide                   |
| **ELK Stack**          | Supervision avancée                    |

---

## 2.3. Analyse structurée avec Python et Pandas

### Exemple pratique de lecture d’un fichier log Apache

**Fichier :** `access.log`

```python
import pandas as pd

# Définir les colonnes attendues
cols = ['ip', 'dash1', 'dash2', 'datetime', 'tz', 'method', 'url', 'protocol', 'code', 'size', 'ref', 'ua']

# Lecture du fichier
logs = pd.read_csv("access.log", sep=' ', header=None, usecols=[0,3,4,5,6,7,8,9], names=cols[:8])

# Nettoyage du champ date
logs['datetime'] = logs['datetime'].str.strip('[')

# Conversion du code HTTP en entier
logs['code'] = logs['code'].astype(int)
```

**Résultat attendu :**
Un DataFrame de ce type :

| ip          | datetime             | method | url         | protocol | code |
| ----------- | -------------------- | ------ | ----------- | -------- | ---- |
| 192.168.0.1 | 18/Oct/2025:09:45:12 | GET    | /index.html | HTTP/1.1 | 200  |
| 192.168.0.2 | 18/Oct/2025:09:46:18 | GET    | /admin      | HTTP/1.1 | 403  |

---

### Exemple d’analyse basique

#### Comptage des requêtes totales et des erreurs :

```python
total = len(logs)
erreurs = len(logs[logs['code'] >= 400])
taux_erreurs = round(erreurs / total * 100, 2)
print(f"Total requêtes : {total}")
print(f"Taux d’erreurs : {taux_erreurs}%")
```

**Sortie possible :**

```
Total requêtes : 1254
Taux d’erreurs : 3.24%
```

---

### Extraction d’informations spécifiques

#### 1. Top 10 des pages consultées :

```python
print(logs['url'].value_counts().head(10))
```

#### 2. Top 5 des IP les plus actives :

```python
print(logs['ip'].value_counts().head(5))
```

#### 3. Erreurs fréquentes :

```python
erreurs_freq = logs[logs['code'] >= 400]['code'].value_counts()
print(erreurs_freq)
```

#### 4. Répartition des requêtes par heure :

```python
logs['hour'] = logs['datetime'].str.extract(r':(\d{2}):')
print(logs['hour'].value_counts().sort_index())
```

---

## 2.4. Exercices

### Exercice 1 – Lecture et affichage

**Objectif :** Charger un fichier `access.log` et afficher le nombre total de requêtes.

**Corrigé :**

```python
logs = pd.read_csv("access.log", sep=' ', header=None)
print(f"Nombre total de lignes : {len(logs)}")
```

---

### Exercice 2 – Identifier les erreurs 404 et 500

**Objectif :** Extraire les lignes contenant des erreurs HTTP.

**Énoncé :**
Afficher toutes les lignes du fichier dont le code HTTP est 404 ou 500.

**Corrigé :**

```python
logs = pd.read_csv("access.log", sep=' ', header=None, usecols=[0,6,8], names=['ip','url','code'])
logs = logs[logs['code'].astype(str).str.match('404|500')]
print(logs)
```

---

### Exercice 3 – Statistiques de base

**Énoncé :**
À partir du fichier `access.log`, produire :

1. Le total de requêtes.
2. Le pourcentage d’erreurs.
3. Le top 3 des IP les plus fréquentes.
4. Le top 3 des pages les plus consultées.

**Corrigé :**

```python
logs = pd.read_csv("access.log", sep=' ', header=None, usecols=[0,6,8], names=['ip','url','code'])
total = len(logs)
errors = len(logs[logs['code'] >= 400])
taux = errors / total * 100
print(f"Taux d’erreurs : {round(taux,2)}%")
print("Top IPs :")
print(logs['ip'].value_counts().head(3))
print("Top URLs :")
print(logs['url'].value_counts().head(3))
```

---

## 2.5. Travaux Pratiques (TP2) – Rapport d’analyse automatisé

### Objectif

Créer un script Python capable d’extraire automatiquement les informations clés d’un fichier `access.log` et de générer un rapport `.csv`.

---

### Énoncé

Le script doit :

1. Charger le fichier `access.log`.
2. Nettoyer et structurer les données.
3. Calculer :

   * le nombre total de requêtes ;
   * le taux d’erreurs ;
   * le top 10 des pages ;
   * le top 5 des IP ;
   * la distribution horaire des requêtes.
4. Exporter un fichier `rapport_log.csv`.

---

### Corrigé complet

```python
import pandas as pd

# Chargement du fichier
logs = pd.read_csv("access.log", sep=' ', header=None,
                   usecols=[0,3,5,6,8],
                   names=['ip', 'datetime', 'method', 'url', 'code'])

# Nettoyage
logs['datetime'] = logs['datetime'].str.strip('[')
logs['code'] = logs['code'].astype(int)

# Statistiques
total = len(logs)
erreurs = len(logs[logs['code'] >= 400])
taux_erreurs = round(erreurs / total * 100, 2)

# Top pages et IP
top_pages = logs['url'].value_counts().head(10)
top_ip = logs['ip'].value_counts().head(5)

# Distribution horaire
logs['hour'] = logs['datetime'].str.extract(r':(\d{2}):')
heure_stats = logs['hour'].value_counts().sort_index()

# Sauvegarde
rapport = {
    "Total requêtes": [total],
    "Taux d’erreurs (%)": [taux_erreurs]
}
df = pd.DataFrame(rapport)
df.to_csv("rapport_log.csv", index=False)

print("Rapport généré : rapport_log.csv")
print("\n--- TOP 5 IP ---")
print(top_ip)
print("\n--- TOP 10 PAGES ---")
print(top_pages)
print("\n--- Répartition horaire ---")
print(heure_stats)
```

**Résultats attendus :**

```
Total requêtes : 1254
Taux d’erreurs : 3.24%
Top 5 IP :
192.168.0.15 : 120
192.168.0.10 : 85
...
```

---

## 2.6. Conclusion du chapitre

À la fin de ce chapitre, l’étudiant doit être capable de :

* interpréter la structure d’un log web (Apache ou Nginx) ;
* manipuler des logs sous Python et en extraire des statistiques simples ;
* construire un rapport automatisé permettant de superviser un site web ;
* détecter les erreurs fréquentes (codes 4xx et 5xx).

---

## 2.7. Lexique

| Terme                       | Définition                                                                             |
| --------------------------- | -------------------------------------------------------------------------------------- |
| **Pandas**                  | Bibliothèque Python de manipulation de données tabulaires (DataFrame).                 |
| **Regex**                   | Expressions régulières : outil de recherche et de filtrage de texte.                   |
| **DataFrame**               | Structure de données tabulaire utilisée dans Pandas.                                   |
| **Parser**                  | Processus de lecture et d’analyse d’un texte pour en extraire les données structurées. |
| **Common Log Format (CLF)** | Format standard de logs Apache/Nginx.                                                  |
| **HTTP Code**               | Réponse numérique du serveur à une requête.                                            |
| **Datetime**                | Formatage d’une date et heure dans un fichier log.                                     |
| **Taux d’erreurs**          | Pourcentage de requêtes ayant renvoyé un code ≥ 400.                                   |
| **Extraction horaire**      | Technique consistant à regrouper les logs par heure d’accès.                           |

---

## 2.8. Bibliographie et ressources

1. **Apache HTTP Server Documentation** – Logging and CustomLog Directive
   [https://httpd.apache.org/docs/2.4/logs.html](https://httpd.apache.org/docs/2.4/logs.html)

2. **Pandas Documentation** – Reading and Writing Data
   [https://pandas.pydata.org/docs](https://pandas.pydata.org/docs)

3. **OWASP (2024)** – Logging and Monitoring Best Practices
   [https://cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org)

4. **Python Official Docs** – Regular Expressions (`re` module)
   [https://docs.python.org/3/library/re.html](https://docs.python.org/3/library/re.html)

5. **RFC 7231** – Hypertext Transfer Protocol (HTTP/1.1)
   [https://www.rfc-editor.org/rfc/rfc7231](https://www.rfc-editor.org/rfc/rfc7231)

6. **ISO/IEC 27001:2022** – Control A.5.31 (Logging and Monitoring)
