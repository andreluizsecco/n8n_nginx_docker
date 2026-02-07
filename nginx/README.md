# Criar a rede manualmente antes

```sh
docker network create shared-net
```

## Setup inicial (com instalação de certificado)

* 1 - Criar pastas locais ou transferí-las caso já existam
```sh
mkdir -p certbot/conf
mkdir -p certbot/www
mkdir -p certbot/www/.well-known/acme-challenge
```

* 2 - Primeira versão do arquivo nginx.conf tem que ser sem ter apontamentos para SSL. Exemplo abaixo:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name n8n.seusite.com.br;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 200 "Waiting for SSL certificate";
    }
}
```

* 3 - Executar docker compose apenas com o nginx

```sh
docker compose up -d nginx
```

OBS: olhar os logs e não deve estar apontando nenhum erro (docker logs nginx)

* 3 - Executar o comando para instalar o certificado inicial (só é preciso fazer na primeira instalação):

```sh
docker compose run --rm --entrypoint certbot certbot certonly --webroot --webroot-path=/var/www/certbot --email email@seusite.com.br --agree-tos --no-eff-email -d n8n.seusite.com.br --verbose
```

**IMPORTANTE**: A porta 80 deve estar liberada no firewall da VPS

* 4 - Parar o nginx

```sh
docker compose down nginx
```

* 5 - Alterar o arquivo nginx.conf para versão final com SSL. Exemplo abaixo:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name n8n.seusite.com.br;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
	http2 on;

    server_name n8n.seusite.com.br;
    server_tokens off;

    ssl_certificate /etc/letsencrypt/live/n8n.seusite.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/n8n.seusite.com.br/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    location / {
        proxy_pass http://n8n:5678;
		
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
* 6 - Subir novamente o nginx com o certbot

```sh
docker compose up -d
```

* 7 - Liberar a porta 443 no firewall da VPS
