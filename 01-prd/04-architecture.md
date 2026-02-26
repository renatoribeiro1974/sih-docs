---
title: Arquitetura de Features
parent: PRD
nav_order: 4
---

# 4. Arquitetura de Features

---

## 4.1 Visão Geral dos Épicos

O SIH está organizado em **7 épicos**, sendo 6 para implementação na v1.0 e 1 documentado para implementação futura.

```
SIH v1.0
├── Epic 01: Relatorios de Abate (FM 7.1.4.x)
├── Epic 02: Relatorios de Producao (FM 7.1.3.x / FM 7.1.8.x)
├── Epic 03: Relatorios de Embarque (FM 7.1.7.x / DCPOA)
├── Epic 04: Nao-Conformidades (FM 7.1.6.1)
├── Epic 05: Escala de Supervisores
├── Epic 06: Dashboard e Relatorios
│
SIH Futuro
└── Epic 07: Inventario (FM 7.1.5.x / FM 7.1.3.6) [DOCUMENTADO]
```

---

## 4.2 Mapeamento FM → Épico → Módulo

| FM | Épico | Módulo Backend | Páginas Frontend |
|----|-------|----------------|------------------|
| FM 7.1.4.1 (Aves) | Epic 01 | `slaughter-report/` | `pages/slaughter/` |
| FM 7.1.4.2 (Bovino) | Epic 01 | `slaughter-report/` | `pages/slaughter/` |
| FM 7.1.3.1 (Produção) | Epic 02 | `production-report/` | `pages/production/` |
| FM 7.1.8.x (Especial) | Epic 02 | `production-report/` | `pages/production/` |
| FM 7.1.7.1 (Exportação) | Epic 03 | `shipping-report/` | `pages/shipping/` |
| FM 7.1.7.4 (Venda) | Epic 03 | `shipping-report/` | `pages/shipping/` |
| DCPOA (Transferência) | Epic 03 | `shipping-report/` | `pages/shipping/` |
| FM 7.1.6.1 (NC) | Epic 04 | `non-conformity/` | `pages/non-conformity/` |
| - (Escala) | Epic 05 | `schedule/` | `pages/schedule/` |
| - (Dashboard) | Epic 06 | `dashboard/` | `pages/Dashboard.tsx` |
| FM 7.1.5.1 (Carne) | Epic 07 | `inventory/meat/` [futuro] | `pages/inventory/` [futuro] |
| FM 7.1.5.6 (Lotes) | Epic 07 | `inventory/batch/` [futuro] | `pages/inventory/` [futuro] |
| FM 7.1.3.6 (Rotulagem) | Epic 07 | `inventory/labeling/` [futuro] | `pages/inventory/` [futuro] |

---

## 4.3 Resumo por Épico

### Epic 01: Relatórios de Abate

**FM**: 7.1.4.1 (Aves) + 7.1.4.2 (Bovino)
**Prioridade**: Must Have (P0)

Digitaliza os relatórios de acompanhamento de abate Halal. Cobre aves e bovinos com um único modelo (`SlaughterReport`) diferenciado por especie. Bovinos possuem seção adicional de insensibilização (stunning).

**Componentes-chave**:
- 14 itens de verificação C/NC (fixos, não configuráveis)
- Seção de insensibilização (bovino): avaliação de pressão + 2 conferências/turno
- Contagem: total, aprovados, rejeitados + sequencial de rejeicao
- Camaras de resfriamento, subprodutos
- Número serial automático (SIF/ANO/SEQ)
- Workflow: rascunho → assinado (final). Sem etapa de aprovação

**User Stories**: 7 | **Story Points**: 47

---

### Epic 02: Relatórios de Produção

**FM**: 7.1.3.1 (Industrializados) + 7.1.8.x (Produção Especial)
**Prioridade**: Must Have (P0)

Digitaliza o acompanhamento de fabricação de produtos industrializados a base de carne Halal. Inclui rastreabilidade completa de matérias-primas cárneas e ingredientes aprovados.

**Componentes-chave**:
- 5 itens de verificação C/NC (fixos)
- Tabela de matérias-primas cárneas (frigorífico, SIF, data abate, CSN, certificado Halal)
- Tabela de ingredientes aprovados (não-carneos: fornecedor, lote, válidade)
- Dados do produto final (código, lote, datas, pesos, embalagem)
- Flag `specialProduction` para FM 7.1.8.x

**User Stories**: 6 | **Story Points**: 40

---

### Epic 03: Relatórios de Embarque, Venda e Transferência

**FM**: 7.1.7.1 (Exportação) + 7.1.7.4 (Venda Interna) + DCPOA (Transferência)
**Prioridade**: Must Have (P0)

Unifica 3 tipos de movimentação de produtos em um único modelo (`ShippingReport`) diferenciado por `shippingType`. Exportação e o mais completo com dados de importador, container, portos e lacre.

**Componentes-chave**:
- 2 itens de verificação C/NC (fixos: exclusividade Halal no container + selo)
- Tabela de produtos padrão (produto, código, lote, datas, pesos, temperatura)
- Campos condicionais por tipo: exportação (importer, container, ports) vs. venda interna vs. transferência
- Dados de transporte (tipo, veiculo, container, lacre)

**User Stories**: 6 | **Story Points**: 38

---

### Epic 04: Não-Conformidades

**FM**: 7.1.6.1
**Prioridade**: Must Have (P0)

Gestão completa de não-conformidades identificadas durante a supervisão. Pode ser criada a partir de qualquer relatório (abate, produção, embarque) ou de forma avulsa. Implementa o prazo de 7 dias corridos conforme PR 7.1.

**Componentes-chave**:
- Vínculo opcional a relatórios de origem (1 NC pode vir de qualquer tipo)
- Severidade: crítica, maior, menor, observação
- Categorias: higiene, processo, equipamento, materia-prima, rotulagem, etc.
- Evidencias (fotos/descricoes)
- Workflow: aberta → em_tratamento → resolvida → verificada → encerrada
- Prazo automático de 7 dias com alertas
- Ações corretivas e preventivas

**User Stories**: 6 | **Story Points**: 35

---

### Epic 05: Escala de Supervisores

**Prioridade**: Should Have (P1)

Gestão da escala de trabalho dos supervisores nas plantas industriais. Permite ao coordenador distribuir supervisores por planta, turno e tipo de escala.

**Componentes-chave**:
- Calendário visual mensal
- Alocação: supervisor + planta + data + turno
- Tipos: regular, substituicao, extra, folga
- Restrição: um supervisor por planta/data/turno (unique constraint)
- Visão por supervisor e por planta

**User Stories**: 4 | **Story Points**: 20

---

### Epic 06: Dashboard e Relatórios

**Prioridade**: Should Have (P1)

Dashboard em tempo real com indicadores operacionais e de conformidade. Visualizações diferentes por role (supervisor/operador básico, coordenador/admin completo).

**Componentes-chave**:
- Contadores: relatórios do dia/semana/mês por tipo e status
- NCs ativas por severidade e planta
- Relatórios pendentes de revisão
- Produtividade por supervisor
- Graficos de tendência

**User Stories**: 4 | **Story Points**: 22

---

### Epic 07: Inventário (FUTURO)

**FM**: 7.1.5.1 (Carne) + 7.1.5.6 (Lotes) + 7.1.3.6 (Rotulagem)
**Prioridade**: Nice to Have (P2) - Implementação futura

Módulo de inventário baseado no conceito de **conta corrente** (saldo = entrada - saída). Modelo de dados já documentado para evitar retrabalho. Inclui 3 tipos distintos de inventário com volumes de dados significativos (1.000+ linhas/mês).

**Componentes-chave** (documentados):
- Conta corrente de carne: recebido vs. utilizado em produção
- Inventário de lotes: gerado vs. transferido entre unidades
- Inventário de rotulagem: sem rótulo → rotulado → embarcado/descartado
- Referências cruzadas entre inventário, relatórios e certificados

**User Stories**: 6 | **Story Points**: TBD (será estimado quando implementado)

---

## 4.4 Dependências entre Épicos

```
[Auth Self-Contained] ← Prerequisito para todos
       │
       ├── Epic 01: Abate ──────┐
       ├── Epic 02: Producao ───┤──→ Epic 04: NCs (vinculo a relatorios)
       ├── Epic 03: Embarque ───┘
       │
       ├── Epic 05: Escala (independente, usa plantas + supervisores)
       │
       └── Epic 06: Dashboard (depende de dados dos epicos 01-05)
              │
              └── Epic 07: Inventario [FUTURO] (depende de 01-03 para referencias cruzadas)
```

---

## 4.5 Componentes Compartilhados

Os seguintes componentes frontend são reutilizados entre multiplos épicos:

| Componente | Usado por | Descrição |
|------------|-----------|-----------|
| `VerificationChecklist` | Epics 01, 02, 03 | Renderiza itens C/NC com checkboxes. Recebe lista de itens como prop (diferentes por tipo FM). |
| `ProductTable` | Epics 02, 03 | Tabela editavel de produtos (produto, código, lote, datas, pesos, temperatura). |
| `ReportHeader` | Epics 01, 02, 03 | Cabeçalho padrão: número FM, serial, planta, supervisor, data. |

---

## 4.6 Contagem Total

| Item | Quantidade |
|------|-----------|
| Épicos v1.0 | 6 |
| Épicos futuros | 1 |
| User Stories v1.0 | 33 |
| User Stories futuras | 6 |
| **Story Points v1.0** | **202** |
