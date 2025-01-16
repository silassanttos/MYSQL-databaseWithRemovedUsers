# Recuperação de Usuários e Permissões no MySQL com WampServer

Este guia detalha os passos para recuperar usuários e permissões no MySQL instalado via WampServer, utilizando a versão 5.5.28 como exemplo.
Após um incidente os usuários foram removidos do servidor de banco de dados, Segue passo a passo para normalização.

---

## 1. Verificar a Instalação do MySQL
Certifique-se de que o MySQL está instalado e configurado no WampServer.

1. Abra o prompt de comando.
2. Navegue até o diretório `bin` do MySQL:
   ```
   cd C:\wamp\bin\mysql\mysql5.5.28\bin
   ```
3. Teste o acesso ao MySQL:
   ```
   mysql -u root
   ```
   Se você ver o erro "'mysql' não é reconhecido", navegue para o diretório correto antes de executar o comando acima.
   Se conseguir acessar o prompt do MySQL (mysql>), o servidor está rodando no modo de segurança.
---

## 2. Iniciar o MySQL no Modo de Segurança
Se você perdeu o acesso ou os usuários foram deletados, inicie o MySQL em modo de segurança:

1. Pare os serviços do WampServer:
   - Clique no ícone do WampServer na bandeja do sistema.
   - Escolha "Stop All Services".

2. Execute o comando no prompt:
   ```
   mysqld --skip-grant-tables
   ```
   O terminal ficará travado, indicando que o MySQL está rodando no modo de segurança.

3. Abra outro prompt de comando e conecte-se ao MySQL:
   ```
   cd C:\wamp\bin\mysql\mysql5.5.28\bin
   mysql -u root
   ```

---

## 3. Verificar e Configurar Usuários

### 3.1 Listar Usuários
Para visualizar os usuários existentes:
```sql
SELECT User, Host FROM mysql.user;
```

Saída esperada:
```
+--------------+-----------+
| User         | Host      |
+--------------+-----------+
| ADM          | %         |
| root         | %         |
| silas.santos | %         |
|              | localhost |
+--------------+-----------+
```

### 3.2 Remover Usuários Inválidos
Se houver um usuário anônimo (vazio), remova-o:
```sql
DELETE FROM mysql.user WHERE User='' AND Host='localhost';
FLUSH PRIVILEGES;
```

### 3.3 Redefinir ou Criar Usuários
#### Atualizar a Senha do Usuário `root`
```sql
UPDATE mysql.user SET Password=PASSWORD('nova_senha') WHERE User='root';
FLUSH PRIVILEGES;
```
Substitua `nova_senha` pela senha desejada.

#### Criar um Novo Usuário Administrativo (Opcional)
```sql
CREATE USER 'novo_admin'@'localhost' IDENTIFIED BY 'sua_senha';
GRANT ALL PRIVILEGES ON *.* TO 'novo_admin'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
Substitua `sua_senha` pela senha desejada.

### 3.4 Verificar Permissões
Para conferir as permissões do usuário `root`:
```sql
SHOW GRANTS FOR 'root'@'%';
```
Saída esperada:
```
+--------------------------------------------------------------------------------------------------------------------------------+
| Grants for root@%                                                                                                              |
+--------------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY PASSWORD '*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B' WITH GRANT OPTION |
+--------------------------------------------------------------------------------------------------------------------------------+
```

---

## 4. Reiniciar o MySQL em Modo Normal
Depois de corrigir os problemas de permissões:

1. Pare o processo rodando no modo de segurança pressionando `Ctrl + C` no terminal onde `mysqld --skip-grant-tables` foi executado.

2. Reinicie o WampServer:
   - Clique no ícone do WampServer.
   - Escolha "Start All Services".

3. Teste o acesso ao MySQL:
   ```
   mysql -u root -p
   ```
   Insira a senha configurada.

---

## 5. Conferir Bases de Dados
Liste as bases de dados para garantir que tudo está acessível:
```sql
SHOW DATABASES;
```

---

## 6. Diagnóstico Adicional (Se Necessário)

### Verificar Porta do MySQL
Confirme que o MySQL está ouvindo na porta 3306:
```bash
netstat -an | find "3306"
```

### Verificar Logs de Erro
Caso algo não funcione, verifique o arquivo de log de erros:
```
C:\wamp\bin\mysql\mysql5.5.28\data\<nome_do_servidor>.err
```

---

Seguindo esses passos, você deve conseguir restaurar o acesso ao MySQL e corrigir problemas relacionados a usuários e permissões. Documente suas modificações no GitHub para referência futura!

