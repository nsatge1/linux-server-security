# 🛡️ Projet Cybersécurité : Sécurisation d’un serveur Linux + Tests d’attaque avec Kali

## 🎯 Objectif du projet

Créer un serveur Linux minimal, le configurer de manière sécurisée, puis simuler des attaques réseau avec Kali Linux pour tester sa résistance.
But pédagogique : apprendre le hardening, les outils de surveillance, et la posture défensive face à des menaces de base.

## 📦 Environnement requis

### Machines virtuelles
1. Serveur Linux (Ubuntu 22.04 ou Debian 12)
Utilisé comme serveur cible
2. Kali Linux
Utilisé comme machine d’attaque



## 🔧 Installation du serveur

### Étapes :
- Créer une VM Ubuntu/Debian dans UTM
- Configurer l’IP en mode bridge (ou réseau interne) pour que Kali et le serveur
- Installer les outils suivants sur le serveur :

```bash
sudo apt update && sudo apt upgrade
sudo apt install apache2 openssh-server ufw fail2ban lynis net-tools
```

- Apache → pour avoir un port 80 exposé
- OpenSSH → pour te connecter à distance (donc une cible à sécuriser)
- UFW → pare-feu simple
- Fail2Ban → bloque les tentatives de brute-force
- Lynis → scanner de sécurité
- Net-tools → outils réseau (ping, ifconfig…)



## 🔐 Sécurisation de base du serveur


### Pare-feu UFW :
```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw enable
```

### Fail2Ban :
1.  Active et démarre :
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

2. Crée un fichier de configuration personnalisé :
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

3. Édite la section [sshd] :
```bash
[sshd]
enabled = true
port = ssh
maxretry = 3
bantime = 3600
```

4. Redémarre le service :
```bash
sudo systemctl restart fail2ban
```

5. Vérifie l’état :
```bash
sudo fail2ban-client status sshd
```

### SSH sécurisée
1. Édition de /etc/ssh/sshd_config :
```bash
Port 2222
PermitRootLogin no
PasswordAuthentication no
AllowUsers ton_user
```
2. Redémarrage SSH :
```bash
sudo systemctl restart ssh
```

3. Génération et configuration de clé SSH depuis Kali

Sur Kali
```bash
ssh-keygen -t rsa -b 4096
ssh-copy-id -p 2222 ton_user@adresse_ip_serveur
ssh -p 2222 ton_user@adresse_ip_serveur
```


## 🔍 Analyse de sécurité avec Lynis
```bash
    sudo lynis audit system
```


## 💣 Attaques depuis Kali

1. Depuis Kali, fais un scan de reconnaissance :
```bash
nmap -p- -sV adresse_ip_serveur
```

2. Port SSH : essaye un bruteforce avec hydra sur le port 2222 :
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://<IP_SERVER> -s 2222
```

3. Lance un scan de vulnérabilités (reconnaissance passive) :
```bash
nikto -h http://<IP-serveur>
```


## 📁 Analyse des logs sur le serveur
- Apache : **sudo tail -f /var/log/apache2/access.log**
- SSH : **sudo tail -f /var/log/auth.log**
- Fail2Ban : **/var/log/fail2ban.log**


## 📌 Améliorations futures
- Implémenter un **reverse proxy avec HTTPS (Let’s Encrypt + Nginx)**
- Utiliser **Ansible ou Bash script** pour tout automatiser (DevSecOps)
- Mettre en place une surveillance avec **OSSEC, Wazuh ou ELK**

