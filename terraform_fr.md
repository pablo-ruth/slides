## Infrastructure as Code



## Présentation de l'IaC

* Gestion et provisionnement d'infrastructure via du code plutôt que des processus manuels
* Décrire l'infrastructure en utilisant des fichiers de configuration
* Gestion plus efficace, automatisée et fiable


## Avantages

* Les fichiers de configuration peuvent être versionnés, partagés et testés
* Traçabilité et reproductibilité des modifications
* Collaboration entre les équipes via un outil de versioning (Gitlab, Github)
* De plus en plus populaire dans l'industrie informatique


## Inconvénients

* Nécessite une période d'apprentissage
* Accroisement de la complexité en utilisant du code
* Augmentation du "blast radius"
* Plus difficile à changer rapidement
* Personnes qualifiées pour gérer les outils et configurations


### IaC et Configuration Management

* IaC se concentre sur la gestion de l'infrastructure
* CM sur la configuration des OS et des applications
* Utilisation combinée pour la gestion complète de l'infrastructure


## Outils

* IaC :
  * AWS CloudFormation, Microsoft ARM, ...
  * Terraform (HCL)
  * Pulumi (Python, Go, TypeScript...)

* Configuration Management :
  * Ansible (YAML)
  * Chef
  * Puppet
  * SaltStack



## Terraform



## Présentation

* Gestion du cycle de vie de l'infrastructure
* Opensource et écrit en Go
* Binaire unique et multi-plateforme
* En développement actif
* +36 000 étoiles sur Github
* Version actuelle 1.3.7 (04/01/2023)
* Stable depuis juin 2021


## Hashicorp

* Société fondée en 2012
* Focalisée sur la création d'outils DevOps
* A product from Hashicorp
  * Vault: gestion des secrets statiques et dynamiques
  * Consul: base clé/valeur et service discovery
  * Packer: création automatisée d'images pour VM
  * Nomad: alternative à Kubernetes



## Workflow

* Write (décrire l'infra)
* Plan (relire les changements)
* Apply (appliquer les changements)


## Arborescence

```
$ tree example-project/
.
+-- main.tf
+
+-- modules/
|   +-- nestedA/
|   |   +-- main.tf
|   |   +-- vars.tf
|   |   +-- outputs.tf
|   |   +-- versions.tf
```



## TP 1

1. Installer Terraform
* Créer le fichier vide `main.tf'
* Initialiser le projet
* Se connecter à la console AWS
  * https://console.aws.amazon.com



## Provider

* Plugin qui permet de se connecter à un service cloud
* Provider par cloud, ex: AWS, GCP, Azure
* Développés par Hashicorp et des tiers
* Publiés sur la registry (registry.terraform.io)
* Plusieurs providers en simultané


## Providers

* Core
  * lecture et interpolation des fichiers de configuration
  * gestion du state
  * communication avec les plugins
* Provider
  * abstraction logique d'une API upstream (CRUD)
  * spécifique par service (AWS, vSphere, ...)
  * officiel, partenaire, communauté


## Provider

* Définir la version et les paramètres de connexion

```
provider "aws" {
  region     = "eu-west-1"
  access_key = "XXXXXXXXXXXX"
  secret_key = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "4.54.0"
    }
  }
}
```

* Télécharger en lançant un init



## TP 2

1. Remplir le fichier `main.tf` à partir de la doc
  * https://registry.terraform.io/providers/hashicorp/aws
* Ajouter les options `access_key` et `secret_key` 
* Télécharger le provider avec l'init



## HCL

* Langage de configuration de HashiCorp
* Conçu pour être facile à lire et à écrire
* Décrit les ressources, les relations et les paramètres


## HCL

* Types de base:
  * chaînes, les nombres, les booléens et les tableaux
* Types complexes:
  * Structures et tableaux d'objets
* Support des expressions et des variables


## HCL

* Syntaxe

```
<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

* Exemple de ressource

```
resource "aws_instance" "example1" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```



## Ressource

* Représente un élément d'infrastructure
* Une VM, un bucket, un sous-réseau, une BDD...
* Peuvent être mises à jour, supprimées ou modifiées


## Ressource

* Définie dans un bloc `resource` comprenant:
  * Un type, ex: `aws_s3_bucket`
  * Un nom interne, ex: `b`
  * Des arguments, ex: `bucket`

```
resource "aws_s3_bucket" "b" {

  bucket = "pablo"

}
```



## Plan

* Rafraîchir le state
* Générer le graph de dépendance
* Afficher le résumé du plan

```
$ terraform plan
```
```
# aws_s3_bucket.b will be created
+ resource "aws_s3_bucket" "b" {
    ...
    + bucket                      = "pablo"
    ...
  }

Plan: 1 to add, 0 to change, 0 to destroy.
```


## Apply

* Créer/mettre à jour/détruire des ressources
* Paralléliser ou sérialiser les changements
* Mettre à jour le state

```
$ terraform apply
```
```
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket.b: Creating...
aws_s3_bucket.b: Creation complete after 1s [id=pablo]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```


## Destroy

* Détruire toutes les ressources gérées par Terraform

```
$ terraform destroy
```
```
  # aws_s3_bucket.b will be destroyed
  - resource "aws_s3_bucket" "b" {
      - arn    = "arn:aws:s3:::pablo" -> null
      - bucket = "pablo" -> null
      ...
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure.
  There is no undo. Only 'yes' will be accepted to confirm.
```



## TP 3

1. Créer une ressource `aws_s3_bucket`
  * Nom du bucket `prenom`
* Planifier les changements
* Appliquer la création
* Ajouter un tag
  * `Name` avec valeur `prenom`
* Appliquer le tag
* Détruire la ressource



## Interpolations

* Permet de référencer les arguments ou les attributs d'une ressource
* Utiliser les attributs des datasources
* Faire appel à des variables ou des locals
* Passer les informations sorties des modules
* Transformer ces valeurs via des fonctions


## Interpolations (ressource)

* Arguments ou attributs

```
resource "aws_instance" "web" {
  ami             = "ami-06d94a781b544c133"
  instance_type   = "t3.nano"
  security_groups = [aws_security_group.allow_ssh.name]
}

resource "aws_security_group" "allow_ssh" {
  vpc_id = "vpc-0a482dd403e03f8c0"
  ...
}
```


## Interpolations (datasource)

* Uniquement des attributs
* Ressources non managées par TF

```
data "aws_ami" "ubuntu" {
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  most_recent = true
}

resource "aws_instance" "web" {
  ami             = data.aws_ami.ubuntu.id
  instance_type   = "t3.nano"
  security_groups = [aws_security_group.allow_ssh.name]
}
```


## Interpolations (locals)

```
locals {
  ami    = "ami-06d94a781b544c133"
}

resource "aws_instance" "web" {
  ami             = local.ami
  instance_type   = "t3.nano"
  security_groups = [aws_security_group.allow_ssh.name]
}

resource "aws_instance" "bdd" {
  ami             = local.ami
  instance_type   = "t3.nano"
  security_groups = [aws_security_group.allow_ssh.name]
}
```


## Interpolations (variables)

* Pour paramètrer les modules

```
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
}

variable "instance_type" {
  default = "t2.micro"
}
```



## TP 4

* Créer la datasource `aws_ami` pour ubuntu
* Créer une ressource de type `aws_instance`
* Nom interne: `web`
* Arguments:
    * ami: appeler la datasource `aws_ami` ubuntu
    * instance_type: **t3.nano**


## TP 5

* Créer la ressource de type `aws_security_group`
* Ajouter le security group à `aws_instance`
* Nom de l'argument d'instance: `security_groups`


## TP 6

* Créer la ressource de type `aws_key_pair`
* Ajouter l'argument `key_name` à `aws_instance`



## State


## State

* Stocké dans un fichier JSON
* Stocké localement par défaut
* Peut être stocké à distance (S3, Consul, TF Cloud...)
* Gère uniquement les ressources dans le state


## State (Refresh)

* L'état est mis à jour à chaque execution
* Maintenir la synchro avec l'état de l'infrastructure
* Choisir les actions à executer pour MAJ l'infra


## State (Lock)

  * Empêcher les actions simultanées et conflictuelles
  * Verrouiller le fichier d'état pendant la modification
  * Bloquer les utilisateurs / CI le temps d'appliquer


## State (Backup)

* Versioning ou sauvegarde régulière
* Si perdu ou corrompu:
  * importation manuelle
  * recréation des ressources


## State (Remote)

* S3 permet la sauvegarde et le versioning
* DynamoDB pour le lock du state

```
terraform {
  backend "s3" {
    bucket         = "terraform-states"
    key            = "myproject"
    region         = "eu-west-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}
```


## State (Edit)

Commandes pour éditer le state

```
terraform pull       # télécharger le state distant
terraform push       # uploader le state
terraform state rm   # supprimer une ressource du state
terraform import     # importer une ressource dans le state
```



## Modules


## Modules

* Les modules Terraform permettent de regrouper du code réutilisable
* Un module peut inclure du code pour un ensemble spécifique de ressources
* La logique et les dépendances sont encapsulées
* Les modules peuvent être partagés et réutilisés


## Modules

* Les modules Terraform sont appelés dans le code principal en utilisant la directive module
* Les paramètres peuvent être passés au module pour personnaliser son utilisation
* Les sorties des modules peuvent être utilisées comme entrées pour d'autres ressources ou modules
* Les modules peuvent être hébergés localement ou sur un registry tiers tel que Terraform Registry.
* La configuration du provider reste dans le Terraform principal


## Arbo

```
$ tree example-project/
.
+-- main.tf
+
+-- modules/
|   +-- bucket/
|   |   +-- main.tf
|   |   +-- vars.tf
|   |   +-- outputs.tf
|   |   +-- versions.tf
```


## Utilisation

* Source locale, git ou registry
* Chargé lors du `terraform init`

```
# Local
module "nom_module" {
  source = "./modules/nomdumodule"
}

# Git
module "nom_module" {
  source = "git::https://github.com/utilisateur/nom_repo.git"
}

# Registry
module "nom_module" {
  source = "nom_utilisateur/nom_module"
}
```


## Variables

* Passer des valeurs pour paramétrer les modules
* Type (nombre, chaine, bool, ...) et valeur par défaut

```
variable "bucket_name" {           # vars.tf
  default = "my_bucket"
  type = string
}

resource "aws_s3_bucket" "b" {     # main.tf module
  bucket = var.bucket_name
}
```

```
module "module_b" {                # main.tf principal
  source = "./module_b"
  bucket_name = "mon_bucket"
}
```


## Outputs

* Renvoyer des valeurs à l'exterieur du module

```
# Dans le module
output "bucket_arn" {
  value = aws_s3_bucket.b.arn
  description = "Identifiant ARN du bucket"
}
```

```
# Dans le terraform principal
module "bucket_a" {
  source = "./modules/bucket"
}

module "instance_a" {
  source = "./module_b"

  bucket_arn = module.bucket_a.bucket_arn
}
```


## Boucle


## Boucle (count)

* Créer plusieurs ressources en même temps
* ID à partir de l'index (non constant)

```
# Créer 3 instance
resource "aws_instance" "server" {
  count = 3

  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}

output "server0_public_ip" {
  value = aws_instance.server[0].public_ip
}
```


## Boucle (for_each)

* Créer plusieurs ressources en fonction d'un tableau
* ID à partir de la valeur (constant)

```
locals {
  ports  = { ssh = "22", http = "80" }
}

resource "aws_security_group_rule" "in" {
  for_each = var.ports

  ...
  to_port           = each.value
  ...
}

output "security_group_rule_id_ssh" {
  value = aws_security_group_rule.in["ssh"].id
}
```


## Expressions conditionnelles

* Expression booléenne pour sélectionner une des deux valeurs

```
condition ? valeur_vraie : valeur_fausse

# Si condition est vraie, le résultat est valeur_vraie
# Si condition est fausse, le résultat est valeur_fausse
```

* Ex: Overrider une variable via une autre si définie

```
var.override_name != "" ? var.override_name : var.bucket_name
```


## Expressions conditionnelles

* Conditionner la création d'une ressource via une var

```
variable "create_resource" {
  type = bool
  default = false
}

resource "aws_instance" "example" {
  count = var.create_resource == true ? 1 : 0

  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```



## FIN



## Créer inventaire ansible

foo

## TP 15

* Créer la VM BDD
* Créer une entrée prenomweb.formation.wpg-hosting.fr