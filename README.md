# serverga-deploy-qilish
Django project deploy qilish
# Django-Template

## Python Django Template

Django Frameworkida qilingan loyihamizni Debian/Ubuntu 20.04 serveriga joylash, PostgreSQL, Nginx, Gunicorn paketlarini sozlash va ishga tushurish

**ESLATMA:**
- Loyiha nomi `project_name`
- Ma'lumotlar bazasining nomi `database_name`
- Ma'lumotlar bazasining foydalanuvchisining ismi `user_name`
- Ma'lumotlar bazasining paroli `password`
- Ubuntu foydalanuvchi nomi `sammy`

Ushbu o'zgaruvchilar orniga o'zingizning ma'lumotlaringizni kiriting!

> Ushbu qo'llanmada men quyidagi loyihadan foydalanaman, sizning loyihangiz tuzilishi mening loyihamdan farq qilishi mumkin.

### Loyihaning tuzilishi:
├── projectname ├── appname │ ├── init.py │ ├── admin.py │ ├── apps.py │ ├── models.py │ ├── views.py │ └── urls.py ├── projectname │ ├── init.py │ ├── settings.py │ ├── urls.py │ └── wsgi.py ├── templates ├── static ├── media ├── manage.py └── requirements.txt

shell
Copy
Edit

## 1. Kerakli paketlarni o'rnatish

```bash
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install -y python3-pip python-dev libpq-dev vim git htop postgresql postgresql-contrib nginx redis-server
2. PostgreSQL ni sozlash
Oldin ma'lumotlar bazasini va shu ma'lumotlar bazasidan foydalanuvchi yaratib olamiz.

PostgreSQL ma'lumotlar bazasini yaratish:
bash
Copy
Edit
$ sudo -u postgres psql
postgres=# CREATE DATABASE database_name;
postgres=# CREATE USER user_name WITH PASSWORD 'password';
postgres=# ALTER ROLE user_name SET client_encoding TO 'utf8';
postgres=# ALTER ROLE user_name SET default_transaction_isolation TO 'read committed';
postgres=# ALTER ROLE user_name SET timezone TO 'UTC';
postgres=# GRANT ALL PRIVILEGES ON DATABASE database_name TO user_name;
postgres=# \q
3. Loyihani git repositoriyasidan yuklab olish
bash
Copy
Edit
$ git clone your_repository_link
4. Virtual muhit
Birinchi navbatda Virtual muhitni yaratib olamiz.

bash
Copy
Edit
$ cd my_project
$ python3 -m venv venv
Virtual muhitni aktivatsiya qilish va kerakli kutubxonalarni o'rnatish:

bash
Copy
Edit
$ . venv/bin/activate && pip3 install -r requirements.txt
Loyihani sozlash
Loyihani serverda ishga tushirish uchun settings.py faylida quyidagi o'zgartirishlarni kiritamiz:

python
Copy
Edit
ALLOWED_HOSTS = ['your_server_domain_or_IP', 'second_domain_or_IP']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'database_name',
        'USER': 'user_name',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

STATIC_ROOT = BASE_DIR / 'staticfiles'
Migratsiyalarni yaratib, ma'lumotlar bazasiga saqlaymiz:

bash
Copy
Edit
(venv)$ python3 manage.py makemigrations
(venv)$ python3 manage.py migrate
Superuser yaratish:

bash
Copy
Edit
(venv)$ python3 manage.py createsuperuser
Statik fayllarni bir joyga jamlash:

bash
Copy
Edit
(venv)$ python3 manage.py collectstatic
Gunicorn va Nginx bilan sozlash
Gunicorn uchun systemd xizmatini yaratish:
bash
Copy
Edit
$ sudo nano /etc/systemd/system/gunicorn.service
Quyidagi sozlamalarni kiriting:

ini
Copy
Edit
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
ExecStart=/home/sammy/myproject/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/sammy/myproject/myproject.sock myproject.wsgi:application

[Install]
WantedBy=multi-user.target
Xizmatni ishga tushurish:

bash
Copy
Edit
$ sudo systemctl start gunicorn
$ sudo systemctl enable gunicorn
$ sudo systemctl status gunicorn
Nginxni sozlash
Nginx uchun yangi konfiguratsiya faylini yaratish:

bash
Copy
Edit
$ sudo nano /etc/nginx/sites-available/myproject
Quyidagi sozlamalarni kiritamiz:

nginx
Copy
Edit
server {
    server_name server_domain_or_IP;
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        alias /home/sammy/myproject/staticfiles/;
    }    
    location /media/ {
        alias /home/sammy/myproject/media/;
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/sammy/myproject/myproject.sock;
    }
}
Nginx konfiguratsiyasini faollashtiramiz:

bash
Copy
Edit
$ sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
$ sudo nginx -t
$ sudo systemctl restart nginx
Serverni tekshirish
Brauzeringizda server IP yoki domen nomini yozing:

makefile
Copy
Edit
server_ip:8000
Shuningdek, SSL sertifikat olishni unutmang.

Xatoliklar
Agar 502 xatoligi bilan duch kelsangiz, quyidagi linklarga murojaat qiling:

Gunicorn and Django Error: Permission Denied for Sock
Nginx Stat Failed 13 Permission Denied
NGINX File Permission Issue
rust
Copy
Edit

Markdown faylini to'g'ridan-to'g'ri ko'chirib olib, foydalanishingiz mumkin.
