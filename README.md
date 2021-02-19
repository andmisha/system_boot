# system_boot by andmisha

1. Для домашней работы взял ВМ из работы по LVM
2. Проверил способ 1 - добавил init=/bin/sh в конец строки linux16 и нажал Ctrl+X для загрузки системы
![](https://github.com/andmisha/system_boot/blob/main/Screenshot_13.png)
3. Получил kernel panic - вывод, данный способ для CentOS не подходит
![](https://github.com/andmisha/system_boot/blob/main/Screenshot_14.png)
---

4. Проверил способ 2 - удалил rhgb quet в конце строки linux16, добавил rd.break и нажал Ctrl+X для загрузки системы
![](https://github.com/andmisha/system_boot/blob/main/Screenshot_15.png)
5. Система загрузилась в обычном режиме.
6. Перезагрузился и еще раз поправил строку с linux16, удалив console=tty0, console=ttyS0, rhgb quet и добавил в конец строки rd.break. Нажал Ctrl+X для загрузки системы
7. Система загрузилась в Emergency mode
8. Выполнил mount для проверки того, куда смонтирован раздел с системой - в /sysroot
![](https://github.com/andmisha/system_boot/blob/main/Screenshot_16.png)
9. Выполнил перемонтирование раздела /sysroot на чтение и запись (rw)
```
mount -o remount,rw /sysroot
```
10. Выполнил chroot в раздел /sysroot
![](https://github.com/andmisha/system_boot/blob/main/Screenshot_17.png)
11. Выполнил команду passwd для смены пароля root
![](https://github.com/andmisha/system_boot/blob/main/Screenshot_18.png)
12. Выполнил создание файла /.autorelable для переназначения флагов безопаности SELinux (необходимо только при ключенном SELinux)

![](https://github.com/andmisha/system_boot/blob/main/Screenshot_19.png)

13. Выполнил перезагрузку ОС
14. Авторизация прошла успешно

---

15. Проверил способ 3 - в строке linux16, заменил ro на rw и добавл строку rw init=/sysroot/bin/sh, затем нажал Ctrl+X для загрузки системы
16. Система просто не загрузилась, моргает курсор и на этом все.
![](https://github.com/andmisha/system_boot/blob/main/Screenshot_20.png)

---

17. Текущая система у меня уже с LVM, выполнил vgs
```
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0
```
18. Переименовал VG
```
[root@lvm ~]# vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"
```
19. Внес изменения в файлы /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg - везде заменил VolGroup00 на OtusRoot
20. Выполнил пересоздание initrd image для того, чтобы система могла грузится с новым именем VG - OtusRoot
```
[root@lvm ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
21. Выполнил перезагрузку ОС и проверил командой vgs результат
```
[root@lvm ~]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0
```
---
22. Для добавления модуля в initrd необходимо создать директорию в /usr/lib/dracut/modules.d/01test
23. Скопировать в директорию 2 скрипта:
### https://gist.github.com/lalbrekht/e51b2580b47bb5a150bd1a002f16ae85 - установка модуля и вызов скрипта
### https://gist.github.com/lalbrekht/ac45d7a6c6856baea348e64fac43faf0 - сам скрипт
24. Пересобрал образ initrd
```
[root@lvm 01test]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: test ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
25. Проверил, что созданный ранее модуль test загружен в образ
```
[root@lvm 01test]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
26. Перезагрузил ОС, в grub при загрузке убрал опции rhgb quet и нажал Ctrl+X для загрузки
27. Дождался появления пингвинчика
![](https://github.com/andmisha/system_boot/blob/main/Screenshot_21.png)

