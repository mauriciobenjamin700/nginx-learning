# 🛡️ Checklist de Firewall no Linux (UFW)

## 1. Instalação e ativação

- Instalar UFW (se não estiver presente):
  ```bash
  sudo apt install ufw
  ```
- Ativar o firewall:
  ```bash
  sudo ufw enable
  ```

---

## 2. Definir políticas padrão

- Bloquear todas as conexões de entrada:
  ```bash
  sudo ufw default deny incoming
  ```
- Permitir todas as conexões de saída:
  ```bash
  sudo ufw default allow outgoing
  ```

---

## 3. Liberar portas essenciais

- SSH (para acesso remoto):
  ```bash
  sudo ufw allow 22/tcp
  ```
- HTTP (sites sem HTTPS):
  ```bash
  sudo ufw allow 80/tcp
  ```
- HTTPS (sites seguros):
  ```bash
  sudo ufw allow 443/tcp
  ```

---

## 4. Verificação do status

- Conferir se está ativo e quais regras estão aplicadas:
  ```bash
  sudo ufw status verbose
  ```

---

## 5. Testes práticos

- Escanear externamente com **nmap**:
  ```bash
  nmap -Pn SEU_IP
  ```
  → Deve mostrar apenas as portas liberadas (22, 80, 443).  
- Conferir serviços em escuta:
  ```bash
  sudo ss -tuln
  ```

---

## 6. Reforços adicionais

- **Restringir SSH** a IPs específicos (opcional):
  ```bash
  sudo ufw allow from SEU_IP to any port 22
  sudo ufw delete allow 22/tcp
  ```
- **Ativar logging** para monitorar tentativas de acesso:
  ```bash
  sudo ufw logging medium
  ```
- Instalar **fail2ban** para bloquear ataques de força bruta:
  ```bash
  sudo apt install fail2ban
  ```

---

## ✅ Resultado final
- Firewall ativo e iniciando junto com o sistema.  
- Políticas padrão seguras: **deny incoming / allow outgoing**.  
- Apenas portas necessárias (22, 80, 443) liberadas.  
- Testes externos confirmaram que o firewall bloqueia todas as outras portas.  

---
