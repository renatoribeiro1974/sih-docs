---
title: Epic 06 - Dashboard
parent: User Stories
grand_parent: PRD
nav_order: 6
---

# Epic 06: Dashboard e Relatórios Gerênciais

**4 User Stories | 22 Story Points | Prioridade P1**

---

## Contexto

O dashboard fornece visão consolidada das atividades de supervisão industrial. Cada persona tem uma visão diferente: supervisor vê seus próprios indicadores, coordenador vê indicadores de toda a equipe, e gestor vê indicadores executivos.

### Código Relacionado

- Backend: `src/dashboard/` (module, controller, service)
- Frontend: `src/pages/Dashboard.tsx`

---

## User Stories

### Feature 6.1: Dashboard por Persona

#### SIH-030: Dashboard do Supervisor

```
Como supervisor muculmano,
Eu quero ver um dashboard com meus indicadores pessoais,
Para que eu acompanhe minha produtividade e pendencias.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 5 story points

**Critérios de Aceitação**:

- [ ] **Cards de resumo**:
  - Relatórios do dia (por tipo: abate, produção, embarque)
  - Relatórios da semana
  - Relatórios pendentes (rascunhos não enviados)
  - NCs abertas sob minha responsabilidade

- [ ] **Lista de ações pendentes**:
  - Rascunhos para finalizar
  - NCs para tratar (próximas do prazo em destaque)
  - Escala de hoje/próxima

- [ ] **Acesso rápido**:
  - Botão "Novo Relatório" com selecao de tipo
  - Link direto para a planta da escala de hoje

---

#### SIH-031: Dashboard do Coordenador

```
Como coordenador de supervisores,
Eu quero ver um dashboard com indicadores da equipe,
Para que eu monitore a operacao e tome acoes rapidamente.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Cards de resumo operacional**:
  - Total de relatórios hoje/semana/mês (por tipo)
  - Relatórios pendentes de revisão (fila)
  - NCs ativas por severidade (crítica, maior, menor, observação)
  - NCs próximas do prazo (< 3 dias) / vencidas
  - Supervisores em campo hoje (escala)

- [ ] **Filtros**:
  - Período (hoje, semana, mês, custom)
  - Planta (select ou "todas")

- [ ] **Tabela de relatórios pendentes de revisão**:
  - Top 10 mais antigos
  - Link direto para revisar
  - Badge de tipo (abate, produção, embarque)

- [ ] **Mapa de NCs por planta**:
  - Lista de plantas com contagem de NCs ativas
  - Cores: verde (0), amarelo (1-2), vermelho (3+)

---

#### SIH-032: Dashboard do Gestor

```
Como gestor de certificacao,
Eu quero ver um dashboard executivo com indicadores estrategicos,
Para que eu tenha visibilidade da operacao de supervisao.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 5 story points

**Critérios de Aceitação**:

- [ ] **Indicadores executivos**:
  - Total de relatórios no período (grafico de tendência)
  - Taxa de conformidade: % de relatórios sem NCs
  - NCs por severidade (grafico pizza/barra)
  - Plantas com mais NCs (ranking)
  - Cobertura de supervisão: % de turnos cobertos vs. programados

- [ ] **Graficos de tendência**:
  - Relatórios por semana (últimas 12 semanas)
  - NCs por semana
  - Taxa de resolução de NCs no prazo

- [ ] **Filtros**: Período, planta

---

### Feature 6.2: Relatórios e Exportação

#### SIH-033: Exportar Dados do Dashboard

```
Como coordenador ou gestor,
Eu quero exportar dados e indicadores,
Para que eu use em apresentacoes e relatorios externos.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 4 story points

**Critérios de Aceitação**:

- [ ] **Exportação de listagens**:
  - Relatórios filtrados → CSV
  - NCs filtradas → CSV
  - Escalas do período → CSV

- [ ] **Exportação de indicadores**:
  - Resumo do dashboard → PDF (futuro)
  - Dados brutos → JSON (para integração)

- [ ] **Filtros aplicados na exportação**: Mesmos filtros do dashboard ativo
