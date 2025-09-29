## Linux Архитектура и файловые системы

### Задание 1. Kernel and Module Inspection

Версия ядра ОС:

![Версия ядра ОС:](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image18.png)

Список файлов в каталоге модулей ядра:

![Список файлов в каталоге модулей ядра:](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image9.png)

Все загруженные модули ядра:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image10.png)

Создала файл правилами:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image5.png)

В файле прописала правила:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image17.png)

Применила правила:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image15.png)

Посмотрела основное расположение конфигурационных файлов:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image3.png)

Вывела параметр CONFIG_XFS_FS:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image14.png)

Описание: 
- У параметра CONFIG_XFS_FS значение m (module)
- Файл конфигурации: /boot/config-6.14.0-15-generic
- Статус: Поддержка XFS доступна как загружаемый модуль
- Поддержка файловой системы XFS скомпилирована в виде отдельного загружаемого модуля
- Модуль не встроен непосредственно в ядро, а загружается по требованию
- Экономит память ядра когда XFS не используется

### Задание 2. Наблюдение за VFS

Использовала strace для анализа команды cat  /etc/os-release > /dev/null:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img.png)

Когда пишем > /dev/null, шелл до запуска cat открывает /dev/null поэтому когда трассируем только cat и не видим открытие /dev/null
сам cat открывает файл и читает содержимое кусками. Дальше показывает, что прочитал 386 байт, в stdout (fd=1), который уже «смотрит» на /dev/null:
Закрывает файл: “close(3) = 0”

вызовы write(...) есть - cat честно пишет в fd=1.
Просто fd=1 перенаправлен на /dev/null это “местная черная дыра”, которая принимает любые данные и сразу их отбрасывает. Поэтому пользовательского вывода не видно, но системный вызов успешный и возвращает количество записанных байт.
(делала через wsl)

### Задание 3. LVM Management 

Добавила к своей виртуальной машине диск размером 2GB (добавляла вручную):
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image6.png)

Создала раздел, используя parted, создала Physical Volume (PV) на этом разделе:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image19.png)

Создала два Logical Volume (LV):
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image1.png)

Logical Volume:
- data_lv на 1200 MiB
- logs_lv на всё оставшееся

Обновила индекс пакетов и поставила утилиты для XFS:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image4.png)

Подготовка logs_lv как XFS с меткой app_logs и готов к монтированию в /mnt/app_logs:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image2.png)

Отформатировала data_lv в ext4 и примонтировала в /mnt/app_data:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image16.png)

Проверила, что все смонтировано:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image23.png)

- vg_highload-data_lv → ext4, метка app_data, смонтирован в /mnt/app_data, ~1 ГиБ.
- vg_highload-logs_lv → xfs, метка app_logs, смонтирован в /mnt/app_logs, ~733–780 МиБ.


### Задание 4. Использование pseudo filesystem

Извлекла из /proc модель CPU:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image12.png)

Посмотрела объём памяти (KiB):
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image21.png)

$$ - это специальный параметр шелла (не обычная переменная). Он всегда развертывается в PID процесса шелла, который выполняет текущую команду.

Нашла PPid (родителя шелла) через /proc/$$/status:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image8.png)


- Pid: 7718 — это PID текущего шелла (bash). Значение переменной $$.
- PPid: 7709 — PID родительского процесса шелла

Посмотрела как называется основной диск и посмотрела I/O-scheduler именно для этого диска:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image13.png)

Определила свой основной интерфейс:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image11.png)

Посмотрела размер MTU для основного сетевого интерфейса:
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/image24.png)
