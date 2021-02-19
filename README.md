# system_boot

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

17. 
