# Rendu - Séance 2
**Nom et prénom :** <AGODA Essokpazim Maca Marina>
**Identifiant GitHub :** <agodaMarina>
**Date de soumission :** <23/06/2026>
## Résumé de la séance
<2-4 lignes : Dockerfile écrit, image construite et exécutée, stack Compose à 3 services orchestrée, notebook Jupyter lisant MinIO.>
## Étapes principales
1. Écriture du Dockerfile et construction de l'image `anfa-analyse:v1` (taille observée : XX Go).
2. Mise en place du `.dockerignore` et observation du cache de Docker.
3. Écriture du `docker-compose.yml` orchestrant MinIO, Jupyter, et l'image custom.
4. Création du notebook `exploration_minio.ipynb` qui lit les données depuis MinIO via boto3 et pandas.
## Captures d'écran
### docker compose ps
![docker compose ps](captures/docker-ps.png)
### Notebook Jupyter
![Notebook Jupyter](captures/jupyter-pandas.png)
![Notebook Jupyter](captures/jupyter-pandas2.png)
## Bonus multi-stage (optionnel)
<Si réalisé : taille image v1 vs taille image v2-multistage, gain en pourcentage.>
## Réponses aux exercices d'application

## Difficultés rencontrées
J'ai eu des difficulté sur la fin du tp pour accéder à minio mais j'ai pu les resoudres car c'étaot lié à mes credentials denifie au tp précédent

