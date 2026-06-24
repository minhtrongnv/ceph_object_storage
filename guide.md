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
sudo tee /etc/hosts >/dev/null <<'EOF'
127.0.0.1 localhost

192.168.136.200 ceph-1
192.168.136.201 ceph-2
192.168.136.202 ceph-3
192.168.136.203 ceph-4
EOF
```

Verify
```
getent hosts ceph-1
getent hosts ceph-2
getent hosts ceph-3
getent hosts ceph-4
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

