# devmachine

## Pré-requis

- Ansible doit être installé sur votre machine locale
- Les hôtes cibles doivent être accessibles via SSH

installation de ssh sur Ubuntu
```sh
sudo apt update
sudo apt install openssh-server
sudo apt install curl
```

Il faut avoir échangé les clés SSH entre votre machine locale et les hôtes cibles pour permettre une connexion sans mot de passe.

```sh
mkdir -p ~/.ssh
chmod 700 ~/.ssh
curl -fsSL https://github.com/TON_USERNAME.keys >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## Quickstart

Voici un petit Quickstart pour Ansible

- inventory.yaml : fichier d'inventaire qui liste les hôtes et groupes d'hôtes
- playbook.yaml : playbook Ansible qui définit les tâches à exécuter sur les hôtes

Cas d'utilisation simple
```sh
ansible-playbook -i inventory.yaml playbooks/ping/playbook.yaml
```

cas d'utilisation avec sudo
```sh
ansible-playbook -i inventory.yaml playbooks/update/playbook.yaml --ask-become-pass
```

## Packages

Les packages sont des playbooks d'agrégation. Ils commencent tous par `playbooks/update/playbook.yaml`, puis exécutent les playbooks applicatifs associés.

### Infra

- cato
- crowdstrike
- intune

```sh
ansible-playbook -i inventory.yaml packages/infra.yaml --ask-become-pass
```

### Base

- teams
- google-chrome

```sh
ansible-playbook -i inventory.yaml packages/base.yaml --ask-become-pass
```

### IA

- ollama
- lm-studio

```sh
ansible-playbook -i inventory.yaml packages/ia.yaml --ask-become-pass
```

### Dev

- vscode
- obsidian
- pycharm

```sh
ansible-playbook -i inventory.yaml packages/dev.yaml --ask-become-pass
```

### Full JetBrains

- datagrip
- dataspell
- phpstorm
- webstorm
- pycharm

```sh
ansible-playbook -i inventory.yaml packages/full-jetbrains.yaml --ask-become-pass
```



De la documentation est disponible sur le site officiel d'Ansible : https://docs.ansible.com/ansible/latest/index.html
