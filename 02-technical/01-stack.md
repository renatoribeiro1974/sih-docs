---
title: Stack Tecnológica
parent: Documentação Técnica
nav_order: 1
---

# 1. Stack Tecnológica

**SIH - Supervisão Industrial Halal**

---

## 1.1 Visão Geral

O SIH utiliza uma stack moderna baseada em TypeScript end-to-end, com NestJS no backend e React no frontend. O sistema e containerizado com Docker para desenvolvimento local e implantado na AWS para produção.

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND                             │
│  React 19 + Vite 7 + Tailwind CSS 4 + shadcn/ui            │
├─────────────────────────────────────────────────────────────┤
│                        BACKEND                              │
│  NestJS 11 + Prisma 7 + Passport.js + Swagger              │
├──────────────────────────┬──────────────────────────────────┤
│      PostgreSQL 16       │           Redis 7                │
│   (dados persistentes)   │    (cache + rate limiting)       │
└──────────────────────────┴──────────────────────────────────┘
```

---

## 1.2 Backend

### Framework e Runtime

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **NestJS** | 11.x | Framework principal do backend (modular, com DI) |
| **Node.js** | 22+ | Runtime JavaScript server-side |
| **TypeScript** | 5.7.x | Tipagem estatica em todo o backend |
| **Express** | via @nestjs/platform-express | HTTP server subjacente ao NestJS |

### ORM e Banco de Dados

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **Prisma** | 7.x | ORM principal (schema-first, migração, seed) |
| **@prisma/client** | 7.2.x | Cliente gerado para queries tipadas |
| **@prisma/adapter-pg** | 7.2.x | Adaptador PrismaPg para conexão nativa com PostgreSQL |
| **pg** | 8.17.x | Driver PostgreSQL nativo (usado pelo adapter) |
| **PostgreSQL** | 16+ | Banco de dados relacional principal |
| **Redis** | 7+ | Cache de sessão e raté limiting |
| **ioredis** | 5.9.x | Cliente Redis para Node.js |

#### Prisma 7 - Configuração Específica

O SIH utiliza Prisma 7 com as seguintes particularidades:

- **Provider**: `prisma-client-js` (gerador padrão)
- **Adapter PrismaPg**: Usa `@prisma/adapter-pg` para conexão direta via driver `pg`, sem o query engine binario do Prisma. Isso resulta em deploys menores e melhor compatibilidade com containers.
- **Arquivo de configuração**: `prisma.config.ts` (TypeScript nativo, substituindo configurações do `schema.prisma`)
- **Seed**: Executado via `tsx prisma/seed.ts` (configurado no `package.json` na chave `prisma.seed`)
- **Extensões do PostgreSQL**: `uuid-ossp`, `pgcrypto`, `pg_trgm` (inicializadas via script Docker)

### Autenticação e Segurança

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **@nestjs/jwt** | 11.x | Módulo JWT para NestJS |
| **@nestjs/passport** | 11.x | Integração Passport.js com NestJS |
| **passport** | 0.7.x | Middleware de autenticação |
| **passport-jwt** | 4.0.x | Estratégia JWT para Passport |
| **bcrypt** | 6.x | Hash de senhas |
| **cookie-parser** | 1.4.x | Parse de cookies nas requisicoes |

#### Estratégia JWT (v1.0)

Na v1.0, o SIH usa autenticação self-contained (não depende do HalalSphere):

| Algoritmo | Variavel | Observação |
|-----------|----------|------------|
| **HS256** (simetrico) | `JWT_SECRET` | Backend SIH emite e válida tokens via `POST /auth/login` |

### Validação e Documentação

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **class-validator** | 0.14.x | Decorators de validação em DTOs |
| **class-transformer** | 0.5.x | Transformação de objetos (serialization) |
| **zod** | 4.3.x | Validação de schemas (complementar) |
| **zod-válidation-error** | 5.x | Formatação de erros Zod |
| **@nestjs/swagger** | 11.2.x | Documentação OpenAPI/Swagger automática |

### Infraestrutura e Observabilidade

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **@nestjs/terminus** | 11.x | Health checks (banco, Redis, app) |
| **@nestjs/config** | 4.x | Gerênciamento de configuração por ambiente |
| **@aws-sdk/client-secrets-manager** | 3.x | Acesso ao AWS Secrets Manager |
| **@aws-sdk/client-ssm** | 3.x | Acesso ao AWS Systems Manager Parameter Store |
| **dotenv** | 17.x | Variaveis de ambiente em desenvolvimento |
| **reflect-metadata** | 0.2.x | Suporte a decorators TypeScript |
| **rxjs** | 7.8.x | Programação reativa (core do NestJS) |

---

## 1.3 Frontend

### Framework e Build

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **React** | 19.2.x | Biblioteca de UI principal |
| **React DOM** | 19.2.x | Renderização DOM |
| **Vite** | 7.2.x | Build tool e dev server |
| **@vitejs/plugin-react** | 5.1.x | Plugin Vite para React (Fast Refresh) |
| **TypeScript** | ~5.9.x | Tipagem estatica em todo o frontend |

### Estilização e Componentes

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **Tailwind CSS** | 3.4.x (v4 config) | Framework CSS útility-first |
| **shadcn/ui** | - | Componentes UI baseados em Radix (copiados para o projeto) |
| **@radix-ui/react-dialog** | 1.1.x | Componente Dialog acessível |
| **@radix-ui/react-alert-dialog** | 1.1.x | Componente Alert Dialog acessível |
| **@radix-ui/react-dropdown-menu** | 2.1.x | Componente Dropdown Menu acessível |
| **@radix-ui/react-tabs** | 1.1.x | Componente Tabs acessível |
| **class-variance-authority** | 0.7.x | Variantes de estilos para componentes |
| **clsx** | 2.1.x | Concatenação condicional de classes CSS |
| **tailwind-merge** | 2.6.x | Merge inteligente de classes Tailwind |
| **lucide-react** | 0.468.x | Biblioteca de icones (Lucide) |

### Roteamento e Estado

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **react-router-dom** | 7.2.x | Roteamento SPA |
| **@tanstack/react-query** | 5.62.x | Gerênciamento de estado do servidor (cache, fetch, sync) |
| **axios** | 1.7.x | Cliente HTTP para chamadas a API |

### Formulários e Validação

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **react-hook-form** | 7.54.x | Gerênciamento de formulários performatico |
| **@hookform/resolvers** | 3.10.x | Integração de schemas de validação (Zod) |
| **zod** | 3.23.x | Validação de schemas no frontend |

### UX e Feedback

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **react-hot-toast** | 2.6.x | Notificações toast |

### PWA

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **vite-plugin-pwa** | 0.21.x | Geração de manifest.json e Service Worker via Vite |

---

## 1.4 Ferramentas de Desenvolvimento

### Backend DevDependencies

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **@nestjs/cli** | 11.x | CLI do NestJS (build, generate, etc.) |
| **@nestjs/schematics** | 11.x | Templates de código NestJS |
| **@nestjs/testing** | 11.x | Útilitarios de teste NestJS |
| **Jest** | 30.x | Framework de testes unitários |
| **ts-jest** | 29.x | Transformador TypeScript para Jest |
| **supertest** | 7.x | Testes de integração HTTP |
| **ESLint** | 9.x | Linter de código |
| **eslint-config-prettier** | 10.x | Desativa regras ESLint que conflitam com Prettier |
| **eslint-plugin-prettier** | 5.x | Executa Prettier como regra ESLint |
| **typescript-eslint** | 8.x | Regras ESLint para TypeScript |
| **Prettier** | 3.4.x | Formatação de código |
| **tsx** | 4.21.x | Executor de TypeScript (usado para seed do Prisma) |
| **ts-node** | 10.x | Executor TypeScript para Node.js |
| **ts-loader** | 9.x | Loader TypeScript para Webpack (build NestJS) |
| **tsconfig-paths** | 4.x | Resolve path aliases do tsconfig |
| **source-map-support** | 0.5.x | Source maps para debugging |

### Frontend DevDependencies

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **ESLint** | 9.x | Linter de código |
| **eslint-plugin-react-hooks** | 7.x | Regras de lint para React Hooks |
| **eslint-plugin-react-refresh** | 0.4.x | Regras para React Fast Refresh |
| **typescript-eslint** | 8.x | Regras ESLint para TypeScript |
| **PostCSS** | 8.5.x | Processador CSS (pipeline Tailwind) |
| **Autoprefixer** | 10.x | Adiciona vendor prefixes automáticamente |
| **Terser** | 5.x | Minificador JavaScript para build de produção |

---

## 1.5 Infraestrutura

### Containerização

| Tecnologia | Versão | Proposito |
|------------|--------|-----------|
| **Docker** | 24+ | Containerização de serviços |
| **Docker Compose** | 2.x | Orquestração de containers locais |

### Containers Docker (Desenvolvimento)

| Container | Imagem | Porta Host | Porta Interna |
|-----------|--------|:----------:|:-------------:|
| `sih-postgres` | `postgres:16-alpine` | 5433 | 5432 |
| `sih-redis` | `redis:7-alpine` | 6380 | 6379 |
| `sih-backend` | Build local (profile: full) | 3334 | 3334 |

### AWS (Produção)

| Serviço | Proposito |
|---------|-----------|
| **ECS (Fargate)** | Execução de containers do backend |
| **RDS (PostgreSQL)** | Banco de dados gerênciado |
| **ElastiCache (Redis)** | Cache gerênciado |
| **S3** | Armazenamento de arquivos estaticos e evidencias |
| **CloudFront** | CDN para frontend e assets |
| **Secrets Manager** | Armazenamento seguro de credenciais |
| **SSM Parameter Store** | Configurações de ambiente |

---

## 1.6 Tabela Consolidada de Dependências

### Backend - dependencies

| Pacote | Versão | Categoria |
|--------|--------|-----------|
| `@aws-sdk/client-secrets-manager` | ^3.969.0 | Infraestrutura AWS |
| `@aws-sdk/client-ssm` | ^3.969.0 | Infraestrutura AWS |
| `@nestjs/common` | ^11.0.1 | Framework Core |
| `@nestjs/config` | ^4.0.2 | Configuração |
| `@nestjs/core` | ^11.0.1 | Framework Core |
| `@nestjs/jwt` | ^11.0.2 | Autenticação |
| `@nestjs/passport` | ^11.0.5 | Autenticação |
| `@nestjs/platform-express` | ^11.0.1 | HTTP Server |
| `@nestjs/swagger` | ^11.2.5 | Documentação API |
| `@nestjs/terminus` | ^11.0.0 | Health Check |
| `@prisma/adapter-pg` | ^7.2.0 | ORM - Adapter |
| `@prisma/client` | ^7.2.0 | ORM - Client |
| `bcrypt` | ^6.0.0 | Segurança |
| `class-transformer` | ^0.5.1 | Validação |
| `class-validator` | ^0.14.3 | Validação |
| `cookie-parser` | ^1.4.7 | HTTP |
| `dotenv` | ^17.3.1 | Configuração |
| `ioredis` | ^5.9.1 | Cache (Redis) |
| `passport` | ^0.7.0 | Autenticação |
| `passport-jwt` | ^4.0.1 | Autenticação |
| `pg` | ^8.17.0 | Driver PostgreSQL |
| `prisma` | ^7.2.0 | ORM - CLI/Engine |
| `reflect-metadata` | ^0.2.2 | Runtime |
| `rxjs` | ^7.8.1 | Runtime |
| `zod` | ^4.3.5 | Validação |
| `zod-validation-error` | ^5.0.0 | Validação |

### Backend - devDependencies

| Pacote | Versão | Categoria |
|--------|--------|-----------|
| `@eslint/eslintrc` | ^3.2.0 | Linting |
| `@eslint/js` | ^9.18.0 | Linting |
| `@nestjs/cli` | ^11.0.0 | Tooling |
| `@nestjs/schematics` | ^11.0.0 | Tooling |
| `@nestjs/testing` | ^11.0.1 | Testes |
| `@types/bcrypt` | ^6.0.0 | Tipagem |
| `@types/cookie-parser` | ^1.4.10 | Tipagem |
| `@types/express` | ^5.0.0 | Tipagem |
| `@types/jest` | ^30.0.0 | Tipagem |
| `@types/node` | ^22.10.7 | Tipagem |
| `@types/passport-jwt` | ^4.0.1 | Tipagem |
| `@types/pg` | ^8.16.0 | Tipagem |
| `@types/supertest` | ^6.0.2 | Tipagem |
| `eslint` | ^9.18.0 | Linting |
| `eslint-config-prettier` | ^10.0.1 | Linting |
| `eslint-plugin-prettier` | ^5.2.2 | Linting |
| `globals` | ^16.0.0 | Linting |
| `jest` | ^30.0.0 | Testes |
| `prettier` | ^3.4.2 | Formatação |
| `source-map-support` | ^0.5.21 | Debug |
| `supertest` | ^7.0.0 | Testes |
| `ts-jest` | ^29.2.5 | Testes |
| `ts-loader` | ^9.5.2 | Build |
| `ts-node` | ^10.9.2 | Runtime |
| `tsconfig-paths` | ^4.2.0 | Build |
| `tsx` | ^4.21.0 | Runtime (Seed) |
| `typescript` | ^5.7.3 | Linguagem |
| `typescript-eslint` | ^8.20.0 | Linting |

### Frontend - dependencies

| Pacote | Versão | Categoria |
|--------|--------|-----------|
| `@hookform/resolvers` | ^3.10.0 | Formulários |
| `@radix-ui/react-alert-dialog` | ^1.1.15 | Componentes UI |
| `@radix-ui/react-dialog` | ^1.1.15 | Componentes UI |
| `@radix-ui/react-dropdown-menu` | ^2.1.16 | Componentes UI |
| `@radix-ui/react-tabs` | ^1.1.13 | Componentes UI |
| `@tanstack/react-query` | ^5.62.0 | Estado/Cache |
| `axios` | ^1.7.0 | HTTP Client |
| `class-variance-authority` | ^0.7.1 | Estilização |
| `clsx` | ^2.1.1 | Estilização |
| `lucide-react` | ^0.468.0 | Icones |
| `react` | ^19.2.0 | Framework UI |
| `react-dom` | ^19.2.0 | Framework UI |
| `react-hook-form` | ^7.54.1 | Formulários |
| `react-hot-toast` | ^2.6.0 | UX/Feedback |
| `react-router-dom` | ^7.2.0 | Roteamento |
| `tailwind-merge` | ^2.6.0 | Estilização |
| `zod` | ^3.23.8 | Validação |

### Frontend - devDependencies

| Pacote | Versão | Categoria |
|--------|--------|-----------|
| `@eslint/js` | ^9.39.1 | Linting |
| `@types/node` | ^24.10.0 | Tipagem |
| `@types/react` | ^19.2.2 | Tipagem |
| `@types/react-dom` | ^19.2.2 | Tipagem |
| `@vitejs/plugin-react` | ^5.1.0 | Build |
| `autoprefixer` | ^10.4.20 | CSS |
| `eslint` | ^9.39.1 | Linting |
| `eslint-plugin-react-hooks` | ^7.0.1 | Linting |
| `eslint-plugin-react-refresh` | ^0.4.24 | Linting |
| `globals` | ^16.5.0 | Linting |
| `postcss` | ^8.5.1 | CSS |
| `tailwindcss` | ^3.4.17 | CSS |
| `terser` | ^5.46.0 | Build |
| `typescript` | ~5.9.3 | Linguagem |
| `typescript-eslint` | ^8.46.3 | Linting |
| `vite` | ^7.2.2 | Build |
| `vite-plugin-pwa` | ^0.21.1 | PWA |

---

## 1.7 Observações sobre Versões

### TypeScript

- **Backend**: TypeScript 5.7.x (versão estavel para NestJS 11)
- **Frontend**: TypeScript ~5.9.x (versão mais recente, compatível com Vite 7)
- Ambos os projetos usam `strict: true` no `tsconfig.json`

### Zod

- **Backend**: Zod 4.x (versão mais recente, com breaking changes do v3)
- **Frontend**: Zod 3.x (versão estavel, compatível com `@hookform/resolvers`)
- A diferença de versões e intencional: o frontend depende de resolvers que ainda não suportam Zod 4

### ESLint

- Ambos os projetos usam ESLint 9.x com flat config (novo formato de configuração)
- O backend usa `eslint-plugin-prettier` para integração com Prettier
- O frontend usa plugins específicos para React Hooks e React Refresh
