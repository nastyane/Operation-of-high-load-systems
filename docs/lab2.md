## Linux работа с памятью и процессами

### Задание 1. Systemd
Скачала пакеты:

![Скачала пакеты:](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img1.png)

Создала bash-скрипт:
![Создала bash-скрипт:](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img7.png)

Создала systemd unit файл для скрипта, который переживает любые обновления системы:
![systemd unit файл для скрипта](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img6.png)

Проверила, что сервис работает:
![Проверила, что сервис работает:](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img5.png)

Используя systemd-analyze посмотрела systemd unit`ов стартующих дольше всего:
![топ-5 systemd unit](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img3.png)

### Задание 2. Межпроцессное взаимодействие (IPC) с разделяемой памятью 

Создала шареную память на языке kotlin.
Код для создания шареной памяти и написания в шареную память:

``` ShmCreator.kt
import java.nio.file.*
import java.nio.channels.FileChannel
import java.nio.ByteBuffer
import java.nio.charset.StandardCharsets
import java.util.EnumSet

fun main() {
    val path = Paths.get("/dev/shm/homework_shm")
    val size = 1024
    val fc = FileChannel.open(path, EnumSet.of(StandardOpenOption.CREATE, StandardOpenOption.READ, StandardOpenOption.WRITE))
    fc.truncate(size.toLong())
    val buf = fc.map(FileChannel.MapMode.READ_WRITE, 0, size.toLong())
    val text = "Hello from Kotlin SHM"
    val bytes = text.toByteArray(StandardCharsets.UTF_8)
    buf.putInt(0xC0FFEE)
    buf.putInt(bytes.size)
    buf.put(bytes)
    println("Shared memory created at $path")
    println("PID: ${ProcessHandle.current().pid()}")
    println("Wrote: \"$text\"")
    println("Sleeping 60s, run reader meanwhile...")
    Thread.sleep(60_000)
    fc.close()
    Files.deleteIfExists(path)
    println("Shared memory removed")
}
```

Создание читателя:
``` ShmCreator.kt
cat > ShmReader.kt <<'EOF'
import java.nio.file.*
import java.nio.channels.FileChannel
import java.nio.charset.StandardCharsets
import java.util.EnumSet

fun main() {
    val path = Paths.get("/dev/shm/homework_shm")
    val fc = FileChannel.open(path, EnumSet.of(StandardOpenOption.READ, StandardOpenOption.WRITE))
    val size = fc.size()
    val buf = fc.map(FileChannel.MapMode.READ_WRITE, 0, size)
    val magic = buf.int
    val len = buf.int
    val data = ByteArray(len)
    buf.get(data)
    println("Magic: 0x${magic.toUInt().toString(16)}")
    println("Read: \"${String(data, StandardCharsets.UTF_8)}\"")
    fc.close()
}
EOF

```

Компиляция 2 файлов:

![компиляция](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img11.png)

Запустила файл для создания и написания в 1 терминале:

![компиляция](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img12.png)

Запустила файл для читателя во 2 терминале: 

![компиляция](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img13.png)

Потом я поняла, что ipcs -m не работает потому что я создала по POSIX IPC и немного переделала. 

Создание шарерной памяти: 

![компиляция](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img14.png)

Выполнение ipcs -m:

![компиляция](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img16.png)

Выполнив команду ipcs -m можно увидеть ключ, id сегмента, владельца, права, размер сегмента, сколько процессов сейчас прикреплены. У меня 0 тк никаких процессов я не прикрепляла. 

### Задание 3. Анализ памяти процессов (VSZ vs RSS) 

Открыла 1 окно терминала и запустила питон скрипт, который запрашивает 250 MiB памяти и держит ее 2 минуты:

![питон скрипт](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img2.png)

Открыла 2 окно терминала, узнала PID запущенного скрипта:

![PID](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img8.png)

Вывела данные о использовании RSS и VSZ:

![RSS и VSZ](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img9.png)

#### Почему vsz больше rss

VSZ больше RSS потому что VSZ - это всё адресное пространство процесса: heap, stack, код, мапнутые so-библиотеки, участки под аллокатор, guard-страницы, куски, которые процесс ещё не трогал. RSS - это только те страницы, которые реально находятся в RAM прямо сейчас.

#### Почему RSS далеко не 0

В скрипте есть строка a = 'X' * (250 * 1024 * 1024), это означает, что записывается ‘X’ в каждый байт новой строки. Запись = page fault = страница поднимается в RAM и становится «грязной» поэтому почти все 50 MiB будут физически заняты и еще есть расходы на интерпретатор, интерпретатор и тд

### Задание 4. NUMA и cgroups 

Количество NUMA нод на сервере:

![Количество NUMA нод](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img4.png)

Количество памяти для каждой NUMA ноды:

![Количество памяти](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img17.png)

