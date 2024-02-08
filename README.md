# Kittygram — социальная сеть для обмена фотографий котиков.

[![Main Kittygram workflow](https://github.com/Rudakov19/kittygram_final/actions/workflows/main.yml/badge.svg?branch=main)](https://github.com/Rudakov19/kittygram_final/actions/workflows/main.yml)

## Описание проекта
Данный проект служит интернет-площадкой для котоводов, где владельцы своих котиков могут выставлять их на показ включая имя, цвет, дату рождения и даже фото, а так же отмечать достижения кота. Благодаря данному ресурсу хозяева кошачих смогут создать реестр своих питомцев

## Технологии
 - Python 3.9
 - Django 3.2.3
 - Django REST framework 3.12.4
 - Nginx
 - Gunicorn
 - Docker
 - Postgres


## Локальное развертывание проекта
1. Клонируйте репозиторий [kittygram_final](https://github.com/Rudakov19/kittygram_final).
2. В каталоге с проектом создайте и активируйте виртуальное окружение: `python3 -m venv venv && source venv/bin/activate`
3. Установите зависимости: `pip install -r requirements.txt`.
4. Выполните миграции: `python manage.py migrate`.
5. Создайте суперюзера: `python manage.py createsuperuser`.
6. В файле settings.py список ALLOWED_HOSTS должен выглядеть так:  `ALLOWED_HOSTS = ['your_ip', '127.0.0.1', 'localhost', 'your_domain']`.

### Создание Docker-образов
1. Замените username на ваш логин на DockerHub:
```
cd frontend
docker build -t username/kittygram_frontend .
cd ../backend
docker build -t username/kittygram_backend .
cd ../nginx
docker build -t username/kittygram_gateway .
```
2. Загрузите образы на DockerHub:
```
docker push username/kittygram_frontend
docker push username/kittygram_backend
docker push username/kittygram_gateway
```

## Установка проекта на сервер

1. Подключитесь к удаленному серверу

```ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера ```

2. Создайте на сервере директорию kittygram

`mkdir kittygram`

3. Установка docker compose на сервер:
```
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt-get install docker-compose-plugin
```

В kittygram скопируйте файлы docker-compose.production.yml и .env:
```
scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
```

4. Запустите docker compose в режиме демона:

`sudo docker compose -f docker-compose.production.yml up -d`

5. Выполните миграции, соберите статику бэкенда и скопируйте их в /backend_static/static/:
```
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

6. На сервере в редакторе nano откройте конфиг Nginx:

`sudo nano /etc/nginx/sites-enabled/default`

7. Добавте настройки location в секции server:
```
location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:9000;
}
```

8. Проверьте работоспособность конфигураций и перезапустите Nginx:
```
sudo nginx -t
sudo service nginx reload
```
### Переменные окружения
В корневом каталоге проекта создайте файл .env:
```
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=kittygram_password
DB_NAME=kittygram
SECRET_KEY = '<ваш_ключ>'
DEBUG=False
ALLOWED_HOSTS='ваш домен'
```

### Автор:
[Дмитрий Рудаков](https://github.com/Rudakov19)
