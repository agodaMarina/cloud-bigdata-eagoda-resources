# Rendu - Séance 2
**Nom et prénom :** <Votre nom complet>
**Identifiant GitHub :** <votre-username>
**Date de soumission :** <JJ/MM/AAAA>
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
## Bonus multi-stage (optionnel)
<Si réalisé : taille image v1 vs taille image v2-multistage, gain en pourcentage.>
## Réponses aux exercices d'application
<À compléter d'après les énoncés fournis avec l'assignment.>
## Difficultés rencontrées
J'ai eu des difficulté sur la fin du tp pour accéder à minio mais j'ai pu les resoudres car c'étaot lié à mes credentials denifie au tp précédent

