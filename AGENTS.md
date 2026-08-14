# Contrôle SSH du PC Linux

Quand une tâche concerne le PC Linux (services systemd, fichiers de configuration, logs, redémarrages), l'exécuter via SSH sans demander à l'utilisateur de copier-coller des commandes. Demander une confirmation explicite avant toute action modificative, notamment un déploiement, une modification de fichier ou un redémarrage de service.

**Connexion :**

Les paramètres de connexion sont dans le fichier `.env` à la racine du projet (non versionné). Lire ce fichier avant toute commande SSH pour récupérer `SSH_KEY`, `SSH_USER` et `IP_LINUX`. Vérifier que la clé de l'hôte Linux a été contrôlée et enregistrée dans `known_hosts` ; ne jamais désactiver cette vérification.

```bash
ssh -i "$SSH_KEY" -o StrictHostKeyChecking=yes $SSH_USER@$IP_LINUX "commande"
```

Si `.env` est absent, demander à l'utilisateur de lancer la procédure `init_connexion.md`.

**Règles :**
- Lire `.env` pour récupérer les paramètres de connexion
- Vérifier la connectivité avant d'exécuter (`ping $IP_LINUX -n 1`)
- Demander confirmation avant toute commande qui modifie le système distant
- Lire le retour de chaque commande et en tenir compte avant la suivante
- Ne pas utiliser de commandes interactives (nano, vim, htop)
- Pour les fichiers multi-lignes : écrire localement puis `scp`, ou utiliser `cat > fichier << 'EOF'`
- Informer l'utilisateur de ce qui est exécuté et du résultat, sans lui demander de le faire manuellement

## Protocole de déploiement de fichiers

Toute modification de fichier applicatif est d'abord faite en local, puis déployée via `scp` après confirmation explicite :

```bash
scp -i "$SSH_KEY" "<chemin_local>" $SSH_USER@$IP_LINUX:"<chemin_linux>"
```

Redémarrer le service après déploiement si nécessaire.
