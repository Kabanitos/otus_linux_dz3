# LVM
Перед началом выполнения домашнего задания необходимо поставить пакет `xfsdump`, который позволит нам снять копию / тома.
```
yum install xfsdump
```
###Уменьшить том под / до 8G
Подготовим временый том для / раздела
```
pvcreate /dev/sdb
vgcreate vg_root /dev/sdb
lvcreate -n lv_root -l +100%FREE /dev/vg_root
```
