# Linux Cheat Sheet

Commands I use regularly — adding to this as I learn more.

---

## Navigation

```bash
ls            # list files
ls -la        # list all files with details
pwd           # where am i
cd /path      # go to folder
mkdir -p dir  # create folder (+ parent folders if needed)
rm -r dir     # delete folder
cp -r src dst # copy
mv src dst    # move or rename
```

---

## Search

```bash
grep "word" file        # search in file
grep -r "word" folder   # search in all files inside folder
grep -i "word" file     # ignore uppercase/lowercase
find . -name "*.txt"    # find files by name
```

---

## Archives

```bash
tar -cvf archive.tar folder      # create archive
tar -xvf archive.tar             # extract archive
tar -czvf archive.tar.gz folder  # create compressed archive
tar -xzvf archive.tar.gz         # extract compressed archive
```

---

## Text & Files

```bash
cat file              # show file content
head -n 10 file       # first 10 lines
tail -n 10 file       # last 10 lines
wc -l file            # count lines
sort file             # sort lines

awk '{print $1}' file               # print first column
awk -F: '{print $1}' /etc/passwd    # print usernames
```

---

## Pipes & Redirects

```bash
cmd1 | cmd2        # send output of cmd1 to cmd2
cmd > file         # write output to file (overwrites)
cmd >> file        # append output to file
echo "text" > file # write text to file

# examples
grep "error" /var/log/syslog | tail -20
cat /etc/passwd | awk -F: '{print $1}' | sort
```

---

## Users & Permissions

```bash
sudo useradd username
sudo passwd username
sudo chown -R user:group folder
chmod 755 file   # rwxr-xr-x
chmod 775 file   # rwxrwxr-x
```

Permission numbers: 7=rwx, 6=rw-, 5=r-x, 4=r--

---

## Services

```bash
sudo systemctl start service
sudo systemctl stop service
sudo systemctl restart service
sudo systemctl enable service   # start at boot
sudo systemctl status service
```

---

## NFS

```bash
sudo exportfs -a                              # apply /etc/exports changes
showmount -e localhost                        # show shared folders
sudo mount -t nfs server:/path /mountpoint   # mount share
sudo umount /mountpoint                       # unmount
df -h                                         # check mounted disks
```

---

## Disks & Devices

```bash
df -h                        # disk usage
lsblk                        # list all devices
sudo mount /dev/sdb1 /media/usb
sudo umount /media/usb
```
