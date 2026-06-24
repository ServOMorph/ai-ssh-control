# Contrôle SSH du PC Linux

Quand une tâche concerne le PC Linux (services systemd, fichiers de config, logs, redémarrages), l'exécuter directement via SSH sans demander à l'utilisateur de copier-coller des commandes.

**Connexion :**

Les paramètres de connexion sont dans le fichier `.env` à la racine du projet (non versionné). Lire ce fichier avant toute commande SSH pour récupérer `SSH_KEY`, `SSH_USER` et `IP_LINUX`.

```powershell
ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $SSH_USER@$IP_LINUX "commande"
```

Si `.env` est absent, demander à l'utilisateur de lancer la procédure `init_connexion.md`.

**Règles :**
- Lire `.env` pour récupérer les paramètres de connexion
- Vérifier la connectivité avant d'exécuter (`ping $IP_LINUX -n 1`)
- Lire le retour de chaque commande et en tenir compte avant la suivante
- Ne pas utiliser de commandes interactives (nano, vim, htop)
- Pour les fichiers multi-lignes : écrire localement puis `scp`, ou utiliser `cat > fichier << 'EOF'`
- Informer l'utilisateur de ce qui est exécuté et du résultat, sans lui demander de le faire manuellement

## Protocole de déploiement de fichiers

Toute modification de fichier applicatif est d'abord faite sur Windows, puis déployée via `scp` :

```powershell
scp -i "$SSH_KEY" "<chemin_windows>" $SSH_USER@$IP_LINUX:"<chemin_linux>"
```

Redémarrer le service après déploiement si nécessaire.
