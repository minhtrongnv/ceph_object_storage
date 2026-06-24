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
192.168.136.200 ceph-1 rustfs1
192.168.136.201 ceph-2 rustfs2
192.168.136.202 ceph-3 rustfs3
192.168.136.203 ceph-4 rustfs4
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
sudo apt install -y curl gnupg lsb-release chrony lvm2 podman openssh-server
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

### 4. Install cephadm on Node1
```
sudo apt update
sudo apt install -y cephadm
sudo cephadm add-repo --release squid
sudo cephadm install
sudo cephadm install ceph-common
```

Verify
```
which cephadm
cephadm version
ceph -v
```

























