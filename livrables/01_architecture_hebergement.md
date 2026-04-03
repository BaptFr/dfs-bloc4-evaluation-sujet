# Architecture cible et choix de l'hebergement

> Competence evaluee : `C29` — Selectionner une plateforme d'hebergement adaptee aux exigences techniques, economiques, qualitatives et reglementaires.

## 1. Analyse des besoins techniques

<!-- Decrire les besoins techniques de l'application a partir de l'environnement de qualification : composants, dependances, volumetrie estimee, performances attendues. -->

L'application OpsTrack Field Service repose sur les composants suivants, identifiés sur l'environnement de qualification car pas d'accès au code source:

- Laravel 12 (PHP 8.4.18) — application principale, API REST, webhook
- Apache 2.4.58 — serveur web
- MySQL — base de données relationnelle (tickets, utilisateurs et interventions)
- MongoDB —  journalisation et traçabilité des événements applicatifs
- Redis — cache applicatif et sessions
- Next.js — microservice dispatch-dashboard (service dédié)

L'application est hébergée sur AWS eu-west-3 (Paris). Elle expose une interface web, une API REST, un webhook (hooks.php) et consomme une API publique tierce.

</br>
</br>


## 2. Architecture cible proposee

### 2.1 Diagramme de deploiement

<!-- Inserer ou decrire le diagramme d'architecture cible (composants, reseaux, services managés ou auto-heberges). -->
L'architecture cible repose sur AWS eu-west-3 (Paris) et s'articule autour des couches suivantes :

- Couche exposition : Load Balancer (ALB) + certificat TLS
- Couche applicative : instances EC2 Auto Scaling (Laravel + Apache)
- Couche microservice : instance EC2 dédiée ou Lambda (Next.js dispatch-dashboard)
- Couche données : RDS MySQL, DocumentDB (MongoDB compatible), ElastiCache (Redis)
- Couche sécurité : VPC, Security Groups, IAM, Secrets Manager
- Couche supervision : CloudWatch, alertes, logs centralisés

</br>


### 2.2 Description des composants

<!-- Pour chaque composant de l'architecture cible, decrire le role, le dimensionnement retenu et le service ou la technologie choisis. -->

| Composant | Service ou technologie | Dimensionnement | Justification |
| --- | --- | --- | --- |
| Serveur web + Laravel | EC2 t3.medium | 2 vCPU, 4 Go RAM | Charge applicative standard |
| Base relationnelle | RDS MySQL t3.small | 2 vCPU, 2 Go RAM | Données transactionnelles fiables |
| Base NoSQL | DocumentDB t3.medium | 2 vCPU, 4 Go RAM | Journalisation / traçabilité |
| Cache | ElastiCache Redis t3.micro | 1 vCPU, 1 Go RAM | Sessions et cache applicatif |
| Microservice Next.js | EC2 t3.small ou Lambda | Léger | Tableau de bord secondaire |
| LoadBalancer | ALB | Managé | Exposition publique et TLS |

</br>
</br>

## 3. Choix du fournisseur et des services

<!-- Justifier le choix du fournisseur d'hebergement et des services retenus. -->
### 3.1 Fournisseur retenu
AWS (Amazon Web Services) — région eu-west-3 (Paris)

</br>

### 3.2 Justification du choix
Ce choix est justifié par la mise à disposition de:

- L'Application Load Balancer (ALB) permettant de répartir le trafic entre plusieurs instances et de maintenir le service disponible en cas de défaillance d'un composant
- du Service Level Agreement (accord de niveau de service) élevé entre 99.9% et 99.99% selon les services, garantissant une continuité de service optimale
- d'une gestion  des accès via Identity and Access Management (IAM) et des secrets via Secrets Manager, limitant les risques d'accès non autorisés
- de l'isolation réseau via Virtual Private Cloud (VPC) cloisonnant les composants entre eux
- de services de base de données managés (Relational Database Service, DocumentDB, ElastiCache) réduisant la charge d'administration et garantissant les sauvegardes automatiques
- d'un modèle de facturation à l'usage qui est adapté à une croissance progressive sans surcoût initial
- de la possibilité de choix d'un Datacenter physiquement localisé en France (Paris), réduisant la latence pour les utilisateurs français et européens et garantie la conformité au RGPD pour la localisation des données en Europe.


</br>
</br>

## 4. Estimation des couts

<!-- Fournir une estimation annuelle coherente du cout d'hebergement. -->

Cette estimation correspond à une architecture entièrement managée via AWS. Certains composants comme le certificat TLS ou la supervision peuvent être assurés par des outils open source (Let's Encrypt, Prometheus...) ce qui réduirait le coût total.

Estimation 
| Poste de depense | Cout mensuel estimé | Cout annuel estimé |
| --- | --- | --- |
| EC2 t3.medium (Laravel + Apache) | 30 € | 360 € |
| Relational Database Service MySQL t3.small | 25 € | 300 € |
| DocumentDB t3.medium (MongoDB) | 50 € | 600 € |
| ElastiCache t3.micro (Redis) | 15 € | 180 € |
| EC2 t3.small (Next.js microservice) | 15 € | 180 € |
| Application Load Balancer | 20 € | 240 € |
| Stockage et sauvegardes (S3 + snapshots) | 10 € | 120 € |
| CloudWatch (supervision et logs) | 10 € | 120 € |
| Certificat TLS via AWS Certificate Manager | 0 € | 0 € |
| **Total** |~175€ | ~2100€  |

</br>
</br>

## 5. Elasticite et evolutivite

<!-- Expliquer la strategie retenue pour absorber la croissance de l'application : scalabilite horizontale, verticale, auto-scaling, etc. -->

L'application OpsTrack repose sur Laravel, framework stateless par nature, couplé à Redis pour la gestion des sessions. Les données sont déjà séparées  de la couche applicative (MySQL, MongoDB, Redis sur des services distincts). Cette architecture permet d'envisager une montée en charge sans refonte.  Pour la croissancede l'application voilà ce qui est retenu :

- Scalabilité verticale : augmentation des ressources (CPU, RAM) des instances 
EC2 et des services managés sans interruption de service, adaptée à une 
croissance progressive et prévisible
- Scalabilité horizontale : ajout d'instances EC2 supplémentaires derrière 
l'Application Load Balancer (ALB) qui peuvent absorber les pics de charge
- Auto Scaling de AWS : configuration de règles de déclenchement automatique d'instances supplémentaires selon des seuils de CPU ou de trafic permettant une adaptation sans intervention manuelle
- Les services managés (Relational Database Service, ElastiCache, DocumentDB) 
disposent de leur propre mécanisme de mise à l'échelle

</br>
</br>

## 6. Disponibilite et continuite de service

<!-- Decrire les mesures prevues pour garantir la disponibilite : redondance, basculement, SLA cible, tolerance de panne. -->

Comme décrit pour l'architecture cible, il est visé un Service Level Agreement (accord de niveau de service) de 99.9% donc moins de 9 heures d'interruption par an. Les mesures 
suivantes sont prévues :

- Redondance applicative : plusieurs instances EC2 derrière l'Application Load 
Balancer (ALB) permettant la bascules automatique du trafic en cas de 
défaillance d'une instance
- Redondance des données : les services managés AWS (Relational Database 
Service, DocumentDB, ElastiCache) proposent une réplication multi-zones 
(Multi-AZ) garantissant la continuité en cas de défaillance d'une zone de 
disponibilité
- Sauvegardes automatiques : snapshots quotidiens des bases de données avec 
une rétention de 7 jours, permettant une restauration à un point dans le temps
- Health checks : l'Application Load Balancer vérifie en continu l'état des 
instances et retire automatiquement du trafic toute instance défaillante
- Le microservice Next.js étant un composant secondaire sa défaillance n'impacte pas le fonctionnement du coeur Laravel. 

</br>
</br>

## 7. Securite et sauvegarde

<!-- Decrire les mesures de securite integrees a l'architecture cible : isolation reseau, chiffrement, gestion des secrets, strategie de sauvegarde. -->
Plusieurs points de vigilance identifiés sur l'environnement de qualification.  Voilà les choix de sécurisation de l'architecture cible :

- Mot de passe MySQL trivial (0000) et aucune d'authentification sur MongoDB et Redis — A corriger avec toute mise en prod.
- L'accès au fichier des variables d'environnement .env n'a aucune restriciton d'accès

Les mesures de sécurité prévues sont:

- Virtual Private Cloud (VPC) : isolation réseau complète, les bases de données et services internes ne sont pas exposés publiquement
- Security Groups : règles de pare-feu limitant les accès entrants et sortants à ce qui est uniquement nécessaire (principe de moindres privilèges)
- Identity and Access Management (IAM) : gestion  des droits d'accès aux services AWS
- AWS Secrets Manager : permet de centraliser mais aussi de générer une rotation automatique des secrets (mots de passe, clés API) en remplacement du fichier .env
- Chiffrement des données au repos et en transit sur tous les services managés sur AWS

</br>
</br>

## 8. Conformite et contraintes reglementaires

<!-- Expliciter les contraintes reglementaires prises en compte : protection des donnees, localisation, tracabilite, RGPD, etc. -->
L'application OpsTrack traite des données métier (utilisateurs, tickets, 
interventions) soumises au RGPD. Les contraintes suivantes doivent être prises en compte :

- Localisation des données en France (région AWS eu-west-3 Paris) conformément 
aux exigences sur les données
- Traçabilité des accès et des actions via MongoDB (journalisation applicative) 
et CloudWatch (journalisation infrastructure)
- Gestion des secrets et des accès via AWS Secrets Manager et Identity and 
Access Management (IAM) limitant l'accès aux données sensibles
- Durée de rétention des logs et sauvegardes définie et documentée
- Chiffrement des données au repos et en transit sur l'ensemble des services