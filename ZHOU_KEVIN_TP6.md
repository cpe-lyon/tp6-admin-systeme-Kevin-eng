# Exercice 1. Disques et partitions
**1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement
alloués ; puis démarrez la VM**

**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**

```bash
kz@serveur:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0   89M  1 loop /snap/core/7713
loop1    7:1    0 54,7M  1 loop /snap/lxd/12181
loop2    7:2    0 89,1M  1 loop /snap/core/7917
loop3    7:3    0 54,6M  1 loop /snap/lxd/11985
sda      8:0    0   10G  0 disk
├─sda1   8:1    0    1M  0 part
└─sda2   8:2    0   10G  0 part /
sdb      8:16   0    5G  0 disk
sr0     11:0    1 1024M  0 rom
```

**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)**

```bash
kz@serveur:~$ sudo fdisk /dev/sdb
Command (m for help): m
Partition type 
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10497607, default 10497607): +2G
Created a new partition 1 of type 'Linux' and of size 2 GiB.
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2):
First sector (4196352-10497607, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-10497607, default 10497607):

Created a new partition 2 of type 'Linux' and of size 3 GiB.

Command (m for help): t
Partition number (1,2, default 2): 1
Hex code (type L to list all codes): 83

Changed type of partition 'Linux' to 'Linux'.

Command (m for help): t
Partition number (1,2, default 2):
Hex code (type L to list all codes): 7

Changed type of partition 'Linux' to 'HPFS/NTFS/exFAT'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

**4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers.
A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)**

```bash
kz@serveur:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.44.6 (5-Mar-2019)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: 328760a6-31c6-4da5-b694-4162785fd416
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

kz@serveur:~$ sudo mkfs.ntfs /dev/sdb2
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.
```

**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?**

*On ne peut afficher le type de système de fichier car les partitions créées ne sont pas montées sur le système de fichier*

**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la
machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des
UUID en raison de l’impossibilité d’effectuer des copier-coller)**

```bash

kz@serveur:~$ nano  /etc/fstab
UUID=d783b0e5-f0e0-447b-b69c-0b3a24f56586 / ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
/dev/sdb1 /data ext4 defaults 0 0
/dev/sdb2 /win ntfs defaults 0 0

kz@serveur:~$ sudo mkdir /data /win

```
**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**
```bash
kz@serveur:~$ sudo mount -a
kz@serveur:~$ sudo reboot
```
# Exercice 2. Partitionnement LVM

**1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de
fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes
du fichier /etc/fstab**

*On supprime les lignes du fichier:*
```bash
kz@serveur:~$ sudo nano /etc/fstab
kz@serveur:~$ sudo mount -a
```
**2. Supprimez les deux partitions du disque, et créez une patition unique de type LVM**

** La création d’une partition LVM n’est pas indispensable, mais vivement recommandée quand
on utilise LVM sur un disque entier. En effet, elle permet d’indiquer à d’autres OS ou logiciels de
gestion de disques (qui ne reconnaissent pas forcément le format LVM) qu’il y a des données sur
ce disque.**

** Attention à ne pas supprimer la partition système !**

```bash

kz@serveur:~$ sudo fdisk /dev/sdb
Command (m for help): m
Command (m for help): d
Partition number (1,2, default 2): 1

Partition 1 has been deleted.

Command (m for help): d
Selected partition 2
Partition 2 has been deleted.
Command (m for help): w

Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'.

```



**3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en
utilisant la commande pvdisplay.**
** Toutes les commandes concernant les volumes physiques commencent par pv. Celles concernant
les groupes de volumes commencent par vg, et celles concernant les volumes logiques par lv**

```bash
kz@serveur:~$ sudo pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
kz@serveur:~$ sudo pvdisplay /dev/sdb1
  "/dev/sdb1" is a new physical volume of "5,00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               5,00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               iJUw6T-gYQZ-2V1y-Wckj-YVMf-PZQ8-vcOjvo
```

**4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que
le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.
 Par convention, on nomme généralement les groupes de volumes vgxx (où xx représente l’indice
du groupe de volume, en commençant par 00, puis 01...)**

```bash

kz@serveur:~$ sudo vgcreate vg01 /dev/sdb1
  Volume group "vg01" successfully created

kz@serveur:~$ sudo vgdisplay vg01
```

**5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.
 On peut renseigner la taille d’un volume logique soit de manière absolue avec l’option -L (par
exemple -L 10G pour créer un volume de 10 Gio), soit de manière relative avec l’option -l : -l
60%VG pour utiliser 60% de l’espace total du groupe de volumes, ou encore -l 100%FREE pour
utiliser la totalité de l’espace libre.**

```bash
kz@serveur:~$ sudo lvcreate -l 100%FREE vg01
  Logical volume "lvol0" created.

kz@serveur:~$ sudo lvdisplay vg01
  --- Logical volume ---
  LV Path                /dev/vg01/lvol0
  LV Name                lvol0
  VG Name                vg01
  LV UUID                PmnUF1-smzK-X4eu-g31l-2y2N-zrYp-ArREFc
  LV Write Access        read/write
  LV Creation host, time serveur, 2019-10-19 22:47:21 +0000
  LV Status              available
  # open                 0
  LV Size                5,00 GiB
  Current LE             1280
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

```
**6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans
l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.
 A ce stade, l’utilité de LVM peut paraître limitée. Il trouve tout son intérêt quand on veut par
exemple agrandir une partition à l’aide d’un nouveau disque.**

```bash
kz@serveur:~$ sudo fdisk /dev/mapper/vg01-lvol0
kz@serveur:~$ sudo mkfs.ext4 /dev/mapper/vg01-lvol0

UUID=d783b0e5-f0e0-447b-b69c-0b3a24f56586 / ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
/dev/mapper/vg01-lvol0 /data ext4 defaults 0 0
kz@serveur:~$ sudo mkdir /data

kz@serveur:~$ sudo mount -a
kz@serveur:~$ sudo reboot

```

**7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez
la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.**

```bash
kz@serveur:~$ lsblk
NAME           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0            7:0    0 54,7M  1 loop /snap/lxd/12211
loop1            7:1    0 89,1M  1 loop /snap/core/7917
loop2            7:2    0   89M  1 loop /snap/core/7713
loop3            7:3    0 54,7M  1 loop /snap/lxd/12181
sda              8:0    0   10G  0 disk
├─sda1           8:1    0    1M  0 part
└─sda2           8:2    0   10G  0 part /
sdb              8:16   0    5G  0 disk
└─sdb1           8:17   0    5G  0 part
  └─vg01-lvol0 253:0    0    5G  0 lvm  /data
sdc              8:32   0    1G  0 disk
sr0             11:0    1 1024M  0 rom

kz@serveur:~$ fdisk /dev/sdc

kz@serveur:~$ sudo pvcreate /dev/sdc1
  Physical volume "/dev/sdc1" successfully created.


```

**8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes**

```bash
kz@serveur:~$ sudo vgextend vg01 /dev/sdc1
  Volume group "vg01" successfully extended
```
**9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas
oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.**

```bash
kz@serveur:~$ sudo lvextend -l 100%FREE /dev/vg01/lvol0
  New size given (258 extents) not larger than existing size (1280 extents)

kz@serveur:~$ sudo resize2fs /dev/vg01/lvol0
resize2fs 1.44.6 (5-Mar-2019)


```
