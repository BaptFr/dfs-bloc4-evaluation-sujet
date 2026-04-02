# Changelog

Toutes les modifications notables apportees pendant l'epreuve sont documentees dans ce fichier.

Le format s'inspire de [Keep a Changelog](https://keepachangelog.com/).

## [Session du 2 avril 2026]

### Ajoute

<!-- Lister les elements ajoutes. -->

- Déploiement de l'application OpsTrack sur l'environnement de production
- Configuration du virtual host Apache avec le domaine public eval-dfs-p-tpl-20262-08.it-students.fr
- Certificat TLS Let's Encrypt via Certbot, valable jusqu'au 01/07/2026
- Import de la base de données MySQL depuis la qualification vers la production

### Modifie

<!-- Lister les elements modifies. -->

- app/Http/Controllers/DashboardController.php : TTL cache Redis réduit de 30 minutes à 1 minute
- app/Http/Controllers/Api/TicketController.php : groupement des conditions OR dans une closure
- app/Http/Controllers/WebhookController.php : utilisation du statut reçu dans le payload
- .env : APP_ENV=production, APP_DEBUG=false, APP_URL mis à jour

### Corrige

<!-- Lister les bugs corriges. -->

- Compteurs du tableau de bord incohérents dus au cache Redis non invalidé
- Résultats de recherche incohérents lors de la combinaison search et filtre priorité
- Webhook hooks.php ignorait le statut externe et forçait le statut scheduled

### Securite

<!-- Lister les corrections de securite appliquees. -->

- Mot de passe MySQL trivial (0000) remplacé par un mot de passe fort
- Authentification Redis activée via requirepass
- APP_DEBUG désactivé en production pour éviter l'exposition de données sensibles