# system_boot

1. Для домашней работы взял ВМ из работы по LVM
2. Проверил способ 1 - добавил init=/bin/sh в конец строки linux16 и нажал Ctrl+X для загрузки системы
3. Получил kernel panic - вывод, данный способ для CentOS не подходит

---

5. Проверил способ 2 - удалил rhgb quet в конце строки linux16, добавил rd.break и нажал Ctrl+X для загрузки системы
6. Система загрузилась в обычном режиме.
7. Перезагрузился и еще раз поправил строку с linux16, удалив console=tty0, console=ttyS0, rhgb quet и добавил в конец строки rd.break. Нажал Ctrl+X для загрузки системы
8. Система загрузилась в Emergency mode
9. Выполнил mount для проверки того, куда смонтирован раздел с системой - в /sysroot
10. Выполнил перемонтирование раздела /sysroot на чтение и запись (rw)
```
mount -o remount,rw /sysroot
```
11. Выполнил chroot в раздел /sysroot
12. Выполнил команду passwd для смены пароля root
13. Выполнил создание файла /.autorelable для переназначения флагов безопаности SELinux (необходимо только при ключенном SELinux)
14. Выполнил перезагрузку ОС
15. Авторизация прошла успешно

---

16. Проверил способ 3 - в строке linux16, заменил ro на rw и добавл строку rw init=/sysroot/bin/sh, затем нажал Ctrl+X для загрузки системы
17. Система просто не загрузилась.

---

18. 