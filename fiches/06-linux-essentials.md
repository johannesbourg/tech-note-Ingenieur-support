# Fiche 6 — Linux Essentials pour le Support Production

# Fiche 6 — Linux Essentials pour le Support Production

<aside>
🎯

**Pourquoi cette fiche ?**

Les offres 2 (Support Applicatif Finance) et 3 (Ingénieur Production) exigent Linux comme prerequis. Cette fiche synthétise ce qu'un support L2 doit maîtriser sur Linux, avec un plan d'entraînement pratique via **Google Cloud Shell** (100 % gratuit, aucune CB, dans votre navigateur).

</aside>

**Ce que vous saurez faire après cette fiche :**

- Naviguer dans un système Linux (fichiers, dossiers, permissions)
- Diagnostiquer un service qui ne démarre pas
- Lire des logs et trouver l'erreur
- Gérer utilisateurs et processus
- Comprendre le réseau Linux de base
- Faire du SSH proprement

---

## 🌐 1. Pourquoi Linux domine en production

- ~85 % des serveurs web dans le monde tournent sous Linux (Nginx, Apache)
- Quasi 100 % des serveurs applicatifs Java (JBoss, WildFly, Tomcat) en finance de marché
- Base de tous les conteneurs (Docker, Kubernetes)
- Gratuit, stable, personnalisable, très bien documenté

### Distributions courantes

| Famille | Distributions | Contexte typique |
| --- | --- | --- |
| **RHEL** (Red Hat) | RHEL, CentOS Stream, Rocky Linux, AlmaLinux | Banques, assurances, grands comptes |
| **Debian** | Debian, Ubuntu Server | Startups, cloud, web |
| **SUSE** | SLES, openSUSE | SAP, Europe, industrie |

<aside>
💡

**Bon à savoir :** Les commandes de base sont identiques sur toutes les distributions. Ce qui change principalement, c'est le gestionnaire de paquets (`apt` sur Debian/Ubuntu, `dnf`/`yum` sur RHEL).

</aside>

---

## 🗂️ 2. Le système de fichiers Linux (FHS)

En Linux, **tout est fichier**. Structure standard à connaître :

| Dossier | Contenu | Exemple d'usage support |
| --- | --- | --- |
| `/` | Racine du système | Point de départ de tout |
| `/home` | Dossiers personnels des utilisateurs | `/home/johannes/` |
| `/root` | Dossier de l'utilisateur root | ⚠️ accès admin |
| `/etc` | Fichiers de configuration | `/etc/ssh/sshd_config` |
| `/var/log` | **Logs système** | Point clé pour diagnostiquer |
| `/var/www` | Sites web | Apache/Nginx par défaut |
| `/opt` | Logiciels tiers installés manuellement | Souvent apps métier |
| `/tmp` | Fichiers temporaires | Vidé au redémarrage |
| `/usr/bin` | Exécutables installés | Commandes système |
| `/proc` | Vue temps réel du noyau | `/proc/meminfo`, `/proc/cpuinfo` |
| `/mnt`, `/media` | Points de montage | Disques externes, NFS |

---

## 📂 3. Commandes essentielles par catégorie

### 3.1 Navigation & inspection

```bash
pwd                    # Print Working Directory : où suis-je ?
ls                     # Lister le contenu du dossier
ls -la                 # Lister avec détails + fichiers cachés
ls -lh /var/log        # Tailles lisibles (K, M, G)
cd /etc                # Se déplacer
cd ~                   # Retour au home
cd -                   # Retour au dossier précédent
tree -L 2              # Arborescence (2 niveaux)
```

### 3.2 Manipulation de fichiers

```bash
touch fichier.txt      # Créer un fichier vide
mkdir -p a/b/c         # Créer dossiers imbriqués
cp source dest        # Copier
cp -r dir1 dir2        # Copier récursivement
mv ancien nouveau     # Déplacer / renommer
rm fichier             # Supprimer
rm -rf dossier         # Supprimer récursivement (⚠️ attention)
ln -s /path/reel lien  # Lien symbolique
```

<aside>
⚠️

**Attention :** `rm -rf /` ou `rm -rf /*` en tant que root **détruit tout le système**. Toujours vérifier deux fois avant de lancer `rm -rf`. Il n'existe **PAS** de corbeille en ligne de commande.

</aside>

### 3.3 Lire un fichier / chercher dans un fichier

```bash
cat fichier.log                    # Afficher tout le contenu
less fichier.log                   # Afficher page par page (q pour quitter)
head -n 20 fichier.log             # 20 premières lignes
tail -n 50 fichier.log             # 50 dernières lignes
tail -f /var/log/syslog            # Suivre en temps réel (Ctrl+C pour quitter)
grep "error" fichier.log           # Chercher "error"
grep -i "error" fichier.log        # Insensible à la casse
grep -rn "password" /etc/          # Récursif + numéro de ligne
zcat archive.log.gz | grep ERROR   # Lire un log compressé
```

### 3.4 Permissions & propriété

`ls -l` affiche par exemple : `-rwxr-xr-- 1 johannes users`

- 1ᵉʳ caractère : type (`-` fichier, `d` dossier, `l` lien)
- 3 blocs de 3 : permissions **Owner / Group / Other**
- `r` = read (4), `w` = write (2), `x` = execute (1)

```bash
chmod 755 script.sh        # rwxr-xr-x (owner tout, autres lecture+exéc)
chmod +x script.sh         # Rendre exécutable
chmod -R 640 /etc/monapp/  # Récursif
chown johannes:users file  # Changer propriétaire et groupe
```

<aside>
🔐

**Modes courants à connaître :**

- `644` = fichiers de conf standard (rw-r--r--)
- `755` = scripts et binaires (rwxr-xr-x)
- `600` = fichiers sensibles (rw-------), typiquement clés SSH privées
- `700` = dossiers personnels (rwx------)
</aside>

### 3.5 Processus & performance

```bash
ps aux                 # Tous les processus
ps aux | grep java     # Processus Java uniquement
top                    # Vue temps réel (q pour quitter)
htop                   # Version améliorée (à installer)
kill 1234              # Terminer proprement (SIGTERM)
kill -9 1234           # Forcer (SIGKILL, dernier recours)
pkill -f monapp        # Tuer par nom
nice -n 10 commande    # Lancer avec priorité abaissée
free -h                # Mémoire disponible
uptime                 # Charge CPU (load average)
```

### 3.6 Espace disque

```bash
df -h                  # Espace libre par partition
du -sh /var/log        # Taille totale d'un dossier
du -h --max-depth=1 /var | sort -h   # Top-consommateurs de /var
lsblk                  # Liste des disques et partitions
```

<aside>
🚨

**Cas réel support production :** "Le serveur ne répond plus." Réflexe n° 1 : `df -h`. 9 fois sur 10, une partition est à 100 % (souvent `/var` à cause de logs qui ont explosé).

</aside>

### 3.7 Réseau

```bash
ip a                       # Interfaces réseau (remplace ifconfig)
ip r                       # Table de routage (remplace route)
ss -tulpn                  # Ports en écoute (remplace netstat)
ping -c 4 8.8.8.8          # 4 pings vers Google DNS
traceroute google.com     # Chemin réseau
curl -I https://site.com   # Headers HTTP
curl -v https://api.com    # Verbeux (debug)
dig google.com             # Résolution DNS
dig @8.8.8.8 google.com    # Forcer DNS Google
nc -vz host 443           # Tester si un port est ouvert
```

### 3.8 Services & logs (systemd)

La majorité des Linux modernes utilisent **systemd** pour gérer les services.

```bash
systemctl status nginx             # État du service
systemctl start nginx              # Démarrer
systemctl stop nginx               # Arrêter
systemctl restart nginx            # Redémarrer
systemctl reload nginx             # Recharger config (sans downtime)
systemctl enable nginx             # Démarrer au boot
systemctl disable nginx            # Ne plus démarrer au boot
systemctl list-units --failed      # Services en échec

journalctl -u nginx                # Logs du service
journalctl -u nginx -f             # Suivre en temps réel
journalctl -u nginx --since "1h ago"
journalctl -p err -b               # Erreurs depuis le dernier boot
```

### 3.9 Utilisateurs & groupes

```bash
whoami                     # Qui suis-je ?
id                         # Mon UID, GID, groupes
id johannes                # Idem pour un autre user
sudo useradd -m -s /bin/bash alice   # Créer user avec home
sudo passwd alice          # Définir mot de passe
sudo usermod -aG sudo alice          # Ajouter au groupe sudo
groups alice               # Groupes d'un user
sudo userdel -r alice      # Supprimer + son home
```

### 3.10 Archives & compression

```bash
tar -czvf backup.tar.gz /etc/       # Créer archive gzippée
tar -xzvf backup.tar.gz             # Extraire
tar -tzvf backup.tar.gz             # Lister sans extraire
zip -r site.zip /var/www/            # Créer ZIP
unzip site.zip                       # Extraire ZIP
```

---

## 🔗 4. Redirections & pipes — la vraie puissance

```bash
commande > fichier         # Rediriger stdout dans un fichier (écrase)
commande >> fichier        # Ajouter à la fin
commande 2> erreurs.log    # Rediriger stderr
commande &> tout.log       # stdout + stderr
commande < entree.txt      # Lire depuis un fichier

# Pipes (|) : sortie d'une commande = entrée de la suivante
ps aux | grep java | grep -v grep
cat /var/log/syslog | grep ERROR | tail -20
du -sh /var/* | sort -h | tail -5    # Top 5 consommateurs de /var
```

### Exemple concret : trouver un utilisateur bloqué dans les logs

```bash
grep "authentication failure" /var/log/auth.log | \
    grep "johannes" | \
    tail -10
```

---

## 🔑 5. SSH — accès distant sécurisé

```bash
# Se connecter
ssh utilisateur@serveur
ssh -p 2222 utilisateur@serveur    # Port personnalisé
ssh -i ~/.ssh/ma_cle utilisateur@serveur

# Générer une paire de clés (moderne)
ssh-keygen -t ed25519 -C "johannes@abidjan"
# Génère ~/.ssh/id_ed25519 (privée) et id_ed25519.pub (publique)

# Copier sa clé publique vers un serveur
ssh-copy-id utilisateur@serveur

# Transfert de fichiers
scp fichier.txt utilisateur@serveur:/tmp/
scp -r dossier/ utilisateur@serveur:/tmp/
rsync -avz local/ utilisateur@serveur:/dest/    # Alternative moderne
```

<aside>
🔐

**Sécurité SSH essentielle :**

- ❌ JAMAIS partager sa clé privée (`id_ed25519` sans `.pub`)
- ✅ Protéger `~/.ssh/` en `700` et les clés en `600`
- ✅ Désactiver l'authentification par mot de passe en production (config `sshd_config`)
- ✅ Utiliser `ed25519` plutôt que `rsa` (plus rapide, plus sûr)
</aside>

---

## ⏰ 6. Tâches planifiées (cron)

```bash
crontab -l              # Lister mes tâches
crontab -e              # Éditer
```

Syntaxe : `* * * * * commande`

```
┌───────────── minute (0-59)
│ ┌─────────── heure (0-23)
│ │ ┌───────── jour du mois (1-31)
│ │ │ ┌─────── mois (1-12)
│ │ │ │ ┌───── jour de la semaine (0-6, 0=dimanche)
│ │ │ │ │
* * * * * commande
```

**Exemples :**

```
0 2 * * *       /opt/backup.sh              # Chaque jour à 02h00
*/15 * * * *    /opt/check_service.sh        # Toutes les 15 min
0 9 * * 1-5     /opt/report.sh               # 9h du lundi au vendredi
0 0 1 * *       /opt/monthly.sh              # Le 1er de chaque mois à minuit
```

---

## 🔍 7. Scénarios de troubleshooting support production

### Scénario 1 : "Le site web ne répond plus"

```bash
# 1. Vérifier le service
systemctl status nginx

# 2. Voir les logs récents
journalctl -u nginx -n 100 --no-pager

# 3. Vérifier le port
ss -tulpn | grep :80

# 4. Tester en local
curl -I http://localhost

# 5. Vérifier la conf
nginx -t

# 6. Espace disque
df -h
```

### Scénario 2 : "L'application Java plante"

```bash
# 1. Le processus tourne-t-il ?
ps aux | grep java

# 2. Consommation mémoire
free -h

# 3. Logs applicatifs
tail -f /opt/monapp/logs/application.log

# 4. Erreurs OOM (Out Of Memory) ?
dmesg | grep -i "killed process"
grep -i "OutOfMemoryError" /opt/monapp/logs/*.log
```

### Scénario 3 : "La base de données est lente"

```bash
# 1. Charge CPU
uptime
top

# 2. I/O disque
iostat -x 2 5              # (paquet sysstat)

# 3. Processus consommateurs
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head

# 4. Espace disque data
df -h /var/lib/postgresql
```

### Scénario 4 : "Utilisateur ne peut plus se connecter en SSH"

```bash
# 1. Le service SSH tourne-t-il ?
systemctl status ssh

# 2. Logs d'authentification
tail -50 /var/log/auth.log            # Debian/Ubuntu
tail -50 /var/log/secure               # RHEL

# 3. Le user existe-t-il et n'est pas verrouillé ?
getent passwd johannes
passwd -S johannes

# 4. Permissions du .ssh du user ?
ls -la /home/johannes/.ssh/
```

---

## 🌐 8. Plan d'entraînement pratique — Google Cloud Shell

**Google Cloud Shell** = une VM Linux **Ubuntu 24.04 LTS** (dérivée de Debian), gratuite, dans votre navigateur, avec 5 Go persistants. Il vous faut juste un **compte Gmail**.

### Étape 1 — Ouvrir Cloud Shell

1. Aller sur 👉 `https://shell.cloud.google.com`
2. Se connecter avec votre compte Gmail
3. Accepter les CGU au premier lancement (30 sec)
4. Un terminal noir s'ouvre : vous êtes dans une VM Linux ✅

### Étape 2 — Exercices de découverte (30 min)

<aside>
💡

**Astuce :** l'écran se remplit vite. On divise l'exercice en 3 petits blocs, avec `clear` entre chaque pour repartir sur un écran propre. **3 captures** au total pour cette étape (au lieu d'une seule).

</aside>

**Bloc 2.1 — Identité et environnement**

```bash
clear
whoami
pwd
ls -la
```

📸 **Capture :** `01a-identite.png`

**Bloc 2.2 — Distribution et noyau**

```bash
clear
cat /etc/os-release
uname -a
```

📸 **Capture :** `01b-systeme.png`

**Bloc 2.3 — Ressources système**

```bash
clear
free -h
df -h
nproc
```

📸 **Capture :** `01c-ressources.png`

### Étape 3 — Manipulation de fichiers (20 min)

<aside>
⚙️

**Pré-requis :** l'outil `tree` n'est pas installé par défaut sur Cloud Shell. Lancez d'abord `sudo apt install tree -y` (sans mot de passe requis), puis tapez `clear` avant d'attaquer les blocs ci-dessous.

</aside>

**Bloc 3.1 — Créer et manipuler les fichiers**

```bash
clear
mkdir -p ~/portfolio/labs/linux
cd ~/portfolio/labs/linux
touch note1.txt note2.txt
echo "Ceci est mon premier fichier" > note1.txt
cat note1.txt
cp note1.txt note1-backup.txt
mv note2.txt notes-avancees.txt
```

📸 **Capture :** `02a-creation-fichiers.png`

**Bloc 3.2 — Vérifier le résultat**

```bash
clear
cd ~/portfolio/labs/linux
ls -la
tree
```

📸 **Capture :** `02b-arborescence.png`

### Étape 4 — Permissions (15 min)

```bash
echo '#!/bin/bash' > hello.sh
echo 'echo "Hello depuis ma VM Linux !"' >> hello.sh
cat hello.sh
ls -l hello.sh
chmod +x hello.sh
ls -l hello.sh                # Voir le changement (-rwxr-xr-x)
./hello.sh                     # Exécuter
```

📸 **Capture :** `03-permissions-execution.png`

### Étape 5 — Recherche dans les logs (20 min)

<aside>
🔍

**C'est LA compétence clé du support L2 :** savoir chercher une erreur dans un fichier de log. On découpe en 2 blocs.

</aside>

**Bloc 5.1 — Générer un faux fichier de logs et le lire**

```bash
clear
cat > ~/faux.log <<'EOF'
2026-08-01 10:00:00 INFO  Application demarree
2026-08-01 10:00:15 DEBUG Chargement config
2026-08-01 10:01:23 WARN  Timeout DB 500ms
2026-08-01 10:02:45 ERROR Connection refused sur port 5432
2026-08-01 10:03:01 ERROR Retry 1/3 echoue
2026-08-01 10:03:15 ERROR Retry 2/3 echoue
2026-08-01 10:03:29 FATAL Impossible de joindre la base
EOF
cat ~/faux.log
```

📸 **Capture :** `04a-log-genere.png`

**Bloc 5.2 — Analyser le log (grep, tail, awk)**

```bash
clear
grep ERROR ~/faux.log
echo "---"
grep -c ERROR ~/faux.log
echo "---"
tail -3 ~/faux.log
echo "---"
awk '/ERROR|FATAL/ {print $2, $4}' ~/faux.log
```

📸 **Capture :** `04b-analyse-log.png`

### Étape 6 — Réseau (10 min)

<aside>
🌐

**Diagnostic réseau L2 :** ces 4 commandes sont le réflexe absolu quand "le serveur ne répond plus". On les découpe en 3 blocs.

</aside>

**Bloc 6.1 — Interfaces réseau et connectivité**

```bash
clear
ip a
echo "---"
ping -c 3 8.8.8.8
```

📸 **Capture :** `05a-interfaces-ping.png`

**Bloc 6.2 — Test HTTP**

```bash
clear
curl -I https://www.google.com
```

📸 **Capture :** `05b-curl-http.png`

**Bloc 6.3 — Résolution DNS**

```bash
clear
dig google.com
```

📸 **Capture :** `05c-dig-dns.png`

### Étape 7 — Sauvegarder pour la suite

✅ Cloud Shell **persiste automatiquement** votre `$HOME` (5 Go). Vos fichiers seront toujours là la prochaine fois. Aucune action à faire.

---


## 📚 Sources officielles

- Documentation Google Cloud Shell : [https://cloud.google.com/shell/docs](https://cloud.google.com/shell/docs)
- Filesystem Hierarchy Standard (FHS) : [https://refspecs.linuxfoundation.org/fhs.shtml](https://refspecs.linuxfoundation.org/fhs.shtml)
- Manuel Linux (`man`) : accessible directement via `man commande` (ex : `man ls`)
- The Linux Documentation Project : [https://tldp.org](https://tldp.org)
- systemd documentation : [https://systemd.io/](https://systemd.io/)
- Cheatsheet SSH : [https://www.ssh.com/academy/ssh](https://www.ssh.com/academy/ssh)

---

<aside>
➡️

**Prochaine étape pour VOUS :**

1. Ouvrir [https://shell.cloud.google.com](https://shell.cloud.google.com) avec votre Gmail
2. Suivre les étapes 1 à 6 de la section 8
3. Prendre les 5 captures d'écran indiquées
4. Revenir me voir avec les captures : je créerai le **Lab 2 — Linux en pratique** qui intégrera VOS captures. Ça deviendra votre premier lab **100 % authentique**.
</aside>
