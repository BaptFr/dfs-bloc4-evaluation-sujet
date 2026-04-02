# Deploiement automatise

> Competence evaluee : `C31` — Mettre en oeuvre un systeme de deploiement automatise respectant les bonnes pratiques DevOps.

> Note : le script de déploiement décrit ci-dessous constitue la procédure formalisée issue du déploiement initial réalisé manuellement. Il est reproductible et peut être exécuté tel quel pour tout déploiement ultérieur.

## 1. Strategie de deploiement

### 1.1 Vue d'ensemble

<!-- Decrire la strategie globale de deploiement retenue : outil, flux, environnements concernes. -->

Le déploiement est réalisé via un script bash déclenché manuellement depuis le serveur de qualification vers le serveur de production. Le flux est le suivant : vérification de l'état de la qualification, transfert du code via rsync, import de la base de données, mise à jour de la configuration, puis smoke tests.

### 1.2 Diagramme du pipeline

<!-- Inserer ou decrire le flux de deploiement (etapes, declencheurs, points de controle). -->

Le pipeline suit les étapes suivantes : Qualification -> Contrôles préalables -> Transfert du code (rsync) -> Import base de données -> Configuration .env -> Smoke tests -> Production opérationnelle.

## 2. Outillage retenu

<!-- Lister et justifier les outils utilises pour le deploiement automatise. -->

| Outil | Role dans le pipeline | Justification |
| --- | --- | --- |
| rsync | Transfert du code de qualification vers production | Rapide, différentiel, natif Linux |
| mysqldump | Export et import de la base de données | Outil natif MySQL fiable |
| bash | Script de déploiement | Simple, portable, sans dépendance |
| curl | Smoke tests post-déploiement | Natif, permet de vérifier les endpoints HTTP |
| Certbot | Gestion du certificat TLS | Renouvellement automatique Let's Encrypt |

## 3. Declenchement du deploiement

<!-- Decrire le mode de declenchement : manuel, sur push, sur tag, sur merge request, etc. -->

### 3.1 Mode de declenchement

Le déploiement est déclenché manuellement via l'exécution du script deploy.sh depuis le serveur de qualification. Ce choix garantit un contrôle explicite avant toute mise en production.

### 3.2 Reproductibilite

<!-- Expliquer comment le deploiement peut etre relance de maniere identique. -->

Le script deploy.sh est idempotent — il peut être relancé plusieurs fois avec le même résultat. Chaque exécution effectue les mêmes étapes dans le même ordre.

## 4. Controles prealables au deploiement

<!-- Decrire les verifications executees avant le deploiement en production. -->

| Controle | Description | Critere de passage |
| --- | --- | --- |
| Santé de l'API | curl sur /api/health de la qualification | Réponse HTTP 200 avec status ok |
| Connexion MySQL | Vérification de la connexion à la base | Pas d'erreur de connexion |
| Connexion Redis | redis-cli ping | Réponse PONG |
| APP_DEBUG | Vérification que le debug est désactivé | APP_DEBUG=false dans .env |

## 5. Mise a jour de la production

<!-- Decrire les etapes effectives de mise a jour de l'environnement de production. -->

Transfert du code depuis la qualification :

    rsync -avz -e "ssh -i /home/ubuntu/.ssh/ubuntu.pem -o StrictHostKeyChecking=no" /var/www/opstrack/ ubuntu@13.37.105.89:/var/www/opstrack/

Export et transfert de la base de données :

    mysqldump -u user1 -p'NouveauMdpFort2026++' -h 127.0.0.1 opstrack > /tmp/opstrack.sql
    scp -i /home/ubuntu/.ssh/ubuntu.pem /tmp/opstrack.sql ubuntu@13.37.105.89:/tmp/opstrack.sql

Migrations et vidage du cache sur la production :

    ssh -i /home/ubuntu/.ssh/ubuntu.pem ubuntu@13.37.105.89 "php /var/www/opstrack/artisan migrate --force"
    ssh -i /home/ubuntu/.ssh/ubuntu.pem ubuntu@13.37.105.89 "php /var/www/opstrack/artisan config:clear && php /var/www/opstrack/artisan cache:clear"

## 6. Verification post-deploiement

### 6.1 Smoke tests

<!-- Decrire les tests de verification executes apres le deploiement. -->

| Test | Commande ou méthode | Resultat attendu |
| --- | --- | --- |
| Santé API | curl https://eval-dfs-p-tpl-20262-08.it-students.fr/api/health | HTTP 200, status ok |
| Page principale | curl -I https://eval-dfs-p-tpl-20262-08.it-students.fr | HTTP 200 |
| HTTPS actif | curl -I https://eval-dfs-p-tpl-20262-08.it-students.fr | HTTP/2 200 |
| Redirection HTTP | curl -I http://eval-dfs-p-tpl-20262-08.it-students.fr | HTTP 301 vers HTTPS |

### 6.2 Preuve de deploiement reussi

<!-- Fournir une preuve du deploiement effectif sur la production (capture, log, etc.). -->

Résultat de la commande curl sur le endpoint de santé :

    curl https://eval-dfs-p-tpl-20262-08.it-students.fr/api/health
    {"status":"ok","service":"OpsTrack","timestamp":"2026-04-02T..."}

## 7. Conduite a tenir en cas d'echec

<!-- Decrire la procedure en cas d'echec du deploiement : rollback, notification, diagnostic. -->

1. Identifier l'étape en échec via les logs du script
2. Vérifier les logs Apache : tail -n 100 /var/log/apache2/opstrack_error.log
3. Vérifier les logs Laravel : tail -n 100 /var/www/opstrack/storage/logs/laravel.log
4. En cas d'échec critique, restaurer le dump SQL de la qualification
5. Relancer le script deploy.sh après correction

## 8. Scripts et fichiers de configuration

<!-- Lister et decrire les fichiers cles du pipeline de deploiement. -->

| Fichier | Role |
| --- | --- |
| deploy.sh | Script principal de déploiement qualification vers production |
| .env | Configuration de l'environnement (non versionné) |
| /etc/apache2/sites-available/opstrack.conf | Virtual host Apache production |
| /etc/apache2/sites-available/000-default-le-ssl.conf | Configuration HTTPS Certbot |