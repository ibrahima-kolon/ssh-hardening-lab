🛡️ Durcissement d'un serveur SSH sous Linux

Projet pratique réalisé dans le cadre de ma formation en Master 1 Cybersécurité (Efrei Paris). Objectif : auditer la configuration par défaut d'un serveur SSH, identifier ses faiblesses, puis le durcir selon les bonnes pratiques — en documentant chaque étape.

Démarche suivie : Auditer → Corriger → Vérifier.

🎯 Objectif

Prendre un serveur SSH fraîchement installé (configuration par défaut) et le transformer en un service sécurisé et conforme aux bonnes pratiques, tout en comprenant le pourquoi de chaque réglage.

🧰 Environnement
Élément	Détail
Système	Ubuntu (machine virtuelle, Hyper-V)
Service	OpenSSH Server
Réseau	Lab isolé, à but pédagogique
🔍 Phase 1 — Audit (identification des faiblesses)

Analyse de la configuration par défaut de SSH (/etc/ssh/sshd_config et overrides).

#	Faiblesse constatée	Gravité
1	Connexion root autorisée à distance (prohibit-password)	🟠 Moyen
2	Authentification par mot de passe activée (brute-force possible)	🔴 Élevé
3	Pare-feu inactif (aucune restriction réseau)	🔴 Élevé
4	Aucune protection anti-brute-force	🟠 Moyen
🔧 Phase 2 — Durcissement (corrections)
1. Mise en place de l'authentification par clé (Ed25519)

Génération d'une paire de clés moderne, puis installation de la clé publique sur le serveur.

bash
ssh-keygen -t ed25519
ssh-copy-id localhost

La clé privée reste secrète ; le serveur vérifie l'identité sans jamais qu'elle transite. Bien plus robuste qu'un mot de passe face au brute-force.

2. Désactivation de root et des mots de passe

Création d'un fichier d'override dédié /etc/ssh/sshd_config.d/99-hardening.conf :

PermitRootLogin no
PasswordAuthentication no
3. Activation du pare-feu (UFW) — seul SSH exposé
bash
sudo ufw allow ssh
sudo ufw enable

Règle d'or : autoriser SSH avant d'activer le pare-feu, pour ne pas se verrouiller dehors.

4. Protection anti-brute-force (fail2ban)
bash
sudo apt install fail2ban

fail2ban bannit automatiquement les adresses IP qui multiplient les tentatives échouées : une couche de défense en profondeur.

✅ Phase 3 — Vérification

Un réglage n'est validé que s'il est prouvé actif.

bash
# Tester la config AVANT de l'appliquer (évite de casser SSH)
sudo sshd -t

# Vérifier la configuration réellement appliquée
sudo sshd -T | grep -iE "permitrootlogin|passwordauthentication"

# État du pare-feu
sudo ufw status verbose

Le test sshd -t a d'ailleurs rattrapé une faute de frappe avant l'application — sur un serveur distant, cela aurait pu provoquer un verrouillage total.

📊 Avant / Après
Élément	❌ Avant	✅ Après
Connexion root	Autorisée (par clé)	Interdite
Mots de passe SSH	Activés	Désactivés (clé obligatoire)
Clé SSH	Aucune	Ed25519
Pare-feu (UFW)	Inactif	Actif (seul SSH exposé)
Anti-brute-force	Aucun	fail2ban actif
🎓 Ce que j'ai appris
Installer et piloter un service Linux (systemctl : start / enable / status)
Auditer une configuration de sécurité et en lire les réglages effectifs
Le principe de l'authentification par clé (couple privé / public)
L'importance de tester avant d'appliquer (sshd -t)
Les concepts de pare-feu et de défense en profondeur
🚀 Pour aller plus loin
Changer le port SSH par défaut (réduction du bruit de scan)
Restreindre les utilisateurs autorisés (AllowUsers)
Centraliser et analyser les journaux d'authentification
