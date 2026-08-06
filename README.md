# Setup e Padrões do Ambiente Backend (Node.js + JavaScript)

## 📌 TL;DR

> Setup de backend Node.js + JavaScript com Docker: pastas, dependências, segurança e containers - passo a passo, do zero.

Este documento é o guia de onboarding para configurar o ambiente de back-end da **TechStore**. Cada passo tem: **comando exato**, **arquivo/pasta afetado**, **resultado esperado** e o **porquê**.

> 🖥️ **Nota:** todos os comandos deste documento usam sintaxe **bash**. Funciona direto no Linux, macOS e WSL. No Windows "puro", use o **Git Bash** (não o PowerShell/CMD, onde alguns comandos como `mkdir -p` e `touch` não existem do mesmo jeito).

---

## 📑 Sumário

1. [Pré-Requisitos](#1-pré-requisitos)
2. [Visão Geral da Estrutura Final](#2-visão-geral-da-estrutura-final)
3. [Passo a Passo do Setup](#3-passo-a-passo-do-setup)
   - [✅ Passo 1 - Criar o diretório e iniciar o projeto](#-passo-1---criar-o-diretório-e-iniciar-o-projeto)
   - [✅ Passo 2 - Criar a estrutura básica de pastas](#-passo-2---criar-a-estrutura-básica-de-pastas)
   - [✅ Passo 3 - Instalar as dependências de desenvolvimento](#-passo-3---instalar-as-dependências-de-desenvolvimento)
   - [✅ Passo 4 - Instalar as dependências essenciais de produção](#-passo-4---instalar-as-dependências-essenciais-de-produção)
   - [✅ Passo 5 - Criar `.gitignore` e `.env.example`](#-passo-5---criar-gitignore-e-envexample)
   - [✅ Passo 6 - Remover o header `x-powered-by`](#-passo-6---remover-o-header-x-powered-by)
   - [✅ Passo 7 - Configurar o Helmet e a rota `/health`](#-passo-7---configurar-o-helmet-e-a-rota-health)
   - [✅ Passo 8 - Criar o `Dockerfile`](#-passo-8---criar-o-dockerfile)
   - [✅ Passo 9 - Criar o `.dockerignore`](#-passo-9---criar-o-dockerignore)
   - [✅ Passo 10 - Criar/atualizar o `docker-compose.yml` na raiz do repositório](#-passo-10---criaratualizar-o-docker-composeyml-na-raiz-do-repositório)
   - [✅ Passo 11 - Testar se o ambiente sobe com Docker Compose](#-passo-11---testar-se-o-ambiente-sobe-com-docker-compose)
   - [✅ Passo 12 - Criar o `README.md` do backend](#-passo-12---criar-o-readmemd-do-backend)
4. [Padrões de Código (ESLint, Prettier, Husky)](#4-padrões-de-código-eslint-prettier-husky)
5. [Comandos Úteis (resumo)](#5-comandos-úteis-resumo)
6. [Padrões de Commit e Branch](#6-padrões-de-commit-e-branch)
7. [Observabilidade](#7-observabilidade)
8. [Troubleshooting (Erros Comuns)](#8-troubleshooting-erros-comuns)
9. [Referências Técnicas](#9-referências-técnicas)
10. [Dúvidas e Evoluções](#10-dúvidas-e-evoluções)

---

## 1. Pré-Requisitos

Antes de começar, confirme no terminal que todos os itens abaixo estão instalados:

```bash
node -v    # precisa ser >= v20.x
npm -v     # precisa ser >= v9.x
docker -v  # opcional, mas recomendado
docker compose version  # opcional
```

Se algum comando não existir, instalar antes de continuar:
- Node.js: https://nodejs.org/en/download (a instalação já vem com npm)
- Docker Desktop: https://docs.docker.com/get-docker/ (já vem com Docker Compose)

> 💡 Para múltiplos projetos Node com versões diferentes, considere usar `nvm` (Node Version Manager) para trocar de versão facilmente.

---

## 2. Visão Geral da Estrutura Final

Depois de seguir todos os passos, o projeto deve ficar assim:

```
seu-repositorio/                  ← raiz do repositório Git
├── docker-compose.yml            ← orquestra backend + frontend
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── middlewares/
│   │   └── index.js              ← ponto de entrada da aplicação
│   ├── tests/
│   ├── data/                     ← arquivos JSON de persistência (RN04 do PRD)
│   ├── .env                      ← NUNCA vai pro Git (segredos reais)
│   ├── .env.example              ← vai pro Git (modelo sem valores reais)
│   ├── .gitignore
│   ├── .dockerignore             ← controla o que NÃO entra na imagem Docker
│   ├── package.json
│   ├── package-lock.json
│   ├── Dockerfile
│   └── README.md
└── frontend/                     ← (fora do escopo deste documento)
```

**Por quê:** `docker-compose.yml` fica na raiz porque orquestra *mais de um* serviço (backend, frontend, e futuramente banco/cache) - se ficasse dentro de `backend/`, só "enxergaria" o backend. Já o `.dockerignore` fica dentro de `backend/`, porque é lá que está o **contexto de build** do Docker (mais detalhes no Passo 9).

---

## 3. Passo a Passo do Setup

### ✅ Passo 1 - Criar o diretório e iniciar o projeto

**Comando (bash):**
```bash
mkdir backend && cd backend && npm init -y
```

**O que isso faz:**
- `mkdir backend` → cria a pasta do projeto.
- `cd backend` → entra nela (todos os próximos comandos rodam de dentro dela).
- `npm init -y` → cria o `package.json` com valores padrão, sem perguntar nada (o `-y` significa "yes para tudo").

**Resultado esperado:** um arquivo `backend/package.json` foi criado, parecido com:
```json
{
  "name": "backend",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": { "test": "echo \"Error: no test specified\" && exit 1" }
}
```

⚠️ **Atenção:** a partir daqui, todo comando do restante deste documento assume execução **dentro da pasta `backend/`**, salvo indicação contrária (ex: os comandos do `docker-compose.yml`, que rodam na raiz do repositório).

---

### ✅ Passo 2 - Criar a estrutura básica de pastas

**Comando (bash, rodando de dentro de `backend/`):**
```bash
mkdir -p src/controllers src/models src/routes src/middlewares tests data
touch src/index.js
```

**Para que serve cada pasta:**
| Pasta | Serve para |
|---|---|
| `src/controllers` | Funções que recebem a requisição HTTP e decidem o que responder (ex: `criarPedido`, `listarProdutos`) |
| `src/models` | Estrutura dos dados (ex: como é um "Produto", como é um "Pedido") e acesso à persistência (Repository Pattern, conforme RN04 do PRD) |
| `src/routes` | Mapeamento de URL → controller (ex: `POST /produtos` → `criarProduto`) |
| `src/middlewares` | Código que roda *antes* do controller (ex: checar se o usuário está logado, checar rate limit) |
| `tests` | Testes automatizados (Jest) |
| `data` | Arquivos JSON de persistência do MVP (ex: `database.json`, `products.json` - conforme RN04) |

**Resultado esperado:** rodando `ls -R` dentro de `backend/`, todas as pastas acima aparecem, e `src/index.js` existe (vazio por enquanto).

---

### ✅ Passo 3 - Instalar as dependências de desenvolvimento

**Comando (bash):**
```bash
npm install --save-dev nodemon jest supertest eslint prettier
```

**O que é cada pacote:**
| Pacote | Para que serve |
|---|---|
| `nodemon` | Reinicia o servidor automaticamente sempre que um arquivo é salvo, sem precisar parar e rodar `node` de novo manualmente |
| `jest` | Framework de testes automatizados |
| `supertest` | Testar rotas HTTP (ex: simular um `POST /login`) sem precisar subir o servidor de verdade |
| `eslint` + `prettier` | Padronização e formatação de código |

**Resultado esperado:** essas dependências aparecem em `package.json`, dentro de `"devDependencies"`.

---

### ✅ Passo 4 - Instalar as dependências essenciais de produção

**Comando (bash):**
```bash
npm install express jsonwebtoken bcrypt dotenv helmet express-rate-limit
```

**O que é cada pacote (e onde ele conecta com o PRD):**
| Pacote | Para que serve | Relação com o PRD |
|---|---|---|
| `express` | Framework web/HTTP | Base de tudo |
| `jsonwebtoken` | Gerar/validar tokens JWT | Autenticação (RF02) |
| `bcrypt` | Fazer hash de senha | RN02 - senha **nunca** em texto plano |
| `dotenv` | Ler variáveis do arquivo `.env` no código | Segredos fora do código (Passo 5) |
| `helmet` | Configurar cabeçalhos HTTP seguros automaticamente | RNF02 - segurança da aplicação |
| `express-rate-limit` | Limitar número de requisições por IP/janela de tempo | RF02/RF03, RNF02 - rate limiting em login e recuperação de senha |

**Resultado esperado:** essas dependências aparecem em `package.json`, dentro de `"dependencies"` (não `devDependencies` - são necessárias em produção, não só em desenvolvimento).

> 📌 **Nota:** a instalação do `express-rate-limit` faz parte do esqueleto do ambiente. A aplicação do middleware (ex: `app.use('/login', rateLimiter)`) fica para quando as rotas de login e recuperação de senha forem implementadas, já que depende da lógica específica de cada rota.

---

### ✅ Passo 5 - Criar `.gitignore` e `.env.example`

**Comando (bash):**
```bash
touch .gitignore .env .env.example
```

**Conteúdo do `backend/.gitignore`:**
```
node_modules/
.env
*.log
coverage/
```

**Conteúdo do `backend/.env.example`** (vai para o Git - é só o *modelo*, sem valores reais):
```
PORT=3000
JWT_SECRET=
DB_URL=
```

**Conteúdo do `backend/.env`** (não vai para o Git - aqui vão os valores reais, mesmo que fictícios em ambiente de dev):
```
PORT=3000
JWT_SECRET=uma-string-longa-e-aleatoria-aqui
DB_URL=./data/database.json
```

**Por quê:**
- `.env` fica de fora do Git (`.gitignore`) para que segredos (senhas, chaves JWT) nunca sejam versionados nem apareçam no histórico do repositório.
- `.env.example` **fica** no Git - documenta *quais* variáveis existem, sem revelar os valores, para que qualquer pessoa do time saiba o que precisa configurar ao clonar o projeto.
- 🚨 Nunca escrever uma chave secreta ou senha padrão diretamente no código (hardcoded), nem mesmo "só para teste" ou como valor de fallback - é um dos erros de segurança mais comuns.

**Resultado esperado:** ao rodar `git status`, o arquivo `.env` **não aparece** como arquivo a ser commitado, mas `.env.example` e `.gitignore` aparecem.

---

### ✅ Passo 6 - Remover o header `x-powered-by`

**Onde:** `src/index.js` (ponto de entrada da aplicação)

```javascript
require('dotenv').config()

const express = require('express')

const app = express()
app.disable('x-powered-by')
```

**Por quê:**
- `require('dotenv').config()` carrega as variáveis do `.env` para `process.env` - precisa ser a primeira coisa executada no arquivo, antes de qualquer código que dependa dessas variáveis.
- Por padrão, o Express envia um header HTTP `X-Powered-By: Express` em toda resposta. Isso entrega de graça, para qualquer atacante, qual framework/stack está em uso - informação que facilita a busca por vulnerabilidades conhecidas daquele framework. `app.disable('x-powered-by')` remove esse header.

**Resultado esperado:** depois de subir o servidor, uma requisição de teste (`curl -I http://localhost:3000`) **não** mostra o header `X-Powered-By` na resposta.

---

### ✅ Passo 7 - Configurar o Helmet e a rota `/health`

**Onde:** mesmo arquivo `src/index.js`, logo depois do Passo 6

```javascript
const helmet = require('helmet')

app.use(helmet())

app.get('/health', (_req, res) => {
  res.status(200).json({ status: 'ok' })
})

app.listen(process.env.PORT || 3000, () => {
  console.log(`Servidor rodando na porta ${process.env.PORT || 3000}`)
})
```

**Por quê:**
- `helmet()` configura de uma vez só um conjunto de cabeçalhos HTTP de segurança recomendados (ex: `Content-Security-Policy`, `X-Frame-Options`, `Strict-Transport-Security`), reduzindo a superfície de ataque contra vulnerabilidades comuns como clickjacking e MIME sniffing. É basicamente "boas práticas de segurança de header" empacotadas em uma linha, em vez de configurar cada header manualmente.
- A rota `GET /health` precisa existir desde já porque o `healthcheck` do `docker-compose.yml` (Passo 10) depende dela para saber se o container está saudável. Sem essa rota, o Docker Compose marca o serviço `backend` como `unhealthy` mesmo que a aplicação esteja funcionando normalmente.

**Resultado esperado:**
- `curl -I http://localhost:3000` mostra vários headers novos de segurança na resposta.
- `curl http://localhost:3000/health` retorna `{"status":"ok"}` com status HTTP `200`.

---

### ✅ Passo 8 - Criar o `Dockerfile`

**Arquivo:** `backend/Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
RUN apk add --no-cache curl
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev
COPY src ./src
EXPOSE 3000
USER node
CMD ["node", "src/index.js"]
```

**Por quê, linha a linha:**
| Linha | Por quê |
|---|---|
| `FROM node:20-alpine` | Imagem Node oficial baseada em Alpine Linux - bem mais leve que a imagem padrão, reduz superfície de ataque e tempo de build. |
| `WORKDIR /app` | Define a pasta de trabalho dentro do container. |
| `RUN apk add --no-cache curl` | A imagem `node:20-alpine` não vem com `curl` instalado por padrão. Sem essa linha, o `healthcheck` do `docker-compose.yml` (Passo 10), que roda `curl -f http://localhost:3000/health` **dentro do container**, falha com `curl: not found` e marca o serviço como `unhealthy` mesmo com a aplicação funcionando. |
| `COPY package*.json ./` + `RUN npm ci --omit=dev` | Copia só os arquivos de dependência primeiro e instala apenas as dependências de produção - `nodemon`, `jest`, `eslint`, etc. não entram na imagem final. Isso também aproveita o cache do Docker: se o código mudar mas as dependências não, o build fica bem mais rápido. |
| `COPY src ./src` | Copia apenas o código da aplicação - não copia `tests/`, `.env`, `.git`, etc. (reforçado pelo `.dockerignore` do Passo 9). |
| `EXPOSE 3000` | Documenta que o container escuta na porta 3000. |
| `USER node` | Troca para um usuário **não-root** dentro do container - se uma falha na aplicação for explorada, o dano fica limitado, sem privilégios de root no container. |
| `CMD [...]` | Comando que roda quando o container inicia: executa o código diretamente com `node`, sem etapa de build/compilação. |

**Resultado esperado:** rodando `docker build -t techstore-backend .` dentro de `backend/`, o build termina sem erro.

---

### ✅ Passo 9 - Criar o `.dockerignore`

**Arquivo:** `backend/.dockerignore` - **no mesmo nível do `Dockerfile`, dentro de `backend/`**

> ⚠️ Diferente do que se costuma imaginar, o `.dockerignore` **não** fica na raiz do repositório por padrão - ele fica na raiz do **contexto de build** do Docker. O contexto de build é a pasta apontada em `build:` no `docker-compose.yml` (Passo 10), que neste projeto é `./backend`. Por isso o arquivo vai dentro de `backend/`, e não na raiz do repositório.

**Comando (bash):**
```bash
touch .dockerignore
```

**Conteúdo do `backend/.dockerignore`:**
```
node_modules
npm-debug.log
.git
.gitignore
.env
.env.example
*.md
tests
coverage
```

**Por quê:**
- Mesmo o Dockerfile copiando apenas `src/` explicitamente (Passo 8), o `.dockerignore` continua importante para qualquer comando que use o contexto de build inteiro (ex: `docker build` calculando o hash do contexto) e para evitar que arquivos sensíveis, como o `.env`, cheguem perto da imagem em qualquer cenário.
- Ignorar `node_modules` e `.git` deixa o envio do contexto de build **mais rápido**, já que o Docker não precisa nem considerar esses arquivos.

**Resultado esperado:** rodando `docker build -t techstore-backend .` novamente, o build fica visivelmente mais rápido, e inspecionando a imagem gerada (`docker run --rm -it techstore-backend sh` e depois `ls -la`) não aparecem `node_modules` duplicado, `.git` nem `.env`.

---

### ✅ Passo 10 - Criar/atualizar o `docker-compose.yml` na raiz do repositório

**Arquivo:** `docker-compose.yml` - **na raiz do repositório, fora da pasta `backend/`**

```yaml
services:
  backend:
    build: ./backend
    env_file:
      - ./backend/.env
    volumes:
      - ./backend/data:/app/data
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    depends_on:
      - backend
```

**Por quê:**
- `build: ./backend` → aponta para a pasta do backend (caminho relativo à raiz, onde este arquivo está). Essa é a pasta que vira o **contexto de build** - por isso o `.dockerignore` do Passo 9 fica dentro dela.
- `env_file: ./backend/.env` → garante que todas as variáveis do `.env` local (`JWT_SECRET`, `DB_URL`, etc.) sejam injetadas no container em runtime, e não apenas a variável `NODE_ENV` declarada explicitamente em `environment`.
- `environment: NODE_ENV=production` continua declarado explicitamente porque sobrescreve/garante esse valor independente do que estiver no `.env` local (que em dev pode ter `NODE_ENV` diferente ou nem ter a chave).
- `volumes: ./backend/data:/app/data` → garante que os arquivos JSON de persistência (`backend/data/`) sobrevivem mesmo se o container for recriado.
- `healthcheck` → o Docker passa a monitorar se `/health` responde (rota criada no Passo 7); se a aplicação travar, o Docker sabe que o container está "não saudável".
- O serviço `frontend` está incluído como esqueleto - ajustar o `build`/porta quando o front-end estiver com Dockerfile próprio.

**Resultado esperado:** o comando `docker compose config` (rodado na raiz) não mostra erro de sintaxe, e mostra as variáveis de `backend/.env` já resolvidas dentro do serviço `backend`.

---

### ✅ Passo 11 - Testar se o ambiente sobe com Docker Compose

**Comando (bash, na raiz do repositório):**
```bash
docker compose up --build
```

**O que observar:**
- O build de `backend` (e `frontend`, se já existir) termina sem erro.
- O terminal mostra o log do servidor Express subindo (ex: `Servidor rodando na porta 3000`).
- Abrindo `http://localhost:3000/health` no navegador (ou `curl http://localhost:3000/health`), a resposta é `{"status":"ok"}` (não erro de conexão recusada).
- Rodando `docker compose ps`, o serviço `backend` aparece como `healthy` depois de alguns segundos.

**Para parar:** `Ctrl + C`, depois `docker compose down` para remover os containers.

**Resultado esperado:** os dois ambientes (back e front) sobem juntos com um único comando, sem erro.

---

### ✅ Passo 12 - Criar o `README.md` do backend

**Arquivo:** `backend/README.md`

Estrutura mínima recomendada (padrão de mercado):

```markdown
# TechStore - Backend

API do TechStore: e-commerce com gestão de catálogo, carrinho e pagamento via PIX.

## Tech Stack
- Node.js 20 + Express
- Persistência: arquivos JSON (MVP) - ver `RN04` no PRD
- Autenticação: sessão via cookie + JWT
- Segurança: Helmet, rate limiting, bcrypt

## Como rodar localmente

\`\`\`bash
cp .env.example .env   # preencher os valores reais
npm install
npm run dev
\`\`\`

## Como rodar com Docker

\`\`\`bash
docker compose up --build
\`\`\`

## Scripts disponíveis
| Comando | O que faz |
|---|---|
| `npm run dev` | Sobe em modo desenvolvimento com reload automático (nodemon) |
| `npm start` | Sobe em modo produção, sem reload automático |
| `npm test` | Roda a suíte de testes (Jest) |
| `npm run lint` | Checa padrões de código (ESLint) |

## Estrutura de pastas
Ver seção 2 do `setup-backend.md`.
```

**Por quê:** todo repositório de mercado tem um README que responde, em 30 segundos de leitura, três perguntas: *o que é isso*, *qual stack* e *como rodar na máquina local*. Sem isso, cada pessoa nova no time perde tempo perguntando o óbvio.

**Resultado esperado:** alguém que nunca viu o projeto consegue, só lendo o README, rodar a aplicação localmente sem precisar perguntar nada a ninguém.

---

## 4. Padrões de Código (ESLint, Prettier, Husky)

**Instalação adicional (bash):**
```bash
npm install --save-dev eslint-config-prettier eslint-plugin-prettier husky lint-staged
npx husky install
```

**Scripts recomendados no `package.json`:**
```json
{
  "scripts": {
    "dev": "nodemon src/index.js",
    "start": "node src/index.js",
    "test": "jest",
    "lint": "eslint 'src/**/*.js'",
    "format": "prettier --write 'src/**/*.{js,json,md}'",
    "prepare": "husky install"
  }
}
```

**Por quê:** `husky` + `lint-staged` fazem o lint/format rodar **automaticamente antes de cada commit**, pegando erro de estilo antes de chegar no Pull Request - em vez de descobrir só quando o CI falhar.

---

## 5. Comandos Úteis (resumo)

| Comando | O que faz |
|---|---|
| `npm run dev` | Start em modo desenvolvimento, com reload automático (`nodemon`) |
| `npm start` | Start em modo produção |
| `npm test` | Roda testes automatizados |
| `npm run lint` | Checa lint do código |
| `docker compose up --build` | Sobe backend + frontend juntos |
| `docker compose ps` | Mostra status (incluindo `healthy`/`unhealthy`) dos containers |
| `docker compose logs -f backend` | Acompanha os logs do backend em tempo real |

---

## 6. Padrões de Commit e Branch

- Seguir **Conventional Commits** (em português, conforme decisão do time), ex: `feat: adiciona rota de login`, `fix: corrige validação de e-mail`.
- Pull Requests obrigatórios, aprovados antes de mergear (squash merge).
- Pipelines de CI (GitHub Actions e GitLab CI, já que o repositório é espelhado nas duas plataformas) rodando build, lint e testes a cada PR.
- Husky + Lint-Staged rodando localmente antes do push, para pegar problema antes mesmo de abrir o PR.

---

## 7. Observabilidade

- Expor métricas básicas em formato Prometheus, quando possível (RNF04 do PRD).
- Logs da aplicação **nunca** devem conter dados sensíveis (senha, token, dado pessoal do cliente) - nem mesmo em nível de log de debug.

---

## 8. Troubleshooting (Erros Comuns)

| Sintoma | Causa provável | Como resolver |
|---|---|---|
| `Error: listen EADDRINUSE :::3000` | Porta 3000 já está em uso por outro processo (ou outro container). | Rodar `lsof -i :3000` (Linux/macOS) para localizar o processo, ou trocar a porta em `.env` (`PORT=`) e no `docker-compose.yml`. |
| `permission denied` ao rodar comandos `docker` | Usuário do sistema não está no grupo `docker` (comum em Linux). | Rodar com `sudo` como paliativo, ou adicionar o usuário ao grupo `docker` (`sudo usermod -aG docker $USER`) e reiniciar a sessão. |
| Build do Docker falha em `RUN npm ci` com erro de lockfile | `package-lock.json` desatualizado em relação ao `package.json`, ou lockfile não foi commitado. | Rodar `npm install` localmente para regenerar o lock, commitar o `package-lock.json` e tentar o build novamente. |
| `docker compose up` sobe, mas `backend` fica `unhealthy` | Rota `/health` não existe, está em outra porta, ou a linha `RUN apk add --no-cache curl` do Passo 8 foi removida/esquecida em uma alteração posterior do Dockerfile. | Confirmar que a rota `/health` do Passo 7 existe e que o Dockerfile instala o `curl`; como alternativa, trocar o teste do healthcheck por um comando Node equivalente. |
| `Error: Cannot find module 'xyz'` | Dependência usada no código mas não instalada, ou `node_modules` desatualizado em relação ao `package.json`. | Rodar `npm install` para sincronizar as dependências e conferir se o pacote está listado no `package.json`. |
| `.env` não é lido pela aplicação | `require('dotenv').config()` não está na primeira linha do `src/index.js`, ou o container não recebeu `env_file`. | Confirmar que o `require('dotenv').config()` é a primeira instrução do entrypoint, e confirmar o `env_file` no `docker-compose.yml` (Passo 10). |
| Mudança em `src/` não aparece com `docker compose up --build` | Cache de camadas do Docker não foi invalidado, ou volume antigo sobrepondo os arquivos. | Rodar `docker compose build --no-cache backend` e depois `docker compose up`. |

---

## 9. Referências Técnicas

- [Node.js](https://nodejs.org/en/docs)
- [Express.js](https://expressjs.com/)
- [Docker](https://docs.docker.com/)
- [Helmet.js](https://helmetjs.github.io/)
- [express-rate-limit](https://express-rate-limit.mintlify.app/)
- [nodemon](https://github.com/remy/nodemon)
- [JWT](https://jwt.io/)

---

## 10. Dúvidas e Evoluções

Dúvidas comuns, melhorias deste setup e RFCs de evolução devem ser abertas como Pull Requests na pasta `/rfcs` deste repositório, para discussão pública do time. Este `setup-backend.md` deve ser atualizado a cada alteração relevante - é a fonte de verdade do ambiente, não a memória de quem configurou primeiro.

---

> [!Note]
> Seguindo os 12 passos da Seção 3 na ordem, o ambiente está pronto para começar a implementar as rotas do [PRD](./PRD.md).
