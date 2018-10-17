# Ansible
## Best practices



## Project Layout

```
hosts/
    dev
    qual
group_vars/
    dev/
        main.yml
        encrypted.yml   
host_vars/
   hostname1.yml

playbook.yml

roles/
    requirements.yml
    mysql/
```



## Role


## Versioning

* Each role must be in a git repository
* README
* Tags on versions
* Install roles via ansible-galaxy (requirements.yml)


## Vars

* Prefix all role vars by role name
```
kafka_version: 2.2
kafka_user: kafka
...
```


## Galaxy

requirements.yml
```
- src: git@gitserver.domain:roles/zookeeper.git
  scm: git
  version: v0.1

- src: git@gitserver.domain:roles/kafka.git
  scm: git
  version: v0.1
```
install
```
ansible-galaxy install -r roles/requirements.yml -p ./roles/
```


## Role Layout

```
kafka/
        tasks/
            main.yml
        handlers/
            main.yml
        templates/
            kafka.conf.j2
        files/
            bar.txt
        defaults/
            main.yml
        meta/
            main.yml
```



## Inventory

```
[qual-kafka]
qualkafkasrver.domain

[qual-zookeeper]
qualzookeeperserver.domain

[qual:children]
kafka
zookeeper

[kafka:children]
qual-kafka

[zookeeper:children]
qual-zookeeper
```



## Project


## Versioning

* Each project must be in a git repository
* README
* file .gitignore

```
*.retry

roles/*
!roles/requirements.yml
```


## Playbook

```
- name: Deploy Zookeeper servers
  hosts: zookeeper
  become: yes
  roles:
    - zookeeper
  tags: [zookeeper]

- name: Deploy Kafka servers
  hosts: kafka
  become: yes
  roles:
    - kafka
  tags: [kafka]
```


## Execution

All roles
```
ansible-playbook -i hosts/qual playbook.yml
```

Specific role
```
ansible-playbook -i hosts/qual playbook.yml --tags kafka
```



## Syntax


## Short notation

Avoid short notation (not valid YAML)
```
- copy: src=/srv/myfiles/foo.conf dest=/etc/foo.conf
```

YAML valid notation
```
- copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
```


## Task names

Avoid vars in task names (doesn't work in loop)
```
- name: host {{ hostname }}
```