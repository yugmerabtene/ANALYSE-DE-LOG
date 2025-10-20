# SOMMAIRE DU MODULE

## **Analyse des logs d’un site web**

*(Bachelor Data & Business Intelligence – RNCP40857)*
**Durée totale : 14 heures (2 jours)**

---

## **CHAPITRE 1 – Introduction aux logs**

**Objectifs pédagogiques :**

* Comprendre ce qu’est un log et son rôle dans un système web.
* Identifier les différentes sources de logs (serveur, application, base de données).
* Lire et interpréter les fichiers de logs manuellement.
* Connaître les principaux codes d’état HTTP et leur signification.

**Contenu :**

1. Définition et rôle des logs
2. Typologie : logs serveur, applicatifs, sécurité, base de données
3. Structure d’un fichier log (Apache/Nginx)
4. Codes HTTP (1xx à 5xx) et leur signification complète
5. Bonnes pratiques de journalisation et rotation
6. TP1 : Exploration manuelle d’un fichier `access.log`
7. Exercices corrigés (lecture, filtrage, extraction d’erreurs)
8. Lexique du chapitre
9. Bibliographie (Apache, CNIL, ISO/IEC 27001, OWASP)

---

## **CHAPITRE 2 – Structure et extraction des logs avec Python**

**Objectifs pédagogiques :**

* Savoir lire, filtrer et manipuler des logs à l’aide de Python (Pandas, Regex).
* Produire un rapport statistique de base sur le trafic et les erreurs.
* Identifier les tendances dans les logs (pages les plus visitées, IP actives, pics d’erreurs).

**Contenu :**

1. Formats de logs (Common Log Format, JSON logs)
2. Extraction et structuration des logs
3. Utilisation de Pandas pour charger et filtrer des logs
4. Nettoyage et typage des données (codes HTTP, dates, heures)
5. Calculs statistiques (taux d’erreurs, top IP, top pages)
6. TP2 : Script d’analyse automatisée `tp2_analyse.py`
7. Exercices corrigés (comptage, filtrage, export CSV)
8. Lexique du chapitre
9. Bibliographie (Pandas, OWASP, RFC 7231, ISO 27001)

---

## **CHAPITRE 3 – Analyse de sécurité et détection d’attaques via les logs**

**Objectifs pédagogiques :**

* Identifier les comportements suspects et les attaques dans les logs.
* Détecter les tentatives de brute-force, injections SQL, XSS et explorations de répertoires.
* Créer un rapport de sécurité automatisé à partir des logs.
* Comprendre les obligations ISO 27001 / ANSSI liées à la journalisation.

**Contenu :**

1. Introduction à la cybersécurité et au rôle des logs
2. Typologie d’attaques détectables (Brute-force, SQLi, XSS, Directory Traversal)
3. Indicateurs d’anomalies (codes HTTP, patterns suspects, user-agents)
4. TP3 : Script de détection automatisée `tp3_detection.py`
5. Génération du rapport `rapport_securite.csv`
6. Exercices corrigés (blocage IP, filtrage, analyse par motif)
7. Conformité ISO/IEC 27001, ANSSI et RGPD (A.5.31, journalisation)
8. Lexique (SQLi, XSS, Regex, IDS/IPS, forensic)
9. Bibliographie (OWASP, ANSSI, MITRE ATT&CK)

---

## **CHAPITRE 4 – Supervision, planification des mises à jour et suivi post-déploiement**

**Objectifs pédagogiques :**

* Utiliser les logs pour suivre la performance et la stabilité d’un site web.
* Identifier les signaux faibles nécessitant une mise à jour.
* Comparer les performances avant et après déploiement.
* Construire un tableau de bord d’indicateurs (KPI : taux d’erreurs, uptime, latence).

**Contenu :**

1. Introduction à la supervision et à la maintenance applicative
2. Signaux faibles dans les logs (erreurs récurrentes, lenteurs, compatibilité)
3. Analyse comparative avant/après déploiement
4. TP4 : Script de comparaison `tp4_kpi.py`
5. Création d’un tableau de bord KPI (temps moyen, taux d’erreurs, uptime)
6. Planification d’une mise à jour supervisée (rollback, validation, post-déploiement)
7. Exercices corrigés (analyse post-déploiement, déclenchement rollback)
8. Lexique (supervision, canary release, DevOps, KPI, rollback)
9. Bibliographie (ITIL v4, ISO 27001, ANSSI, Google SRE)

---

## **CHAPITRE 5 – Étude de cas fil rouge : supervision complète d’un site e-commerce**

**Objectifs pédagogiques :**

* Appliquer l’ensemble des concepts appris à un cas réel.
* Analyser les performances et incidents de sécurité sur un site complet.
* Produire un rapport technique et décisionnel.
* Proposer un plan de correction et d’amélioration continue.

**Contenu :**

1. Présentation du cas “ShopZone” (avant/après mise à jour)
2. Fichiers à analyser : `access_before.log`, `access_after.log`, `access_malicious.log`
3. TP5 : Script d’analyse globale `report_analyse.py`
4. Détection des anomalies de performance et sécurité
5. Génération automatique du rapport Markdown `rapport_modele.md`
6. Calculs et visualisation graphique (Matplotlib)
7. Synthèse des KPI (taux d’erreur, temps de réponse, disponibilité)
8. Évaluation finale (sur 20 points)
9. Lexique final et références
10. Bibliographie (OWASP, ISO 27001, ANSSI, SRE Google, CNIL)

---

## **Annexes**

1. **Structure de fichiers pour le TP complet**
   (avec `access.log`, `tp3_detection.py`, `tp4_kpi.py`, etc.)
2. **Procédure d’installation sous Linux (Ubuntu)**
3. **Grille d’évaluation du module (20 points)**
4. **Modèle de rapport Markdown à rendre**
5. **Bibliographie générale**
