# Guess Game

**Guess Game** é um jogo de adivinhação desenvolvido com backend em **Flask/Python** e frontend em **React**. O projeto tem arquitetura em três camadas, suporte a containerização com Docker e abstração de persistência por repositório.

---

## Visão Geral

- Backend: Flask + Python 3.10
- Frontend: React + TypeScript
- Banco de dados principal: PostgreSQL
- Alternativas: SQLite e DynamoDB
- Orquestração: Docker Compose
- Proxy reverso: Nginx serve o frontend e encaminha `/api/` para o backend

---

## Funcionalidades

- Criar um novo jogo com senha secreta
- Enviar tentativas de adivinhação
- Feedback de acertos por posição correta e letra correta em posição errada
- Armazenamento de jogos via repositório configurável
- Execução local e via Docker Compose

---

## Estrutura do Projeto

- `/frontend`: aplicação React, testes e Dockerfile do frontend
- `/guess`: backend Flask e lógica de jogo
- `/repository`: abstração de persistência e implementações para PostgreSQL, SQLite e DynamoDB
- `Dockerfile`: imagem do backend Flask
- `docker-compose.yaml`: orquestração de `db`, `backend` e `frontend`
- `nginx.conf`: proxy reverso para frontend e API
- `run.py`: inicializador do Flask
- `start-backend.sh`: script de ambiente para execução local

---

## 3. Executar com Docker Compose

O Docker Compose orquestra os serviços de frontend, backend e banco de dados juntos.

1. No diretório raiz do projeto, execute:

   ```bash
   docker compose up --build
   ```

2. Acesse o frontend em:

   ```bash
   http://localhost:8080
   ```

### Serviços provisionados

- `db`: PostgreSQL 15-alpine com volume persistente `db_data`
- `backend`: Flask app expondo a API
- `frontend`: React servido pelo Nginx na porta `8080`

---

## Arquitetura e Decisões Técnicas

### Arquitetura do Sistema

A aplicação segue uma arquitetura de três camadas:

- **Frontend**: SPA React isolada em `/frontend`.
- **Backend**: API Flask em `/guess`.
- **Persistência**: abstração por repositório em `/repository`.

### Containerização e Otimização

A ideia principal foi deixar tudo mais fácil de executar e também apresentar um fluxo que mostre uso de container em cada camada.

- O backend usa `python:3.10-slim` porque é uma imagem mais leve e suficiente para rodar o Flask sem muita coisa extra.
- No Dockerfile do backend, as dependências são instaladas antes de copiar o código. Isso ajuda o Docker a reaproveitar a camada de instalação quando o código muda, o que é um detalhe importante para builds mais rápidos.
- O frontend foi feito com um build em duas etapas (`node:18-alpine` após `nginx:alpine`) para separar a fase de compilação do React da fase de execução. Assim, a imagem final só contém os arquivos estáticos e o Nginx, evitando peso desnecessário.
- O Nginx é usado como proxy reverso para servir a interface e encaminhar as chamadas de API ao backend. Esse arranjo deixa claro que a aplicação web e a API são serviços distintos.

### Docker Compose e Rede

- O `docker-compose.yaml` junta `db`, `backend` e `frontend` em uma rede interna chamada `game_net`.
- O frontend expõe a porta `8080` para fora, enquanto o banco e o backend ficam internos. Isso ajuda a demonstrar um padrão comum de segurança e isolamento em containerização.
- O PostgreSQL usa volume nomeado `db_data` para manter dados entre reinícios dos containers.
- O backend declara `depends_on: db` para garantir que o serviço de banco esteja disponível antes de iniciar. Isso não substitui verificações de saúde completas, mas serve para o fluxo básico de inicialização.
- O `deploy.replicas: 3` no backend mostra um conceito de escalabilidade, mesmo que no Docker Compose isso não seja um balanceamento de produção completo.

