# 🚀 ASP.NET Core no Docker: Do Build ao Rollback com Docker Compose

## 📌 Sobre o Projeto

Este projeto faz parte do laboratório de Integração DevOps – Unidade 03, cujo objetivo é demonstrar o processo de containerização de uma aplicação ASP.NET Core utilizando Docker e Docker Compose.

Durante a atividade, foi desenvolvida uma API Web em C# e realizado o processo de construção de imagens Docker, execução de containers, configuração de ambientes, versionamento de releases, validação de saúde da aplicação e rollback.

O laboratório simula um fluxo de implantação utilizado em ambientes de desenvolvimento de software.

## 🎯 Objetivos

- Criar uma aplicação ASP.NET Core Web API.
- Construir imagens Docker utilizando Dockerfile.
- Aplicar o conceito de Multi-stage Build.
- Executar containers Docker.
- Configurar ambientes de Staging e Production.
- Trabalhar com variáveis de ambiente.
- Versionar imagens Docker.
- Realizar Health Checks.
- Simular falhas controladas.
- Executar rollback para uma versão estável.
- Gerenciar ambientes utilizando Docker Compose.

## 🛠️ Tecnologias Utilizadas

- C#
- ASP.NET Core
- .NET 10
- Docker
- Docker Desktop
- Docker Compose
- Git e GitHub

## 📂 Estrutura do Projeto

```text
DevOpsWebApi/
│
├── Program.cs
├── DevOpsWebApi.csproj
├── appsettings.json
├── Dockerfile
├── .dockerignore
├── compose.yaml
└── README.md
```

## ⚙️ Pré-requisitos

Antes de executar o projeto, é necessário ter instalado:

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Git](https://git-scm.com/)

Para verificar as instalações, execute:

```bash
dotnet --version
docker --version
git --version
```

## 1. Criando a aplicação ASP.NET Core

Para criar o projeto, utilize:

```bash
dotnet new webapi -n DevOpsWebApi
```

Entre na pasta:

```bash
cd DevOpsWebApi
```

Execute a aplicação localmente:

```bash
dotnet run
```

A API possui os seguintes endpoints:

| Endpoint | Descrição |
|---|---|
| `/` | Exibe informações da aplicação |
| `/health` | Verifica a saúde da aplicação |
| `/version` | Exibe informações da versão |

## 2. Construindo a imagem Docker

O projeto utiliza um Dockerfile com Multi-stage Build, separando o processo de compilação da execução da aplicação.

Para construir a primeira imagem:

```bash
docker build -t devops-webapi:1.0.0 .
```

Para verificar as imagens criadas:

```bash
docker images
```

## 3. Executando o primeiro container

Execute:

```bash
docker run -d --name devops-api -p 8080:8080 devops-webapi:1.0.0
```

Acesse a aplicação:

- http://localhost:8080/
- http://localhost:8080/health

Para verificar os containers em execução:

```bash
docker ps
```

## 4. Configurando os ambientes

O laboratório utiliza dois ambientes diferentes:

**Staging:**

```bash
docker run -d --name devops-api-staging -p 8081:8080 -e APP_ENVIRONMENT=staging -e APP_VERSION=1.0.0 devops-webapi:1.0.0
```

**Production:**

```bash
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.0.0 devops-webapi:1.0.0
```

A mesma imagem Docker é utilizada nos dois ambientes, modificando apenas as configurações por meio de variáveis de ambiente.

## 5. Versionamento das imagens

Para criar uma nova versão da aplicação, foi utilizada a tag `1.1.0`.

```bash
docker build -t devops-webapi:1.1.0 .
```

Assim, as versões anteriores continuam disponíveis:

```text
devops-webapi:1.0.0
devops-webapi:1.1.0
```

A nova versão é implantada primeiro em Staging para realização dos testes.

## 6. Health Check

A verificação de saúde permite identificar se a aplicação está respondendo corretamente.

No Windows, execute:

```bash
curl.exe -f http://localhost:8081/health
```

Para Production:

```bash
curl.exe -f http://localhost:8082/health
```

Se a aplicação responder com sucesso, poderá ser considerada apta para a próxima etapa de implantação.

## 7. Simulação de falha controlada

Para simular um problema, foi criada uma versão `1.2.0` com uma falha proposital no endpoint `/health`, retornando o código HTTP 500.

Construção da imagem:

```bash
docker build -t devops-webapi:1.2.0 .
```

Execução em Staging:

```bash
docker rm -f devops-api-staging
```

```bash
docker run -d --name devops-api-staging -p 8081:8080 -e APP_ENVIRONMENT=staging -e APP_VERSION=1.2.0 devops-webapi:1.2.0
```

Teste:

```bash
curl.exe -f http://localhost:8081/health
```

O resultado esperado é uma falha na requisição HTTP, demonstrando que a versão não deve ser promovida para produção.

## 8. Rollback

O rollback consiste em retornar para uma versão anterior e estável da aplicação.

Para realizar o procedimento:

Remova o container problemático:

```bash
docker rm -f devops-api-prod
```

Execute novamente a versão estável `1.1.0`:

```bash
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.1.0 devops-webapi:1.1.0
```

Valide a aplicação:

```bash
curl.exe -f http://localhost:8082/health
```

Dessa forma, o ambiente de produção retorna à versão anterior que apresentou funcionamento correto.

## 9. Docker Compose

Para facilitar o gerenciamento dos ambientes, foi criado o arquivo `compose.yaml`.

```yaml
services:

  staging:
    image: devops-webapi:1.1.0
    container_name: devops-api-staging
    ports:
      - "8081:8080"
    environment:
      APP_ENVIRONMENT: staging
      APP_VERSION: "1.1.0"

  production:
    image: devops-webapi:1.1.0
    container_name: devops-api-prod
    ports:
      - "8082:8080"
    environment:
      APP_ENVIRONMENT: production
      APP_VERSION: "1.1.0"
```

Para iniciar os ambientes:

```bash
docker compose up -d
```

Verificar os serviços:

```bash
docker compose ps
```

Visualizar os logs:

```bash
docker compose logs
```

Acompanhar os logs:

```bash
docker compose logs -f
```

Validar os ambientes:

```bash
curl.exe -f http://localhost:8081/health
```

```bash
curl.exe -f http://localhost:8082/health
```

Para encerrar os containers:

```bash
docker compose down
```

## 10. Limpeza do ambiente

Para remover os containers:

```bash
docker compose down
```

Para visualizar as imagens:

```bash
docker images
```

Opcionalmente, remova as imagens criadas no laboratório:

```bash
docker rmi devops-webapi:1.0.0
docker rmi devops-webapi:1.1.0
docker rmi devops-webapi:1.2.0
```

## 📊 Fluxo de Implantação

```text
CÓDIGO C#
    |
    v
DOCKERFILE
    |
    v
DOCKER BUILD
    |
    v
IMAGEM VERSIONADA
    |
    v
CONTAINER
    |
    v
STAGING
    |
    v
HEALTH CHECK
    |
    v
VALIDAÇÃO
    |
    v
PRODUCTION
    |
    v
NOVA RELEASE
    |
    v
FALHA CONTROLADA
    |
    v
ROLLBACK
```

## 📸 Evidências da Atividade

As seguintes evidências devem ser apresentadas no repositório:

- [ ] Aplicação ASP.NET Core executando.
- [ ] Dockerfile criado.
- [ ] Arquivo `.dockerignore` criado.
- [ ] Imagem Docker versão 1.0.0.
- [ ] Imagem Docker versão 1.1.0.
- [ ] Container de Staging em execução.
- [ ] Container de Production em execução.
- [ ] Health Check funcionando.
- [ ] Simulação da falha na versão 1.2.0.
- [ ] Rollback realizado para a versão 1.1.0.
- [ ] Arquivo `compose.yaml` funcionando.

## 📚 Aprendizados

Com a realização deste laboratório, foi possível compreender como o Docker facilita a implantação de aplicações ASP.NET Core, permitindo criar ambientes mais organizados, reproduzíveis e independentes da máquina utilizada.

Também foi possível entender a importância do versionamento de imagens, da separação entre Staging e Production, das verificações de saúde e do rollback para recuperar uma versão estável após uma falha.

O Docker Compose contribuiu para simplificar o gerenciamento dos ambientes, centralizando suas configurações em um único arquivo.

## 👩‍💻 Autoria

**Aluna:** Emanuelle Almeida de Freitas

**Disciplina:** Integração DevOps

**Unidade:** 03

**Professor:** Daniel Lim-Apo

---

**Projeto desenvolvido
