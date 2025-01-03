FROM nginx:1.27.3-alpine-slim

# Copia o arquivo de configuração nginx.conf personalizado para o contêiner
COPY nginx/nginx.conf /etc/nginx/

# Atualiza e instala os pacotes necessários
# certbot certbot-nginx são os responsáveis pela geração de HTTPS
RUN apk update && apk upgrade && \
    apk --update add logrotate openssl bash && \
    apk add --no-cache certbot certbot-nginx

# add user www-data
RUN adduser -D -H -u 1000 -s /bin/bash www-data -G www-data

# Cria diretórios para o conteúdo do site e dá suas respectivas permissões
RUN mkdir -p /var/www && \
    chown -R www-data:www-data /var/www && \
    chmod 755 -R /var/www

# Cria diretórios para as configurações do NGINX
RUN mkdir -p /etc/nginx/sites-available /etc/nginx/conf.d && \
    chown -R www-data:www-data /etc/nginx/sites-available /etc/nginx/conf.d

# Define o diretório de trabalho para o NGINX
WORKDIR /etc/nginx

# Limpeza: Remove pacotes não utilizados para reduzir o tamanho da imagem
RUN apk del --no-cache

# Inicia o NGINX quando o contêiner é executado
CMD ["nginx", "-g", "daemon off;"]