# Primeiros Passos

Este trabalho é um resumo adaptado do conteúdo apresentado na documentação oficial que está na seção de referências.

## Configuração base Para Tests

OBS:

se você não quer ter o trabalho de configurar mais a fundo o servidor web nginx, basta fazer o seguinte:

Copie o conteúdo do seu site estático para o seguinte o seguinte diretório do nginx

```bash
/usr/share/nginx/html# 
```

prontinho, basta reiniciar o nginx e seu site estático estará pronto para visualização

## NGINX CLI

Podemos passar instruções para o NGINX usando `nginx -s "COMANDO"` onde a flag `-s` avisa ao nginx que uma instrução será passada.

Podemos passar as seguintes `flags` para as seguintes ações

- `stop` : Para o Nginx
- `quit` : Encerra de forma elegante o processo do NGINX
- `reload` : Recarrega os arquivos de configuração, onde as alterações feitas serão aplicadas
- `reopen`: Reabre os arquivos de log

## Arquivos de Configuração

O arquivo de configuração do nginx é nomeado como `nginx.conf`. Ele pode ser colocado em um dos seguintes diretórios `/usr/local/nginx/conf, /etc/nginx, or /usr/local/etc/nginx.`

Ele é encontrado mais comumente em `/etc/nginx`, principalmente se você usa a image `nginx a partir do docker`

Por padrão ele vem com esta configuração

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

## Sintaxe do Nginx

O `nginx` usa dois tipos de diretivas para suas configurações

- Diretiva em Linha: Neste formato, o comando fica em uma linha e deve ser sempre encerrar com `;`
  - Ex: `access_log  /var/log/nginx/access.log  main;`
- Diretiva em Bloco: Neste formato, o comando fica entre `{}` e pode conter outros comandos dentro.
  - Ex: `events { worker_connections  1024; }`

## Configurando O Ambiente do NGINX para Servir Várias Aplicações

Ao lado dos seus arquivos de configuração no diretório `/etc/nginx` iremos criar a pasta `www` e dentro dela duas pastas: `web1` e `web2`. Ao final teremos

```bash
conf.d/
fastcgi_params
mime.types
modules
nginx.conf
scgi_params
uwsgi_params
www/
    web1/
    web2/
```

Agora adicione um arquivo `index.html` tanto em `web1` quanto em `web2`

exemplo de conteúdo para os arquivos html

```html
<h1>Web 1</h1>
```

```html
<h1>Web 2</h1>
```

Com os arquivos html prontos, vamos configurar o arquivo `nginx.conf`, lembrando que todos os arquivos que configuramos estão em /etc/nginx/

Obs:

- *events* : Eventos que o nginx esta pronto para monitorar
- *worker_connections*: Número máximo de processos em paralelo acontecendo
- *http* : Protocolo Usado para as Requisições
- *server*: Configurando um "sub-servidor" dentro do nginx para cuidar de ações específicas
- *root* : Pega o conteúdo passado a frente de location e concatena como valor passado em root. Exemplo: `/etc/nginx/www/web1/`
- *index* : Define o arquivo dentro do caminho escolhido que será lido, caso nenhum nome de arquivo seja especificado na url. Exemplo: `/etc/nginx/www/web1/index.html
- *location* : URL passada pelo usuário ao fazer a requisição. Exemplo: `http://localhost:8080/web1/`
- *try_files* : Tenta executar em ordem, as possibilidades. Neste Caso tenta acessar a url de web1, caso não consiga, tenta acessar a url de web1/index.html

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;

        location / {
            root /etc/nginx/www;
            index index.html;
        }

        location /web1 {
            root /etc/nginx/www;
            index index.html;
            try_files $uri /web1/index.html;
        }

        location /web2 {
            root /etc/nginx/www;
            index index.html;
            try_files $uri /web2/index.html;
        }
    }
}
```

Se sempre usamos o mesmo root, podemos definilo dentro bloco server, para evitar repeticoes

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        root /etc/nginx/www;

        location / {
            index index.html;
        }

        location /web1 {
            index index.html;
            try_files $uri /web1/index.html;
        }

        location /web2 {
            index index.html;
            try_files $uri /web2/index.html;
        }
    }
}
```

## Referências

[Material Oficial Usado como Base](https://nginx.org/en/docs/beginners_guide.html)
