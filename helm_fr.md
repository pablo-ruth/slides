# Helm



## Présentation

* Gestionnaire de paquets pour Kubernetes
* Créer, déployer et gérer des applications
* Opens-source et large communauté > 23k stars
* Projet porté par la CNCF
* Helm 3


## Utilisation

* Standardise le cycle de vie des applications
* Créer des packages reproductibles et modulaires
* Charts custom pour des applications internes
* Charts communautaires pour des applis existantes



## Concepts

* Charts
* Releases
* Values
* Templates


## Charts

* Format de package Helm
* Fichiers YAML avec les ressources Kubernetes
* Configurations personnalisées via des variables
* Gestion des dépendances
* Versionning


## Releases

* Instance spécifique d'un chart
* Chaque release a un nom unique qui l'identifie
* Déploiement de plusieurs releases d'un même chart


## Values

* Variables pour personnaliser la config d'une release
* Définies dans un fichier YAML ou sur la CLI
* Utilisées pour spécifier des paramètres:
  * image docker
  * ports
  * hostname...


## Templates

* Fichiers de configuration Kubernetes avec des variables
* Interpolation de "values" et de variables "built-in"
* Génération complexes avec des boucles et conditions



## Structure Chart

* Le fichier Chart.yaml est le fichier de métadonnées:
  * nom
  * version
  * description
  * dependences
* Le dossier templates contient les templates
* Le fichier values.yaml contient les values par défaut


## Arborescence

```
wordpress/
├── Chart.yaml
├── templates/
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── service.yaml
└── values.yaml
```


## Chart.yaml

* Description et versioning du Chart

```
apiVersion: v2
name: wordpress
description: A Helm chart for deploying WordPress and MariaDB
version: 0.1.0
```


## Values.yaml

* YAML avec les valeurs par défaut

```
wordpress:
  image: wordpress:6

mariadb:
  image: mariadb:10
```


## Variables built-in

* Variable prédéfinie par Helm pour récupérer des informations sur le chart, la release...

```
{{ .Chart.Name }} : le nom du chart (constant)
{{ .Chart.Version }} : la version du chart.
{{ .Release.Name }} : le nom de la release (unique).
{{ .Release.Namespace }} : l'espace de nom de la release.
```

```
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    version: {{ .Chart.Version }}
```


## Variables values

* Variable définie via un fichier "values.yaml" par défaut, un fichier value supplémentaire ou via la CLI

```
# values.yaml
replicas: 3
```

```
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicas }}
```



## Commandes (install)

* Déploiement d'une release à partir d'un Chart

```
helm install myrelease ./wordpress \
    --set wordpress.image=wordpress:latest \
    -f myvalues.yaml
```


## Commandes (upgrade)

* Mise à jour d'une release existante

```
helm upgrade myrelease ./wordpress
```

* Install ou upgrade

```
helm upgrade --install myrelease ./wordpress
```


## Commandes (list)

* List les releases et leur état

```
helm list -a
```


## Commandes (uninstall)

* Supprimer une release

```
helm uninstall myrelease
```


## TP 1-2-3-4