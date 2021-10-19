# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.
    
![изображение](https://user-images.githubusercontent.com/89098193/137985834-76020e5b-3606-45df-8593-1b3835a3ca77.png)



2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

	Метрики:
   
	CPU:
   
	    node_cpu_seconds_total{cpu="0",mode="idle"} 2238.49

	    node_cpu_seconds_total{cpu="0",mode="system"} 16.72
       
	    node_cpu_seconds_total{cpu="0",mode="user"} 6.86
       
	    process_cpu_seconds_total
       
	    
	Memory:
   
	    node_memory_MemAvailable_bytes 
       
	    node_memory_MemFree_bytes
	    
	Disk:
   
	    node_disk_io_time_seconds_total{device="sda"} 
       
	    node_disk_read_bytes_total{device="sda"} 
       
	    node_disk_read_time_seconds_total{device="sda"} 
       
	    node_disk_write_time_seconds_total{device="sda"}
       
	    
	Network:
   
	    node_network_receive_errs_total{device="eth0"} 
       
	    node_network_receive_bytes_total{device="eth0"} 
       
	    node_network_transmit_bytes_total{device="eth0"}
       
       node_network_transmit_errs_total{device="eth0"}
  
3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

![изображение](https://user-images.githubusercontent.com/89098193/137985932-cb58bd8c-3081-4546-ac44-af145e4d0990.png)


4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?


Да, в выводе dmesg есть указания на то что система виртуализована 
![изображение](https://user-images.githubusercontent.com/89098193/137985988-91d2bc2c-cc9d-4c60-abef-6a426204171c.png)


5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?

	Параметр означает количество открытых файлов.
   
	Командой ulimit -n можно увеличить количество открытых файлов.

![изображение](https://user-images.githubusercontent.com/89098193/137986398-737adadb-5c32-42bf-939f-1588240982a5.png)


6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.

![изображение](https://user-images.githubusercontent.com/89098193/137986416-598615f4-7020-4ecc-a4f5-362d2b2d8d8a.png)


7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

 
:(){ :|:& };: - fork bomb ( заполняет всю память и переводить виртуальную машину в swap). Функция внутри "{}", с именем ":" , которая запускает саму себя.
	
	Посмотреть текущие настройки можно cat /sys/fs/cgroup/pids/user.slice/user-1000.slice/pids.max, изменить echo 10 | sudo tee /sys/fs/cgroup/pids/user.slice/user-1000.slice/pids.max
	
Также вариант установить ulimit -u 50 - число процессов будет ограниченно 50 для пользователя. 
