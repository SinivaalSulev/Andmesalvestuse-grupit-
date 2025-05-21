![ceph_slide_pic](https://github.com/user-attachments/assets/37d71548-4761-4f95-a60b-6d8b08b7a288)


© Rookie

### Algus
Logisime ILO-desse, seadistasime RAID 5 logical volume'i. Parem tõrketaluvus ja efektiivsem salvestusruumi kasutus koos parema lugemiskiirusega on RAID 5 valimise põhjuseks.

Paigaldasime ILO kaudu Ubuntu 20.04.

Ubuntu paigalduse käiguse lõime partitsioonid (80GB for OP SYS, 1.8TB unformated).

# Ettevalmistus Ceph püstipanekuks
### Kontrollisime OS ühilduvust Ceph dokumentatsioonist 
https://docs.ceph.com/en/reef/start/os-recommendations/
### Serverite 192.168.184.207, 192.168.184.208, 192.168.184.209 ettevalmistus CEPH paigalduseks
1. Nimeserverite confimine /etc/netplan/00-installer-config.yaml
2. OpenSSH serveri paigaldus 
3. Root user ettevalmistus (passwd, PermitRootLogin)
4. Teenuste paigaldamine (Docker, LVM2) ja timedatectl status sync check
5. Server 3 (192.168.184.209) managerile ssh võtme genereerimine ja Server1, Server2 avaliku ssh võtme saatmine (ssh-copy-id)

# Ceph paigaldamine

### CEPH installation
```
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/octopus/src/cephadm/cephadm
chmod +x cephadm

./cephadm add-repo --release octopus

./cephadm install

cephadm bootstrap --mon-ip 192.168.184.209

```
### Paigalduse õnnestumisel saab ligipääsu Ceph dashboardile
```
Ceph Dashboard is now available at:
            URL: https://192.168.184.209:8443/
            User: admin
            Password: (keepass)
```
### Paigalda ceph common ja lisa clusterisse server 1, server 2
```
cephadm install ceph-common

ssh-copy-id -f -i /etc/ceph/ceph.pub root@192.168.184.207
ssh-copy-id -f -i /etc/ceph/ceph.pub root@192.168.184.208

ceph orch host add server1 192.168.184.207
ceph orch host add server2 192.168.184.208 --labels _admin

```

### Server 1 OSD loomine
```
sudo pvcreate /dev/sda3
sudo vgcreate vg_osd_srv1 /dev/sda3
sudo lvcreate -n lv_osd_srv1 -l 100%FREE vg_osd_srv1

ceph orch daemon add osd server1:/dev/vg_osd_srv1/lv_osd_srv1
```
### Server 2 OSD loomine
```
sudo pvcreate /dev/sda3
sudo vgcreate vg_osd /dev/sda3
sudo lvcreate -n lv_osd -l 100%FREE vg_osd

ceph orch daemon add osd server2:/dev/vg_osd/lv_osd
```
### Server 3 OSD loomine
```
sudo pvcreate /dev/sda3
sudo vgcreate vg_osd_srv3 /dev/sda3
sudo lvcreate -n lv_osd_srv3 -l 100%FREE vg_osd_srv3

ceph orch daemon add osd server3:/dev/vg_osd_srv3/lv_osd_srv3
```

## CEPH failisüsteemi loomine
```
ceph fs volume create cephfs
```

### Failisüsteemi mount
```
mkdir /mnt/ceph
mount -t ceph :/ /mnt/ceph -o name=admin

```
# Ceph manager serverisse mounti järel saab clusteri välistesse serveritesse cephi mountida:
```
# Remote TF machine
apt install ceph-common

# transfer credentials from from manager server /etc/ceph # ceph.client.admin.keyring ceph.conf ceph.pub rbdmap)

scp /etc/ceph/* root@192.168.180.41:/etc/ceph

mkdir /mnt/ceph
mount -t ceph :/ /mnt/ceph -o name=admin
```

# Kasulikud käsud CEPH haldamiseks

### CEPH staatuse vaatamiseks
```
ceph -s 
```
### CEPH puu vaatamine
```
ceph osd tree
```

# Ligipääsud

<img width="583" alt="Pilt" src="https://github.com/user-attachments/assets/5edbee0e-811e-479a-a86e-ad6bfdd8d2f7" />

Läbivalt on kasutatud kasutajatunnustena: student, student1234. Töökeskkonnas kasutaksime paroolihaldurit (Keepass, Secret server jne)

### Terraform VM
1. Skriptile lisasin UNI-id, parooli .env muutujana ja grupi nime
2. Skripti käivitati WSL2 ubuntu masinas
3. Võrguühenduse confimine /etc/netplan/00-installer-config.yaml (DHCP, nimiserver)
4. Paketide uuendamine
5. MySQL paigaldus

### MySQL backup
Töökeskkonnas rakendaksime 3-2-1 reeglit. Saadaksime krüpteeritud backupi rsynciga offsite serverisse ja kohalikku serverisse salvestaks kaks koopiat.

```
mysql -u root -p -e "CREATE DATABASE ylikoolid_test;"
mysql -u root -p ylikoolid_test < /mnt/ceph/ylikoolid.sql
```

## Sources
1. https://docs.ceph.com/en/reef/start/os-recommendations/
2. https://docs.ceph.com/en/octopus/install/get-packages/
3. https://docs.ceph.com/en/octopus/cephadm/install/
4. https://docs.ceph.com/en/reef/cephadm/install/#cephadm-deploying-new-cluster
5. https://docs.docker.com/engine/install/ubuntu/
