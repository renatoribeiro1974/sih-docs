---
title: Epic 03 - Relatórios de Embarque
parent: User Stories
grand_parent: PRD
nav_order: 3
---

# Epic 03: Relatórios de Embarque, Venda e Transferência

**FM 7.1.7.1 (Exportação) + FM 7.1.7.4 (Venda Interna) + DCPOA (Transferência)**
**6 User Stories | 38 Story Points | Prioridade P0**

---

## Contexto

Os relatórios de embarque cobrem 3 tipos de movimentação de produtos Halal: exportação, venda no mercado interno e transferência entre unidades industriais. Todos compartilham um modelo único (`ShippingReport`) diferenciado pelo campo `shippingType`. Exportação e o mais completo, com dados de importador, container, portos e lacre.

### Modelo de Dados: `ShippingReport`

- `shippingType`: exportação | venda_interna | transferência
- `formNumber`: "FM 7.1.7.1" (exportação), "FM 7.1.7.4" (venda), "DCPOA" (transferência)
- `products`: JSON com tabela de produtos padrão
- 2 itens de verificação C/NC fixos
- Campos condicionais por tipo de embarque
- Serial automático: `SIF/ANO/SEQUENCIAL`

### Código Relacionado

- Backend: `src/shipping-report/` (module, controller, service, DTOs)
- Backend: `src/shipping-report/constants/verification-items.ts`
- Frontend: `src/pages/shipping/` (List, Form, Details)
- Componentes: `VerificationChecklist`, `ProductTable`, `ReportHeader`

---

## User Stories

### Feature 3.1: CRUD de Relatório de Embarque

#### SIH-014: Criar Relatório de Embarque/Venda/Transferência

```
Como supervisor muculmano,
Eu quero criar um novo relatorio de embarque, venda ou transferencia,
Para que eu registre a movimentacao de produtos Halal na planta.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Selecao do tipo de movimentação**:
  - Exportação (FM 7.1.7.1)
  - Venda Mercado Interno (FM 7.1.7.4)
  - Transferência (DCPOA)
  - `formNumber` preenchido automáticamente conforme o tipo

- [ ] **Campos comuns (todos os tipos)**:
  - Planta
  - Data do carregamento (obrigatório)
  - Tabela de produtos (ver SIH-015)
  - Observações

- [ ] **Campos exclusivos de Exportação** (FM 7.1.7.1):
  - Exportador (nome da empresa)
  - Frigorífico de abate (nome + SIF)
  - Unidade produtora (nome + SIF)
  - Endereço do carregamento
  - Importador/Consignatario
  - Tipo de transporte (terrestre, aereo, maritimo)
  - Porto de embarque
  - Identificação do veiculo (navio, caminhao, cia aerea)
  - Número do container
  - Porto de destino
  - País de destino
  - Número do pedido/Invoice
  - Número CSI (Certificado Sanitario Internacional)
  - Número do lacre

- [ ] **Campos condicionais ocultos para outros tipos**:
  - Venda interna: apenas campos comuns
  - Transferência: apenas campos comuns

- [ ] **Gera serial automáticamente** e salva como rascunho

---

#### SIH-015: Registrar Tabela de Produtos do Embarque

```
Como supervisor muculmano,
Eu quero registrar os produtos sendo embarcados/vendidos/transferidos,
Para que haja rastreabilidade completa da carga.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points
**Dependências**: SIH-014

**Critérios de Aceitação**:

- [ ] **Tabela editavel de produtos** (`ProductTable` componente) com campos por linha:
  - Nome do produto (obrigatório)
  - Código do produto
  - Lote
  - Período de abate (data início - data fim)
  - Período de produção (data início - data fim)
  - Período de válidade (data início - data fim)
  - Tipo de embalagem
  - Número de volumes
  - Peso liquido (kg, decimal 12,3)
  - Peso bruto (kg, decimal 12,3)
  - Temperatura de embarque (graus C)

- [ ] **Operações na tabela**:
  - Adicionar, remover, editar inline
  - Mínimo 1 produto para envio
  - Totais automáticos: soma de volumes, peso liquido, peso bruto

- [ ] **Armazenamento**: JSON no campo `products`

- [ ] **Componente `ProductTable` reutilizavel**:
  - Mesmo componente usado no Epic 02 (produção)
  - Props configuráveis: quais colunas exibir por contexto
  - Scroll horizontal em tablet

---

#### SIH-016: Preencher Verificação C/NC do Embarque

```
Como supervisor muculmano,
Eu quero preencher os itens de verificacao do embarque,
Para que eu confirme a conformidade Halal da carga.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 3 story points

**Critérios de Aceitação**:

- [ ] **Sistema exibe os 2 itens fixos de verificação** (conforme FM 7.1.7.1):
  1. Container/veiculo contém exclusivamente produtos Halal
  2. Selo de garantia Halal aplicado corretamente

- [ ] **Mesmo componente `VerificationChecklist`** dos Epics 01 e 02:
  - Selecao C/NC por item
  - Notas obrigatórias se NC

---

### Feature 3.2: Workflow e Gestão

#### SIH-017: Enviar e Revisar Relatório de Embarque

```
Como supervisor muculmano,
Eu quero enviar meu relatorio de embarque para revisao,
Para que o coordenador aprove o registro da movimentacao.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 5 story points
**Dependências**: SIH-014, SIH-015, SIH-016

**Critérios de Aceitação**:

- [ ] **Validação antes do envio**:
  - Tipo de embarque selecionado
  - Data de carregamento preenchida
  - Pelo menos 1 produto na tabela
  - Ambos itens de verificação respondidos
  - Para exportação: campos obrigatórios adicionais (importador, container, país destino)

- [ ] **Workflow identico aos Epics 01 e 02**:
  - `rascunho` → `enviado` → `aprovado`/`rejeitado`
  - Declaração do supervisor + cancelamento com serial "A"

---

### Feature 3.3: Listagem e Consulta

#### SIH-018: Listar e Filtrar Relatórios de Embarque

```
Como supervisor ou coordenador,
Eu quero listar e filtrar relatorios de embarque,
Para que eu encontre registros de movimentacao especificos.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 8 story points

**Critérios de Aceitação**:

- [ ] **Listagem páginada** com colunas:
  - Serial | Data | Planta | Tipo | Destino/Cliente | Supervisor | Status
  - Ordenação padrão: data descrescente

- [ ] **Filtros disponíveis**:
  - Planta (select)
  - Tipo de embarque (exportação, venda_interna, transferência)
  - Status (multi-select)
  - Período (daté range)
  - País de destino (apenas exportação)
  - Supervisor (coordenador/gestor)

- [ ] **Detalhes expandidos**:
  - Tabela de produtos completa
  - Dados de exportação (quando aplicavel)
  - Itens C/NC com destaque visual

- [ ] **Responsivo**: Adaptado para tablet e desktop

---

#### SIH-019: Visualizar Detalhes do Relatório de Embarque

```
Como coordenador ou gestor,
Eu quero visualizar os detalhes completos de um relatorio de embarque,
Para que eu revise todas as informacoes da movimentacao.
```

**Prioridade**: P0 - Must Have
**Estimativa**: 6 story points

**Critérios de Aceitação**:

- [ ] **Tela de detalhes organizada em seções**:
  - Cabeçalho (FM, serial, planta, supervisor, data)
  - Dados do embarque (tipo-específico)
  - Tabela de produtos (com totais)
  - Itens de verificação (com destaque NC)
  - Observações
  - Status e histórico de revisão

- [ ] **Ações disponíveis por role**:
  - Supervisor (próprio): editar rascunho
  - Coordenador: aprovar/rejeitar (se enviado)
  - Todos: ver detalhes (somente leitura)

- [ ] **Impressao/PDF** (futuro): Layout preparado para geração de PDF
