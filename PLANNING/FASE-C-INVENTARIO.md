---
title: Fase C — Inventario (Planejamento Detalhado)
nav_order: 5
---

# Fase C — Inventario

**Projeto**: SIH - Supervisao Industrial Halal
**Status**: PLANEJAMENTO DETALHADO
**Pre-requisito**: Fase A.2 completa (14 FMs especializados implementados)
**Ultima Atualizacao**: 2026-04-11
**Epic**: 07 — Inventario (FM 7.1.5.1, FM 7.1.5.6, FM 7.1.3.6)
**Baseado em**: 6 planilhas Excel reais (`C:\SIH\`), 25 abas, ~30.000 linhas

---

## 1. Resumo Executivo

O modulo de inventario digitaliza o controle de **conta corrente** (running balance) que conecta toda a cadeia produtiva Halal — do abate ao embarque. Sao 3 tipos distintos de inventario, cada um rastreando um elo diferente:

| # | FM | Tipo | Conceito | Volume real |
|---|-----|------|----------|-------------|
| 1 | 7.1.5.1 | **Carne Halal** | Recebido vs. utilizado em producao | ~1.700 linhas (MSP) |
| 2 | 7.1.5.6 | **Lotes de Producao** | Gerado vs. transferido entre unidades | ~11.000 linhas (GELNEX, 11 meses) |
| 3 | 7.1.3.6 | **Rotulagem** | Sem rotulo → rotulado → embarcado/descartado | ~17.000 linhas (PAMPEANO + ODERICH) |

### Cadeia de Rastreabilidade

```
Relatorio de Abate (FM 7.1.4.x)
    ↓ Certificado Halal + CSN + SIF
Inventario de Carne (FM 7.1.5.1)  ←  ENTRADA: carne recebida
    ↓ Codigo produto + data producao
Relatorio de Producao (FM 7.1.3.x)
    ↓ Lote gerado + peso
Inventario de Lotes (FM 7.1.5.6)  ←  ENTRADA: lote produzido
    ↓ Lote transferido + doc sanitario
Inventario de Rotulagem (FM 7.1.3.6)  ←  FLUXO: sem rotulo → rotulado → embarcado
    ↓ Numero do pedido
Relatorio de Embarque (FM 7.1.7.x)
```

### Diferenca vs. Relatorios

| Aspecto | Relatorios (FM 7.1.x) | Inventario (FM 7.1.5.x) |
|---------|----------------------|-------------------------|
| Granularidade | 1 evento = 1 relatorio | N movimentacoes = 1 planilha continua |
| Assinatura | Cada relatorio e assinado | **Nao tem assinatura** — e registro operacional |
| Volume | ~10-50 por mes/planta | **1.000+ linhas/mes** (lotes), **8.000+ por categoria** (rotulagem) |
| Workflow | rascunho → assinado | **Nao tem status** — lancamento direto |
| Saldo | Nao se aplica | **Conta corrente** (saldo = entrada - saida) |

---

## 2. Analise dos Dados Reais (Planilhas Excel)

### 2.1 FM 7.1.5.1 — Conta Corrente de Carne (MSP)

**Arquivo**: `F.M.7.1.5.1- INVENTARIO MSP.xlsx`
**Abas**: 2 (Bovinos: 1.530 linhas, Aves: 172 linhas)

**Estrutura da planilha (colunas)**:

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| Data recebimento | date | Data em que a carne chegou na planta |
| Especie | enum | bovino / ave |
| Frigorifico fornecedor | text | Nome do frigorifico de origem |
| SIF fornecedor | text | Numero SIF do fornecedor |
| Certificado Halal | text | Numero do certificado Halal |
| Periodo abate (inicio) | date | Inicio do periodo de abate |
| Periodo abate (fim) | date | Fim do periodo de abate |
| CSN | text | Certificado Sanitario Nacional |
| Peso total recebido (kg) | decimal | Peso total do recebimento |

**Sub-tabela por tipo de corte**:

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| Tipo de corte | text | C. DURO, C. MOLE, LAGARTO, ACEN, PATINHO, etc. |
| Peso recebido (kg) | decimal | Peso recebido deste corte |
| Peso utilizado (kg) | decimal | Peso utilizado em producao |
| Saldo (kg) | decimal | Calculado: recebido - utilizado |

**Observacoes**:
- Cada recebimento pode ter 5-15 tipos de corte
- Saldos sao acumulativos (corte pode ter multiplos recebimentos)
- Separacao total por especie (bovinos e aves em abas diferentes)
- O saldo e **por tipo de corte**, nao por recebimento

### 2.2 FM 7.1.5.6 — Inventario de Lotes (GELNEX)

**Arquivo**: `FM 7.1.5.6 - INVENTÁRIO GELNEX-ROUSSELOT-GENU-IN.xlsx`
**Abas**: 11 (uma por mes, mar/2025 a jan/2026)
**Volume**: ~1.000 linhas/mes

**Estrutura da planilha (colunas)**:

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| Data producao | date | Data em que o lote foi produzido |
| Doc sanitario | text | Numero do documento sanitario |
| Codigo lote | text | Codigo unico do lote |
| Peso liquido gerado (kg) | decimal | Peso do lote produzido |
| Estoque atual (kg) | decimal | Saldo: gerado - transferido |

**Sub-tabela de transferencias**:

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| Data transferencia | date | Data da saida |
| Doc sanitario transferencia | text | Documento da transferencia |
| Codigo lote transferencia | text | Codigo do lote na saida |
| Peso transferido (kg) | decimal | Peso da transferencia |
| Unidade destino | text | Nome da unidade industrial destino |

**Observacoes**:
- Alto volume — maior tabela do sistema
- Lotes podem ter multiplas transferencias parciais
- Saldo por lote individual (nao agregado)
- Particionamento mensal recomendado para queries performaticas

### 2.3 FM 7.1.3.6 — Inventario de Rotulagem

**Arquivos**: `FM 7.1.3.6 - INVENTÁRIO DE ACOMPANHAMENTO DE ROTULAGEM ODERICH.xlsx` (3 abas), `PLANILHA INVENTARIO PAMPEANO.xlsx` (11 abas)
**Volume**: ~2.000 (ODERICH) + ~15.000 (PAMPEANO) linhas

**Estrutura da planilha (colunas)**:

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| Data | date | Data do registro |
| Categoria produto | text | E 340, POUCH, EXTRATO, HAMBURGUER, etc. |
| Codigo interno (sem rotulo) | text | Codigo do produto ainda sem rotulo |
| Codigo produto rotulado | text | Codigo apos rotulagem |
| Nome produto | text | Descricao do produto |

**Entradas**:

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| Qtd recebida sem rotulo | integer | Quantidade recebida da producao |
| Qtd rotulada | integer | Quantidade que recebeu rotulo |

**Saidas**:

| Coluna | Tipo | Descricao |
|--------|------|-----------|
| Qtd embarcada | integer | Quantidade enviada (embarque) |
| Numero pedido | text | Numero do pedido de embarque |
| Qtd descartada (com rotulo) | integer | Descarte de produto rotulado |
| Qtd descartada (sem rotulo) | integer | Descarte de produto sem rotulo |

**Saldos calculados**:
- `estoqueComRotulo = acumulado(rotulados) - acumulado(embarcados + descarteRotulado)`
- `estoqueSemRotulo = acumulado(recebidos) - acumulado(rotulados + descarteSemRotulo)`

**Observacoes**:
- Organizado por **categoria de produto** (cada aba = 1 categoria)
- PAMPEANO tem 11 categorias, uma com 8.068 linhas (E 340)
- Saldos sao acumulativos dia-a-dia (running total)
- Referencia cruzada com embarque via numero de pedido

---

## 3. Modelo de Dados

### 3.1 Diagrama Entidade-Relacionamento

```
Plant (1) ──────── (N) MeatInventoryReceipt
                            │
                            ├── (N) MeatInventoryCut
                            │          │
                            │          └── (N) MeatInventoryUsage
                            │
                            └── refs: ProductionReport (opcional)

Plant (1) ──────── (N) BatchInventory
                            │
                            └── (N) BatchTransfer
                                    │
                                    └── refs: ShippingReport (opcional)

Plant (1) ──────── (N) LabelingInventory
                            │
                            └── refs: ShippingReport (opcional, via orderNumber)
```

### 3.2 Schema Prisma (6 tabelas novas)

```prisma
// ============================================
// INVENTARIO DE CARNE (FM 7.1.5.1)
// ============================================

model MeatInventoryReceipt {
  id                  String   @id @default(uuid()) @db.Uuid
  plantId             String   @db.Uuid
  plant               Plant    @relation(fields: [plantId], references: [id])
  receiptDate         DateTime @db.Date
  species             Species  // bovino | ave
  supplierName        String   // Frigorifico fornecedor
  supplierSif         String   // SIF do fornecedor
  halalCertNumber     String?  // Numero certificado Halal
  slaughterDateStart  DateTime? @db.Date
  slaughterDateEnd    DateTime? @db.Date
  csnNumber           String?  // Certificado Sanitario Nacional
  totalWeightKg       Decimal  @db.Decimal(12,3)
  notes               String?

  cuts                MeatInventoryCut[]

  createdById         String   @db.Uuid
  createdBy           SupervisorProfile @relation(fields: [createdById], references: [id])
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
}

model MeatInventoryCut {
  id                  String   @id @default(uuid()) @db.Uuid
  receiptId           String   @db.Uuid
  receipt             MeatInventoryReceipt @relation(fields: [receiptId], references: [id], onDelete: Cascade)
  cutType             String   // C. DURO, C. MOLE, LAGARTO, etc.
  receivedWeightKg    Decimal  @db.Decimal(12,3)

  usages              MeatInventoryUsage[]
}

model MeatInventoryUsage {
  id                  String   @id @default(uuid()) @db.Uuid
  cutId               String   @db.Uuid
  cut                 MeatInventoryCut @relation(fields: [cutId], references: [id], onDelete: Cascade)
  usageDate           DateTime @db.Date
  productCodes        String[] // Codigos dos produtos fabricados
  usedWeightKg        Decimal  @db.Decimal(12,3)
  productionReportId  String?  @db.Uuid // Referencia opcional ao relatorio de producao

  createdById         String   @db.Uuid
  createdBy           SupervisorProfile @relation(fields: [createdById], references: [id])
  createdAt           DateTime @default(now())
}

// ============================================
// INVENTARIO DE LOTES (FM 7.1.5.6)
// ============================================

model BatchInventory {
  id                  String   @id @default(uuid()) @db.Uuid
  plantId             String   @db.Uuid
  plant               Plant    @relation(fields: [plantId], references: [id])
  productionDate      DateTime @db.Date
  sanitaryDocNumber   String?  // Documento sanitario
  batchCode           String   // Codigo do lote
  netWeightGeneratedKg Decimal @db.Decimal(12,3) // Peso gerado
  notes               String?

  transfers           BatchTransfer[]

  createdById         String   @db.Uuid
  createdBy           SupervisorProfile @relation(fields: [createdById], references: [id])
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
}

model BatchTransfer {
  id                  String   @id @default(uuid()) @db.Uuid
  batchId             String   @db.Uuid
  batch               BatchInventory @relation(fields: [batchId], references: [id], onDelete: Cascade)
  transferDate        DateTime @db.Date
  sanitaryDocNumber   String?  // Doc sanitario da transferencia
  transferBatchCode   String?  // Codigo do lote na transferencia
  transferredWeightKg Decimal  @db.Decimal(12,3)
  destinationUnit     String   // Unidade industrial destino
  shippingReportId    String?  @db.Uuid // Referencia opcional ao relatorio de embarque

  createdById         String   @db.Uuid
  createdBy           SupervisorProfile @relation(fields: [createdById], references: [id])
  createdAt           DateTime @default(now())
}

// ============================================
// INVENTARIO DE ROTULAGEM (FM 7.1.3.6)
// ============================================

model LabelingInventory {
  id                    String   @id @default(uuid()) @db.Uuid
  plantId               String   @db.Uuid
  plant                 Plant    @relation(fields: [plantId], references: [id])
  entryDate             DateTime @db.Date
  productCategory       String   // E 340, POUCH, EXTRATO, HAMBURGUER, etc.
  internalCode          String?  // Codigo interno (sem rotulo)
  labeledProductCode    String?  // Codigo do produto rotulado
  productName           String   // Nome do produto
  // Entradas
  receivedUnlabeledQty  Int      @default(0)
  labeledQty            Int      @default(0)
  // Saidas
  shippedQty            Int      @default(0)
  orderNumber           String?  // Numero do pedido (ref embarque)
  discardedLabeledQty   Int      @default(0)
  discardedUnlabeledQty Int      @default(0)
  notes                 String?

  createdById           String   @db.Uuid
  createdBy             SupervisorProfile @relation(fields: [createdById], references: [id])
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt
}
```

### 3.3 Views de Saldo (SQL)

Saldos calculados no banco via **views** (nao em codigo aplicacao):

```sql
-- View: Saldo por tipo de corte (FM 7.1.5.1)
CREATE VIEW meat_inventory_balance AS
SELECT
  c.id AS "cutId",
  r."plantId",
  r.species,
  c."cutType",
  c."receivedWeightKg",
  COALESCE(SUM(u."usedWeightKg"), 0) AS "totalUsedKg",
  c."receivedWeightKg" - COALESCE(SUM(u."usedWeightKg"), 0) AS "balanceKg"
FROM "MeatInventoryCut" c
JOIN "MeatInventoryReceipt" r ON r.id = c."receiptId"
LEFT JOIN "MeatInventoryUsage" u ON u."cutId" = c.id
GROUP BY c.id, r."plantId", r.species, c."cutType", c."receivedWeightKg";

-- View: Saldo por lote (FM 7.1.5.6)
CREATE VIEW batch_inventory_balance AS
SELECT
  b.id AS "batchId",
  b."plantId",
  b."batchCode",
  b."productionDate",
  b."netWeightGeneratedKg",
  COALESCE(SUM(t."transferredWeightKg"), 0) AS "totalTransferredKg",
  b."netWeightGeneratedKg" - COALESCE(SUM(t."transferredWeightKg"), 0) AS "balanceKg"
FROM "BatchInventory" b
LEFT JOIN "BatchTransfer" t ON t."batchId" = b.id
GROUP BY b.id;

-- View: Saldo de rotulagem por categoria (FM 7.1.3.6)
CREATE VIEW labeling_inventory_balance AS
SELECT
  l."plantId",
  l."productCategory",
  l."productName",
  SUM(l."receivedUnlabeledQty") AS "totalReceived",
  SUM(l."labeledQty") AS "totalLabeled",
  SUM(l."shippedQty") AS "totalShipped",
  SUM(l."discardedLabeledQty") AS "totalDiscardedLabeled",
  SUM(l."discardedUnlabeledQty") AS "totalDiscardedUnlabeled",
  SUM(l."receivedUnlabeledQty") - SUM(l."labeledQty") - SUM(l."discardedUnlabeledQty") AS "unlabeledStockQty",
  SUM(l."labeledQty") - SUM(l."shippedQty") - SUM(l."discardedLabeledQty") AS "labeledStockQty"
FROM "LabelingInventory" l
GROUP BY l."plantId", l."productCategory", l."productName";
```

### 3.4 Indices para Performance

```sql
-- FM 7.1.5.1: Busca por planta + especie + periodo
CREATE INDEX idx_meat_receipt_plant_species ON "MeatInventoryReceipt" ("plantId", species, "receiptDate");
CREATE INDEX idx_meat_usage_cut ON "MeatInventoryUsage" ("cutId", "usageDate");

-- FM 7.1.5.6: Busca por planta + periodo (ALTO VOLUME)
CREATE INDEX idx_batch_plant_date ON "BatchInventory" ("plantId", "productionDate");
CREATE INDEX idx_batch_transfer_batch ON "BatchTransfer" ("batchId", "transferDate");

-- FM 7.1.3.6: Busca por planta + categoria + periodo (ALTO VOLUME)
CREATE INDEX idx_labeling_plant_category ON "LabelingInventory" ("plantId", "productCategory", "entryDate");
```

---

## 4. Arquitetura Backend

### 4.1 Estrutura de Modulos (3 modulos NestJS)

```
src/inventory/
├── meat/
│   ├── meat-inventory.module.ts
│   ├── meat-inventory.controller.ts
│   ├── meat-inventory.service.ts
│   └── dto/
│       ├── create-meat-receipt.dto.ts
│       ├── create-meat-usage.dto.ts
│       └── query-meat-inventory.dto.ts
├── batch/
│   ├── batch-inventory.module.ts
│   ├── batch-inventory.controller.ts
│   ├── batch-inventory.service.ts
│   └── dto/
│       ├── create-batch-inventory.dto.ts
│       ├── create-batch-transfer.dto.ts
│       └── query-batch-inventory.dto.ts
├── labeling/
│   ├── labeling-inventory.module.ts
│   ├── labeling-inventory.controller.ts
│   ├── labeling-inventory.service.ts
│   └── dto/
│       ├── create-labeling-entry.dto.ts
│       └── query-labeling-inventory.dto.ts
└── inventory.module.ts  // Agrega os 3 submodulos
```

### 4.2 Endpoints

#### Inventario de Carne (FM 7.1.5.1)

| Metodo | Rota | Descricao | Roles |
|--------|------|-----------|-------|
| GET | `/inventory/meat` | Listar recebimentos (paginado, filtros) | admin, coordenador, supervisor |
| GET | `/inventory/meat/:id` | Detalhe do recebimento + cortes + usos | admin, coordenador, supervisor |
| GET | `/inventory/meat/balance` | Saldos por tipo de corte (view) | admin, coordenador, supervisor |
| POST | `/inventory/meat` | Criar recebimento + cortes | supervisor, operador |
| PATCH | `/inventory/meat/:id` | Editar recebimento | supervisor, operador |
| DELETE | `/inventory/meat/:id` | Excluir recebimento (se sem usos) | admin |
| POST | `/inventory/meat/:id/usage` | Registrar uso de corte em producao | supervisor, operador |
| PATCH | `/inventory/meat/usage/:usageId` | Editar uso | supervisor |
| DELETE | `/inventory/meat/usage/:usageId` | Excluir uso | admin |

#### Inventario de Lotes (FM 7.1.5.6)

| Metodo | Rota | Descricao | Roles |
|--------|------|-----------|-------|
| GET | `/inventory/batch` | Listar lotes (paginado, filtros por mes) | admin, coordenador, supervisor |
| GET | `/inventory/batch/:id` | Detalhe do lote + transferencias | admin, coordenador, supervisor |
| GET | `/inventory/batch/balance` | Saldos por lote (view) | admin, coordenador, supervisor |
| POST | `/inventory/batch` | Criar lote produzido | supervisor, operador |
| PATCH | `/inventory/batch/:id` | Editar lote | supervisor, operador |
| DELETE | `/inventory/batch/:id` | Excluir lote (se sem transferencias) | admin |
| POST | `/inventory/batch/:id/transfer` | Registrar transferencia | supervisor, operador |
| PATCH | `/inventory/batch/transfer/:transferId` | Editar transferencia | supervisor |
| DELETE | `/inventory/batch/transfer/:transferId` | Excluir transferencia | admin |

#### Inventario de Rotulagem (FM 7.1.3.6)

| Metodo | Rota | Descricao | Roles |
|--------|------|-----------|-------|
| GET | `/inventory/labeling` | Listar entradas (paginado, filtros por categoria) | admin, coordenador, supervisor |
| GET | `/inventory/labeling/balance` | Saldos por categoria (view) | admin, coordenador, supervisor |
| POST | `/inventory/labeling` | Criar entrada de movimentacao | supervisor, operador |
| POST | `/inventory/labeling/bulk` | Criar multiplas entradas (para lancamento rapido) | supervisor, operador |
| PATCH | `/inventory/labeling/:id` | Editar entrada | supervisor, operador |
| DELETE | `/inventory/labeling/:id` | Excluir entrada | admin |

#### Dashboard de Inventario

| Metodo | Rota | Descricao | Roles |
|--------|------|-----------|-------|
| GET | `/inventory/summary` | Resumo consolidado (3 tipos) | admin, coordenador |
| GET | `/inventory/alerts` | Alertas (saldo negativo, zerado, lotes antigos) | admin, coordenador |

### 4.3 Validacoes de Negocio

| Regra | Tipo | Descricao |
|-------|------|-----------|
| V1 | Carne | Soma dos pesos dos cortes deve ser = peso total do recebimento |
| V2 | Carne | Peso utilizado nao pode exceder saldo disponivel do corte |
| V3 | Lotes | Peso transferido nao pode exceder saldo do lote |
| V4 | Lotes | Codigo de lote deve ser unico por planta |
| V5 | Rotulagem | Qtd rotulada nao pode exceder estoque sem rotulo |
| V6 | Rotulagem | Qtd embarcada nao pode exceder estoque com rotulo |
| V7 | Todos | Exclusao so permitida se nao ha movimentacoes filhas (Carne: sem usos, Lotes: sem transferencias) |
| V8 | Todos | Filtro por planta obrigatorio (plantId) — usuario so ve inventario da sua planta |

---

## 5. Arquitetura Frontend

### 5.1 Estrutura de Paginas

```
src/pages/inventory/
├── meat/
│   ├── MeatInventoryList.tsx        // Lista de recebimentos + saldos
│   ├── MeatReceiptForm.tsx          // Formulario de recebimento + cortes
│   └── MeatUsageForm.tsx            // Formulario de uso em producao
├── batch/
│   ├── BatchInventoryList.tsx       // Lista de lotes + saldos
│   ├── BatchInventoryForm.tsx       // Formulario de lote
│   └── BatchTransferForm.tsx        // Formulario de transferencia
├── labeling/
│   ├── LabelingInventoryList.tsx    // Lista de movimentacoes + saldos
│   └── LabelingEntryForm.tsx        // Formulario de entrada (linha unica ou bulk)
├── InventoryDashboard.tsx           // Dashboard consolidado
└── index.ts
```

### 5.2 Rotas

```typescript
// Inventario de Carne
'/inventory/meat'                    // Lista
'/inventory/meat/new'                // Novo recebimento
'/inventory/meat/:id'                // Editar recebimento
'/inventory/meat/:id/usage'          // Registrar uso

// Inventario de Lotes
'/inventory/batch'                   // Lista
'/inventory/batch/new'               // Novo lote
'/inventory/batch/:id'               // Editar lote
'/inventory/batch/:id/transfer'      // Registrar transferencia

// Inventario de Rotulagem
'/inventory/labeling'                // Lista (filtro por categoria)
'/inventory/labeling/new'            // Nova entrada
'/inventory/labeling/:id'            // Editar entrada

// Dashboard
'/inventory/dashboard'               // Visao consolidada
```

### 5.3 Services (TanStack Query)

```
src/services/
├── meat-inventory.service.ts        // Hooks: useReceipts, useReceipt, useMeatBalance, useCreateReceipt, useCreateUsage
├── batch-inventory.service.ts       // Hooks: useBatches, useBatch, useBatchBalance, useCreateBatch, useCreateTransfer
├── labeling-inventory.service.ts    // Hooks: useLabelingEntries, useLabelingBalance, useCreateEntry, useBulkCreate
└── inventory-dashboard.service.ts   // Hooks: useInventorySummary, useInventoryAlerts
```

### 5.4 Componentes Compartilhados (novos)

| Componente | Descricao | Usado por |
|------------|-----------|-----------|
| `BalanceCard` | Card com saldo atual (entrada, saida, saldo), cor condicional (verde/amarelo/vermelho) | Todos os 3 inventarios |
| `BalanceTable` | Tabela de saldos por item (corte, lote, categoria) com destaque para saldo zero/negativo | Todos os 3 inventarios |
| `MovementTimeline` | Timeline de movimentacoes (entrada/saida) por item | Carne (usos por corte), Lotes (transferencias por lote) |
| `CategoryFilter` | Filtro de categoria de produto (tabs ou dropdown) | Rotulagem |
| `MonthSelector` | Seletor de mes para navegacao em alto volume | Lotes, Rotulagem |

### 5.5 Sidebar — Novo Grupo "Inventario"

```
Inventario (NOVO grupo)
  Carne Halal           → /inventory/meat
  Lotes de Producao     → /inventory/batch
  Rotulagem             → /inventory/labeling
  Dashboard Inventario  → /inventory/dashboard
```

Posicionado entre "Subprodutos" e "Gestao" na sidebar.

---

## 6. Importacao de Dados Excel

### 6.1 Necessidade

As plantas ja operam com planilhas Excel ha meses/anos. A migracao para o sistema digital requer **importacao dos dados historicos** para que os saldos iniciais sejam corretos.

### 6.2 Estrategia

| Aspecto | Decisao |
|---------|---------|
| Biblioteca | `xlsx` (SheetJS) no backend |
| Endpoint | `POST /inventory/{type}/import` (upload multipart) |
| Validacao | Validar colunas obrigatorias + tipos antes de inserir |
| Transacao | Import inteiro dentro de uma transacao Prisma |
| Preview | Endpoint `POST /inventory/{type}/import/preview` retorna primeiras N linhas parseadas para confirmacao |
| Roles | Somente `admin` pode importar |
| Limite | Max 5.000 linhas por import (chunked se necessario) |

### 6.3 Mapeamento de Colunas

O sistema deve suportar **mapeamento flexivel de colunas** porque cada planta pode nomear colunas de forma diferente. O fluxo:

1. Upload do arquivo .xlsx
2. Backend le os headers da planilha
3. Frontend exibe mapeamento sugerido (fuzzy match) + permite correcao manual
4. Confirmacao → import com mapeamento definido

### 6.4 Frontend — Tela de Import

```
src/pages/inventory/
├── import/
│   ├── InventoryImport.tsx          // Upload + selecao de tipo
│   ├── ColumnMapper.tsx             // Mapeamento de colunas
│   └── ImportPreview.tsx            // Preview + confirmacao
```

---

## 7. Plano de Implementacao

### Fase C.1 — Schema + Migration + Views (1 sprint)

- [ ] Adicionar 6 modelos ao schema Prisma (MeatInventoryReceipt, MeatInventoryCut, MeatInventoryUsage, BatchInventory, BatchTransfer, LabelingInventory)
- [ ] Adicionar relations em Plant e SupervisorProfile
- [ ] Criar migration `add_inventory_tables`
- [ ] Criar migration com SQL raw para as 3 views de saldo
- [ ] Criar migration com SQL raw para os indices de performance
- [ ] Testar migration: `prisma migrate deploy` + `prisma generate`

### Fase C.2 — Backend: Modulo Carne (1 sprint)

- [ ] Criar modulo `inventory/meat/` (module, controller, service)
- [ ] DTOs com Zod: CreateMeatReceipt (com array de cortes), CreateMeatUsage
- [ ] Query DTO com filtros: plantId, species, dateFrom, dateTo, page, limit
- [ ] CRUD de recebimentos (GET list, GET :id, POST, PATCH, DELETE)
- [ ] Endpoint de uso (POST /usage, PATCH /usage/:id, DELETE /usage/:id)
- [ ] Endpoint de saldo (GET /balance — query na view)
- [ ] Validacoes V1, V2, V7 (soma cortes, saldo uso, exclusao protegida)
- [ ] Testes unitarios do service

### Fase C.3 — Backend: Modulo Lotes (1 sprint)

- [ ] Criar modulo `inventory/batch/` (module, controller, service)
- [ ] DTOs com Zod: CreateBatchInventory, CreateBatchTransfer
- [ ] Query DTO com filtros: plantId, month, year, batchCode, page, limit
- [ ] CRUD de lotes (GET list, GET :id, POST, PATCH, DELETE)
- [ ] Endpoint de transferencia (POST /transfer, PATCH /transfer/:id, DELETE /transfer/:id)
- [ ] Endpoint de saldo (GET /balance — query na view)
- [ ] Validacoes V3, V4, V7 (saldo transferencia, lote unico, exclusao protegida)
- [ ] Testes unitarios do service

### Fase C.4 — Backend: Modulo Rotulagem (1 sprint)

- [ ] Criar modulo `inventory/labeling/` (module, controller, service)
- [ ] DTOs com Zod: CreateLabelingEntry, BulkCreateLabelingEntry
- [ ] Query DTO com filtros: plantId, productCategory, dateFrom, dateTo, page, limit
- [ ] CRUD de entradas (GET list, POST, POST /bulk, PATCH, DELETE)
- [ ] Endpoint de saldo (GET /balance — query na view, agrupado por categoria)
- [ ] Validacoes V5, V6 (estoque rotulagem)
- [ ] Testes unitarios do service

### Fase C.5 — Backend: Dashboard + Alertas (0.5 sprint)

- [ ] Endpoint `GET /inventory/summary` — totais consolidados dos 3 tipos por planta
- [ ] Endpoint `GET /inventory/alerts` — saldos negativos, zerados, lotes > 90 dias em estoque
- [ ] Registrar InventoryModule no AppModule

### Fase C.6 — Frontend: Paginas de Carne (1 sprint)

- [ ] Service `meat-inventory.service.ts` com hooks TanStack Query
- [ ] `MeatInventoryList.tsx` — tabela de recebimentos + card de saldos por corte
- [ ] `MeatReceiptForm.tsx` — formulario com tabela dinamica de cortes (add/remove linhas)
- [ ] `MeatUsageForm.tsx` — select de corte (filtrado por saldo > 0) + peso utilizado
- [ ] Componente `BalanceCard` (reutilizavel)
- [ ] Componente `BalanceTable` (reutilizavel)

### Fase C.7 — Frontend: Paginas de Lotes (1 sprint)

- [ ] Service `batch-inventory.service.ts` com hooks TanStack Query
- [ ] `BatchInventoryList.tsx` — tabela de lotes + saldo individual + MonthSelector
- [ ] `BatchInventoryForm.tsx` — formulario de lote
- [ ] `BatchTransferForm.tsx` — select de lote (filtrado por saldo > 0) + dados transferencia
- [ ] Componente `MovementTimeline` (reutilizavel)
- [ ] Componente `MonthSelector` (reutilizavel)

### Fase C.8 — Frontend: Paginas de Rotulagem (1 sprint)

- [ ] Service `labeling-inventory.service.ts` com hooks TanStack Query
- [ ] `LabelingInventoryList.tsx` — tabela com CategoryFilter (tabs) + saldos por categoria
- [ ] `LabelingEntryForm.tsx` — formulario de entrada individual ou bulk (multiplas linhas)
- [ ] Componente `CategoryFilter` (tabs por categoria de produto)

### Fase C.9 — Frontend: Dashboard Inventario + Sidebar (0.5 sprint)

- [ ] Service `inventory-dashboard.service.ts`
- [ ] `InventoryDashboard.tsx` — 3 secoes (carne, lotes, rotulagem) com BalanceCards + alertas
- [ ] Adicionar grupo "Inventario" na sidebar (4 itens)
- [ ] Adicionar rotas no App.tsx
- [ ] Roles: admin e coordenador veem dashboard, supervisor e operador veem inventarios

### Fase C.10 — Import Excel (1 sprint)

- [ ] Instalar dependencia `xlsx` no backend
- [ ] Endpoints `POST /inventory/{type}/import/preview` e `POST /inventory/{type}/import`
- [ ] Parser de Excel: ler headers, detectar formato, mapear colunas
- [ ] Validacao + insercao em transacao
- [ ] Frontend: `InventoryImport.tsx`, `ColumnMapper.tsx`, `ImportPreview.tsx`
- [ ] Acessivel somente para admin

### Fase C.11 — Seed + Build + Revisao (0.5 sprint)

- [ ] Seed: 5 recebimentos de carne (3 bovino, 2 ave) com cortes e usos
- [ ] Seed: 10 lotes com transferencias parciais
- [ ] Seed: 20 entradas de rotulagem em 3 categorias
- [ ] Build backend + frontend sem erros
- [ ] Revisao: comparar telas com planilhas Excel originais
- [ ] Teste manual: verificar saldos calculados vs. planilhas reais

---

## 8. Estimativa de Esforco

| Fase | Descricao | Complexidade | Sprints |
|------|-----------|-------------|---------|
| C.1 | Schema + Migration + Views | Baixa | 1 |
| C.2 | Backend Carne | Media | 1 |
| C.3 | Backend Lotes | Media | 1 |
| C.4 | Backend Rotulagem | Media | 1 |
| C.5 | Backend Dashboard + Alertas | Baixa | 0.5 |
| C.6 | Frontend Carne | Media | 1 |
| C.7 | Frontend Lotes | Media | 1 |
| C.8 | Frontend Rotulagem | Media | 1 |
| C.9 | Frontend Dashboard + Sidebar | Baixa | 0.5 |
| C.10 | Import Excel | **Alta** | 1 |
| C.11 | Seed + Build + Revisao | Baixa | 0.5 |
| | **Total** | | **~9.5 sprints** |

### Riscos e Mitigacoes

| # | Risco | Probabilidade | Impacto | Mitigacao |
|---|-------|:------------:|:-------:|-----------|
| 1 | **Performance com alto volume** (FM 7.1.5.6: 1000+ linhas/mes) | Media | Alto | Views + indices + paginacao obrigatoria + MonthSelector |
| 2 | **Mapeamento Excel variavel** entre plantas | Alta | Medio | ColumnMapper com fuzzy match + mapeamento manual |
| 3 | **Saldos inconsistentes** apos import parcial | Media | Alto | Import em transacao atomica + validacao pre-import |
| 4 | **Referencia cruzada com relatorios** que ainda nao existem | Baixa | Baixo | Campos opcionais (productionReportId, shippingReportId) |
| 5 | **Dados historicos com formatos inesperados** | Alta | Medio | Preview obrigatorio antes de import + log de erros por linha |

---

## 9. Decisoes de Design

| # | Decisao | Opcoes Consideradas | Escolha | Justificativa |
|---|---------|-------------------|---------|---------------|
| 1 | Saldos no banco ou no codigo? | Views SQL vs. calculo no service | **Views SQL** | Volume alto (30k+ linhas) — calculo no banco e mais performatico e consistente |
| 2 | Inventario com assinatura? | Com assinatura vs. sem | **Sem assinatura** | Inventario e registro operacional continuo, nao documento formal como relatorios |
| 3 | Inventario com status/workflow? | Com workflow vs. lancamento direto | **Lancamento direto** | Planilhas atuais nao tem conceito de rascunho/aprovacao |
| 4 | Cortes em tabela separada ou JSON? | Tabela relacional vs. Json | **Tabela relacional** | Cortes precisam de saldo individual — JSON nao permite queries eficientes |
| 5 | Import Excel: mapeamento auto ou manual? | Auto vs. manual vs. hibrido | **Hibrido** | Fuzzy match sugere, usuario confirma — cada planta tem nomes diferentes |
| 6 | Rotulagem: 1 tabela ou entrada+saida separadas? | 1 tabela unica vs. 2 tabelas | **1 tabela unica** | Planilha real e uma linha por dia com entradas e saidas na mesma linha |
| 7 | Particionamento para alto volume? | Particionamento vs. indices | **Indices (por enquanto)** | Particionamento e complexo de gerenciar com Prisma; indices + paginacao devem bastar para v1. Avaliar se performance degradar |

---

## 10. Dependencias e Pre-requisitos

| # | Dependencia | Status | Impacto |
|---|-------------|--------|---------|
| 1 | Fase A.2 completa (14 FMs especializados) | PENDENTE | Inventario referencia tipos de producao e embarque especializados |
| 2 | v1.0 em producao com dados reais | PENDENTE | Saldos iniciais dependem de dados de abate/producao/embarque existentes |
| 3 | Plant model com SIF | COMPLETO | Usado para filtro por planta |
| 4 | SupervisorProfile com roles | COMPLETO | Controle de acesso |
| 5 | Biblioteca `xlsx` | NAO INSTALADA | Necessario para import Excel |

---

## 11. Referencia: Planilhas Analisadas

| Planilha | FM | Tipo | Abas | Linhas | Localizacao |
|----------|-----|------|:----:|:------:|-------------|
| F.M.7.1.5.1- INVENTARIO MSP.xlsx | 7.1.5.1 | Carne | 2 | 1.702 | `C:\SIH\` |
| FM 7.1.5.6 - INVENTÁRIO GELNEX-ROUSSELOT-GENU-IN.xlsx | 7.1.5.6 | Lotes | 11 | ~11.000 | `C:\SIH\` |
| FM 7.1.3.6 - INVENTÁRIO DE ACOMPANHAMENTO DE ROTULAGEM ODERICH.xlsx | 7.1.3.6 | Rotulagem | 3 | ~2.000 | `C:\SIH\` |
| PLANILHA INVENTARIO PAMPEANO.xlsx | 7.1.3.6 | Rotulagem | 11 | ~15.000 | `C:\SIH\` |
| PLANILHA INVENTARIO CARAPRETA.xlsx | — | Produto | 2 | ~500 | `C:\SIH\` |
| inventario kin master.xlsx | — | Produto | 1 | ~200 | `C:\SIH\` |
