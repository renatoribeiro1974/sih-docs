---
title: Epic 02 - Relatórios de Produção
parent: User Stories
grand_parent: PRD
nav_order: 2
---

# Epic 02: Relatórios de Produção Industrial

**FM 7.1.3.1 (Industrializados) + FM 7.1.8.x (Produção Especial)**
**6 User Stories | 40 Story Points | Prioridade P0**

---

## Contexto

Os relatórios de produção acompanham a fabricação de produtos industrializados a base de carne Halal. O supervisor registra as matérias-primas cárneas utilizadas (com rastreabilidade completa), os ingredientes aprovados (não-carneos), os dados do produto final e os itens de verificação C/NC.

### Modelo de Dados: `ProductionReport`

- `formNumber`: "FM 7.1.3.1" (regular) ou "FM 7.1.8.x" (especial)
- `isSpecialProduction`: Boolean para distinguir produção especial
- `meatRawMaterials`: JSON com rastreabilidade completa
- `approvedIngredients`: JSON com ingredientes não-carneos
- 5 itens de verificação C/NC fixos
- Serial automático: `SIF/ANO/SEQUENCIAL`

### Código Relacionado

- Backend: `src/production-report/` (module, controller, service, DTOs)
- Backend: `src/production-report/constants/verification-items.ts`
- Frontend: `src/pages/production/` (List, Form, Details)
- Componentes: `VerificationChecklist`, `ProductTable`, `ReportHeader`

---

## User Stories

### Feature 2.1: CRUD de Relatório de Produção

#### SIH-008: Criar Relatório de Produção Industrial

```
Como supervisor muculmano,
Eu quero criar um novo relatorio de producao industrial,
Para que eu registre o acompanhamento da fabricacao de produtos Halal.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Sistema gera número serial automáticamente**:
  - Formato: `{SIF_PLANTA}/{ANO}/{SEQUENCIAL_6_DIGITOS}`
  - `formNumber` preenchido como "FM 7.1.3.1" (ou "FM 7.1.8.x" se produção especial)

- [ ] **Campos de período de produção**:
  - Data/hora início da produção (obrigatório)
  - Data/hora fim da produção (obrigatório)
  - Quantidade de turnos operados (inteiro > 0)

- [ ] **Campos do produto final**:
  - Nome do produto fabricado (obrigatório)
  - Código do produto
  - Lote
  - Data de fabricação
  - Data de válidade
  - Tipo de embalagem (caixa, saco, tambor)
  - Número de volumes
  - Peso liquido (kg, decimal 12,3)
  - Peso bruto (kg, decimal 12,3)
  - Quantidade total produzida (decimal 12,3)

- [ ] **Flag de produção especial**:
  - Toggle para marcar como FM 7.1.8.x
  - Quando ativado, `formNumber` muda para "FM 7.1.8.x"

- [ ] **Salva como rascunho por padrão**

---

#### SIH-009: Registrar Matérias-Primas Cárneas

```
Como supervisor muculmano,
Eu quero registrar as materias-primas carneas utilizadas na producao,
Para que haja rastreabilidade completa da origem da carne Halal.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points
**Dependências**: SIH-008

**Critérios de Aceitação**:

- [ ] **Tabela editavel de matérias-primas cárneas** com campos por linha:
  - Tipo de proteina (texto: "Bovino CMS", "Mecanicamente Separada", etc.)
  - Nome do frigorífico/abatedouro de origem
  - Número SIF do frigorífico
  - Período de abate (data início - data fim)
  - Número CSN (Certificado Sanitario Nacional)
  - Número do certificado Halal ou controle interno
  - Peso total (kg)

- [ ] **Operações na tabela**:
  - Adicionar nova linha
  - Remover linha (com confirmação)
  - Editar campos inline
  - Mínimo 1 materia-prima para envio (validação)

- [ ] **Armazenamento**: JSON no campo `meatRawMaterials`

- [ ] **Interface otimizada para tablet**:
  - Scroll horizontal na tabela se necessário
  - Campos com teclado numérico para SIF e pesos
  - Autocomplete para frigoríficos já cadastrados (histórico)

---

#### SIH-010: Registrar Ingredientes Aprovados

```
Como supervisor muculmano,
Eu quero registrar os ingredientes nao-carneos aprovados,
Para que eu documente todos os insumos utilizados na producao.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 5 story points
**Dependências**: SIH-008

**Critérios de Aceitação**:

- [ ] **Tabela editavel de ingredientes aprovados** com campos por linha:
  - Nome do produto/ingrediente
  - Código do produto
  - Nome do fornecedor
  - Número do lote
  - Data de válidade

- [ ] **Operações na tabela**:
  - Adicionar, remover, editar inline
  - Campo opcional (pode não ter ingredientes não-carneos)

- [ ] **Armazenamento**: JSON no campo `approvedIngredients`

---

### Feature 2.2: Verificação e Workflow

#### SIH-011: Preencher Itens de Verificação C/NC da Produção

```
Como supervisor muculmano,
Eu quero preencher os 5 itens de verificacao C/NC da producao,
Para que eu registre a conformidade dos processos produtivos.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 3 story points

**Critérios de Aceitação**:

- [ ] **Sistema exibe os 5 itens fixos de verificação** (conforme FM 7.1.3.1):
  1. Limpeza e higiene da area de produção e equipamentos
  2. Matérias-primas cárneas com documentação Halal válida
  3. Documentação de produção e rastreabilidade completa
  4. Armazenamento adequado (temperatura, separação Halal)
  5. Rotulagem conforme requisitos Halal

- [ ] **Mesmo componente `VerificationChecklist`** do Epic 01:
  - Selecao C/NC por item
  - Notas obrigatórias se NC
  - JSON retornado no formato padrão

---

#### SIH-012: Enviar e Revisar Relatório de Produção

```
Como supervisor muculmano,
Eu quero enviar meu relatorio de producao para revisao,
Para que o coordenador avalie a conformidade do registro.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points
**Dependências**: SIH-008, SIH-009, SIH-011

**Critérios de Aceitação**:

- [ ] **Validação completa antes do envio**:
  - Campos de período preenchidos (início e fim)
  - Pelo menos 1 materia-prima cárnea registrada
  - Todos os 5 itens de verificação respondidos
  - Dados do produto final preenchidos (nome obrigatório)

- [ ] **Workflow identico ao Epic 01**:
  - `rascunho` → `enviado` → `revisado` → `aprovado`/`rejeitado`
  - Declaração do supervisor antes do envio
  - Revisão pelo coordenador com aprovar/rejeitar

- [ ] **Cancelamento e substituicao**:
  - Mesmo mecanismo do Epic 01 (serial + sufixo "A")

---

### Feature 2.3: Listagem e Consulta

#### SIH-013: Listar e Filtrar Relatórios de Produção

```
Como supervisor ou coordenador,
Eu quero listar e filtrar relatorios de producao,
Para que eu encontre registros especificos de fabricacao.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Listagem páginada** com colunas:
  - Serial | Período | Planta | Produto | Supervisor | Status
  - Ordenação padrão: data de início descrescente

- [ ] **Filtros disponíveis**:
  - Planta (select)
  - Status (multi-select)
  - Período (daté range)
  - Produção especial (checkbox)
  - Supervisor (select - coordenador/gestor)
  - Produto (busca por nome)

- [ ] **Detalhes expandidos**:
  - Tabela de matérias-primas visivel no detalhe
  - Tabela de ingredientes visivel no detalhe
  - Itens C/NC com destaque visual para NCs

- [ ] **Responsivo**: Adaptado para tablet e desktop
