---
title: Roadmap Detalhado
nav_order: 5
---

# Roadmap Detalhado de Implementação

**Projeto**: SIH - Supervisão Industrial Halal
**Versão**: v1.0
**Última Atualização**: 2026-02-24

---

## Status Atual

| Fase | Descrição | Status | Progresso |
|------|-----------|--------|-----------|
| Fase 1 | Scaffolding (Repos + Configs) | COMPLETA | 4/4 tarefas |
| Fase 2 | Backend Core (Infra) | COMPLETA | 4/4 tarefas |
| Fase 3 | Backend Domínio (Negócio) | COMPLETA | 8/8 módulos |
| Fase 4 | Frontend Core (Base) | PROXIMO | 0/5 tarefas |
| Fase 5 | Frontend Pages (Páginas) | PENDENTE | 0/7 grupos |
| Fase 6 | Finalização (Deploy) | PENDENTE | 0/4 tarefas |

**Resumo de User Stories v1.0**: 33 stories | 202 story points | 6 épicos

---

## Fase 1: Scaffolding (COMPLETA)

**Objetivo**: Criar os 3 repositorios com toda a configuração de projeto.

- [x] **1.1** Criar repositorio `sih-docs/`
  - Estrutura de documentação criada
  - PRD completo com 39 user stories em 7 épicos
  - Arquitetura de features documentada
  - Roadmap original definido
- [x] **1.2** Criar repositorio `sih-backend/`
  - NestJS scaffolding funcional
  - package.json, tsconfig, eslint, prettier configurados
  - Docker e docker-compose preparados
  - Prisma schema completo com todos os modelos v1.0
- [x] **1.3** Criar repositorio `sih-frontend/`
  - React + Vite scaffolding funcional
  - Tailwind CSS configurado
  - Estrutura de pastas: components/, hooks/, pages/, services/, types/
  - PWA manifest preparado
- [x] **1.4** Criar workspace VS Code
  - `SupervisaoIndustrialHalal.code-workspace` configurado
  - 3 repos integrados em workspace único

**Verificação**: Repos criados, configs válidas, workspace abre corretamente.

---

## Fase 2: Backend Core (COMPLETA)

**Objetivo**: Infraestrutura funcional do backend (NestJS).

- [x] **2.1** Config Module
  - `config.module.ts` - Módulo de configuração global
  - `aws-config.service.ts` - Serviço de configuração AWS (Cognito, S3, etc.)
  - Variaves de ambiente validadas
- [x] **2.2** Prisma Module
  - `prisma.module.ts` - Módulo global do Prisma
  - `prisma.service.ts` - Serviço com lifecycle hooks
  - Schema completo com 6 modelos + 11 enums + índices + constraints
  - **Nota**: Resolvido problema de migração Prisma 7 usando provider `prisma-client-js` com adapter `PrismaPg`
- [x] **2.3** Auth Module
  - `jwt.strategy.ts` - Validação JWT RS256/HS256 (via Gestão de Certificações/HalalSphere)
  - `jwt-auth.guard.ts` - Guard de autenticação global
  - `roles.guard.ts` - Guard de autorização por role (admin, coordenador, supervisor, gestor)
  - `current-user.decorator.ts` - Decorator para extrair usuário do JWT
  - `roles.decorator.ts` - Decorator para definir roles permitidas
  - `public.decorator.ts` - Decorator para rotas públicas
  - `jwt-payload.type.ts` - Tipagem do payload JWT
- [x] **2.4** Health + Common
  - `health.controller.ts` - Endpoint `GET /health` funcional
  - `http-exception.filter.ts` - Filtro global de exceções
  - `logging.interceptor.ts` - Interceptor de logging de requests
  - `serial-number.util.ts` - Gerador de número serial (SIF/ANO/SEQ)
- [x] **2.5** Banco de Dados
  - PostgreSQL 16 com extensões: uuid-ossp, pgcrypto, pg_trgm
  - Migrações aplicadas com sucesso
  - Tabelas criadas conforme schema Prisma

**Verificação**: `npm run start:dev` funciona, `GET /health` retorna OK, Swagger acessível em `/api/docs`.

---

## Fase 3: Backend Domínio (COMPLETA)

**Objetivo**: Todos os 8 módulos de negócio com CRUD completo e lógica de domínio.

### 3.1 SupervisorProfile Module

- [ ] **DTOs**
  - `create-supervisor-profile.dto.ts` - name, email, phone?, registration?, qualifications?, preferences?
  - `update-supervisor-profile.dto.ts` - PartialType do create
  - `supervisor-profile-query.dto.ts` - Filtros: role, isActive, search (nome/email)
  - `supervisor-profile-response.dto.ts` - Resposta com campos formatados
- [ ] **Service** (`supervisor-profile.service.ts`)
  - `findOrCreateFromJwt(payload)` - Auto-cria perfil no primeiro acesso via JWT
  - `findAll(query)` - Listagem com filtros e páginação
  - `findOne(id)` - Busca por ID
  - `update(id, dto)` - Atualização parcial
  - `deactivate(id)` - Soft delete (isActive = false)
  - `updateLastLogin(id)` - Atualiza timestamp de último login
- [ ] **Controller** (`supervisor-profile.controller.ts`)
  - `GET /supervisor-profiles` - Lista com filtros (coordenador+)
  - `GET /supervisor-profiles/me` - Perfil do usuário autenticado
  - `GET /supervisor-profiles/:id` - Busca por ID
  - `PATCH /supervisor-profiles/:id` - Atualiza perfil
  - `PATCH /supervisor-profiles/:id/deactivate` - Desativa (admin)
- [ ] **Testes** - Unitários do service + e2e do controller

**Épico relacionado**: Prerequisito para todos os épicos (autoria de relatórios)

---

### 3.2 Plant Module

- [ ] **DTOs**
  - `create-plant.dto.ts` - name, sifCode, type, address?, contact?, species[]
  - `update-plant.dto.ts` - PartialType do create
  - `plant-query.dto.ts` - Filtros: type, species, isActive, search (nome/SIF)
  - `plant-response.dto.ts` - Resposta com contadores de relatórios
- [ ] **Service** (`plant.service.ts`)
  - `create(dto)` - Cria planta com validação de SIF único
  - `findAll(query)` - Listagem com filtros e páginação
  - `findOne(id)` - Busca por ID com contadores
  - `update(id, dto)` - Atualização parcial
  - `deactivate(id)` - Soft delete
- [ ] **Controller** (`plant.controller.ts`)
  - `POST /plants` - Cria planta (coordenador+)
  - `GET /plants` - Lista com filtros (todos)
  - `GET /plants/:id` - Detalhes (todos)
  - `PATCH /plants/:id` - Atualiza (coordenador+)
  - `PATCH /plants/:id/deactivate` - Desativa (admin)
- [ ] **Testes** - Unitários + e2e

**Épico relacionado**: Prerequisito para todos os épicos (local dos relatórios)

---

### 3.3 SlaughterReport Module (FM 7.1.4.x)

- [ ] **DTOs**
  - `create-slaughter-report.dto.ts` - Todos os campos do modelo + validações
  - `update-slaughter-report.dto.ts` - PartialType (somente status=rascunho)
  - `slaughter-report-query.dto.ts` - Filtros: plantId, daté range, species, status, shift
  - `submit-slaughter-report.dto.ts` - Transicao rascunho -> enviado
  - `review-slaughter-report.dto.ts` - Transicao enviado -> revisado/aprovado/rejeitado
  - `slaughter-verification-item.dto.ts` - 14 itens C/NC (fixos)
  - `stunning-verification.dto.ts` - Avaliação de insensibilização (bovinos)
- [ ] **Service** (`slaughter-report.service.ts`)
  - `create(dto, supervisorId)` - Cria com serial automático (SIF/ANO/SEQ)
  - `findAll(query)` - Listagem com filtros, páginação e includes
  - `findOne(id)` - Detalhes completos com plant e supervisor
  - `update(id, dto)` - Somente rascunhos
  - `submit(id)` - Válida e envia para revisão
  - `review(id, dto, reviewerId)` - Coordenador revisa/aprova/rejeita
  - `cancel(id, reason)` - Cancela com motivo
  - `generateSerialNumber(plantSif, year)` - SIF/ANO/SEQ sequencial
  - `validateVerificationItems(items)` - Válida 14 itens obrigatórios
  - `validateStunning(species, data)` - Válida dados de stunning para bovinos
- [ ] **Controller** (`slaughter-report.controller.ts`)
  - `POST /slaughter-reports` - Cria relatório (supervisor+)
  - `GET /slaughter-reports` - Lista com filtros (todos)
  - `GET /slaughter-reports/:id` - Detalhes (todos)
  - `PATCH /slaughter-reports/:id` - Atualiza rascunho (autor)
  - `POST /slaughter-reports/:id/submit` - Envia para revisão (autor)
  - `POST /slaughter-reports/:id/review` - Revisa (coordenador+)
  - `POST /slaughter-reports/:id/cancel` - Cancela (coordenador+)
- [ ] **Testes** - Unitários + e2e + validação de itens C/NC

**Épico**: Epic 01 - Relatórios de Abate | **Stories**: 7 | **Points**: 47 | **Prioridade**: P0

---

### 3.4 ProductionReport Module (FM 7.1.3.x / FM 7.1.8.x)

- [ ] **DTOs**
  - `create-production-report.dto.ts` - Todos os campos + matérias-primas + ingredientes
  - `update-production-report.dto.ts` - PartialType (somente rascunho)
  - `production-report-query.dto.ts` - Filtros: plantId, daté range, status, isSpecialProduction
  - `submit-production-report.dto.ts` - Transicao rascunho -> enviado
  - `review-production-report.dto.ts` - Transicao enviado -> revisado/aprovado/rejeitado
  - `production-verification-item.dto.ts` - 5 itens C/NC (fixos)
  - `meat-raw-material.dto.ts` - frigorífico, SIF, data abate, CSN, certificado Halal
  - `approved-ingredient.dto.ts` - fornecedor, lote, válidade
- [ ] **Service** (`production-report.service.ts`)
  - `create(dto, supervisorId)` - Cria com serial automático
  - `findAll(query)` - Listagem com filtros e páginação
  - `findOne(id)` - Detalhes completos
  - `update(id, dto)` - Somente rascunhos
  - `submit(id)` - Válida e envia
  - `review(id, dto, reviewerId)` - Coordenador revisa
  - `cancel(id, reason)` - Cancela com motivo
  - `validateRawMaterials(materials)` - Válida matérias-primas obrigatórias
  - `validateIngredients(ingredients)` - Válida ingredientes aprovados
- [ ] **Controller** (`production-report.controller.ts`)
  - `POST /production-reports` - Cria relatório (supervisor+)
  - `GET /production-reports` - Lista com filtros (todos)
  - `GET /production-reports/:id` - Detalhes (todos)
  - `PATCH /production-reports/:id` - Atualiza rascunho (autor)
  - `POST /production-reports/:id/submit` - Envia para revisão (autor)
  - `POST /production-reports/:id/review` - Revisa (coordenador+)
  - `POST /production-reports/:id/cancel` - Cancela (coordenador+)
- [ ] **Testes** - Unitários + e2e + validação de matérias-primas

**Épico**: Epic 02 - Relatórios de Produção | **Stories**: 6 | **Points**: 40 | **Prioridade**: P0

---

### 3.5 ShippingReport Module (FM 7.1.7.x / DCPOA)

- [ ] **DTOs**
  - `create-shipping-report.dto.ts` - Campos base + campos condicionais por tipo
  - `update-shipping-report.dto.ts` - PartialType (somente rascunho)
  - `shipping-report-query.dto.ts` - Filtros: plantId, daté range, shippingType, status
  - `submit-shipping-report.dto.ts` - Transicao rascunho -> enviado
  - `review-shipping-report.dto.ts` - Transicao enviado -> revisado/aprovado/rejeitado
  - `shipping-verification-item.dto.ts` - 2 itens C/NC (exclusividade Halal + selo)
  - `shipping-product.dto.ts` - produto, código, lote, datas, pesos, temperatura
- [ ] **Service** (`shipping-report.service.ts`)
  - `create(dto, supervisorId)` - Cria com serial automático
  - `findAll(query)` - Listagem com filtros e páginação
  - `findOne(id)` - Detalhes completos
  - `update(id, dto)` - Somente rascunhos
  - `submit(id)` - Válida campos condicionais por tipo e envia
  - `review(id, dto, reviewerId)` - Coordenador revisa
  - `cancel(id, reason)` - Cancela com motivo
  - `validateByType(type, data)` - Válida campos obrigatórios por shippingType:
    - exportação: importer, container, ports, sealNumber obrigatórios
    - venda_interna: campos simplificados
    - transferência: DCPOA campos mínimos
  - `validateProducts(products)` - Válida lista de produtos
- [ ] **Controller** (`shipping-report.controller.ts`)
  - `POST /shipping-reports` - Cria relatório (supervisor+)
  - `GET /shipping-reports` - Lista com filtros (todos)
  - `GET /shipping-reports/:id` - Detalhes (todos)
  - `PATCH /shipping-reports/:id` - Atualiza rascunho (autor)
  - `POST /shipping-reports/:id/submit` - Envia para revisão (autor)
  - `POST /shipping-reports/:id/review` - Revisa (coordenador+)
  - `POST /shipping-reports/:id/cancel` - Cancela (coordenador+)
- [ ] **Testes** - Unitários + e2e + validação por tipo

**Épico**: Epic 03 - Relatórios de Embarque | **Stories**: 6 | **Points**: 38 | **Prioridade**: P0

---

### 3.6 NonConformity Module (FM 7.1.6.1)

- [ ] **DTOs**
  - `create-non-conformity.dto.ts` - description, severity, category, evidence?, vínculo a relatório?
  - `update-non-conformity.dto.ts` - PartialType (somente aberta/em_tratamento)
  - `non-conformity-query.dto.ts` - Filtros: plantId, severity, status, category, daté range, overdue
  - `treat-non-conformity.dto.ts` - correctiveAction, preventiveAction (aberta -> em_tratamento)
  - `resolve-non-conformity.dto.ts` - Resolução com evidencia (em_tratamento -> resolvida)
  - `verify-non-conformity.dto.ts` - Verificação final (resolvida -> verificada -> encerrada)
- [ ] **Service** (`non-conformity.service.ts`)
  - `create(dto, supervisorId)` - Cria NC com prazo automático de 7 dias
  - `findAll(query)` - Listagem com filtros, páginação e flag de vencidas
  - `findOne(id)` - Detalhes com relatório de origem e timeline
  - `update(id, dto)` - Somente aberta/em_tratamento
  - `treat(id, dto)` - Adiciona ação corretiva/preventiva
  - `resolve(id, dto, resolverId)` - Marca como resolvida
  - `verify(id, dto, verifierId)` - Coordenador verifica e encerra
  - `reopen(id, reason)` - Reabre NC (verificação falhou)
  - `findOverdue()` - Lista NCs que ultrapassaram prazo de 7 dias
  - `calculateDeadline(createdAt)` - Calcula data limite (+7 dias corridos)
  - **Workflow**: aberta -> em_tratamento -> resolvida -> verificada -> encerrada
- [ ] **Controller** (`non-conformity.controller.ts`)
  - `POST /non-conformities` - Cria NC (supervisor+)
  - `GET /non-conformities` - Lista com filtros (todos)
  - `GET /non-conformities/overdue` - Lista vencidas (coordenador+)
  - `GET /non-conformities/:id` - Detalhes (todos)
  - `PATCH /non-conformities/:id` - Atualiza (autor/coordenador)
  - `POST /non-conformities/:id/treat` - Inicia tratamento (supervisor+)
  - `POST /non-conformities/:id/resolve` - Resolve (supervisor+)
  - `POST /non-conformities/:id/verify` - Verifica e encerra (coordenador+)
  - `POST /non-conformities/:id/reopen` - Reabre (coordenador+)
- [ ] **Testes** - Unitários + e2e + workflow completo + prazo 7 dias

**Épico**: Epic 04 - Não-Conformidades | **Stories**: 6 | **Points**: 35 | **Prioridade**: P0

---

### 3.7 Schedule Module

- [ ] **DTOs**
  - `create-schedule.dto.ts` - supervisorId, plantId, date, shift, type, notes?
  - `update-schedule.dto.ts` - PartialType
  - `schedule-query.dto.ts` - Filtros: supervisorId, plantId, daté range, shift, type
  - `bulk-create-schedule.dto.ts` - Criação em lote (escala mensal)
- [ ] **Service** (`schedule.service.ts`)
  - `create(dto)` - Cria alocação com validação de unique constraint
  - `bulkCreate(dtos)` - Criação em lote para escala mensal
  - `findAll(query)` - Listagem com filtros e páginação
  - `findByMonth(year, month, plantId?)` - Calendário mensal
  - `findBySupervisor(supervisorId, dateRange)` - Escala do supervisor
  - `findByPlant(plantId, dateRange)` - Escala da planta
  - `update(id, dto)` - Atualiza alocação
  - `delete(id)` - Remove alocação
  - `checkConflict(supervisorId, date, shift)` - Verifica conflito de horário
- [ ] **Controller** (`schedule.controller.ts`)
  - `POST /schedules` - Cria alocação (coordenador+)
  - `POST /schedules/bulk` - Cria em lote (coordenador+)
  - `GET /schedules` - Lista com filtros (todos)
  - `GET /schedules/month/:year/:month` - Calendário mensal (todos)
  - `GET /schedules/supervisor/:id` - Escala do supervisor (supervisor próprio ou coordenador+)
  - `GET /schedules/plant/:id` - Escala da planta (coordenador+)
  - `PATCH /schedules/:id` - Atualiza (coordenador+)
  - `DELETE /schedules/:id` - Remove (coordenador+)
- [ ] **Testes** - Unitários + e2e + unique constraint + bulk

**Épico**: Epic 05 - Escala de Supervisores | **Stories**: 4 | **Points**: 20 | **Prioridade**: P1

---

### 3.8 Dashboard Module

- [ ] **DTOs**
  - `dashboard-query.dto.ts` - Filtros: plantId?, dateRange, supervisorId?
  - `dashboard-summary.dto.ts` - Resposta com todos os indicadores
- [ ] **Service** (`dashboard.service.ts`)
  - `getSummary(query, userRole)` - Indicadores consolidados por persona:
    - **Supervisor**: seus relatórios, suas NCs, sua escala
    - **Coordenador**: todos os relatórios, todas as NCs, todas as escalas
    - **Gestor**: visão executiva com tendências
  - `getReportCounts(query)` - Contadores por tipo e status (dia/semana/mês)
  - `getNCCounts(query)` - NCs ativas por severidade e planta
  - `getPendingReviews(query)` - Relatórios pendentes de revisão
  - `getProductivity(query)` - Relatórios por supervisor por período
  - `getTrends(query)` - Dados para graficos de tendência (30/60/90 dias)
  - `getOverdueNCs(query)` - NCs que ultrapassaram prazo de 7 dias
- [ ] **Controller** (`dashboard.controller.ts`)
  - `GET /dashboard/summary` - Resumo consolidado (todos, filtrado por role)
  - `GET /dashboard/reports` - Contadores de relatórios
  - `GET /dashboard/non-conformities` - Indicadores de NCs
  - `GET /dashboard/pending-reviews` - Pendentes de revisão (coordenador+)
  - `GET /dashboard/productivity` - Produtividade (coordenador+)
  - `GET /dashboard/trends` - Tendências (gestor+)
- [ ] **Testes** - Unitários + e2e + visoes por persona

**Épico**: Epic 06 - Dashboard e Relatórios | **Stories**: 4 | **Points**: 22 | **Prioridade**: P1

---

### Ordem de Implementação da Fase 3

A ordem recomendada leva em conta dependências entre módulos:

```
Semana 1:  SupervisorProfile + Plant (prerequisitos, sem dependencia externa)
Semana 2:  SlaughterReport (P0, modulo mais complexo, 47 SP)
Semana 3:  ProductionReport (P0, similar ao SlaughterReport)
Semana 4:  ShippingReport (P0, 3 tipos condicionais)
Semana 5:  NonConformity (P0, vinculado aos 3 relatorios anteriores)
Semana 6:  Schedule + Dashboard (P1, dependem de dados existentes)
```

**Verificação da Fase 3**: Swagger mostra todos os endpoints, CRUD funciona via Swagger/Postman, workflows testados.

---

## Fase 4: Frontend Core (PENDENTE)

**Objetivo**: Base funcional do frontend com auth e layout.

- [ ] **4.1** Axios Instances
  - `services/api.ts` - Instancia Axios para SIH backend (baseURL, interceptors, token)
  - `services/auth-api.ts` - Instancia Axios para Gestão de Certificações (autenticação)
  - `services/utils.ts` - Helpers: error handling, retry logic, response parsing
  - Interceptors: token automático no header, refresh token, redirect 401
- [ ] **4.2** shadcn/ui Components (16 componentes base)
  - Button, Card, Input, Label, Select, Textarea
  - Table, Dialog, Sheet, Tabs
  - Badge, Alert, Toast (Sonner)
  - Calendar, Popover, DropdownMenu
  - Configuração de tema Halal (cores: verde, dourado, branco)
- [ ] **4.3** Layout
  - `components/layout/AppLayout.tsx` - Layout principal com sidebar + header + content
  - `components/layout/Header.tsx` - Barra superior: logo, nome do usuário, notificações, logout
  - `components/layout/Sidebar.tsx` - Menu lateral: navegação por módulo, collapse em mobile
  - Responsividade: sidebar -> bottom nav em mobile/tablet
- [ ] **4.4** Auth Flow
  - `pages/Login.tsx` - Tela de login (redirect para Gestão de Certificações)
  - `hooks/useAuth.ts` - Hook de autenticação: login, logout, isAuthenticated, user
  - `components/guards/PrivateRoute.tsx` - Guard de rota autenticada
  - `components/guards/RoleRoute.tsx` - Guard de rota por role
  - Cache de JWT no localStorage com validação de expiração
  - Refresh token automático
- [ ] **4.5** Shared Components (reutilizados entre épicos)
  - `components/shared/VerificationChecklist.tsx` - Renderiza itens C/NC com checkboxes (parametrizavel)
  - `components/shared/ProductTable.tsx` - Tabela editavel de produtos (add/edit/remove linhas)
  - `components/shared/ReportHeader.tsx` - Cabeçalho padrão: FM, serial, planta, supervisor, data
  - `components/shared/StatusBadge.tsx` - Badge colorido por status de relatório/NC
  - `components/shared/PaginatedTable.tsx` - Tabela com páginação, sort e filtros
  - `components/shared/ConfirmDialog.tsx` - Dialog de confirmação reutilizavel
  - `components/shared/LoadingSpinner.tsx` - Spinner de carregamento
  - `components/shared/EmptyState.tsx` - Estado vazio com ilustração e ação

**Verificação**: Login funciona (com Gestão de Certificações rodando), layout renderiza, navegação funcional.

---

## Fase 5: Frontend Pages (PENDENTE)

**Objetivo**: Todas as páginas funcionais com integração completa ao backend.

- [ ] **5.1** Dashboard (Epic 06)
  - `pages/Dashboard.tsx` - Página principal com 3 visoes por persona
  - Componentes: KPI cards, graficos de tendência, lista de pendencias
  - Filtros: planta, período, tipo de relatório
  - Auto-refresh a cada 30 segundos
- [ ] **5.2** Relatórios de Abate (Epic 01)
  - `pages/slaughter/SlaughterReportList.tsx` - Listagem com filtros avancados
  - `pages/slaughter/SlaughterReportForm.tsx` - Formulário com:
    - 14 itens de verificação C/NC
    - Seção de insensibilização (bovinos): avaliação de pressão + conferências
    - Contagem: total, aprovados, rejeitados + sequencial
    - Camaras de resfriamento, subprodutos
  - `pages/slaughter/SlaughterReportDetails.tsx` - Visualização + ações de workflow
- [ ] **5.3** Relatórios de Produção (Epic 02)
  - `pages/production/ProductionReportList.tsx` - Listagem com filtros
  - `pages/production/ProductionReportForm.tsx` - Formulário com:
    - 5 itens de verificação C/NC
    - Tabela de matérias-primas cárneas (frigorífico, SIF, data abate, CSN, cert Halal)
    - Tabela de ingredientes aprovados (fornecedor, lote, válidade)
    - Dados do produto final (código, lote, datas, pesos, embalagem)
    - Flag produção especial (FM 7.1.8.x)
  - `pages/production/ProductionReportDetails.tsx` - Visualização + ações de workflow
- [ ] **5.4** Relatórios de Embarque (Epic 03)
  - `pages/shipping/ShippingReportList.tsx` - Listagem com filtros + tipo
  - `pages/shipping/ShippingReportForm.tsx` - Formulário com:
    - 2 itens de verificação C/NC
    - Selecao de tipo: exportação, venda interna, transferência
    - Campos condicionais por tipo (exportação: importer, container, portos, lacre)
    - Tabela de produtos (produto, código, lote, datas, pesos, temperatura)
  - `pages/shipping/ShippingReportDetails.tsx` - Visualização + ações de workflow
- [ ] **5.5** Não-Conformidades (Epic 04)
  - `pages/non-conformity/NCList.tsx` - Listagem com filtros + indicador vencidas
  - `pages/non-conformity/NCForm.tsx` - Formulário com:
    - Vínculo opcional a relatório de origem
    - Severidade (crítica, maior, menor, observação)
    - Categoria (higiene, processo, equipamento, materia-prima, rotulagem, etc.)
    - Evidencias (descricoes/fotos)
    - Ações corretivas e preventivas
  - `pages/non-conformity/NCDetails.tsx` - Visualização com:
    - Timeline do workflow completo
    - Prazo de 7 dias com countdown
    - Botões de transicao de status conforme role
- [ ] **5.6** Escala de Supervisores (Epic 05)
  - `pages/schedule/ScheduleCalendar.tsx` - Calendário visual mensal
    - Visão por planta ou por supervisor
    - Drag-and-drop para alocação
    - Cores por tipo (regular, substituicao, extra, folga)
  - `pages/schedule/ScheduleManagement.tsx` - Gestão da escala
    - Criação individual e em lote (mensal)
    - Validação de conflitos em tempo real
- [ ] **5.7** Plantas (Cadastro)
  - `pages/plants/PlantList.tsx` - Listagem com filtros e busca
  - `pages/plants/PlantForm.tsx` - Formulário CRUD completo

**Verificação**: Navegação completa, formulários funcionam, listagens com filtros, workflow de NCs.

---

## Fase 6: Finalização (PENDENTE)

**Objetivo**: Sistema pronto para uso end-to-end.

- [ ] **6.1** Seed Data
  - 5 plantas de teste (abatedouro, frigorífico, processamento, etc.)
  - 10 supervisores com diferentes roles
  - 20+ relatórios de exemplo (todos os tipos e status)
  - 10+ NCs em diferentes estagios do workflow
  - Escala de 1 mês para todas as plantas
- [ ] **6.2** Docker
  - `docker-compose.yml` testado e funcional (backend + frontend + postgres + redis)
  - `Dockerfile` multi-stage para backend (build + prod)
  - `Dockerfile` multi-stage para frontend (build + nginx)
  - `init-extensions.sql` - Cria extensões PostgreSQL automáticamente
  - Health checks em todos os serviços
- [ ] **6.3** Git
  - Repos inicializados com histórico limpo
  - `.gitignore` validado (node_modules, .env, dist, .prisma)
  - Branches: main, develop
  - Tags de versão: v1.0.0
- [ ] **6.4** Build Test
  - `npm install` sem erros em ambos os repos
  - `npm run build` sem erros em ambos os repos
  - `npm run lint` sem warnings em ambos os repos
  - Smoke test: login -> criar relatório -> submeter -> revisar -> dashboard

**Verificação**: Sistema completo funcionando end-to-end com dados de teste.

---

## Fases Futuras

### Fase Futura A: Offline Completo

**Prerequisito**: v1.0 em produção, PWA base já instalada.

- [ ] IndexedDB (Dexie.js) - Armazena relatórios preenchidos offline no tablet
- [ ] Background Sync - Envia relatórios pendentes automáticamente quando reconectar
- [ ] Cache de referência - Plantas, templates de verificação, perfil do supervisor
- [ ] Indicador visual - Badge online/offline + quantidade pendente de sincronização
- [ ] Resolução de conflitos - Timestamp-based (último ganha) ou merge manual para NCs

**Impacto**: Permite uso em plantas sem internet estavel (realidade de muitos frigoríficos).

---

### Fase Futura B: Inventário (Epic 07)

**Prerequisito**: v1.0 em produção, relatórios gerando dados.

- [ ] Conta corrente de carne (FM 7.1.5.1) - Recebido vs. utilizado em produção
- [ ] Inventário de lotes (FM 7.1.5.6) - Gerado vs. transferido entre unidades
- [ ] Inventário de rotulagem (FM 7.1.3.6) - Sem rótulo -> rotulado -> embarcado/descartado
- [ ] Dashboard de inventário - Visão consolidada de estoques
- [ ] Migração de dados - Importação de planilhas Excel historicas

**User Stories**: 6 | **Story Points**: TBD

---

## Dependências entre Fases

```
                    +-------------------+
                    |   Fase 1:         |
                    |   Scaffolding     |
                    |   [COMPLETA]      |
                    +---------+---------+
                              |
                              v
                    +-------------------+
                    |   Fase 2:         |
                    |   Backend Core    |
                    |   [COMPLETA]      |
                    +---------+---------+
                              |
                              v
                    +-------------------+
                    |   Fase 3:         |
                    |   Backend Dominio |
                    |   [PROXIMO]       |
                    +---------+---------+
                              |
                    +---------+---------+
                    |                   |
                    v                   v
          +-------------------+  +-------------------+
          |   Fase 4:         |  |                   |
          |   Frontend Core   |  |   (pode iniciar   |
          |   [PENDENTE]      |  |    em paralelo    |
          +---------+---------+  |    com Fase 3)    |
                    |            +-------------------+
                    v
          +-------------------+
          |   Fase 5:         |
          |   Frontend Pages  |
          |   [PENDENTE]      |
          +---------+---------+
                    |
                    v
          +-------------------+
          |   Fase 6:         |
          |   Finalizacao     |
          |   [PENDENTE]      |
          +---------+---------+
                    |
          +---------+---------+
          |                   |
          v                   v
+-------------------+  +-------------------+
|   Futuro A:       |  |   Futuro B:       |
|   Offline         |  |   Inventario      |
+-------------------+  +-------------------+
```

### Dependências Internas da Fase 3

```
+---------------------+     +---------------------+
| SupervisorProfile   |     |       Plant          |
| (prerequisito)      |     | (prerequisito)       |
+----------+----------+     +----------+----------+
           |                           |
           +-------------+-------------+
                         |
           +-------------+-------------+
           |             |             |
           v             v             v
  +--------+---+  +------+------+  +--+----------+
  | Slaughter  |  | Production  |  |  Shipping   |
  | Report     |  | Report      |  |  Report     |
  +--------+---+  +------+------+  +--+----------+
           |             |             |
           +-------------+-------------+
                         |
                         v
              +----------+----------+
              |   NonConformity     |
              |   (vinculada aos 3  |
              |    relatorios)      |
              +----------+----------+
                         |
           +-------------+-------------+
           |                           |
           v                           v
  +--------+--------+     +-----------+---------+
  |    Schedule      |     |     Dashboard       |
  |  (independente,  |     |  (depende de dados  |
  |   usa Plant +    |     |   dos outros)       |
  |   Supervisor)    |     |                     |
  +-----------------+      +---------------------+
```

### Nota sobre Paralelismo

- **Fase 4 pode iniciar em paralelo com Fase 3**: O Frontend Core (layout, auth, componentes) não depende dos endpoints de domínio. Pode ser desenvolvido simultaneamente.
- **Fase 5 depende de Fase 3 + 4**: As páginas precisam tanto dos endpoints backend quanto dos componentes frontend.
- **Fase 6 depende de Fase 5**: A finalização só faz sentido com todas as páginas prontas.

---

## Dependências Externas

| Dependência | Descrição | Status |
|-------------|-----------|--------|
| Gestão de Certificações (HalalSphere) | API de autenticação JWT RS256 | Em produção |
| AWS (S3, ECS, CloudFront) | Infraestrutura de deploy | Disponível |
| PostgreSQL 16 | Banco de dados com extensões | Configurado |
| Redis 7 | Cache de sessões e dados frequentes | Disponível |

---

## Métricas do Projeto

| Métrica | Valor |
|---------|-------|
| Épicos v1.0 | 6 |
| Épicos futuros | 1 (Inventário) |
| User Stories v1.0 | 33 |
| User Stories futuras | 6 |
| Story Points v1.0 | 202 |
| Módulos backend | 8 (Fase 3) |
| Páginas frontend | 22+ (Fase 5) |
| Componentes compartilhados | 8+ (Fase 4) |
| Modelos Prisma | 6 |
| Enums Prisma | 11 |
