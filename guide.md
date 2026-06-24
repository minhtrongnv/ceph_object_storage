### 1. Node verification
```
hostname
hostname -I
ip -br addr
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL
df -h
free -h
nproc
timedatectl
uname -a

sudo systemctl status ssh --no-pager
sudo ufw status
```


### 2. Add host for each Ceph node
```
sudo sed -i '/# BEGIN RUSTFS HOSTS/,/# END RUSTFS HOSTS/d' /etc/hosts

sudo tee -a /etc/hosts >/dev/null <<'EOF'
# BEGIN RUSTFS HOSTS
192.168.136.200 rustfs1
192.168.136.201 rustfs2
192.168.136.202 rustfs3
192.168.136.203 rustfs4
# END RUSTFS HOSTS
EOF
```

Verify
```
getent hosts rustfs1
getent hosts rustfs2
getent hosts rustfs3
getent hosts rustfs4
```

### 3. Install base packages
```
sudo apt update
sudo apt install -y curl wget unzip xfsprogs chrony openssh-server
sudo systemctl enable --now chrony
sudo systemctl enable --now ssh
```

Enable Services
```
sudo systemctl enable --now chrony
sudo systemctl enable --now ssh
```

Verify
```
systemctl is-active chrony
systemctl is-active ssh
timedatectl

Result
active
active
System clock synchronized: yes
NTP service: active
```

### 4. Format Disk as XFS
```
sudo mkfs.xfs -f /dev/sdb
sudo mkfs.xfs -f /dev/sdc
sudo mkfs.xfs -f /dev/sdd
sudo mkfs.xfs -f /dev/sde
```

Create mount directories
```
sudo mkdir -p /data/rustfs0
sudo mkdir -p /data/rustfs1
sudo mkdir -p /data/rustfs2
sudo mkdir -p /data/rustfs3
```

Get UUIDs
```
sudo blkid /dev/sdb /dev/sdc /dev/sdd /dev/sde

SDB_UUID=$(sudo blkid -s UUID -o value /dev/sdb)
SDC_UUID=$(sudo blkid -s UUID -o value /dev/sdc)
SDD_UUID=$(sudo blkid -s UUID -o value /dev/sdd)
SDE_UUID=$(sudo blkid -s UUID -o value /dev/sde)

sudo cp /etc/fstab /etc/fstab.bak.$(date +%Y%m%d%H%M%S)

sudo sed -i '/\/data\/rustfs0/d' /etc/fstab
sudo sed -i '/\/data\/rustfs1/d' /etc/fstab
sudo sed -i '/\/data\/rustfs2/d' /etc/fstab
sudo sed -i '/\/data\/rustfs3/d' /etc/fstab

echo "UUID=$SDB_UUID /data/rustfs0 xfs defaults,noatime,nofail 0 2" | sudo tee -a /etc/fstab
echo "UUID=$SDC_UUID /data/rustfs1 xfs defaults,noatime,nofail 0 2" | sudo tee -a /etc/fstab
echo "UUID=$SDD_UUID /data/rustfs2 xfs defaults,noatime,nofail 0 2" | sudo tee -a /etc/fstab
echo "UUID=$SDE_UUID /data/rustfs3 xfs defaults,noatime,nofail 0 2" | sudo tee -a /etc/fstab

sudo systemctl daemon-reload
sudo mount -a
```



set ownership for RustFS -> For now we create a system user named rustfs
```
sudo useradd --system --home /var/lib/rustfs --shell /usr/sbin/nologin rustfs || true

sudo mkdir -p /var/lib/rustfs
sudo mkdir -p /var/log/rustfs

sudo chown -R rustfs:rustfs /data/rustfs0 /data/rustfs1 /data/rustfs2 /data/rustfs3
sudo chown -R rustfs:rustfs /var/lib/rustfs /var/log/rustfs

sudo chmod 750 /data/rustfs0 /data/rustfs1 /data/rustfs2 /data/rustfs3
sudo chmod 750 /var/lib/rustfs /var/log/rustfs
```

Verify
```
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL
df -h | grep rustfs
mount | grep rustfs
sudo findmnt /data/rustfs0 /data/rustfs1 /data/rustfs2 /data/rustfs3
sudo ls -ld /data/rustfs0 /data/rustfs1 /data/rustfs2 /data/rustfs3

lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL
df -h | grep rustfs
sudo ls -ld /data/rustfs0 /data/rustfs1 /data/rustfs2 /data/rustfs3
```



### 5. Install Rustfs binary on all nodes
```
cd /tmp

wget https://dl.rustfs.com/artifacts/rustfs/release/rustfs-linux-x86_64-musl-latest.zip

unzip -o rustfs-linux-x86_64-musl-latest.zip

chmod +x rustfs

sudo mv rustfs /usr/local/bin/rustfs

/usr/local/bin/rustfs --version || true
```


















