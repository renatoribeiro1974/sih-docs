---
title: Epic 05 - Escala de Supervisores
parent: User Stories
grand_parent: PRD
nav_order: 5
---

# Epic 05: Escala de Supervisores

**4 User Stories | 20 Story Points | Prioridade P1**

---

## Contexto

A escala de supervisores controla a alocação de supervisores muçulmanos nas plantas industriais. Cada supervisor e designado para uma planta em uma data e turno específicos. O coordenador monta a escala e pode fazer substituicoes, escalas extras ou registrar folgas.

### Modelo de Dados: `SupervisorSchedule`

- Relação: supervisor + planta + data + turno (unique constraint)
- Tipos: regular, substituicao, extra, folga
- Restrição: um supervisor por planta/data/turno

### Código Relacionado

- Backend: `src/schedule/` (module, controller, service, DTOs)
- Frontend: `src/pages/schedule/` (ScheduleCalendar, ScheduleManagement)

---

## User Stories

### Feature 5.1: Gestão de Escala

#### SIH-026: Criar Alocação de Supervisor na Escala

```
Como coordenador de supervisores,
Eu quero alocar um supervisor em uma planta para uma data e turno,
Para que a cobertura de supervisao Halal esteja organizada.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 5 story points

**Critérios de Aceitação**:

- [ ] **Campos da alocação**:
  - Supervisor (select entre supervisores ativos)
  - Planta (select entre plantas ativas)
  - Data (daté picker)
  - Turno: matutino, vespertino, noturno, integral
  - Tipo: regular (padrão), substituicao, extra, folga
  - Notas (texto livre, opcional)

- [ ] **Válidações**:
  - Não permite duplicata: mesmo supervisor + planta + data + turno
  - Alerta se supervisor já está alocado em outra planta na mesma data/turno
  - Não permite alocar supervisor inativo

- [ ] **Criação em lote** (nice to have):
  - Selecionar range de datas para criar escalas recorrentes
  - Copiar escala da semana anterior

---

#### SIH-027: Visualizar Escala em Calendário

```
Como coordenador de supervisores,
Eu quero visualizar a escala em formato de calendario,
Para que eu tenha visao clara da cobertura de todas as plantas.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Calendário mensal interativo**:
  - Visão por mês (padrão)
  - Cada dia mostra alocações com nome do supervisor e planta
  - Cores por tipo: regular (azul), substituicao (amarelo), extra (verde), folga (cinza)

- [ ] **Dois modos de visualização**:
  - **Por planta**: Calendário de uma planta, mostrando qual supervisor está alocado em cada dia/turno
  - **Por supervisor**: Calendário de um supervisor, mostrando em quais plantas está alocado

- [ ] **Interação**:
  - Clicar em dia vazio: criar nova alocação
  - Clicar em alocação existente: editar/remover
  - Drag-and-drop (nice to have): mover alocação entre datas

- [ ] **Responsivo**: Funciona em tablet (modo landscape recomendado)

---

#### SIH-028: Editar e Remover Alocação da Escala

```
Como coordenador de supervisores,
Eu quero editar ou remover alocacoes da escala,
Para que eu ajuste a cobertura conforme necessidade.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 3 story points

**Critérios de Aceitação**:

- [ ] **Edição**:
  - Trocar supervisor (substituicao)
  - Mudar turno
  - Mudar tipo (regular → substituicao, etc.)
  - Adicionar/editar notas

- [ ] **Remocao**:
  - Confirmação antes de remover
  - Não permite remover se supervisor já tem relatórios na data/planta

---

### Feature 5.2: Consulta de Escala

#### SIH-029: Supervisor Consulta Própria Escala

```
Como supervisor muculmano,
Eu quero consultar minha escala de trabalho,
Para que eu saiba em quais plantas devo comparecer.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 4 story points

**Critérios de Aceitação**:

- [ ] **Visão simplificada**:
  - Lista cronológica das próximas alocações
  - Filtro por mês
  - Informações: data, planta (nome + SIF), turno, tipo

- [ ] **Destaque**:
  - Alocação de hoje em destaque
  - Próxima alocação em evidencia

- [ ] **Acesso rápido**:
  - Da escala, clicar na planta leva aos relatórios daquela planta
  - Atalho para criar novo relatório na planta da escala de hoje
