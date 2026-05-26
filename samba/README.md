# 🔗 Samba — Cross-Platform File Sharing Lab

> **Course:** Système d'Exploitation & Sécurité — 420-244-CH
> **Session:** Hiver 2026
> **Environment:** Ubuntu Server on VirtualBox

---

## What is Samba?

Samba is used to share files, accounts, permissions, and devices (like printers) between clients on a network, including between Linux and Windows machines. It uses the SMB/CIFS protocol which runs over TCP/IP, making it flexible enough to work across complex networks.

---

## Table of Contents

1. [Installation](#1-installation)
2. [Public Share](#2-public-share)
3. [Verify the Share](#3-verify-the-share)
4. [Secure Share with User Auth](#4-secure-share-with-user-auth)
5. [Troubleshooting](#5-troubleshooting)
6. [Useful Commands](#6-useful-commands)
7. [Key Concepts](#7-key-concepts)

---

## 1. Installation

```bash
sudo apt update
sudo apt install samba samba-client -y
systemctl status smbd
smbd --version
```

**smbd vs nmbd:**
- `smbd`: handles file and printer sharing between clients
- `nmbd`: handles NetBIOS name resolution over IP (lets Windows find the Linux machine by name)

---

## 2. Public Share

Create a public user and set up the shared directory.

```bash
sudo adduser public
sudo chmod ugo+rw /home/public
sudo groupadd public2
sudo usermod -aG public2 public
sudo chown root:public2 /home/public
```

Edit the Samba config:

```bash
sudo nano /etc/samba/smb.conf
```

Add this block at the end (see [`smb.conf`](./smb.conf)):

```ini
[Public]
path = /srv/samba/public
browseable = yes
writable = yes
guest ok = yes
read only = no
```

Restart Samba:

```bash
sudo systemctl restart smbd
```

**Security note:** `guest ok = yes` means anyone can access the share without a password — fine for a lab, risky in production.

**read only vs writable:**
- `read only = yes` → clients can only read files
- `writable = yes` → clients can read and write files

---

## 3. Verify the Share

From a Linux client on the same network:

```bash
smbclient -L localhost -N           # list available shares
smbclient //localhost/Public -N     # connect to the public share
```

The `-N` flag skips the password prompt (used for guest/anonymous access).

**SMB vs CIFS:**
CIFS is an older Microsoft implementation of SMB. Modern systems use SMB2 or SMB3. SMB1 is considered dangerous and should be disabled — it has known vulnerabilities (EternalBlue, WannaCry).

---

## 4. Secure Share with User Auth

Create a dedicated user and secure directory:

```bash
sudo adduser secure
mkdir secure
sudo chown -R secure:secure /home/secure/secure
```

Add a Samba password for the user:

```bash
sudo smbpasswd -a secure
```

Edit `/etc/samba/smb.conf` and add:

```ini
[Secure]
path = /srv/samba/secure
valid users = secure
read only = no
browseable = yes
```

Restart and connect:

```bash
sudo systemctl restart smbd
smbclient //localhost/secure -U secure
```

**Why two separate users (Linux + Samba)?**
- The Linux user controls file system permissions
- The Samba user handles authentication to the share
- If Linux permissions are wrong → access is denied even with correct Samba credentials

---

## 5. Troubleshooting

| Problem | Where to look |
|---------|--------------|
| Share not visible | Check `/etc/samba/smb.conf` or firewall (`ufw`) |
| Access denied | Wrong Linux permissions or Samba user not authorized |
| Service won't start | `journalctl -u smbd` |
| SMB1 warning | Disable it — it's vulnerable |

```bash
sudo ufw allow samba      # allow Samba through firewall
testparm                  # validate smb.conf syntax
journalctl -u smbd        # check service logs
smbstatus                 # show active connections
ss -tulnp | grep smb      # verify ports are open
```

---

## 6. Useful Commands

```bash
testparm                        # check config for errors
smbstatus                       # show who is connected
journalctl -u smbd              # service logs
ss -tulnp | grep smb            # check open ports
sudo systemctl restart smbd     # restart Samba
sudo smbpasswd -a username      # add Samba user
smbclient -L localhost -N       # list shares
```

---

## 7. Key Concepts

**SMB vs NFS: which to choose?**
- NFS is faster and lighter — better for Linux-to-Linux
- Samba (SMB) is better when Windows clients are involved

**Is Samba good for cloud?**
Not really — cloud environments have better-suited alternatives (like object storage or cloud-native file services).

**Main security risks:**
- SMB1 enabled (vulnerable to exploits)
- `guest ok = yes` on sensitive shares
- Weak passwords
- Incorrect Linux file permissions
