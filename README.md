# 🔐 Tela de Login com Criptografia de Senha

Sistema de login feito com **Spring Boot**, **MySQL** e **BCrypt**. As senhas são guardadas no banco como **hash** (nunca em texto puro). O servidor usa **sessão** para lembrar quem está logado.

## 📸 Telas

| Login | Página inicial |
|-------|----------------|
| Formulário de email e senha, com mensagem de erro | Abre só quando o login está certo e mostra o email logado |

> Dica: coloque prints das telas na pasta `docs/` e troque esta tabela por imagens.

## 🛠️ Tecnologias

- **Java 25**
- **Spring Boot 4** (Spring Web MVC)
- **MySQL** (acesso via JDBC puro, com `PreparedStatement`)
- **jBCrypt**: criptografia das senhas
- **HTML, CSS e JavaScript** (sem frameworks no front-end)

## 📁 Estrutura do projeto

```
src/main/java/com/example/demo/
├── DemoApplication.java   # Liga o servidor
├── LoginController.java   # Rotas /login, /usuario e /logout
├── UsuarioDAO.java        # Conversa com o banco de dados
├── SenhaUtil.java         # Gera e confere o hash BCrypt
└── GerarHash.java         # Classe auxiliar: gera o INSERT de um usuário

src/main/resources/static/
├── criptografia.html      # Tela 1: login
└── home.html              # Tela 2: página inicial (protegida)

banco.sql                  # Script de criação do banco
```

## ⚙️ Como funciona

1. O usuário digita email e senha na tela de login.
2. O JavaScript envia os dados para `POST /login`.
3. O Java busca no banco o **hash** salvo para aquele email.
4. O `BCrypt.checkpw` confere se a senha digitada corresponde ao hash.
5. Se estiver certo, o email é guardado na **sessão** e a página inicial é aberta.
6. A página inicial chama `GET /usuario`. Se ninguém estiver logado, volta para o login.
7. O botão **Sair** chama `POST /logout`, que apaga a sessão.

### Rotas

| Método | Rota       | O que faz                                        |
|--------|------------|--------------------------------------------------|
| POST   | `/login`   | Confere email e senha e cria a sessão            |
| GET    | `/usuario` | Retorna o email logado (ou 401 se não houver)    |
| POST   | `/logout`  | Encerra a sessão                                 |

## 🔒 Segurança aplicada

- **Hash com BCrypt**: a senha nunca é salva em texto puro.
- **Salt automático**: a mesma senha gera hashes diferentes a cada vez.
- **PreparedStatement**: protege contra SQL Injection.
- **Mensagem de erro genérica**: não revela se o erro foi no email ou na senha.
- **Página protegida**: a tela inicial só aparece depois que o servidor confirma o login.

## 🚀 Como rodar

### Pré-requisitos

- Java 25 (JDK)
- Maven (ou a IDE com Maven integrado)
- MySQL rodando na porta `3306`

### 1. Criar o banco

Rode o script no MySQL (Workbench, DBeaver ou terminal):

```sql
CREATE DATABASE IF NOT EXISTS cripto;
USE cripto;

CREATE TABLE IF NOT EXISTS usuarios (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    email      VARCHAR(150) NOT NULL UNIQUE,
    senha_hash VARCHAR(255) NOT NULL
);
```

### 2. Configurar a conexão

No arquivo `UsuarioDAO.java`, ajuste o usuário e a senha **do seu MySQL**:

```java
private static final String URL =
    "jdbc:mysql://localhost:3306/cripto?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC";
private static final String USER = "root";
private static final String PASS = "sua_senha_do_mysql";
```

> ⚠️ O nome do banco na URL (`cripto`) precisa ser o mesmo criado no passo 1.

### 3. Criar um usuário

Como não há tela de cadastro, o usuário é inserido direto no banco:

1. Abra `GerarHash.java` e troque o email e a senha.
2. Rode o `main` dessa classe.
3. Copie o `INSERT` que aparece no console e execute no MySQL.

### 4. Iniciar o servidor

Rode a classe `DemoApplication` pela IDE, ou pelo terminal:

```bash
mvn spring-boot:run
```

### 5. Acessar

Abra no navegador: **http://localhost:8080/criptografia.html**

## 🧩 Problemas comuns

| Problema | Causa provável | Solução |
|----------|----------------|---------|
| "Email ou senha inválidos" mesmo com dados certos | Nome do banco na URL diferente do criado | Confira se a URL usa `cripto` |
| `Access denied for user 'root'` no console | Senha do MySQL errada | Ajuste `PASS` no `UsuarioDAO` |
| `Public Key Retrieval is not allowed` | MySQL 8 sem SSL | Adicione `allowPublicKeyRetrieval=true` na URL |
| Página não abre (erro 404) | HTML fora da pasta certa | Coloque os HTML em `src/main/resources/static/` |
| Usuário não encontrado | `INSERT` não foi executado | Rode `SELECT * FROM cripto.usuarios;` para conferir |

## 📌 Melhorias futuras

- Tela de cadastro de usuários
- Configuração do banco em `application.properties` em vez de fixa no código
- Uso do Spring Security
- Limite de tentativas de login
