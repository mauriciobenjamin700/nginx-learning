# Guia de Configuração de Proxy Reverso com Nginx

## Índice

1. [O que é Proxy Reverso?](#o-que-é-proxy-reverso)
2. [Pré-requisitos](#pré-requisitos)
3. [Estrutura Recomendada de Configuração](#estrutura-recomendada-de-configuração)
4. [Configuração Passo a Passo](#configuração-passo-a-passo)
5. [Explicação Detalhada dos Parâmetros](#explicação-detalhada-dos-parâmetros)
6. [Testando a Configuração](#testando-a-configuração)
7. [Casos de Uso Avançados](#casos-de-uso-avançados)
8. [Solução de Problemas](#solução-de-problemas)

## O que é Proxy Reverso?

Um proxy reverso é um servidor que fica entre os clientes e os servidores de backend. Ele recebe as solicitações dos clientes e as encaminha para um ou mais servidores backend, retornando a resposta do servidor para o cliente.

### Vantagens do Proxy Reverso

- **Balanceamento de Carga**: Distribui requisições entre múltiplos servidores
- **Cache**: Armazena respostas para melhorar performance
- **SSL Termination**: Gerencia certificados SSL/TLS
- **Compressão**: Reduz o tamanho das respostas
- **Segurança**: Oculta detalhes dos servidores backend

## Pré-requisitos

1. **Nginx instalado** no servidor
2. **Serviços backend** rodando (ex: aplicação frontend na porta 3000, API na porta 8000)
3. **Acesso de administrador** para editar configurações
4. **Conhecimento básico** de linha de comando

### Instalação do Nginx (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install nginx
```

### Instalação do Nginx (CentOS/RHEL)

```bash
sudo yum install nginx
# ou para versões mais recentes:
sudo dnf install nginx
```

## Estrutura Recomendada de Configuração

### Arquitetura de Arquivos

```
/etc/nginx/
├── nginx.conf                 # Configuração principal (MÍNIMA)
├── sites-available/           # Configurações de sites disponíveis
│   ├── meudominio.com.conf
│   └── api.meudominio.com.conf
├── sites-enabled/             # Links simbólicos para sites ativos
│   ├── meudominio.com.conf -> ../sites-available/meudominio.com.conf
│   └── api.meudominio.com.conf -> ../sites-available/api.meudominio.com.conf
└── conf.d/                    # Configurações adicionais
```

### Filosofia da Estrutura

- **`nginx.conf`**: Apenas configurações globais básicas
- **`sites-available/`**: Todas as configurações de sites (repositório)
- **`sites-enabled/`**: Links simbólicos apenas para sites ativos
- **Vantagens**: Organização, facilidade de ativar/desativar sites, backup simples

## Configuração Passo a Passo

### Passo 1: Verificar o nginx.conf Principal

O arquivo `/etc/nginx/nginx.conf` deve permanecer **MÍNIMO** e limpo:

```bash
sudo nano /etc/nginx/nginx.conf
```

**Mantenha apenas as configurações globais:**

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    # Configurações básicas
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # SSL Settings
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    
    # Logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    # Gzip
    gzip on;
    
    # INCLUIR SITES (ESSENCIAL)
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### Passo 2: Criar Configuração de Site Individual

Crie um arquivo para cada domínio/aplicação:

```bash
sudo nano /etc/nginx/sites-available/meudominio.com.conf
```

### Passo 3: Configuração de Proxy Reverso para Frontend

**Exemplo 1: Frontend (React/Vue/Angular)**

```nginx
# Configuração de upstream (opcional mas recomendado)
upstream frontend_server {
    server 0.0.0.0:3000;
    # server 127.0.0.1:3000;  # Alternativa mais segura
}

server {
    listen 80;
    server_name meudominio.com www.meudominio.com;
    
    # Proxy para Frontend
    location / {
        proxy_pass http://frontend_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (para apps que precisam)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Passo 4: Configuração para API Backend

**Exemplo 2: API Backend**

```bash
sudo nano /etc/nginx/sites-available/api.meudominio.com.conf
```

```nginx
# Upstream para API
upstream api_server {
    server 0.0.0.0:8000;
}

server {
    listen 80;
    server_name api.meudominio.com;
    
    # Configurações específicas para API
    keepalive_timeout 5;
    client_max_body_size 4G;
    
    # Proxy para API
    location / {
        proxy_pass http://api_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts para API
        proxy_connect_timeout 120s;
        proxy_read_timeout 120s;
        proxy_send_timeout 120s;
    }
    
    # WebSocket específico para API
    location /api/v1/websocket/ {
        proxy_pass http://api_server;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Passo 5: Ativar os Sites

```bash
# Criar links simbólicos para ativar os sites
sudo ln -s /etc/nginx/sites-available/meudominio.com.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/api.meudominio.com.conf /etc/nginx/sites-enabled/

# Verificar links criados
ls -la /etc/nginx/sites-enabled/
```

### Passo 6: Testar e Aplicar

```bash
# Testar configuração
sudo nginx -t

# Se passou no teste, recarregar
sudo systemctl reload nginx

# Verificar status
sudo systemctl status nginx
```

## Explicação Detalhada dos Parâmetros

### Bloco `upstream`

```nginx
upstream nome_servidor {
    server IP:PORTA;
    # server IP:PORTA backup;  # Servidor de backup
    # server IP:PORTA weight=3; # Peso para balanceamento
}
```

**Vantagens do upstream:**

- Facilita balanceamento de carga
- Permite configurações avançadas
- Melhor organização do código

### Configurações de Proxy Essenciais

| Parâmetro | Função | Exemplo |
|-----------|--------|---------|
| `proxy_pass` | URL do backend | `http://backend:8000` |
| `proxy_set_header Host $host` | Preserva host original | `meusite.com` |
| `proxy_set_header X-Real-IP $remote_addr` | IP real do cliente | `192.168.1.100` |
| `proxy_set_header X-Forwarded-For` | Cadeia de proxies | `192.168.1.100, 10.0.0.1` |
| `proxy_set_header X-Forwarded-Proto` | Protocolo original | `https` |

### Configurações de Performance

| Parâmetro | Descrição | Valor Recomendado |
|-----------|-----------|-------------------|
| `keepalive_timeout` | Tempo de conexão ativa | `5s` (API), `65s` (Web) |
| `client_max_body_size` | Tamanho máximo do upload | `4G` (API), `1M` (Web) |
| `proxy_connect_timeout` | Timeout de conexão | `120s` |
| `proxy_read_timeout` | Timeout de leitura | `120s` |
| `proxy_send_timeout` | Timeout de envio | `120s` |

## Testando a Configuração

### 1. Comandos de Verificação

```bash
# Testar sintaxe
sudo nginx -t

# Verificar sites ativos
ls -la /etc/nginx/sites-enabled/

# Status do nginx
sudo systemctl status nginx

# Verificar portas em uso
sudo netstat -tlnp | grep nginx
```

### 2. Testes de Conectividade

```bash
# Testar frontend
curl -I http://meudominio.com

# Testar API
curl -I http://api.meudominio.com

# Testar com headers específicos
curl -H "Host: meudominio.com" http://localhost
```

### 3. Monitoramento de Logs

```bash
# Logs em tempo real
sudo tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Filtrar por domínio específico
sudo grep "meudominio.com" /var/log/nginx/access.log
```

## Casos de Uso Avançados

### 1. Site com Frontend + API no Mesmo Domínio

```nginx
# /etc/nginx/sites-available/meudominio.com.conf

upstream frontend_server {
    server 0.0.0.0:3000;
}

upstream api_server {
    server 0.0.0.0:8000;
}

server {
    listen 80;
    server_name meudominio.com;
    
    # Frontend principal
    location / {
        proxy_pass http://frontend_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # API no mesmo domínio
    location /api/ {
        # Remove prefixo /api antes de enviar para backend
        rewrite ^/api(/.*)$ $1 break;
        
        proxy_pass http://api_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts para API
        proxy_connect_timeout 120s;
        proxy_read_timeout 120s;
        proxy_send_timeout 120s;
    }
    
    # WebSocket específico
    location /api/websocket/ {
        rewrite ^/api(/.*)$ $1 break;
        
        proxy_pass http://api_server;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### 2. Configuração com SSL (HTTPS)

```nginx
# /etc/nginx/sites-available/meudominio.com.conf

upstream app_server {
    server 0.0.0.0:3000;
}

# Redirecionamento HTTP -> HTTPS
server {
    listen 80;
    server_name meudominio.com www.meudominio.com;
    
    # Redirecionamento automático para HTTPS
    if ($host = meudominio.com) {
        return 301 https://$host$request_uri;
    }
    return 404;
}

# Servidor HTTPS
server {
    listen 443 ssl http2;
    server_name meudominio.com www.meudominio.com;
    
    # Certificados SSL (Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/meudominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/meudominio.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    
    # Configurações do proxy
    location / {
        proxy_pass http://app_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 3. Balanceamento de Carga

```nginx
# /etc/nginx/sites-available/meudominio.com.conf

upstream app_cluster {
    # Diferentes estratégias de balanceamento
    # least_conn;           # Menos conexões
    # ip_hash;              # Por IP do cliente
    # fair;                 # Por tempo de resposta
    
    server 10.0.0.1:3000 weight=3;
    server 10.0.0.2:3000 weight=2;
    server 10.0.0.3:3000 weight=1;
    server 10.0.0.4:3000 backup;
}

server {
    listen 80;
    server_name meudominio.com;
    
    location / {
        proxy_pass http://app_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Gerenciamento de Sites

### Comandos Úteis para Gerenciar Sites

```bash
# Listar sites disponíveis
ls /etc/nginx/sites-available/

# Listar sites ativos
ls /etc/nginx/sites-enabled/

# Ativar um site
sudo ln -s /etc/nginx/sites-available/meusite.conf /etc/nginx/sites-enabled/

# Desativar um site
sudo rm /etc/nginx/sites-enabled/meusite.conf

# Testar configuração após mudanças
sudo nginx -t

# Recarregar nginx
sudo systemctl reload nginx
```

### Backup de Configurações

```bash
# Backup de um site específico
sudo cp /etc/nginx/sites-available/meusite.conf /etc/nginx/sites-available/meusite.conf.backup

# Backup de todos os sites
sudo tar -czf nginx-sites-backup-$(date +%Y%m%d).tar.gz /etc/nginx/sites-available/

# Backup completo do nginx
sudo tar -czf nginx-complete-backup-$(date +%Y%m%d).tar.gz /etc/nginx/
```

## Solução de Problemas

### Problemas Comuns

#### 1. Site não carrega após ativação

```bash
# Verificar se o link simbólico foi criado
ls -la /etc/nginx/sites-enabled/

# Verificar se a configuração está correta
sudo nginx -t

# Verificar se o backend está rodando
sudo netstat -tlnp | grep :3000
```

#### 2. Erro 502 Bad Gateway

**Causas mais comuns:**

- Backend não está rodando
- Firewall bloqueando conexão
- IP/porta incorretos no upstream

**Soluções:**

```bash
# Verificar se o serviço backend está ativo
sudo systemctl status meu-app

# Testar conectividade direta com o backend
curl http://localhost:3000

# Verificar logs de erro do nginx
sudo tail -f /var/log/nginx/error.log
```

#### 3. Headers não chegam no backend

**Problema:** Backend não recebe informações do cliente original

**Solução:** Verificar headers de proxy

```nginx
location / {
    proxy_pass http://backend;
    
    # Headers essenciais
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # Para preservar porta original
    proxy_set_header X-Forwarded-Port $server_port;
}
```

#### 4. WebSocket não funciona

**Solução:** Configurações específicas para WebSocket

```nginx
location /websocket/ {
    proxy_pass http://backend;
    
    # Configurações WebSocket
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    
    # Timeouts maiores para WebSocket
    proxy_read_timeout 86400;
}
```

### Comandos de Debug

```bash
# Verificar configuração completa
sudo nginx -T

# Testar configuração específica
sudo nginx -t -c /etc/nginx/sites-available/meusite.conf

# Verificar processos nginx
ps aux | grep nginx

# Monitorar conexões
sudo ss -tulpn | grep nginx

# Testar DNS
nslookup meudominio.com

# Verificar certificados SSL
openssl s_client -connect meudominio.com:443
```

## Monitoramento e Logs

### Configuração de Logs Personalizados

```nginx
# /etc/nginx/sites-available/meudominio.com.conf

server {
    listen 80;
    server_name meudominio.com;
    
    # Logs específicos do site
    access_log /var/log/nginx/meudominio-access.log;
    error_log /var/log/nginx/meudominio-error.log;
    
    location / {
        proxy_pass http://backend;
        # ... configurações de proxy
    }
}
```

### Análise de Logs

```bash
# Logs de acesso mais recentes
sudo tail -f /var/log/nginx/access.log

# Filtrar por código de status
sudo grep " 502 " /var/log/nginx/access.log

# Estatísticas de acesso
sudo cat /var/log/nginx/access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr

# Monitorar erros específicos
sudo grep "upstream" /var/log/nginx/error.log
```

## Conclusão

### Vantagens desta Estrutura

1. **Organização**: Cada site tem seu arquivo próprio
2. **Flexibilidade**: Fácil ativar/desativar sites
3. **Manutenção**: nginx.conf permanece limpo
4. **Backup**: Fácil fazer backup de sites específicos
5. **Colaboração**: Diferentes pessoas podem gerenciar sites diferentes

### Boas Práticas

1. **Sempre teste** antes de aplicar: `sudo nginx -t`
2. **Use upstream** para melhor organização
3. **Configure logs específicos** por site
4. **Implemente SSL** em produção
5. **Monitore logs** regularmente
6. **Faça backups** das configurações
7. **Use nomes descritivos** para arquivos
8. **Documente configurações** especiais

### Próximos Passos

1. Configurar **certificados SSL** com Let's Encrypt
2. Implementar **cache** para melhor performance
3. Configurar **rate limiting** para segurança
4. Monitorar **métricas** de performance
5. Automatizar **deploys** e **backups**

Para documentação oficial, visite: [nginx.org/en/docs/](https://nginx.org/en/docs/)