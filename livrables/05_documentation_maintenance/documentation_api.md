# Documentation d'API

> Si la documentation d'API n'est pas pertinente dans le contexte de l'epreuve, remplacer ce contenu par une note de justification.

## 1. Vue d'ensemble de l'API

<!-- Decrire le role de l'API, son URL de base, le format des echanges. -->

| Champ | Valeur |
| --- | --- |
| URL de base | https://eval-dfs-p-tpl-20262-08.it-students.fr/api |
| Format | JSON |
| Authentification | Token Bearer via middleware api.token |

## 2. Endpoints disponibles

<!-- Lister les endpoints principaux de l'API. -->

| Methode | Endpoint | Description | Authentification requise |
| --- | --- | --- | --- |
| GET | /api/health | Vérification de l'état de l'API | Non |
| GET | /api/v1/tickets | Liste des tickets avec filtres search et priority | Oui |
| POST | /api/v1/tickets | Création d'un ticket | Oui |
| GET | /api/v1/tickets/{id} | Détail d'un ticket | Oui |
| PUT | /api/v1/tickets/{id} | Mise à jour d'un ticket | Oui |
| GET | /api/v1/technicians | Liste des techniciens | Oui |
| GET | /api/v1/external/weather | Données météo externes | Oui |
| POST | /hooks.php | Réception des événements webhook externes | Basic Auth |

## 3. Exemples de requetes et reponses

<!-- Fournir au moins un exemple de requete et de reponse pour les endpoints principaux. -->

Vérification de l'état de l'API :

    curl https://eval-dfs-p-tpl-20262-08.it-students.fr/api/health
    {"status":"ok","service":"OpsTrack","timestamp":"2026-04-02T11:20:40+00:00"}

Récupération des tickets avec filtre :

    curl https://eval-dfs-p-tpl-20262-08.it-students.fr/api/v1/tickets?search=INC&priority=critical \
      -H "Authorization: Bearer TOKEN"

Appel webhook :

    curl -X POST https://eval-dfs-p-tpl-20262-08.it-students.fr/hooks.php \
      -u "webhook_user:webhook_password" \
      -d "ticket_reference=INC-1234&status=resolved&summary=Intervention terminee"

## 4. Codes d'erreur

<!-- Documenter les codes d'erreur retournes par l'API. -->

| Code | Signification |
| --- | --- |
| 200 | Succès |
| 401 | Non authentifié — token manquant ou invalide |
| 404 | Ressource introuvable |
| 422 | Données de la requête invalides |
| 500 | Erreur serveur interne |