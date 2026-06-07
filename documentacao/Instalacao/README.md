# ?? Guia Completo de Instala  o e Configura  o do Zabbix 7.4 no Ubuntu 26.04

Este guia detalha passo a passo a instala  o do **Zabbix Server 7.4** no **Ubuntu 26.04 LTS**, utilizando **MySQL 8.4.9**, **Apache2**, **PHP 8.5** e incluindo o Frontend e Agent do Zabbix.

---

## ?? Vis o Geral

| Componente | Vers o |
|---|---|
| Sistema Operacional | Ubuntu 26.04 LTS (codename: *resolute*) |
| Banco de Dados | MySQL 8.4.9 |
| Servidor Web | Apache2 |
| PHP | 8.5 |
| Componentes Zabbix | Server, Frontend, Agent |

> ?? **Dica importante:** Antes de instalar o Zabbix,   essencial conhecer sua vers o do Ubuntu. O reposit rio e os pacotes do Zabbix dependem diretamente da vers o do sistema. Instalar a vers o errada pode gerar conflitos e erros de compatibilidade.

---

## ?? 1. Pr -requisitos

### Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
```

> Atualizar o Ubuntu garante que todos os pacotes estejam na vers o mais recente, prevenindo conflitos durante a instala  o.

### Verificar vers es instaladas

```bash
mysql --version
lsb_release -a
```

> Certifique-se de que o MySQL e a vers o do Ubuntu s o compat veis com o Zabbix 7.4.

---

## ?? 2. Instalar o reposit rio oficial do Zabbix

### Baixar pacote oficial

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu26.04_all.deb
```

### Instalar reposit rio

```bash
sudo dpkg -i zabbix-release_latest_7.4+ubuntu26.04_all.deb
sudo apt update
```

> O reposit rio oficial garante que voc  instale a vers o correta do Zabbix para sua distribui  o, com atualiza  es de seguran a autom ticas.

---

## ?? 3. Instalar Zabbix Server, Frontend e Agent

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

> Isso instala o servidor Zabbix, o frontend (interface web) e o agente que coleta dados das m quinas monitoradas.

---

## ?? 4. Configurar MySQL (Banco de dados do Zabbix)

### Acessar MySQL

```bash
sudo mysql -uroot -p
```

### Criar banco de dados e usu rio

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
FLUSH PRIVILEGES;
EXIT;
```

>   importante usar `utf8mb4` para evitar problemas com caracteres especiais na interface do Zabbix.
>
> O comando `log_bin_trust_function_creators`   necess rio para importar fun  es do schema do Zabbix sem erros.

### Importar schema do Zabbix

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

### Restaurar configura  o de seguran a

```bash
sudo mysql -uroot -p
```

```sql
SET GLOBAL log_bin_trust_function_creators = 0;
EXIT;
```

---

## ?? 5. Configurar Zabbix Server

### Editar arquivo principal

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

### Ajustar os par metros do banco de dados

```ini
DBName=zabbix
DBUser=zabbix
DBPassword=password
```

> ?? **Importante:** Assegure-se de que esses dados coincidem com os do MySQL. Se estiver incorreto, o Zabbix n o conseguir  conectar ao banco.

---

## ?? 6. Configurar Apache + PHP

### Ativar m dulos necess rios

```bash
sudo a2enmod proxy
sudo a2enmod proxy_fcgi
```

### Reiniciar Apache

```bash
sudo systemctl restart apache2
```

### Ativar configura  o do Zabbix

```bash
sudo a2enconf zabbix-frontend-php
```

> Caso o comando acima n o funcione, crie um symlink manual:

```bash
sudo ln -s /etc/zabbix/apache.conf /etc/apache2/conf-enabled/zabbix.conf
```

### Reiniciar Apache ap s ativar configura  o

```bash
sudo systemctl restart apache2
```

> Isso garante que o frontend do Zabbix seja servido corretamente pelo Apache.

---

## ?? 7. Iniciar servi os

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2 php8.5-fpm
sudo systemctl enable zabbix-server zabbix-agent apache2 php8.5-fpm
```

> O `enable` garante que os servi os iniciem automaticamente no boot do servidor.

---

## ?? 8. Verificar status dos servi os

```bash
sudo systemctl status zabbix-server
sudo systemctl status apache2
sudo systemctl status zabbix-agent
```

> Assim voc  confirma que todos os servi os est o funcionando corretamente antes de acessar a interface web.

---

## ?? 9. Acessar a interface web

Abra no navegador:

```
http://SEU_IP/zabbix
```

**Login padr o:**

| Campo | Valor |
|---|---|
| User | `Admin` |
| Password | `zabbix` |

> ?? Ap s o primeiro login,   altamente recomendado **alterar a senha padr o** para garantir a seguran a do servidor.