# Plano de Execução — Reunião com Cliente 2026-04-13

> 12 Tasks priorizadas, organizadas em 5 Sprints com dependências mapeadas.
> TASK-10 postergada. TASK-12 no backlog futuro.

---

## Regras de Negócio

| RN | Descrição | Task |
|----|-----------|------|
| **RN-01** | Relatório válido = assinatura do Supervisor + aprovação da Controladoria | TASK-01 |
| **RN-02** | Rejeição exige registro de motivo obrigatório | TASK-01, TASK-02 |
| **RN-03** | Somente o supervisor que assinou pode remover assinatura após rejeição | TASK-01 |
| **RN-04** | Controlador com acesso por grupo (IN/IND) + acesso extra por empresa | TASK-03 |
| **RN-05** | Relatórios de venda/transferência (exceto abate) suportam anexo de documentos | TASK-09 |
| **RN-06** | Campo "degolador" não se aplica a abate de frango | TASK-08 |
| **RN-07** | Inventário pode ter Certificado Halal anexado, vinculado ao SysHalal | TASK-11 |

---

## Mapa de Dependências

```
TASK-02 (Perfil Controladoria) ──┐
                                 ├──► TASK-01 (Duplo Check) ──► TASK-06 (Notificações)
TASK-03 (Segmentação por Grupo) ─┤
                                 └──► TASK-04 (Dashboard Controladoria)

TASK-05 (Bug Transferências) ──────── independente
TASK-08 (Remover Degolador) ───────── independente
TASK-07 (Relatório Desossa) ───────── independente
TASK-09 (Anexos Venda/Transf) ─────── independente
TASK-10 (Integração Cadastro) ─────── independente
TASK-11 (Certificado Halal Inv) ───── reutiliza infra da TASK-09
TASK-12 (OCR Documentos) ─────────── BACKLOG FUTURO (depende de TASK-09)
```

---

## Sprint 1 — Fundação: Perfil Controladoria + Segmentação por Grupo (Alta Prioridade)

### TASK-02 — Perfil "Controladoria" (Est: 8 SP)

**Contexto de negócio:**
O Supervisor trabalha localmente na planta e é responsável pelos relatórios, validando-os
com sua assinatura. Porém, a assinatura do Supervisor sozinha não é suficiente — a Controladoria
FAMBRAS precisa avaliar e aprovar (ou rejeitar) cada relatório assinado. Esse perfil não existe
hoje no sistema.

**Contexto técnico:**
- Roles existentes: `admin`, `coordenador`, `supervisor`, `operador`
- Auth via JWT com role no payload (`src/auth/types/jwt-payload.type.ts`)
- Guard de roles em `src/auth/guards/roles.guard.ts`

**Decisão: Nova role `controlador` no enum `SupervisorRole`**

Isolamento total — sem risco de conflito com roles existentes.

**Backend — Alterações:**

1. **Migration**: Adicionar `controlador` ao enum `SupervisorRole`
   ```sql
   ALTER TYPE "SupervisorRole" ADD VALUE 'controlador';
   ```

2. **Prisma schema** (`prisma/schema.prisma`):
   - Adicionar `controlador` ao enum `SupervisorRole`
   - Novos campos no model `SupervisorProfile`:
     ```prisma
     companyGroup       String?    // "IN" ou "IND" — obrigatório para controlador
     extraPlantAccess   Json?      // ["uuid-plant-1", ...] — plantas do grupo oposto
     isManager          Boolean    @default(false)  // gestor de controladoria
     ```
   - `isManager = true`: pode reatribuir relatórios, liberar de outros analistas,
     ver ambos os grupos IN e IND

3. **Auth module** (`src/auth/`):
   - `jwt-payload.type.ts`: Adicionar `companyGroup?: string` ao payload
   - `auth.service.ts`: Incluir `companyGroup` no token ao fazer login
   - `roles.guard.ts`: Sem alteração (já é genérico)

4. **Supervisor Profile** (`src/supervisor-profile/`):
   - `supervisor-profile.service.ts`: Validar que `companyGroup` é obrigatório quando role = `controlador`
   - DTO de criação: Incluir campos `companyGroup` e `extraCompanyAccess`

5. **Report Controllers** (slaughter, production, shipping):
   - Adicionar `@Roles('controlador')` aos endpoints de leitura (GET)
   - **BLOQUEAR** edição/criação para `controlador` (403 Forbidden)
   - Criar novo endpoint `PATCH :id/approve` (role: controlador)
   - Criar novo endpoint `PATCH :id/reject` com body `{ reason: string }` (role: controlador, reason obrigatório)

**Frontend — Alterações:**

6. **Types** (`src/types/auth.types.ts`):
   - Adicionar `'controlador'` ao type `UserRole`

7. **Guards** (`src/components/guards/RoleRoute.tsx`):
   - Incluir `controlador` nas rotas permitidas de visualização

8. **Forms de relatório** (SlaughterReportForm, ProductionReportForm, ShippingReportForm):
   - `controlador` = readOnly (não pode criar nem editar)
   - Exibir botões "Aprovar" e "Rejeitar" quando status = `assinado` e role = `controlador`
   - Modal de rejeição com campo "Motivo" obrigatório (textarea + validação)

9. **Sidebar** (`src/components/layout/Sidebar.tsx`):
   - Adicionar menu "Controladoria" visível apenas para role `controlador`

**Critérios de aceite:**
- [ ] Controlador não consegue criar/editar relatórios
- [ ] Controlador consegue aprovar relatórios assinados
- [ ] Controlador consegue rejeitar com motivo obrigatório
- [ ] Motivo de rejeição é persistido no banco
- [ ] Supervisor original consegue editar relatório rejeitado e re-assinar

---

### TASK-03 — Segmentação de Dados por Grupo Empresarial (Est: 5 SP)

**Contexto de negócio:**
A Controladoria FAMBRAS opera em dois grupos: **IN** (In Natura) e **IND** (Industrializados).
Cada grupo avalia relatórios das plantas do seu grupo. Dentro do mesmo grupo, **não há
segregação** — todos os controladores IN veem e podem assumir qualquer relatório IN.

**Exceção — Plantas híbridas:**
Algumas plantas são abatedouro E processadora (ex: abate + industrialização). Essas plantas
pertencem ao grupo IND, mas seus relatórios de abate precisam ser avaliados por um controlador
IN. Para isso, o admin habilita um controlador IN específico com acesso extra àquela planta IND.

**Regra de visibilidade do controlador:**
```
Relatórios visíveis = relatórios de plantas do meu grupo (IN ou IND)
                    + relatórios de plantas extras (extraPlantAccess)
```

**Abordagem: Filtro app-level com decorator `@CompanyGroupFilter`**

Consistente com a arquitetura existente — decorator/interceptor no NestJS que injeta filtro
WHERE automaticamente nos queries de relatório para o perfil controlador.

**Backend — Alterações:**

1. **Model Plant** — Adicionar campo:
   ```prisma
   companyGroup  String?  // "IN" ou "IND"
   ```

2. **Model SupervisorProfile** — Campos de segmentação (já definidos na TASK-02):
   ```prisma
   companyGroup       String?   // "IN" ou "IND" — grupo principal
   extraPlantAccess   Json?     // ["uuid-plant-1", "uuid-plant-2"] — exceções
   ```

3. **Migration**:
   ```sql
   ALTER TABLE plants ADD COLUMN "companyGroup" TEXT;
   ```

4. **Novo decorator** `@CompanyGroupFilter()`:
   ```typescript
   // src/common/decorators/company-group-filter.decorator.ts
   // Extrai companyGroup e extraPlantAccess do JWT
   // Injeta filtro WHERE nos queries de relatório
   ```

5. **Novo interceptor** `CompanyGroupInterceptor`:
   - Para role `controlador`:
     - WHERE `plant.companyGroup = user.companyGroup`
       OR `plant.id IN user.extraPlantAccess`
   - Para outras roles: sem filtro adicional (supervisores, operadores, etc.)

6. **Services de relatório** (slaughter, production, shipping):
   - `findAll()`: Aceitar filtro de grupo
   - Aplicar automaticamente quando `user.role = 'controlador'`

**Frontend — Alterações:**

7. **PlantForm** (admin): Campo para definir `companyGroup` (IN/IND)
8. **UserForm** (admin): Para controladores, campo `companyGroup` (obrigatório)
   e seleção de plantas extras (`extraPlantAccess`) — multiselect de plantas
   do grupo oposto

**Critérios de aceite:**
- [ ] Controlador IN vê todos os relatórios de plantas IN
- [ ] Controlador IND vê todos os relatórios de plantas IND
- [ ] Dentro do mesmo grupo, todos os controladores veem a mesma fila
- [ ] Admin consegue habilitar acesso extra a plantas do grupo oposto (exceção para plantas híbridas)
- [ ] Supervisores, operadores e coordenadores não são afetados pelo filtro

---

## Sprint 2 — Fluxo de Aprovação Dupla (Alta Prioridade)

### TASK-01 — Duplo Check de Validação (Est: 8 SP)

**Depende de:** TASK-02 (perfil controladoria) + TASK-03 (segmentação por grupo)

**Contexto de negócio:**
A assinatura do Supervisor não é suficiente para validar o relatório. A Controladoria
precisa avaliar cada relatório assinado. Se rejeitado, o relatório **não é descartado** —
o supervisor edita o mesmo relatório e re-assina. Histórico completo de transições
deve ser mantido em JSON para auditoria.

**Contexto técnico:**
- Fluxo atual: `rascunho` → `assinado` → `cancelado`
- Os valores `aprovado` e `rejeitado` já existiram no enum e foram removidos
  na migration `20260226050000_remove_legacy_enums` — precisam ser readicionados

**Novo ciclo de vida do relatório:**
```
RASCUNHO ──[Supervisor assina]──► ASSINADO ──[Controlador aprova]──► APROVADO ✓
                                     │
                                     └──[Controlador rejeita]──► REJEITADO
                                                                     │
                                                  [Supervisor edita e re-assina]
                                                                     │
                                                                     ▼
                                                                 RASCUNHO
                                                          (edita → re-assina)
```

**Regra-chave:** Relatório rejeitado é editado, não substituído.
O mesmo registro é reaproveitado — o supervisor corrige e assina novamente.

**Backend — Alterações:**

1. **Migration**: Adicionar valores ao enum `ReportStatus`:
   ```sql
   ALTER TYPE "ReportStatus" ADD VALUE 'aprovado';
   ALTER TYPE "ReportStatus" ADD VALUE 'rejeitado';
   ```

2. **Schema Prisma** — Novos campos em cada report model:
   ```prisma
   statusHistory   Json?       // Array de transições (audit trail)
   ```
   Formato do `statusHistory`:
   ```json
   [
     {
       "from": "rascunho",
       "to": "assinado",
       "by": "uuid-supervisor",
       "at": "2026-04-13T10:00:00Z"
     },
     {
       "from": "assinado",
       "to": "rejeitado",
       "by": "uuid-controlador",
       "at": "2026-04-13T14:30:00Z",
       "reason": "Dados de temperatura inconsistentes"
     },
     {
       "from": "rejeitado",
       "to": "rascunho",
       "by": "uuid-supervisor",
       "at": "2026-04-13T15:00:00Z"
     },
     {
       "from": "rascunho",
       "to": "assinado",
       "by": "uuid-supervisor",
       "at": "2026-04-13T16:00:00Z"
     },
     {
       "from": "assinado",
       "to": "aprovado",
       "by": "uuid-controlador",
       "at": "2026-04-14T09:00:00Z"
     }
   ]
   ```
   Consistente com o padrão existente no projeto (verificationItems, customFields, products).

3. **Novos campos em cada report model** (atribuição):
   ```prisma
   assignedToId    String?     // controlador que assumiu o relatório
   assignedAt      DateTime?   // quando assumiu
   ```

4. **Novos endpoints** (em cada report controller):
   - `PATCH /:id/assign` — `@Roles('controlador')`
     - Valida: status == 'assinado', assignedToId == null (livre na fila)
     - Seta: assignedToId = user.id, assignedAt = now
     - Append no statusHistory: { action: 'assigned', by, at }
   - `PATCH /:id/release` — `@Roles('controlador')`
     - Valida: assignedToId == user.id OU user.isManager == true
     - Limpa: assignedToId, assignedAt
     - Append no statusHistory: { action: 'released', by, at }
   - `PATCH /:id/reassign` — controlador com `isManager == true`
     - Valida: user.isManager == true
     - Seta: assignedToId = body.targetUserId, assignedAt = now
     - Append no statusHistory: { action: 'reassigned', by, at, to: targetUserId }
   - `PATCH /:id/approve` — `@Roles('controlador')`
     - Valida: status == 'assinado', assignedToId == user.id (tem que ter assumido)
     - Seta: status = 'aprovado'
     - Append no statusHistory: { from, to, by, at }
   - `PATCH /:id/reject` — `@Roles('controlador')`
     - Valida: status == 'assinado', assignedToId == user.id, body.reason não vazio
     - Seta: status = 'rejeitado'
     - Append no statusHistory: { from, to, by, at, reason }
   - `PATCH /:id/reopen` — `@Roles('supervisor')`
     - Valida: status == 'rejeitado', user.id == signedById (mesmo supervisor)
     - Seta: status = 'rascunho', limpa signedById/signedAt/signatureHash,
       limpa assignedToId/assignedAt
     - Append no statusHistory: { from, to, by, at }
     - Relatório fica editável para o supervisor corrigir e re-assinar

5. **Regras de negócio**:
   - Relatório só é VÁLIDO para fins regulatórios quando status = `aprovado`
   - `cancelado` só pode ser aplicado a relatórios `aprovado` (pelo coordenador/admin)
   - PDF pode ser gerado para status `assinado` ou `aprovado`
   - statusHistory é append-only (nunca editado, nunca removido)
   - Controlador só pode aprovar/rejeitar relatório que ele assumiu
   - Gestor (isManager) pode liberar/reatribuir relatórios de qualquer analista

**Frontend — Alterações:**

6. **StatusBadge** (`src/components/shared/StatusBadge.tsx`):
   - Adicionar: `aprovado` (verde forte), `rejeitado` (vermelho)
   - Indicador visual "Em análise por [nome]" quando assignedToId != null

7. **Forms de relatório** — Lógica condicional:
   ```
   Se role == 'controlador' && status == 'assinado' && assignedToId == null:
     Exibir botão [Assumir para Análise]
   Se role == 'controlador' && assignedToId == user.id:
     Exibir botões [Aprovar] [Rejeitar] [Devolver à Fila]
   Se role == 'controlador' && isManager && assignedToId != null:
     Exibir botões [Reatribuir] [Devolver à Fila]
   Se role == 'supervisor' && status == 'rejeitado' && user.id == signedById:
     Exibir botão [Reabrir para Edição]
     Exibir card com motivo da rejeição (destaque visual)
   Se status == 'rascunho' && statusHistory contém rejeição anterior:
     Exibir banner "Relatório devolvido pela Controladoria" com motivo
   ```

8. **Histórico de transições**: Componente colapsável no relatório mostrando
   timeline do statusHistory (quem, quando, ação, motivo se houver)

9. **Lista de relatórios**: Filtro por novos status (aprovado, rejeitado)
   e por atribuição (meus / na fila / todos)

**Critérios de aceite:**
- [ ] Status reflete: Rascunho → Assinado → Aprovado / Rejeitado
- [ ] Controlador precisa assumir o relatório antes de aprovar/rejeitar
- [ ] Relatório assumido fica visível como "em análise por [nome]"
- [ ] Controlador pode devolver relatório à fila (liberar)
- [ ] Gestor (isManager) pode reatribuir ou liberar relatório de qualquer analista
- [ ] Motivo de rejeição é obrigatório e visível
- [ ] Relatório rejeitado é editado pelo mesmo supervisor (não cria novo)
- [ ] Supervisor corrige e re-assina — volta para fila da controladoria
- [ ] statusHistory (JSON) registra todas as transições incluindo atribuições
- [ ] Histórico visível no relatório para consulta/auditoria

---

### TASK-04 — Dashboard Controladoria (Est: 8 SP)

**Depende de:** TASK-02 + TASK-03 + TASK-01

**Contexto de negócio:**
Dashboard operacional para a Controladoria — fila de trabalho compartilhada onde
analistas assumem relatórios para avaliação. Gestores (isManager) têm visão
adicional com indicadores de performance da equipe.

**Backend — Alterações:**

1. **Endpoints de fila** `GET /controladoria/queue`:
   - Retorna relatórios assinados filtrados por grupo do controlador
   - Filtros: na fila (assignedToId == null), meus (assignedToId == user.id), todos
   - Todos os tipos de relatório (slaughter, production, shipping) em lista unificada
   - Paginação e filtros por data, planta, tipo

2. **Endpoint de histórico** `GET /controladoria/history`:
   - Relatórios aprovados e rejeitados
   - Filtros por período, planta, tipo, analista

3. **Endpoint de indicadores** `GET /controladoria/metrics`:
   - Total na fila (não atribuídos)
   - Total em análise (atribuídos)
   - Aprovados/rejeitados no período
   - Tempo médio entre assinatura → aprovação
   - Tempo médio de análise (assignedAt → aprovação/rejeição)
   - Relatórios represados (em análise há mais de X horas)
   - **Apenas para isManager:** métricas por analista:
     - Relatórios atribuídos por analista
     - Tempo médio de cada analista
     - Taxa de rejeição por analista

4. **Query consolidada**: Union dos 3 tipos de relatório com campos comuns:
   ```typescript
   {
     id, serialNumber, type, plantName, date, status,
     signedBy, signedAt,
     assignedTo, assignedAt,        // quem está analisando
     approvedBy, approvedAt,
     rejectedBy, rejectedAt, rejectionReason
   }
   ```

**Frontend — Alterações:**

5. **Nova página** `ControladoriaDashboard.tsx`:

   **Visão Analista (todos os controladores):**
   - **KPIs**: Na fila | Em análise (meus) | Aprovados hoje | Rejeitados hoje
   - **Fila de Pendentes** (assignedToId == null): lista com botão [Assumir]
   - **Meus** (assignedToId == user.id): lista com ações [Aprovar] [Rejeitar] [Devolver]
   - **Histórico**: aprovados e rejeitados com filtros

   **Visão Gestor (isManager = true) — aba adicional:**
   - **Indicadores de performance da equipe:**
     - Tempo médio de análise por analista
     - Volume processado por analista (período)
     - Taxa de rejeição por analista
     - Relatórios represados (em análise há muito tempo) — aging visual
   - **Gestão de atribuições:**
     - Ver todos os relatórios em análise com nome do analista
     - Ações: [Reatribuir] [Devolver à Fila]

6. **Tabs por tipo**: Abate | Produção | Embarque | Todos

7. **Indicadores visuais de aging** na fila:
   - Verde: < 24h na fila
   - Amarelo: 24-48h
   - Vermelho: > 48h

8. **Rota**: `/controladoria` — acessível apenas para role `controlador`

9. **Sidebar**: Item "Controladoria" com ícone de shield/check

**Critérios de aceite:**
- [ ] Dashboard carrega relatórios filtrados por grupo do controlador logado
- [ ] Fila compartilhada — todos do grupo veem os mesmos relatórios na fila
- [ ] Controlador pode assumir relatório da fila
- [ ] Relatório assumido aparece como "em análise por [nome]"
- [ ] Ações disponíveis apenas em relatórios assumidos pelo próprio
- [ ] Gestor vê indicadores de performance por analista
- [ ] Gestor pode reatribuir ou devolver relatórios de qualquer analista
- [ ] Indicadores de aging (verde/amarelo/vermelho) por tempo na fila
- [ ] Todos os tipos de relatório presentes na fila unificada

---

## Sprint 3 — Bug Fix + Quick Wins (Paralelo)

### TASK-05 — Padronizar Campos de Origem/Destino nas Transferências (Est: 3 SP)

**Contexto de negócio:**
Testes em sala durante a reunião com o cliente identificaram que há tipos de relatório
de transferência que não mostram ou não habilitam os campos de origem e destino.
Não é um bug de código — é que nem todos os subtipos foram implementados com esses campos.
Precisa revisar e padronizar.

**Subtipos de transferência a revisar:**
- `transferencia`
- `transferencia_industrializados`
- `transferencia_in_natura`
- `transferencia_subprodutos`
- `transferencia_generica`

**Ações:**
1. **Levantar estado atual**: Para cada subtipo, verificar quais campos de
   origem/destino estão habilitados no frontend (ShippingReportForm + shipping-types/)
2. **Padronizar frontend**: Garantir que todos os subtipos de transferência
   renderizem campos de origem E destino
3. **Padronizar PDF**: Garantir que `shipping-report-pdf.ts` inclua origem
   e destino para todos os subtipos de transferência
4. **Backend**: Verificar se `create()`/`update()` persistem os campos
   e se `findOne()` retorna todos os dados necessários
5. **Validação**: Considerar tornar destino obrigatório para tipos de transferência

**Critérios de aceite:**
- [ ] Todos os 5 subtipos de transferência exibem campos de origem E destino
- [ ] PDF de transferência inclui ambos endereços para todos os subtipos
- [ ] Dados existentes no banco são exibidos corretamente
- [ ] Comportamento padronizado e consistente entre todos os subtipos

---

### TASK-08 — Ocultar Identificação Individual de Degolador no Abate de Aves (Est: 2 SP)

**Contexto de negócio:**
No abate de aves, há muitos degoladores atuando simultaneamente na linha de produção
durante um turno, tornando inviável a identificação individual de cada um no relatório.
No abate bovino, são poucos degoladores e a identificação individual faz sentido.

**Regra:** A seção de identificação do degolador (nome + documento) aparece apenas
para relatórios de abate **bovino**. Para **aves**, a seção não é exibida.

**Nota:** O item de verificação do checklist de aves ("Degolador muçulmano invocou a Tasmya")
**permanece** — é uma verificação do processo (os degoladores seguiram o procedimento),
não uma identificação individual.

**Alterações:**

1. **Frontend** — `SlaughterReportForm.tsx` (linhas 409-430):
   - Seção "Degolador" (campos `slaughtererName`, `slaughtererDoc`):
     renderizar apenas quando `species === 'bovino'`

2. **Frontend** — `ReportStaffSelector.tsx`:
   - `degolador` no typeLabels — **manter** (usado em bovino)

3. **Backend** — `slaughter-report.service.ts`:
   - Campos `slaughtererName`, `slaughtererDoc` — **manter** no schema (nullable)
   - Validação: não exigir para species=ave

4. **PDF** — `slaughter-report-pdf.ts`:
   - Renderizar seção degolador apenas quando species=bovino
   - Item de verificação "Tasmya" permanece no checklist de aves

**Critérios de aceite:**
- [ ] Formulário de abate de AVES não exibe seção de identificação do Degolador
- [ ] Formulário de abate de BOVINO continua exibindo Degolador normalmente
- [ ] PDF de aves não inclui seção de identificação do degolador
- [ ] Checklist de aves mantém item de verificação "Tasmya"
- [ ] Dados existentes de bovino não são afetados

---

## Sprint 4 — Notificações + Anexos (Média-Alta)

### TASK-06 — Sistema de Notificações (Est: 8 SP)

**Depende de:** TASK-01 (novos status aprovado/rejeitado)

**Canais definidos:**
- **In-app (tela)**: todas as transições de status relevantes
- **E-mail**: apenas para **rejeição** — enviado ao supervisor responsável pelo relatório

**Backend — Alterações:**

1. **Novo módulo** `src/notifications/`:
   - `notifications.module.ts`
   - `notifications.service.ts` — lógica de criação e disparo
   - `notifications.controller.ts` — endpoints REST
   - `email.service.ts` — integração com AWS SES para envio de e-mail

2. **Migration** — Tabela `notifications`:
   ```sql
   CREATE TABLE notifications (
     id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
     recipientId UUID NOT NULL REFERENCES supervisor_profiles(id),
     type TEXT NOT NULL,
     title TEXT NOT NULL,
     message TEXT NOT NULL,
     metadata JSONB,      -- { reportId, reportType, serialNumber, ... }
     readAt TIMESTAMPTZ,
     createdAt TIMESTAMPTZ DEFAULT NOW()
   );
   ```

3. **Eventos que geram notificação in-app:**

   | Evento | Destinatários | Tipo |
   |--------|--------------|------|
   | Supervisor assina relatório | Controladores do grupo | `report_pending_approval` |
   | Controlador aprova | Supervisor que assinou | `report_approved` |
   | Controlador rejeita | Supervisor que assinou | `report_rejected` |
   | Controlador assume relatório | — (sem notificação) | — |
   | Gestor reatribui relatório | Novo analista atribuído | `report_reassigned` |

4. **Evento que gera e-mail (além da notificação in-app):**

   | Evento | Destinatário | Conteúdo |
   |--------|-------------|----------|
   | Controlador **rejeita** | Supervisor responsável | Serial do relatório, planta, motivo da rejeição, link para o relatório |

5. **E-mail via AWS SES:**
   - Credenciais via AWS Secrets Manager (padrão existente)
   - Template HTML simples para rejeição (serial, planta, motivo, link)
   - Fallback: se envio falhar, notificação in-app é garantida (e-mail é best-effort)
   - Campo `email` já existe no `SupervisorProfile`

6. **Endpoints**:
   - `GET /notifications` — lista (paginada, filtro read/unread)
   - `PATCH /notifications/:id/read` — marcar como lida
   - `PATCH /notifications/read-all` — marcar todas como lidas
   - `GET /notifications/unread-count` — contagem para badge

**Frontend — Alterações:**

7. **Header** (`src/components/layout/Header.tsx`, linha 44-46):
   - Badge de contagem no ícone de sino (já existe o ícone, não funcional)
   - Dropdown com lista de notificações recentes
   - Link "Ver todas" → `/notifications`

8. **Nova página** `NotificationsList.tsx`:
   - Lista completa com filtro read/unread
   - Click na notificação → navega para o relatório

9. **React Query**: Hook `useNotifications()`, `useUnreadCount()` com polling 30s

**Critérios de aceite:**
- [ ] Notificação in-app ao assinar relatório (→ controladores do grupo)
- [ ] Notificação in-app ao aprovar (→ supervisor)
- [ ] Notificação in-app + **e-mail** ao rejeitar (→ supervisor), com motivo
- [ ] Badge no sino com contagem de não lidas
- [ ] Marcar como lida individual e em lote
- [ ] E-mail contém: serial, planta, motivo, link para o relatório
- [ ] Falha no envio de e-mail não bloqueia o fluxo (best-effort)

---

### TASK-09 — Anexos em Relatórios de Venda e Transferência (Est: 5 SP)

**Contexto de negócio:**
Relatórios de embarque (venda e transferência) precisam ser acompanhados de documentos
comprobatórios como CSI (Certificado Sanitário Internacional), CSN (Certificado Sanitário
Nacional) e outros documentos regulatórios. Hoje esses documentos ficam fora do sistema.

**Escopo desta entrega:** Upload, armazenamento e download de documentos.
OCR/extração de dados dos documentos fica como feature futura (Tesseract/pdf-parse, sem custo de IA).

**Contexto técnico:**
- AWS SDK já configurado (`src/config/aws-config.service.ts`) — SecretsManager/SSM
- Necessário adicionar `@aws-sdk/client-s3` e configurar Multer
- Collaborator photo upload existe como referência de padrão

**Backend — Alterações:**

1. **Migration** — Tabela `report_attachments`:
   ```sql
   CREATE TABLE report_attachments (
     id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
     shippingReportId UUID REFERENCES shipping_reports(id),
     fileName TEXT NOT NULL,
     fileKey TEXT NOT NULL,     -- S3 key
     fileSize INT NOT NULL,
     mimeType TEXT NOT NULL,
     category TEXT,             -- 'CSI', 'CSN', 'INVOICE', 'BL', 'NOTA_FISCAL', 'OUTRO'
     uploadedById UUID REFERENCES supervisor_profiles(id),
     createdAt TIMESTAMPTZ DEFAULT NOW()
   );
   ```

2. **Novo módulo** `src/attachments/`:
   - S3Client configurado via AwsConfigService
   - Upload via presigned URL (S3)
   - Download via presigned URL
   - Delete attachment
   - Validação: tipos permitidos (PDF, JPG, PNG), tamanho máximo (10MB)

3. **Endpoints**:
   - `POST /shipping-reports/:id/attachments` — upload (multipart)
   - `GET /shipping-reports/:id/attachments` — listar
   - `GET /attachments/:id/download` — presigned URL
   - `DELETE /attachments/:id` — remover (apenas rascunho)

4. **Regra**: Anexos só editáveis em status `rascunho`; visíveis em todos os status

**Frontend — Alterações:**

5. **Componente** `FileUpload.tsx`:
   - Drag & drop + click to upload
   - Seleção de categoria do documento (CSI, CSN, Invoice, etc.)
   - Preview de arquivos anexados
   - Botão remover (apenas rascunho)
   - Indicador de progresso

6. **ShippingReportForm.tsx**:
   - Adicionar seção "Documentos Anexos" para tipos venda_* e transferencia_*
   - Não exibir para tipos de abate

**Critérios de aceite:**
- [ ] Upload funcional com drag & drop
- [ ] Arquivo salvo no S3 e vinculado ao relatório
- [ ] Download via link
- [ ] Categoria do documento selecionável
- [ ] Tipos de arquivo validados (PDF, JPG, PNG)
- [ ] Tamanho máximo 10MB
- [ ] Apenas em relatórios de venda e transferência (não abate)
- [ ] Infraestrutura reutilizável para TASK-11 (certificado Halal no inventário)

---

## Sprint 5 — Novos Tipos + Integrações (Média)

### TASK-07 — Relatório de Desossa — Bovinos (Est: 5 SP)

**Contexto:**
- Desossa (deboning) é um novo tipo de relatório no módulo Bovinos
- Deve seguir o padrão dos demais relatórios (rascunho → assinado → aprovado)
- Provavelmente um novo `productionType` no enum existente

**Decisão arquitetural:**
- Desossa é mais próximo de **produção** (transformação de carcaça em cortes)
- Adicionar como novo `ProductionType`: `desossa`
- Usar o padrão de `customFields` JSON (como os demais tipos especializados)

**Backend:**

1. **Migration**: `ALTER TYPE "ProductionType" ADD VALUE 'desossa';`
2. **FM Metadata**: Adicionar constantes para FM da desossa
3. **Custom fields**: Definir campos específicos (rendimento, cortes, temperaturas, etc.)
4. **Verification items**: Criar checklist específico de desossa
5. **PDF template**: Layout específico para desossa

**Frontend:**

6. **Componente** `DesossaFields.tsx` em `src/components/production-types/`
7. **FM metadata service**: Registrar tipo `desossa`
8. **ProductionReportForm**: Já suporta tipos dinâmicos via discriminator

**Critérios de aceite:**
- [ ] Tipo "Desossa" disponível ao criar relatório de produção
- [ ] Campos específicos de desossa renderizados
- [ ] PDF gerado corretamente
- [ ] Checklist de verificação específico

---

### TASK-10 — Integração Cadastro de Empresas SIH ↔ CERT (POSTERGADA)

**Status:** Postergada até validação completa do sistema de Gestão de Certificações.
Cadastro de empresas no SIH segue funcional de forma independente.

**Quando retomar:** Após sistema CERT estar completamente validado em produção.
Escopo original mantido para referência futura:
- Importar tipo/categoria de indústria do certificado para o cadastro de empresas do SIH
- Integração via API REST entre os dois sistemas

---

### TASK-11 — Certificado Halal em Inventário e Transferências (Est: 8 SP)

**Contexto de negócio:**
Quando um registro envolve inventário (recebimento de carne, lotes) ou transferência
entre unidades, precisa estar vinculado ao respectivo certificado Halal. Como a FAMBRAS
já opera o SysHalal com API disponível, certificados emitidos pela FAMBRAS devem ser
buscados automaticamente. Certificados de outras certificadoras: upload manual.

**Escopo — afeta dois módulos:**
- **Inventário**: MeatInventoryReceipt (recebimento de carne) e BatchInventory (lotes)
- **Transferências**: ShippingReport com shippingType = transferencia_*

**Backend — Alterações:**

1. **Novo módulo** `src/integration/syshalal-integration.service.ts`:
   - Credenciais via AWS Secrets Manager: `user-token` + `user-owner`
   - Base URL configurável por ambiente (staging vs produção)
   - Cache com TTL (evitar chamadas repetidas para mesmo certificado)

   **Endpoints SysHalal utilizados:**
   - `GET /certified?certificate-number=XXX` — buscar certificado por número
   - `GET /certified_status?certificate-number=XXX` — verificar vigência
   - `GET /certified_pdf?certificate-number=XXX` — **download do PDF** direto da API
   - `GET /slaughterhouse?company-sanitary-code=XXX` — buscar planta por SIF

   **Ref:** `c:\sih\Syshalal External API Integration Manual.pdf` (v2.0, May 2025)

2. **Endpoint SIH** `GET /halal-cert/lookup?certNumber=XXX`:
   - Busca no SysHalal → retorna dados do certificado + status
   - Se encontrou: retorna dados + PDF disponível para download automático
   - Se não encontrou (não é FAMBRAS): retorna vazio → usuário faz upload manual

3. **Endpoint SIH** `GET /halal-cert/pdf?certNumber=XXX`:
   - Proxy para `GET /certified_pdf` do SysHalal
   - Permite baixar/visualizar o PDF do certificado sem sair do SIH

3. **Migration** — Novos campos:

   No `MeatInventoryReceipt` (já tem `halalCertNumber`):
   ```prisma
   halalCertData          Json?      // dados vindos do SysHalal (snapshot)
   halalCertAttachmentId  String?    // FK para upload manual (fallback)
   halalCertExpiryDate    DateTime?
   halalCertSource        String?    // "syshalal" ou "manual"
   ```

   No `BatchInventory`:
   ```prisma
   halalCertNumber        String?
   halalCertData          Json?
   halalCertAttachmentId  String?
   halalCertExpiryDate    DateTime?
   halalCertSource        String?
   ```

   No `ShippingReport` (para subtipos transferencia_*):
   - Já tem `halalSerialNumber` — complementar com:
   ```prisma
   halalCertData          Json?
   halalCertAttachmentId  String?
   halalCertSource        String?
   ```

4. **Fluxo de preenchimento:**
   ```
   Usuário digita número do certificado ou SIF do fornecedor
     → Sistema consulta SysHalal via API
       → Encontrou:
           - Preenche dados automaticamente (certSource = "syshalal")
           - PDF disponível via GET /certified_pdf (sem upload manual)
           - Status de vigência verificado automaticamente
       → Não encontrou (certificado de outra certificadora):
           - Habilita upload manual do documento (certSource = "manual")
           - Campos de número e validade preenchidos manualmente
   ```

5. **Reutilizar** módulo de attachments da TASK-09 para uploads manuais (fallback)

**Frontend — Alterações:**

6. **Componente reutilizável** `HalalCertField.tsx`:
   - Campo de número do certificado com botão "Buscar"
   - Se encontrado no SysHalal: exibe dados (número, validade, escopo, status)
     como read-only, com badge "Certificado FAMBRAS verificado"
   - Se não encontrado: campo de upload manual + campos de número e validade

7. **Integrar `HalalCertField` nos formulários:**
   - `MeatReceiptForm.tsx` (inventário de carne)
   - `BatchInventoryForm.tsx` (lotes de produção)
   - `ShippingReportForm.tsx` (subtipos transferencia_*)

8. **Indicador visual**: Certificado verificado (SysHalal) vs manual (upload)

**Critérios de aceite:**
- [ ] Busca automática de certificado via API SysHalal por número ou SIF
- [ ] Dados do certificado FAMBRAS preenchidos automaticamente
- [ ] PDF do certificado FAMBRAS disponível direto da API (sem upload)
- [ ] Status de vigência verificado automaticamente
- [ ] Upload manual funcional para certificados não-FAMBRAS
- [ ] Certificado vinculado a registros de inventário (carne e lotes)
- [ ] Certificado vinculado a relatórios de transferência
- [ ] Indicação visual clara de fonte (SysHalal vs manual)
- [ ] Falha na API SysHalal não bloqueia o fluxo (fallback para manual)

---

## Backlog Futuro

### TASK-12 (Futura) — OCR e Extração de Dados de Documentos Anexados

**Depende de:** TASK-09 (módulo de anexos em produção)

**Escopo planejado:**
- Extrair dados de documentos anexados (CSI, CSN, invoices) via OCR
- Abordagem: `pdf-parse` para PDFs digitais + `tesseract.js` para documentos escaneados/fotos
- Sem custo de IA — processamento local no servidor
- Campos a extrair: número do documento, data, origem, destino, produtos, pesos
- Avaliar também consulta via API do MAPA (Ministério da Agricultura)

**A definir quando priorizado:**
- Tipos de documento prioritários para extração
- Campos específicos por tipo de documento
- Disponibilidade da API do MAPA
- UX de revisão dos dados extraídos

---

## Resumo de Estimativas

| Sprint | Tasks | Story Points | Prioridade |
|--------|-------|-------------|------------|
| **Sprint 1** | TASK-02 + TASK-03 | 13 SP | Alta |
| **Sprint 2** | TASK-01 + TASK-04 | 16 SP | Alta |
| **Sprint 3** | TASK-05 + TASK-08 | 5 SP | Alta + Média |
| **Sprint 4** | TASK-06 + TASK-09 | 13 SP | Alta + Média |
| **Sprint 5** | TASK-07 + TASK-11 | 13 SP | Média |
| **TOTAL** | 9 Tasks (ativas) | **60 SP** | — |
| **Postergada** | TASK-10 (Integração CERT) | 5 SP | Aguarda validação CERT |
| **Backlog** | TASK-12 (OCR) | A estimar | Futura |

---

## Riscos e Pontos de Atenção

| # | Risco | Mitigação |
|---|-------|-----------|
| 1 | Migration de enum em produção pode causar downtime | Usar `ALTER TYPE ADD VALUE` (non-blocking no PostgreSQL) |
| 2 | Segmentação por grupo pode precisar de refinamento conforme estrutura real das empresas | Levantar mapeamento completo de plantas↔grupos com cliente |
| 3 | Envio de e-mail depende de AWS SES configurado | Validar domínio/remetente SES antes da Sprint 4 |
| 4 | Bug de transferências pode ser mais profundo que campos faltantes | Sprint 3 começa com investigação antes de fix |
| 5 | Integração CERT depende de API externa existente | Validar endpoints disponíveis antes de iniciar |
| 6 | Desossa — campos específicos não definidos pelo cliente | Levantar com cliente antes da Sprint 5 |
| 7 | OCR futuro (TASK-12) — definir escopo antes de priorizar | Levantar tipos de documento mais comuns e campos a extrair |

---

## Decisões Pendentes com Cliente

1. ~~**Canais de notificação**~~: DEFINIDO — in-app (todas transições) + e-mail (apenas rejeição)
2. **Campos do relatório de Desossa**: Aguardando FM FAMBRAS de referência do cliente
3. **API do MAPA**: Backlog futuro (TASK-12)
4. ~~**SysHalal API**~~: DEFINIDO — API operacional, endpoints mapeados
5. **Grupo IN/IND**: Quais plantas pertencem a cada grupo? (seed data)
6. ~~**Supervisor remove assinatura**~~: DEFINIDO — edita o mesmo relatório e re-assina
7. ~~**Relatórios existentes**~~: DEFINIDO — ficam como `assinado`, não migram. Fluxo novo se aplica a relatórios futuros
8. ~~**TASK-10 Integração CERT**~~: POSTERGADA — aguardar validação completa do sistema de Gestão de Certificações. Cadastro no SIH segue funcional independente

---

*Documento gerado em 2026-04-13 com base na reunião com cliente.*
*Próxima revisão: início da Sprint 1.*
