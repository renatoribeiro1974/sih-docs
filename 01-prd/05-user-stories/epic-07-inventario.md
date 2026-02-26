---
title: Epic 07 - Inventário
parent: User Stories
grand_parent: PRD
nav_order: 7
---

# Epic 07: Inventário (FASE FUTURA)

**FM 7.1.5.1 (Carne) + FM 7.1.5.6 (Lotes) + FM 7.1.3.6 (Rotulagem)**
**6 User Stories | Story Points TBD | Prioridade P2 (FUTURO)**

---

> **NOTA**: Este épico está **documentado mas NÃO será implementado na v1.0**. O modelo de dados foi desenhado e validado com dados reais (6 planilhas Excel, 25 abas, 23.000+ linhas) para evitar retrabalho quando for implementado. A implementação será via migration separada adicionando as tabelas ao banco existente.

---

## Contexto

O inventário Halal opera no conceito de **conta corrente** (running balance): saldo = entrada - saída. Existem 3 tipos distintos de inventário, cada um rastreando um elo diferente da cadeia produtiva.

### Cadeia de Rastreabilidade

```
Relatorio de Abate (FM 7.1.4.x)
    ↓ Certificado Halal + CSN + SIF
Inventario de Carne (FM 7.1.5.1)  ←  recebido vs. usado em producao
    ↓ Codigo produto + data producao
Relatorio de Producao (FM 7.1.3.x)
    ↓ Lote gerado + peso
Inventario de Lotes (FM 7.1.5.6)  ←  gerado vs. transferido
    ↓ Lote transferido + doc sanitario
Inventario de Rotulagem (FM 7.1.3.6)  ←  sem rotulo → rotulado → embarcado
    ↓ Numero do pedido
Relatorio de Embarque (FM 7.1.7.x)
```

### Modelos de Dados (6 tabelas)

**FM 7.1.5.1 - Conta Corrente de Carne:**
- `MeatInventoryReceipt` → `MeatInventoryCut` → `MeatInventoryUsage`
- Hierárquico: recebimento pai → cortes filhos → usos em produção

**FM 7.1.5.6 - Inventário de Lotes:**
- `BatchInventory` → `BatchTransfer`
- Lote gerado → transferências para outras unidades

**FM 7.1.3.6 - Inventário de Rotulagem:**
- `LabelingInventory`
- Fluxo: recebido sem rótulo → rotulado → embarcado/descartado

### Código Futuro

- Backend: `src/inventory/meat/`, `src/inventory/batch/`, `src/inventory/labeling/`
- Frontend: `src/pages/inventory/`

---

## User Stories

### Feature 7.1: Inventário de Carne Halal (FM 7.1.5.1)

#### SIH-034: Registrar Recebimento de Carne Halal

```
Como supervisor muculmano,
Eu quero registrar o recebimento de carne Halal na planta,
Para que o inventario de conta corrente seja atualizado com a entrada.
```

**Prioridade**: P2 - Nice to Have (Futuro)
**Estimativa**: TBD

**Critérios de Aceitação**:

- [ ] **Dados do recebimento**:
  - Data de recebimento
  - Especie (bovino, ave)
  - Nome do frigorífico fornecedor
  - SIF do fornecedor (obrigatório)
  - Número do certificado Halal
  - Período de abate (data início - data fim)
  - Número CSN (Certificado Sanitario Nacional)
  - Peso total recebido (kg)

- [ ] **Detalhamento por tipo de corte** (sub-linhas):
  - Tipo de corte: C. DURO, C. MOLE, LAGARTO, ACEN, PATINHO, etc.
  - Peso recebido por corte (kg)
  - Soma dos cortes = peso total recebido

- [ ] **Saldo automático**: Saldo inicial = peso total recebido

---

#### SIH-035: Registrar Uso de Carne em Produção

```
Como supervisor muculmano,
Eu quero registrar o uso de carne do inventario em producao,
Para que o saldo de conta corrente seja atualizado com a saida.
```

**Prioridade**: P2 - Nice to Have (Futuro)
**Estimativa**: TBD

**Critérios de Aceitação**:

- [ ] **Dados do uso**:
  - Data de produção
  - Tipo de corte utilizado (select do inventário disponível)
  - Códigos dos produtos fabricados
  - Peso utilizado (kg)
  - Link opcional ao relatório de produção (ProductionReport)

- [ ] **Válidações**:
  - Peso utilizado <= saldo disponível do corte
  - Saldo atualizado automáticamente: `currentBalanceKg = totalRecebido - somaUtilizado`

- [ ] **Visão de saldo**:
  - Tabela com: recebimento, cortes, peso recebido, peso utilizado, saldo atual
  - Destaque para itens com saldo zerado
  - Separação por especie (bovino, ave)

---

### Feature 7.2: Inventário de Lotes (FM 7.1.5.6)

#### SIH-036: Registrar Lote Produzido

```
Como supervisor muculmano,
Eu quero registrar um lote produzido no inventario,
Para que o rastreamento de lotes gerados seja mantido.
```

**Prioridade**: P2 - Nice to Have (Futuro)
**Estimativa**: TBD

**Critérios de Aceitação**:

- [ ] **Dados do lote**:
  - Data de produção
  - Número do documento sanitario
  - Código do lote gerado (obrigatório)
  - Peso liquido gerado (kg)

- [ ] **Saldo inicial**: `currentStockKg = weightGeneratedKg`

---

#### SIH-037: Registrar Transferência de Lote

```
Como supervisor muculmano,
Eu quero registrar a transferencia de um lote para outra unidade,
Para que o saldo do inventario de lotes seja atualizado.
```

**Prioridade**: P2 - Nice to Have (Futuro)
**Estimativa**: TBD

**Critérios de Aceitação**:

- [ ] **Dados da transferência**:
  - Lote de origem (select do inventário)
  - Data de transferência
  - Número do documento sanitario da transferência
  - Código do lote na transferência
  - Peso transferido (kg)
  - Unidade industrial destino (texto)
  - Link opcional ao relatório de embarque (ShippingReport type=transferência)

- [ ] **Válidações**:
  - Peso transferido <= saldo do lote
  - Saldo atualizado: `currentStockKg = gerado - somaTransferido`
  - Histórico de transferências por lote

---

### Feature 7.3: Inventário de Rotulagem (FM 7.1.3.6)

#### SIH-038: Registrar Movimentação de Rotulagem

```
Como supervisor muculmano,
Eu quero registrar entradas e saidas do inventario de rotulagem,
Para que o rastreamento de produtos rotulados seja mantido.
```

**Prioridade**: P2 - Nice to Have (Futuro)
**Estimativa**: TBD

**Critérios de Aceitação**:

- [ ] **Dados por registro**:
  - Data
  - Categoria do produto (E 340, POUCH, EXTRATO, HAMBURGUER, etc.)
  - Código interno (sem rótulo)
  - Código do produto rotulado
  - Nome do produto

- [ ] **Entradas**:
  - Quantidade recebida sem rótulo
  - Quantidade rotulada

- [ ] **Saídas**:
  - Quantidade embarcada
  - Número do pedido
  - Quantidade descartada (com rótulo)
  - Quantidade descartada (sem rótulo)

- [ ] **Saldos calculados**:
  - Estoque com rotulagem = acumulado(rotulados) - acumulado(embarcados + descarte rotulado)
  - Estoque sem rotulagem = acumulado(recebidos) - acumulado(rotulados + descarte sem rótulo)

---

#### SIH-039: Dashboard de Inventário

```
Como coordenador ou gestor,
Eu quero um dashboard consolidado de inventario,
Para que eu tenha visibilidade dos estoques Halal em cada planta.
```

**Prioridade**: P2 - Nice to Have (Futuro)
**Estimativa**: TBD

**Critérios de Aceitação**:

- [ ] **Visão consolidada por planta**:
  - Inventário de carne: saldo atual por tipo de corte e especie
  - Inventário de lotes: lotes em estoque, lotes transferidos
  - Inventário de rotulagem: estoque com/sem rótulo por categoria

- [ ] **Alertas**:
  - Saldos negativos (inconsistência)
  - Lotes com muito tempo em estoque
  - Saldos zerados (sem material disponível)

---

## Decisoes de Design para Implementação Futura

| Decisao | Detalhe |
|---------|---------|
| **Saldos** | Calculados via triggers/views no banco (não em código aplicação) |
| **Particionamento** | Possível particionamento mensal para tabelas de alto volume (FM 7.1.5.6: ~1000 linhas/mês) |
| **Migração** | Importação em massa de planilhas Excel existentes (dados históricos) |
| **Migration** | Tabelas criadas via migration separada (não inclusa na init) |
| **Módulos backend** | 3 módulos separados: `inventory/meat/`, `inventory/batch/`, `inventory/labeling/` |

## Dados de Referência (Planilhas Analisadas)

| Planilha | Tipo | Abas | Linhas | Observação |
|----------|------|:----:|:------:|------------|
| FM 7.1.5.1 - INVENTARIO MSP.xlsx | Carne | 2 | 1.702 | Bovinos (1530) + Aves (172) |
| FM 7.1.5.6 - INVENTARIO GELNEX.xlsx | Lotes | 11 | ~11.000 | Mensal (mar/25 - jan/26) |
| FM 7.1.3.6 - ODERICH.xlsx | Rotulagem | 3 | ~2.000 | 3 categorias |
| PLANILHA INVENTARIO PAMPEANO.xlsx | Rotulagem | 11 | ~15.000 | 11 categorias (E 340: 8068 linhas) |
| PLANILHA INVENTARIO CARAPRETA.xlsx | Produto | 2 | ~500 | Hamburguer + File Mignon |
| inventário kin master.xlsx | Produto | 1 | ~200 | Tracking individual |
