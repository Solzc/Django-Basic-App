# Django-Basic-App

Aplicación básica en Django para demostración con AWS.
Este repositorio incluye lo necesario para lanzar un servicio
básico de Django en una instancia EC2 en AWS.

Realizado por Andrés Arias para la charla introductoria
de computación en la nube.

## Pasos

### Instalar pip3
La AMI de Ubuntu 18.04 no provee pip3 por defecto.
```
sudo apt update
sudo apt install python3-pip
```

### Instalar virtualenv
Es buena práctica aislar aplicaciones de Python en su propio
ambiente virtual.
```
sudo pip3 install virtualenv
```

### Clonar este repositorio
Clone este repositorio en su instancia de EC2
```
git clone https://github.com/andres-arias/Django-Basic-App.git aws_talk
cd aws_talk
```

### Inicialice un ambiente virtual en el proyecto
```
virtualenv venv
source venv/bin/activate
```

### Instale los paquetes de Python requeridos
En este caso es Django y Gunicorn
```
pip3 install -r requirements.txt
```

### Inicialice el servicio de Django
```
python3 manage.py migrate
python3 manage.py createsuperuser
python3 manage.py collectstatic
```

### Cree un servicio de Gunicorn con systemd
Mueva el unit file al directorio de servicios de systemd
```
sudo cp files/gunicorn.service /etc/systemd/system
```

Luego inicialice el servicio
```
sudo systemctl enable gunicorn.service
sudo systemctl start gunicorn.service
```

### Instale y configure NGINX
NGINX se va a encargar de escuchar el puerto HTTP (80) y
direccionar las solicitudes al socket de Gunicorn.
```
sudo apt install nginx
sudo systemctl stop nginx.service
```

Luego mueva el archivo de configuración para NGINX y
actualice el host
```
sudo cp files/aws_talk.nginx /etc/nginx/sites-available/aws_talk
```

```
server {
    listen 80;
    server_name HOSTAQUI;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/aws_talk;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/aws_talk/aws_talk.sock;
    }
}
```

Haga el symlink a `sites-enabled` para indicarle a NGINX cuál
configuración utilizar:
```
sudo ln -s /etc/nginx/sites-available/aws_talk /etc/nginx/sites-enabled
```

Pruebe la configuración de NGINX
```
sudo nginx -t
```

Finalmente, reinicialice el servicio de NGINX:
```
sudo systemctl start nginx
```
