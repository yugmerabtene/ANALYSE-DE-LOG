# STRUCTURE DU DOSSIER DE TRAVAIL

```
~/cours_logs_web/
│
├── CHAPITRE_01_INTRO_LOGS/
│   ├── access_chap1.log
│   ├── tp1_exploration.sh
│   └── rapport_exploration.txt
│
├── CHAPITRE_02_EXTRACTION_PANDAS/
│   ├── access_chap2.log
│   ├── tp2_analyse.py
│   └── rapport_log.csv
│
├── CHAPITRE_03_SECURITE/
│   ├── access_malicious.log
│   ├── tp3_detection.py
│   └── rapport_securite.csv
│
├── CHAPITRE_04_SUPERVISION/
│   ├── logs_before_update.log
│   ├── logs_after_update.log
│   ├── tp4_kpi.py
│   ├── comparaison_temps.png
│   └── kpi_logs.png
│
└── CHAPITRE_05_FIL_ROUGE/
    ├── access_before.log
    ├── access_after.log
    ├── access_malicious.log
    ├── report_analyse.py
    ├── rapport_modele.md
    └── rapport_securite.csv
```

---

# INSTALLATION PRÉALABLE

Sous Ubuntu ou Debian :

```bash
sudo apt update && sudo apt install -y python3 python3-pip git unzip
pip install pandas matplotlib
mkdir -p ~/cours_logs_web
cd ~/cours_logs_web
```

---

# CHAPITRE 1 — FICHIERS ET TP

## Fichier : `access_chap1.log`

```
192.168.0.15 - - [18/Oct/2025:09:12:45 +0200] "GET /index.html HTTP/1.1" 200 1452
192.168.0.21 - - [18/Oct/2025:09:13:12 +0200] "GET /produits HTTP/1.1" 200 1632
192.168.0.22 - - [18/Oct/2025:09:13:18 +0200] "GET /contact HTTP/1.1" 404 128
192.168.0.23 - - [18/Oct/2025:09:14:01 +0200] "GET /login HTTP/1.1" 403 512
192.168.0.15 - - [18/Oct/2025:09:15:02 +0200] "POST /checkout HTTP/1.1" 500 754
192.168.0.21 - - [18/Oct/2025:09:15:18 +0200] "GET /index.html HTTP/1.1" 200 1421
```

## Script Bash : `tp1_exploration.sh`

```bash
#!/bin/bash
echo "=== Exploration du log access_chap1.log ==="
echo "Nombre total de lignes :"
wc -l access_chap1.log

echo "Top 5 des IP :"
awk '{print $1}' access_chap1.log | sort | uniq -c | sort -nr | head -5

echo "Répartition des codes HTTP :"
awk '{print $9}' access_chap1.log | sort | uniq -c | sort -nr
```

---

# CHAPITRE 2 — FICHIERS ET TP

## Fichier : `access_chap2.log`

```
192.168.0.10 - - [18/Oct/2025:09:45:12 +0200] "GET /index.html HTTP/1.1" 200 512
192.168.0.11 - - [18/Oct/2025:09:46:14 +0200] "GET /contact HTTP/1.1" 404 150
192.168.0.12 - - [18/Oct/2025:09:46:45 +0200] "GET /produits HTTP/1.1" 200 890
192.168.0.11 - - [18/Oct/2025:09:48:10 +0200] "POST /login HTTP/1.1" 403 520
192.168.0.13 - - [18/Oct/2025:09:49:52 +0200] "GET /panier HTTP/1.1" 200 432
192.168.0.12 - - [18/Oct/2025:09:51:20 +0200] "GET /checkout HTTP/1.1" 500 910
```

## Script : `tp2_analyse.py`

```python
import pandas as pd

logs = pd.read_csv("access_chap2.log", sep=' ', header=None,
                   usecols=[0,3,5,6,8],
                   names=['ip', 'datetime', 'method', 'url', 'code'])

logs['code'] = logs['code'].astype(int)

total = len(logs)
errors = len(logs[logs['code'] >= 400])
taux = round(errors / total * 100, 2)
print(f"Total requêtes : {total}")
print(f"Taux d’erreurs : {taux}%")

print("\nTop 3 IPs :")
print(logs['ip'].value_counts().head(3))

print("\nTop 3 URLs :")
print(logs['url'].value_counts().head(3))

logs.to_csv("rapport_log.csv", index=False)
print("\nRapport généré : rapport_log.csv")
```

---

# CHAPITRE 3 — FICHIERS ET TP

## Fichier : `access_malicious.log`

```
192.168.0.22 - - [20/Oct/2025:14:42:10 +0200] "POST /login.php HTTP/1.1" 401
192.168.0.22 - - [20/Oct/2025:14:42:12 +0200] "POST /login.php HTTP/1.1" 401
192.168.0.22 - - [20/Oct/2025:14:42:14 +0200] "POST /login.php HTTP/1.1" 401
192.168.0.33 - - [20/Oct/2025:14:45:00 +0200] "GET /search.php?q=UNION+SELECT+password+FROM+users HTTP/1.1" 200
192.168.0.41 - - [20/Oct/2025:14:46:03 +0200] "GET /contact?q=<script>alert('xss')</script> HTTP/1.1" 200
192.168.0.15 - - [20/Oct/2025:14:47:00 +0200] "GET /../../etc/passwd HTTP/1.1" 404
```

## Script : `tp3_detection.py`

```python
import pandas as pd
import re

logs = pd.read_csv("access_malicious.log", sep=' ', header=None,
                   usecols=[0,6], names=['ip','url'])

patterns = {
    'SQL Injection': r"SELECT|UNION|DROP|INSERT",
    'XSS': r"<script>|onerror=|onload=",
    'Brute Force': r"/login",
    'Directory Traversal': r"\.\./"
}

detections = []

for attack_type, regex in patterns.items():
    matches = logs[logs['url'].str.contains(regex, flags=re.IGNORECASE, na=False)]
    if not matches.empty:
        counts = matches['ip'].value_counts()
        for ip, n in counts.items():
            detections.append({'ip': ip, 'attaque': attack_type, 'occurences': n})

rapport = pd.DataFrame(detections)
rapport.to_csv("rapport_securite.csv", index=False)
print("Rapport généré : rapport_securite.csv")
print(rapport)
```

---

# CHAPITRE 4 — FICHIERS ET TP

## `logs_before_update.log`

```
192.168.0.10 - - [18/Oct/2025:09:45:12 +0200] "GET /index.html HTTP/1.1" 200 2.10
192.168.0.11 - - [18/Oct/2025:09:46:12 +0200] "GET /checkout HTTP/1.1" 500 2.45
192.168.0.12 - - [18/Oct/2025:09:47:48 +0200] "GET /search?q=phone HTTP/1.1" 200 1.89
```

## `logs_after_update.log`

```
192.168.0.10 - - [19/Oct/2025:10:15:12 +0200] "GET /index.html HTTP/1.1" 200 1.02
192.168.0.11 - - [19/Oct/2025:10:16:12 +0200] "GET /checkout HTTP/1.1" 200 1.22
192.168.0.12 - - [19/Oct/2025:10:16:48 +0200] "GET /search?q=phone HTTP/1.1" 200 0.98
```

## Script : `tp4_kpi.py`

```python
import pandas as pd
import matplotlib.pyplot as plt

before = pd.read_csv("logs_before_update.log", sep=' ', header=None,
                     names=['ip','date','method','url','code','time'])
after = pd.read_csv("logs_after_update.log", sep=' ', header=None,
                    names=['ip','date','method','url','code','time'])

before['time'] = before['time'].astype(float)
after['time'] = after['time'].astype(float)
before['code'] = before['code'].astype(int)
after['code'] = after['code'].astype(int)

mean_before = before['time'].mean()
mean_after = after['time'].mean()

taux_before = len(before[before['code'] >= 400]) / len(before) * 100
taux_after = len(after[after['code'] >= 400]) / len(after) * 100

print(f"Avant : {mean_before:.2f}s / {taux_before:.2f}% erreurs")
print(f"Après : {mean_after:.2f}s / {taux_after:.2f}% erreurs")

plt.bar(['Avant','Après'], [mean_before, mean_after], color=['gray','green'])
plt.title("Temps de réponse moyen")
plt.ylabel("Secondes")
plt.savefig("comparaison_temps.png")
plt.show()
```

---

# CHAPITRE 5 — déjà complet plus haut

*(cf. `access_before.log`, `access_after.log`, `access_malicious.log`, `report_analyse.py`, `rapport_modele.md`)*

---

# CONSEILS POUR TA DÉMONSTRATION DEVANT LES ÉTUDIANTS

1. **Démarre sur un terminal vide** et montre :

   ```bash
   ls -lh ~/cours_logs_web/CHAPITRE_03_SECURITE/
   cat access_malicious.log
   ```

2. Explique **la structure d’un log**, puis lance :

   ```bash
   python3 tp3_detection.py
   ```

   → Montre le `rapport_securite.csv` généré.

3. Passe ensuite à la **comparaison avant/après déploiement** :

   ```bash
   python3 CHAPITRE_04_SUPERVISION/tp4_kpi.py
   ```

   → Montre le graphique `comparaison_temps.png`.

4. Termine avec le **chapitre 5 complet** :

   ```bash
   cd ~/cours_logs_web/CHAPITRE_05_FIL_ROUGE/
   python3 report_analyse.py
   ```

   → Montre les 3 rapports : `rapport_securite.csv`, `comparaison_temps.png`, et `rapport_modele.md`.
