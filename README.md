# ai-ssh-control

Protocole pour permettre à un agent IA (Claude Code, Codex) de contrôler un PC Linux depuis Windows via SSH — sans copier-coller manuel.

## Principe

L'agent IA tourne sur Windows. Il reçoit les instructions dans son fichier de configuration (`CLAUDE.md` pour Claude Code, `AGENTS.md` pour Codex). Quand une tâche concerne le PC Linux, il exécute directement les commandes SSH au lieu de les afficher pour que l'utilisateur les copie.

## Contenu

- [CONTROLE_SSH_AI.md](CONTROLE_SSH_AI.md) — documentation complète : prérequis, instruction à donner à l'agent, commandes de référence, limites, dépannage
- [CLAUDE.md](CLAUDE.md) — instruction pour Claude Code (à placer dans `.claude/CLAUDE.md`)
- [AGENTS.md](AGENTS.md) — instruction pour OpenAI Codex
- [init_connexion.md](init_connexion.md) — procédure guidée d'initialisation de la connexion SSH

## Prérequis

- PC Windows avec un agent IA capable d'exécuter des commandes shell (PowerShell / Bash)
- PC Linux avec SSH actif et une clé SSH configurée
- Les deux PC sur le même réseau (LAN, VPN, ou partage de connexion Windows)

## Initialisation rapide avec l'agent IA

Plutôt que de configurer manuellement, donne simplement ce prompt à ton agent (Claude Code ou Codex) :

```
Lis le fichier init_connexion.md et suis la procédure pour initialiser la connexion SSH.
```

L'agent te guidera étape par étape, testera la connexion, et créera automatiquement un fichier `.env` avec tes paramètres de connexion.

---

## Démarrage rapide (manuel)

1. Générer une clé SSH sur Windows :
```powershell
ssh-keygen -t ed25519 -f C:\Users\<user>\.ssh\ma_cle_ed25519
```

2. Copier la clé publique sur le Linux :
```powershell
type C:\Users\<user>\.ssh\ma_cle_ed25519.pub | ssh <user>@<IP_LINUX> "cat >> ~/.ssh/authorized_keys"
```

3. Tester la connexion :
```powershell
ssh -i "C:\Users\<user>\.ssh\ma_cle_ed25519" -o StrictHostKeyChecking=no <user>@<IP_LINUX> "echo OK"
```

4. Ajouter l'instruction dans le fichier de config de ton agent IA :
   - Claude Code → placer [CLAUDE.md](CLAUDE.md) à la racine du projet
   - Codex → placer [AGENTS.md](AGENTS.md) à la racine du projet

## Licence

MIT
