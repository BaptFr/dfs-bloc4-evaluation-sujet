# Base de connaissances — Note de passation

Ce document est destine a un pair charge de reprendre la maintenance de l'application. Il doit permettre de comprendre le fonctionnement, les points d'attention et les procedures essentielles sans zone d'ombre majeure.

## 1. Presentation de l'application

<!-- Decrire brievement le role de l'application, ses utilisateurs cibles et ses fonctionnalites principales. -->

OpsTrack Field Service est une application de gestion des interventions terrain. Elle permet de créer et suivre des tickets d'intervention, d'affecter des techniciens, de recevoir des mises à jour externes via webhook et d'afficher un tableau de bord de supervision. Les utilisateurs cibles sont les équipes support et les techniciens terrain.

## 2. Architecture technique

<!-- Decrire l'architecture technique de l'application : composants, interactions, technologies. -->

### 2.1 Composants principaux

| Composant | Technologie | Role |
| --- | --- | --- |
| Application principale | Laravel 12 / PHP 8.4 | Interface web, API REST, traitement webhook |
| Serveur web | Apache 2.4.58 | Exposition HTTP/HTTPS, proxy vers Next.js |
| Base relationnelle | MySQL 8.0 | Tickets, utilisateurs, interventions, commentaires |
| Base NoSQL | MongoDB | Journaux techniques et événements applicatifs |
| Cache | Redis | Cache applicatif, sessions |
| Microservice | Next.js | Tableau de bord dispatch secondaire sur port 3000 |

### 2.2 Schema d'architecture

<!-- Inserer ou decrire le schema d'architecture. -->

Internet -> Apache (80/443) -> Laravel (PHP) -> MySQL / MongoDB / Redis
Apache -> Proxy /dispatch-dashboard -> Next.js (port 3000)
Webhook externe -> hooks.php -> WebhookController -> MySQL

## 3. Points d'attention connus

<!-- Lister les points de vigilance pour la maintenance : comportements particuliers, limitations, dependances critiques. -->

- MongoDB sans authentification — à sécuriser en priorité
- Un seul serveur de production sans redondance — tout incident impacte l'ensemble des services
- Le fichier .env contient les secrets en clair — migration vers un gestionnaire de secrets recommandée
- Le certificat TLS expire le 01/07/2026 — renouvellement automatique via Certbot à vérifier
- Redis utilisé pour les sessions — un redémarrage sans persistence déconnecte tous les utilisateurs
- Le microservice Next.js (dispatch-dashboard) n'est pas encore déployé en production

## 4. Procedures operationnelles

### 4.1 Deploiement

<!-- Decrire la procedure de deploiement. -->

Le déploiement s'effectue depuis le serveur de qualification vers la production :

    rsync -avz -e "ssh -i /home/ubuntu/.ssh/ubuntu.pem" /var/www/opstrack/ ubuntu@13.37.105.89:/var/www/opstrack/
    mysqldump -u user1 -p'MOT_DE_PASSE' -h 127.0.0.1 opstrack > /tmp/opstrack.sql
    scp -i /home/ubuntu/.ssh/ubuntu.pem /tmp/opstrack.sql ubuntu@13.37.105.89:/tmp/opstrack.sql
    ssh -i /home/ubuntu/.ssh/ubuntu.pem ubuntu@13.37.105.89 "php /var/www/opstrack/artisan migrate --force"
    ssh -i /home/ubuntu/.ssh/ubuntu.pem ubuntu@13.37.105.89 "php /var/www/opstrack/artisan config:clear && php /var/www/opstrack/artisan cache:clear"

### 4.2 Sauvegarde et restauration

<!-- Decrire les procedures de sauvegarde et de restauration. -->

Sauvegarde MySQL :

    mysqldump -u user1 -p'MOT_DE_PASSE' -h 127.0.0.1 opstrack > /tmp/opstrack_$(date +%Y%m%d).sql

Restauration MySQL :

    mysql -u user1 -p'MOT_DE_PASSE' -h 127.0.0.1 opstrack < /tmp/opstrack_YYYYMMDD.sql

### 4.3 Supervision et alertes

<!-- Decrire les mecanismes de supervision en place et la conduite a tenir en cas d'alerte. -->

Vérification de l'état des services :

    systemctl status apache2 mysql redis-server mongod

Vérification de l'API :

    curl https://eval-dfs-p-tpl-20262-08.it-students.fr/api/health

Lecture des logs en cas d'incident :

    tail -n 100 /var/log/apache2/opstrack_error.log
    tail -n 100 /var/www/opstrack/storage/logs/laravel.log

### 4.4 Acces et secrets

<!-- Decrire comment acceder aux differents environnements et ou trouver les secrets (sans les divulguer). -->

- Accès SSH qualification : ubuntu@15.188.15.2 via clé ubuntu.pem
- Accès SSH production : ubuntu@13.37.105.89 via clé ubuntu.pem
- Secrets applicatifs : fichier /var/www/opstrack/.env sur chaque serveur
- Accès phpMyAdmin : voir fiche confidentielle individuelle

## 5. Bugs et failles corriges pendant l'epreuve

<!-- Resumer les corrections apportees pour que le mainteneur comprenne ce qui a change et pourquoi. -->

- DashboardController.php : cache Redis des KPIs réduit de 30min à 1min pour éviter les compteurs obsolètes
- Api/TicketController.php : correction du filtre de recherche combiné avec la priorité
- WebhookController.php : le statut reçu par le webhook est maintenant appliqué au ticket
- .env : APP_DEBUG=false et APP_ENV=production pour éviter l'exposition de données sensibles
- MySQL : mot de passe trivial remplacé, utilisateur configuré sur localhost et 127.0.0.1
- Redis : authentification activée via requirepass

## 6. Ameliorations recommandees

<!-- Lister les ameliorations non realisees pendant l'epreuve mais jugees pertinentes pour la suite. -->

- Sécuriser MongoDB avec authentification
- Migrer les secrets du .env vers AWS Secrets Manager
- Mettre en place une redondance du serveur de production
- Automatiser les sauvegardes MySQL via cron
- Déployer le microservice Next.js en production
- Mettre en place une supervision automatisée via CloudWatch

## 7. Contacts et ressources

<!-- Indiquer les references utiles : depot de code, documentation, contacts techniques. -->

- Dépôt de code : https://github.com/itakademy/dfs-bloc4-evaluation-app (accès restreint)
- Sujet d'épreuve : https://github.com/itakademy/dfs-bloc4-evaluation-sujet
- Documentation Laravel : https://laravel.com/docs/12.x
- Documentation Certbot : https://certbot.eff.org