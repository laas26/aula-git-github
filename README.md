# Setup e Padrões do Ambiente Backend (Node.js + TypeScript)

## 📌 TL;DR

> Setup de backend Node.js + TypeScript com Docker: pastas, dependências, segurança e containers - passo a passo, do zero.

Este documento é o guia de onboarding e a fonte de verdade para configurar o ambiente de back-end do **TechStore**. Cada passo tem: **comando exato**, **arquivo/pasta afetado**, **resultado esperado** e o **porquê**.

Reflete o item do board **"Configuração do ambiente Node.js com TypeScript"** (Sprint Atual) - cada seção corresponde a um checkbox de lá.
🔗 [Ver Board do Projeto](https://seu-link-do-board-aqui)

> 🖥️ **Nota:** todos os comandos deste documento usam sintaxe **bash**. Funciona direto no Linux, macOS e WSL. No Windows "puro", use o **Git Bash** (não o PowerShell/CMD, onde alguns comandos como `mkdir -p` e `touch` não existem do mesmo jeito).

---

## 📑 Sumário

1. [Pré-Requisitos](#1-pré-requisitos)
2. [Visão Geral da Estrutura Final](#2-visão-geral-da-estrutura-final)
3. [Checklist Interativo](#3-checklist-interativo)
4. [Passo a Passo do Setup](#4-passo-a-passo-do-setup)
   - [✅ Passo 1 - Criar o diretório e iniciar o projeto](#-passo-1---criar-o-diretório-e-iniciar-o-projeto)
   - [✅ Passo 2 - Criar a estrutura básica de pastas](#-passo-2---criar-a-estrutura-básica-de-pastas)
   - [✅ Passo 3 - Instalar as dependências de desenvolvimento](#-passo-3---instalar-as-dependências-de-desenvolvimento)
   - [✅ Passo 4 - Instalar as dependências essenciais de produção](#-passo-4---instalar-as-dependências-essenciais-de-produção)
   - [✅ Passo 5 - Criar `.gitignore` e `.env.example`](#-passo-5---criar-gitignore-e-envexample)
   - [✅ Passo 6 - Criar `tsconfig.json` com configuração estrita](#-passo-6---criar-tsconfigjson-com-configuração-estrita)
   - [✅ Passo 7 - Remover o header `x-powered-by`](#-passo-7---remover-o-header-x-powered-by)
   - [✅ Passo 8 - Configurar o Helmet](#-passo-8---configurar-o-helmet)
   - [✅ Passo 9 - Criar o `Dockerfile`](#-passo-9---criar-o-dockerfile)
   - [✅ Passo 10 - Criar o `.dockerignore`](#-passo-10---criar-o-dockerignore)
   - [✅ Passo 11 - Criar/atualizar o `docker-compose.yml` na raiz do repositório](#-passo-11---criaratualizar-o-docker-composeyml-na-raiz-do-repositório)
   - [✅ Passo 12 - Testar se o ambiente sobe com Docker Compose](#-passo-12---testar-se-o-ambiente-sobe-com-docker-compose)
   - [✅ Passo 13 - Criar o `README.md` do backend](#-passo-13---criar-o-readmemd-do-backend)
5. [Padrões de Código (ESLint, Prettier, Husky)](#5-padrões-de-código-eslint-prettier-husky)
6. [Comandos Úteis (resumo)](#6-comandos-úteis-resumo)
7. [Padrões de Commit e Branch](#7-padrões-de-commit-e-branch)
8. [Observabilidade](#8-observabilidade)
9. [Referências Técnicas](#9-referências-técnicas)
10. [Dúvidas e Evoluções](#10-dúvidas-e-evoluções)

---

## 1. Pré-Requisitos

Antes de começar, confirme no terminal que você tem tudo instalado:

```bash
node -v    # precisa ser >= v20.x
npm -v     # precisa ser >= v9.x
docker -v  # opcional, mas recomendado
docker compose version  # opcional
```

Se algum comando não existir, instale antes de continuar:
- Node.js: https://nodejs.org/en/download (a instalação já vem com npm)
- Docker Desktop: https://docs.docker.com/get-docker/ (já vem com Docker Compose)

> 💡 Se você usa múltiplos projetos Node com versões diferentes, considere usar `nvm` (Node Version Manager) para trocar de versão facilmente.

---

## 2. Visão Geral da Estrutura Final

Depois de seguir todos os passos, seu projeto deve ficar assim:

```
seu-repositorio/                  ← raiz do repositório Git
├── docker-compose.yml            ← orquestra backend + frontend
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── middlewares/
│   │   └── index.ts              ← ponto de entrada da aplicação
│   ├── tests/
│   ├── data/                     ← arquivos JSON de persistência (RN04 do PRD)
│   ├── .env                      ← NUNCA vai pro Git (segredos reais)
│   ├── .env.example              ← vai pro Git (modelo sem valores reais)
│   ├── .gitignore
│   ├── .dockerignore             ← controla o que NÃO entra na imagem Docker
│   ├── package.json
│   ├── package-lock.json
│   ├── tsconfig.json
│   ├── Dockerfile
│   └── README.md
└── frontend/                     ← (fora do escopo deste documento)
```

**Por quê:** `docker-compose.yml` fica na raiz porque ele orquestra *mais de um* serviço (backend, frontend, e futuramente banco/cache) - se ficasse dentro de `backend/`, ele só "enxergaria" o backend. Já o `.dockerignore` fica dentro de `backend/`, porque é lá que está o **contexto de build** do Docker (mais detalhes no Passo 10).

---

## 3. Checklist Interativo

Copie esta lista para o seu board/PR e marque conforme avança (troque `[ ]` por `[x]`):

- [ ] Criar o diretório `backend` e iniciar o projeto (`npm init -y`)
- [ ] Criar a estrutura básica de pastas (`controllers`, `models`, `routes`, `middlewares`, `tests`, `data`)
- [ ] Instalar as dependências de desenvolvimento
- [ ] Instalar as dependências essenciais de produção
- [ ] Criar `.gitignore` e `.env.example` documentando as variáveis obrigatórias
- [ ] Criar `tsconfig.json` com `strict: true` e `esModuleInterop: true`
- [ ] Remover o header `x-powered-by`
- [ ] Configurar o Helmet e os cabeçalhos seguros no Express
- [ ] Criar o `Dockerfile` (Node Alpine, non-root, porta 3000)
- [ ] Criar o `.dockerignore` no contexto de build
- [ ] Criar/atualizar o `docker-compose.yml` na raiz, interligando backend + frontend
- [ ] Testar localmente se `docker compose up --build` sobe os dois ambientes com sucesso
- [ ] Criar o `README.md` do backend com apresentação do projeto e como rodar

---

## 4. Passo a Passo do Setup

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

⚠️ **Atenção:** a partir daqui, todo comando do restante deste documento assume que você está **dentro da pasta `backend/`**, a não ser que eu diga o contrário (ex: os comandos do `docker-compose.yml`, que rodam na raiz do repositório).

---

### ✅ Passo 2 - Criar a estrutura básica de pastas

**Comando (bash, rodando de dentro de `backend/`):**
```bash
mkdir -p src/controllers src/models src/routes src/middlewares tests data
touch src/index.ts
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

**Resultado esperado:** rodando `ls -R` dentro de `backend/`, você vê todas as pastas acima, e `src/index.ts` existe (vazio por enquanto).

---

### ✅ Passo 3 - Instalar as dependências de desenvolvimento

**Comando (bash):**
```bash
npm install --save-dev typescript @types/node @types/express ts-node-dev jest ts-jest @types/jest supertest eslint prettier
```

**O que é cada pacote:**
| Pacote | Para que serve |
|---|---|
| `typescript` | O compilador TS em si |
| `@types/node` | Tipagem das APIs do Node (fs, path, etc.) |
| `@types/express` | Tipagem do Express |
| `ts-node-dev` | Roda o TypeScript direto (sem precisar compilar toda hora) e reinicia sozinho quando você salva um arquivo |
| `jest` + `ts-jest` + `@types/jest` | Framework de testes automatizados com suporte a TypeScript |
| `supertest` | Testar rotas HTTP (ex: simular um `POST /login`) sem precisar subir o servidor de verdade |
| `eslint` + `prettier` | Padronização e formatação de código |

**Resultado esperado:** essas dependências aparecem em `package.json`, dentro de `"devDependencies"`.

---

### ✅ Passo 4 - Instalar as dependências essenciais de produção

**Comando (bash):**
```bash
npm install express jsonwebtoken bcrypt dotenv helmet
```

**O que é cada pacote (e onde ele conecta com o PRD):**
| Pacote | Para que serve | Relação com o PRD |
|---|---|---|
| `express` | Framework web/HTTP | Base de tudo |
| `jsonwebtoken` | Gerar/validar tokens JWT | Autenticação (RF02) |
| `bcrypt` | Fazer hash de senha | RN02 - senha **nunca** em texto plano |
| `dotenv` | Ler variáveis do arquivo `.env` no código | Segredos fora do código (Passo 5) |
| `helmet` | Configurar cabeçalhos HTTP seguros automaticamente | RNF02 - segurança da aplicação |

**Resultado esperado:** essas dependências aparecem em `package.json`, dentro de `"dependencies"` (não `devDependencies` - elas são necessárias em produção, não só em desenvolvimento).

> 📌 **Nota para mais adiante:** o PRD (RF02/RF03, RNF02) também pede *rate limiting* em login e recuperação de senha. Isso normalmente é feito com o pacote `express-rate-limit`, mas como ele é usado dentro de um middleware específico (não faz parte do "esqueleto" do ambiente), a instalação dele fica para quando a rota de login/recuperação for implementada - não é um passo deste setup inicial.

---

### ✅ Passo 5 - Criar `.gitignore` e `.env.example`

**Comando (bash):**
```bash
touch .gitignore .env .env.example
```

**Conteúdo do `backend/.gitignore`:**
```
node_modules/
dist/
.env
*.log
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
- `.env.example` **fica** no Git - ele documenta *quais* variáveis existem, sem revelar os valores, para que qualquer pessoa do time saiba o que precisa configurar ao clonar o projeto.
- 🚨 Nunca escreva uma chave secreta ou senha padrão diretamente no código (hardcoded), nem mesmo "só para teste" ou como valor de fallback - é um dos erros de segurança mais comuns.

**Resultado esperado:** ao rodar `git status`, o arquivo `.env` **não aparece** como arquivo a ser commitado, mas `.env.example` e `.gitignore` aparecem.

---

### ✅ Passo 6 - Criar `tsconfig.json` com configuração estrita

**Comando (bash):**
```bash
npx tsc --init
```

Isso gera um `tsconfig.json` grande, cheio de opções comentadas. Abra o arquivo e garanta que estas duas opções estejam **ativas** (sem `//` na frente):

```json
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true
  }
}
```

**Por quê:**
- `"strict": true` → liga um conjunto de verificações (como proibir `any` implícito, checar `null`/`undefined`) que pegam erros de tipagem **em tempo de compilação**, antes de virarem bug em produção.
- `"esModuleInterop": true` → permite escrever `import express from 'express'` normalmente, mesmo quando a biblioteca foi escrita no padrão antigo do Node (CommonJS/`module.exports`). Sem isso, alguns imports quebram ou exigem uma sintaxe estranha.

**Resultado esperado:** rodando `npx tsc --noEmit` (só checa tipos, sem gerar arquivo), nenhum erro aparece (mesmo com o projeto quase vazio).

---

### ✅ Passo 7 - Remover o header `x-powered-by`

**Onde:** `src/index.ts` (ou o arquivo onde você cria a instância do Express)

```typescript
import express from 'express'

const app = express()
app.disable('x-powered-by')
```

**Por quê:** por padrão, o Express envia um header HTTP `X-Powered-By: Express` em toda resposta. Isso entrega de graça, para qualquer atacante, qual framework/stack você usa - informação que facilita a busca por vulnerabilidades conhecidas daquele framework. `app.disable('x-powered-by')` remove esse header.

**Resultado esperado:** depois de subir o servidor, uma requisição de teste (`curl -I http://localhost:3000`) **não** mostra o header `X-Powered-By` na resposta.

---

### ✅ Passo 8 - Configurar o Helmet

**Onde:** mesmo arquivo `src/index.ts`, logo depois do Passo 7

```typescript
import helmet from 'helmet'

app.use(helmet())
```

**Por quê:** o `helmet()` configura de uma vez só um conjunto de cabeçalhos HTTP de segurança recomendados (ex: `Content-Security-Policy`, `X-Frame-Options`, `Strict-Transport-Security`), reduzindo a superfície de ataque contra vulnerabilidades comuns como clickjacking e MIME sniffing. É basicamente "boas práticas de segurança de header" empacotadas em uma linha, em vez de você configurar cada header manualmente.

**Resultado esperado:** o mesmo teste `curl -I http://localhost:3000` agora mostra vários headers novos de segurança na resposta.

---

### ✅ Passo 9 - Criar o `Dockerfile`

**Arquivo:** `backend/Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
EXPOSE 3000
USER node
CMD ["node", "dist/index.js"]
```

**Por quê, linha a linha:**
| Linha | Por quê |
|---|---|
| `FROM node:20-alpine` | Imagem Node oficial baseada em Alpine Linux - bem mais leve que a imagem padrão, reduz superfície de ataque e tempo de build |
| `WORKDIR /app` | Define a pasta de trabalho dentro do container |
| `COPY package*.json ./` + `RUN npm ci` | Copia só os arquivos de dependência primeiro e instala - isso aproveita o cache do Docker: se o código mudar mas as dependências não, o build fica bem mais rápido |
| `COPY . .` | Copia o resto do código |
| `RUN npm run build` | Compila o TypeScript para JavaScript (pasta `dist/`) |
| `EXPOSE 3000` | Documenta que o container escuta na porta 3000 |
| `USER node` | Troca para um usuário **não-root** dentro do container - se alguém conseguir explorar uma falha na aplicação, o dano fica limitado, sem privilégios de root no container |
| `CMD [...]` | Comando que roda quando o container inicia: executa o JS já compilado |

**Resultado esperado:** rodando `docker build -t techstore-backend .` dentro de `backend/`, o build termina sem erro.

---

### ✅ Passo 10 - Criar o `.dockerignore`

**Arquivo:** `backend/.dockerignore` - **no mesmo nível do `Dockerfile`, dentro de `backend/`**

> ⚠️ Diferente do que se costuma imaginar, o `.dockerignore` **não** fica na raiz do repositório por padrão - ele fica na raiz do **contexto de build** do Docker. O contexto de build é a pasta que você aponta em `build:` no `docker-compose.yml` (Passo 11), que neste projeto é `./backend`. Por isso o arquivo vai dentro de `backend/`, e não na raiz do repositório.

**Comando (bash):**
```bash
touch .dockerignore
```

**Conteúdo do `backend/.dockerignore`:**
```
node_modules
npm-debug.log
dist
.git
.gitignore
.env
.env.example
*.md
tests
coverage
```

**Por quê:**
- Quando o Dockerfile roda `COPY . .`, ele copia **tudo** da pasta `backend/` para dentro da imagem - inclusive coisas que não deveriam ir, como `node_modules` (será reinstalado do zero com `npm ci`, então copiar é desperdício), `.git` (histórico do repositório, não serve pra nada em produção) e, mais importante, o `.env` (que pode conter segredos reais - nunca deveria parar dentro de uma imagem Docker, que pode ser compartilhada ou enviada a um registry).
- Ignorar essas pastas/arquivos deixa o build **mais rápido** (menos dados para copiar) e a **imagem final menor e mais segura**.

**Resultado esperado:** rodando `docker build -t techstore-backend .` novamente, o build fica visivelmente mais rápido (menos arquivos sendo copiados no passo `COPY . .`), e inspecionando a imagem gerada (`docker run --rm -it techstore-backend sh` e depois `ls -la`) você **não** encontra `node_modules` duplicado, `.git` nem `.env` lá dentro.

---

### ✅ Passo 11 - Criar/atualizar o `docker-compose.yml` na raiz do repositório

**Arquivo:** `docker-compose.yml` - **na raiz do repositório, fora da pasta `backend/`**

```yaml
version: '3.8'
services:
  backend:
    build: ./backend
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
- `build: ./backend` → aponta para a pasta do backend (caminho relativo à raiz, onde este arquivo está). Essa é a pasta que vira o **contexto de build** - e é por isso que o `.dockerignore` do Passo 10 fica dentro dela.
- `volumes: ./backend/data:/app/data` → garante que os arquivos JSON de persistência (`backend/data/`) sobrevivem mesmo se o container for recriado.
- `healthcheck` → o Docker passa a monitorar se `/health` responde; se a aplicação travar, o Docker sabe que o container está "não saudável" (pressupõe que existe uma rota `GET /health` no Express - se ainda não existir, crie uma que só responde `200 OK`).
- O serviço `frontend` está incluído como esqueleto - ajuste o `build`/porta quando o front-end estiver com Dockerfile próprio.

**Resultado esperado:** o comando `docker compose config` (rodado na raiz) não mostra erro de sintaxe.

---

### ✅ Passo 12 - Testar se o ambiente sobe com Docker Compose

**Comando (bash, na raiz do repositório):**
```bash
docker compose up --build
```

**O que observar:**
- O build de `backend` (e `frontend`, se já existir) termina sem erro.
- O terminal mostra o log do servidor Express subindo (ex: `Servidor rodando na porta 3000`).
- Abrindo `http://localhost:3000/health` no navegador (ou `curl http://localhost:3000/health`), você recebe uma resposta (não erro de conexão recusada).

**Para parar:** `Ctrl + C`, depois `docker compose down` para remover os containers.

**Resultado esperado:** os dois ambientes (back e front) sobem juntos com um único comando, sem erro.

---

### ✅ Passo 13 - Criar o `README.md` do backend

**Arquivo:** `backend/README.md`

Estrutura mínima recomendada (padrão de mercado):

```markdown
# TechStore - Backend

API do TechStore: e-commerce com gestão de catálogo, carrinho e pagamento via PIX.

## Tech Stack
- Node.js 20 + Express + TypeScript
- Persistência: arquivos JSON (MVP) - ver `RN04` no PRD
- Autenticação: sessão via cookie + JWT
- Segurança: Helmet, rate limiting, bcrypt

## Como rodar localmente

\`\`\`bash
cp .env.example .env   # preencha os valores reais
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
| `npm run dev` | Sobe em modo desenvolvimento com reload automático |
| `npm run build` | Compila TypeScript para `dist/` |
| `npm test` | Roda a suíte de testes (Jest) |
| `npm run lint` | Checa padrões de código (ESLint) |

## Estrutura de pastas
Ver seção 2 do `setup-backend.md`.
```

**Por quê:** todo repositório de mercado tem um README que responde, em 30 segundos de leitura, três perguntas: *o que é isso*, *qual stack* e *como eu rodo na minha máquina*. Sem isso, cada pessoa nova no time perde tempo perguntando o óbvio.

**Resultado esperado:** alguém que nunca viu o projeto consegue, só lendo o README, rodar a aplicação localmente sem perguntar nada a ninguém.

---

## 5. Padrões de Código (ESLint, Prettier, Husky)

**Instalação adicional (bash):**
```bash
npm install --save-dev eslint-config-prettier eslint-plugin-prettier husky lint-staged
npx husky install
```

**Scripts recomendados no `package.json`:**
```json
{
  "scripts": {
    "dev": "ts-node-dev src/index.ts",
    "build": "tsc",
    "test": "jest",
    "lint": "eslint 'src/**/*.{js,ts}'",
    "format": "prettier --write 'src/**/*.{js,ts,json,md}'",
    "prepare": "husky install"
  }
}
```

**Por quê:** `husky` + `lint-staged` fazem o lint/format rodar **automaticamente antes de cada commit**, pegando erro de estilo antes de chegar no Pull Request - em vez de descobrir só quando o CI falhar.

---

## 6. Comandos Úteis (resumo)

| Comando | O que faz |
|---|---|
| `npm run dev` | Start em modo desenvolvimento (`ts-node-dev`) |
| `npm run build` | Build do projeto (compila TS → JS) |
| `npm test` | Roda testes automatizados |
| `npm run lint` | Checa lint do código |
| `docker compose up --build` | Sobe backend + frontend juntos |

---

## 7. Padrões de Commit e Branch

- Seguir **Conventional Commits** (em português, conforme decisão do time), ex: `feat: adiciona rota de login`, `fix: corrige validação de e-mail`.
- Pull Requests obrigatórios, aprovados antes de mergear (squash merge).
- Pipelines de CI (GitHub Actions e GitLab CI, já que o repositório é espelhado nas duas plataformas) rodando build, lint e testes a cada PR.
- Husky + Lint-Staged rodando localmente antes do push, para pegar problema antes mesmo de abrir o PR.

---

## 8. Observabilidade

- Expor métricas básicas em formato Prometheus, quando possível (RNF04 do PRD).
- Logs da aplicação **nunca** devem conter dados sensíveis (senha, token, dado pessoal do cliente) - nem mesmo em nível de log de debug.

---

## 9. Referências Técnicas

- [Node.js](https://nodejs.org/en/docs)
- [Express.js](https://expressjs.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [Docker](https://docs.docker.com/)
- [Helmet.js](https://helmetjs.github.io/)
- [JWT](https://jwt.io/)

---

## 10. Dúvidas e Evoluções

Dúvidas comuns, melhorias deste setup e RFCs de evolução devem ser abertas como Pull Requests na pasta `/rfcs` deste repositório, para discussão pública do time. Atualize este `setup-backend.md` a cada alteração relevante - ele é a fonte de verdade do ambiente, não a memória de quem configurou primeiro.

---

**Resumo:** este documento é a fonte de verdade e o guia de onboarding rápido para configuração, boas práticas e evolução do ambiente de back-end do TechStore. Se você seguiu os 13 passos da Seção 4 na ordem, o ambiente está pronto para começar a implementar as rotas do PRD.
