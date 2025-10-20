# CHAPITRE 1 — INTRODUCTION AUX LOGS

**Durée totale :** 1 h 30
**Niveau :** Bachelor 3 – Data & Business Intelligence
**Compétences visées :**

* Comprendre la nature et le rôle des journaux (logs) dans un système web.
* Identifier les différents types de logs produits par un serveur, une application ou une base de données.
* Lire, interpréter et extraire manuellement les informations essentielles d’un log.
* Reconnaître les principaux **codes de réponse HTTP** et savoir les classifier (succès, redirection, erreur).

---

## 1.1. Introduction générale

Chaque système informatique génère des **traces d’exécution** pour suivre son activité. Ces traces, appelées **logs**, constituent un élément fondamental de la supervision technique, de la sécurité, de la conformité réglementaire et de l’analyse de performance.

### Définition

Un **log** (ou journal) est un fichier texte qui enregistre automatiquement des événements produits par un système (serveur, application, réseau, base de données, système d’exploitation, etc.).
Chaque ligne représente un événement daté, souvent accompagné d’informations telles que :

* la source (adresse IP ou nom de processus) ;
* la date et l’heure ;
* l’action réalisée (requête, erreur, accès) ;
* le résultat ou le code de statut.

---

## 1.2. Rôle et importance des logs

Les logs permettent de :

| Objectif                | Utilisation concrète                                                          |
| ----------------------- | ----------------------------------------------------------------------------- |
| **Surveillance**        | Suivre la fréquentation, le comportement utilisateur, la charge du serveur    |
| **Diagnostic**          | Identifier les erreurs applicatives ou serveurs (codes 4xx, 5xx)              |
| **Sécurité**            | Détecter les tentatives d’intrusion, brute-force, injections, etc.            |
| **Traçabilité**         | Reconstituer la chronologie d’un incident ou d’une action utilisateur         |
| **Conformité ISO/RGPD** | Garantir l’auditabilité des traitements (exigence ISO 27001, article 32 RGPD) |

---

## 1.3. Les principaux types de logs

Un site web complet s’appuie sur plusieurs couches de traitement.
Chaque couche produit son propre type de log.

| Type de log                     | Description                                                         | Exemples de fichiers                          | Contenu typique                                     |
| ------------------------------- | ------------------------------------------------------------------- | --------------------------------------------- | --------------------------------------------------- |
| **Logs serveur web**            | Historique de toutes les requêtes HTTP reçues (Apache, Nginx, IIS). | `/var/log/apache2/access.log`                 | IP client, méthode HTTP, code de retour, user-agent |
| **Logs d’erreurs**              | Regroupe toutes les anomalies de traitement ou d’exécution.         | `/var/log/apache2/error.log`                  | erreurs PHP, erreurs 500, scripts non trouvés       |
| **Logs applicatifs**            | Messages générés par le code de l’application.                      | `app.log`, `django.log`, `laravel.log`        | événements personnalisés, exceptions, connexions    |
| **Logs de base de données**     | Traçabilité des requêtes SQL exécutées.                             | `mysql-slow.log`, `postgresql.log`            | requêtes lentes, erreurs de connexion               |
| **Logs système**                | Traces générales de l’OS.                                           | `/var/log/syslog`, `/var/log/auth.log`        | authentifications, démarrages, démons               |
| **Logs de sécurité / pare-feu** | Enregistre les alertes de sécurité réseau.                          | `/var/log/fail2ban.log`, `/var/log/suricata/` | blocage d’IP, détections d’attaques, IDS/IPS        |

---

## 1.4. Structure d’une ligne de log (exemple Apache)

**Exemple :**

```
192.168.1.10 - - [18/Oct/2025:09:45:12 +0200] "GET /index.html HTTP/1.1" 200 512 "-" "Mozilla/5.0"
```

**Explication détaillée :**

| Élément                 | Description                                           | Exemple                        |
| ----------------------- | ----------------------------------------------------- | ------------------------------ |
| Adresse IP              | Adresse du client ayant envoyé la requête             | `192.168.1.10`                 |
| Identifiant distant     | Rarement utilisé, identifiant d’authentification HTTP | `-`                            |
| Utilisateur authentifié | Nom d’utilisateur si connexion HTTP basique           | `-`                            |
| Timestamp               | Date, heure et fuseau horaire                         | `[18/Oct/2025:09:45:12 +0200]` |
| Requête                 | Méthode, ressource et protocole                       | `"GET /index.html HTTP/1.1"`   |
| Code HTTP               | Statut de la réponse du serveur                       | `200`                          |
| Taille                  | Octets transférés au client                           | `512`                          |
| Référent (referrer)     | Page précédente                                       | `"-"`                          |
| User-Agent              | Navigateur ou robot                                   | `"Mozilla/5.0"`                |

---

## 1.5. Les codes de statut HTTP (liste complète)

Les **codes HTTP** sont des nombres à trois chiffres retournés par le serveur pour indiquer le résultat d’une requête.

### 1. Codes 1xx – Informations

| Code | Signification       |
| ---- | ------------------- |
| 100  | Continue            |
| 101  | Switching Protocols |
| 102  | Processing          |
| 103  | Early Hints         |

### 2. Codes 2xx – Succès

| Code | Signification   |
| ---- | --------------- |
| 200  | OK (succès)     |
| 201  | Created         |
| 202  | Accepted        |
| 204  | No Content      |
| 206  | Partial Content |

### 3. Codes 3xx – Redirections

| Code | Signification      |
| ---- | ------------------ |
| 301  | Moved Permanently  |
| 302  | Found              |
| 303  | See Other          |
| 304  | Not Modified       |
| 307  | Temporary Redirect |
| 308  | Permanent Redirect |

### 4. Codes 4xx – Erreurs côté client

| Code | Signification      |
| ---- | ------------------ |
| 400  | Bad Request        |
| 401  | Unauthorized       |
| 403  | Forbidden          |
| 404  | Not Found          |
| 405  | Method Not Allowed |
| 408  | Request Timeout    |
| 410  | Gone               |
| 413  | Payload Too Large  |
| 429  | Too Many Requests  |

### 5. Codes 5xx – Erreurs côté serveur

| Code | Signification         |
| ---- | --------------------- |
| 500  | Internal Server Error |
| 501  | Not Implemented       |
| 502  | Bad Gateway           |
| 503  | Service Unavailable   |
| 504  | Gateway Timeout       |
| 507  | Insufficient Storage  |

---

## 1.6. Bonnes pratiques de gestion des logs

1. **Uniformiser le format de log** (Common Log Format, JSON, LTSV).
2. **Activer la rotation** (logrotate sous Linux) pour éviter les fichiers volumineux.
3. **Sécuriser l’accès** : les logs contiennent parfois des données personnelles (RGPD).
4. **Horodater en UTC** pour faciliter la corrélation multi-serveurs.
5. **Centraliser les logs** via des outils comme Syslog, Graylog, ou ELK Stack.
6. **Archiver et signer les logs critiques** (exigence ISO 27001 A.5.31).

---

## 1.7. Exercices

### Exercice 1 – Identification des fichiers de logs

**Objectif :** repérer les fichiers journaux d’un serveur Apache.

**Commande :**

```bash
ls -lh /var/log/apache2/
```

**Questions :**

1. Quels fichiers sont dédiés aux accès et aux erreurs ?
2. Que signifient les suffixes `.1` et `.gz` ?
3. Pourquoi la rotation est-elle importante ?

**Corrigé :**

1. `access.log` → requêtes HTTP, `error.log` → erreurs.
2. `.1`, `.gz` sont les versions archivées.
3. La rotation évite la surcharge disque et facilite la consultation.

---

### Exercice 2 – Lecture d’une ligne de log

**Ligne donnée :**

```
10.0.0.1 - - [20/Oct/2025:12:03:55 +0200] "POST /login HTTP/1.1" 403 210 "-" "Mozilla/5.0"
```

**Questions :**

1. Quelle méthode HTTP est utilisée ?
2. Quelle est la ressource demandée ?
3. Le code de retour indique-t-il une erreur ?
4. Quelle est la cause probable ?

**Corrigé :**

1. Méthode = `POST`
2. Ressource = `/login`
3. Code = `403` → accès refusé
4. Cause probable : authentification invalide ou accès interdit.

---

### Exercice 3 – Extraction d’erreurs HTTP

**Commande :**

```bash
grep ' 4[0-9][0-9] ' /var/log/apache2/access.log
grep ' 5[0-9][0-9] ' /var/log/apache2/access.log
```

**Résultat attendu :**
Affichage de toutes les lignes comportant une erreur client ou serveur.

---

## 1.8. Travaux pratiques (TP 1) – Exploration initiale

**Objectif :**
Découvrir la structure des logs et produire un rapport synthétique.

**Consignes :**

1. Télécharger un fichier `access.log`.
2. Identifier :

   * le nombre total de requêtes ;
   * les 5 IP les plus fréquentes ;
   * les pages les plus consultées ;
   * la répartition des codes HTTP.
3. Exporter les résultats dans `rapport_exploration.txt`.

**Commandes d’aide :**

```bash
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -5
cat access.log | awk '{print $9}' | sort | uniq -c
```

**Corrigé attendu :**

```
IP les plus actives :
  145 192.168.0.15
  120 192.168.0.21

Pages les plus consultées :
  /index.html
  /produits

Codes HTTP :
  200 : 320
  404 : 18
  500 : 3
```

**Évaluation :**

* Lecture correcte des logs (4 pts)
* Interprétation des codes (4 pts)
* Qualité du rapport (2 pts)

---

## 1.9. Lexique

| Terme                | Définition                                                                                  |
| -------------------- | ------------------------------------------------------------------------------------------- |
| **Log**              | Fichier texte consignant des événements systèmes.                                           |
| **Rotation de logs** | Processus de remplacement et archivage automatique des anciens fichiers.                    |
| **Timestamp**        | Indication précise de la date et de l’heure de l’événement.                                 |
| **User-Agent**       | Chaîne identifiant le navigateur ou robot client.                                           |
| **Code HTTP**        | Statut de la réponse envoyée par le serveur web.                                            |
| **Traçabilité**      | Capacité à retracer les actions et anomalies à partir des journaux.                         |
| **Syslog**           | Protocole standard pour la centralisation des journaux sur un serveur distant.              |
| **RGPD**             | Règlement Général sur la Protection des Données (UE 2016/679).                              |
| **ISO 27001**        | Norme internationale sur les Systèmes de Management de la Sécurité de l’Information (SMSI). |

---

## 1.10. Bibliographie et ressources

1. **Apache Foundation.** *Apache HTTP Server Log Files – Documentation officielle*.
   [https://httpd.apache.org/docs/2.4/logs.html](https://httpd.apache.org/docs/2.4/logs.html)

2. **Nginx Inc.** *Logging and Monitoring Guide*.
   [https://nginx.org/en/docs/http/ngx_http_log_module.html](https://nginx.org/en/docs/http/ngx_http_log_module.html)

3. **IETF RFC 7231.** *Hypertext Transfer Protocol (HTTP/1.1) – Semantics and Content*.
   [https://www.rfc-editor.org/rfc/rfc7231](https://www.rfc-editor.org/rfc/rfc7231)

4. **CNIL (2019).** *Comment anonymiser des données ?*
   [https://www.cnil.fr](https://www.cnil.fr)

5. **ISO/IEC 27001:2022.** *Information security, cybersecurity and privacy protection – ISMS Requirements.*

6. **OWASP (2024).** *Logging and Monitoring Cheat Sheet.*
   [https://cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org)

