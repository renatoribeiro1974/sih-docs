---
title: BI/Analytics
nav_order: 3
---

# Módulo de BI/Analytics

**Projeto**: SIH - Supervisão Industrial Halal
**Status**: EM IMPLEMENTAÇÃO
**Última Atualização**: 2026-02-26

---

## Objetivo

Criar módulo de Business Intelligence com gráficos interativos, filtros por período/planta, e indicadores de performance operacional para supervisão Halal.

**Biblioteca de gráficos**: Recharts (React-native, composable, ~45KB gzipped)

---

## Estrutura de Páginas

| Rota | Página | Conteúdo |
|------|--------|----------|
| /analytics | Visão Geral | 6 KPIs + 5 gráficos |
| /analytics/slaughter | Abate | 4 gráficos + tabela |
| /analytics/production | Produção | 3 gráficos + tabela |
| /analytics/shipping | Embarque | 3 gráficos |
| /analytics/non-conformities | NCs | 5 gráficos |
| /analytics/supervisors | Supervisores | 3 gráficos + ranking (admin/coord only) |

---

## Indicadores

### Visão Geral

**Filtros globais**: Período (7/30/90 dias, mês, custom) + Planta (multi-select)

**KPIs**:
- Total Animais Processados
- Produção Total (kg)
- Embarques Realizados
- Taxa de Conformidade (%)
- NCs Abertas
- Tempo Médio Resolução NC (dias)

**Gráficos**:
1. Tendência de Relatórios (LineChart) — 3 linhas por tipo
2. Volume de Abate (BarChart stacked) — aprovados vs rejeitados
3. NCs por Severidade (DonutChart) — 4 fatias
4. Embarques por Tipo (BarChart horizontal) — 3 barras
5. Top 5 Plantas por Volume (BarChart horizontal stacked)

### Abate

**Filtros**: espécie, turno

1. Volume por Período (AreaChart stacked) + média móvel 7 dias
2. Distribuição por Espécie (PieChart)
3. Taxa de Rejeição por Planta (BarChart) + linha referência
4. Volume por Turno (BarChart agrupado)
5. Tabela resumo (sort + CSV export)

### Produção

1. Volume (kg) por Período (AreaChart)
2. Top 10 Produtos (BarChart horizontal)
3. Embalagens por Tipo (PieChart)
4. Tabela resumo

### Embarque

1. Por Tipo ao Longo do Tempo (BarChart stacked)
2. Destinos de Exportação (BarChart horizontal)
3. Por Tipo de Transporte (PieChart)

### Não-Conformidades

1. Tendência por Severidade (AreaChart stacked)
2. Tempo de Resolução por Severidade (BarChart — média, mediana, P90)
3. Funil de Status (BarChart horizontal)
4. NCs por Planta (BarChart stacked por severidade)
5. NCs Vencidas (KPI + tabela)

### Performance Supervisores (admin + coordenador)

1. Relatórios por Supervisor (BarChart horizontal stacked)
2. Tempo Médio de Assinatura (BarChart)
3. Cobertura de Escalas (heatmap/calendar)
4. Tabela ranking (sort + CSV export)

---

## Backend — Endpoints

6 novos endpoints no controller `/dashboard`:

```
GET /dashboard/reports-trend
GET /dashboard/slaughter-analytics
GET /dashboard/production-analytics
GET /dashboard/shipping-analytics
GET /dashboard/nc-analytics
GET /dashboard/supervisor-analytics
```

Todos aceitam: `dateFrom`, `dateTo`, `plantId?` + filtros específicos.

---

## Frontend — Componentes Reutilizáveis

| Componente | Descrição |
|-----------|-----------|
| DateRangeFilter | Seletor de período com presets |
| PlantFilter | Multi-select de plantas |
| KPICard | Card com ícone + valor + label |
| TrendLineChart | Line/Area chart wrapper |
| DistributionPieChart | Pie/Donut chart wrapper |
| RankingBarChart | Bar chart horizontal |
| StackedBarChart | Bar chart stacked |
| DataTable | Tabela com sort + export CSV |
| ChartCard | Container card com loading state |
