# CHAPITRE 5 — ÉTUDE DE CAS FIL ROUGE : SUPERVISION COMPLÈTE D’UN SITE E-COMMERCE

**Durée totale :** 3 h 30 à 4 h
**Niveau :** Bachelor 3 – Data & Business Intelligence
**Compétences visées :**

* Mettre en œuvre l’ensemble des compétences acquises dans les chapitres précédents.
* Exploiter les logs d’un site e-commerce réel pour identifier les problèmes.
* Produire un **rapport de supervision complet** (technique et analytique).
* Automatiser l’analyse avec un script Python.
* Construire un tableau de bord synthétique de performance et de sécurité.

---

## 5.1. Contexte général

L’entreprise **ShopZone** gère un site e-commerce permettant la consultation de produits et le paiement en ligne.
Depuis la dernière mise à jour (v3.2 → v3.3), plusieurs anomalies ont été signalées :

* ralentissements sur les pages de paiement,
* erreurs HTTP 500 ponctuelles,
* pics de trafic sur `/login` et `/search`,
* quelques alertes du pare-feu signalant des requêtes suspectes.

L’objectif est d’effectuer une **analyse complète des logs** du site afin de :

1. Identifier les causes racines (Root Cause Analysis) ;
2. Évaluer les performances post-déploiement ;
3. Rédiger un **rapport d’analyse** avec KPI ;
4. Proposer un **plan de correction et de supervision continue**.

---

## 5.2. Fichiers de travail

À utiliser pendant ce TP :

| Fichier                | Description                                                       |
| ---------------------- | ----------------------------------------------------------------- |
| `access_before.log`    | Logs avant la mise à jour (version 3.2)                           |
| `access_after.log`     | Logs après la mise à jour (version 3.3)                           |
| `access_malicious.log` | Logs contenant des tentatives d’attaques (brute-force, SQLi, XSS) |
| `report_analyse.py`    | Script Python d’analyse automatisée                               |
| `rapport_modele.md`    | Modèle de rapport de synthèse à compléter                         |

---

## 5.3. Objectifs opérationnels du TP

1. **Importer et traiter** les fichiers logs dans Python.
2. **Analyser la performance** (temps moyen, lenteurs, pics).
3. **Détecter les attaques ou anomalies de sécurité.**
4. **Comparer les comportements avant/après mise à jour.**
5. **Rédiger un rapport final complet** (résultats + recommandations).

---

## 5.4. Étapes de réalisation

### Étape 1 – Chargement et préparation des logs

```python
import pandas as pd

before = pd.read_csv("access_before.log", sep=' ', header=None,
                     names=['ip','date','method','url','code','time'])
after = pd.read_csv("access_after.log", sep=' ', header=None,
                    names=['ip','date','method','url','code','time'])

before['code'] = before['code'].astype(int)
after['code'] = after['code'].astype(int)
before['time'] = before['time'].astype(float)
after['time'] = after['time'].astype(float)
```

---

### Étape 2 – Analyse comparative des performances

```python
mean_before = before['time'].mean()
mean_after = after['time'].mean()
taux_erreur_before = len(before[before['code'] >= 400]) / len(before) * 100
taux_erreur_after = len(after[after['code'] >= 400]) / len(after) * 100

print("=== COMPARAISON DE PERFORMANCE ===")
print(f"Temps moyen avant : {mean_before:.2f}s, après : {mean_after:.2f}s")
print(f"Taux d’erreurs avant : {taux_erreur_before:.2f}%, après : {taux_erreur_after:.2f}%")
```

---

### Étape 3 – Détection d’attaques et anomalies

```python
patterns = {
    'SQL Injection': r"SELECT|UNION|DROP|INSERT|UPDATE",
    'XSS': r"<script>|onerror=|onload=",
    'Brute Force': r"/login",
    'Traversal': r"\.\./"
}

malicious = pd.read_csv("access_malicious.log", sep=' ', header=None,
                        names=['ip','date','method','url','code'])
resultats = []

for t, reg in patterns.items():
    subset = malicious[malicious['url'].str.contains(reg, case=False, na=False)]
    for ip, count in subset['ip'].value_counts().items():
        resultats.append({'ip': ip, 'attaque': t, 'occurrences': count})

df_result = pd.DataFrame(resultats)
df_result.to_csv("rapport_securite.csv", index=False)
```

---

### Étape 4 – Génération du rapport final

**Modèle `rapport_modele.md` à compléter :**

```
# Rapport de supervision - ShopZone (v3.3)

## 1. Contexte
Version précédente : 3.2  
Version actuelle : 3.3  
Date d’analyse : {{DATE}}

## 2. Résumé des logs
- Nombre total de requêtes : {{N}}
- Taux d’erreurs global : {{X}}%
- Temps moyen de réponse : {{Y}} s

## 3. Incidents de sécurité
- SQL Injection détectée : {{nombre}}
- XSS détecté : {{nombre}}
- Brute-force détecté : {{nombre}}

## 4. Comparaison avant / après mise à jour
| Indicateur | Avant | Après | Variation |
|-------------|--------|-------|-----------|
| Temps moyen (s) | 2.1 | 1.4 | -33% |
| Taux d’erreurs (%) | 2.3 | 1.1 | -52% |

## 5. Recommandations
- Optimiser le module de paiement (analyse lenteur SQL).
- Renforcer la journalisation et la corrélation sécurité.
- Mettre en place un dashboard temps réel (ELK / Grafana).

*Fait le : {{DATE}}*  
*Responsable analyse : {{Nom étudiant}}*
```

---

### Étape 5 – Visualisation graphique

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(6,4))
plt.bar(['Avant', 'Après'], [mean_before, mean_after], color=['gray','green'])
plt.title("Comparaison du temps de réponse moyen (ShopZone)")
plt.ylabel("Secondes")
plt.savefig("comparaison_temps.png")
plt.show()
```

---

## 5.5. Évaluation

| Critère                              | Pondération |
| ------------------------------------ | ----------- |
| Chargement et nettoyage des logs     | 4 pts       |
| Analyse des erreurs et performances  | 4 pts       |
| Détection d’attaques                 | 4 pts       |
| Rapport final complet et clair       | 4 pts       |
| Interprétation et présentation orale | 4 pts       |

**Total : 20 points**

---

## 5.6. Résultat attendu (exemple)

```
=== COMPARAISON DE PERFORMANCE ===
Temps moyen avant : 2.12 s, après : 1.45 s
Taux d’erreurs avant : 2.85 %, après : 1.02 %

=== RAPPORT DE SÉCURITÉ ===
3 IP suspectes détectées :
 - 192.168.0.15 : 8 attaques SQLi
 - 192.168.0.22 : 3 brute-force
 - 192.168.0.41 : 2 XSS
```

---

## 5.7. Lexique

| Terme                                 | Définition                                                                 |
| ------------------------------------- | -------------------------------------------------------------------------- |
| **Root Cause Analysis (RCA)**         | Méthode d’analyse permettant d’identifier la cause première d’un problème. |
| **Dashboard**                         | Tableau de bord affichant les indicateurs en temps réel.                   |
| **KPIs (Key Performance Indicators)** | Indicateurs de suivi de performance et qualité.                            |
| **Logs applicatifs**                  | Journaux produits par le code du site.                                     |
| **Logs serveur web**                  | Journaux produits par le serveur HTTP (Apache, Nginx).                     |
| **Rollback**                          | Retour automatique à une version antérieure.                               |
| **Versionning**                       | Suivi des versions logicielles et des déploiements.                        |
| **Correlation logs**                  | Analyse conjointe de plusieurs sources de logs.                            |
| **Anomalie**                          | Événement sortant du comportement attendu.                                 |
| **Supervision continue**              | Observation permanente de la performance et de la sécurité.                |

---

## 5.8. Bibliographie et ressources

1. **OWASP (2024)** – Logging and Monitoring Cheat Sheet.
   [https://cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org)
2. **ANSSI (2020)** – Guide de supervision de la sécurité des systèmes d’information.
   [https://www.ssi.gouv.fr](https://www.ssi.gouv.fr)
3. **Google SRE Handbook** – Service Reliability Engineering (Postmortem Analysis).
4. **ISO/IEC 27001:2022** – Clause A.5.31 – Logging and Monitoring.
5. **Python Pandas Documentation** – Data Analysis Library.
   [https://pandas.pydata.org/docs](https://pandas.pydata.org/docs)

---

# INSTALLATION ET PRÉREQUIS SUR LINUX (environnement pédagogique)

## 1. Préparation du système

Sous Ubuntu/Debian :

```bash
sudo apt update && sudo apt install -y python3 python3-pip git unzip
```

Créer un dossier de travail :

```bash
mkdir -p ~/cours_logs_web && cd ~/cours_logs_web
```

---

## 2. Installation des bibliothèques Python

```bash
pip install pandas matplotlib
```

(Vérifie la version : `python3 -m pip show pandas` → >= 2.0)

---

## 3. Structure du répertoire à créer

```
~/cours_logs_web/
│
├── access_before.log
├── access_after.log
├── access_malicious.log
├── report_analyse.py
├── rapport_modele.md
└── requirements.txt
```

**Contenu du fichier `requirements.txt` :**

```
pandas
matplotlib
```

---

## 4. Fichiers de logs d’exemple à copier

### `access_before.log`

```
192.168.0.10 - - [18/Oct/2025:09:45:12 +0200] "GET /index.html HTTP/1.1" 200 0.95
192.168.0.11 - - [18/Oct/2025:09:46:12 +0200] "GET /checkout HTTP/1.1" 500 2.21
192.168.0.12 - - [18/Oct/2025:09:46:48 +0200] "GET /search?q=phone HTTP/1.1" 200 1.87
```

### `access_after.log`

```
192.168.0.10 - - [19/Oct/2025:10:15:12 +0200] "GET /index.html HTTP/1.1" 200 0.45
192.168.0.11 - - [19/Oct/2025:10:16:12 +0200] "GET /checkout HTTP/1.1" 200 1.02
192.168.0.12 - - [19/Oct/2025:10:16:48 +0200] "GET /search?q=phone HTTP/1.1" 200 0.87
```

### `access_malicious.log`

```
192.168.0.22 - - [20/Oct/2025:14:42:10 +0200] "POST /login.php HTTP/1.1" 401
192.168.0.33 - - [20/Oct/2025:14:45:00 +0200] "GET /search.php?q=UNION+SELECT+password+FROM+users HTTP/1.1" 200
192.168.0.41 - - [20/Oct/2025:14:46:03 +0200] "GET /contact?q=<script>alert('xss')</script> HTTP/1.1" 200
```

---

## 5. Exécution du TP

Lance le script :

```bash
python3 report_analyse.py
```

Les fichiers générés :

```
rapport_securite.csv
comparaison_temps.png
rapport_modele.md (complété)
```

---

## 6. Vérification et évaluation

* Le rapport CSV doit contenir les IP suspectes et les attaques détectées.
* Le graphique doit montrer l’amélioration post-déploiement.
* Le rapport Markdown doit être complété puis exporté en PDF (`Ctrl+P` → “Enregistrer en PDF”).

