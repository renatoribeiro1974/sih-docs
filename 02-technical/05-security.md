---
title: Segurança
parent: Documentação Técnica
nav_order: 5
---

# 5. Segurança

---

## 5.1 Visão Geral

Na v1.0, o SIH implementa **autenticação própria (self-contained)** — não depende do HalalSphere nem de qualquer sistema externo para autenticar usuários.

```
Frontend SIH ──(POST /auth/login)──> Backend SIH ──(bcrypt + JWT HS256)──> Token
```

---

## 5.2 Autenticação JWT

### 5.2.1 Fluxo de Autenticação

1. O usuário faz login diretamente no **frontend do SIH**
2. O frontend envia `POST /auth/login` com `{ email, password }` para o backend SIH
3. O backend válida a senha com **bcrypt** (salt rounds 10)
4. O backend SIH emite um token JWT assinado com **HS256** usando `JWT_SECRET`
5. O frontend armazena o token em `localStorage` (`sih_token`)
6. Em cada requisicao, o token e enviado no header `Authorization: Bearer <token>`
7. O `JwtAuthGuard` bloqueia requisicoes sem token válido
8. O `RolesGuard` verifica se o role do usuário tem permissão para o endpoint

### 5.2.2 Algoritmo de Assinatura

| Algoritmo | Variavel de Ambiente | Descrição |
|-----------|---------------------|-----------|
| **HS256** (simetrico) | `JWT_SECRET` | Chave secreta usada para assinar e validar tokens. Mínimo 32 caracteres. |

| Variavel | Descrição | Exemplo |
|----------|-----------|---------|
| `JWT_SECRET` | Chave secreta HS256 | `dev-secret-key-minimum-32-chars...` |
| `JWT_EXPIRES_IN` | Válidade do token | `7d` |

### 5.2.3 JwtPayload

Interface do payload do token JWT emitido pelo SIH:

```typescript
interface JwtPayload {
  sub: string;        // ID do usuario (SupervisorProfile.id)
  email: string;      // Email do usuario
  name: string;       // Nome completo
  role: string;       // Role: admin | coordenador | supervisor | operador
  iat?: number;       // Issued at (automatico)
  exp?: number;       // Expiration (automatico)
}
```

Após validação, o payload é transformado em `AuthenticatedUser`:

```typescript
interface AuthenticatedUser {
  userId: string;     // Mapeado de JwtPayload.sub
  email: string;
  name: string;
  role: string;
}
```

---

## 5.3 Guards

Ambos os guards são registrados **globalmente** via `APP_GUARD` no `AuthModule`, o que significa que se aplicam a **todos os endpoints** automáticamente.

### 5.3.1 JwtAuthGuard (Global)

**Arquivo**: `src/auth/guards/jwt-auth.guard.ts`

- Registrado como `APP_GUARD` - se aplica a todos os endpoints
- Estende `AuthGuard('jwt')` do Passport
- Verifica se o endpoint possui o metadata `isPublic`
- Se `@Public()` estiver presente, permite acesso sem token
- Caso contrario, exige token JWT válido no header `Authorization`

```typescript
{
  provide: APP_GUARD,
  useClass: JwtAuthGuard,
}
```

### 5.3.2 RolesGuard (Global)

**Arquivo**: `src/auth/guards/roles.guard.ts`

- Registrado como `APP_GUARD` - se aplica a todos os endpoints
- Verifica o metadata `roles` definido pelo decorator `@Roles()`
- Se nenhum role for especificado no endpoint, permite acesso (qualquer usuário autenticado)
- Se roles forem especificados, verifica se `user.role` está na lista permitida
- Retorna `false` (403 Forbidden) se o usuário não tiver o role necessário

```typescript
{
  provide: APP_GUARD,
  useClass: RolesGuard,
}
```

---

## 5.4 Decorators

### 5.4.1 @Public()

**Arquivo**: `src/auth/decorators/public.decorator.ts`

Marca um endpoint como público, dispensando autenticação JWT.

```typescript
@Public()
@Get('health')
healthCheck() {
  return { status: 'ok' };
}
```

**Implementação**: Define o metadata `isPublic = true`, que é verificado pelo `JwtAuthGuard`.

### 5.4.2 @Roles()

**Arquivo**: `src/auth/decorators/roles.decorator.ts`

Define quais roles podem acessar o endpoint.

```typescript
@Roles('admin', 'coordenador')
@Post('plants')
createPlant(@Body() dto: CreatePlantDto) {
  // Apenas admin e coordenador podem criar plantas
}
```

**Implementação**: Define o metadata `roles = [...]`, que é verificado pelo `RolesGuard`.

### 5.4.3 @CurrentUser()

**Arquivo**: `src/auth/decorators/current-user.decorator.ts`

Extrai o usuário autenticado da requisicao. Pode retornar o objeto completo ou um campo específico.

```typescript
// Objeto completo
@Get('me')
getProfile(@CurrentUser() user: AuthenticatedUser) {
  return user;
}

// Campo especifico
@Get('my-id')
getMyId(@CurrentUser('userId') userId: string) {
  return { userId };
}
```

---

## 5.5 RBAC - Controle de Acesso por Roles

### 5.5.1 Roles Disponíveis

| Role | Descrição | Permissões Tipicas |
|------|-----------|-------------------|
| `admin` | Administrador do sistema | Acesso total: CRUD completo, gerênciamento de usuários e plantas |
| `coordenador` | Coordenador de supervisão | Gerência escalas, cria/edita usuários (sup+oper), visualiza relatórios (read-only), cancela relatórios, gerência NCs. **NÃO assina relatórios** |
| `supervisor` | Supervisor Halal | Cria, edita e **assina** relatórios (próprios), registra NCs, visualiza escala própria |
| `operador` | Operador de apoio | Cria e edita relatórios (próprios), registra NCs. **NÃO pode assinar** relatórios |

> **Nota**: O role `gestor` existe no enum Prisma por compatibilidade mas **NÃO e usado na v1.0**.

### 5.5.2 Matriz de Acesso por Módulo

| Módulo | admin | coordenador | supervisor | operador |
|--------|-------|-------------|------------|----------|
| SupervisorProfile | CRUD completo | CRUD (sup+oper) | Leitura/edição (próprio) | Leitura (próprio) |
| Plant | CRUD completo | Leitura | Leitura | Leitura |
| SlaughterReport | CRUD + cancelar | Leitura + cancelar | Criar/editar/assinar (próprio) | Criar/editar (próprio) |
| ProductionReport | CRUD + cancelar | Leitura + cancelar | Criar/editar/assinar (próprio) | Criar/editar (próprio) |
| ShippingReport | CRUD + cancelar | Leitura + cancelar | Criar/editar/assinar (próprio) | Criar/editar (próprio) |
| NonConformity | CRUD + workflow | CRUD + verificar/encerrar | Criar/editar (próprio) | Criar/editar (próprio) |
| Schedule | CRUD completo | CRUD completo | Leitura (própria) | Leitura (própria) |
| Dashboard | Completo | Completo | Visão básica | Visão básica |
| PDF Export | Todos | Todos | Próprios (assinados) | Próprios (assinados) |

---

## 5.6 CORS

Configurado em `main.ts` com origins permitidas:

```typescript
const allowedOrigins = [
  'http://localhost:5174',   // Frontend SIH (dev)
  'http://localhost:3000',   // Alternativo
];

// Adiciona origins da variavel de ambiente (producao)
if (process.env.FRONTEND_URL) {
  const envOrigins = process.env.FRONTEND_URL.split(',').map(url => url.trim());
  allowedOrigins.push(...envOrigins);
}
```

**Configurações CORS**:

| Opcao | Valor |
|-------|-------|
| `credentials` | `true` (permite cookies e headers de auth) |
| `methods` | GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD |
| `allowedHeaders` | Content-Type, Authorization, Accept, Origin, X-Requested-With |
| `preflightContinue` | `false` |
| `optionsSuccessStatus` | `204` |

Requisicoes de origins não permitidas recebem um warning no log e são bloqueadas.

---

## 5.7 Validação de Entrada

O `ValidationPipe` global garante que todos os dados de entrada sejam validados:

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,              // Remove campos nao declarados
    forbidNonWhitelisted: true,   // Rejeita campos nao declarados com erro 400
    transform: true,              // Converte tipos automaticamente
    transformOptions: {
      enableImplicitConversion: true,
    },
  }),
);
```

**Protecoes**:
- **Mass assignment**: Campos não declarados no DTO são removidos (`whitelist`) ou rejeitados (`forbidNonWhitelisted`)
- **Tipagem**: Conversão automática previne erros de tipo
- **Validação de formato**: `class-validator` válida emails, UUIDs, enums, ranges, etc.

---

## 5.8 Tratamento de Erros

### 5.8.1 AllExceptionsFilter

**Arquivo**: `src/common/filters/http-exception.filter.ts`

Filtro global que padroniza todas as respostas de erro:

- **Erros 5xx**: Logados com `logger.error()` incluindo stack trace (apenas no servidor)
- **Erros 4xx**: Logados com `logger.warn()` sem stack trace
- **Resposta ao cliente**: Nunca inclui stack traces, independente do ambiente

```json
{
  "statusCode": 401,
  "timestamp": "2026-01-15T10:30:00.000Z",
  "path": "/slaughter-reports",
  "message": "Unauthorized"
}
```

### 5.8.2 LoggingInterceptor

**Arquivo**: `src/common/interceptors/logging.interceptor.ts`

Registra todas as requisicoes HTTP com método, URL, status e duração:

```
[HTTP] GET /slaughter-reports 200 - 45ms
[HTTP] POST /non-conformities 201 - 120ms
```

---

## 5.9 Variaveis de Ambiente de Segurança

| Variavel | Descrição | Exemplo |
|----------|-----------|---------|
| `JWT_SECRET` | Chave secreta HS256 para assinar/validar tokens | `dev-secret-key-minimum-32-characters...` |
| `JWT_EXPIRES_IN` | Tempo de expiração do token | `7d` |
| `FRONTEND_URL` | Origins permitidas no CORS | `http://localhost:5174` |
| `CORS_ORIGIN` | Origin adicional do CORS | `http://localhost:5174` |

---

## 5.10 Estratégia Futura: Cache Offline de Tokens

Para suportar o modo offline (Fase Futura A), o SIH implementara cache de tokens no frontend:

1. **Armazenamento local**: O token JWT será armazenado em `localStorage` ou `IndexedDB`
2. **Validação offline**: O frontend verificara a expiração do token localmente
3. **Cache de perfil**: Dados do `AuthenticatedUser` serão cacheados para uso sem conexão
4. **Sincronização**: Ao reconectar, o frontend tentara renovar o token via backend SIH
5. **Fallback**: Se o token estiver expirado e não houver conexão, relatórios serão enfileirados para envio posterior

Esta estratégia será implementada junto com o suporte a IndexedDB (Dexie.js) e Background Sync.
