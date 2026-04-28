Домашнее задание к занятию 1 «Disaster recovery и Keepalived»   Дедяхин Игорь

### Задание 1
- Дана [схема](1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

### Решение
Настрока Router1:
```
enable
configure terminal
interface GigabitEthernet0/1
standby 1 track GigabitEthernet0/0
standby 1 priority 95
standby 1 preempt 
end
write memory
```
![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/Router1.jpg)


Настрока Router2:
```
enable
configure terminal
interface GigabitEthernet0/1
standby 1 track GigabitEthernet0/0
end
write memory
```

![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/Router2.jpg)


Пинг между PC0 и Server0 при разрыве

![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/PING_1.jpg)

![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/PING_2.jpg)



Схема в формате pkt: https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/hsrp_homework_Dedyakhin_IV.pkt


------

### Задание 2
- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

### Решение

Запускаем две ВМ с IP 192.168.1.76  и 192.168.1.77 (плавающий IP 192.168.1.222):

![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/ip_vm.jpg)

BASH скрипт для проверки доступности порта и существования index.html:
```bash
#!/bin/bash
ss -tunlp | grep -q '0.0.0.0:80' && [ -f /var/www/html/index.html ] && exit 0 || exit 1
```
Конфигурационный файл keepalived:

```
vrrp_script check_webserver {

        script "/home/igor/hm_keep/script.sh"
        interval 3
}

vrrp_instance VI_1 {
        state MASTER
        interface enp0s3
        virtual_router_id 222
        priority 255
        advert_int 1

        virtual_ipaddress {
              192.168.1.222/24
        }
        track_script {
                check_webserver
        }
}

```

Демонстрация переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

При обращении к плавающему IP в браузере видим стартовую страницу на ВМ 192.168.1.76

![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/Demo1.jpg)

Изменим порт в настройках nginx:

![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/8880.jpg)

После перезапуска nginx при обращении к плавающему IP в браузере видим уже стартовую страницу на ВМ 192.168.1.77:

![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/77.jpg)

Возвращаем работу nginx на 80 порт и переименуем index.html, в браузере снова открывается страница на ВМ 192.168.1.77:

![screen](https://github.com/igorsprint-code/sflt_homeworks_1/blob/main/index.jpg)










 



