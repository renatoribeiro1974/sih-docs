# Briefing de Execução — Reunião Cliente 2026-04-13

> Documento de referência para continuidade do planejamento e execução.
> Plano detalhado completo em: `PLANNING/PLANO-REUNIAO-2026-04-13.md`

---

## Contexto

Reunião com cliente FAMBRAS gerou 12 tasks para o SIH (Supervisão Industrial Halal).
Após análise de negócio e refinamento técnico, o plano ficou com **9 tasks ativas**
em **5 sprints (60 SP)**, 1 task postergada e 1 no backlog futuro.

**Repositórios:** sih-backend (NestJS + Prisma + PostgreSQL), sih-frontend (React 19), sih-docs

---

## Regras de Negócio (RN) — Todas cobertas

| RN | Regra | Task |
|----|-------|------|
| RN-01 | Relatório válido = assinatura Supervisor + aprovação Controladoria | TASK-01 |
| RN-02 | Rejeição exige motivo obrigatório | TASK-01/02 |
| RN-03 | Só o supervisor original pode remover assinatura após rejeição | TASK-01 |
| RN-04 | Controlador segmentado por grupo (IN/IND) + acesso extra por empresa | TASK-03 |
| RN-05 | Anexo de documentos em venda/transferência (exceto abate) | TASK-09 |
| RN-06 | Degolador não se aplica a abate de frango (são muitos, inviável listar) | TASK-08 |
| RN-07 | Inventário com Certificado Halal vinculado ao SysHalal | TASK-11 |

---

## Sprints e Tasks

### Sprint 1 — Fundação (13 SP)

**TASK-02 — Perfil Controladoria (8 SP)**
- Nova role `controlador` no enum `SupervisorRole`
- Campos: `companyGroup` (IN/IND), `extraPlantAccess` (JSON), `isManager` (boolean)
- Controlador: só pode aprovar/rejeitar, não cria nem edita relatórios
- Gestor (`isManager`): pode reatribuir relatórios, ver ambos os grupos

**TASK-03 — Segmentação por Grupo Empresarial (5 SP)**
- Campo `companyGroup` na tabela `Plant` (IN ou IND)
- Filtro app-level via decorator `@CompanyGroupFilter` no NestJS
- Regra: controlador vê plantas do seu grupo + plantas extras (exceção para híbridas)
- Dentro do mesmo grupo, todos controladores veem a mesma fila (sem segregação)
- Exceção: plantas híbridas (abatedouro + processadora) — controlador IN recebe acesso extra

### Sprint 2 — Fluxo de Aprovação (16 SP)

**TASK-01 — Duplo Check de Validação (8 SP)**
- Novo fluxo: `rascunho → assinado → aprovado / rejeitado`
- Relatório rejeitado é **editado** (não cria novo) — supervisor corrige e re-assina
- `statusHistory` (JSON, append-only) registra todas as transições:
  `{ from, to, by, at, reason?, action? }`
- Mecanismo de atribuição: controlador **assume** relatório antes de avaliar
  - `assignedToId`, `assignedAt` em cada relatório
  - Só pode aprovar/rejeitar o que assumiu
  - Gestor pode reatribuir ou devolver à fila
- Relatórios existentes em `assinado` **NÃO migram** — ficam como estão

**TASK-04 — Dashboard Controladoria (8 SP)**
- Dashboard operacional (fila de trabalho), não analítico
- **Visão analista**: fila pendente (assumir) | meus (aprovar/rejeitar/devolver) | histórico
- **Visão gestor** (isManager): métricas por analista (tempo médio, volume, taxa rejeição, represados)
- Indicadores de aging: verde (<24h), amarelo (24-48h), vermelho (>48h)
- Fila unificada com todos os tipos de relatório (abate, produção, embarque)

### Sprint 3 — Quick Wins (5 SP)

**TASK-05 — Padronizar Transferências (3 SP)**
- Revisar 5 subtipos de transferência e garantir campos origem/destino em todos
- Padronizar frontend (formulários) + PDF
- Não é bug — é que nem todos os subtipos foram implementados com esses campos

**TASK-08 — Degolador no Abate de Aves (2 SP)**
- Ocultar seção de identificação individual do degolador quando `species === 'ave'`
- Motivo: muitos degoladores simultâneos na linha, inviável listar
- Manter para bovino (poucos, identificáveis)
- Checklist "Tasmya" permanece em aves (verificação do processo, não identificação)

### Sprint 4 — Notificações + Anexos (13 SP)

**TASK-06 — Notificações (8 SP)**
- **In-app**: todas as transições (pendente aprovação → controladores, aprovado/rejeitado → supervisor, reatribuição → novo analista)
- **E-mail**: apenas rejeição → supervisor responsável (via AWS SES)
- E-mail é best-effort (falha não bloqueia)
- Badge no sino do header (já existe ícone, não funcional)
- Tabela `notifications` com recipientId, type, metadata (JSONB), readAt

**TASK-09 — Anexos em Venda/Transferência (5 SP)**
- Upload de documentos (CSI, CSN, Invoice, BL, Nota Fiscal)
- S3 via presigned URL, Multer para multipart
- Categoria do documento selecionável (enum)
- Apenas em rascunho; visível em todos os status
- Infra reutilizável para TASK-11
- **Sem OCR** por ora (backlog futuro via Tesseract/pdf-parse)

### Sprint 5 — Desossa + Certificado Halal (13 SP)

**TASK-07 — Relatório de Desossa Bovinos (5 SP)**
- **BLOQUEADA**: aguardando FM FAMBRAS de referência do cliente
- Quando desbloqueada: novo `ProductionType: 'desossa'` (reutiliza infra existente)
- Componente `DesossaFields.tsx` + customFields JSON + checklist específico

**TASK-11 — Certificado Halal em Inventário e Transferências (8 SP)**
- Afeta: MeatInventoryReceipt, BatchInventory, ShippingReport (transferencias)
- Integração com API SysHalal (operacional):
  - `GET /certified?certificate-number=XXX` — buscar certificado
  - `GET /certified_status?certificate-number=XXX` — verificar vigência
  - `GET /certified_pdf?certificate-number=XXX` — download PDF direto
  - Auth: headers `user-token` + `user-owner` via Secrets Manager
- Fluxo: digita número → busca SysHalal → encontrou = preenche auto + PDF disponível / não encontrou = upload manual
- Documentação da API: `c:\sih\Syshalal External API Integration Manual.pdf` (v2.0)

---

## Postergada

**TASK-10 — Integração Cadastro SIH ↔ CERT**
- Aguarda validação completa do sistema de Gestão de Certificações
- Cadastro de empresas no SIH segue funcional de forma independente

## Backlog Futuro

**TASK-12 — OCR para Documentos Anexados**
- Tesseract.js (WASM) para scans/fotos + pdf-parse para PDFs digitais
- Sem custo de IA
- Depende de TASK-09 estar em produção

---

## Decisões Técnicas Importantes

1. **Não usa Supabase** — NestJS + Prisma + PostgreSQL direto. Nunca mencionar Supabase.
2. **Segmentação app-level** via decorator/interceptor, não RLS nativo do PostgreSQL
3. **statusHistory em JSON** — append-only, padrão consistente com verificationItems/customFields
4. **Relatório rejeitado é editado, não substituído** — mesmo registro, mesmo serialNumber
5. **Toda feature é produção** — sistema em uso real, nunca tratar como POC
6. **Enum `ReportStatus`**: valores `aprovado` e `rejeitado` já existiram e foram removidos na migration `20260226050000` — readicionar

---

## Pendências com o Cliente

| # | Pendência | Impacto |
|---|-----------|---------|
| 1 | FM FAMBRAS de referência para desossa | Bloqueia TASK-07 |
| 2 | Mapeamento de plantas → grupo IN/IND (seed data) | Necessário para testar TASK-03 |

---

## Arquivos de Referência

| Arquivo | Conteúdo |
|---------|----------|
| `PLANNING/PLANO-REUNIAO-2026-04-13.md` | Plano detalhado completo com todas as tasks |
| `c:\sih\Syshalal External API Integration Manual.pdf` | Documentação API SysHalal v2.0 |
| `prisma/schema.prisma` (sih-backend) | Schema atual do banco |
| `src/auth/guards/roles.guard.ts` (sih-backend) | Guard de roles existente |
| `src/components/shared/StatusBadge.tsx` (sih-frontend) | Componente de status a estender |
| `src/components/layout/Header.tsx:44-46` (sih-frontend) | Ícone de sino (futuras notificações) |
| `src/pages/slaughter/SlaughterReportForm.tsx:409-430` (sih-frontend) | Seção degolador a condicionar |

---

## Ordem de Execução Recomendada

```
Sprint 1: TASK-02 → TASK-03 (sequenciais, TASK-03 depende dos campos de TASK-02)
Sprint 2: TASK-01 → TASK-04 (sequenciais, TASK-04 depende do fluxo de TASK-01)
Sprint 3: TASK-05 + TASK-08 (paralelas, independentes)
Sprint 4: TASK-06 + TASK-09 (paralelas, independentes)
Sprint 5: TASK-07 (se desbloqueada) + TASK-11 (paralelas)
```

Para cada task, o plano detalhado contém: contexto de negócio, alterações backend,
alterações frontend, critérios de aceite. Seguir o `PLANO-REUNIAO-2026-04-13.md`.

---

*Gerado em 2026-04-13. Pronto para execução.*
