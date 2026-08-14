# /init_connexion - Initialisation de la connexion SSH Linux

Configure la connexion SSH vers le PC Linux en guidant l'utilisateur étape par étape, puis crée un fichier `.env` local (non versionné) avec les vraies valeurs. `CLAUDE.md` et `AGENTS.md` restent génériques et lisent ce `.env` au moment de l'exécution.

---

## Prérequis pour l'agent

L'agent doit être capable de :
- Poser des questions à l'utilisateur et attendre les réponses
- Exécuter des commandes **PowerShell sur Windows** (ping, ssh, Write-Content…)
- Créer et écrire des fichiers locaux (`.env`)

Les commandes Linux ne sont **jamais exécutées directement par l'agent** — elles sont données à l'utilisateur pour qu'il les lance lui-même sur son PC Linux.

---

## Processus

### Étape 1 — Récupérer l'IP du PC Linux

**→ AGENT : poser la question suivante à l'utilisateur**

> **Quelle est l'adresse IP de ton PC Linux ?**
>
> Pour la trouver, ouvre un terminal **sur le PC Linux** et lance :
>
> ```
> [PC LINUX - terminal]
> ip a | grep "inet " | grep -v 127.0.0.1
> ```
>
> Tu verras quelque chose comme `inet 192.168.1.100/24` — l'IP est la partie avant le `/` (commence souvent par `192.168.` ou `10.`).
>
> ⚠️ Les deux PC doivent être sur le même réseau (Wi-Fi, câble, ou partage de connexion).

**→ AGENT : attendre la réponse, sauvegarder dans `IP_LINUX`.**

---

### Étape 2 — Récupérer le nom d'utilisateur SSH

**→ AGENT : poser la question suivante à l'utilisateur**

> **Quel est le nom d'utilisateur SSH sur le Linux ?**
>
> Pour le vérifier, lance **sur le PC Linux** :
>
> ```
> [PC LINUX - terminal]
> whoami
> ```
>
> C'est généralement `root`, ou ton nom d'utilisateur personnel (ex : `raph`, `ubuntu`, `debian`…).
>
> ℹ️ Si tu utilises `root`, assure-toi que `PermitRootLogin yes` est dans `/etc/ssh/sshd_config`.

**→ AGENT : attendre la réponse, sauvegarder dans `SSH_USER`.**

---

### Étape 3 — Identifier la clé SSH privée

**→ AGENT : poser la question suivante à l'utilisateur**

> **Quelle est ta clé SSH privée ?**
>
> Par défaut sur Windows, les clés SSH se trouvent dans `C:\Users\TON_NOM\.ssh\`.
>
> Pour lister tes clés, lance **sur Windows** dans PowerShell :
>
> ```
> [WINDOWS - PowerShell]
> Get-ChildItem "$env:USERPROFILE\.ssh\" | Where-Object { $_.Name -notlike "*.pub" -and $_.Name -notlike "known_hosts*" -and $_.Name -notlike "config*" }
> ```
>
> Si tu n'as pas encore de clé SSH, dis-le moi — je peux t'en générer une et t'expliquer comment l'installer sur le Linux.
>
> Donne-moi le **chemin complet** (ex : `C:\Users\raph6\.ssh\ma_cle_ed25519`).

**→ AGENT : attendre la réponse, sauvegarder dans `SSH_KEY`.**

---

### Étape 4 — Test de connectivité réseau

**→ AGENT : exécuter la commande suivante sur Windows**

```
[WINDOWS - PowerShell] → exécuté par l'agent
ping <IP_LINUX> -n 1
```

**→ AGENT : analyser le résultat :**
- Réponse reçue → passer à l'étape 5
- Aucune réponse / timeout → informer l'utilisateur que le PC Linux semble éteint ou hors réseau, et arrêter le processus

---

### Étape 5 — Test de connexion SSH

Avant le test, demander à l'utilisateur de comparer l'empreinte de la clé d'hôte affichée sur le PC Linux avec celle renvoyée sur Windows :

```
[WINDOWS - PowerShell] → exécuté par l'agent
ssh-keyscan -t ed25519 <IP_LINUX> | ssh-keygen -lf -
```

Ne poursuivre que si l'utilisateur confirme que les empreintes correspondent. À ce moment, ajouter la clé vérifiée à `known_hosts` :

```
[WINDOWS - PowerShell] → exécuté par l'agent après confirmation
ssh-keyscan -t ed25519 <IP_LINUX> | Add-Content "$env:USERPROFILE\.ssh\known_hosts"
```

**→ AGENT : exécuter la commande suivante sur Windows**

```
[WINDOWS - PowerShell] → exécuté par l'agent
ssh -i "<SSH_KEY>" -o StrictHostKeyChecking=yes -o ConnectTimeout=5 <SSH_USER>@<IP_LINUX> "echo OK"
```

**→ AGENT : analyser le résultat :**

| Erreur | Cause probable | Action à proposer à l'utilisateur |
|--------|---------------|-----------------------------------|
| `Permission denied` | La clé publique n'est pas dans `~/.ssh/authorized_keys` sur le Linux | Proposer de l'installer via mot de passe (`ssh-copy-id` ou équivalent) |
| `Connection refused` | SSH n'est pas démarré sur le Linux | Demander à l'utilisateur de lancer sur le Linux : `sudo systemctl start ssh` |
| `No route to host` / timeout | Problème réseau | Vérifier que les deux PC sont sur le même réseau |
| `echo OK` reçu | Connexion réussie | Passer à l'étape 6 |

---

### Étape 6 — Créer le fichier `.env`

**→ AGENT : créer le fichier `.env` à la racine du projet avec les vraies valeurs**

```
[WINDOWS - créé par l'agent dans le projet]
IP_LINUX=<IP_LINUX>
SSH_USER=<SSH_USER>
SSH_KEY=<SSH_KEY>
```

Ce fichier est dans `.gitignore` — il ne sera jamais versionné. `CLAUDE.md` et `AGENTS.md` restent génériques avec des placeholders ; les agents lisent `.env` pour obtenir les vraies valeurs à chaque session.

---

### Étape 7 — Préserver la confidentialité

**→ AGENT : ne jamais enregistrer les valeurs de `IP_LINUX`, `SSH_USER` ou `SSH_KEY` dans une mémoire persistante, un log, un message ou un commit.** Le fichier `.env` local et ignoré par Git est la seule source de configuration durable.

---

### Étape 8 — Confirmation

**→ AGENT : afficher le récapitulatif suivant**

```
✅ Connexion SSH configurée avec succès !

  PC Linux    : configuré localement
  Utilisateur : configuré localement
  Clé SSH     : configurée localement

  .env         ✅ créé (non versionné, valeurs réelles)
  CLAUDE.md / AGENTS.md restent génériques (repo public safe)

Les agents peuvent maintenant exécuter des commandes Linux directement via SSH.
```

**→ AGENT : proposer un premier test :** `"Veux-tu que je récupère l'uptime et l'utilisation RAM de ton Linux ?"`
