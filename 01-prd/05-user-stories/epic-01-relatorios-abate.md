---
title: Epic 01 - Relatórios de Abate
parent: User Stories
grand_parent: PRD
nav_order: 1
---

# Epic 01: Relatórios de Abate Halal

**FM 7.1.4.1 (Aves) + FM 7.1.4.2 (Bovino)**
**7 User Stories | 47 Story Points | Prioridade P0**

---

## Contexto

Os relatórios de abate são preenchidos diáriamente pelos supervisores muçulmanos em plantas de abate (abatedouros e frigoríficos). Cada relatório registra os dados de um turno de abate, incluindo contagens de animais, itens de verificação C/NC, e para bovinos, a seção de insensibilização (stunning).

### Modelo de Dados: `SlaughterReport`

- Diferenciado por `species` (ave vs. bovino)
- `formNumber`: "FM 7.1.4.1" (aves) ou "FM 7.1.4.2" (bovino)
- 14 itens de verificação C/NC fixos
- Seção de stunning exclusiva para bovinos
- Serial automático: `SIF/ANO/SEQUENCIAL`

### Código Relacionado

- Backend: `src/slaughter-report/` (module, controller, service, DTOs)
- Backend: `src/slaughter-report/constants/verification-items.ts`
- Frontend: `src/pages/slaughter/` (List, Form, Details)
- Componentes: `VerificationChecklist`, `ReportHeader`

---

## User Stories

### Feature 1.1: CRUD de Relatório de Abate

#### SIH-001: Criar Relatório de Abate

```
Como supervisor muculmano,
Eu quero criar um novo relatorio de abate Halal,
Para que eu registre digitalmente os dados do turno de abate na planta.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Sistema permite selecionar especie** (ave, bovino, ovino, caprino, bubalino):
  - Formulário ajusta campos condicionais baseado na especie
  - Para bovino: exibe seção de insensibilização
  - Para outros: oculta seção de insensibilização

- [ ] **Sistema gera número serial automáticamente**:
  - Formato: `{SIF_PLANTA}/{ANO}/{SEQUENCIAL_6_DIGITOS}`
  - Exemplo: `451/2026/000001`
  - Sequencial único por planta/ano, sem duplicatas
  - `formNumber` preenchido automáticamente ("FM 7.1.4.1" ou "FM 7.1.4.2")

- [ ] **Sistema exibe campos de identificação**:
  - Planta (selecionada ou pre-definida pela escala)
  - Data do abate
  - Turno (matutino, vespertino, noturno, integral)
  - Nome do degolador
  - Registro/documento do degolador

- [ ] **Sistema exibe campos de contagem**:
  - Total de animais abatidos (obrigatório, inteiro > 0)
  - Animais aprovados (obrigatório, inteiro >= 0)
  - Animais rejeitados (default 0)
  - Sequencial das cabecas rejeitadas (texto livre, se rejeitados > 0)
  - Validação: aprovados + rejeitados = total

- [ ] **Sistema salva como rascunho por padrão**:
  - Status inicial: `rascunho`
  - Pode ser editado e retomado posteriormente
  - Não exige todos os campos obrigatórios para salvar como rascunho

---

#### SIH-002: Preencher Itens de Verificação C/NC do Abate

```
Como supervisor muculmano,
Eu quero preencher os 14 itens de verificacao C/NC do formulario de abate,
Para que eu registre a conformidade de cada item conforme o PR 7.1.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 5 story points

**Critérios de Aceitação**:

- [ ] **Sistema exibe os 14 itens fixos de verificação** (conforme FM 7.1.4.x):
  1. Limpeza e higiene do local de abate
  2. Condição dos equipamentos
  3. Identificação dos animais
  4. Bem-estar animal no curral de espera
  5. Conducao dos animais a area de abate
  6. Posicionamento para degola
  7. Invocação do nome de Allah (Bismillah)
  8. Degola com instrumento afiado
  9. Corte das veias jugulares e arteria carotida
  10. Sangria completa
  11. Tempo de espera antes do processamento
  12. Separação Halal/Não-Halal
  13. Identificação das carcaças aprovadas
  14. Armazenamento adequado

- [ ] **Cada item permite selecao C (Conforme) ou NC (Não-Conforme)**:
  - Checkbox ou toggle para cada item
  - Campo de notas/observação opcional por item
  - Se NC selecionado: campo de notas se torna obrigatório

- [ ] **Componente `VerificationChecklist` reutilizavel**:
  - Recebe lista de itens como prop
  - Retorna JSON: `[{ id, description, result: "C"|"NC", notes? }]`
  - Interface otimizada para tablet (targets grandes, fácil de clicar)

---

#### SIH-003: Registrar Insensibilização (Stunning) - Bovino

```
Como supervisor muculmano em planta de abate bovino,
Eu quero registrar os dados de insensibilizacao (stunning),
Para que eu documente a avaliacao de conformidade deste processo critico.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points
**Dependências**: SIH-001 (apenas quando species=bovino)

**Critérios de Aceitação**:

- [ ] **Seção exibida apenas para bovinos** (species=bovino):
  - Oculta para aves e outras especies
  - Posicionada após os itens de verificação C/NC

- [ ] **Campo de avaliação geral**:
  - `stunningUsed`: Boolean (sim/não) - se o frigorífico utiliza insensibilização
  - `stunningEvaluation`: Texto livre para avaliação de pressão/parâmetros do equipamento

- [ ] **2 conferências por turno** (conforme FM 7.1.4.2):
  - Conferência 1 (início do turno):
    - Animal estava vivo no momento do abate? (Sim/Não)
    - Animal apresentou lesões craniais? (Sim/Não)
  - Conferência 2 (meio/fim do turno):
    - Mesmos campos da conferência 1
  - Armazenado em `stunningVerifications`: JSON array

- [ ] **Campos de camaras de resfriamento**:
  - `coolingCarcasses`: Qtd de carcaças enviadas ao resfriamento (inteiro)
  - `coolingCameras`: Identificação das camaras utilizadas (texto, ex: "C-10-11-09-14")

- [ ] **Campos de subprodutos**:
  - `byproductsSold`: Boolean - subprodutos foram comercializados?
  - `byproductsDescription`: Descrição dos subprodutos (se vendidos)

---

### Feature 1.2: Workflow e Gestão

#### SIH-004: Enviar Relatório de Abate para Revisão

```
Como supervisor muculmano,
Eu quero enviar meu relatorio de abate para revisao,
Para que o coordenador possa avaliar e aprovar meu registro.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 5 story points
**Dependências**: SIH-001, SIH-002

**Critérios de Aceitação**:

- [ ] **Validação completa antes do envio**:
  - Todos os campos obrigatórios preenchidos
  - Todos os 14 itens de verificação respondidos (C ou NC)
  - Para bovinos: seção de insensibilização preenchida
  - Contagem válida: aprovados + rejeitados = total
  - Se tem rejeitados: sequencial de rejeicao preenchido

- [ ] **Transicao de status**: `rascunho` → `enviado`
  - Data/hora de envio registrada
  - Relatório não pode mais ser editado pelo supervisor após envio
  - Apenas coordenador/gestor pode devolver para edição (rejeicao)

- [ ] **Declaração do supervisor**:
  - Texto padrão bilíngue (PT/EN) exibido antes do envio
  - Supervisor confirma ciencia/assinatura digital (aceite)

---

#### SIH-005: Revisar e Aprovar/Rejeitar Relatório de Abate

```
Como coordenador de supervisores,
Eu quero revisar e aprovar ou rejeitar relatorios de abate,
Para que eu garanta a qualidade dos registros antes da aprovacao final.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points
**Dependências**: SIH-004

**Critérios de Aceitação**:

- [ ] **Lista de relatórios pendentes de revisão**:
  - Filtro por status `enviado`
  - Ordenação por data (mais antigos primeiro)
  - Filtros: planta, supervisor, especie, período

- [ ] **Tela de revisão detalhada**:
  - Visualização completa do relatório (somente leitura)
  - Destaque visual para itens NC
  - Histórico de alterações (se houve rejeicao anterior)

- [ ] **Ações de revisão**:
  - **Aprovar**: `enviado` → `aprovado` (registra reviewedBy + reviewedAt)
  - **Rejeitar**: `enviado` → `rejeitado` (obriga comentário do motivo)
  - Relatório rejeitado volta como editavel para o supervisor

---

#### SIH-006: Cancelar Relatório de Abate

```
Como supervisor muculmano,
Eu quero cancelar um relatorio de abate com erro grave,
Para que eu emita um novo relatorio corrigido com rastreabilidade.
```

**Prioridade**: P1 - Should Have
**Estimativa**: 5 story points

**Critérios de Aceitação**:

- [ ] **Cancelamento com motivo obrigatório**:
  - Status → `cancelado`
  - Campo `cancelledReason` obrigatório
  - Relatório cancelado permanece no sistema (não e excluido)

- [ ] **Emissão de relatório substituto**:
  - Novo relatório criado com serial sufixo "A" (ex: `451/2026/000001A`)
  - Campo `replacedBy` no relatório cancelado aponta para o novo serial
  - Dados do relatório cancelado copiados para o novo (editavel)

---

### Feature 1.3: Listagem e Consulta

#### SIH-007: Listar e Filtrar Relatórios de Abate

```
Como supervisor ou coordenador,
Eu quero listar e filtrar relatorios de abate,
Para que eu encontre rapidamente relatorios especificos.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Listagem páginada** com colunas:
  - Serial | Data | Planta | Especie | Supervisor | Total Animais | Status
  - Ordenação padrão: data descrescente

- [ ] **Filtros disponíveis**:
  - Planta (select)
  - Especie (select)
  - Status (multi-select: rascunho, enviado, revisado, aprovado, rejeitado, cancelado)
  - Período (daté range)
  - Supervisor (select - apenas para coordenador/gestor)

- [ ] **Busca por serial**: Campo de busca rápida por número serial

- [ ] **Ações na lista**:
  - Clicar para ver detalhes
  - Editar (apenas rascunhos próprios ou coordenador)
  - Badge visual de status (cores diferenciadas)

- [ ] **Responsivo**: Layout adaptado para tablet (cards em mobile, tabela em desktop)
