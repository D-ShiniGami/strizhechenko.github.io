---
title: Воспроизведение пакетов из PCAP-файла (tcpreplay)
---

Натолкнулся на замечательную утилиту tcpreplay. Для тестирования одной рабочей задачи требовалось обеспечить бесконечный поток трафика. В первом случае - с определённых IP на другие определённые IP. Во втором - с максимальной утилизацией канала. В обоих - трафик не должен был покидать изолированный сегмент сети (иначе бы я забил весь канал в офисе).

Первый случай решился утилитой `tcprewrite`. Мне нужен был двунаправленный поток трафика между двумя хостами, а оригинальный дамп содержал только исходящие запросы. На содержимое соединений мне было пофигу, поэтому сделал что-то в духе:

Генерируем исходящий поток:

``` shell
tcprewrite -i dump_target.pcap -o tx.pcap \
    -S 0.0.0.0/0:10.20.20.20/32 \
    -D 0.0.0.0/0:192.168.20.20/32
```

- `-D 0.0.0.0/0:10.20.20.20/32` - любой DST IP заменяется на 10.20.20.20. Можно было указать не конкретный хост, а сеть, например 10.20.20.0/24.
- `-S` - то же самое, но для SRC IP.


Генерируем входящий поток:

``` shell
tcprewrite -i dump_target.pcap -o rx.pcap \
    -D 0.0.0.0/0:10.20.20.20/32 \
    -S 0.0.0.0/0:192.168.20.20/32
```

Объединяем потоки в одном файле:

``` shell
mergecap rxtx.pcap rx.pcap tx.pcap
```

Во втором случае пришлось напрячь `tcpreplay` по максимуму.

``` shell
tcpreplay -K -t -l 10 --stats=1 -i eth1 dump.pcap
```

- `-K` - загрузить PCAP файл в память, чтобы не тратить время на перечитывание его с диска.
- `-t` - обеспечить отправку на максимальной скорости.
- `-l 10` - воспроизвести PCAP 10 раз. "Бесконечность" можно получить указав значение 99999999999.
- `--stats=1` - выводить статистику об отправке раз в 1 сек.

В принципе первое можно частично объединить со вторым, для этого служит утилита `tcpreplay-edit`, поддерживающая модификацию пакетов "на ходу", но скорость отправки пакетов с её помощью падала где-то на 25-30%.
