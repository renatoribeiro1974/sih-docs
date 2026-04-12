---
title: Fase C — Plano de Execucao Completo
nav_order: 6
---

# Fase C — Plano de Execucao Completo

**Projeto**: SIH - Supervisao Industrial Halal
**Status**: PRONTO PARA INICIAR
**Ultima Atualizacao**: 2026-04-12
**Pre-requisitos**: Fase A.2 COMPLETA, migrations validadas
**Referencia tecnica**: [FASE-C-INVENTARIO.md](FASE-C-INVENTARIO.md)

---

## PARTE 0 — Validacao de Migrations (Pre-requisito)

Antes de iniciar a Fase C, validar que todas as migrations recentes estao aplicadas
nos bancos de dados de desenvolvimento e producao.

### 0.1 Inventario de Migrations Desde 09/04/2026

| Projeto | DB (porta) | Migrations pendentes | Commits |
|---------|-----------|---------------------|---------|
| **sih-backend** | sih (5433) | `20260410120000_add_production_type_and_custom_fields` | `d73d930` |
| **halalsphere-backend** | halalsphere (5432) | `20260410200000_add_epic_i_fields` | `452ec49c` |
| | | `20260410210000_add_bloco6_epic_i_round2` | `ddc9d926` |
| | | `20260410220000_add_supplier_homologation` | `235db6c3` |
| **qrtracecode-cockpit-backend** | cockpit (5432) | Nenhuma desde 09/04 | — |
| **syshalal-api** | syshalal | Nenhuma desde 09/04 | — |

### 0.2 Checklist de Validacao — Ambiente LOCAL

Para cada projeto com migrations recentes, executar:

#### SIH Backend

```bash
cd /c/Projetos/sih-backend

# 1. Verificar que o DB esta rodando
docker compose up -d postgres
# ou: verificar se PostgreSQL local na porta 5433 responde
pg_isready -h localhost -p 5433

# 2. Verificar status das migrations
npx prisma migrate status

# 3. Se houver migrations pendentes, aplicar
npx prisma migrate deploy

# 4. Regenerar o client Prisma
npx prisma generate

# 5. Validar que o schema esta sincronizado
npx prisma validate

# 6. Rodar o build para garantir que nada quebrou
npm run build

# 7. Rodar o seed (opcional — revalida dados)
npx prisma db seed
```

**Resultado esperado**: `prisma migrate status` mostra 9 migrations aplicadas,
a mais recente sendo `20260410120000_add_production_type_and_custom_fields`.

#### HalalSphere Backend

```bash
cd /c/Projetos/halalsphere-backend

# 1. Verificar DB
pg_isready -h localhost -p 5432

# 2. Status
npx prisma migrate status

# 3. Aplicar pendentes
npx prisma migrate deploy

# 4. Regenerar client
npx prisma generate

# 5. Build
npm run build
```

**Resultado esperado**: 3 migrations de 10/04 aplicadas (epic_i_fields, bloco6_round2, supplier_homologation).

### 0.3 Estado dos docker-entrypoint.sh

| Projeto | Migrate deploy | Condicional? | Risco |
|---------|---------------|:------------:|-------|
| **sih-backend** | `npx prisma migrate deploy` | **SIM** — so roda se `AUTO_MIGRATE=true` | **ALTO** — se a env var nao estiver configurada na ECS Task Definition, migrations NAO sao aplicadas em producao |
| **halalsphere-backend** | `npx prisma migrate deploy` | NAO — roda sempre | Baixo — fail-safe com `\|\| echo WARNING` |

> **ACAO CRITICA**: Verificar se `AUTO_MIGRATE=true` esta configurado na ECS Task Definition
> do SIH Backend. Se nao estiver, as migrations desde 10/04 (ProductionType + customFields)
> **NAO foram aplicadas em producao**, e o backend pode estar rodando com schema desatualizado.

### 0.4 Checklist de Validacao — PRODUCAO (AWS ECS)

As migrations em producao sao aplicadas via `docker-entrypoint.sh` durante o deploy.
Para validar que o pipeline aplicou corretamente:

#### Verificacao via AWS Console

- [ ] **SIH Backend**: Verificar logs do ECS task mais recente (deploy de `2deb687`)
  - Procurar por: `prisma migrate deploy` no stdout do container
  - Confirmar: "1 migration applied" ou "All migrations have been applied"
  - Se nao houver log de migration: a migration pode nao estar sendo executada no entrypoint

- [ ] **HalalSphere Backend**: Verificar logs do ECS task mais recente
  - Procurar por: "3 migrations applied" (as 3 de 10/04)

#### Verificacao via Query Direta (se tiver acesso ao RDS)

```sql
-- Verificar tabela de controle do Prisma
SELECT * FROM "_prisma_migrations" ORDER BY "finished_at" DESC LIMIT 5;

-- Para SIH: verificar que a coluna productionType existe
SELECT column_name FROM information_schema.columns 
WHERE table_name = 'ProductionReport' AND column_name = 'productionType';

-- Para HalalSphere: verificar supplier_homologation
SELECT table_name FROM information_schema.tables 
WHERE table_name LIKE '%Supplier%' OR table_name LIKE '%supplier%';
```

### 0.5 Plano de Correcao (se migrations nao aplicadas)

| Cenario | Acao |
|---------|------|
| Migration pendente em LOCAL | `npx prisma migrate deploy` + `npx prisma generate` |
| Migration pendente em PRODUCAO | Verificar/corrigir docker-entrypoint.sh + redeploy via push para `release` |
| Schema drift (DB diverge do schema) | `npx prisma migrate diff` para diagnosticar, criar migration corretiva |
| Migration falhou parcialmente | Verificar `_prisma_migrations` para migration com `rolled_back_at` nao nulo |

---

## PARTE 1 — Fase C: Implementacao Completa

### Visao Geral das Fatias

```
PRE-REQ      Validacao de migrations (Parte 0)
             ↓
FATIA 1      Schema (6 tabelas, 3 views, indices) + Carne ponta-a-ponta + Sidebar
  C.1          Schema + Migration + Views
  C.2          Backend Carne + Frontend Carne + Sidebar + Rotas
             ↓
FATIA 2      Lotes ponta-a-ponta
  C.3          Backend Lotes + Frontend Lotes
             ↓
FATIA 3      Rotulagem ponta-a-ponta
  C.4          Backend Rotulagem + Frontend Rotulagem
             ↓
FATIA 4      Dashboard + Alertas
  C.5          Backend + Frontend consolidado
             ↓
FATIA 5      Import Excel + Seed + Build
  C.6          Import Excel (backend + frontend)
  C.7          Seed + Build + Revisao Final
```

---

### FATIA 1 — Fundacao + Carne Halal (2.5 sprints)

#### C.1 — Schema + Migration + Views

**Objetivo**: Criar toda a infraestrutura de banco para os 3 tipos de inventario.
Faz tudo junto porque a migration e atomica e as views referenciam as 3 tabelas.

**Arquivos a criar/editar**:

| Arquivo | Acao |
|---------|------|
| `prisma/schema.prisma` | Adicionar 6 modelos + relations em Plant e SupervisorProfile |
| `prisma/migrations/YYYYMMDDHHMMSS_add_inventory_tables/migration.sql` | DDL das 6 tabelas |
| `prisma/migrations/YYYYMMDDHHMMSS_add_inventory_views/migration.sql` | 3 views + 6 indices |

**Tarefas detalhadas**:

1. **Schema Prisma** (editar `schema.prisma`):
   - [ ] Modelo `MeatInventoryReceipt` — 12 campos + relation Plant + relation SupervisorProfile
   - [ ] Modelo `MeatInventoryCut` — 4 campos + relation cascade Receipt
   - [ ] Modelo `MeatInventoryUsage` — 6 campos + relation cascade Cut + relation SupervisorProfile
   - [ ] Modelo `BatchInventory` — 8 campos + relation Plant + relation SupervisorProfile
   - [ ] Modelo `BatchTransfer` — 8 campos + relation cascade BatchInventory + relation SupervisorProfile
   - [ ] Modelo `LabelingInventory` — 13 campos + relation Plant + relation SupervisorProfile
   - [ ] Relations inversas em `Plant` (meatReceipts, batches, labelingEntries)
   - [ ] Relations inversas em `SupervisorProfile` (meatReceipts, meatUsages, batches, batchTransfers, labelingEntries)

2. **Migration tabelas** (`add_inventory_tables`):
   - [ ] Criar migration com `prisma migrate dev --name add_inventory_tables`
   - [ ] Revisar SQL gerado (nomes de tabelas, tipos, constraints)
   - [ ] Se `migrate dev` nao funcionar (terminal nao-interativo): criar migration vazia + escrever SQL manualmente

3. **Migration views + indices** (`add_inventory_views`):
   - [ ] Criar migration vazia: `mkdir prisma/migrations/YYYYMMDDHHMMSS_add_inventory_views`
   - [ ] Escrever SQL raw com:
     - `CREATE VIEW meat_inventory_balance AS ...`
     - `CREATE VIEW batch_inventory_balance AS ...`
     - `CREATE VIEW labeling_inventory_balance AS ...`
     - 6 indices de performance (ver secao 3.4 do FASE-C-INVENTARIO.md)

4. **Validacao**:
   - [ ] `npx prisma migrate deploy` — aplica as 2 migrations
   - [ ] `npx prisma generate` — regenera o client
   - [ ] `npx prisma validate` — schema valido
   - [ ] `npm run build` — build sem erros
   - [ ] Query manual nas views (deve retornar 0 linhas mas sem erro)

**Criterio de aceite**: `prisma migrate status` mostra 11 migrations aplicadas (9 existentes + 2 novas).
Build passa. Views acessiveis via `SELECT * FROM meat_inventory_balance LIMIT 1`.

---

#### C.2 — Backend Carne + Frontend Carne + Sidebar

**Objetivo**: Primeiro modulo funcional completo — criar recebimento, registrar cortes,
registrar uso em producao, ver saldos.

##### C.2.1 — Backend Carne

**Arquivos a criar**:

```
src/inventory/
├── inventory.module.ts              // Modulo agregador (registra meat, batch, labeling)
├── meat/
│   ├── meat-inventory.module.ts
│   ├── meat-inventory.controller.ts
│   ├── meat-inventory.service.ts
│   └── dto/
│       ├── create-meat-receipt.dto.ts
│       ├── update-meat-receipt.dto.ts
│       ├── create-meat-usage.dto.ts
│       └── query-meat-inventory.dto.ts
```

**Tarefas detalhadas**:

1. **DTOs** (class-validator + class-transformer):
   - [ ] `CreateMeatReceiptDto`:
     - plantId (UUID, required)
     - receiptDate (Date, required)
     - species (enum bovino|ave, required)
     - supplierName (string, required)
     - supplierSif (string, required)
     - halalCertNumber (string, optional)
     - slaughterDateStart, slaughterDateEnd (Date, optional)
     - csnNumber (string, optional)
     - totalWeightKg (Decimal, required)
     - notes (string, optional)
     - cuts: array de { cutType: string, receivedWeightKg: Decimal } (min 1 item)
   - [ ] `CreateMeatUsageDto`:
     - cutId (UUID, required)
     - usageDate (Date, required)
     - usedWeightKg (Decimal, required, > 0)
     - productCodes (string[], optional)
     - productionReportId (UUID, optional)
   - [ ] `QueryMeatInventoryDto`:
     - plantId (UUID, required) — **filtro obrigatorio**
     - species (enum, optional)
     - dateFrom, dateTo (Date, optional)
     - page (int, default 1)
     - limit (int, default 20, max 100)

2. **Service** (`meat-inventory.service.ts`):
   - [ ] `findAll(query)` — lista recebimentos paginados + include cuts
   - [ ] `findOne(id)` — detalhe com cuts + usages
   - [ ] `getBalance(plantId, species?)` — query raw na view `meat_inventory_balance`
   - [ ] `create(dto, userId)` — transacao: criar receipt + cuts
     - **V1**: soma dos `receivedWeightKg` dos cortes deve == `totalWeightKg` (tolerancia 0.01)
   - [ ] `update(id, dto, userId)` — editar receipt + sync cortes
   - [ ] `remove(id)` — **V7**: rejeitar se algum corte tiver usages
   - [ ] `createUsage(dto, userId)` — registrar uso de corte
     - **V2**: `usedWeightKg` nao pode exceder saldo do corte (consultar view)
   - [ ] `updateUsage(usageId, dto)` — editar uso
   - [ ] `removeUsage(usageId)` — excluir uso

3. **Controller** (`meat-inventory.controller.ts`):
   - [ ] `GET /inventory/meat` — @Roles('admin', 'coordenador', 'supervisor')
   - [ ] `GET /inventory/meat/balance` — @Roles('admin', 'coordenador', 'supervisor')
   - [ ] `GET /inventory/meat/:id` — @Roles('admin', 'coordenador', 'supervisor')
   - [ ] `POST /inventory/meat` — @Roles('supervisor', 'operador')
   - [ ] `PATCH /inventory/meat/:id` — @Roles('supervisor', 'operador')
   - [ ] `DELETE /inventory/meat/:id` — @Roles('admin')
   - [ ] `POST /inventory/meat/:id/usage` — @Roles('supervisor', 'operador')
   - [ ] `PATCH /inventory/meat/usage/:usageId` — @Roles('supervisor')
   - [ ] `DELETE /inventory/meat/usage/:usageId` — @Roles('admin')

4. **Module** (`inventory.module.ts`):
   - [ ] Criar modulo agregador que importa MeatInventoryModule
   - [ ] Registrar no AppModule

5. **Validacao backend**:
   - [ ] Swagger mostra os 9 endpoints sob `/inventory/meat`
   - [ ] Testar via Swagger: criar recebimento → criar uso → consultar saldo
   - [ ] Build sem erros

##### C.2.2 — Frontend Carne

**Arquivos a criar**:

```
src/services/meat-inventory.service.ts
src/pages/inventory/meat/
├── MeatInventoryList.tsx
├── MeatReceiptForm.tsx
└── MeatUsageForm.tsx
src/components/inventory/
├── BalanceCard.tsx
└── BalanceTable.tsx
```

**Tarefas detalhadas**:

1. **Service** (`meat-inventory.service.ts`):
   - [ ] `useReceipts(query)` — GET /inventory/meat (paginado)
   - [ ] `useReceipt(id)` — GET /inventory/meat/:id
   - [ ] `useMeatBalance(plantId, species?)` — GET /inventory/meat/balance
   - [ ] `useCreateReceipt()` — POST mutation
   - [ ] `useUpdateReceipt()` — PATCH mutation
   - [ ] `useDeleteReceipt()` — DELETE mutation
   - [ ] `useCreateUsage()` — POST mutation
   - [ ] `useUpdateUsage()` — PATCH mutation
   - [ ] `useDeleteUsage()` — DELETE mutation

2. **Componentes reutilizaveis**:
   - [ ] `BalanceCard` — props: titulo, entrada (kg/un), saida (kg/un), saldo
     - Cor: verde (saldo > 20%), amarelo (0-20%), vermelho (negativo/zero)
     - Reutilizado por Lotes e Rotulagem
   - [ ] `BalanceTable` — props: colunas, dados, campoSaldo
     - Destaque linha vermelha se saldo <= 0
     - Reutilizado por Lotes e Rotulagem

3. **Paginas**:
   - [ ] `MeatInventoryList.tsx`:
     - Filtros: planta (obrigatorio), especie, periodo
     - Tabela paginada de recebimentos (data, fornecedor, SIF, cert Halal, peso total)
     - Secao "Saldos por Corte" com BalanceTable
     - Botao "+ Novo Recebimento" (supervisor, operador)
     - Botao "Registrar Uso" em cada corte com saldo > 0
   - [ ] `MeatReceiptForm.tsx`:
     - Campos do recebimento (data, especie, fornecedor, SIF, etc.)
     - Tabela dinamica de cortes (add/remove linhas): tipo de corte + peso
     - Validacao: soma dos cortes = peso total
     - Modo edicao: carrega dados existentes
   - [ ] `MeatUsageForm.tsx`:
     - Select de corte (filtrado por saldo > 0, mostra saldo disponivel)
     - Data do uso, peso utilizado, codigos de produto (chips/tags)
     - Validacao: peso <= saldo disponivel
     - Link opcional para relatorio de producao

##### C.2.3 — Sidebar + Rotas

**Arquivos a editar**:

| Arquivo | Acao |
|---------|------|
| `src/components/layout/Sidebar.tsx` | Adicionar grupo "Inventario" com 4 itens |
| `src/App.tsx` | Adicionar rotas `/inventory/*` |

**Tarefas**:
- [ ] Grupo "Inventario" na sidebar (entre Subprodutos e Nao-Conformidades)
  - Icone do grupo: `Package` ou `Warehouse` (Lucide)
  - Carne Halal → `/inventory/meat` (icone: `Beef`)
  - Lotes de Producao → `/inventory/batch` (icone: `Layers`)
  - Rotulagem → `/inventory/labeling` (icone: `Tag`)
  - Dashboard → `/inventory/dashboard` (icone: `BarChart3`)
- [ ] Visibilidade: todos os roles veem Carne/Lotes/Rotulagem; Dashboard so admin+coordenador
- [ ] Rotas em App.tsx para `/inventory/meat/*`
- [ ] Placeholder para rotas de Lotes, Rotulagem, Dashboard ("Em construcao")

**Criterio de aceite Fatia 1**:
- Sidebar mostra grupo Inventario
- Criar recebimento de carne com 5 cortes → registrar uso de 1 corte → saldo atualizado
- BalanceTable mostra saldos por corte com cores condicionais
- Build backend + frontend sem erros

---

### FATIA 2 — Lotes de Producao (1.5 sprints)

#### C.3 — Backend Lotes + Frontend Lotes

**Arquivos a criar (backend)**:

```
src/inventory/batch/
├── batch-inventory.module.ts
├── batch-inventory.controller.ts
├── batch-inventory.service.ts
└── dto/
    ├── create-batch-inventory.dto.ts
    ├── update-batch-inventory.dto.ts
    ├── create-batch-transfer.dto.ts
    └── query-batch-inventory.dto.ts
```

**Arquivos a criar (frontend)**:

```
src/services/batch-inventory.service.ts
src/pages/inventory/batch/
├── BatchInventoryList.tsx
├── BatchInventoryForm.tsx
└── BatchTransferForm.tsx
src/components/inventory/
├── MovementTimeline.tsx
└── MonthSelector.tsx
```

##### C.3.1 — Backend Lotes

**DTOs**:
- [ ] `CreateBatchInventoryDto`: plantId, productionDate, sanitaryDocNumber?, batchCode, netWeightGeneratedKg, notes?
- [ ] `CreateBatchTransferDto`: batchId, transferDate, sanitaryDocNumber?, transferBatchCode?, transferredWeightKg, destinationUnit, shippingReportId?
- [ ] `QueryBatchInventoryDto`: plantId (required), month?, year?, batchCode?, page, limit

**Service**:
- [ ] `findAll(query)` — paginado + include transferencias + filtro por mes/ano
- [ ] `findOne(id)` — detalhe com transferencias
- [ ] `getBalance(plantId, month?, year?)` — query view `batch_inventory_balance`
- [ ] `create(dto, userId)` — **V4**: batchCode unico por planta
- [ ] `update(id, dto, userId)`
- [ ] `remove(id)` — **V7**: rejeitar se tiver transferencias
- [ ] `createTransfer(dto, userId)` — **V3**: peso nao excede saldo do lote
- [ ] `updateTransfer(transferId, dto)`
- [ ] `removeTransfer(transferId)`

**Controller**: 9 endpoints sob `/inventory/batch` (mesmas roles que Carne)

##### C.3.2 — Frontend Lotes

**Componentes novos**:
- [ ] `MonthSelector` — seletor de mes (< Anterior | Mes/Ano | Proximo >) para alto volume
- [ ] `MovementTimeline` — timeline vertical: entrada (verde) → transferencias (azul) com data + peso + destino

**Paginas**:
- [ ] `BatchInventoryList.tsx`:
  - MonthSelector no topo (default: mes atual)
  - Filtros: planta, codigo lote
  - Tabela: data producao, doc sanitario, codigo lote, peso gerado, peso transferido, **saldo**
  - Expandir linha: MovementTimeline com transferencias do lote
  - Botao "+ Novo Lote" e "Transferir" por lote com saldo > 0
- [ ] `BatchInventoryForm.tsx`: formulario simples (5 campos)
- [ ] `BatchTransferForm.tsx`:
  - Select de lote (mostra saldo disponivel)
  - Data, doc sanitario, codigo lote transferencia, peso, unidade destino
  - Link opcional para relatorio de embarque

**Integracao**:
- [ ] Ativar rota `/inventory/batch/*` (substituir placeholder)

**Criterio de aceite Fatia 2**:
- Criar lote → transferir parcialmente → saldo atualiza
- MonthSelector navega entre meses
- MovementTimeline mostra historico de transferencias
- Lote com saldo 0 nao permite nova transferencia

---

### FATIA 3 — Rotulagem (1.5 sprints)

#### C.4 — Backend Rotulagem + Frontend Rotulagem

**Arquivos a criar (backend)**:

```
src/inventory/labeling/
├── labeling-inventory.module.ts
├── labeling-inventory.controller.ts
├── labeling-inventory.service.ts
└── dto/
    ├── create-labeling-entry.dto.ts
    ├── bulk-create-labeling.dto.ts
    └── query-labeling-inventory.dto.ts
```

**Arquivos a criar (frontend)**:

```
src/services/labeling-inventory.service.ts
src/pages/inventory/labeling/
├── LabelingInventoryList.tsx
└── LabelingEntryForm.tsx
src/components/inventory/
└── CategoryFilter.tsx
```

##### C.4.1 — Backend Rotulagem

**DTOs**:
- [ ] `CreateLabelingEntryDto`: plantId, entryDate, productCategory, internalCode?, labeledProductCode?, productName, receivedUnlabeledQty, labeledQty, shippedQty, orderNumber?, discardedLabeledQty, discardedUnlabeledQty, notes?
- [ ] `BulkCreateLabelingDto`: entries[] (array de CreateLabelingEntryDto, max 50 itens)
- [ ] `QueryLabelingInventoryDto`: plantId (required), productCategory?, dateFrom?, dateTo?, page, limit

**Service**:
- [ ] `findAll(query)` — paginado + filtro por categoria + periodo
- [ ] `getBalance(plantId, productCategory?)` — view `labeling_inventory_balance`
- [ ] `getCategories(plantId)` — lista de categorias distintas para a planta
- [ ] `create(dto, userId)` — validacoes V5, V6
- [ ] `bulkCreate(dto, userId)` — transacao atomica com N entries
- [ ] `update(id, dto, userId)`
- [ ] `remove(id)`

**Validacoes especificas**:
- **V5**: `labeledQty` <= estoque acumulado sem rotulo para aquela categoria (query na view)
- **V6**: `shippedQty` <= estoque acumulado com rotulo para aquela categoria (query na view)
- Obs: validacoes V5/V6 sao "soft" no inicio — alertar mas nao bloquear (dados historicos podem nao bater)

**Controller**: 6 endpoints sob `/inventory/labeling`

##### C.4.2 — Frontend Rotulagem

**Componentes novos**:
- [ ] `CategoryFilter` — tabs horizontais com as categorias do produto (E 340, POUCH, EXTRATO, etc.)
  - Carrega categorias do endpoint `getCategories`
  - Tab "Todas" como default

**Paginas**:
- [ ] `LabelingInventoryList.tsx`:
  - CategoryFilter no topo
  - Filtros: planta, periodo
  - Secao superior: BalanceCards por categoria selecionada (estoque sem rotulo, com rotulo)
  - Tabela: data, produto, recebido, rotulado, embarcado, descartado
  - Botao "+ Nova Entrada" e "+ Lancamento em Lote"
- [ ] `LabelingEntryForm.tsx`:
  - **Modo individual**: formulario com todos os campos
  - **Modo bulk**: tabela editavel com N linhas (similar a planilha)
    - Colunas: data, categoria, produto, recebido, rotulado, embarcado, descartado
    - Botao "+ Linha" para adicionar
    - Botao "Salvar Tudo" envia POST /bulk
  - Toggle entre modos (individual / lote)

**Integracao**:
- [ ] Ativar rota `/inventory/labeling/*` (substituir placeholder)

**Criterio de aceite Fatia 3**:
- Criar entrada individual → saldos atualizam
- Criar 5 entradas via bulk → todas salvas em transacao
- CategoryFilter filtra por tipo de produto
- Saldos por categoria mostram estoque sem rotulo e com rotulo

---

### FATIA 4 — Dashboard + Alertas (1 sprint)

#### C.5 — Backend + Frontend Dashboard

##### C.5.1 — Backend

**Arquivos a criar/editar**:

```
src/inventory/dashboard/
├── inventory-dashboard.controller.ts
└── inventory-dashboard.service.ts
```

**Endpoints**:
- [ ] `GET /inventory/summary?plantId=X` — totais consolidados:
  ```json
  {
    "meat": { "totalReceipts": 45, "totalWeightKg": 12500, "cutsWithZeroBalance": 3 },
    "batch": { "totalBatches": 230, "totalWeightKg": 85000, "batchesWithStock": 42 },
    "labeling": { "totalEntries": 1200, "unlabeledStock": 450, "labeledStock": 320 }
  }
  ```
- [ ] `GET /inventory/alerts?plantId=X` — lista de alertas:
  - Saldo negativo (qualquer tipo)
  - Saldo zerado em corte que tinha estoque
  - Lotes com estoque > 90 dias (potencial vencimento)
  - Estoque sem rotulo > 30 dias (risco de validade)

##### C.5.2 — Frontend

**Arquivos a criar**:

```
src/services/inventory-dashboard.service.ts
src/pages/inventory/InventoryDashboard.tsx
```

**Pagina**:
- [ ] `InventoryDashboard.tsx`:
  - Filtro de planta no topo
  - 3 secoes (Carne, Lotes, Rotulagem), cada uma com:
    - BalanceCard resumo (total entrada, total saida, saldo)
    - Mini-tabela com top 5 itens de maior movimentacao
  - Secao "Alertas" com lista de alertas (icone + descricao + link para o item)
  - Roles: admin e coordenador

**Integracao**:
- [ ] Ativar rota `/inventory/dashboard` (substituir placeholder)

**Criterio de aceite Fatia 4**:
- Dashboard mostra resumo dos 3 modulos
- Alertas destacam saldos negativos/zerados
- Somente admin e coordenador acessam

---

### FATIA 5 — Import Excel + Seed + Build (1.5 sprints)

#### C.6 — Import Excel

##### C.6.1 — Backend

**Dependencia**: `npm install xlsx` (SheetJS)

**Arquivos a criar**:

```
src/inventory/import/
├── inventory-import.controller.ts
├── inventory-import.service.ts
└── dto/
    ├── import-preview.dto.ts
    └── import-confirm.dto.ts
```

**Endpoints**:
- [ ] `POST /inventory/import/preview` — upload .xlsx + retorna:
  - Headers da planilha
  - Mapeamento sugerido (fuzzy match com colunas esperadas)
  - Primeiras 10 linhas parseadas
  - Contagem total de linhas
- [ ] `POST /inventory/import` — confirmar import com mapeamento:
  - Tipo de inventario (meat, batch, labeling)
  - Mapeamento de colunas (header planilha → campo do sistema)
  - Aba selecionada (se xlsx tiver multiplas)
  - Insercao em transacao atomica
  - Retorna: { imported: N, errors: [...] }

**Regras**:
- [ ] Max 5.000 linhas por import
- [ ] Somente role `admin`
- [ ] Transacao atomica: se qualquer linha falhar, rollback total
- [ ] Log de erros por linha (numero da linha + descricao do erro)
- [ ] Fuzzy match de colunas: normalizar (lowercase, sem acentos, sem espacos) e comparar

##### C.6.2 — Frontend

**Arquivos a criar**:

```
src/pages/inventory/import/
├── InventoryImport.tsx
├── ColumnMapper.tsx
└── ImportPreview.tsx
```

**Fluxo**:
1. `InventoryImport.tsx`: selecao de tipo + upload de arquivo .xlsx
2. `ColumnMapper.tsx`: exibe headers da planilha → campos do sistema, com sugestoes
3. `ImportPreview.tsx`: mostra primeiras 10 linhas mapeadas + contagem total + botao confirmar

**Sidebar**: adicionar item "Importar" no grupo Inventario (admin only)

---

#### C.7 — Seed + Build + Revisao

**Arquivo a editar**: `prisma/seed.ts`

**Seed data**:
- [ ] 5 recebimentos de carne (3 bovino com 5 cortes cada, 2 ave com 3 cortes cada)
- [ ] 8 usos de corte (vinculados aos recebimentos)
- [ ] 10 lotes de producao (datas variadas, pesos variados)
- [ ] 6 transferencias parciais (vinculadas a lotes, 2 lotes com multiplas transferencias)
- [ ] 20 entradas de rotulagem em 3 categorias (E 340, POUCH, HAMBURGUER)

**Validacao final**:
- [ ] `npm run build` backend sem erros
- [ ] `npm run build` frontend sem erros
- [ ] `npx prisma migrate deploy` aplica sem erros
- [ ] `npx prisma db seed` roda sem erros
- [ ] Swagger mostra ~35 novos endpoints sob `/inventory/*`
- [ ] Saldos calculados no seed batem com a soma manual
- [ ] Comparar telas com planilhas Excel originais (`C:\SIH\`)

---

## PARTE 2 — Ordem de Execucao Recomendada

```
Sessao 1:  PARTE 0 — Validar migrations (local + prod)
Sessao 2:  C.1 — Schema + Migration + Views
Sessao 3:  C.2.1 — Backend Carne (module, service, controller, DTOs)
Sessao 4:  C.2.2 — Frontend Carne (service, pages, componentes)
Sessao 5:  C.2.3 — Sidebar + Rotas + Teste ponta-a-ponta
           → DEPLOY FATIA 1 (push release)
Sessao 6:  C.3.1 — Backend Lotes
Sessao 7:  C.3.2 — Frontend Lotes + MonthSelector + MovementTimeline
           → DEPLOY FATIA 2 (push release)
Sessao 8:  C.4.1 — Backend Rotulagem
Sessao 9:  C.4.2 — Frontend Rotulagem + CategoryFilter + Bulk entry
           → DEPLOY FATIA 3 (push release)
Sessao 10: C.5 — Dashboard + Alertas (backend + frontend)
           → DEPLOY FATIA 4 (push release)
Sessao 11: C.6 — Import Excel (backend + frontend)
Sessao 12: C.7 — Seed + Build + Revisao final
           → DEPLOY FATIA 5 (push release)
```

Cada deploy e independente — o sistema funciona com qualquer combinacao de fatias completas.

---

## PARTE 3 — Checklist de Verificacao por Deploy

### Antes de cada push para release

- [ ] `npm run build` sem erros (backend)
- [ ] `npm run build` sem erros (frontend)
- [ ] `npx prisma validate` sem erros
- [ ] `npx prisma migrate status` — todas as migrations aplicadas localmente
- [ ] Testar fluxo principal no browser (criar → listar → editar → saldo)
- [ ] Verificar que endpoints existentes (relatorios, NCs) continuam funcionando

### Apos deploy (verificar no AWS)

- [ ] Logs ECS: migrations aplicadas com sucesso
- [ ] Health check: `GET /health` retorna OK
- [ ] Swagger: novos endpoints visiveis em `/api/docs`
- [ ] Smoke test: criar registro via Swagger em producao
