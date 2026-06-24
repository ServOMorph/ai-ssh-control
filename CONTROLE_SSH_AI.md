# Contrôle SSH d'un PC Linux depuis Windows via un agent IA

Permet à un agent IA (Claude Code, Cursor, Copilot, etc.) tournant sur un PC Windows d'exécuter des commandes directement sur un PC Linux sans copier-coller manuel.

---

## Prérequis

| Élément | Valeur |
|---|---|
| IP du PC Linux | ex. `192.168.1.100` (à adapter) |
| Utilisateur SSH | `root` ou tout utilisateur avec les droits nécessaires |
| Clé privée SSH | ex. `C:\Users\<user>\.ssh\ma_cle_ed25519` |
| Accès SSH actif | `PermitRootLogin yes` dans `/etc/ssh/sshd_config` si accès root |

> **Sécurité :** L'accès root SSH est pratique en phase de dev. Le désactiver avant tout déploiement public (`PermitRootLogin no`).

---

## Condition préalable

Les deux PC doivent être sur le même réseau (LAN, VPN, ou partage de connexion Windows).

**Vérifier que le Linux est joignable :**
```powershell
ping <IP_LINUX>
```
→ Si pas de réponse : vérifier que le PC Linux est allumé et connecté au réseau.

---

## Initialisation

Avant la première utilisation, lancer la procédure guidée `init_connexion.md` — elle collecte l'IP, l'utilisateur et la clé SSH, teste la connexion, puis crée automatiquement un fichier `.env` (non versionné) à la racine du projet.

---

## Instruction à donner à l'agent IA

Inclure dans les instructions système le fichier adapté à l'agent :

| Agent | Fichier à utiliser |
|---|---|
| Claude Code | `.claude/CLAUDE.md` |
| Codex (OpenAI) | `AGENTS.md` |
| Cursor, Copilot, autre | system prompt ou `.cursorrules` |

Ces fichiers indiquent à l'agent de lire `.env` pour récupérer les paramètres de connexion (`SSH_KEY`, `SSH_USER`, `IP_LINUX`) avant chaque commande SSH :

```
Quand une tâche concerne le PC Linux, l'exécuter directement via SSH sans demander à l'utilisateur de copier-coller des commandes.

Les paramètres de connexion sont dans le fichier .env à la racine du projet.
Lire .env avant toute commande SSH pour récupérer SSH_KEY, SSH_USER et IP_LINUX.

Connexion :
ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $SSH_USER@$IP_LINUX "commande"

Règles :
- Lire .env pour récupérer les paramètres de connexion
- Vérifier la connectivité avant d'exécuter
- Lire le retour de chaque commande avant la suivante
- Ne pas utiliser de commandes interactives (nano, vim, htop)
- Pour les fichiers multi-lignes : utiliser cat > fichier << 'EOF' ou écrire localement puis scp
- Informer l'utilisateur de ce qui est exécuté et du résultat
```

---

## Ce que l'agent peut faire via SSH

- Lire des fichiers de config sur le Linux
- Modifier des services systemd (`daemon-reload`, `restart`, `status`)
- Vérifier l'état de la RAM, des processus, des logs
- Redémarrer des services
- Modifier des fichiers de configuration
- Exécuter des scripts
- Déployer des fichiers via `scp`

---

## Commande SSH de référence

Les valeurs réelles sont dans `.env` — les exemples ci-dessous utilisent les variables après lecture de ce fichier.

```powershell
ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $SSH_USER@$IP_LINUX "commande"
```

**Déployer un fichier :**
```powershell
scp -i "$SSH_KEY" "C:\chemin\fichier.txt" $SSH_USER@$IP_LINUX:"/chemin/destination/"
```

---

## Limites

- **Commandes interactives impossibles** : nano, vim, htop — tout ce qui requiert un TTY interactif. Utiliser des alternatives non-interactives (`sed`, `tee`, `cat <<EOF`).
- **Réseau requis** : fonctionne uniquement quand les deux PC sont connectés.
- **Accès root** : à désactiver avant déploiement en production.

---

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| `Connection refused` | SSH non démarré sur Linux | `sudo systemctl start ssh` sur le Linux |
| `Permission denied` | Mauvaise clé ou clé non autorisée | Vérifier `/root/.ssh/authorized_keys` sur Linux |
| `No route to host` | Linux pas sur le réseau | Vérifier la connectivité réseau |
| Timeout | IP changée | Vérifier l'IP avec `ip a` sur Linux |
