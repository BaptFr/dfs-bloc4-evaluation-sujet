# Journal de securite

Ce document recense les failles de securite identifiees pendant l'epreuve, leur evaluation et les mesures correctives appliquees.

## Faille 1

| Champ | Description |
| --- | --- |
| Date de detection | 02/04/2026 |
| Composant concerne | .env — configuration applicative |
| Description de la faille | APP_DEBUG=true et APP_ENV=local actifs en production, exposant les stack traces, chemins système et secrets techniques lors d'une erreur |
| Severite estimee | Haute |
| Impact potentiel | Exposition de données sensibles sans authentification à tout utilisateur provoquant une erreur |
| Mesure corrective appliquee | APP_DEBUG=false et APP_ENV=production dans le .env, puis config:clear |
| Statut | Corrigé |
| Preuve de correction | curl -I http://localhost retourne HTTP 200 sans exposition de détails techniques |

## Faille 2

| Champ | Description |
| --- | --- |
| Date de detection | 02/04/2026 |
| Composant concerne | MySQL et Redis |
| Description de la faille | Mot de passe MySQL trivial (0000) et Redis sans authentification |
| Severite estimee | Haute |
| Impact potentiel | Accès non autorisé aux bases de données depuis le réseau local |
| Mesure corrective appliquee | Mot de passe MySQL remplacé par NouveauMdpFort2026++, requirepass activé sur Redis |
| Statut | Corrigé |
| Preuve de correction | Connexion MySQL et Redis validées avec les nouveaux mots de passe |

## Faille 3

| Champ | Description |
| --- | --- |
| Date de detection | 02/04/2026 |
| Composant concerne | MongoDB |
| Description de la faille | MongoDB accessible sans authentification |
| Severite estimee | Haute |
| Impact potentiel | Accès non autorisé aux journaux techniques et événements applicatifs |
| Mesure corrective appliquee | Aucune — identifiée mais non corrigée faute de temps |
| Statut | Identifie, non corrige |
| Preuve de correction | A traiter en priorité lors de la prochaine intervention |