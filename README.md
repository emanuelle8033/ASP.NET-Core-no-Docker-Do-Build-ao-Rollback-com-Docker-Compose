🚀 ASP.NET Core no Docker: Do Build ao Rollback com Docker ComposeRepositório desenvolvido para o Laboratório de Containers com Docker (Unidade 3 — Integração DevOps). O objetivo deste projeto é demonstrar o ciclo completo de vida de uma aplicação ASP.NET Core em containers: desde a construção da imagem utilizando multi-stage build até a promoção entre ambientes, validação via Health Checks, simulação de falha controlada e execução de rollback.📌 SumárioArquitetura e Fluxo do LaboratórioPré-requisitosEstrutura do ProjetoPasso a Passo de Execução1. Execução Local (.NET CLI)2. Build da Imagem Docker (v1.0.0)3. Deploy nos Ambientes Staging e Production4. Nova Release (v1.1.0) e Promoção5. Simulação de Falha Controlada (v1.2.0) e Rollback6. Orquestração com Docker ComposeEndpoints da APIEvidências e Defesa Técnica🏗️ Arquitetura e Fluxo do Laboratório CÓDIGO C#
    │
    ▼
 DOCKERFILE (Multi-Stage)
    │
    ▼
 docker build ──► devops-webapi:1.0.0
    │
    ├──► STAGING (Porta 8081)
    │     │
    │     ▼ Health Check OK
    │
    └──► PRODUCTION (Porta 8082)
          │
          ├─► Release v1.1.0 (Sucesso)
          ├─► Release v1.2.0 (Falha 500 no Health Check)
          └─► ROLLBACK ──► Retorno seguro para v1.1.0
🛠️ Pré-requisitosAntes de iniciar, certifique-se de possuir instalado na máquina:Docker Desktop ou Docker Engine funcional.NET 10 SDKGitPara verificar as ferramentas no terminal:docker --version
dotnet --version
git --version
📂 Estrutura do ProjetoDevOpsWebApi/
├── .dockerignore         # Arquivo para otimização do contexto do Docker
├── .gitignore            # Exclusões para controle de versão Git
├── Dockerfile            # Multi-stage build (.NET SDK + ASP.NET Runtime)
├── compose.yaml          # Declaração dos ambientes Staging e Production
├── Program.cs            # Código-fonte da Web API com endpoints e health check
├── DevOpsWebApi.csproj   # Configuração do projeto .NET
└── README.md             # Documentação do laboratório
🚀 Passo a Passo de Execução1. Execução Local (.NET CLI)Para validar a aplicação antes da conteinerização:# Restaurar dependências e rodar a API
dotnet run
Acesse http://localhost:5000/health (ou porta informada no console) para testar a resposta inicial.2. Build da Imagem Docker (v1.0.0)Construa a primeira versão da imagem utilizando o Dockerfile otimizado:docker build -t devops-webapi:1.0.0 .
Verifique a imagem gerada:docker images | grep devops-webapi
3. Deploy nos Ambientes Staging e ProductionUtilizando variáveis de ambiente para isolar as configurações sem alterar o código do container:# Container Staging
docker run -d --name devops-api-staging \
  -p 8081:8080 \
  -e APP_ENVIRONMENT=staging \
  -e APP_VERSION=1.0.0 \
  devops-webapi:1.0.0

# Container Production
docker run -d --name devops-api-prod \
  -p 8082:8080 \
  -e APP_ENVIRONMENT=production \
  -e APP_VERSION=1.0.0 \
  devops-webapi:1.0.0
Validando os ambientes:Staging: curl -f http://localhost:8081/healthProduction: curl -f http://localhost:8082/health4. Nova Release (v1.1.0) e PromoçãoApós adicionar o endpoint /version no Program.cs, construa a versão 1.1.0:docker build -t devops-webapi:1.1.0 .
Atualizando Staging:docker rm -f devops-api-staging
docker run -d --name devops-api-staging \
  -p 8081:8080 \
  -e APP_ENVIRONMENT=staging \
  -e APP_VERSION=1.1.0 \
  devops-webapi:1.1.0
Validação do Health Check em Staging:curl -f http://localhost:8081/health
Promovendo para Production:Após o "Quality Gate" de Staging aprovar:docker rm -f devops-api-prod
docker run -d --name devops-api-prod \
  -p 8082:8080 \
  -e APP_ENVIRONMENT=production \
  -e APP_VERSION=1.1.0 \
  devops-webapi:1.1.0
5. Simulação de Falha Controlada (v1.2.0) e RollbackPara demonstrar a importância dos Health Checks, foi gerada uma versão problemática 1.2.0 que retorna status HTTP 500 no endpoint /health.docker build -t devops-webapi:1.2.0 .
Simulando Incidente em Production:docker rm -f devops-api-prod
docker run -d --name devops-api-prod \
  -p 8082:8080 \
  -e APP_ENVIRONMENT=production \
  -e APP_VERSION=1.2.0 \
  devops-webapi:1.2.0
Detecção do Problema:# O comando abaixo retornará erro de execução (Exit Code HTTP 500)
curl -f http://localhost:8082/health
Executando Rollback Imediato:Remoção da versão corrompida e re-instanciação da versão estável previamente catalogada (1.1.0):docker rm -f devops-api-prod

docker run -d --name devops-api-prod \
  -p 8082:8080 \
  -e APP_ENVIRONMENT=production \
  -e APP_VERSION=1.1.0 \
  devops-webapi:1.1.0
Confirmação de Restabelecimento:curl -f http://localhost:8082/health
6. Orquestração com Docker ComposePara automatizar e declarar a infraestrutura dos ambientes em código (compose.yaml), execute:# Remover containers isolados
docker rm -f devops-api-staging devops-api-prod

# Subir todos os serviços declarados
docker compose up -d

# Inspecionar status dos serviços
docker compose ps

# Visualizar logs unificados
docker compose logs -f

# Encerrar ambientes
docker compose down
📡 Endpoints da APIMétodoEndpointDescriçãoGET/Retorna o nome da aplicação, versão e ambiente ativoGET/healthEndpoint de verificação de saúde da aplicaçãoGET/versionRetorna detalhes da release instalada (Disponível a partir da v1.1.0)🧪 Evidências e Defesa TécnicaMulti-Stage Build: Garante imagens mais leves e seguras ao separar a fase de compilação (dotnet/sdk) do ambiente final de execução (dotnet/aspnet).Imutabilidade: O mesmo binário e imagem de container testados em Staging são promovidos sem alterações para Produção.Variáveis de Ambiente: Diferenciação de ambientes realizada de forma externa via APP_ENVIRONMENT e APP_VERSION.Resiliência e Observabilidade: Uso do /health para habilitar verificações automáticas e tomada de decisão sobre Rollback.
