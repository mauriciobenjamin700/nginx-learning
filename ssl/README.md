# Configurações para o Servidor Web Nginx

Este Tutorial irá te orientar o passo a passo mínimo para configurar seu Servidor Web NGINX para Servir sua Aplicação JS com build na Web com Certificado SSL.

A pasta [nginx](./nginx/) contem um modelo pré-configurado para executar um servidor web nginx com as devidas certificações SSL para que sua aplicação seja servida com qualidade

O arquivo [Dockerfile](./docker/Dockerfile) tem as configurações de uma Imagem nginx rodando em um linux alpine, de forma a ser eficiente e performático para seu uso

## Passos para Colocar Sua Aplicação no Ar

### 1. Implemente em seu Projeto

Copie tanto o arquivo [Dockerfile](./docker/Dockerfile) quanto os diretórios [nginx](./nginx/) e [dist](./dist/) para a raiz do seu projeto web.

### 2. Ajustes de Configurações Para Sua Aplicação

- Configure sua aplicação JS para que quando realizar o build, salve o resultado no diretório [dist](./dist/) na raiz do projeto
- Acesse o Arquivo [app.conf](./nginx/sites-available/app.conf) e altere a linha 3 `server_name SEUS_DOMÍNIOS_AQUI;` trocando `SEUS_DOMÍNIOS_AQUI` por todos os seus domínios que deseja gerar certificados.
- obs: Os domínios devem ser separados por um único espaço
- No arquivo [docker-compose.yaml](./docker-compose.yaml) você pode alterar o caminho para os arquivos contidos no diretório [nginx](./nginx/) de acordo com sua vontade, mas nunca mexendo onde eles ficam salvos no container

### 3. Servidor VPS

Obs: Se você está configurando um servidor web, provavelmente tem uma VPS para manter a aplicação online 24/7. Logo este tutorial irá considerar que você tem isso para darmos sequência. Detalhe importante é que você já deve ter um domínio comprado também, para poder cerificá-lo!

- Acesse seu Servidor VPS via SSH

- Use o Git+GitHub para trazer seu projeto para o servidor

### 4. Preparando DNS

Para este tutorial, iremos configurar o serviço DNS usando o [Registro.br](https://registro.br), mas pode ser replicado, contato que consiga seguir os mesmos paços em sua plataforma

- Procure o painel do DNS conforme mostrado na Figura: ![01](../images/input-text-painel.png)
- Você irá precisar inserir os apontamentos para seu Servidor Conforme a Figura ![02](../images/dns-configs-to-ssl.png)
- Detalhe:
  - O item `A` em `Amarelo` deve ser respectivamente `domínio` e `endereço IPV4 do Servidor VPS que está usando`
  - O item `AAAA` em `Verde` deve ser respectivamente `domínio` e `endereço IPV6 do Servidor VPS que está usando`
- Cada domínio que você for certificar, deve estar registrado no DNS seguindo este padrão
- Pronto, agora temos o mínimo configurado.

### 5. Docker

Volte para o seu projeto no servidor VPS e inicie sua aplicação com `docker compose up -d --build`

obs: Se sua aplicação quebrou e não funciona, provavelmente é culpa sua, então concerte e prepare-se para configurar o servidor 😊!

Com a aplicação rodando, acesse o conteiner da sua aplicação web usando `docker exec -it web-server /bin/bash` lembrando de trocar `web-server` pelo ID ou Nome do container que sua aplicação está rodando.

Entrando dentro do container e acessando seu terminal, use o comando `certbot --nginx`

Informe um e-mail valido e siga o famoso `yes yes yes next next finish` e ao final, seus certificados estarão prontos, caso você tenha configurado o DNS e IPS corretamente
