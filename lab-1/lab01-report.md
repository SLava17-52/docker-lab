# docker-lab
# Лабораторная работа №1

## Среда

* **ОС:** Windows 11
* **Терминал:** Windows PowerShell
* **Версия docker:** 29.8.1

В качестве базового образа был выбран `nginx:alpine`.

## Задания

### 1) Версии и теги

* **Определите версии клиента и демона Docker.**
  
```bash
docker version
```

![docker version.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/dockerversion.png)

* **Запуск образа в конкретном теге(`nginx:alpine`)**

```bash
docker run -d --name test-nginx nginx:1.25-alpine
```

![nginx:alpine.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/nginxalpine.png)

(ссылка на лекцию: [запуск конкретного тега](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md))

### 2) Первый запуск сервиса (detached)

* Запуск контейнера в фоновом режиме с именем `lab-web-recrut17` на порту 8080, затем перезапуск на порту 6767

```bash
#запуск на порту 8080
docker run -d --name lab-web-recrut17 -p 8080:80 nginx:alpine
```
![Docker port 8080 site.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/docker%20site%208080.png)

```bash
#перезапуск контейнера через остановку(stop) и удаление(rm) 
docker stop lab-web-recrut17
docker rm lab-web-recrut17
```

```bash
#запуск на порту 6767
docker run -d --name lab-web-recrut17 -p 6767:80 nginx:alpine
```
![Docker port 6767 site.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/docker%20site%206767.png)

* Список работающих контейнеров
![Docker container.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/container.png)

(ссылки на лекции: [проброс портов](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#3-%D0%BF%D1%80%D0%BE%D0%B1%D1%80%D0%BE%D1%81-%D0%BF%D0%BE%D1%80%D1%82%D0%BE%D0%B2--p), [именование контейнеров](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#4-%D0%B8%D0%BC%D0%B5%D0%BD%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%BE%D0%B2---name))

### 3) Том (bind‑mount): сайт/данные из папки хоста
```bash
#создание папки
mkdir site

#создание и добавление текста в тестовый файл
"Hello" | Set-Content .\site\index.html
```

* Перезапуск `lab-web-recrut17`, смонтировав папку как `read‑only`
```bash
#перезапуск контейнера через остановку(stop) и удаление(rm) 
docker stop lab-web-recrut17
docker rm lab-web-recrut17

#монтирование папки через :ro
docker run -d --name lab-web-recrut17 -p 8080:80 -v "${PWD}\site:/usr/share/nginx/html:ro" nginx:alpine 
```

![Create directory and test file site.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/create%20directory%20and%20test%20file%20site.png)


* Изменение текста внутри файла
```bash
"Good bay" | Set-Content .\site\index.html
```

![Change test file site](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/change%20test%20file%20site.png)

* Почему данные переживут удаление контейнера?<br>
  Так как был создан одноразовый контейнер, в котором есть своя временная файловая система, которая удалится вместе с ним, но из-за того, что мы использовали том(-v), данные останутся. Все благодаря тому, что данные хранятся на постоянном носителе вне контейнера, а именно в папке на хосте.

(ссылка на лекцию: [монтирование тома](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#5-%D1%82%D0%BE%D0%BC-%D0%B4%D0%BB%D1%8F-%D1%85%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85--v))

### 4) Интерактив / exec

* Вход в интерактивный режим и просмотр содержимого каталога со статикой
```bash
#вход в интерактивный режим внутрь работающего контейнера
docker exec it lab-web-recrut17 /bin/sh

#переход в каталог со статикой 
cd /usr/share/nginx/html

#просмотр листинга каталога
ls -la
```
![interactive.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/interactive.png)

* Попытка создания файла с :ro

```bash
touch test.txt
```

![interactive and create file ro.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/interactive%20and%20create%20file%20ro.png)

* Удаление :ro и добавление файла с переходом в режим `read-write`
  
![interactive and create file.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/interactive%20and%20create%20file.png)

* Файл на хосте после режима RW

![interactive and list.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/interactive%20and%20list.png)

### 5) Логи и attach

```bash
#обновление страницы и переход по несуществующему пути
curl http://localhost:8080/
curl http://localhost:8080/nope
curl http://localhost:8080/nope123
curl http://localhost:8080/nope1234

#вывод 10 строк из журнала контейнера
docker logs --tail 10 lab-web-recrut17
```

![fragment magazine.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/logs.png)

```bash
#прикрепляемся к основному процессу контейнера
docker attach lab-web-recrut17
```
Для выхода из attach ( без остановки контейнера),  нажимаем Ctrl + P, затем Ctrl + Q.
Также ещё можно нажать Ctrl + C с помощью которого можно остановить основной процесс контейнера

(ссылки на лекции: [получение логов](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#8-%D0%BB%D0%BE%D0%B3%D0%B8-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%B0-docker-logs), [прикрепление и открепление от контейнера](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#10-docker-attach-%D0%B8--d-%D0%BD%D0%B0-%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80%D0%B5-%D0%B0%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0-%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0))

(ссылка на лекцию: [интерактивный режим](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#%D1%88%D0%B0%D0%B3-3-%D0%B2%D1%8B%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-%D0%B2-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D1%8E%D1%89%D0%B5%D0%BC-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%B5-docker-exec))

### 6) Краткоживущий процесс

```bash
#запускаем контейнер, указавая вывод строки Privet
docker run --name test alpine echo "Privet"

#вывод статуса контейнера после выполнения
docker ps -a
```
![short container.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/shortprocces.png)

* Контейнер работает только пока выполняется его основная команда. Команда `echo` срабатывает моментально, поэтому как только она заканчивается, контейнеру больше нечего делать — он сразу останавливается и получает статус `Exited`.

(ссылка на лекцию: [краткоживущий процесс](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#6-%D0%B0%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5-%D1%83%D0%B4%D0%B0%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5---rm))

### 7) Inspect
* Вырезка блока `mounts`
![mounts.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/mounts.png)

* Вырезка блока `ports`

![ports.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/ports.png)

(ссылка на лекцию: [инспекция контейнера](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#7-%D0%B8%D0%BD%D1%81%D0%BF%D0%B5%D0%BA%D1%86%D0%B8%D1%8F-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%B0-docker-inspect))

### 8) Чистка
* Используем команды для удаления контейнеров и образов

```bash
#удаление контейнеров
docker rm -f lab-web-recrut17 test

#удаление образов
docker rmi docker/welcome-to-docker:latest hello-world:latest
```
![full delete.png](https://github.com/SLava17-52/docker-lab/blob/main/lab-1/clear.png)
Не удалось удалить с первого раза потому что Docker хранит слои образов и ссылки на них. Даже если контейнер остановлен (Exited), он всё еще занимает место и "держит" образ, на основе которого был создан. Пока контейнер существует (пусть и в статусе Exited), его образ удалить нельзя. Поэтому сначала нужно удалить контейнеры, затем образы, тогда ошибки не будет

(ссылки на лекции: [удаление контейнеров](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#4-docker-rm), [удаление образов](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#4-docker-rm](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#6-docker-rmi)))

## Мини-квиз
### 1) Что произойдёт с данными, созданными внутри контейнера, если удалить контейнер без тома?
* Все данные будут утеряны без возможности восстановления. По умолчанию файлы, созданные внутри контейнера, хранятся в его записываемом слое `writable layer`. Этот слой существует только пока жив сам контейнер, поэтому при его удалении все изменения исчезают.
### 2) Чем отличается порт хоста от порта в контейнере?
* Порт контейнера — это внутренний, виртуальный порт, который существует только в изолированной сети контейнера и недоступен извне напрямую. Порт хоста — это реальный физический порт на самой машине (хосте). Чтобы обратиться к сервису внутри контейнера снаружи, нужно "пробросить" (опубликовать) внутренний порт на порт хоста.
### 3) Для чего нужна пара флагов интерактивного запуска, и когда одного из них достаточно?
Флаг `-i` (interactive) оставляет стандартный ввод открытым, чтобы можно было передавать команды внутрь контейнера.

Флаг `-t` (tty) выделяет псевдотерминал, делая работу с контейнером похожей на обычную работу в командной строке.
Чаще всего флаги используют вместе (-it) для полноценного интерактивного сеанса. Однако если нужно просто передать данные в контейнер через стандартный ввод (например, через пайп), достаточно одного -i, без создания терминала.
### 4) Что показывает `Mounts` в `inspect` и как понять, что это именно bind‑mount?
* Раздел `Mounts` в выводе docker `inspect`содержит список всех монтирований, подключённых к контейнеру (тома, bind-mount'ы и т.д.). Понять, что перед нами именно `bind-mount`, можно по полю `"Type": "bind"`. Дополнительно это подтверждает поле Source, в котором указан путь к реальной папке на хосте.
### 5) Почему образ может не удаляться, и что нужно сделать перед удалением?
* Образ не удаляется, если на него ссылается хотя бы один контейнер — даже остановленный. Docker не позволяет удалить образ, пока существует зависимый контейнер. Перед удалением образа нужно сначала удалить все связанные с ним контейнеры. Альтернатива — использовать флаг `-f`, но тогда контейнеры останутся в системе и станут неработоспособными, так как их базовый образ будет удалён.
