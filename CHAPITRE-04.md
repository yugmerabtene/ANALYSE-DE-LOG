# CHAPITRE 4 — SUPERVISION, PLANIFICATION DES MISES À JOUR ET SUIVI POST-DÉPLOIEMENT VIA LES LOGS

**Durée totale :** 3 h 30
**Niveau :** Bachelor 3 – Data & Business Intelligence
**Compétences visées :**

* Identifier dans les logs les signes de dégradation des performances ou d’incompatibilités.
* Utiliser les journaux pour planifier des mises à jour applicatives.
* Définir un processus de déploiement supervisé (tests, validation, rollback).
* Exploiter les logs post-déploiement pour mesurer la stabilité et la performance.
* Construire un mini tableau de bord de suivi (KPI : taux d’erreurs, temps de réponse, disponibilité).

---

## 4.1. Introduction : le rôle des logs dans la maintenance applicative

Les logs ne servent pas uniquement à détecter des incidents ponctuels : ils permettent aussi de **superviser en continu** la santé d’un site web, d’anticiper des anomalies et d’optimiser la performance avant qu’un problème majeur ne survienne.

Les logs sont donc une **source d’indicateurs de performance et de fiabilité** :

* hausse progressive du temps de réponse ;
* répétition d’erreurs fonctionnelles (ex. formulaires invalides) ;
* erreurs spécifiques après une mise à jour logicielle ;
* messages de compatibilité navigateur ou API obsolète.

---

## 4.2. Cycle de vie d’une mise à jour

Le **cycle de mise à jour** d’une application web suit généralement quatre étapes :

| Étape                                | Description                                                    | Objectif                                 |
| ------------------------------------ | -------------------------------------------------------------- | ---------------------------------------- |
| **1. Observation préliminaire**      | Analyse des logs existants pour identifier les signaux faibles | Préparer les correctifs                  |
| **2. Déploiement supervisé**         | Mise en production progressive et contrôlée                    | Réduire le risque d’erreur               |
| **3. Vérification post-déploiement** | Suivi des logs en temps réel                                   | Détecter immédiatement les anomalies     |
| **4. Amélioration continue**         | Intégration des retours et correction                          | Optimiser la stabilité et la performance |

Les logs accompagnent donc chaque étape du **cycle DevOps** : **Build → Test → Deploy → Monitor → Improve.**

---

## 4.3. Signaux faibles observables dans les logs

Les **signaux faibles** sont des symptômes précoces d’un dysfonctionnement, souvent invisibles sans analyse.

| Type de signal             | Exemple de trace dans les logs | Risque associé                             |
| -------------------------- | ------------------------------ | ------------------------------------------ |
| Lenteurs répétées          | Temps de réponse > 2 s         | Saturation serveur ou mauvaise requête SQL |
| Erreurs 404 fréquentes     | Page ou ressource manquante    | Mauvaise configuration de routage          |
| Erreurs 500 intermittentes | Application instable           | Dépendance cassée ou bug logiciel          |
| User-agent obsolète        | “IE 9.0”                       | Incompatibilité navigateur                 |
| Erreurs de cache           | “MISS” répétés                 | Configuration CDN ou cache défaillante     |

---

## 4.4. Analyse comparative avant / après mise à jour

Les logs permettent d’évaluer l’impact réel d’une mise à jour sur les performances globales du site.

### Exemple :

* Avant la mise à jour : moyenne de 2.15 secondes par requête.
* Après la mise à jour : moyenne de 1.43 secondes.
* Amélioration : +33,4 % de rapidité.

---

### TP4A – Analyse comparative des performances

**Objectif :** comparer les temps de réponse avant et après déploiement.

**Données fournies :**

* `logs_before_update.log`
* `logs_after_update.log`

**Champs :**
`IP – Date – Méthode – URL – Code HTTP – Temps de réponse (s)`

---

### Corrigé Python

```python
import pandas as pd
import matplotlib.pyplot as plt

# Lecture des fichiers
before = pd.read_csv("logs_before_update.log", sep=' ', header=None,
                     names=['ip','date','method','url','code','time'])
after = pd.read_csv("logs_after_update.log", sep=' ', header=None,
                    names=['ip','date','method','url','code','time'])

# Conversion numérique
before['time'] = before['time'].astype(float)
after['time'] = after['time'].astype(float)

# Calculs statistiques
mean_before = before['time'].mean()
mean_after = after['time'].mean()

print(f"Temps moyen avant : {mean_before:.2f} s")
print(f"Temps moyen après : {mean_after:.2f} s")
print(f"Amélioration : {(mean_before - mean_after)/mean_before*100:.2f}%")

# Visualisation comparative
plt.boxplot([before['time'], after['time']], labels=['Avant', 'Après'])
plt.title("Comparaison du temps de réponse")
plt.ylabel("Secondes")
plt.savefig("comparaison_temps.png")
plt.show()
```

**Résultats attendus :**

```
Temps moyen avant : 2.15 s
Temps moyen après : 1.43 s
Amélioration : 33.49 %
```

---

## 4.5. Exercice 1 – Analyse des erreurs post-déploiement

**Énoncé :**
Après une mise à jour, le site présente des erreurs HTTP.
À partir de `logs_after_update.log`, affiche :

* le nombre total de 500,
* les pages concernées,
* la distribution horaire des erreurs.

**Corrigé :**

```python
import pandas as pd

logs = pd.read_csv("logs_after_update.log", sep=' ', header=None,
                   names=['ip','date','method','url','code','time'])
logs['code'] = logs['code'].astype(int)

errors_500 = logs[logs['code'] == 500]
print("Nombre d'erreurs 500 :", len(errors_500))
print("Pages concernées :")
print(errors_500['url'].value_counts().head(5))
```

---

## 4.6. Exercice 2 – Analyse du trafic après mise à jour

**Énoncé :**
Le trafic a augmenté. Vérifie si le taux d’erreur s’est aggravé ou non.

**Corrigé :**

```python
total = len(logs)
errors = len(logs[logs['code'] >= 400])
taux = round(errors / total * 100, 2)
print(f"Taux d’erreurs global : {taux}%")
```

---

## 4.7. Planification d’une mise à jour supervisée

Une mise à jour réussie s’appuie sur :

1. **Analyse préalable** des logs pour identifier les causes réelles de lenteur.
2. **Plan de déploiement progressif (canary release)** : d’abord sur un petit groupe d’utilisateurs.
3. **Rollback planifié** : retour immédiat à la version précédente en cas de défaillance.
4. **Surveillance post-déploiement** avec comparaison automatisée.

### Exemple de tableau de suivi de déploiement :

| Étape                  | Description                               | Responsable    | Indicateur de validation |
| ---------------------- | ----------------------------------------- | -------------- | ------------------------ |
| Pré-analyse            | Lecture des logs et détection d’anomalies | DevOps         | Aucun code 5xx > 0.5%    |
| Déploiement test       | Déploiement sur 10% des utilisateurs      | Chef de projet | Temps moyen < 2 s        |
| Déploiement global     | Mise à jour complète                      | Équipe Dev     | Aucun rollback           |
| Suivi post-déploiement | Analyse journalière des logs              | Analyste BI    | Taux d’erreurs stable    |

---

## 4.8. TP4B – Création d’un tableau de bord de supervision

### Objectif

Construire un tableau de bord minimal à partir des logs du site pour suivre les **indicateurs de performance (KPI)** suivants :

* Taux d’erreurs (%)
* Temps de réponse moyen
* Disponibilité du site (% uptime)

---

### Corrigé Python (avec Matplotlib)

```python
import pandas as pd
import matplotlib.pyplot as plt

logs = pd.read_csv("logs_after_update.log", sep=' ', header=None,
                   names=['ip','date','method','url','code','time'])
logs['code'] = logs['code'].astype(int)
logs['time'] = logs['time'].astype(float)

# KPI
total = len(logs)
errors = len(logs[logs['code'] >= 400])
uptime = 100 - (errors / total * 100)
mean_time = logs['time'].mean()

print(f"Taux d’erreurs : {round(errors/total*100,2)}%")
print(f"Temps de réponse moyen : {round(mean_time,2)} s")
print(f"Disponibilité (uptime) : {round(uptime,2)}%")

# Graphique
plt.bar(['Erreurs (%)','Temps moyen (s)','Uptime (%)'],
        [round(errors/total*100,2), round(mean_time,2), round(uptime,2)])
plt.title("KPI de supervision post-déploiement")
plt.savefig("kpi_logs.png")
plt.show()
```

**Exemple de sortie :**

```
Taux d’erreurs : 1.47%
Temps de réponse moyen : 1.35 s
Disponibilité : 98.53%
```

---

## 4.9. Méthodologie ISO/IEC et bonnes pratiques

* **ISO/IEC 27001:2022 – A.5.31 (Journalisation et supervision)** : obligation de surveillance active des systèmes.
* **ISO/IEC 20000-1 (IT Service Management)** : planification des changements et gestion des incidents.
* **ITIL v4** : relie les événements à la gestion du cycle de vie applicatif.
* **ANSSI (Guide PRA/PCA)** : recommande la journalisation des phases de mise à jour et rollback.

---

## 4.10. Exercice 3 – Simulation de rollback

**Énoncé :**
Après un déploiement, les temps de réponse doublent et le taux d’erreur dépasse 5 %.
Décide s’il faut déclencher un rollback.

**Corrigé :**

* Critère de rollback :
  `if taux_erreur > 5 or mean_time > 2.5: rollback = True`
* Décision :
  Si le rollback est activé, documenter la cause dans le fichier `rapport_postdeploy.txt`.

---

## 4.11. Conclusion du chapitre

À la fin de ce chapitre, l’étudiant doit être capable de :

1. Surveiller les performances d’un site web via les logs.
2. Identifier les indicateurs d’une mise à jour réussie ou défaillante.
3. Mettre en place un processus de **déploiement supervisé et réversible**.
4. Générer un rapport automatisé de suivi post-déploiement.
5. Exploiter les logs dans une approche **DevOps / ITIL / ISO 27001**.

---

## 4.12. Lexique

| Terme                               | Définition                                                                          |
| ----------------------------------- | ----------------------------------------------------------------------------------- |
| **Supervision**                     | Observation continue de l’état d’un système (disponibilité, erreurs, performances). |
| **Déploiement supervisé**           | Processus de mise à jour contrôlé par étapes (test, validation, rollback).          |
| **Rollback**                        | Retour automatique à une version précédente en cas d’échec de mise à jour.          |
| **Canary Release**                  | Technique de déploiement partiel sur un échantillon restreint.                      |
| **Uptime**                          | Pourcentage de temps pendant lequel le site reste disponible.                       |
| **KPI (Key Performance Indicator)** | Indicateur chiffré mesurant la performance.                                         |
| **DevOps**                          | Culture unifiant développement et exploitation via l’automatisation.                |
| **Monitoring**                      | Ensemble des outils et processus de supervision active.                             |
| **CI/CD**                           | Chaîne d’intégration et déploiement continus.                                       |
| **PRA/PCA**                         | Plan de Reprise et de Continuité d’Activité.                                        |

---

## 4.13. Bibliographie et ressources

1. **ISO/IEC 27001:2022** – *Information Security Management Systems, clause A.5.31 – Logging and Monitoring*.
2. **ITIL v4** – *Service Operation & Continual Improvement*.
3. **ANSSI (2020)** – *Guide pratique de supervision de la sécurité*.
4. **OWASP DevSecOps Maturity Model (2023)** – *Change and Deployment Monitoring*.
5. **Google SRE Handbook** – *Postmortems & Continuous Improvement*.
6. **RFC 2119 & 7231** – *HTTP Operational Guidelines*.
7. **Pandas / Matplotlib Official Documentation** – *Performance monitoring with Python*.
