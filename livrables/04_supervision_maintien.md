# Supervision, journalisation, sauvegarde et maintenance corrective

> Competence evaluee : `C32` — Mettre en oeuvre un systeme de supervision pour detecter, diagnostiquer et corriger bugs, incidents et failles.

## 1. Journalisation

### 1.1 Services journalises

<!-- Lister les services dont les journaux sont exploites et leur emplacement. -->

| Service | Emplacement des journaux | Niveau de detail |
| --- | --- | --- |
| Apache | /var/log/apache2/opstrack_error.log | Erreurs HTTP et applicatives |
| Apache | /var/log/apache2/opstrack_access.log | Toutes les requêtes HTTP |
| Laravel | /var/www/opstrack/storage/logs/laravel.log | Erreurs et événements applicatifs |
| MongoDB | /var/log/mongodb/mongod.log | Événements base de données NoSQL |
| MySQL | /var/log/mysql/error.log | Erreurs base de données relationnelle |
| Redis | /var/log/redis/redis-server.log | Événements cache |
| Système | /var/log/syslog | Événements système généraux |

### 1.2 Configuration de la journalisation

<!-- Decrire la configuration mise en place pour une journalisation exploitable. -->

Laravel journalise les événements applicatifs dans storage/logs/laravel.log via le canal daily (rotation quotidienne). MongoDB reçoit les journaux techniques via le service EventLogService de l'application, qui enregistre les appels API, les mises à jour de tickets et les appels webhook. Apache journalise toutes les requêtes entrantes avec le format combined, permettant de corréler les accès avec les erreurs.

## 2. Outils et configurations d'audit

<!-- Decrire les outils ou configurations d'audit mis en place pour analyser l'etat du systeme et de l'application. -->

Les outils suivants sont utilisés pour l'audit de l'environnement :

- Lecture des logs Apache pour identifier les requêtes suspectes ou les erreurs HTTP
- Lecture des logs Laravel pour identifier les exceptions applicatives
- Consultation de MongoDB pour la traçabilité des événements applicatifs
- Commande systemctl status pour vérifier l'état des services
- Commande redis-cli info pour auditer l'état du cache
- Endpoint /api/health pour vérifier la disponibilité de l'API

## 3. Supervision et alertes

### 3.1 Sondes mises en place

<!-- Decrire les sondes de supervision configurees. -->

| Sonde | Cible | Seuil ou condition | Action en cas d'alerte |
| --- | --- | --- | --- |
| curl /api/health | API Laravel | Réponse non 200 | Vérifier Apache et PHP |
| systemctl status apache2 | Serveur web | Service inactif | Redémarrer Apache |
| systemctl status mysql | Base de données | Service inactif | Redémarrer MySQL |
| systemctl status redis-server | Cache | Service inactif | Redémarrer Redis |
| systemctl status mongod | Base NoSQL | Service inactif | Redémarrer MongoDB |

### 3.2 Mecanisme d'alerte

<!-- Decrire comment les alertes sont remontees et traitees. -->

La supervision est actuellement manuelle via les commandes listées ci-dessus. En architecture cible, les alertes seraient remontées via AWS CloudWatch avec notification par email ou SMS en cas de dépassement de seuil.

## 4. Strategie de sauvegarde et restauration

### 4.1 Elements sauvegardes

| Element | Methode | Frequence | Retention |
| --- | --- | --- | --- |
| Base MySQL | mysqldump | À chaque déploiement | 7 jours |
| Code applicatif | rsync depuis qualification | À chaque déploiement | Version qualification |
| Logs Apache | Rotation automatique logrotate | Quotidienne | 14 jours |
| Logs Laravel | Rotation daily Laravel | Quotidienne | 14 jours |

### 4.2 Procedure de restauration

<!-- Decrire la procedure de restauration et, si possible, fournir une preuve de validation. -->

En cas d'incident sur la base de données, la restauration s'effectue comme suit :

    mysql -u user1 -p'NouveauMdpFort2026++' -h 127.0.0.1 opstrack < /tmp/opstrack.sql

En cas d'incident sur le code applicatif, relancer le script de déploiement depuis la qualification vers la production.

Cette procédure a été validée lors du déploiement initial de la qualification vers la production.

## 5. Diagnostic et correction du bug technique

### 5.1 Symptome observe

<!-- Decrire le symptome constate. -->

Le tableau de bord principal affichait des compteurs de tickets ne correspondant pas à l'état réel. Les KPIs (tickets ouverts, critiques, planifiés) restaient figés pendant de longues périodes malgré des mises à jour de tickets.

### 5.2 Demarche de diagnostic

<!-- Decrire les etapes de diagnostic suivies, les outils utilises, les hypotheses testees. -->

Lecture du fichier app/Http/Controllers/DashboardController.php. Identification d'un commentaire explicite signalant un défaut intentionnel : le cache Redis des KPIs était configuré avec un TTL de 30 minutes sans invalidation lors des mises à jour de tickets.

### 5.3 Cause racine identifiee

<!-- Expliquer la cause technique du bug. -->

La méthode Cache::remember stockait les KPIs pendant 30 minutes sans jamais être invalidée lors des modifications de tickets. Redis conservait donc des valeurs obsolètes pendant toute la durée du TTL.

### 5.4 Correctif applique

<!-- Decrire le correctif mis en oeuvre. -->

Fichier modifié : app/Http/Controllers/DashboardController.php

TTL réduit de 30 minutes à 1 minute :

    'kpis' => Cache::remember('dashboard.kpis', now()->addMinutes(1), function (): array {

### 5.5 Verification apres correction

<!-- Fournir une preuve que le bug est resolu (test, capture, log, etc.). -->

    php /var/www/opstrack/artisan cache:clear
    curl -I http://localhost
    HTTP/1.1 200 OK

## 6. Diagnostic et correction de la faille de securite

### 6.1 Faille identifiee

<!-- Decrire la faille de securite identifiee. -->

Le fichier .env contenait APP_DEBUG=true et APP_ENV=local. En mode debug, Laravel expose les stack traces complètes, les variables d'environnement et les requêtes SQL lors d'une erreur, ce qui peut révéler des informations sensibles à un utilisateur malveillant.

### 6.2 Demarche de diagnostic

<!-- Decrire les etapes de diagnostic suivies. -->

Lecture du fichier .env via la commande :

    grep "APP_DEBUG\|APP_ENV" /var/www/opstrack/.env

Résultat : APP_ENV=local et APP_DEBUG=true confirmés.

### 6.3 Evaluation du risque

<!-- Evaluer l'impact et la criticite de la faille. -->

Criticité haute. Une entrée mal formée sur n'importe quel endpoint expose la stack trace complète de Laravel, les chemins système, les variables d'environnement et potentiellement les secrets techniques. Cette faille est exploitable sans authentification.

### 6.4 Mesure corrective appliquee

<!-- Decrire la remediation mise en oeuvre. -->

Modification du fichier .env :

    APP_ENV=production
    APP_DEBUG=false

Puis vidage du cache de configuration :

    php /var/www/opstrack/artisan config:clear

### 6.5 Verification apres correction

<!-- Fournir une preuve que la faille est corrigee. -->

    curl -I http://localhost
    HTTP/1.1 200 OK

En cas d'erreur, Laravel retourne désormais une page générique sans exposition de détails techniques.

## 7. Autres observations

<!-- Documenter toute autre anomalie identifiee, meme si elle n'a pas pu etre entierement corrigee pendant l'epreuve. -->

Plusieurs anomalies supplémentaires ont été identifiées et corrigées :

Bug 2 — Recherche combinée avec filtre priorité : le fichier app/Http/Controllers/Api/TicketController.php utilisait un orWhereRaw qui cassait la logique du filtre priorité. Correctif appliqué : groupement des conditions OR dans une closure where(function()).

Bug 3 — Webhook hooks.php : le WebhookController ignorait le statut reçu et forçait le statut scheduled sur tous les tickets. Correctif : utilisation de $payload['status'] pour respecter le statut transmis par le webhook.

Faille 2 — Mots de passe : DB_PASSWORD=0000 remplacé par NouveauMdpFort2026++, authentification Redis activée via requirepass.

Faille 3 — MongoDB sans authentification : identifiée, non corrigée faute de temps. A traiter en priorité.