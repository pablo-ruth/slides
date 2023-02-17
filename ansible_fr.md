## Infrastructure as Code
## (Ansible)



## Présentation

* Projet opensource créé en 2012
* Codé en Python
* Développer une alternative plus simple à Puppet
* Projet très populaire > 56k stars
* En 2015, Ansible a été acquis par Red Hat
* En 2017, Red Hat a créatop, la Fondation Ansible
* En 2019, Red Hat a été racheté par IBM


## Présentation

* Automatisation de configurations de serveurs et de déploiements applicatifs
* Pas besoin d'agent installé sur les machines gérées
* Basé sur SSH pour la communication
* Utilisable en CLI ou en CI


## Présentation

* Inventaires pour lister et grouper les machines
* Configuration YAML pour décrire les tâches
* Plugins (modules) pour chaque type de tâche
* Templates pour la génération des fichiers
* Réutilisation de code avec les rôles


## Avantages

* Les fichiers de configuration peuvent être versionnés, partagés et testés
* Traçabilité et reproductibilité des modifications
* Collaboration entre les équipes via un outil de versioning (Gitlab, Github)


## Inconvénients

* Plus lent pour les grands environnements
* Pas de state et dry-run limité
* Pas de verrouillage des ressources



## TP 1

1. Installer Ansible, via:
  * Ubuntu: apt (ppa)
  * Fedora: dnf
  * Python: pip



## Arborescence

```
├── ansible.cfg
├── inventory
│   ├── hosts
│   ├── group_vars/
│   ├── host_vars/
├── roles
│   ├── common
│   │   ├── tasks/
│   │   ├── files/
│   │   ├── templates/
│   │   └── handlers/
└── deploy_web.yml
```


## TP 2

* Créer l'arborescence du projet Ansible



## Inventaire

* Fichier texte qui contient la liste des hôtes
* Le format peut être INI, YAML ou JSON
* Variables pour définir des informations
  * Adresse IP
  * Alias
  * User
* Définition des groupes


## Inventaire

```
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1 ansible_host=54.155.144.162 ansible_user=ubuntu
db2 ansible_host=54.155.144.163 ansible_user=ubuntu

[servers:children]
webservers
dbservers
```


## Inventaire

* Inventaire dynamique
* Passé en utilisant via l'option -i ou ansible.cfg

```
ansible servers -i inventory -m nom_du_module
```



## Modules

* Plugins pour les tâches spécifiques
* Ecrits en Python et exécutés sur la machine distante
  * Configurer les services
  * Installer des packages
  * Gérer des fichiers
  * https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html
* Idempotents
* Appelés individuellement ou via un playbook


## TP 3

* Création d'un inventaire
* Appel du module ping avec l'inventaire



## Playbook

* Fichier YAML qui contient des tâches à exécuter
* Sur un hôte ou un groupe d'hôtes
* Groupées par bloc
* Executées en parralèle par défaut
* Dans l'ordre où elles apparaissent


## Playbook

* Commande `ansible-playbook` et un inventaire

```
---
- name: Ping webservers
  hosts: webservers
  become: true
  tasks:
    - name: Ping all webservers
      ping:
```

```
$ ansible-playbook -i inventory/hosts example.yml
```


## Playbook (modules)

* Appels aux modules pour executer les actions
* Les modules peuvent prendre des paramètres

```
---
- name: Install webservers
  hosts: webservers
  become: true
  tasks:
    - name: Mettre à jour les paquets
      become: true
      apt:
        update_cache: yes
```


## Playbook (roles)

* Appels aux rôles pour utiliser le code mutualisé

```
- name: Install webservers
  hosts: webservers
  become: true
  roles:
    - nginx
    - mariadb
```


## Playbook (tags)

* Tags pour conditionner l'execution

```
- name: Playbook de déploiement sur les serveurs web
  hosts: webservers
  become: true
  tasks:
    - name: Installer nginx
      apt:
        name: nginx
      tags:
        - nginx
    - name: Installer mariadb-server
      apt:
        name: mariadb-server
      tags:
        - mariadb
```

```
$ ansible-playbook -i inventory/hosts example.yml --tags nginx
```


## TP 4

* Installer Mariadb via APT
  * Faire un update APT pour rafraichir les metadatas
  * Installer le package "mariadb-server"



## Boucles (loop)

* Boucle avec "loop"
* Itérer sur une liste

```
- name: Création des utilisateurs
  user:
    name: "{{ item }}"
    state: present
  loop:
    - user1
    - user2
    - user3
```


## Boucles (dict)

* Boucle avec "with_dict"
* Itérer sur un dictionnaire

```yaml
- name: Configuration des utilisateurs
  user:
    name: "{{ item.key }}"
    shell: "/bin/bash"
    password: "{{ item.value.password }}"
  with_dict:
    user1:
      password: "password1"
    user2:
      password: "password2"
    user3:
      password: "password3"
```


## TP 5

* Installer le packages avec une boucle loop
  * mariadb-server
  * python3-pymysql



## Variables

* Valeurs à utiliser dans les playbooks/rôles
* A définir au niveau:
  * inventaire
  * playbook
  * rôle
  * tâche
* Types: string, int, bool, list, dict


## Variables (inventaire)

* Dossier host_vars, fichier avec le nom de l'hôte
* Dossier group_vars, fichier avec le nom du groupe
* Disponibles pour tous les playbooks et tâches

```
inventory/
├── hosts
├── group_vars/
│   └── my_group.yml
└── host_vars/
    └── my_host.yml
```


## Variables (playbook)

* Au niveau du playbook

```
- name: Mon playbook
  hosts: webservers
  become: true
  vars:
    ma_variable: ma_valeur
```


## Variables

* Utilisation via l'interpolation `{{ }}`

```
  tasks:
    - name: Ma tâche
      debug:
        msg: "{{ ma_variable }}"
```

* Modification via une fonction

```
{{ ma_variable | default("valeur_par_defaut") }}
```


## TP 6

* Créer les variables de groupe "db_name", "db_user" et "db_password"
* Créer la variable de playbook "db_login_unix_socket"
* Créer la base de donnée et l'utilisateur



## Role

* Regroupe tâches, fichiers, templates et variables
* On peut créer un rôle avec la commande

```
ansible-galaxy init role_name
```

```
role_name/
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
└── vars
    └── main.yml
```


## Role

* Execution d'un rôle depuis un playbook

```
- name: Install webservers
  hosts: webservers
  become: true
  roles:
    - nginx
    - mariadb
```


## TP 7

* Créer un rôle "nginx"
  * Installer "nginx", "php-fpm" et "php-mysql"
  * Supprimer le fichier de configuration par défaut



## Templates

* Générer des fichiers de configuration dynamiques
* Basés sur Jinja2 qui permet d'interpoler
  * variables
  * boucles
  * conditions
  * filter...
* Stockés dans le dossier "templates", suffixe .j2


## Templates

* Fichier role_name/templates/myapp.conf.j2

```
{% if variable_name == "value" %}
  {% for item in item_list %}
    {{ item.attribute }}
  {% endfor %}
{% else %}
  {{ another_variable.attribute }}
{% endif %}
```

* Executé via le module "template"

```
- name: Render template
  template:
    src: myapp.conf.j2
    dest: /etc/myapp/main.conf
```


## TP 8

* Créer un rôle pour déployer Wordpress



## Handlers

* Tâches déclenchées uniquement lorsque nécéssaire
* Appelés via l'option notify dans une tâche
* Exécutés une seule fois après la fin du playbook
* Utiles pour la gestion des services

```
# roles/myrole/handlers/main.yml
- name: Restart nginx
  systemd:
    name: nginx
    state: restarted
```

```
- name: Copier le fichier de configuration Nginx pour WordPress
  template:
    src: templates/wordpress.conf.j2
    ...
  notify:
    - Restart nginx
```


## FIN