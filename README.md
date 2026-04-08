# devmachine

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



De la documentation est disponible sur le site officiel d'Ansible : https://docs.ansible.com/ansible/latest/index.html