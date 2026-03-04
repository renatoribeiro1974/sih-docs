---
title: Roadmap
parent: PRD
nav_order: 6
---

# 6. Roadmap de Implementação

---

## 6.1 Visão Geral

O SIH foi implementado em **9 fases**, mais **4 fases futuras**.

```
Fase 1: Scaffolding ────────→ Repos, configs, workspace           [COMPLETA]
Fase 2: Backend Core ───────→ Infra (Prisma, Auth, Health)        [COMPLETA]
Fase 3: Backend Dominio ────→ Modulos de negocio (8 modulos)      [COMPLETA]
Fase 4: Frontend Core ──────→ Layout, auth, componentes base      [COMPLETA]
Fase 5: Frontend Pages ─────→ Paginas + API hooks + design system [COMPLETA]
Fase 6: Finalizacao ────────→ Seed, Docker, builds                [COMPLETA]
Fase 7: Revisao Abrangente ─→ Fidelidade FM FAMBRAS, roles, PDF   [COMPLETA]
Fase 8: BI/Analytics ──────→ 6 dashboards Recharts + endpoints    [COMPLETA]
Fase 9: Colaboradores ─────→ CRUD, equipe por relatorio           [COMPLETA]
─────────────────────────────────────────────────────
Futuro A: 14 FMs Especializados → Subprodutos, Industrializados, Sidebar
Futuro B: Offline Completo ────→ IndexedDB, Background Sync
Futuro C: Inventario ──────────→ FM 7.1.5.x, FM 7.1.3.6
Futuro D: Integracao ──────────→ HalalSphere ↔ SIH ↔ SysHalal
```

---

## 6.2 Detalhamento por Fase

### Fase 1: Scaffolding (Repos + Configs)

**Objetivo**: Criar os 3 repositorios com toda a configuração de projeto.

| # | Tarefa | Entregavel |
|---|--------|-----------|
| 1 | Criar `sih-docs/` | Estrutura de docs + PRD completo |
| 2 | Criar `sih-backend/` | NestJS scaffolding (package.json, tsconfig, eslint, prettier, docker, prisma schema) |
| 3 | Criar `sih-frontend/` | React scaffolding (package.json, vite, tailwind, tsconfig, eslint, PWA manifest) |
| 4 | Criar workspace | `SupervisaoIndustrialHalal.code-workspace` |

**Verificação**: Repos criados, configs válidas, workspace abre corretamente.

---

### Fase 2: Backend Core

**Objetivo**: Infraestrutura funcional do backend.

| # | Tarefa | Entregavel |
|---|--------|-----------|
| 5 | Config module | `config.module.ts` + `aws-config.service.ts` |
| 6 | Prisma module | `prisma.module.ts` + `prisma.service.ts` + schema completo |
| 7 | Auth module | Auth self-contained (bcrypt + JWT HS256), guards, decorators, strategy |
| 8 | Health + Common | Health check, exception filter, logging interceptor, serial number útil |

**Verificação**: `npm run start:dev` funciona, `GET /health` retorna OK, Swagger acessível em `/api/docs`.

---

### Fase 3: Backend Domínio

**Objetivo**: Todos os módulos de negócio com CRUD completo.

| # | Tarefa | FM | Entregavel |
|---|--------|-----|-----------|
| 9 | SupervisorProfile | - | CRUD + auto-create on JWT access |
| 10 | Plant | - | CRUD completo |
| 11 | SlaughterReport | 7.1.4.x | CRUD + verificação C/NC + stunning + serial |
| 12 | ProductionReport | 7.1.3.x / 7.1.8.x | CRUD + raw materials + ingredients + serial |
| 13 | ShippingReport | 7.1.7.x / DCPOA | CRUD + 3 tipos + products + serial |
| 14 | NonConformity | 7.1.6.1 | CRUD + workflow + prazo 7 dias |
| 15 | Schedule | - | CRUD + unique constraint |
| 16 | Dashboard | - | Endpoints de agregação |

**Verificação**: Swagger mostra todos os endpoints, CRUD funciona via Swagger/Postman.

---

### Fase 4: Frontend Core

**Objetivo**: Base funcional do frontend com auth e layout.

| # | Tarefa | Entregavel |
|---|--------|-----------|
| 17 | Axios instances | `api.ts` (SIH backend) + `auth-api.ts` (SIH auth) + `utils.ts` |
| 18 | shadcn/ui components | 16 componentes base (Button, Card, Input, etc.) |
| 19 | Layout | AppLayout, Header, Sidebar |
| 20 | Auth flow | Login page, useAuth hook, route guards, JWT cache |
| 21 | Shared components | VerificationChecklist, ProductTable, ReportHeader |

**Verificação**: Login funciona (auth self-contained no SIH), layout renderiza, navegação funcional.

---

### Fase 5: Frontend Pages

**Objetivo**: Todas as páginas funcionais.

| # | Tarefa | Páginas |
|---|--------|---------|
| 22 | Dashboard | Dashboard.tsx (3 visoes por persona) |
| 23 | Abate | SlaughterReportList, SlaughterReportForm (com stunning), SlaughterReportDetails |
| 24 | Produção | ProductionReportList, ProductionReportForm (com raw materials), ProductionReportDetails |
| 25 | Embarque | ShippingReportList, ShippingReportForm (3 subtipos), ShippingReportDetails |
| 26 | NCs | NCList, NCForm, NCDetails (com workflow) |
| 27 | Escala | ScheduleCalendar, ScheduleManagement |
| 28 | Plantas | PlantList, PlantForm |

**Verificação**: Navegação completa, formulários funcionam, listagens com filtros, workflow de NCs.

---

### Fase 6: Finalização

**Objetivo**: Sistema pronto para uso.

| # | Tarefa | Entregavel |
|---|--------|-----------|
| 29 | Seed data | Plantas de teste, supervisores, relatórios de exemplo |
| 30 | Docker | docker-compose testado, Dockerfile multi-stage, init-extensions.sql |
| 31 | Git init | Repos inicializados, .gitignore validado |
| 32 | Build test | `npm install` + `npm run build` sem erros em ambos repos |

**Verificação**: Sistema completo funcionando end-to-end com dados de teste.

---

### Fase 7: Revisão Abrangente (Fidelidade FM FAMBRAS)

**Objetivo**: Alinhar formulários ao modelo real dos FMs FAMBRAS, ativar papel coordenador, melhorar navegação e adicionar funcionalidades administrativas.

| # | Tarefa | Entregavel |
|---|--------|-----------|
| 33 | Papel coordenador | Reativado com permissões corretas (read-only em relatórios, gerência de escalas/usuários/NCs, cancelamento) |
| 34 | Sidebar agrupada | Menu In Natura / Industrializados / Gestão com visibilidade por role |
| 35 | CRUD de usuários | Tela de gestão de usuários (admin cria todos, coordenador cria supervisor/operador) |
| 36 | NC dropdown | Seleção de relatório por dropdown (serial + data + tipo) em vez de UUID |
| 37 | Abate por espécie | Aves (FM 7.1.4.1): amperagem, voltagem, frequência, tempo cuba, velocidade. Bovino (FM 7.1.4.2): pressão, vitalidade, lesões. Verificações diferenciadas por espécie |
| 38 | Embarque enriquecido | Campos vendedor/cliente/endereço (venda interna), nº série Halal, tabela expandida com datas, pesos, temperatura |
| 39 | Produção enriquecida | Tabela matérias-primas com 8 colunas (proteína, frigorífico, SIF, data abate, CSN, cert. Halal, qtd, unid.) |
| 40 | Vinculação planta-FM | Tipo de planta filtra formulários disponíveis, espécie da planta filtra dropdown de espécie |
| 41 | PDF FAMBRAS | Geração PDF (PDFKit) com logo FAMBRAS, layout bilíngue, hash de assinatura no rodapé |

**Verificação**: Coordenador vê relatórios read-only, sidebar agrupada por tipo, abate aves com 5 parâmetros ×2 horários, PDF fiel ao modelo FAMBRAS.

---

## 6.3 Fases Pós-v1.0

### Fase 8: BI/Analytics (COMPLETA)

**Objetivo**: Módulo de Business Intelligence com gráficos interativos.

| # | Tarefa | Entregável |
|---|--------|-----------|
| 42 | Recharts + componentes base | ChartCard, KPICard, DateRangeFilter, PlantFilter |
| 43 | Backend analytics (6 endpoints) | reports-trend, slaughter/production/shipping/nc/supervisor-analytics |
| 44 | Frontend analytics (6 páginas) | AnalyticsOverview, Slaughter, Production, Shipping, NC, Supervisor |
| 45 | Sidebar + rotas | Item "Analytics" (BarChart3), 6 rotas /analytics/* |

**Verificação**: Filtros de período e planta funcionam, gráficos renderizam, builds passam.

---

### Fase 9: Colaboradores (COMPLETA)

**Objetivo**: Cadastro de profissionais não-usuários e vinculação como equipe em relatórios.

| # | Tarefa | Entregável |
|---|--------|-----------|
| 46 | Schema Prisma + Migration | 3 modelos (Collaborator, CollaboratorPlant, ReportStaff) + enum CollaboratorType |
| 47 | Backend Collaborator | CRUD 10 endpoints, `@Roles('admin', 'coordenador')` |
| 48 | Frontend Collaborator | CollaboratorList + CollaboratorForm + sidebar "Colaboradores" |
| 49 | ReportStaff Integration | `syncReportStaff()`, `ReportStaffSelector` nos 3 formulários |
| 50 | Seed Data | 5 colaboradores de exemplo vinculados a plantas |

**Pendente**: PDF com seção "Equipe/Staff" antes da assinatura.

**Verificação**: Backend build OK, frontend build OK, seed OK.

---

## 6.4 Fases Futuras

### Fase Futura A: Formulários Especializados (14 FMs)

**Prerequisito**: Fase 9 completa.

Expandir de 6 para 20 variantes de FM usando arquitetura Discriminador + JSON.

| Família | FMs Novos | Descrição |
|---------|-----------|-----------|
| Produção (7 novos) | FM 7.1.3.3, 7.1.3.5, 7.1.4.5, 7.1.4.6, 7.1.8.2, 7.1.8.5, 7.1.8.6 | Tripas, Fracionamento, Couro, Mucosa, Heparina, Raspa, Gelatina |
| Embarque (6 novos) | FM 7.1.7.2, 7.1.7.5, 7.1.7.9, 7.1.7.12, 7.1.4.8.B, 7.1.8.4 | Export. Ind., Venda Ind., Transf. Ind., Emb. Subprod., Transf. In Natura, Transf. Subprod. |
| Sidebar | — | Novo grupo "Subprodutos" + expansão de "In Natura" e "Industrializados" |
| PDF | — | 13 novos templates especializados |

Referência detalhada: [PLANNING/FASE-A-COLABORADORES.md](../PLANNING/FASE-A-COLABORADORES.md) — Parte 2.

### Fase Futura B: Offline Completo

**Prerequisito**: v1.0 em produção, PWA base já instalada.

| Componente | Descrição |
|-----------|-----------|
| IndexedDB (Dexie.js) | Armazena relatórios preenchidos offline no tablet |
| Background Sync | Envia relatórios pendentes automáticamente quando reconectar |
| Cache de referência | Plantas, templates de verificação, perfil do supervisor |
| Indicador visual | Badge online/offline + qtd pendente de sincronização |
| Resolução de conflitos | Timestamp-based (último ganha) ou merge manual para NCs |

### Fase Futura C: Inventário

**Prerequisito**: v1.0 em produção, relatórios gerando dados.

| Componente | FM | Descrição |
|-----------|-----|-----------|
| Conta corrente de carne | 7.1.5.1 | Recebido vs. utilizado em produção |
| Inventário de lotes | 7.1.5.6 | Gerado vs. transferido entre unidades |
| Inventário de rotulagem | 7.1.3.6 | Sem rótulo → rotulado → embarcado |
| Dashboard de inventário | - | Visão consolidada de estoques |
| Migração de dados | - | Importação de planilhas Excel historicas |

### Fase Futura D: Integração entre Sistemas

**Prerequisito**: HalalSphere e SysHalal em produção.

| Componente | Direcao | Descrição |
|-----------|---------|-----------|
| Cadastros compartilhados | HalalSphere → SIH | Plantas/empresas sincronizadas via API REST |
| Dados de supervisão | SIH → SysHalal | Relatórios assinados alimentam emissão de certificados |
| Notificações | SIH → HalalSphere | Webhooks para NCs críticas e resumos |
| SSO | HalalSphere → SIH | Autenticação centralizada (substituindo self-contained) |

---

## 6.4 Dependências Externas

| Dependência | Descrição | Status |
|-------------|-----------|--------|
| PostgreSQL 16 | Banco de dados (porta 5433) | Disponível |
| Redis 7 | Cache (porta 6380) | Disponível |
| HalalSphere | Integração futura — cadastros e SSO | Em produção (independente na v1.0) |
| SysHalal | Integração futura — dados de supervisão | Em produção (independente na v1.0) |
| AWS (S3, ECS) | Infraestrutura de deploy (futuro) | Disponível |
