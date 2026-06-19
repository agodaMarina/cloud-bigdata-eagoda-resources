# Rendu Séance 1
**Nom et prénom :** AGODA Essokpazim Maca Marina
## Résumé de la séance
## Étapes principales
## Capture d'écran
## Difficultés rencontrées
## Exercices d'application
1.1 -D open source obligatoire
l'open source n'en fait pas partie — un cloud peut reposer sur des technologies entièrement propriétaires.
1.2 C. SaaS
Gmail est une application complète accessible via navigateur sans aucune installation ni gestion d'infrastructure, ce qui correspond exactement au modèle Software as a Service.
1.3  D. FaaS

1.4 C. Cloud hybride
La banque a besoin d'un environnement privé contrôlé pour ses données sensibles réglementées, tout en profitant de l'élasticité du cloud public pour ses analyses non sensibles

1.5 B. La situation où une entreprise ne peut plus changer de fournisseur sans coûts ou risques majeurs
Le vendor lock-in désigne la dépendance technologique et contractuelle qui rend le changement de fournisseur difficile, coûteux ou risqué

1.6  C. Un service open source est forcément moins performant qu'un service managé propriétaire

Exercice 2 : Classification de services
| Service | Modèle | Justification |
|---|---|---|
| Google Compute Engine | IaaS | Fournit des machines virtuelles brutes ; l'utilisateur gère l'OS, le middleware et les applications. |
| AWS Lambda | FaaS | Exécute des fonctions à la demande sans gestion de serveur, facturation à l'invocation. |
| Snowflake | SaaS | Entrepôt de données entièrement managé, accessible via interface web/SQL sans aucune infrastructure à gérer. |
| Heroku | PaaS | Plateforme d'hébergement d'applications où le développeur pousse son code ; l'infrastructure et le runtime sont gérés par Heroku. |
| Microsoft 365 | SaaS | Applications bureautiques complètes consommées via navigateur, sans installation ni gestion technique. |
| Databricks (Spark managé) | PaaS | Plateforme managée pour exécuter des jobs Spark ; le cluster est automatisé mais l'utilisateur écrit ses propres traitements. |
| Microsoft Azure Functions | FaaS | Exécution de fonctions événementielles sans serveur dédié, modèle identique à AWS Lambda. |
| Tableau Online | SaaS | Outil de visualisation entièrement hébergé dans le cloud, accessible par navigateur sans installation. |


Exercice 3 : Lecture et interprétation

| Option | Signification |
|---|---|
| `-d` | Lance le conteneur en arrière-plan (**detached mode**) — le terminal reste disponible. |
| `--name analyse-anfa` | Attribue le nom `analyse-anfa` au conteneur pour le référencer facilement (au lieu de l'ID aléatoire). |
| `-p 8888:8888` | Redirige le port **8888 de la machine hôte** vers le port **8888 du conteneur** (format `hôte:conteneur`). |
| `-v /home/koffi/notebooks:/notebooks` | Monte le dossier local `/home/koffi/notebooks` dans le conteneur au chemin `/notebooks` — les fichiers sont partagés et persistés. |
| `-e JUPYTER_TOKEN=anfa-token` | Définit la variable d'environnement `JUPYTER_TOKEN` à la valeur `anfa-token` pour sécuriser l'accès à Jupyter. |
| `jupyter/pyspark-notebook` | Image Docker utilisée (depuis Docker Hub) : un Jupyter Notebook préconfiguré avec PySpark. |

cette commande démarre en arrière-plan un environnement Jupyter Notebook avec PySpark, accessible depuis le navigateur à http://localhost:8888?token=anfa-token, avec les notebooks du dossier local /home/koffi/notebooks directement accessibles et persistés dans le conteneur.

3.2 — Lecture du docker-compose.yml
a. URLs accessibles depuis le navigateur de l'hôte :

http://localhost:9000 → API S3 compatible (pour les clients comme boto3, mc, etc.)
http://localhost:9001 → Console web d'administration MinIO

b. Que se passe-t-il après docker rm puis docker compose up -d ?
Les données ne sont pas perdues. Le volume nommé minio-data est déclaré dans la section volumes: au niveau racine du compose. Supprimer le conteneur anfa-minio ne supprime pas ce volume ; lorsqu'on relance docker compose up -d, MinIO remonte et retrouve ses données intactes dans minio-data:/data. Les données seraient perdues uniquement si on exécutait docker compose down -v (qui supprime aussi les volumes).
c. Problème de sécurité à corriger pour la production :
Le mot de passe root MINIO_ROOT_PASSWORD: secret est en clair dans le fichier docker-compose.yml, ce qui est dangereux si le fichier est versionné (Git). La correction consiste à utiliser des variables d'environnement externes ou un fichier .env non versionné (ajouté au .gitignore), voire un gestionnaire de secrets (Vault, Docker Secrets).

Exercice 4 : Diagnostic
a. Cause précise de l'erreur :
 la clé applicative créée via mc est anfa-app-key / anfa-app-secret-2026. MinIO ne reconnaît pas anfa-admin comme Access Key ID dans son registre de clés applicatives — c'est un identifiant de console, pas une clé S3.
b. Code corrigé :
python import boto3

s3 = boto3.client(
    "s3",
    endpoint_url="http://localhost:9000",
    aws_access_key_id="anfa-app-key",          # ← clé applicative créée via mc
    aws_secret_access_key="anfa-app-secret-2026",  # ← secret correspondant
    region_name="us-east-1",
)

s3.upload_file("trajets.csv", "anfa-raw", "trajets.csv")
c. Pourquoi MinIO refuse anfa-admin pour l'API S3 ?
MinIO fait une distinction entre les identifiants root (utilisés uniquement pour la console web et la CLI d'administration) et les clés d'accès applicatives (générées via mc admin user ou la console, utilisées pour l'API S3 compatible). Les credentials anfa-admin / anfa-password-2026 sont les credentials root de MinIO, réservés à l'administration ; ils ne sont pas exposés comme Access Key dans l'API S3. C'est une bonne pratique de sécurité : les applications doivent utiliser des clés dédiées avec des permissions limitées (principe du moindre privilège).

Exercice 5 : Mini-cas d'architecture
a. Deux limites concrètes de l'architecture actuelle :

Absence de temps réel : L'export CSV est mensuel, ce qui rend impossible toute prédiction à la granularité horaire souhaitée — les données sont trop vieilles pour être exploitables pour des décisions immédiates.
Scalabilité nulle et point de défaillance unique : Le PC du data scientist est une ressource fixe et personnelle ; en cas de panne, de pic de charge (vendredi soir) ou d'absence, toute la chaîne de prédiction s'arrête — il n'y a aucune élasticité ni redondance.

b. Besoins ↔ Caractéristiques NIST :
Besoin de la directionCaractéristique NISTExplicationPrédictions en quasi temps réel (chaque heure)Service mesuréOn peut déclencher et payer les calculs uniquement quand ils sont nécessaires (chaque heure), sans ressources idle.Tableau de bord partagé sans installationLibre-service à la demandeChaque analyste accède directement via navigateur aux ressources cloud sans intervention IT préalable.Augmenter la capacité lors des picsÉlasticité rapideLe cloud permet de provisionner automatiquement plus de puissance de calcul en quelques minutes lors des pics, puis de réduire.Maîtriser les coûts / pouvoir changer de fournisseurMutualisation des ressourcesLes ressources partagées entre clients permettent des coûts réduits ; combiner cela à des outils portables limite le lock-in.Données clients dans environnement contrôléMutualisation des ressources (cloud privé)Un cloud privé ou segment dédié garantit l'isolation des données sensibles tout en restant dans une logique cloud managée.
c. Modèles de service pour chaque composant :

(i) Tableau de bord partagé → SaaS : les analystes accèdent à un outil de BI hébergé sans installation, avec partage natif des rapports.
(ii) Calcul des prédictions à l'heure → FaaS  ou PaaS (ex. Databricks Jobs) : un job de prédiction déclenché toutes les heures sans serveur permanent minimise les coûts ; si les modèles sont complexes (PySpark), un PaaS managé est plus adapté.
(iii) Stockage des données clients → IaaS ou PaaS privé (ex. serveur PostgreSQL sur VM dédiée, ou un stockage objet auto-hébergé comme MinIO) : pour répondre à la contrainte de conformité, les données restent dans un environnement contrôlé par la PME.

d. Modèle de déploiement recommandé :
Cloud hybride. Les données clients sensibles sont stockées dans un cloud privé (ou on-premise) pour garantir la conformité réglementaire et la souveraineté des données. En parallèle, les calculs de prédiction et le tableau de bord partagé s'appuient sur un cloud public pour bénéficier de l'élasticité lors des pics (vendredi soir, fêtes) et réduire les coûts en dehors des périodes de pointe. Les deux environnements communiquent de manière sécurisée (VPN ou API Gateway).
e. Trois stratégies pour limiter le vendor lock-in :

Utiliser des outils open source et standards ouverts : préférer Kafka (vs Kinesis), PostgreSQL (vs Aurora propriétaire), MinIO (vs S3 natif) — ces outils fonctionnent sur n'importe quel cloud ou on-premise.
Conteneuriser les applications avec Docker/Kubernetes : les conteneurs sont portables entre AWS EKS, GCP GKE, Azure AKS ou un cluster on-premise sans réécriture de code.
Adopter une couche d'abstraction IaC multi-cloud : utiliser Terraform (plutôt que CloudFormation spécifique à AWS) pour décrire l'infrastructure de façon déclarative et reproductible sur un autre fournisseur en cas de migration.
