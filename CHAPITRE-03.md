# CHAPITRE 3 — ANALYSE DE SÉCURITÉ ET DÉTECTION D’ATTAQUES VIA LES LOGS

**Durée totale :** 3 h 30
**Niveau :** Bachelor 3 – Data & Business Intelligence
**Compétences visées :**

* Identifier les signes d’incidents de sécurité à partir des logs.
* Détecter des attaques par force brute, injections SQL, XSS ou explorations de répertoires.
* Exploiter les logs pour confirmer ou infirmer une compromission.
* Automatiser la détection d’événements suspects avec **Python et Regex**.
* Comprendre la valeur légale et normative de la journalisation (ISO 27001 / ANSSI).

---

## 3.1. Introduction à la sécurité par les logs

Les **logs** jouent un rôle essentiel dans la **cybersécurité** :

* Ils servent de **preuve d’incident** (traçabilité).
* Ils permettent une **détection précoce** d’anomalies.
* Ils constituent la base d’une **analyse post-incident (forensic)**.

Dans un environnement web, les logs sont souvent le premier indicateur d’une attaque :

* tentatives d’accès non autorisé à `/admin` ;
* envoi de requêtes avec des mots-clés suspects (`SELECT`, `DROP`, `<script>`) ;
* enchaînements rapides de connexions échouées (force brute).

---

## 3.2. Catégories d’attaques observables dans les logs

| Type d’attaque                 | Description                                                    | Exemple de trace dans les logs                     |
| ------------------------------ | -------------------------------------------------------------- | -------------------------------------------------- |
| **Brute-force**                | Tentatives répétées de connexion avec différents mots de passe | `/login` → code 401 répété 50 fois pour la même IP |
| **Injection SQL (SQLi)**       | Tentative d’exécuter du SQL dans un champ utilisateur          | `/search.php?q=1+UNION+SELECT+*+FROM+users`        |
| **Cross-Site Scripting (XSS)** | Injection de script HTML dans une requête                      | `/search?q=<script>alert('xss')</script>`          |
| **Directory Traversal**        | Accès non autorisé à un répertoire                             | `/../../etc/passwd`                                |
| **Scan automatisé / bots**     | Exploration massive de pages                                   | `/admin`, `/wp-login`, `/xmlrpc.php`               |
| **Command Injection**          | Insertion d’une commande système                               | `/ping?ip=8.8.8.8;cat+/etc/passwd`                 |

---

## 3.3. Indicateurs d’anomalies

Les indicateurs typiques d’incidents observables dans les logs :

1. **Multiples requêtes 401/403** en peu de temps → brute-force.
2. **Chaînes suspectes dans les URLs** : `SELECT`, `DROP`, `<script>`, `../`.
3. **Pics soudains de trafic** depuis la même IP.
4. **Accès à des chemins inexistants** (`/admin123`, `/phpmyadmin`).
5. **User-agent anormal** (ex. `sqlmap`, `nikto`, `curl`).
6. **Heures inhabituelles d’activité** (ex. 03:00 UTC pour un site local).

---

## 3.4. Analyse manuelle d’un fichier log

**Exemple de lignes suspectes (extrait d’`access_malicious.log`) :**

```
192.168.0.22 - - [20/Oct/2025:14:42:10 +0200] "GET /login.php HTTP/1.1" 401 210 "-" "Mozilla/5.0"
192.168.0.22 - - [20/Oct/2025:14:42:12 +0200] "POST /login.php HTTP/1.1" 401 198 "-" "Mozilla/5.0"
192.168.0.22 - - [20/Oct/2025:14:42:14 +0200] "POST /login.php HTTP/1.1" 401 198 "-" "Mozilla/5.0"
192.168.0.33 - - [20/Oct/2025:14:45:00 +0200] "GET /search.php?q=UNION+SELECT+password+FROM+users HTTP/1.1" 200 512 "-" "sqlmap/1.5"
192.168.0.41 - - [20/Oct/2025:14:46:03 +0200] "GET /contact?q=<script>alert('xss')</script> HTTP/1.1" 200 360 "-" "Mozilla/5.0"
```

Analyse :

* IP `192.168.0.22` : 3 échecs consécutifs sur `/login.php` → attaque brute-force probable.
* IP `192.168.0.33` : présence de “UNION SELECT” → tentative d’injection SQL.
* IP `192.168.0.41` : balise `<script>` → attaque XSS.

---

## 3.5. Détection automatisée en Python (TP3A)

### Objectif

Créer un script Python capable de détecter automatiquement les requêtes suspectes dans un fichier `access_malicious.log`.

### Énoncé

Le script doit :

1. Lire le fichier.
2. Identifier les lignes contenant des patterns suspects.
3. Générer un rapport listant :

   * les IP concernées,
   * le type de tentative,
   * le nombre d’occurrences.

---

### Corrigé Python

```python
import pandas as pd
import re

# Chargement des logs
logs = pd.read_csv("access_malicious.log", sep=' ', header=None,
                   usecols=[0, 6], names=['ip', 'url'])

# Définition des patterns
patterns = {
    'SQL Injection': r"SELECT|UNION|DROP|INSERT|UPDATE|DELETE",
    'XSS': r"<script>|onerror=|onload=",
    'Directory Traversal': r"\.\./|\.\.\\",
    'Command Injection': r";|&&|\|",
}

detections = []

# Analyse
for attack_type, regex in patterns.items():
    matches = logs[logs['url'].str.contains(regex, flags=re.IGNORECASE, na=False)]
    if not matches.empty:
        counts = matches['ip'].value_counts()
        for ip, n in counts.items():
            detections.append({'ip': ip, 'attaque': attack_type, 'occurences': n})

# Résultats
rapport = pd.DataFrame(detections)
rapport.to_csv("rapport_securite.csv", index=False)

print("Rapport généré : rapport_securite.csv")
print(rapport)
```

**Exemple de sortie :**

```
Rapport généré : rapport_securite.csv
          ip              attaque  occurences
0  192.168.0.22          Brute Force          3
1  192.168.0.33          SQL Injection        1
2  192.168.0.41          XSS                   1
```

---

## 3.6. Exercice 1 – Filtrage des connexions répétées

**Objectif :** identifier les adresses IP ayant généré plus de 5 codes 401 (Unauthorized).

**Énoncé :**
Écrire une commande Bash ou un script Python affichant ces IP.

**Corrigé :**

*Version Bash :*

```bash
grep ' 401 ' access_malicious.log | awk '{print $1}' | sort | uniq -c | awk '$1 > 5'
```

*Version Python :*

```python
import pandas as pd
logs = pd.read_csv("access_malicious.log", sep=' ', header=None, usecols=[0,8], names=['ip','code'])
logs['code'] = logs['code'].astype(int)
brute_force = logs[logs['code'] == 401]['ip'].value_counts()
print(brute_force[brute_force > 5])
```

---

## 3.7. Exercice 2 – Identifier les scans de répertoires

**Énoncé :**
Détecter les IP ayant tenté d’accéder à des fichiers sensibles :
`/wp-login`, `/admin`, `/phpmyadmin`.

**Corrigé :**

```python
keywords = ['wp-login', 'admin', 'phpmyadmin']
suspicious = logs[logs['url'].str.contains('|'.join(keywords), case=False, na=False)]
print(suspicious['ip'].value_counts())
```

---

## 3.8. Exercice 3 – Génération automatique d’un script de blocage

**Énoncé :**
À partir du rapport de détection (`rapport_securite.csv`), générer une commande `iptables` pour bloquer chaque IP détectée.

**Corrigé :**

```python
rapport = pd.read_csv("rapport_securite.csv")
for ip in rapport['ip'].unique():
    print(f"sudo iptables -A INPUT -s {ip} -j DROP")
```

**Exemple de résultat :**

```
sudo iptables -A INPUT -s 192.168.0.22 -j DROP
sudo iptables -A INPUT -s 192.168.0.33 -j DROP
sudo iptables -A INPUT -s 192.168.0.41 -j DROP
```

---

## 3.9. TP3B – Rapport de sécurité complet

### Objectif :

Construire un rapport d’analyse de sécurité à partir des logs d’un site web.

### Étapes :

1. Charger le fichier `access_malicious.log`.
2. Détecter les attaques (SQLi, XSS, brute-force, traversal).
3. Générer :

   * le nombre d’événements suspects par IP ;
   * la typologie d’attaque dominante ;
   * un score de risque (par exemple nombre_total × gravité).
4. Exporter le rapport sous format CSV ou Markdown.

---

### Exemple de sortie attendue :

```
# Rapport de sécurité – Site E-commerce (20/10/2025)

## Résumé
- 3 IP suspectes détectées
- 5 attaques SQLi, 2 XSS, 1 Directory Traversal

## Détail par IP
192.168.0.22 : 3 brute-force
192.168.0.33 : 2 injections SQL
192.168.0.41 : 1 XSS
```

---

## 3.10. Normes et obligations de journalisation

1. **ISO/IEC 27001:2022 – Contrôle A.5.31** :
   *L’organisation doit conserver des journaux d’événements afin de détecter, comprendre et répondre aux incidents de sécurité.*

2. **ANSSI – Guide de la journalisation (2018)** :

   * Les journaux doivent être **horodatés, intègres et horodatés**.
   * Leur conservation doit respecter la **rétention minimale d’un an** pour les systèmes critiques.
   * L’authentification des journaux peut être assurée par **signature cryptographique** (hash SHA256).

3. **RGPD (art. 32)** : obligation de garantir la **confidentialité et intégrité** des données de log si elles contiennent des données personnelles (IP, identifiants, etc.).

---

## 3.11. Lexique

| Terme                            | Définition                                                                               |
| -------------------------------- | ---------------------------------------------------------------------------------------- |
| **Brute-force**                  | Technique consistant à tester toutes les combinaisons possibles de mots de passe.        |
| **SQL Injection (SQLi)**         | Technique visant à manipuler des requêtes SQL pour accéder à des données non autorisées. |
| **XSS (Cross-Site Scripting)**   | Insertion de code JavaScript malveillant dans une page web.                              |
| **Directory Traversal**          | Tentative d’accès à un répertoire parent (`../`).                                        |
| **IDS/IPS**                      | Systèmes de détection/prévention d’intrusion.                                            |
| **Regex (expression régulière)** | Syntaxe permettant de rechercher des motifs textuels complexes.                          |
| **Forensic**                     | Analyse post-incident pour déterminer les causes d’une compromission.                    |
| **Signature de log**             | Hachage cryptographique permettant de garantir l’intégrité d’un fichier log.             |
| **Rétention des logs**           | Durée légale ou organisationnelle de conservation des journaux.                          |
| **Audit trail**                  | Chaîne de journaux retraçant les actions dans un système.                                |

---

## 3.12. Bibliographie et ressources

1. **ANSSI (2018)** – *Guide de la journalisation et de la détection des incidents.*
   [https://www.ssi.gouv.fr](https://www.ssi.gouv.fr)

2. **OWASP (2024)** – *Logging and Monitoring Cheat Sheet.*
   [https://cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org)

3. **MITRE ATT&CK** – *Enterprise Tactics: Discovery and Credential Access.*
   [https://attack.mitre.org](https://attack.mitre.org)

4. **ISO/IEC 27001:2022** – Contrôle A.5.31 – Logging and Monitoring.

5. **RFC 2616 & RFC 7231** – *HTTP/1.1 Specifications (Status Codes).*

6. **Python Official Docs** – *Regular Expressions (`re` module)*.
   [https://docs.python.org/3/library/re.html](https://docs.python.org/3/library/re.html)
