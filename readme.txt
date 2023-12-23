Deploy Django in Digital Ocean
Droplet Ubuntu Version - 23.10 x64

# Install Docker
sudo apt update

# Install Python
sudo apt install -y python3 python3-pip

clone git repo


sudo apt install python3.11-venv
python3 -m venv myenv
source myenv/bin/activate

pip install -r requirements.txt

pip install django gunicorn

python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic


settings.py:
--------------------------------------------------------------
ALLOW_HOST = ['*']

INSTALLED_APPS = [
'whitenoise.runserver_nostatic',
]

MIDDLEWARE = [
'whitenoise.middleware.WhiteNoiseMiddleware',
]


project/urls.py
---------------------------------------------------------------
from django.conf import settings
from django.conf.urls.static import static

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root = settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root = settings.STATIC_URL)

------------------


pip install whitenoise

sudo vim /etc/systemd/system/gunicorn.socket

--------------------------start

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target

---------------------------end



sudo vim /etc/systemd/system/gunicorn.service
---------------------------

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/YourProjectDirectoryName
ExecStart=/root/YourProjectDirectoryName/env/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          yourSettingsFileFoldername.wsgi:application
[Install]
WantedBy=multi-user.target

---------------------------


sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket

# Install Nginx
sudo apt install -y nginx

# start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx


sudo vim /etc/nginx/sites-available/project
------------------------------------

server {
    listen 80 default_server;
    server_name _;
    location = /favicon.ico { access_log off; log_not_found off; }
    location /YourStaticFilesDirectoryName/ {
        root /root/YourProjectDirectoryName;
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}

-------------------------------------


sudo ln -s /etc/nginx/sites-available/project /etc/nginx/sites-enabled/

sudo gpasswd -a www-data username



sudo systemctl restart nginx
sudo service gunicorn restart
sudo service nginx restart




if there are any error, then see logs:
--------------------------------------
sudo tail -f /var/log/nginx/error.log

#delete the file which is already in here :/etc/nginx/sites-enabled
cd /etc/nginx/sites-enabled
sudo rm -f default





