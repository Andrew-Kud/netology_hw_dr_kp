# Домашнее задание к занятию "`Disaster recovery и Keepalived`" - `Кудряшов Андрей`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

<img width="1358" height="905" alt="3" src="https://github.com/user-attachments/assets/1f1c9b72-ae3e-467c-a57d-4121f255a379" />
 [1.pkt](https://drive.google.com/file/d/13DOm2KZjFuGmtFgFLAqdOosz4WbtibcF/view?usp=sharing)


---

### Задание 2

Скрипт bash:
```
#!/bin/bash


PORT=80
FILE="/var/www/html/index.nginx-debian.html"


if timeout 1 bash -c ":</dev/tcp/localhost/$PORT" 2>/dev/null && [ -f "$FILE" ]; then
    exit 0
else
    exit 1
fi
```

Кофиг-файл Мастера:
```
global_defs {
        enable_script_security
}


vrrp_script check_port {
        script "/usr/local/bin/check1.sh"
        interval 3
        weight 100
        user root
}


vrrp_instance VI_1 {
        state BACKUP
        interface enp0s3
        virtual_router_id 15
        priority 210
        advert_int 1

        virtual_ipaddress {
              192.168.10.15/24
        }

        track_script {
                check_port
        }
}
```

Конфиг-файл Бэкапа:
```
global_defs {
        enable_script_security
}


vrrp_script check_port {
        script "/usr/local/bin/check1.sh"
        interval 3
        weight 100
        user root
}


vrrp_instance VI_1 {
        state BACKUP
        interface enp0s3
        virtual_router_id 15
        priority 200
        advert_int 1

        virtual_ipaddress {
              192.168.10.15/24
        }

        track_script {
                check_port
        }
}
```
<img width="2563" height="563" alt="4" src="https://github.com/user-attachments/assets/3f2418e5-2f75-4543-a3d7-2c56b5a8046d" />

<img width="2561" height="698" alt="5" src="https://github.com/user-attachments/assets/a0bcda1b-af73-426f-b55c-00d24cc9eb25" />

<img width="2567" height="908" alt="6" src="https://github.com/user-attachments/assets/7a7cdd06-8fcc-4d05-bd42-dff90c366aa1" />

<img width="2559" height="1221" alt="7" src="https://github.com/user-attachments/assets/97d401ed-ec34-4f65-bf92-7057b93b27ae" />


---

### Задание 3

keepalived.conf
```
global_defs {
    enable_script_security
}

vrrp_script check_priority {
    script "/usr/local/bin/check_priority.sh"
    interval 3
    weight 100
    user root
}

vrrp_instance VI_1 {
    state BACKUP
    interface enp0s3
    virtual_router_id 15
    priority 200
    advert_int 1

    virtual_ipaddress {
        192.168.10.15/24
    }

    track_script {
        check_priority
    }
}
```

Скрипт проверки загруженности CPU /usr/local/bin/update-priority.sh (он же добавляется в cron):
```
#!/bin/bash

LOAD=$(cat /proc/loadavg | awk '{print $1}')
PRIORITY=$(awk "BEGIN { p = 200 - ($LOAD * 25); if (p < 100) p = 100; if (p > 200) p = 200; print int(p) }")
echo $PRIORITY > /tmp/keepalived_priority
```

Скрипт проверки приоритета = CPU:
```
#!/bin/bash

PRIORITY_FILE="/tmp/keepalived_priority"
CURRENT_PRIORITY=$(cat "$PRIORITY_FILE" 2>/dev/null || echo 150)
THRESHOLD=180

if [ "$CURRENT_PRIORITY" -lt "$THRESHOLD" ]; then
    exit 1
else
    exit 0
fi
```

<img width="2560" height="1440" alt="7" src="https://github.com/user-attachments/assets/fd0e52c2-0e06-42fe-aa99-4c447ecf7c5c" />

<img width="2559" height="1439" alt="8" src="https://github.com/user-attachments/assets/4b6e991e-1667-42c6-982a-13f9da403c81" />

<img width="2558" height="1439" alt="9" src="https://github.com/user-attachments/assets/b60a5efe-2166-4595-8c1f-b8d63929d1df" />



---

