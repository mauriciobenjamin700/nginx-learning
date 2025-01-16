# Configurando o Nginx para Server uma Aplicação Estática Usando Somente HTML, CSS e JavaScript

Requisito: Tenha o Docker instalado em sua maquina

## 1. Preparando o Projetoo

Isole todos os arquivos importantes do seu projeto HTML, CSS e JavaScript em uma pasta, neste caso usaremos o nome de `dist`

exemplo de estrutura:

```bash
dist/
    index.html
    styles.css
    scripts.js
```

## 2. Configurando o Nginx para Servir Sua Aplicação Via Dockerfile

Crie um arquivo `dockerfile` ao lado da sua pasta `dist`

```bash
dist/
    index.html
    styles.css
    scripts.js
dockerfile
```

Insira o seguinte conteúdo no seu arquivo `dockerfile`

```dockerfile
FROM nginx
COPY ./dist/ /usr/share/nginx/html
```

## 3. Configurando o Docker Compose para Mapear as Portas e a Rede

Crie um arquivo docker-compose.yaml ao lado do seu `dockerfile`

```bash
dist/
    index.html
    styles.css
    scripts.js
docker-compose.yaml
dockerfile
```

Insira o seguinte conteúdo no seu arquivo `docker-compose.yaml`

```docker-compose.yaml
services:
  web:
    build: 
      context: .
      dockerfile: dockerfile
    ports:
      - "8080:80"
```

## 4. Build

Use `docker compose up --build -d` para subir sua aplicação. Agora basta acessar usando [localhost:8080](http://localhost:8080)
