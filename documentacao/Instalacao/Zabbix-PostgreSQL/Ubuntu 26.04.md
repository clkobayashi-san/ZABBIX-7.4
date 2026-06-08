# 📘 Guia Completo de Instalação e Configuração do Zabbix 7.4 no Ubuntu 26.04

Este guia detalha passo a passo a instala  o do **Zabbix Server 7.4** no **Ubuntu 26.04 LTS**, utilizando **PostgreSQL 15**, **Apache2**, **PHP 8.5** e incluindo o Frontend e Agent2 do Zabbix.

---

## 📌 Visão Geral

| Componente | Versão |
|---|---|
| Sistema Operacional | Ubuntu 26.04 LTS (codename: *resolute*) |
| Banco de Dados | PostgreSQL 15 |
| Servidor Web | Apache2 |
| PHP | 8.5 |
| Componentes Zabbix | Server, Frontend, Agent2 |

> ⚠️ **Dica importante:** Antes de instalar o Zabbix,   essencial conhecer sua vers o do Ubuntu. O reposit rio e os pacotes do Zabbix dependem diretamente da versão do sistema. Instalar a versão errada pode gerar conflitos e erros de compatibilidade.

---

## ⚙️ 1. Pré-requisitos

### Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
```

> Atualizar o Ubuntu garante que todos os pacotes estejam na versão mais recente, prevenindo conflitos durante a instalação.

### Verificar versões instaladas

```bash
pg_lsclusters
lsb_release -a
```

> Certifique-se de que o PostgreSQL e a versão do Ubuntu são compatíveis com o Zabbix 7.4.

---

## 📦 2. Instalar o repositório oficial do Zabbix

### Baixar pacote oficial

```bash
# wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu26.04_all.deb

```

### Instalar repositório

```bash
sudo dpkg -i zabbix-release_latest_7.4+ubuntu26.04_all.deb
sudo apt update
```

> O repositório oficial garante que você instale a versão correta do Zabbix para sua distribuição, com atualizações de segurança automáticas.

---

## 🧩 3. Instalar Zabbix Server, Frontend e Agent2

```bash
sudo apt install zabbix-server-pgsql zabbix-frontend-php php8.5-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent2 -y
```

> Isso instala o servidor Zabbix, o frontend (interface web) e o agente que coleta dados das máquinas monitoradas.

---

## 🐬 4. Configurar PostgreSQL (Banco de dados do Zabbix)

### Acessar PostgreSQL

### Criar banco de dados e usuário

```bash
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb - O zabbix zabbix
```

### Importar schema do Zabbix

```bash
zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | sudo-u zabbix psql zabbix
```

---

## ⚙️ 5. Configurar Zabbix Server

### Editar arquivo principal /etc/zabbix/zabbix_server.conf

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

### Ajustar os parâmetros do banco de dados

```ini
DBName=zabbix
DBUser=zabbix
DBPassword=password
```

> ⚠️ **Importante:** Assegure-se de que esses dados coincidem com os do PostgreSQL. Se estiver incorreto, o Zabbix não conseguirá conectar ao banco.

---

## 🚀 6. Iniciar serviços

```bash
systemctl restart zabbix-server zabbix-agent2 apache2 php8.5-fpm
systemctl enable zabbix-server zabbix-agent2 apache2 php8.5-fpm
```

> O `enable` garante que os serviços iniciem automaticamente no boot do servidor.

---

## 📊 7. Verificar status dos serviços

```bash
sudo systemctl status zabbix-server
sudo systemctl status apache2
sudo systemctl status zabbix-agent
```

> Assim você confirma que todos os serviços estão funcionando corretamente antes de acessar a interface web.

---

## 🌍 8. Acessar a interface web

Abra no navegador:

```
http://SEU_IP/zabbix
```

**Login padrão:**

| Campo | Valor |
|---|---|
| User | `Admin` |
| Password | `zabbix` |

> 🔐 Após o primeiro login, é altamente recomendado **alterar a senha padrão** para garantir a segurança do servidor.
