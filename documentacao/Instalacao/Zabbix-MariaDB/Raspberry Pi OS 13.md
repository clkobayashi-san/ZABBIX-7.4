# 📘 Guia Completo de Instalação e Configuração do Zabbix 7.4 no Raspberry Pi OS 13

Este guia detalha passo a passo a instala  o do **Zabbix Server 7.4** no **
Raspberry Pi OS 13**, utilizando **MariaDB 11.8.6**, **Apache2**, **PHP 8.4.21** e incluindo o Frontend e Agent2 do Zabbix.

---

## 📌 Visão Geral

| Componente | Versão |
|---|---|
| Sistema Operacional | Raspberry Pi OS (codename: *trixie*) |
| Banco de Dados | MariaDB 11.8.6 |
| Servidor Web | Apache2 |
| PHP | 8.4.21 |
| Componentes Zabbix | Server, Frontend, Agent2 |

> ⚠️ **Dica importante:** Antes de instalar o Zabbix,   essencial conhecer sua vers o do Debian. O reposit rio e os pacotes do Zabbix dependem diretamente da versão do sistema. Instalar a versão errada pode gerar conflitos e erros de compatibilidade.

---

## ⚙️ 1. Pré-requisitos

### Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
```

> Atualizar o Raspberry Pi OS garante que todos os pacotes estejam na versão mais recente, prevenindo conflitos durante a instalação.

### Verificar versões instaladas

```bash
mariadb --version
lsb_release -a
```

> Certifique-se de que o MariaDB e a versão do Debian são compatíveis com o Zabbix 7.4.

---

## 📦 2. Instalar o repositório oficial do Zabbix

### Baixar pacote oficial

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/raspbian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
```

### Instalar repositório

```bash
sudo dpkg -i zabbix-release_latest_7.4+debian13_all.deb
sudo apt update -y
```

> O repositório oficial garante que você instale a versão correta do Zabbix para sua distribuição, com atualizações de segurança automáticas.

---

## 🧩 3. Instalar Zabbix Server, Frontend e Agent2

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent2 -y
```

> Isso instala o servidor Zabbix, o frontend (interface web) e o agente que coleta dados das máquinas monitoradas.

---

## 🐬 4. Configurar MariaDB (Banco de dados do Zabbix)

### Acessar MariaDB

```bash
sudo mariadb -uroot -p
```

### Criar banco de dados e usuário

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
FLUSH PRIVILEGES;
EXIT;
```

> É importante usar `utf8mb4` para evitar problemas com caracteres especiais na interface do Zabbix.
>
> O comando `log_bin_trust_function_creators` é necessário para importar funções do schema do Zabbix sem erros.

### Importar schema do Zabbix

```bash
zcat /usr/share/zabbix/sql-scripts/MariaDB/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

### Restaurar configuração de segurança

```bash
sudo mariadb -uroot -p
```

```sql
SET GLOBAL log_bin_trust_function_creators = 0;
EXIT;
```

---

## ⚙️ 5. Configurar Zabbix Server

### Editar arquivo principal

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

### Ajustar os parâmetros do banco de dados

```ini
DBName=zabbix
DBUser=zabbix
DBPassword=password
```

> ⚠️ **Importante:** Assegure-se de que esses dados coincidem com os do MariaDB. Se estiver incorreto, o Zabbix não conseguirá conectar ao banco.

---

## 🌐 6. Configurar Apache

### Reiniciar Apache

```bash
sudo systemctl restart apache2
```

> Isso garante que o frontend do Zabbix seja servido corretamente pelo Apache2.

---

## 🚀 7. Iniciar serviços

```bash
sudo systemctl restart zabbix-server zabbix-agent2 apache2 
sudo systemctl enable zabbix-server zabbix-agent2 apache2 
```

> O `enable` garante que os serviços iniciem automaticamente no boot do servidor.

---

## 📊 8. Verificar status dos serviços

```bash
sudo systemctl status zabbix-server
sudo systemctl status apache2
sudo systemctl status zabbix-agent2
```

> Assim você confirma que todos os serviços estão funcionando corretamente antes de acessar a interface web.

---

## 🌍 9. Acessar a interface web

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
