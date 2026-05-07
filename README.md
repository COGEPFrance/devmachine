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
      # devmachine_docker_extra_users: []
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

## Paquets De Base

Le playbook `playbooks/base-packages/playbook.yaml` installe les utilitaires communs du poste :

- vim
- curl
- git
- tree
- gnome-shell-extensions
- python3-pip
- gnome-tweaks
- terminator
- net-tools

Zsh, Google Chrome et Clipboard Indicator restent dans leurs playbooks dédiés pour garder les responsabilités séparées.

Ajouter des paquets au cas par cas :

```yaml
devmachine_base_extra_packages: []
```

Installer les paquets de base seuls :

```sh
ansible-playbook -i inventory.yaml playbooks/base-packages/playbook.yaml --ask-become-pass
```

## keyd

Le playbook `playbooks/keyd/playbook.yaml` installe `keyd`, l'active au démarrage et démarre le service. Il n'installe aucun mapping par défaut pour éviter de modifier le comportement clavier avant identification précise du clavier externe.

Installer keyd seul :

```sh
ansible-playbook -i inventory.yaml playbooks/keyd/playbook.yaml --ask-become-pass
```

Identifier les touches et périphériques :

```sh
sudo keyd monitor
```

Sur certaines installations Ubuntu, le binaire est nommé `keyd.rvaiya` au lieu de `keyd`. Si `sudo keyd monitor` répond `commande introuvable`, utiliser :

```sh
sudo keyd.rvaiya monitor
```

Pour observer les événements bruts du clavier, arrêter temporairement le service :

```sh
sudo systemctl stop keyd
sudo keyd monitor
sudo systemctl start keyd
```

Remplacer `keyd` par `keyd.rvaiya` dans cette commande si nécessaire.

Un mapping pourra ensuite être ajouté via `devmachine_keyd_default_config`.

Pour ton cas Logitech MX Keys Mac, le symptôme observé est :

```text
Touche physique | Résultat actuel | Résultat attendu
@               | <               | @
<               | @               | <
```

Cela correspond normalement à un échange des touches keyd `grave` et `102nd`. Ne pas appliquer ce mapping avec `[ids] *`, sinon le clavier intégré du portable sera aussi modifié.

Configuration recommandée après récupération de l'ID du clavier externe avec `sudo keyd monitor` ou `sudo keyd.rvaiya monitor` :

```yaml
devmachine_keyd_keyboard_ids:
  - "046d:XXXX"
devmachine_keyd_swap_grave_102nd: true
```

Le playbook générera alors une configuration équivalente à :

```ini
[ids]
046d:XXXX

[main]
grave = 102nd
102nd = grave
```

## Zsh

Le playbook `playbooks/zsh/playbook.yaml` installe Zsh pour la machine entière, puis configure les comptes existants suivants :

- `ansible_user`, donc `bigboss`
- `devmachine_end_user`, si l'utilisateur final est renseigné et déjà créé
- les comptes listés dans `devmachine_zsh_extra_users`, si besoin

Par défaut, il définit Zsh comme shell de connexion et ajoute un bloc `.zshrc` minimal sans écraser le reste du fichier.

Variables utiles :

```yaml
devmachine_zsh_extra_users: []
devmachine_zsh_set_default_shell: true
devmachine_zsh_configure_zshrc: true
```

Installer Zsh seul :

```sh
ansible-playbook -i inventory.yaml playbooks/zsh/playbook.yaml --ask-become-pass
```

Après installation, fermer et rouvrir la session pour utiliser le nouveau shell par défaut.

## Docker

Le playbook `playbooks/docker/playbook.yaml` installe Docker Engine depuis le dépôt officiel Docker, avec Buildx et Docker Compose V2.

Il installe Docker pour la machine entière, puis ajoute les comptes existants suivants au groupe `docker` :

- `ansible_user`, donc `bigboss`
- `devmachine_end_user`, si l'utilisateur final est renseigné et déjà créé
- les comptes listés dans `devmachine_docker_extra_users`, si besoin

Le groupe `docker` permet de lancer des conteneurs sans `sudo`, mais il donne des privilèges équivalents à root. C'est cohérent pour un compte final qui peut déjà utiliser `sudo`.

Installer Docker seul :

```sh
ansible-playbook -i inventory.yaml playbooks/docker/playbook.yaml --ask-become-pass
```

Après installation, fermer et rouvrir la session de l'utilisateur final pour que le groupe `docker` soit pris en compte, puis vérifier :

```sh
docker run hello-world
docker compose version
```

## GNOME Clipboard Indicator

Le playbook `playbooks/gnome-clipboard-indicator/playbook.yaml` installe l'extension GNOME Shell Clipboard Indicator depuis GitHub dans le profil utilisateur.

Par défaut, l'extension cible `devmachine_end_user` si la variable est renseignée, sinon `ansible_user`. Il est possible de forcer le compte cible avec `devmachine_gnome_extensions_user`.

Le playbook détecte la version majeure de GNOME Shell et choisit la branche compatible du dépôt quand elle est connue. Il est possible de forcer la branche avec `devmachine_gnome_clipboard_indicator_version`.

Installer l'extension seule :

```sh
ansible-playbook -i inventory.yaml playbooks/gnome-clipboard-indicator/playbook.yaml --ask-become-pass
```

## Keeper

Le playbook `playbooks/keeper/playbook.yaml` installe Keeper Password Manager via Snap, avec le paquet officiel `keepersecurity`.

Installer Keeper seul :

```sh
ansible-playbook -i inventory.yaml playbooks/keeper/playbook.yaml --ask-become-pass
```

Canal Snap par défaut :

```yaml
keeper_snap_channel: latest/stable
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

- base-packages
- keyd
- zsh
- gnome-clipboard-indicator
- keeper
- google-chrome

```sh
ansible-playbook -i inventory.yaml packages/base.yaml --ask-become-pass
```

### IA

- ollama
- lm-studio

LM Studio est installé via le paquet Debian officiel fourni par `lmstudio.ai`. Le playbook télécharge `LM-Studio-0.4.12-1-x64.deb`, vérifie son checksum SHA-512, supprime les anciens artefacts AppImage créés précédemment dans `/opt/lm-studio` et `/usr/local/bin/lm-studio`, puis installe le paquet avec `apt`.

```sh
ansible-playbook -i inventory.yaml packages/ia.yaml --ask-become-pass
```

### Dev

- docker
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
