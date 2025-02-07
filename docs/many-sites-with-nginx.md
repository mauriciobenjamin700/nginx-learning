# Configurando Servidor NGINX para Servir Duas Aplicações Web ao Mesmo Tempo

Neste guia, estaremos usando um servidor Linux com ubuntu 24.04 LTS

## Passo 1: Instale o NGINX

Para instalar, use estes comandos

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install nginx -y
```

Para conferir se o NGINX realmente foi instalado use:

```bash
sudo nginx -version
```

O resultado será próximo disso:

```bash
nginx version: nginx/1.24.0 (Ubuntu)
```

## Passo 2: Configurando Permissão de Inicialização do NGINX

Para que o NGINX inicie junto do sistema operacional, em caso de precisar reiniciar, use:

```bash
sudo systemctl enable nginx
```

O resultado esperado deve ser semelhante a isso:

```bash
Synchronizing state of nginx.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable nginx
```

### Alguns Comandos Básicos para Ajudar a Manejar o NGINX

- `sudo systemctl start nginx` - Inicia o NGINX
- `sudo systemctl stop nginx` - Para o NGINX
- `sudo systemctl restart nginx` - Reinicia o NGINX
- `sudo systemctl status nginx` - Verifica o Status do NGINX

O comando status retornará algo semelhante a isso:

```bash
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Fri 2025-02-07 15:42:41 UTC; 5min ago
       Docs: man:nginx(8)
   Main PID: 393065 (nginx)
      Tasks: 3 (limit: 9489)
     Memory: 2.4M (peak: 2.5M)
        CPU: 21ms
     CGroup: /system.slice/nginx.service
             ├─393065 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─3
```

Use `CONTROL + C` para sair do resultado

## Passo 3: Criando um Host Virtual para Cada Aplicação

Os Hosts Virtuais são uma estratégia do NGINX para delegar configurações especiais, que permitem habilitar um servidor web para uma aplicação específica, deixa responsável por um diretório e domínio.

Acesse o diretório de hosts virtuais, e crie um arquivo para o seu projeto. **Lembre de trocar** `SEU_DOMÍNIO_AQUI` pelo domínio que você irá usar.

```bash
sudo nano /etc/nginx/sites-available/SEU_DOMÍNIO_AQUI.conf
```

Adicie o seguinte conteúdo

```nginx
server {
        listen 80;
        listen [::]:80;

        server_name SEU_DOMÍNIO_AQUI;

        root /var/www/SEU_DOMÍNIO_AQUI;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

Teste se deu tudo certo usando:

```bash
sudo nginx -t
```

O resultado esperado dever ser:

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Caso o resultado seja o esperado, ative o Host Virtual no seu Servidor usando

```bash
sudo ln -s /etc/nginx/sites-available/SEU_DOMÍNIO.conf /etc/nginx/sites-enabled/
```

## Passo 4: Adicionando os Arquivos de Configuração do seu Site na Pasta Gerenciada pelo NGINX

Digamos que seu projeto tem a seguinte estrutura

```bash
NOME_DO_SEU_PROJETO/
    index.html
    styles.css
    scripts.js
```

Para que o nginx reconheça seu projeto, você deve ter obrigatoriamente na raiz do projeto o arquivo index.html.

Vamos copiar o seu projeto para dentro da pasta onde o nginx irá trabalhar, usando

```bash
sudo mkdir -p /var/www/SEU_DOMÍNIO
sudo cp -r CAMINHO_PARA_NOME_SEU_PROJETO/* /var/www/SEU_DOMÍNIO
```

Feito isso, reinicie o NGINX para aplicar as mudanças usando:

```bash
sudo systemctl restart nginx
```

Ao fazer uma requisição HTTP para a porta 80 do seu servidor, obterá sua página web corretamente

```http
http://alo-api.alofans.com.br/
```

## Passo 5: SSL e HTTPS

Para configurar o SSL e HTTPS, usaremos os certificados gratuitos da `Let's Encrypt` com `certbot`

Instale  certbot usando:

```bash
sudo snap install --classic certbot
```

Confira se o certbot foi instalado corretamente usando

```bash
sudo certbot --version
```

Ative o certificado SSL para todos os seus dominios usando

```bash
sudo certbot --nginx -d SEU_DOMÍNIO --agree-tos
```

Caso queira passar mais de um domínio, basta usar a flag -d várias vezes, como por exemplo: `sudo certbot --nginx -d SEU_DOMÍNIO -d DOMÍNIO2 -d DOMINIO3 --agree-tos`

## Passo 5: Configurando o Firewall

Permitindo acesso SSH

```bash
sudo ufw allow ssh
```

Ativando o Firewall

```bash
sudo ufw enable
```

Dando permissão de conexões na porta 80 (HTTP)

```bash
sudo ufw allow 80/tcp
```

Dando permissão de conexões na porta 443 (HTTPS)

```bash
sudo ufw allow 443/tcp
```

Checando se deu certo

```bash
sudo ufw status
```

```bash
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)
```

Prontinho.

## Referências Usadas

[docs.vultr.com](https://docs.vultr.com/how-to-install-nginx-web-server-on-ubuntu-24-04?ref=9141995&utm_source=performance-max-latam&utm_medium=paidmedia&obility_id=17096555207&&utm_campaign=LATAM_-_Brazil_-_Performance_Max_-_1001&utm_term=&utm_content=&ref=9141995&gad_source=1&gclid=Cj0KCQiA-5a9BhCBARIsACwMkJ4dsN0wazx1NjXka0cnBGIxBvud1ocACDXU6zRF-WIuP32aflrbt-AaAsa6EALw_wcB)
