# 📂 NFS — Network File System Lab

> **Course:** Système d'Exploitation & Sécurité  
> **Session:** Hiver 2026  
> **Environment:** Ubuntu Server on VirtualBox

---

## 🎯 Objectives

- Install and configure an NFS server on Linux
- Create shared directories with different permission levels (read-only vs read-write)
- Mount shares on a Linux client
- Map a shared folder as a Windows network drive (Z:)
- Configure automatic mounting via `/etc/fstab`
- **Bonus:** Mount a USB drive inside the VM and access it from Windows

---

## 📋 Table of Contents

1. [Install Packages](#1-install-packages)
2. [Create Directories](#2-create-directories)
3. [Set Permissions](#3-set-permissions)
4. [Configure NFS Exports](#4-configure-nfs-exports)
5. [Mount the Shares](#5-mount-the-shares)
6. [Verify with df](#6-verify-with-df)
7. [Create Test Files](#7-create-test-files)
8. [Access Tests](#8-access-tests)
9. [Map to Windows Z:](#9-map-to-windows-z)
10. [Unmount](#10-unmount)
11. [Auto-mount with fstab](#11-auto-mount-with-fstab)
12. [Bonus — USB Drive G:](#12-bonus--usb-drive-g)

---

## 1. Install Packages

Update the system and install the NFS kernel server package.

```bash
sudo apt update
sudo apt upgrade
sudo apt install nfs-kernel-server
sudo systemctl enable nfs-kernel-server
sudo systemctl status nfs-kernel-server   # verify it's active
```

---

## 2. Create Directories

Create the two shared directories: one public (read-only) and one personal (read-write).

```bash
mkdir -p /nfs/etudiant/public
mkdir -p /nfs/etudiant/perso
```

---

## 3. Set Permissions

Create the `etudiant` user and assign ownership and permissions to the directories.

```bash
sudo useradd etudiant
sudo chown -R etudiant:etudiant /nfs/etudiant
sudo chmod 755 /nfs/etudiant/public    # read-only for others
sudo chmod 775 /nfs/etudiant/perso     # read-write for group
```

> **Why different permissions?**  
> `755` allows others to read and execute but not write — perfect for a public share.  
> `775` allows the owner and group to write — needed for the personal share.

---

## 4. Configure NFS Exports

Edit `/etc/exports` to define which directories are shared and with what options.

```bash
sudo nano /etc/exports
```

Add the following lines (see [`exports.conf`](./exports.conf)):

```
/nfs/etudiant/public *(ro,sync,no_subtree_check)
/nfs/etudiant/perso  *(rw,sync,no_subtree_check)
```

| Option | Meaning |
|--------|---------|
| `ro` | Read-only |
| `rw` | Read-write |
| `sync` | Write changes to disk before responding to client |
| `no_subtree_check` | Improves reliability; disables subtree permission checks |
| `*` | Allow any client IP (use specific IPs in production!) |

Apply the changes and restart:

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

---

## 5. Mount the Shares

Create local mount points and mount the NFS shares.

```bash
mkdir ~/nfs-public
mkdir ~/nfs-perso

sudo mount -t nfs localhost:/nfs/etudiant/public ~/nfs-public
sudo mount -t nfs localhost:/nfs/etudiant/perso ~/nfs-perso
```

---

## 6. Verify with df

Use `df` to confirm the shares are mounted correctly.

```bash
df -h
```

This displays:
- All mounted partitions
- Total disk size
- Used space
- Available space
- Mount points

You should see `localhost:/nfs/etudiant/public` and `localhost:/nfs/etudiant/perso` listed.

---

## 7. Create Test Files

Create test files inside the shared directories.

```bash
echo "repertoire public de etudiant" > /nfs/etudiant/public/info.txt
echo "prive" > /nfs/etudiant/perso/donnees.txt
```

---

## 8. Access Tests

| Test | Expected Result | Reason |
|------|----------------|--------|
| List `~/nfs-public` | ✅ Works | Read access allowed |
| Read `info.txt` | ✅ Works | Read access allowed |
| Create file in `nfs-public` | ❌ Fails | Share is read-only (`ro`) |
| Create file in `nfs-perso` | ✅ Works | Share is read-write (`rw`) |
| Delete `donnees.txt` from `nfs-perso` | ✅ Works | Write access allowed |

---

## 9. Map to Windows Z:

On Windows, open File Explorer → Right-click "This PC" → "Map network drive" → choose letter **Z:**.

Or from Command Prompt (requires NFS Client enabled on Windows):

```cmd
mount -o anon \\192.168.2.163\nfs\etudiant\ Z:
```

> ⚠️ **Note:** You must first enable the NFS Client feature on Windows.  
> Some Windows versions do not support NFS — use Samba as an alternative in those cases.

After mapping, the Linux shared folder appears as drive **Z:** in Windows Explorer.  
Files dropped from Windows are immediately visible in Linux.

---

## 10. Unmount

```bash
sudo umount ~/nfs-public
sudo umount ~/nfs-perso
```

After unmounting, the directories are no longer accessible.

---

## 11. Auto-mount with fstab

To automatically mount the NFS shares at boot, edit `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add the following lines (see [`fstab-entry.txt`](./fstab-entry.txt)):

```
localhost:/nfs/etudiant/public  /home/etudiant/nfs-public  nfs  defaults  0  0
localhost:/nfs/etudiant/perso   /home/etudiant/nfs-perso   nfs  defaults  0  0
```

Test without rebooting:

```bash
mount /home/etudiant/nfs-public
```

---

## 12. Bonus — USB Drive G:

> ⚠️ This part was done on VirtualBox, not on Proxmox.

**Steps:**

1. Plug a USB drive into the host machine
2. In VirtualBox menu: **Devices → USB → [select your USB device]**
3. The USB disappears from Windows host and is transferred to the Linux VM

Verify the device is detected:

```bash
lsblk
```

Create a mount point and mount the USB:

```bash
sudo mkdir /media/usb
sudo mount /dev/sdb1 /media/usb
```

From Windows, the drive can be recognized as **G:** through the network share mapping.

> 💡 The tricky part was making sure VirtualBox correctly detected the USB device — some USB 3.0 drives require the VirtualBox Extension Pack.

---

## 📚 References

- [NetApp — How to Set Up an NFS Server and Client](https://www.netapp.com/learn/azure-anf-blg-linux-nfs-server-how-to-set-up-server-and-client/)
- [Akash Rajpurohit — Setup Shareable Drive with NFS](https://akashrajpurohit.com/blog/setup-shareable-drive-with-nfs-in-linux/)
- [Ubuntu Official Docs — Install NFS](https://ubuntu.com/server/docs/how-to/networking/install-nfs/)
