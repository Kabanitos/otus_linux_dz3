# LVM
Перед началом выполнения домашнего задания необходимо поставить пакет `xfsdump`, который позволит нам снять копию / тома.
```
yum install xfsdump
```
### Уменьшить том под / до 8G
Подготовим временый том для / раздела
```
pvcreate /dev/sdb
vgcreate vg_root /dev/sdb
lvcreate -n lv_root -l +100%FREE /dev/vg_root
```
Создаеи файлувую систему и монтируем его, чтобы перенести туда данные:
```
mkfs.xfs /dev/vg_root/lv_root
mount /dev/vg_root/lv_root /mnt
```
Копируем все данные с / раздела в /mnt
```
xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
```
При успешном копирование мы должны увидеть `SUCCESS`. Проверяем что все скопировалось командой `ls /mnt`
Далее нам необходимо переконфигурировать grub, для того чтобы при запуске ОС перейти на новый / раздел.
Для обновления grub используем `chroot`: 
```
 for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
 chroot /mnt/  
 grub2-mkconfig -o /boot/grub2/grub.cfg
```
Обновляем образ `initrd`
```
 cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
```
Для того, чтобы при загрузки ОС был смотирован нужный **root** нужно в файле **/boot/grub2/grub.cfg** заменить **rd.lvm.lv=VolGroup00/LogVol00** на **rd.lvm.lv=vg_root/lv_root**


Перезанружаем ОС с новым  root томом. 

Теперь необходимо изменить размер старой VG и вернуть на него **root**. Удаляем старый LV размером 40Gb и создаем новый на 8Gb 
```
lvremove /dev/VolGroup00/LogVol00
lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
```
Создаем файловую систему, монтируем ее и копируем  данные
```
mkfs.xfs /dev/VolGroup00/LogVol00
mount /dev/VolGroup00/LogVol00 /mnt
xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
```
Так же  как и в предыдущий раз переконфигурируем **grub**,  кроме правки файла  **/etc/grub2/grub.cfg**
```
 for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
 chroot /mnt/
 grub2-mkconfig -o /boot/grub2/grub.cfg
 ```
 ```
 cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
 ```
   

