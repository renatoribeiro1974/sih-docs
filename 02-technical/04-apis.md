---
title: APIs REST
parent: Documentação Técnica
nav_order: 4
---

# 4. APIs REST

---

## 4.1 Visão Geral

O SIH expoe uma API REST construida com **NestJS 11** e documentada via **Swagger/OpenAPI**.

| Item | Valor |
|------|-------|
| Base URL (local) | `http://localhost:3334` |
| Base URL (produção) | `https://supervisao-industrial-api.ecohalal.solutions` |
| Base URL (staging) | `https://staging-supervisao-industrial-api.ecohalal.solutions` |
| Base URL (development) | `https://dev-supervisao-industrial-api.ecohalal.solutions` |
| Documentação Swagger | `{base-url}/api/docs` |
| Formato | JSON |
| Autenticação | Bearer JWT (header `Authorization`) |
| Versionamento | Sem versionamento na URL (v1 implicito) |

---

## 4.2 Endpoints de Infraestrutura

Os endpoints abaixo fazem parte da infraestrutura base do backend.

### GET /

**Descrição**: Informações básicas da API.
**Autenticação**: Pública (`@Public()`)

```json
{
  "name": "SIH API",
  "version": "1.0.0",
  "description": "Supervisao Industrial Halal"
}
```

### GET /health

**Descrição**: Health check do serviço.
**Autenticação**: Pública (`@Public()`)

```json
{
  "status": "ok",
  "timestamp": "2026-01-15T10:30:00.000Z"
}
```

---

### POST /auth/login

**Descrição**: Autenticação de usuário (self-contained).
**Autenticação**: Pública (`@Public()`)

**Request**:
```json
{
  "email": "ahmad@supervisao.com",
  "password": "SihDev@2026"
}
```

**Response**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "uuid",
    "name": "Ahmad Hassan",
    "email": "ahmad@supervisao.com",
    "role": "supervisor"
  }
}
```

---

## 4.3 Endpoints de Domínio

Todos os endpoints abaixo estão implementados e requerem autenticação JWT.

---

### 4.3.1 /supervisor-profiles

Gerênciamento de perfis de supervisores. O perfil é criado automáticamente no primeiro acesso JWT.

| Método | Rota | Descrição | Roles |
|--------|------|-----------|-------|
| GET | `/supervisor-profiles` | Lista todos os perfis | admin, coordenador |
| GET | `/supervisor-profiles/:id` | Busca perfil por ID | admin, coordenador, supervisor (próprio) |
| POST | `/supervisor-profiles` | Cria perfil manualmente | admin |
| PATCH | `/supervisor-profiles/:id` | Atualiza perfil | admin, coordenador, supervisor (próprio) |
| DELETE | `/supervisor-profiles/:id` | Desativa perfil (soft delete) | admin |
| GET | `/supervisor-profiles/me` | Perfil do usuário autenticado | todos |

---

### 4.3.2 /plants

Gerênciamento de plantas industriais (abatedouros, frigoríficos, etc.).

| Método | Rota | Descrição | Roles |
|--------|------|-----------|-------|
| GET | `/plants` | Lista todas as plantas | todos |
| GET | `/plants/:id` | Busca planta por ID | todos |
| POST | `/plants` | Cria nova planta | admin, coordenador |
| PATCH | `/plants/:id` | Atualiza planta | admin, coordenador |
| DELETE | `/plants/:id` | Desativa planta (soft delete) | admin |

---

### 4.3.3 /slaughter-reports

Relatórios de abate Halal (FM 7.1.4.x). Suporta aves e bovinos com seção adicional de stunning para bovinos.

| Método | Rota | Descrição | Roles |
|--------|------|-----------|-------|
| GET | `/slaughter-reports` | Lista relatórios com filtros | todos |
| GET | `/slaughter-reports/:id` | Busca relatório por ID | todos |
| POST | `/slaughter-reports` | Cria novo relatório (gera serial) | supervisor, operador |
| PATCH | `/slaughter-reports/:id` | Atualiza relatório (rascunho) | supervisor (autor), operador (autor) |
| PATCH | `/slaughter-reports/:id/sign` | Assina relatório (gera hash SHA-256) | supervisor (autor) |
| PATCH | `/slaughter-reports/:id/cancel` | Cancela relatório (com motivo) | coordenador, admin |
| GET | `/slaughter-reports/:id/pdf` | Gera PDF fiel ao FM FAMBRAS | todos (relatório assinado) |
| GET | `/slaughter-reports/serial/next` | Retorna próximo número serial | supervisor, operador |

**Filtros de listagem**:
- `?plantId=uuid` - Filtrar por planta
- `?supervisorId=uuid` - Filtrar por supervisor
- `?status=enviado` - Filtrar por status
- `?species=ave` - Filtrar por especie
- `?dateFrom=2026-01-01&dateTo=2026-01-31` - Filtrar por período
- `?page=1&limit=20` - Páginação

**Número serial**: Formato `SIF/ANO/SEQ` (ex: `001/2026/00001`). Gerado automáticamente pelo backend.

---

### 4.3.4 /production-reports

Relatórios de produção de industrializados (FM 7.1.3.x / FM 7.1.8.x).

| Método | Rota | Descrição | Roles |
|--------|------|-----------|-------|
| GET | `/production-reports` | Lista relatórios com filtros | todos |
| GET | `/production-reports/:id` | Busca relatório por ID | todos |
| POST | `/production-reports` | Cria novo relatório (gera serial) | supervisor, operador |
| PATCH | `/production-reports/:id` | Atualiza relatório (rascunho) | supervisor (autor), operador (autor) |
| PATCH | `/production-reports/:id/sign` | Assina relatório (gera hash SHA-256) | supervisor (autor) |
| PATCH | `/production-reports/:id/cancel` | Cancela relatório (com motivo) | coordenador, admin |
| GET | `/production-reports/:id/pdf` | Gera PDF fiel ao FM FAMBRAS | todos (relatório assinado) |
| GET | `/production-reports/serial/next` | Retorna próximo número serial | supervisor, operador |

**Filtros de listagem**:
- `?plantId=uuid` - Filtrar por planta
- `?supervisorId=uuid` - Filtrar por supervisor
- `?status=rascunho` - Filtrar por status
- `?isSpecialProduction=true` - Filtrar produção especial (FM 7.1.8.x)
- `?dateFrom=2026-01-01&dateTo=2026-01-31` - Filtrar por período
- `?page=1&limit=20` - Páginação

**Matérias-primas**: O campo `meatRawMaterials` e um array JSON com: frigorífico, SIF, data de abate, CSN e certificado Halal.

---

### 4.3.5 /shipping-reports

Relatórios de embarque, venda interna e transferência (FM 7.1.7.x / DCPOA).

| Método | Rota | Descrição | Roles |
|--------|------|-----------|-------|
| GET | `/shipping-reports` | Lista relatórios com filtros | todos |
| GET | `/shipping-reports/:id` | Busca relatório por ID | todos |
| POST | `/shipping-reports` | Cria novo relatório (gera serial) | supervisor, operador |
| PATCH | `/shipping-reports/:id` | Atualiza relatório (rascunho) | supervisor (autor), operador (autor) |
| PATCH | `/shipping-reports/:id/sign` | Assina relatório (gera hash SHA-256) | supervisor (autor) |
| PATCH | `/shipping-reports/:id/cancel` | Cancela relatório (com motivo) | coordenador, admin |
| GET | `/shipping-reports/:id/pdf` | Gera PDF fiel ao FM FAMBRAS | todos (relatório assinado) |
| GET | `/shipping-reports/serial/next` | Retorna próximo número serial | supervisor, operador |

**Filtros de listagem**:
- `?plantId=uuid` - Filtrar por planta
- `?supervisorId=uuid` - Filtrar por supervisor
- `?status=aprovado` - Filtrar por status
- `?shippingType=exportacao` - Filtrar por tipo (exportação, venda_interna, transferência)
- `?dateFrom=2026-01-01&dateTo=2026-01-31` - Filtrar por período
- `?page=1&limit=20` - Páginação

**Tipos de embarque**:
- `exportacao`: Campos completos (importador, container, portos, lacre, país destino)
- `venda_interna`: Campos reduzidos (transporte, veiculo, destino nacional)
- `transferencia`: Campos mínimos (unidade destino, DCPOA)

---

### 4.3.6 /non-conformities

Gestão de não-conformidades (FM 7.1.6.1). Pode ser vinculada a qualquer tipo de relatório.

| Método | Rota | Descrição | Roles |
|--------|------|-----------|-------|
| GET | `/non-conformities` | Lista NCs com filtros | todos |
| GET | `/non-conformities/:id` | Busca NC por ID | todos |
| POST | `/non-conformities` | Cria nova NC | supervisor, coordenador |
| PATCH | `/non-conformities/:id` | Atualiza NC | supervisor (autor), coordenador |
| DELETE | `/non-conformities/:id` | Remove NC (apenas se aberta) | admin |
| PATCH | `/non-conformities/:id/status` | Avanca workflow | coordenador, admin |
| GET | `/non-conformities/overdue` | Lista NCs com prazo vencido | coordenador, admin |

**Filtros de listagem**:
- `?plantId=uuid` - Filtrar por planta
- `?supervisorId=uuid` - Filtrar por supervisor
- `?status=aberta` - Filtrar por status
- `?severity=critica` - Filtrar por severidade
- `?category=higiene` - Filtrar por categoria
- `?overdue=true` - Filtrar apenas vencidas
- `?page=1&limit=20` - Páginação

**Workflow de status**: `aberta` -> `em_tratamento` -> `resolvida` -> `verificada` -> `encerrada`

**Prazo**: 7 dias corridos a partir da criação (conforme PR 7.1). Alertas automáticos ao se apróximar do vencimento.

---

### 4.3.7 /schedules

Escala de supervisores nas plantas industriais.

| Método | Rota | Descrição | Roles |
|--------|------|-----------|-------|
| GET | `/schedules` | Lista escalas com filtros | todos |
| GET | `/schedules/:id` | Busca escala por ID | todos |
| POST | `/schedules` | Cria alocação de escala | coordenador, admin |
| PATCH | `/schedules/:id` | Atualiza alocação | coordenador, admin |
| DELETE | `/schedules/:id` | Remove alocação | coordenador, admin |
| GET | `/schedules/calendar` | Visão calendário mensal | todos |
| GET | `/schedules/by-supervisor/:id` | Escala por supervisor | todos |
| GET | `/schedules/by-plant/:id` | Escala por planta | todos |

**Filtros de listagem**:
- `?supervisorId=uuid` - Filtrar por supervisor
- `?plantId=uuid` - Filtrar por planta
- `?month=2026-01` - Filtrar por mês
- `?shift=matutino` - Filtrar por turno
- `?type=regular` - Filtrar por tipo de escala

**Restrição única**: Um supervisor por planta/data/turno (`@@unique([supervisorId, plantId, date, shift])`).

---

### 4.3.8 /dashboard

Endpoints de agregação para o dashboard operacional.

| Método | Rota | Descrição | Roles |
|--------|------|-----------|-------|
| GET | `/dashboard/summary` | Contadores gerais (dia/semana/mês) | todos |
| GET | `/dashboard/reports-by-status` | Relatórios agrupados por status | todos |
| GET | `/dashboard/ncs-by-severity` | NCs ativas por severidade | todos |
| GET | `/dashboard/ncs-by-plant` | NCs ativas por planta | coordenador, admin |
| GET | `/dashboard/pending-reviews` | Relatórios pendentes de revisão | coordenador, admin |
| GET | `/dashboard/supervisor-productivity` | Produtividade por supervisor | coordenador, admin |
| GET | `/dashboard/trends` | Graficos de tendência (período) | todos |

**Filtros comuns**:
- `?period=day|week|month|custom` - Período de agregação
- `?dateFrom=2026-01-01&dateTo=2026-01-31` - Período customizado
- `?plantId=uuid` - Filtrar por planta

---

## 4.4 Autenticação

Todos os endpoints (exceto os marcados com `@Public()`) exigem um token JWT válido no header:

```
Authorization: Bearer <jwt-token>
```

Na v1.0, o token e emitido pelo **próprio backend SIH** via `POST /auth/login` (autenticação self-contained com bcrypt). Suporta RS256 (recomendado, via `JWT_PUBLIC_KEY_SHI_API` / `JWT_PRIVATE_KEY_SHI_API`) com fallback para HS256 (`JWT_SECRET`) em desenvolvimento. NÃO depende do HalalSphere.

Consulte o documento [05-security.md](./05-security.md) para detalhes completos do fluxo de autenticação.

---

## 4.5 Validação

O SIH utiliza `class-validator` com `ValidationPipe` global configurado em `main.ts`:

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,            // Remove campos nao declarados no DTO
    forbidNonWhitelisted: true, // Retorna erro se campo nao declarado for enviado
    transform: true,            // Converte tipos automaticamente
    transformOptions: {
      enableImplicitConversion: true,
    },
  }),
);
```

**Comportamento**:
- Campos não declarados no DTO são rejeitados com erro 400
- Tipos são convertidos automáticamente (ex: `"1"` -> `1` para números)
- Mensagens de erro detalham qual campo falhou e qual validação

---

## 4.6 Formato de Resposta de Erro

O `AllExceptionsFilter` padroniza todas as respostas de erro:

```json
{
  "statusCode": 400,
  "timestamp": "2026-01-15T10:30:00.000Z",
  "path": "/slaughter-reports",
  "message": "Bad Request",
  "error": ["species must be a valid enum value"]
}
```

**Códigos de status HTTP comuns**:

| Código | Significado | Quando |
|--------|------------|--------|
| 200 | OK | Requisicao bem-sucedida |
| 201 | Created | Recurso criado com sucesso |
| 400 | Bad Request | Validação falhou (campos inválidos) |
| 401 | Unauthorized | Token JWT ausente ou inválido |
| 403 | Forbidden | Role do usuário não tem permissão |
| 404 | Not Found | Recurso não encontrado |
| 409 | Conflict | Violação de constraint única |
| 500 | Internal Server Error | Erro interno (stack trace omitido em produção) |

---

## 4.7 Padrão de Páginação (Planejado)

Todos os endpoints de listagem seguirao o padrão:

**Request**:
```
GET /slaughter-reports?page=1&limit=20&sort=createdAt&order=desc
```

**Parâmetros de query**:

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `page` | number | 1 | Número da página (1-based) |
| `limit` | number | 20 | Itens por página (max: 100) |
| `sort` | string | `createdAt` | Campo de ordenação |
| `order` | string | `desc` | Direcao: `asc` ou `desc` |

**Response**:
```json
{
  "data": [...],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
```

---

## 4.8 Swagger

A documentação interativa da API está disponível em `/api/docs` com as seguintes configurações:

- **Autorização persistente**: O token JWT e mantido entre recarregamentos da página
- **Filtro de busca**: Permite buscar endpoints por nome
- **Duração de request**: Exibe o tempo de resposta de cada chamada
- **Servidores**: Local (`http://localhost:3334`), Production (`https://supervisao-industrial-api.ecohalal.solutions`), Staging, Development

Para autenticar no Swagger, clique em "Authorize" e insira o token JWT no formato:
```
<seu-token-jwt>
```

---

## 4.9 Enums da API

Os seguintes enums são utilizados nos endpoints:

| Enum | Valores | Nota |
|------|---------|------|
| UserRole | `admin`, `coordenador`, `supervisor`, `operador` | Legado `gestor` removido |
| Species | `bovino`, `ave` | Legados removidos na migração `remove_legacy_species` |
| CollaboratorType | `degolador`, `sheik`, `auxiliar`, `veterinario`, `outro` | Tipo do colaborador operacional |
| Shift | `matutino`, `vespertino`, `noturno`, `integral` | |
| PlantType | `abatedouro`, `frigorifico`, `laticinio`, `processamento`, `armazenamento`, `outro` | |
| ReportStatus | `rascunho`, `assinado`, `cancelado` | Workflow: rascunho → assinado (final). Sem etapa de aprovação |
| ShippingType | `exportacao`, `venda_interna`, `transferencia` | |
| TransportType | `terrestre`, `aereo`, `maritimo` | |
| Severity | `critica`, `maior`, `menor`, `observacao` | |
| NCStatus | `aberta`, `em_tratamento`, `resolvida`, `verificada`, `encerrada` | |
| ScheduleType | `regular`, `substituicao`, `extra`, `folga` | |

---

## 4.10 Endpoints de Exportação PDF

Cada tipo de relatório possui um endpoint de geração de PDF fiel ao modelo FAMBRAS:

| Endpoint | FM | Descrição |
|----------|----|-----------|
| `GET /slaughter-reports/:id/pdf` | FM 7.1.4.1 (aves) / FM 7.1.4.2 (bovino) | PDF de relatório de abate |
| `GET /production-reports/:id/pdf` | FM 7.1.3.1 | PDF de relatório de produção |
| `GET /shipping-reports/:id/pdf` | FM 7.1.7.1 / 7.1.7.3 / 7.1.7.4 | PDF de relatório de embarque |

**Caracteristicas do PDF gerado**:
- Cabeçalho com logo FAMBRAS Halal + número FM + revisão + data + páginação
- Textos bilingues (portugues/ingles) conforme formulários originais
- Tabelas formatadas conforme layout FM FAMBRAS
- Area de assinatura com declaração bilíngue do supervisor
- Hash SHA-256 da assinatura digital exibido no rodape
- Gerado via **PDFKit** no backend (servidor)
- Retorna `Content-Type: application/pdf`

---

## 4.11 Endpoints — Collaborator (Implementado)

Cadastro de colaboradores operacionais (degoladores, sheiks, auxiliares, veterinários) que atuam nas plantas mas não são usuários do sistema. Todos os endpoints requerem `@Roles('admin', 'coordenador')`.

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/collaborators` | Lista colaboradores com filtros (paginado) |
| GET | `/collaborators/:id` | Busca colaborador por ID (inclui plantas vinculadas) |
| GET | `/collaborators/by-plant/:plantId` | Lista colaboradores vinculados a uma planta |
| POST | `/collaborators` | Cria novo colaborador (aceita `plantIds[]` opcional) |
| PATCH | `/collaborators/:id` | Atualiza colaborador |
| PATCH | `/collaborators/:id/deactivate` | Desativa colaborador (soft delete) |
| POST | `/collaborators/:id/photo` | Upload de foto (multipart/form-data, JPEG/PNG max 2MB) |
| DELETE | `/collaborators/:id/photo` | Remove foto |
| POST | `/collaborators/:id/plants` | Vincula colaborador a planta(s) |
| DELETE | `/collaborators/:id/plants/:plantId` | Desvincula de planta |

**Filtros de listagem** (`GET /collaborators`):
- `?page=1&limit=20` — Paginação
- `?plantId=uuid` — Filtrar por planta vinculada
- `?type=degolador` — Filtrar por tipo (degolador, sheik, auxiliar, veterinario, outro)
- `?isActive=true` — Filtrar ativos/inativos
- `?search=nome` — Busca por nome
- `?sort=name&order=asc` — Ordenação

**Integração com relatórios** (`ReportStaff`):
- Nos DTOs de criação dos 3 relatórios: campo opcional `staffIds: string[]` (array de UUIDs de colaboradores)
- No `findOne` de cada relatório: campo `staff[]` com dados do colaborador (id, name, type, document)
- Frontend: componente `ReportStaffSelector` exibe colaboradores filtrados pela planta do relatório

---

## 4.12 Endpoints — Dashboard Analytics (Implementado)

Endpoints de Business Intelligence com agregação de dados operacionais.

| Método | Rota | Descrição | Filtros |
|--------|------|-----------|---------|
| GET | `/dashboard/summary` | Resumo geral (contadores) | `plantId?`, `supervisorId?` |
| GET | `/dashboard/reports-by-status` | Relatórios agrupados por status | `plantId?` |
| GET | `/dashboard/non-conformities-by-severity` | NCs por severidade | `plantId?` |
| GET | `/dashboard/reports-trend` | Tendência de relatórios no tempo | `dateFrom`, `dateTo`, `plantId?`, `groupBy` |
| GET | `/dashboard/slaughter-analytics` | Analytics de abate | `dateFrom`, `dateTo`, `plantId?`, `species?`, `shift?` |
| GET | `/dashboard/production-analytics` | Analytics de produção | `dateFrom`, `dateTo`, `plantId?` |
| GET | `/dashboard/shipping-analytics` | Analytics de embarque | `dateFrom`, `dateTo`, `plantId?` |
| GET | `/dashboard/nc-analytics` | Analytics de NCs | `dateFrom`, `dateTo`, `plantId?`, `severity?` |
| GET | `/dashboard/supervisor-analytics` | Analytics de supervisores | `dateFrom`, `dateTo`, `supervisorId?` |
