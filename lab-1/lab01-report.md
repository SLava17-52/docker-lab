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

(ссылка на лекцию: [интерактивный режим](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#%D1%88%D0%B0%D0%B3-3-%D0%B2%D1%8B%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-%D0%B2-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D1%8E%D1%89%D0%B5%D0%BC-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%B5-docker-exec))
