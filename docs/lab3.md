## Сетевой стек

### Анализ состояний TCP-соединений

Запустила Python HTTP сервер на порту 8080:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_1.png)

Проверила, слушающих TCP-сокеты: 

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_3.png)

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_2.png)

Подключилась к серверу через curl: 

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_4.png)

#### Проанализируйте состояние TCP-сокетов для порта 8080, объясните, почему есть сокет в состоянии TIME-WAIT, его роль и почему его нельзя удалить.
В TCP сторона, которая активно закрыла соединение (первая отправила FIN), переходит в TIME-WAIT

TIME-WAIT нельзя удалить потому что: 
- Защищает от задержавшихся пакетов, чтобы старые сегменты от предыдущего соединения не влетели в новое
- Надёжно закрывает, например, если придёт повтор FIN от peer, хост в TIME-WAIT может переотослать финальный ACK
- Блокирует немедленное реиспользование 4-кортежа, обеспечивает короткую паузу, чтобы стек не перепутал старое и новое соединение

#### Опишите, к каким проблемам может привести большое количество TIME-WAIT сокетов.
TIME-WAIT — это защита TCP, не баг. Проблемы начинаются, когда соединений очень много и очень коротких. Тогда вылезают ограниченные ресурсы: порты, conntrack, память и таймеры.

### Динамическая маршрутизация с BIRD

Создала dummy-интерфейс с адресом 192.168.14.88/32, назвала его service_0:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_5.png)

При помощи BIRD проаннонсировала этот адрес при помощи протокола RIP v2 включенного на вашем интерфейсе (eth0/ens33), а так же любой другой будущий адрес при помощи протокола RIP v2 включенного на интерфейсе:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_6.png)

Создала три интерфейса:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_7.png)


### Настройка фаервола/ Host Firewalling
С помощью iptables создала правило, запрещающее подключения к порту 8080:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_8.png)

Запустите веб сервер на питоне

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_10.png)

Продемонстрировала работу firewall:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_9.png)
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_11.png)

### Аппаратное ускорение сетевого трафика (offloading)

С помощью ethtool посмотрела offload возможности сетевого адаптера:

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_12.png)

У меня выключен TCP segmentation offload.

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_13.png)

#### Объясните, какую задачу решает TCP segmentation offload.

Технология, при которой сетевой адаптер сам режет большой блок данных на мелкие TCP-пакеты нужного размера. Это делает не процессор, а карта. ОС отдаёт NIC большой кусок (например, 64–128 КБ), а NIC сам делит его на MSS-сегменты, дописывает TCP/IP заголовки и считает контрольные суммы.
Происходит меньше нагрузки на CPU — процессор не тратит время на подготовку тысяч мелких пакетов. Повышается скорость и стабильность на высоких скоростях сети (10G+) и меньше прерываний/оверহеда в сетевом стеке.
