## WEB. High Performance WEB и CDN

### Создание Server Pool для backend

Создала в home dir 2 папки ~/backend1 и ~/backend2, в каждой из которых лежит 1 index.html
внутри этих папок запустила питоновый http сервер на портах 8081 и 8082

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_14.png)

### DNS Load Balancing с помощью dnsmasq

При помощи dnsmasq создала 2 A записи

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_16.png)

Проверила, что все запускается

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_15.png)

Запустила  dnsmasq и при помощи dig обратилась к 127.0.0.1 для резолва my-awesome-highload-app.local

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_17.png)

#### Проанализируйте вывод, что произойдет с DNS записями если backend2 сервер сломается? 

dnsmasq это справочник: он не делает health-check’ов и не «выкидывает» мёртвые IP.
Даже если backend2 ляжет, ответ на DNS-запрос всё равно будет с двумя A-записями
Поэтому ничего с DNS не произойдёт

### Балансировка Layer 4 с  с помощью IPVS

Подняла локальный nginx-прокси на 127.0.0.1:8888, который шлёт трафик в пул hlpool
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_18.png)

Проверила, что все работает
![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_19.png)

Сымитировала падение активного сервера и проверила что запросы идут на 2 сервер

![](https://github.com/nastyane/Operation-of-high-load-systems/blob/master/images/img_20.png)


