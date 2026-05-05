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

Il faut avoir échangé les clés SSH entre votre machine locale et le compte `ansible_user` des hôtes cibles pour permettre une connexion sans mot de passe. Dans ce projet, `ansible_user` vaut `bigboss`, donc les clés doivent être installées dans `/home/bigboss/.ssh/authorized_keys`.

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

## Utilisateur final

`bigboss` est le premier compte local du poste Linux. Il sert de compte d'administration et de compte de connexion Ansible via `ansible_user`.

Le playbook `playbooks/ansible-user/playbook.yaml` sert à installer les clés publiques GitHub sur le compte utilisé par Ansible, donc `bigboss` quand `ansible_user: bigboss`.

Le playbook `playbooks/user/playbook.yaml` sert à créer un second compte local pour l'utilisateur final. Ce compte est ajouté au groupe `sudo` pour pouvoir passer administrateur si besoin.

Exemple :

```yaml
devs:
  hosts:
    pcl007:
      ansible_host: 10.0.0.105
      ansible_user: bigboss
      devmachine_ansible_user_github: TON_USERNAME_GITHUB
      devmachine_end_user: prenom.nom
      # devmachine_end_user_password_hash: "$y$j9T$..."
```

- `ansible_user` reste le compte admin utilisé par Ansible pour se connecter au poste.
- `devmachine_ansible_user_github` est optionnel ; s'il est défini, ses clés publiques GitHub sont ajoutées à `authorized_keys` du compte `ansible_user`.
- `devmachine_end_user` est le compte local final à créer pour l'utilisateur du poste.
- `devmachine_end_user_password_hash` est optionnel ; s'il est défini, il sert de mot de passe initial du compte. Utiliser un hash, idéalement stocké avec Ansible Vault, pas un mot de passe en clair.

Dans `inventory.yaml`, `devmachine_end_user` est vide par défaut pour éviter de créer un mauvais compte par accident. Renseigner le vrai login de l'utilisateur final avant de lancer `playbooks/user/playbook.yaml` ou un package.

Installer ou maintenir les clés SSH de `bigboss` depuis GitHub :

```sh
ansible-playbook -i inventory.yaml playbooks/ansible-user/playbook.yaml --ask-pass --ask-become-pass
```

Si `devmachine_end_user_password_hash` n'est pas défini, créer le mot de passe après coup depuis `bigboss` :

```sh
sudo passwd prenom.nom
```

Créer ou maintenir le compte utilisateur final :

```sh
ansible-playbook -i inventory.yaml playbooks/user/playbook.yaml --ask-become-pass
```

## Intune

L'installation d'Intune se fait avec Ansible, mais l'enrôlement doit être fait depuis la session graphique de l'utilisateur final.

Procédure recommandée :

1. Créer d'abord le compte utilisateur final avec `playbooks/user/playbook.yaml`.
2. Ouvrir une session graphique avec cet utilisateur.
3. Lancer Microsoft Intune depuis cette session, sans `sudo`.

```sh
intune-portal
```

4. Se connecter avec le compte Microsoft/Entra ID professionnel de l'utilisateur.
5. Terminer l'enrôlement dans Intune et Microsoft Edge.

Ne pas enrôler le poste depuis le compte `bigboss` si ce compte sert seulement à l'administration Ansible.

## Packages

Les packages sont des playbooks d'agrégation. Ils commencent tous par `playbooks/update/playbook.yaml`, exécutent `playbooks/ansible-user/playbook.yaml` si `devmachine_ansible_user_github` est renseigné, exécutent `playbooks/user/playbook.yaml` si `devmachine_end_user` est renseigné, puis exécutent les playbooks applicatifs associés.

### Infra

- cato
- crowdstrike
- intune
- ivanti-neurons

Avant d'exécuter ce package, déposer le script Ivanti Neurons dans `playbooks/ivanti-neurons/files/ivanticloudagent-installer-ubuntu24.sh`.

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
