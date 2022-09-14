# ref: https://www.letscloud.io/community/how-to-set-up-an-nginx-with-certbot-on-ubuntu

# cambien el usuario: 'gcp-username' por el propio de su contexto en gcp, en el caso del profe: 'emontoyaeafit'

# 1. en una máquina GCP VP nueva y con la dirección IP Elastica de esa máquina, cree las entradas en el DNS:

    sudominio.com -> IP Elastica
    www.sudominio.com -> misma IP Elastica

# 2. instale certbot:

    sudo apt update
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt install letsencrypt -y
    sudo apt install nginx -y

## 2.1 configurar nginx.conf

use este template para dicho archivo:

    sudo vim /etc/nginx/nginx.conf

    worker_processes auto;
    error_log /var/log/nginx/error.log;
    pid /run/nginx.pid;

    events {
        worker_connections  1024;  ## Default: 1024
    }
    http {
        server {
            listen  80 default_server;
            server_name _;
            location ~ /\.well-known/acme-challenge/ {
                allow all;
                root /var/www/letsencrypt;
                try_files $uri = 404;
                break;
            }
        }
    }


    sudo mkdir -p /var/www/letsencrypt
    sudo nginx -t

    sudo service nginx reload

# 3.1 Ejecute certbot para pedir certificado SSL para registros especificos:

    sudo letsencrypt certonly -a webroot --webroot-path=/var/www/letsencrypt -m username@eafit.edu.co --agree-tos -d www.sudominio.tld

# 3.2  Ejecute certbot para pedir certificado SSL para todo el dominio (wildcard):

## pendiente de revisar para ubuntu 22.04!!!!!

ref: https://medium.com/@utkarsh_verma/how-to-obtain-a-wildcard-ssl-certificate-from-lets-encrypt-and-setup-nginx-to-use-wildcard-cfb050c8b33f

    sudo certbot --server https://acme-v02.api.letsencrypt.org/directory -d *.sudominio.com --manual --preferred-challenges dns-01 certonly

Este comando queda pausado indicando que debe crear un registro TXT en su dominio, una vez lo cree y verifique, dele ENTER para Continuar. Debe terminar con éxito.

# 4. cree el los archivos docker-compose

    mkdir /home/gcp-username/wordpress
    mkdir /home/gcp-username/wordpress/ssl
    sudo su

### comando para registros especificos (ej: www):
    cp /etc/letsencrypt/live/www.sudominio.com/* /home/gcp-username/wordpress/ssl/

### comando para wildcard (*.sudominio.com):
    cp /etc/letsencrypt/live/sudominio.com/* /home/gcp-username/wordpress/ssl/

    cp /etc/letsencrypt/options-ssl-nginx.conf /home/gcp-username/wordpress/ssl/
    cp /etc/letsencrypt/ssl-dhparams.pem /home/gcp-username/wordpress/ssl/
    exit

###     Nota: estas instrucciones son para nginx, pero si se requiere para haproxy:

        DOMAIN='sudominio.com' bash -c 'cat /etc/letsencrypt/live/$DOMAIN/fullchain.pem /etc/letsencrypt/live/$DOMAIN/privkey.pem > /etc/letsencrypt/$DOMAIN.pem'

        cp /etc/letsencrypt/live/sudominio.com/* /home/gcp-username/wordpress/ssl/

        exit

# 5. instalar docker y docker-compose en AMI Ubuntu:22.04

    sudo apt install docker.io -y
    sudo apt install docker-compose -y
    sudo apt install git -y

    sudo systemctl enable docker
    sudo systemctl start docker
    sudo usermod -a -G docker ubuntu

# 6. copie los archivos del docker al sitio propio e inicie

despues de clonar este repositorio en el destino:

    git clone https://github.com/st0263eafit/st0263-2022-2.git

    cd st0263-2022-2/docker-nginx-wordpress-ssl-letsencrypt
    sudo cp docker-compose.yml /home/gcp-username/wordpress
    sudo cp nginx.conf /home/gcp-username/wordpress
    sudo cp ssl.conf /home/gcp-username/wordpress

# 7. inicie el servidor de wordpress en docker.

VERIFIQUE QUE NO ESTE CORRIENDO nginx NATIVO EN LA MÁQUINA, detengalo!!!!

    ps ax | grep nginx
    netstat -an | grep 80

    sudo systemctl disable nginx
    sudo systemctl stop nginx

vuelve y se conecta a la máquina para que ese proceso no esté corriendo.

UNA VEZ DETENIDO:

    cd /home/gcp-username/wordpress
    docker-compose up --build -d

# 8. pruebe desde un browser:

    https://sudominio.com o https://www.sudominio.com

# 9.  FELICITACIONES, lo logro!!!!!

# alternativas:

# Generar sus propios certificados SSL sin una CA:

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ssl/nginx.key -out ssl/nginx.crt
    
        Renombre los archivos: nginx.key y nginx.crt a los correspondientes privkey.pem y fullchain.pem en el directorio: /home/gcp-username/wordpress/ssl/
