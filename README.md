## TRANSFORMAÇÃO DE DADOS NO POWER BI COM MYSQL

---

### 1. Visão geral do projeto

Este projeto tem como objetivo criar um banco de dados Azure Company e realizar transformações de dados para análise e visualização. O escopo inclui a criação do banco, a definição de tabelas, a execução de consultas SQL e a preparação dos dados para consumo no Power BI.

Originalmente, o ambiente de banco seria provisionado no Microsoft Azure. No entanto, por limitações de acesso (ausência de conta empresarial ou estudantil), optou-se por uma solução alternativa totalmente local, utilizando MySQL executado em container Docker e gerenciado por meio do DBeaver.

### 2. Motivação para o uso do Docker em vez do Azure

A criação de um banco de dados no Azure exige uma conta válida, geralmente vinculada a uma assinatura empresarial ou a um plano educacional. Como nenhuma dessas opções estava disponível, foi necessário adotar uma alternativa que permitisse:

*   Reproduzir o mesmo ambiente de banco relacional de forma gratuita;
*   Manter compatibilidade com o MySQL, tecnologia amplamente utilizada;
*   Permitir execução local, sem dependência de credenciais em nuvem.

O Docker Desktop atendeu a esses requisitos, permitindo subir uma instância do MySQL de forma rápida, isolada e reproduzível.

### 3. Instalação e execução do MySQL com Docker

#### 3.1. Subir o container do MySQL

Para criar e iniciar o container com o banco azure_company, execute o comando abaixo:

`docker run --name meu-mysql -e MYSQL_ROOT_PASSWORD= -e MYSQL_DATABASE=azure_company -p 3306:3306 -d mysql:latest`

O que cada parâmetro faz:




Parâmetro
Descrição




--name meu-mysql
Define o nome do container


MYSQL_ROOT_PASSWORD
Define a senha do usuário root


MYSQL_DATABASE
Cria automaticamente o banco azure_company


-p 3306:3306
Expõe a porta 3306 do container para a máquina local


-d
Executa o container em segundo plano


mysql:latest
Imagem oficial do MySQL




#### 3.2. Usar uma senha segura

Em um ambiente de produção ou compartilhado, recomenda-se substituir a senha por um valor forte:

`docker run --name meu-mysql -e MYSQL_ROOT_PASSWORD=SuaSenhaSeguraAqui -p 3306:3306 -d mysql:latest`


Nota: ao usar este comando, o banco azure_company não é criado automaticamente. Nesse caso, crie-o manualmente após conectar ao MySQL.


#### 3.3. Remover o container

Para encerrar e remover o container quando não for mais necessário:

`docker rm -f meu-mysql`

### 4. Configuração da conexão no DBeaver

Após o container estar em execução, configure a conexão no DBeaver com os seguintes parâmetros:

*   **Host:** `localhost`
*   **Porta:** `3306`
*   **Usuário:** `root`
*   **Senha:** a senha definida no comando do Docker
*   **Banco de dados:** `azure_company`

Em Configurações de driver → Propriedades de conexão, adicione os seguintes parâmetros para garantir uma conexão estável:




Propriedade
Valor




allowPublicKeyRetrieval
true


useSSL
false




Essas configurações são necessárias para autenticação com o MySQL 8+ em conexões locais, evitando erros relacionados a chave pública e SSL.


### 5. Banco de dados.

Os scripts estão disponíveis no repositório. Imagem da modelagem.

<img width="678" height="671" alt="image" src="https://github.com/user-attachments/assets/0c96316f-8ef4-48c9-a584-22c007a147fa" />

### 6. Etapas realizadas no Power BI

As seguintes transformações foram executadas no Power BI para preparar os dados:

1.  **Instalação do conector:** instalação do Connector/NET 26.7.0 da Oracle para MySQL, permitindo a conexão do Power BI ao banco.
2.  **Mesclagem de consultas:** utilização da opção Mesclar consulta entre as tabelas departament e dept_locations, unificando os dados de departamentos e suas localizações.
3.  **Criação da coluna FullName:** combinação das colunas FName e LName, separadas por um espaço, para gerar o nome completo dos funcionários.
4.  **Divisão de coluna por delimitador:** divisão de uma coluna usando o hífen (-) como delimitador, gerando as colunas Address Number, Address Name, City e Estado.

### 7. Consulta SQL utilizada

A consulta abaixo foi criada para relacionar cada funcionário ao seu supervisor, exibindo os nomes de ambos:

```sql
SELECT e.Fname, e.Lname, e2.Fname, e2.Lname
FROM employee e
LEFT JOIN employee e2 ON e.Super_ssn = e2.ssn
ORDER BY e2.Fname, e2.Lname, e.Fname, e.Lname;
```

Explicação da consulta:

*   **e** representa o funcionário;
*   **e2** representa o supervisor, obtido pelo auto-relacionamento via `Super_ssn = ssn`;
*   O **LEFT JOIN** garante que todos os funcionários sejam listados, inclusive aqueles sem supervisor;
*   O **ORDER BY** organiza o resultado pelo nome do supervisor e, em seguida, pelo nome do funcionário.

### 8. Conclusão

A substituição do Azure pelo MySQL em Docker não comprometeu o objetivo do projeto, permitindo a criação do banco azure_company, a execução das transformações de dados e a integração com o Power BI de forma local e gratuita. A solução adotada é reproduzível, documentada e adequada para fins de estudo e desenvolvimento.

---
