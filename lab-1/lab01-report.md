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
