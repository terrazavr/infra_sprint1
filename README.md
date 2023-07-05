# О чем проект?
**Kittygram** - социальная сеть для владельцев и любителей сильных и незывисимых животных - КОШЕК и КОТОВ. Не имеет значения, домашнее или дикое животное, если вы счастливый обладатель тигрицы или пантеры - не стесняйтесь, делитесь с миром их фотографиями и достижениями!
# Как запустить проект локально?
Клонировать репозиторий и перейти в него в командной строке:

    git clone <your repository>/infra_sprint1.git
    cd infra_sprint1/backend
Cоздать и активировать виртуальное окружение:
    
    python3 -m venv venv
Если у вас Linux/macOS

    source venv/bin/activate
Если у вас windows
    
    source venv/scripts/activate
Обновить pip

    python3 -m pip install --upgrade pip
Установить зависимости из файла requirements.txt:

    pip install -r requirements.txt
Выполнить миграции:

    python3 manage.py migrate
Создать в корневой папке файл .env, добавить SECRET_KEY
***
### Для всех, кто хочет более глобальный проект, следуйте инструкции ниже
***
# Деплой проекта на удаленный сервер

 ### Подключение к GitHub к удаленному серверу
Установить Git:  
    
    sudo apt install git
Сгенерировать пару SSH-ключей:  
    
    ssh-keygen
Сохранить открытый ключ на gitHub аккаунте:  

    cat .ssh/id_rsa.pub
Клонировать проект на удаленный сервер:  

    git clone <your repository>/infra_sprint1.git
***
### Запуск бэкенд-приложения
Установить на сервер пакетный менеджер и утилтиту для создания виртуального окружения:  

    sudo apt install python3-pip python3-venv -y
Создать виртуальное окружение:
    
    python3 -m venv venv
Активировать виртуальное окружение:  
    
    source venv/bin/activate
Установить зависимости:  

    pip install -r requirements.txt
Выполнить миграции:   

    python manage.py migrate
Создать суперпользователя:  
    
    python manage.py createsuperuser
В файле settings.py в ALLOWED_HOSTS добавить IP-адрес сервера, а так же '127.0.0.1', 'localhost' и ваше доменное имя.  
***
### Запуск фронтенд-приложения
Установить на сервер пакетный менеджер npm:  
    
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
    sudo apt-get install -y nodejs
Установить зависимости для фронтенда из директории ***infra_sprint1/frontend***:  
    
    npm i
***
### Установка и запуск Gunicorn
    
    pip install gunicorn==20.1.0 
Создать юнит для gunicorn:  

    sudo nano /etc/systemd/system/gunicorn.service
В файле gunicorn.service описать конфигурацию процесса:
```html
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=yc-user 
WorkingDirectory=/home/yc-user/taski/backend/
ExecStart=/home/yc-user/taski/backend/venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target  
```

Запустить процесс gunicorn.service: 

    sudo systemctl start gunicorn 
Добавим процесс в автозапуск:  

    sudo systemctl enable gunicorn
***
### Установка Nginx

    sudo apt install nginx -y
Запустить Nginx:  

    sudo systemctl start nginx
Указать файрволу, какие порты должны остаться открытыми:  

    sudo ufw allow 'Nginx Full'
    sudo ufw allow OpenSSH
Включить файрвол:  
    
    sudo ufw enable
***
### Собрать статику фронтенд-приложения
Из директории **infra_sprint1/frontend** запустить:  
    
    npm run build
    sudo cp -r /home/yc-user/infra_sprint1/frontend/build/. /var/www/kittygram/

Написать конфигурационный файл для работы со статикой фронтенд-приложения:   
    
    sudo nano /etc/nginx/sites-enabled/default
Удалить все настроки из файла и добавить:  
```html
server { 
    server_name Your_IP Your_DNS; 
    location /api/ { 
        client_max_body_size 20M; 
        proxy_pass http://127.0.0.1:8080; 
    } 
    location /admin/ { 
        client_max_body_size 20M; 
        proxy_pass http://127.0.0.1:8080; 
    } 
    location /media/ { 
	alias /var/www/kittygram/media/; 
    } 
    location / { 
        root   /var/www/kittygram; 
        index  index.html index.htm; 
        try_files $uri /index.html =404; 
    } 
```
***
### Описать настройки для работы с бэкенд-приложением
В файле **settings.py** изменить(добавить): 

    STATIC_URL = '/static_backend/'
    STATIC_ROOT = BASE_DIR / 'static_backend'
При активированном виртуальном окрежении выполните команду:  
    python manage.py collectstatic
Перейдите в корень проекта и выполните команду:  
    sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/
***
### Натройка шифрования
Установить **certbot**:  
   
    sudo apt install snapd
    sudo snap install core; sudo snap refresh core
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot  
Запустим certbot и получим SSL-сертификуат:  

    sudo certbot --nginx 
Перезагрузить конфигурацию Nginx:  
    
    sudo systemctl reload nginx
Настройка автоматического обновления SSL-сертификата:  
    
    sudo certbot certificates
    sudo certbot renew --dry-run
***
## Ура! Проект в сети!
***
* **Автор** [Диляра Юнусова](https://github.com/terrazavr)
