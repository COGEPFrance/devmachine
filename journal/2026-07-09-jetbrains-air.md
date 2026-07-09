# JetBrains Air

- Ajout d'un playbook `playbooks/air/playbook.yaml` pour installer JetBrains Air depuis les archives Linux officielles exposees par l'API produits JetBrains.
- Integration de JetBrains Air dans `packages/full-jetbrains.yaml`.
- Documentation de la commande de lancement du playbook Air seul dans le README.
- Correction du critere d'extraction pour reinstaller Air si un repertoire partiellement extrait contient deja `bin/Air` mais pas les jars applicatifs.
- Correction des permissions extraites par l'archive Air et ajout d'un lien `app -> lib/app` pour rendre les jars lisibles par un utilisateur normal.
