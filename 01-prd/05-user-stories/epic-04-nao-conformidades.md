---
title: Epic 04 - Não-Conformidades
parent: User Stories
grand_parent: PRD
nav_order: 4
---

# Epic 04: Gestão de Não-Conformidades

**FM 7.1.6.1 - Checklist de Não-Conformidade**
**6 User Stories | 35 Story Points | Prioridade P0**

---

## Contexto

As não-conformidades (NCs) são registradas quando o supervisor identifica desvios nos processos industriais. Podem ser criadas a partir de qualquer relatório (abate, produção, embarque) ou de forma avulsa. O PR 7.1 Rev 22 determina prazo de **7 dias corridos** para resolução.

### Modelo de Dados: `NonConformity`

- Vínculo opcional a `SlaughterReport`, `ProductionReport` ou `ShippingReport`
- `severity`: crítica, maior, menor, observação
- `category`: higiene, processo, equipamento, materia-prima, rotulagem, etc.
- Workflow: aberta → em_tratamento → resolvida → verificada → encerrada
- Prazo automático: 7 dias corridos (PR 7.1)
- Ações corretivas e preventivas

### Código Relacionado

- Backend: `src/non-conformity/` (module, controller, service, DTOs)
- Frontend: `src/pages/non-conformity/` (NCList, NCForm, NCDetails)

---

## User Stories

### Feature 4.1: Registro de Não-Conformidade

#### SIH-020: Registrar Não-Conformidade a partir de Relatório

```
Como supervisor muculmano,
Eu quero registrar uma nao-conformidade vinculada a um relatorio,
Para que o desvio identificado seja formalmente documentado com rastreabilidade.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Vínculo ao relatório de origem**:
  - Botão "Registrar NC" disponível na tela de detalhes de qualquer relatório
  - Preenche automáticamente: planta, supervisor, referência ao relatório
  - Campos `slaughterReportId`, `productionReportId` ou `shippingReportId` (mutuamente exclusivos)

- [ ] **Campos obrigatórios**:
  - Descrição da não-conformidade (texto, min 20 caracteres)
  - Severidade: crítica | maior | menor | observação
  - Categoria (select): higiene, processo, equipamento, materia-prima, rotulagem, documentação, infraestrutura, outro

- [ ] **Campos opcionais**:
  - Evidencias: array de descricoes e/ou fotos (JSON)
  - Ação corretiva sugerida
  - Ação preventiva sugerida

- [ ] **Prazo automático**:
  - `deadline` calculado como data_criação + 7 dias corridos
  - Exibido de forma proeminente no formulário

- [ ] **Status inicial**: `aberta`

---

#### SIH-021: Registrar Não-Conformidade Avulsa

```
Como supervisor muculmano,
Eu quero registrar uma nao-conformidade sem vincular a um relatorio,
Para que eu documente desvios observados fora do contexto de um relatorio especifico.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 5 story points

**Critérios de Aceitação**:

- [ ] **Formulário independente**:
  - Acessível via menu principal + botão de ação rápida
  - Selecao de planta obrigatória (não pre-preenchida)
  - Sem vínculo a relatórios (IDs de relatório = null)
  - Mesmos campos de SIH-020

- [ ] **Mesma lógica de prazo e status**

---

### Feature 4.2: Workflow de Tratamento

#### SIH-022: Tratar e Resolver Não-Conformidade

```
Como supervisor ou coordenador,
Eu quero registrar o tratamento e resolucao de uma NC,
Para que as acoes corretivas sejam documentadas dentro do prazo.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Transicoes de status**:
  - `aberta` → `em_tratamento`: Supervisor inicia o tratamento
    - Campo `correctiveAction` obrigatório (o que foi feito)
  - `em_tratamento` → `resolvida`: Supervisor marca como resolvida
    - Campo `preventiveAction` opcional (o que será feito para prevenir)
    - Registra `resolvedAt` e `resolvedBy`

- [ ] **Indicadores visuais de prazo**:
  - Verde: > 3 dias restantes
  - Amarelo: 1-3 dias restantes
  - Vermelho: vencida ou último dia
  - Badge com dias restantes

- [ ] **Alerta de prazo**:
  - Coordenador vê NCs próximas do vencimento no dashboard
  - NCs vencidas destacadas no topo da lista

---

#### SIH-023: Verificar e Encerrar Não-Conformidade

```
Como coordenador ou gestor,
Eu quero verificar e encerrar uma NC resolvida,
Para que eu confirme que as acoes corretivas foram eficazes.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 5 story points
**Dependências**: SIH-022

**Critérios de Aceitação**:

- [ ] **Transicoes de status** (apenas coordenador/gestor):
  - `resolvida` → `verificada`: Coordenador confirma resolução
    - Registra `verifiedAt` e `verifiedBy`
    - Pode adicionar comentários na verificação
  - `verificada` → `encerrada`: Gestor encerra definitivamente
  - `resolvida` → `em_tratamento`: Coordenador devolve (resolução insuficiente)
    - Obriga comentário explicando o que falta

- [ ] **Histórico de transicoes**: Registro de todas as mudanças de status com data/hora e usuário

---

### Feature 4.3: Listagem e Consulta

#### SIH-024: Listar e Filtrar Não-Conformidades

```
Como supervisor ou coordenador,
Eu quero listar e filtrar nao-conformidades,
Para que eu acompanhe o status de todas as NCs registradas.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 5 story points

**Critérios de Aceitação**:

- [ ] **Listagem páginada** com colunas:
  - # | Planta | Severidade | Categoria | Prazo | Status | Supervisor
  - Ordenação padrão: NCs vencidas primeiro, depois por prazo crescente

- [ ] **Filtros disponíveis**:
  - Planta (select)
  - Severidade (multi-select: crítica, maior, menor, observação)
  - Status (multi-select: aberta, em_tratamento, resolvida, verificada, encerrada)
  - Período de criação (daté range)
  - Supervisor (coordenador/gestor)
  - Origem (com relatório / avulsa)

- [ ] **Indicadores visuais**:
  - Badges de severidade com cores (vermelho=crítica, laranja=maior, amarelo=menor, cinza=observação)
  - Badges de status com cores
  - Icone de alerta para NCs próximas do prazo/vencidas

---

#### SIH-025: Visualizar Detalhes de Não-Conformidade

```
Como coordenador ou gestor,
Eu quero visualizar os detalhes completos de uma NC,
Para que eu acompanhe todo o historico de tratamento.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 4 story points

**Critérios de Aceitação**:

- [ ] **Tela de detalhes organizada em seções**:
  - Cabeçalho: planta, supervisor, data, severidade, categoria
  - Relatório de origem (link clicavel, se vinculada)
  - Descrição da NC
  - Evidencias
  - Ação corretiva (se preenchida)
  - Ação preventiva (se preenchida)
  - Prazo com indicador visual
  - Status atual com workflow visual (stepper)
  - Histórico de transicoes (timeline)

- [ ] **Ações disponveis por role e status**:
  - Supervisor: tratar (aberta → em_tratamento), resolver (em_tratamento → resolvida)
  - Coordenador: verificar, devolver, encerrar
  - Gestor: encerrar
