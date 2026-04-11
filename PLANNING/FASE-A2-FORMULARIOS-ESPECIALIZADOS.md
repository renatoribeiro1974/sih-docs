---
title: Fase A.2 — Formularios Especializados (Planejamento Detalhado)
nav_order: 3
---

# Fase A.2 — Formularios Especializados

**Projeto**: SIH - Supervisao Industrial Halal
**Status**: PLANEJAMENTO DETALHADO
**Pre-requisito**: v1.0 completa (9 fases + PDF Staff)
**Ultima Atualizacao**: 2026-04-10
**Baseado em**: Analise dos FMs FAMBRAS preenchidos reais (`C:\SIH\`)

---

## 1. Resumo Executivo

A analise dos formularios reais revelou que a Fase A.2 e **mais complexa do que o planejamento original** previa. Os 14 FMs previstos sao na verdade **17 FMs distintos** (3 descobertos), e os formularios se dividem em **6 padroes estruturais de producao** e **3 padroes de embarque** — nao variantes simples do FM base.

### Descobertas Criticas

| # | Descoberta | Impacto |
|---|-----------|---------|
| 1 | **FM 7.1.8.3 existe** (Heparina Purificacao) — 2a etapa, distinto do FM 7.1.8.2 | +1 tipo producao |
| 2 | **FM 7.1.4.9 existe** (Venda Subprodutos couro/raspa) — embarque especifico | +1 tipo embarque |
| 3 | **FM 7.1.8.X existe** (Transferencia Generica de Produto) — catch-all | +1 tipo embarque |
| 4 | **FM 7.1.4.5 e HIBRIDO** (producao + venda em 1 formulario) | Arquitetura diferente |
| 5 | **FM 7.1.4.6 e LOG DIARIO** (20+ linhas/dia, nao evento unico) | Estrutura totalmente diferente |
| 6 | **FM 7.1.3.5 e REEMBALAGEM** (tabela entrada + tabela saida) | Conceito diferente de producao |
| 7 | Cada FM tem **colunas diferentes** na tabela de materias-primas | JSON por tipo, nao tabela padrao |
| 8 | Cada FM tem **verificacoes diferentes** (2 a 6 itens, textos distintos) | Array de verificacoes por tipo |

---

## 2. Catalogo Completo dos FMs

### 2.1 Producao — 8 Tipos (7 novos + 1 existente)

| Tipo | FM | Titulo PT | Titulo EN | Rev | Padrao |
|------|-----|-----------|-----------|-----|--------|
| `fabricacao` | 7.1.3.1 | Fabricacao de Produtos Industrializados a Base de Carne | Manufacturing Industrialized Products – Contain Meat | 04 | A (existente) |
| `tripas` | 7.1.3.3 | Producao de Tripas Calibradas e Salgadas | Natural Salt Casing Calibrated Production Report | 06 | B (Casings) |
| `fracionamento` | 7.1.3.5 | Acondicionamento de Produtos Industrializados a Base de Proteinas Animais | Industrial Packing Products Report – Meat/Meat Products | 03 | C (Reembalagem) |
| `couro` | 7.1.4.5 | Producao, Venda e Emissao do Certificado Halal – Pele Bovina | Production, Sale and Issue Report – Bovine Skin | 07 | D (Hibrido Prod+Venda) |
| `mucosa` | 7.1.4.6 | Fabricacao de Mucosa – em Tanques | Halal Mucosa Production Report – In Tanks | 03 | E (Log Diario) |
| `heparina_bruta` | 7.1.8.2 | Fabricacao de Heparina Sodica – Etapa Bruta | Manufacturing Products Report – Sodium Heparin – Gross Step | 02 | F (Quimico Animal) |
| `heparina_purificacao` | 7.1.8.3 | Fabricacao de Heparina Sodica – Purificacao | Manufacturing Products Report – Sodium Heparin – Purification | 02 | F (Quimico Animal) |
| `raspa` | 7.1.8.5 | Fabricacao de Couro/Raspa/Apara | Manufacturing Report – Leather/Splitted/Trimmings | 06 | G (Multi-produto Checkbox) |
| `gelatina` | 7.1.8.6 | Fabricacao de Gelatina | Manufacturing Products Report – Gelatine | 01 | F (Quimico Animal) |

### 2.2 Embarque — 9 Tipos (6 novos + 3 existentes)

| Tipo | FM | Titulo PT | Titulo EN | Rev | Padrao |
|------|-----|-----------|-----------|-----|--------|
| `exportacao` | 7.1.7.1 | Embarque Exportacao de Carne/Produtos Carneos | Export Boarding Report – Meat/Meat Products | 04 | A (existente) |
| `exportacao_industrializados` | 7.1.7.2 | Embarque de Produtos Industrializados | Boarding Report – Industrialized Products | 03 | A (Export) |
| `transferencia` | 7.1.7.3 | Transferencia Halal – Mesmo Grupo | Halal Transfer Report – Same Group | 05 | C (existente) |
| `venda_interna` | 7.1.7.4 | Venda para Mercado Interno – Carne/Carneos | Sales for Internal Market – Meat/Meat Products | 05 | B (existente) |
| `venda_industrializados` | 7.1.7.5 | Venda Mercado Interno – Produtos sem Carneos | Sales for Internal Market – Products Without Meat | 04 | B (Venda) |
| `transferencia_industrializados` | 7.1.7.9 | Transferencia Halal – Exclusivo Searas/JBS Aves | Halal Transfer Report – Exclusive Seara/JBS | 02 | C (Transfer) |
| `venda_subprodutos` | 7.1.7.12 | Venda Mercado Interno de Mucosa/Timo/Pancreas/Placenta/Figado | Sales of Mucosa/Thymus/Pancreas/Placenta/Liver | 05 | D (Venda Simplificada) |
| `transferencia_in_natura` | 7.1.4.8.B | Transferencia Halal – Projeto Genu-In e Novapron | Halal Transfer Report – Genu-In and Novapron | 02 | C (Transfer) |
| `transferencia_subprodutos` | 7.1.8.4 | Transferencia Halal – Heparina Sodica | Halal Transfer Report – Sodium Heparin | 03 | C (Transfer) |
| `transferencia_generica` | 7.1.8.X | Transferencia Halal Produto | Product Halal Transfer Report | 01 | C (Transfer) |
| `venda_subprod_couro` | 7.1.4.9 | Venda Merc. Interno – Emissao Certificado Halal (Couro/Raspa) | Sales Internal Market – Halal Certificate (Skin/Split) | 03 | B (Venda) |

---

## 3. Analise Estrutural por Padrao

### 3.1 PRODUCAO — 6 Padroes Estruturais

#### Padrao A — "Producao Industrial Padrao" (FM 7.1.3.1 — existente)
**Secoes**: Empresa + Supervisor | Producao (datas, turnos) | Materias-primas carneas (7 colunas) | Ingredientes aprovados (5 colunas) | Produto final (nome, codigo, lote, datas, qtd, volumes, peso) | Verificacoes (2+3=5 itens) | Assinatura
**Usado por**: Somente FM 7.1.3.1

#### Padrao B — "Producao de Tripas" (FM 7.1.3.3)
**Secoes**: Empresa + Supervisor | Producao (datas + Nro Pedido/Invoice) | **Materias-primas TRIPAS** (tabela unica: Tipo Tripa, SIF, Data abate, Nro Bombonas, NF, DCPOA, Cert. Halal/Transf.) | Insumos (3 colunas: produto, fornecedor, validade) | **Produto final** (tipo tripa, codigo, lote, datas fab/val, nro macos, nro bombonas, peso liq, peso bruto) | Verificacoes (6 itens especificos tripas) | Assinatura
**Campos unicos**: Tipo de tripa, Numero de bombonas, Numero de macos, verificacao sobre bombonas Halal e solucoes de hidratacao/salga

#### Padrao C — "Fracionamento/Reembalagem" (FM 7.1.3.5)
**Secoes**: Empresa + Supervisor | **Acondicionamento** (data envase, horarios, turnos) | **Tabela ENTRADA** (produto, unidade+SIF, codigo, data fab, data val, volumes, nro vol, peso liq) | **Tabela SAIDA** (produto+codigo, unidade+SIF embalagem, lote, data fab, data val, volumes, nro vol, peso liq) | Verificacoes (6 itens fracionamento) | Assinatura
**Campos unicos**: Conceito entrada/saida, area de processamento sinalizada, equipamentos nao alternados Halal/nao-Halal

#### Padrao D — "Producao + Venda Hibrida" (FM 7.1.4.5 — Pele Bovina)
**Secoes**: Frigorifico + Supervisor | **Producao E Venda (linha unica)**: data abate, horario inicial/final, qtd animais, conservante (produto+lote), peso liq | **Informacoes da Venda**: data transf, destinatario (empresa/curtume, SIF, municipio/estado), nro pedido, placa caminhao, NF, doc sanitario (CSN/DCPOA) | Verificacoes (4 itens couro) | Assinatura + Disclaimer
**Campos unicos**: Produto unico por formulario (nao tabela), horario producao (inicial/final), identificacao do conservante, dados do curtume destinatario, placa caminhao, disclaimer "documento nao valido como certificado halal"

#### Padrao E — "Log Diario de Producao" (FM 7.1.4.6 — Mucosa em Tanques)
**Secoes**: Frigorifico + Supervisor | **Tabela diaria** (20+ linhas): Data, Inicio producao (hora), Termino producao (hora), Marcacao regua inicio (checkbox X), Marcacao regua termino (checkbox X), Quantidade produzida | **Total** | Verificacoes (2 itens tanque) | Assinatura
**Campos unicos**: Formato log diario (nao evento unico), marcacao de regua, total acumulado, apenas 2 verificacoes (exclusividade tanque + condicoes area)

#### Padrao F — "Producao Quimica de Origem Animal" (FM 7.1.8.2, 7.1.8.3, 7.1.8.6)
**Secoes**: Empresa (com CNPJ, sem SIF obrigatorio) + Supervisor | Producao (identificacao produto, datas, turnos) | **Materias-primas ANIMAIS** (colunas variam por FM) | **Insumos quimicos** (4 colunas: produto, fornecedor, lote, validade) | Produto final (lote, nro analise CQ, qtd/volumes/peso) | Verificacoes (4 itens quimicos) | Assinatura

**Variacao por FM**:

| Campo | FM 7.1.8.2 (Bruta) | FM 7.1.8.3 (Purificacao) | FM 7.1.8.6 (Gelatina) |
|-------|-------|-------|-------|
| Mat-prima colunas | Mat-prima, Frigorifico, SIF, Nro Recebimento, Doc Halal, Qtd(Kg) — 6 cols | Mat-prima, Lote, Nro Analise (CQ), Qtd(Kg) — 4 cols | Mat-prima (lote), Empresa distribuidora — 2 cols |
| Campo unico | — | Etapa do processo | Codigo/Code, Qtd produto descartado |
| Verificacoes | 4 itens padrao | 4 itens padrao | 4 itens (texto ligeiramente diferente sobre fornecedores e NF) |

#### Padrao G — "Producao Multi-produto com Checkbox" (FM 7.1.8.5 — Raspa/Apara)
**Secoes**: Empresa + Supervisor | Producao (identificacao com **checkboxes**: Couro/Raspa/Apara, datas, turnos) | **Materias-primas animais** (6 colunas: mat-prima, frigorifico, SIF, DCPOA, Doc Halal (NF), Qtd) | Insumos (4 colunas padrao) | **Produto final multi-linha** (identificacao produto, lote, volumes, nro vol, peso liq + TOTAL) | Verificacoes (3 itens) | Assinatura
**Campos unicos**: Checkbox tipo de produto, DCPOA como coluna em mat-primas, varias linhas de produto na saida

### 3.2 EMBARQUE — 4 Padroes Estruturais

#### Padrao A — "Exportacao" (FM 7.1.7.1 existente, FM 7.1.7.2)
**Campos**: Data carregamento, Exportador, Unidade produtora, Endereco carregamento, Importador/Consignatario, Tipo transporte (terrestre/aereo/maritimo), Porto embarque, Identificacao navio/caminhao/aerea, Nro conteiner, Porto destino, Pais destino, Nro pedido, Nro CSI, Nro lacre
**Tabela produto**: 10-11 colunas (produto, codigo, lote, [data abate], data prod, data val, volumes, nro vol, peso liq, peso bruto, temp embarque)
**Verificacoes**: 2 itens (conteiner exclusivo Halal + selo garantia por mercado destino)
**Diferenca FM 7.1.7.2 vs 7.1.7.1**: 7.1.7.2 NAO tem "Frigorifico de abate" nem "CSI" e tabela produto nao tem "Data de Abate"

#### Padrao B — "Venda Mercado Interno" (FM 7.1.7.4 existente, FM 7.1.7.5, FM 7.1.4.9)
**Campos**: Data carregamento, Vendedor, Unidade produtora, Endereco carregamento, Cliente, Endereco destino, Tipo transporte, Identificacao caminhao, Nro conteiner, Nro pedido/NF, Nro doc sanitario (CSN/DCPOA), Nro lacre
**Tabela produto**: 10 colunas (produto, codigo, lote, volumes, nro vol, peso liq, peso bruto, data fab, data val, temp embarque)
**Verificacoes**: 2 itens
**Diferencas**:
- FM 7.1.7.5: adiciona "Codigo inscricao INDEA" e "SIF unidade destino" (para nao-carneos)
- FM 7.1.4.9: campos de transferencia (unid produtora/receptora, NF, DCPOA), tabela simplificada (6 cols, sem codigo/peso bruto/temp)

#### Padrao C — "Transferencia" (FM 7.1.7.3 existente, FM 7.1.7.9, FM 7.1.4.8.B, FM 7.1.8.4, FM 7.1.8.X)
**Campos**: Data saida/transferencia, Unidade produtora/origem, Unidade receptora/destino, Nro pedido, Nro lacre, [Nro doc sanitario], [Nro serie relatorio Halal]
**Tabela produto**: Varia de 3 a 7 colunas
**Verificacoes**: 2 itens (somente Halal + identificacao)
**Variacao por FM**:

| FM | Tabela Produto (colunas) | Campos extras |
|----|-------------------------|---------------|
| 7.1.7.3 (existente) | Produto, Data abate, Qtd(kg) — 3 cols | Doc sanitario |
| 7.1.7.9 | Produto, Data abate, Qtd(kg) — 3 cols + Total | NF, Doc sanitario, Nro serie relatorio Halal, nota obrigatoria CSN+NF |
| 7.1.4.8.B | Produto, Lote, Volumes, Nro vol, Data fab, Peso liq — 6 cols | NF, DCPOA |
| 7.1.8.4 | Produto, Lote, Volumes, Nro vol, Data fab, Peso liq — 6 cols | — |
| 7.1.8.X | Produto, Lote, Codigo, Volumes, Nro vol, Data fab, Peso liq — 7 cols | — |

#### Padrao D — "Venda Simplificada Subprodutos" (FM 7.1.7.12)
**Campos**: Data carregamento, Vendedor, Frigorifico abate, Unidade produtora, Endereco carregamento, Cliente, Endereco destino, Placa caminhao, Nro pedido/NF, Nro CSN/DCPOA, Nro lacre
**Produto**: NAO E TABELA — campos individuais: Datas de abate (range), Data validade (range), Produto comercializado, Quantidade (Kg)
**Verificacoes**: 2 itens (produto Halal no caminhao + lacre tanque = lacre relatorio anterior — somente mucosa)
**Nota obrigatoria**: Referencia FM 7.1.4.6 / FM 7.1.4.4

---

## 4. Arquitetura Tecnica Revisada

### 4.1 Decisao: Discriminador + JSON (confirmada, ampliada)

A arquitetura Discriminador + JSON continua valida, mas a complexidade e **maior** do que o previsto. Os campos compartilhados (base) sao **poucos** — a maioria da logica esta no JSON.

### 4.2 Schema Prisma — Producao

```prisma
enum ProductionType {
  fabricacao           // FM 7.1.3.1 (existente)
  tripas               // FM 7.1.3.3
  fracionamento        // FM 7.1.3.5
  couro                // FM 7.1.4.5
  mucosa               // FM 7.1.4.6
  heparina_bruta       // FM 7.1.8.2
  heparina_purificacao // FM 7.1.8.3
  raspa                // FM 7.1.8.5
  gelatina             // FM 7.1.8.6
}

model ProductionReport {
  // ... campos existentes ...
  productionType  ProductionType  @default(fabricacao)
  customFields    Json?           // campos especificos do tipo
}
```

**Campos BASE (compartilhados)**:
- plantId, supervisorId, date, productionStart, productionEnd, shiftsCount
- status, signedById, signedAt, signatureHash, serialNumber
- verificationItems (Json — itens variam por tipo)
- staff (ReportStaff)

**customFields por tipo** — exemplos:

```typescript
// FM 7.1.3.3 — Tripas
{
  orderNumber: string,
  casingType: string,
  rawCasings: [{ sif, slaughterDate, drumsCount, invoiceNumber, dcpoa, halalCertNumber }],
  inputs: [{ product, supplier, expiryDate }],
  finalProducts: [{ casingType, code, batch, mfgDate, expiryDate, bundlesCount, drumsCount, netWeight, grossWeight }]
}

// FM 7.1.3.5 — Fracionamento
{
  packingDate: string,
  inputProducts: [{ product, productionUnit, sif, code, mfgDate, expiryDate, packageType, packageCount, netWeight }],
  outputProducts: [{ product, code, productionUnit, sif, batch, mfgDate, expiryDate, packageType, packageCount, netWeight }]
}

// FM 7.1.4.5 — Couro (Hibrido)
{
  slaughterDate: string,
  productionTimeStart: string,
  productionTimeEnd: string,
  halalAnimalsCount: number,
  preservativeProduct: string,
  preservativeBatch: string,
  netWeightKg: number,
  // Sale section
  transferDate: string,
  recipient: { company, sif, city, state },
  orderNumber: string,
  truckPlate: string,
  invoiceNumber: string,
  sanitaryDocNumber: string
}

// FM 7.1.4.6 — Mucosa (Log Diario)
{
  dailyLog: [{
    date: string,
    startTime: string,
    endTime: string,
    rulerStartMarked: boolean,
    rulerEndMarked: boolean,
    quantityProduced: number
  }],
  totalProduced: number
}

// FM 7.1.8.2 — Heparina Bruta
{
  productIdentification: string,
  animalRawMaterials: [{ rawMaterial, slaughterhouse, sif, receivingNumber, halalDocument, quantityKg }],
  inputs: [{ product, supplier, batch, expiryDate }],
  finalProduct: { batch, analysisNumber, packageType, packageCount, netWeight }
}

// FM 7.1.8.3 — Heparina Purificacao
{
  productIdentification: string,
  processStep: string,
  rawMaterials: [{ rawMaterial, batch, analysisNumber, quantityKg }],
  inputs: [{ product, supplier, batch, expiryDate }],
  finalProduct: { batch, analysisNumber, packageType, packageCount, netWeight }
}

// FM 7.1.8.5 — Raspa/Apara
{
  productTypes: string[], // ['raspa', 'apara'] checkbox
  animalRawMaterials: [{ rawMaterial, slaughterhouse, sif, dcpoa, halalDocument, quantityKg }],
  inputs: [{ product, supplier, batch, expiryDate }],
  finalProducts: [{ productIdentification, batch, packageType, packageCount, netWeight }],
  totalNetWeight: number
}

// FM 7.1.8.6 — Gelatina
{
  productIdentification: string,
  productCode: string,
  discardedQuantity: number,
  rawMaterials: [{ lotNumber, distributor }],
  finalProduct: { batch, packageType, packageCount, netWeight }
}
```

### 4.3 Schema Prisma — Embarque

```prisma
enum ShippingType {
  // Existentes
  exportacao                     // FM 7.1.7.1
  venda_interna                  // FM 7.1.7.4
  transferencia                  // FM 7.1.7.3
  // Novos
  exportacao_industrializados    // FM 7.1.7.2
  venda_industrializados         // FM 7.1.7.5
  transferencia_industrializados // FM 7.1.7.9
  venda_subprodutos              // FM 7.1.7.12
  transferencia_in_natura        // FM 7.1.4.8.B
  transferencia_subprodutos      // FM 7.1.8.4
  transferencia_generica         // FM 7.1.8.X
  venda_subprod_couro            // FM 7.1.4.9
}
```

**Embarque usa mesma logica**: campos base compartilhados + `customFields` Json para campos especificos.

**Campos extras por tipo**:

| Tipo | Campos extras em customFields |
|------|------------------------------|
| `exportacao_industrializados` | Mesma estrutura que exportacao, sem frigorifico de abate, sem CSI |
| `venda_industrializados` | +indeaCode, +destinationSif |
| `transferencia_industrializados` | +halalSerialNumber, tabela produto simplificada (3 cols) |
| `venda_subprodutos` | Produto nao-tabela: slaughterDateRange, expiryDateRange, soldProduct, quantityKg, truckPlate. Verificacao especifica mucosa (lacre tanque) |
| `transferencia_in_natura` | Tabela 6 cols (sem codigo, sem peso bruto, sem temp) |
| `transferencia_subprodutos` | Tabela 6 cols padrao transferencia |
| `transferencia_generica` | Tabela 7 cols (com codigo) |
| `venda_subprod_couro` | Similar transferencia com NF+DCPOA, tabela 6 cols |

### 4.4 Verificacoes por Tipo (JSON no campo verificationItems)

Cada tipo de producao/embarque tem sua propria lista de verificacoes. O campo `verificationItems` ja e JSON — basta popular com os itens corretos por tipo.

**Producao — verificacoes por tipo**:

| Tipo | Qtd | Verificacoes especificas |
|------|-----|------------------------|
| fabricacao | 5 | 2 antes/durante + 3 antes/durante/apos (existente) |
| tripas | 6 | Bombonas Halal, equipamentos, materia-prima, solucoes hidratacao/salga, armazenamento, identificacao selo |
| fracionamento | 6 | Salas/mesas, estoque isolado, areas sinalizadas, equipamentos nao alternados, armazenamento, selo |
| couro | 4 | Produtos do abate halal, areas processamento, caminhoes limpos, agua limpeza trocada |
| mucosa | 2 | Mucosa do tanque exclusiva abate Halal, condicoes area processamento |
| heparina_bruta | 4 | Equipamentos limpos, mat-prima aprovada, armazenados segregados, identificados como Halal |
| heparina_purificacao | 4 | Idem heparina_bruta |
| raspa | 3 | Linha producao higiene, mat-prima aprovada, armazenados segregados |
| gelatina | 4 | Equipamentos limpos, mat-prima+fornecedores, validade verificada com NF, armazenados segregados |

**Embarque — verificacoes por tipo**: Todos usam 2 itens padrao (variando texto conforme tipo export/transfer/venda).

### 4.5 Metadata dos FMs (constante no frontend)

```typescript
// fm-metadata.ts — configuracao de cada FM
export const FM_METADATA: Record<string, FmConfig> = {
  fabricacao: {
    formNumber: 'FM 7.1.3.1', revision: '04', revisionDate: '06/06/2022',
    titlePt: 'RELATÓRIO DE ACOMPANHAMENTO DE FABRICAÇÃO DE PRODUTOS INDUSTRIALIZADOS A BASE DE CARNE',
    titleEn: 'MANUFACTURING INDUSTRIALIZED PRODUCTS REPORT – CONTAIN MEAT',
    pattern: 'standard',
  },
  tripas: {
    formNumber: 'FM 7.1.3.3', revision: '06', revisionDate: '15/07/2021',
    titlePt: 'RELATÓRIO DE PRODUÇÃO DE TRIPAS CALIBRADAS E SALGADAS',
    titleEn: 'NATURAL SALT CASING CALIBRATED PRODUCTION REPORT',
    pattern: 'casings',
  },
  // ... (todos os 17 tipos)
};
```

---

## 5. Plano de Implementacao (Revisado)

### Fase A.2.1 — Backend: Schema + Migration (1 sprint)

- [ ] Adicionar `ProductionType` enum ao schema Prisma
- [ ] Expandir `ShippingType` enum (8 novos valores)
- [ ] Adicionar coluna `productionType` com default `fabricacao`
- [ ] Adicionar coluna `customFields Json?` em ProductionReport
- [ ] Adicionar coluna `customFields Json?` em ShippingReport (se ainda nao existir)
- [ ] Criar migration `add_production_type_and_custom_fields`
- [ ] Atualizar DTOs de criacao/edicao (productionType + customFields)
- [ ] Atualizar validacao condicional por tipo nos services

### Fase A.2.2 — Backend: FM Metadata + Verificacoes (1 sprint)

- [ ] Criar `fm-metadata.ts` com constantes de todos os FMs (titulo, revisao, verificacoes)
- [ ] Criar endpoint `GET /fm-metadata` para o frontend consultar
- [ ] Criar helpers de validacao por tipo (campos obrigatorios por customFields schema)
- [ ] Atualizar `verificationItems` para receber itens pre-populados por tipo
- [ ] Testes unitarios de validacao por tipo

### Fase A.2.3 — Frontend: Formularios Condicionais Producao (2 sprints)

- [ ] Criar seletor de `productionType` no ProductionReportForm
- [ ] **Padrao B (Tripas)**: Formulario com tabela casings + bombonas
- [ ] **Padrao C (Fracionamento)**: Formulario com tabela entrada + saida
- [ ] **Padrao D (Couro)**: Formulario hibrido producao + venda
- [ ] **Padrao E (Mucosa)**: Formulario de log diario com linhas dinamicas
- [ ] **Padrao F (Heparina/Gelatina)**: 3 variantes com mat-primas animais + insumos quimicos
- [ ] **Padrao G (Raspa/Apara)**: Formulario com checkbox + multi-produto
- [ ] Verificacoes pre-populadas por tipo (C/NC checklist dinamica)

### Fase A.2.4 — Frontend: Formularios Condicionais Embarque (1 sprint)

- [ ] Expandir seletor de `shippingType` no ShippingReportForm
- [ ] **Exportacao Industrializados**: Sem frigorifico abate, sem CSI
- [ ] **Venda Industrializados**: Com INDEA code + SIF destino
- [ ] **Transferencia Industrializados**: Tabela simplificada 3 cols + serial Halal
- [ ] **Venda Subprodutos**: Produto unico (nao tabela) + verificacao tanque
- [ ] **Transferencias (in natura, subprodutos, generica)**: Tabela 6-7 cols
- [ ] **Venda Subprod Couro**: Campos transferencia + tabela 6 cols

### Fase A.2.5 — Sidebar Expandida + Routing (1 sprint)

- [ ] Novo layout de sidebar com 4 grupos:
  ```
  In Natura
    Abate
    Emb. Exportacao
    Venda Merc. Int.
    Transferencia
    Transf. In Natura (NOVO)
  Industrializados
    Fabricacao
    Tripas (NOVO)
    Fracionamento (NOVO)
    Emb. Export. Ind. (NOVO)
    Venda Ind. (NOVO)
    Transf. Ind. (NOVO)
  Subprodutos
    Couro (NOVO)
    Mucosa (NOVO)
    Heparina Bruta (NOVO)
    Heparina Purif. (NOVO)
    Raspa/Aparas (NOVO)
    Gelatina (NOVO)
    Venda Subprod. (NOVO)
    Transf. Subprod. (NOVO)
    Transf. Generica (NOVO)
    Venda Couro (NOVO)
  Gestao
    Dashboard
    Analytics
    Colaboradores
    Escalas
    Usuarios
    Plantas
    Nao-Conformidades
  ```
- [ ] Rotas com filtro por tipo (`/production?type=tripas`, etc.)
- [ ] Listagens filtradas por productionType/shippingType

### Fase A.2.6 — PDF Templates (2 sprints)

- [ ] Template PDF para cada padrao de producao (6 padroes)
- [ ] Template PDF para cada padrao de embarque (3 padroes com variantes)
- [ ] Reutilizar `pdf-helpers.ts` (cabecalho FAMBRAS, staffSection, signatureBlock)
- [ ] Disclaimer "documento nao valido como certificado halal" onde aplicavel
- [ ] Nota de confidencialidade onde aplicavel
- [ ] Testes: gerar PDFs de exemplo para cada tipo

### Fase A.2.7 — Seed + Testes + Build (1 sprint)

- [ ] Seed com exemplos de cada tipo de producao (8 tipos)
- [ ] Seed com exemplos de cada tipo de embarque (11 tipos)
- [ ] Build backend + frontend sem erros
- [ ] Revisao final de formularios vs FMs originais

---

## 6. Estimativa de Esforco

| Fase | Descricao | Complexidade |
|------|-----------|-------------|
| A.2.1 | Schema + Migration | Baixa |
| A.2.2 | FM Metadata + Validacoes | Media |
| A.2.3 | Frontend Producao (7 formularios) | **Alta** — 6 padroes distintos |
| A.2.4 | Frontend Embarque (8 formularios) | Media — variantes do base |
| A.2.5 | Sidebar + Routing | Media |
| A.2.6 | PDF Templates (17 FMs) | **Alta** — cada FM com layout proprio |
| A.2.7 | Seed + Testes + Build | Baixa |

**Maior risco**: Padroes D (couro hibrido) e E (mucosa log diario) que fogem completamente do modelo "relatorio de producao" padrao.

---

## 7. Dependencias e Decisoes Pendentes

| # | Decisao | Opcoes | Recomendacao |
|---|---------|--------|-------------|
| 1 | FM 7.1.8.3 (Heparina Purificacao) — incluir como tipo separado ou subtipo de heparina? | Tipo separado vs campo "etapa" | **Tipo separado** — colunas de mat-prima sao diferentes |
| 2 | FM 7.1.4.5 (Couro) e producao ou embarque? | ProductionReport vs modelo hibrido | **ProductionReport** com customFields incluindo secao de venda |
| 3 | FM 7.1.4.9 vs FM 7.1.7.5 — sao 2 tipos ou 1? | Separar vs unificar | **Separar** — FM 7.1.4.9 e especifico para couro/raspa com campos DCPOA |
| 4 | FM 7.1.8.X (Transferencia Generica) — manter catch-all? | Tipo especifico vs generico | **Manter** — e usado quando nenhum outro FM especifico se aplica |
| 5 | Sidebar com 20+ itens — colapsar por default? | Tudo aberto vs colapsado | **Colapsado** — abrir grupo conforme role/planta do usuario |

---

## 8. Referencia: Documentos FM Originais Analisados

Localizados em `C:\SIH\Relatorios Preenchidos Industrializados\`:

| FM | Arquivo | Status Analise |
|----|---------|---------------|
| 7.1.3.3 | FM 7.1.3.3 - REL ACOMP FAB PROD TRIPAS.pdf | Completa |
| 7.1.3.5 | FM 7.1.3.5 - RELATORIO FRACIONAMENTO CORNED BEEF.pdf | Completa |
| 7.1.4.5 | FM 7.1.4.5 - PRODUCAO E EMBARQUE COURO VERDE.pdf | Completa |
| 7.1.4.6 | FM 7.1.4.6 - PRODUCAO MUCOSA.pdf | Completa |
| 7.1.4.8.B | FM 7.1.4.8.B - RELATORIO DE TRANFERENCIA RASPA E APARAS.pdf | Completa |
| 7.1.4.9 | FM 7.1.4.9 - EMBARQUE RASPA - APARAS.pdf | Completa |
| 7.1.7.2 | FM_7.1.7.2_-_REL_EMBARQUE_ GELATINA - COLAGENO.pdf | Completa |
| 7.1.7.5 | FM 7.1.7.5 - REL EMBARQUE RASPA - APARAS.pdf | Completa |
| 7.1.7.9 | FM 7.1.7.9 - RELATORIO DE TRANFERENCIA CARNE.pdf | Completa |
| 7.1.7.12 | FM 7.1.7.12 - RELATORIO DE EMBARQUE MUCOSA.pdf | Completa |
| 7.1.8.2 | FM 7.1.8.2 - RELATORIO DE PRODUCAO HEPARINA.pdf | Completa |
| 7.1.8.3 | FM 7.1.8.3 RELATORIO PRODUCAO HEPARINA.pdf | Completa |
| 7.1.8.4 | FM 7.1.8.4 - REL. TRANSF. HALAL HEPARINA.pdf | Completa |
| 7.1.8.5 | FM 7.1.8.5 - PRODUCAO RASPA - APARAS.pdf | Completa |
| 7.1.8.6 | FM 7.1.8.6 - REL. ACOMP. FABRICACAO DE GELATINA.pdf | Completa |
| 7.1.8.X | FM 7.1.8.x - REL PRODUCAO E TRANSFERENCIA GELATINA - COLAGENO.pdf | Completa |
