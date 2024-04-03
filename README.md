# IWNO6: Создание многоконтейнерного приложения

- [IWNO6: Создание многоконтейнерного приложения](#iwno6-создание-многоконтейнерного-приложения)
  - [Цель работы](#цель-работы)
  - [Задание](#задание)
  - [Выводы](#выводы)

## Цель работы
Ознакомиться с работой многоконтейнерного приложения на базе docker-compose.
## Задание
Создать php приложение на базе трех контейнеров: nginx, php-fpm, mariadb, используя docker-compose.

## Выполнение

Создайте репозиторий containers06 и скопируйте его себе на компьютер.

## Сайт на php

В директории containers06 создайте директорию mounts/site. В данную директорию перепишите сайт на php, созданный в рамках предмета по php.

## Конфигурационные файлы

Создайте файл .gitignore в корне проекта и добавьте в него строки:
```bash
# Ignore files and directories
mounts/site/*
```
Создайте в директории containers06 файл nginx/default.conf со следующим содержимым:
```bash
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
Создайте в директории containers06 файл docker-compose.yml со следующим содержимым:

```yml
version: '3.9'

services:
  frontend:
    image: nginx:1.19
    volumes:
      - ./mounts/site:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - internal
  backend:
    image: php:7.4-fpm
    volumes:
      - ./mounts/site:/var/www/html
    networks:
      - internal
    env_file:
      - mysql.env
  database:
    image: mysql:8.0
    env_file:
      - mysql.env
    networks:
      - internal
    volumes:
      - db_data:/var/lib/mysql

networks:
  internal: {}

volumes:
  db_data: {}
```

Создайте файл mysql.env в корне проекта и добавьте в него строки:

```env
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
```

## Запуск и тестирование

Запустите контейнеры командой:

    docker-compose up -d

Проверьте работу сайта в браузере, перейдя по адресу http://localhost. Если отображается базовая страница nginx, то перегрузите страницу.

## Ответы на вопросы

В каком порядке запускаются контейнеры?

Порядок запуска контейнеров в Docker Compose не определен

Где хранятся данные базы данных?

Данные базы данных хранятся в томе Docker, который называется db_data. Этот том монтируется в контейнере database в директорию /var/lib/mysql.

Как называются контейнеры проекта?

Контейнеры проекта называются frontend, backend и database

Вам необходимо добавить еще один файл app.env с переменной окружения APP_VERSION для сервисов backend и frontend. Как это сделать?

```yml
services:
  frontend:
    env_file:
      - mysql.env
      - app.env
  backend:
    env_file:
      - mysql.env
      - app.env
```

## Выводы


Из работы можно сделать следующие выводы:

1. Docker и docker-compose позволяют легко создавать и управлять многоконтейнерными приложениями.
2. Важно правильно организовать структуру файлов и настроек для корректной работы контейнеров.
3. Настройка сети и доступа к сервисам базы данных играет ключевую роль в безопасности и связности приложения.
4. Добавление переменных окружения можно осуществить через файлы .env и указание их в конфигурационных файлах для каждого сервиса.

Эта работа демонстрирует основы создания и управления многоконтейнерными приложениями, а также показывает важность организации файлов и настроек для успешного запуска и работы приложения.
