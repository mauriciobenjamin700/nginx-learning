# Aprendendo Sobre NGINX

O `NGINX` é um servidor web com suporte a balanço de carga e proxy/proxy-reverso extremamente útil e usado no mercado.

Este repositório busca juntar informações em português de maneira simplificada para te ajudar a aprender mais sobre o `NGINX`.

As seções a baixo irão fornecer suporte com relação a materiais adicionais.

## Materiais Adicionais

- [Configurando um Servidor Nginx para Servir Localmente Sua Aplicação HTML, CSS e JS](./docs/custom-live-server.md)
- [Configurando NGINX com Docker para Servir uma Aplicação Web com Certificado SSL](./ssl/)
- [Configurando Servidor NGINX para Fornecer Certificado SSL para Todas as Aplicações Rodando na Maquina de Forma Gratuita](./docs/ssl-servidor-manual.md)
- [Configurando servidor NGINX para servir duas aplicações web ao mesmo tempo](./docs/many-sites-with-nginx.md)

## Documentação Oficial

[Nginx + Docker](https://hub.docker.com/_/nginx)
[Nginx for Beginners](https://nginx.org/en/docs/beginners_guide.html#conf_structure)
[Nginx the Doc](https://nginx.org/en/docs/)

## Guia: Como Resolver o Problema de Rotas no React com Nginx

Quando você está desenvolvendo uma aplicação React com React Router e a hospeda em um servidor Nginx, é comum enfrentar o problema de rotas quebrando ao atualizar a página ou ao acessar uma rota diretamente. Este guia explica por que isso acontece e como resolver o problema.

---

### **Por que isso acontece?**

O React Router é uma biblioteca que gerencia a navegação no lado do cliente (client-side routing). Isso significa que o roteamento é feito pelo JavaScript no navegador, sem a necessidade de recarregar a página ou fazer requisições ao servidor para cada rota.

No entanto, quando você atualiza a página ou acessa uma rota diretamente (por exemplo, `https://seusite.com/sobre`), o navegador faz uma requisição ao servidor para obter o recurso correspondente a essa rota. Se o servidor não estiver configurado corretamente, ele não encontrará o recurso (pois a rota existe apenas no lado do cliente) e retornará um erro **404 (Not Found)**.

---

## **Solução: Configuração do Nginx**

Para resolver esse problema, você precisa configurar o Nginx para redirecionar todas as requisições para o arquivo `index.html`, que é o ponto de entrada da aplicação React. Dessa forma, o React Router pode assumir o controle da navegação no lado do cliente.

Aqui está um exemplo de configuração do Nginx para lidar com rotas do React:

### **Configuração do Nginx**

```nginx
server {
    listen 443 ssl;
    server_name seusite.com www.seusite.com;

    ssl_certificate /etc/letsencrypt/live/seusite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/seusite.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    root /var/www/seusite.com;
    index index.html index.htm;

    location / {
        try_files $uri /index.html;
    }

    location ~* \.(css|js|jpg|jpeg|png)$ {
        expires 6h;
        log_not_found off;
    }
}
```

### **Explicação da Configuração**

1. **`root /var/www/seusite.com;`**:
   - Define o diretório raiz onde os arquivos da aplicação React estão hospedados.

2. **`index index.html index.htm;`**:
   - Define o arquivo de índice padrão como `index.html`.

3. **`try_files $uri /index.html;`**:
   - Tenta servir o arquivo correspondente à URL solicitada (`$uri`). Se o arquivo não for encontrado, serve o `index.html`.
   - Isso é essencial para o React Router, pois todas as rotas do lado do cliente devem ser redirecionadas para o `index.html`.

4. **`location ~* \.(css|js|jpg|jpeg|png)$`**:
   - Configura o cache para arquivos estáticos (CSS, JS, imagens), melhorando o desempenho da aplicação.

---

## **Passos para Implementar a Solução**

1. **Acesse o arquivo de configuração do Nginx**:
   - O arquivo de configuração geralmente está localizado em `/etc/nginx/sites-available/seu-site.conf`.

2. **Edite o arquivo de configuração**:
   - Adicione ou atualize o bloco `server` conforme mostrado acima.

3. **Reinicie o Nginx**:
   - Após salvar as alterações, reinicie o Nginx para aplicar as mudanças:
     ```bash
     sudo systemctl restart nginx
     ```

4. **Teste a aplicação**:
   - Acesse sua aplicação React em `https://seusite.com`.
   - Navegue entre as rotas e atualize a página para verificar se o problema foi resolvido.

---

## **Considerações Adicionais**

### **1. Permissões de Arquivos**
Certifique-se de que o Nginx tem permissão para ler os arquivos no diretório raiz da aplicação. Use os seguintes comandos para ajustar as permissões:

```bash
sudo chown -R www-data:www-data /var/www/seusite.com
sudo chmod -R 755 /var/www/seusite.com
```

### **2. Build da Aplicação React**
Antes de hospedar a aplicação, certifique-se de que o build foi gerado corretamente:

```bash
npm run build
# ou
yarn build
```

Os arquivos gerados devem ser colocados no diretório raiz configurado no Nginx (por exemplo, `/var/www/seusite.com`).

### **3. Verificação de Logs**
Se o problema persistir, verifique os logs do Nginx para obter mais detalhes:

```bash
sudo tail -f /var/log/nginx/error.log
```

---

## **Conclusão**

Com a configuração correta do Nginx, você pode garantir que sua aplicação React com React Router funcione perfeitamente, mesmo ao atualizar a página ou acessar rotas diretamente. A chave é redirecionar todas as requisições para o `index.html`, permitindo que o React Router assuma o controle da navegação no lado do cliente.

Se você seguir este guia, seu problema deve ser resolvido. Caso precise de mais ajuda, consulte a [documentação oficial do Nginx](https://nginx.org/en/docs/) ou deixe um comentário abaixo! 😊

--- 

**Exemplo de Markdown:**

```markdown
# Guia: Como Resolver o Problema de Rotas no React com Nginx

## **Por que isso acontece?**

O React Router é uma biblioteca que gerencia a navegação no lado do cliente...

## **Solução: Configuração do Nginx**

### **Configuração do Nginx**

```nginx
server {
    listen 443 ssl;
    server_name seusite.com www.seusite.com;

    ssl_certificate /etc/letsencrypt/live/seusite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/seusite.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    root /var/www/seusite.com;
    index index.html index.htm;

    location / {
        try_files $uri /index.html;
    }

    location ~* \.(css|js|jpg|jpeg|png)$ {
        expires 6h;
        log_not_found off;
    }
}
```

### **Explicação da Configuração**

1. **`root /var/www/seusite.com;`**:
   - Define o diretório raiz onde os arqu
