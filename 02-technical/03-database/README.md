---
title: Banco de Dados
parent: Documentação Técnica
nav_order: 3
permalink: /técnico/banco-de-dados/
---

# 3. Banco de Dados

---

## 3.1 Visão Geral

O SIH utiliza **PostgreSQL 16** como banco de dados relacional, gerênciado via **Prisma ORM** com migrações automáticas.

### Extensões Habilitadas

| Extensão | Finalidade |
|----------|-----------|
| `uuid-ossp` | Geração de UUID v4 para chaves primarias (`uuid_generate_v4()`) |
| `pgcrypto` | Funções criptograficas para hashing e encriptação de dados sensiveis |
| `pg_trgm` | Busca por similaridade (trigram) para pesquisa textual apróximada |

### Resumo do Schema

| Item | Quantidade |
|------|-----------|
| Modelos (tabelas) | 10 (7 originais + Collaborator, CollaboratorPlant, ReportStaff) |
| Enums | 11 (10 originais + CollaboratorType) |
| Índices compostos | 18 |
| Constraints unique | 7 |

---

## 3.2 ERD (Diagrama Entidade-Relacionamento)

```
┌─────────────────────────┐
│   SupervisorProfile     │
│   (supervisor_profiles) │
│─────────────────────────│
│ PK id            UUID   │
│    externalUserId       │
│    name                 │
│    email         UNIQUE │
│    role          enum   │
│    phone                │
│    registration         │
│    qualifications JSON  │
│    preferences    JSON  │
│    isActive             │
│    lastLoginAt          │
│    created_at           │
│    updated_at           │
└────────┬────────────────┘
         │
         │ 1:N
         │
         ├──────────────────────────────────────────────────────────────────────┐
         │                                                                      │
         │          ┌──────────────────────────┐                                │
         │          │        Plant              │                                │
         │          │       (plants)            │                                │
         │          │──────────────────────────│                                │
         │          │ PK id             UUID   │                                │
         │          │    externalCompanyId     │                                │
         │          │    name                  │                                │
         │          │    sifCode       UNIQUE  │                                │
         │          │    type           enum   │                                │
         │          │    address         JSON  │                                │
         │          │    contact         JSON  │                                │
         │          │    species      enum[]   │                                │
         │          │    isActive              │                                │
         │          │    created_at            │                                │
         │          │    updated_at            │                                │
         │          └────────┬─────────────────┘                                │
         │                   │                                                  │
         │                   │ 1:N                                              │
         │                   │                                                  │
    ┌────┴───────────────────┴──────────────────────────────────────┐           │
    │                                                               │           │
    │  ┌──────────────────────────────┐  ┌────────────────────────────────┐     │
    │  │     SlaughterReport          │  │     ProductionReport           │     │
    │  │    (slaughter_reports)       │  │    (production_reports)        │     │
    │  │──────────────────────────────│  │────────────────────────────────│     │
    │  │ PK id              UUID     │  │ PK id              UUID       │     │
    │  │    serialNumber    UNIQUE   │  │    serialNumber    UNIQUE     │     │
    │  │    formNumber               │  │    formNumber                 │     │
    │  │ FK plantId         UUID ────│──│ FK plantId         UUID ──────│─┐   │
    │  │ FK supervisorId    UUID ────│──│ FK supervisorId    UUID ──────│─│───┘
    │  │    date            Date     │  │    productionStart            │ │
    │  │    shift           enum     │  │    productionEnd              │ │
    │  │    species          enum    │  │    shiftsCount                │ │
    │  │    totalAnimals             │  │    meatRawMaterials    JSON   │ │
    │  │    approvedAnimals          │  │    productName                │ │
    │  │    rejectedAnimals          │  │    totalProduced      Dec    │ │
    │  │    verificationItems JSON   │  │    verificationItems  JSON   │ │
    │  │    status           enum    │  │    status             enum   │ │
    │  │    created_at               │  │    created_at                │ │
    │  │    updated_at               │  │    updated_at                │ │
    │  └──────────┬──────────────────┘  └──────────┬───────────────────┘ │
    │             │ 1:N                            │ 1:N                 │
    │             │                                │                     │
    │             │    ┌───────────────────────┐    │                     │
    │             │    │    NonConformity      │    │                     │
    │             │    │  (non_conformities)   │    │                     │
    │             │    │───────────────────────│    │                     │
    │             │    │ PK id          UUID   │    │                     │
    │             ├───►│ FK plantId     UUID   │◄───┤                     │
    │             │    │ FK supervisorId UUID  │    │                     │
    │             └───►│ FK slaughterReportId? │    │                     │
    │                  │ FK productionReportId?│◄───┘                     │
    │                  │ FK shippingReportId?  │◄──────┐                  │
    │                  │    description        │       │                  │
    │                  │    severity    enum   │       │                  │
    │                  │    category           │       │                  │
    │                  │    evidence    JSON   │       │                  │
    │                  │    status      enum   │       │                  │
    │                  │    created_at         │       │                  │
    │                  │    updated_at         │       │                  │
    │                  └───────────────────────┘       │                  │
    │                                                  │                  │
    │  ┌──────────────────────────────┐                │                  │
    │  │     ShippingReport           │                │                  │
    │  │    (shipping_reports)        │                │                  │
    │  │──────────────────────────────│                │                  │
    │  │ PK id              UUID     │                │                  │
    │  │    serialNumber    UNIQUE   │                │                  │
    │  │    formNumber               │                │                  │
    │  │    shippingType    enum     │                │                  │
    │  │ FK plantId         UUID ────│────────────────│──────────────────┘
    │  │ FK supervisorId    UUID ────│────────────────┘
    │  │    loadingDate     Date     │
    │  │    transportType   enum     │
    │  │    products         JSON    │
    │  │    verificationItems JSON   │
    │  │    status           enum    │
    │  │    created_at               │
    │  │    updated_at               │
    │  └─────────────────────────────┘
    │
    │  ┌──────────────────────────────┐
    │  │   SupervisorSchedule         │
    │  │  (supervisor_schedules)      │
    │  │──────────────────────────────│
    │  │ PK id              UUID     │
    │  │ FK supervisorId    UUID ────│──► SupervisorProfile
    │  │ FK plantId         UUID ────│──► Plant
    │  │    date            Date     │
    │  │    shift           enum     │
    │  │    type            enum     │
    │  │    notes                    │
    │  │    created_at               │
    │  │    updated_at               │
    │  │                             │
    │  │  UNIQUE(supervisorId,       │
    │  │    plantId, date, shift)    │
    │  └─────────────────────────────┘
    │
    └──────────────────────────────────────────────────────────────────────
```

### Resumo dos Relacionamentos

| Relação | Tipo | FK | Descrição |
|---------|------|-----|-----------|
| SupervisorProfile → SlaughterReport | 1:N | `supervisorId` | Supervisor autor do relatório de abate |
| SupervisorProfile → ProductionReport | 1:N | `supervisorId` | Supervisor autor do relatório de produção |
| SupervisorProfile → ShippingReport | 1:N | `supervisorId` | Supervisor autor do relatório de embarque |
| SupervisorProfile → NonConformity | 1:N | `supervisorId` | Supervisor que registrou a NC |
| SupervisorProfile → SupervisorSchedule | 1:N | `supervisorId` | Escala do supervisor |
| Plant → SlaughterReport | 1:N | `plantId` | Relatórios de abate na planta |
| Plant → ProductionReport | 1:N | `plantId` | Relatórios de produção na planta |
| Plant → ShippingReport | 1:N | `plantId` | Relatórios de embarque na planta |
| Plant → NonConformity | 1:N | `plantId` | NCs registradas na planta |
| Plant → SupervisorSchedule | 1:N | `plantId` | Escalas na planta |
| SlaughterReport → NonConformity | 1:N | `slaughterReportId` | NCs vinculadas a abate (opcional) |
| ProductionReport → NonConformity | 1:N | `productionReportId` | NCs vinculadas a produção (opcional) |
| ShippingReport → NonConformity | 1:N | `shippingReportId` | NCs vinculadas a embarque (opcional) |
| Collaborator → CollaboratorPlant | 1:N | `collaboratorId` | Plantas onde o colaborador atua |
| Plant → CollaboratorPlant | 1:N | `plantId` | Colaboradores vinculados à planta |
| Collaborator → ReportStaff | 1:N | `collaboratorId` | Relatórios onde o colaborador atuou |
| SlaughterReport → ReportStaff | 1:N | `slaughterReportId` | Equipe do relatório de abate |
| ProductionReport → ReportStaff | 1:N | `productionReportId` | Equipe do relatório de produção |
| ShippingReport → ReportStaff | 1:N | `shippingReportId` | Equipe do relatório de embarque |

---

## 3.3 Enums

### UserRole
Papéis de usuário no sistema.

| Valor | Descrição |
|-------|-----------|
| `admin` | Administrador do sistema com acesso total |
| `coordenador` | Coordenador de supervisores: gerência escalas, cria/edita usuários, visualiza relatórios (read-only), cancela relatórios, gerência NCs. NÃO assina relatórios |
| `supervisor` | Supervisor Halal que preenche e assina relatórios nas plantas |
| `operador` | Operador que preenche relatórios mas NÃO pode assinar |
### CollaboratorType
Tipo/função do colaborador operacional (não-usuário).

| Valor | Descrição |
|-------|-----------|
| `degolador` | Profissional que realiza o abate Halal |
| `sheik` | Líder religioso que supervisiona o processo |
| `auxiliar` | Auxiliar de supervisão |
| `veterinario` | Médico veterinário |
| `outro` | Outro tipo de colaborador |

### Species
Especies animais cobertas pela supervisão Halal na v1.0. Apenas bovino e ave possuem formulários FM implementados.

| Valor | Descrição | FM |
|-------|-----------|-----|
| `bovino` | Bovinos (gado) | FM 7.1.4.2 |
| `ave` | Aves (frango, peru) | FM 7.1.4.1 |

> **Nota**: As demais espécies (ovino, caprino, bubalino, equino, peixe) foram removidas do schema na migração `remove_legacy_species` pois não possuem FM específico. Poderão ser adicionadas em versões futuras.

### Shift
Turnos de trabalho na planta industrial.

| Valor | Descrição |
|-------|-----------|
| `matutino` | Turno da manha |
| `vespertino` | Turno da tarde |
| `noturno` | Turno da noite |
| `integral` | Turno integral (dia completo) |

### PlantType
Tipo de planta industrial.

| Valor | Descrição |
|-------|-----------|
| `abatedouro` | Estabelecimento de abate |
| `frigorifico` | Frigorífico (abate + processamento) |
| `laticinio` | Laticinio |
| `processamento` | Unidade de processamento de alimentos |
| `armazenamento` | Unidade de armazenamento a frio |
| `outro` | Outros tipos de estabelecimento |

### ReportStatus
Status do ciclo de vida dos relatórios (Abate, Produção, Embarque). Workflow simplificado: rascunho → assinado (final). NÃO há etapa de aprovação.

| Valor | Descrição |
|-------|-----------|
| `rascunho` | Relatório em edição, não assinado |
| `assinado` | Assinado pelo supervisor (estado final — imutável) |
| `cancelado` | Cancelado pelo coordenador/admin (com motivo obrigatório) |

> **Nota**: Os valores legados (`enviado`, `revisado`, `aprovado`, `rejeitado`, `gestor`) foram removidos do schema Prisma na migração `remove_legacy_enums`.

### ShippingType
Tipo de movimentação de embarque/venda.

| Valor | Descrição |
|-------|-----------|
| `exportacao` | Exportação para mercado externo |
| `venda_interna` | Venda no mercado interno |
| `transferencia` | Transferência entre unidades |

### TransportType
Tipo de transporte utilizado no embarque.

| Valor | Descrição |
|-------|-----------|
| `terrestre` | Transporte rodoviario/ferroviario |
| `aereo` | Transporte aereo |
| `maritimo` | Transporte maritimo |

### Severity
Grau de severidade de uma Não-Conformidade.

| Valor | Descrição |
|-------|-----------|
| `critica` | Risco imediato a conformidade Halal; requer ação imediata |
| `maior` | Desvio significativo de processo; ação em 24h |
| `menor` | Desvio pontual sem impacto direto; ação em 7 dias |
| `observacao` | Ponto de atencao para melhoria continua |

### NCStatus
Status do ciclo de vida de uma Não-Conformidade.

| Valor | Descrição |
|-------|-----------|
| `aberta` | NC registrada, aguardando tratamento |
| `em_tratamento` | Ação corretiva em andamento |
| `resolvida` | Ação corretiva implementada, aguardando verificação |
| `verificada` | Verificado que a ação corretiva foi efetiva |
| `encerrada` | NC encerrada definitivamente |

### ScheduleType
Tipo de alocação na escala de supervisores.

| Valor | Descrição |
|-------|-----------|
| `regular` | Escala regular/planejada |
| `substituicao` | Substituicao de outro supervisor |
| `extra` | Escala adicional/hora extra |
| `folga` | Dia de folga programada |

---

## 3.4 Dicionario de Dados

### 3.4.1 SupervisorProfile (`supervisor_profiles`)

Perfil do usuário no sistema. Na v1.0, a autenticação e self-contained (bcrypt + JWT HS256). O campo `passwordHash` armazena o hash bcrypt da senha. O campo `externalUserId` está preparado para integração futura com HalalSphere.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `externalUserId` | String | Não | - | ID externo (preparado para integração com HalalSphere; na v1.0, igual ao `id`) |
| `name` | String | Não | - | Nome completo do supervisor |
| `email` | String | Não | - | Email do supervisor (usado para login) |
| `role` | UserRole | Não | `supervisor` | Papel do usuário no sistema |
| `phone` | String | Sim | - | Telefone de contato |
| `registration` | String | Sim | - | Número de registro profissional / matricula |
| `qualifications` | Json | Sim | - | Qualificações e certificações (formato livre JSON) |
| `preferences` | Json | Sim | - | Preferências do usuário (tema, notificações, idioma, etc.) |
| `isActive` | Boolean | Não | `true` | Se o supervisor está ativo no sistema |
| `lastLoginAt` | DateTime | Sim | - | Data/hora do último login |
| `created_at` | DateTime | Não | `now()` | Data de criação do registro |
| `updated_at` | DateTime | Não | auto | Data da última atualização (gerênciado pelo Prisma) |

**Constraints:**
- `UNIQUE` em `externalUserId`
- `UNIQUE` em `email`

**Índices:** Nenhum índice composto adicional (índices únicos já cobrem as buscas por `externalUserId` e `email`).

---

### 3.4.2 Plant (`plants`)

Planta industrial (abatedouro, frigorífico, etc.) onde a supervisão Halal e realizada.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `externalCompanyId` | String | Sim | - | ID da empresa no sistema externo (ERP/cadastro) |
| `name` | String | Não | - | Nome da planta industrial |
| `sifCode` | String | Não | - | Código SIF (Serviço de Inspecao Federal) da planta |
| `type` | PlantType | Não | - | Tipo de estabelecimento |
| `address` | Json | Sim | - | Endereço completo (logradouro, cidade, estado, CEP, coordenadas) |
| `contact` | Json | Sim | - | Dados de contato (telefone, email, responsável) |
| `species` | Species[] | Não | - | Lista de especies abatidas/processadas na planta |
| `isActive` | Boolean | Não | `true` | Se a planta está ativa no sistema |
| `created_at` | DateTime | Não | `now()` | Data de criação do registro |
| `updated_at` | DateTime | Não | auto | Data da última atualização |

**Constraints:**
- `UNIQUE` em `sifCode`

**Índices:** Nenhum índice composto adicional (índice único em `sifCode` cobre a busca principal).

---

### 3.4.3 SlaughterReport (`slaughter_reports`)

Relatório de acompanhamento de abate Halal (FM 7.1.4.1 para aves, FM 7.1.4.2 para bovinos). Um único modelo cobre todas as especies, diferenciado pelo campo `species`. Bovinos possuem seção adicional de insensibilização.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `serialNumber` | String | Não | - | Número serial único (formato: SIF/ANO/SEQ) |
| `formNumber` | String | Não | - | Número do formulário FM (ex: "FM 7.1.4.1") |
| `plantId` | UUID | Não | - | FK para Plant - planta onde ocorreu o abate |
| `supervisorId` | UUID | Não | - | FK para SupervisorProfile - supervisor responsável |
| `date` | Daté | Não | - | Data do abate |
| `shift` | Shift | Não | - | Turno do abate |
| `species` | Species | Não | - | Especie abatida |
| `slaughtererName` | String | Sim | - | Nome do degolador/abatedor |
| `slaughtererDoc` | String | Sim | - | Documento de identificação do degolador |
| `totalAnimals` | Int | Não | - | Total de animais abatidos |
| `approvedAnimals` | Int | Não | - | Quantidade de animais aprovados |
| `rejectedAnimals` | Int | Não | `0` | Quantidade de animais rejeitados |
| `rejectedSequence` | String | Sim | - | Sequencia numerica dos animais rejeitados |
| `stunningUsed` | Boolean | Sim | - | Se insensibilização foi utilizada (bovinos) |
| `stunningEvaluation` | String | Sim | - | Avaliação da pressão de insensibilização |
| `stunningVerifications` | Json | Sim | - | Verificações de insensibilização (2 por turno) |
| `coolingCarcasses` | Int | Sim | - | Quantidade de carcaças em resfriamento |
| `coolingCameras` | String | Sim | - | Identificação das camaras de resfriamento |
| `byproductsSold` | Boolean | Sim | - | Se houve venda de subprodutos |
| `byproductsDescription` | String | Sim | - | Descrição dos subprodutos vendidos |
| `verificationItems` | Json | Não | - | 14 itens de verificação C/NC (conformidade) |
| `observations` | String | Sim | - | Observações gerais |
| `status` | ReportStatus | Não | `rascunho` | Status atual do relatório |
| `cancelledReason` | String | Sim | - | Motivo do cancelamento (quando status=cancelado) |
| `replacedBy` | String | Sim | - | Serial do relatório substituto |
| `reviewedBy` | UUID | Sim | - | ID do coordenador que revisou |
| `reviewedAt` | DateTime | Sim | - | Data/hora da revisão |
| `created_at` | DateTime | Não | `now()` | Data de criação do registro |
| `updated_at` | DateTime | Não | auto | Data da última atualização |

**Constraints:**
- `UNIQUE` em `serialNumber`

**Índices:**
- `(plantId, date)` - Busca de relatórios por planta e data
- `(supervisorId, date)` - Busca de relatórios por supervisor e data
- `(status)` - Filtragem por status do relatório

---

### 3.4.4 ProductionReport (`production_reports`)

Relatório de acompanhamento de produção de industrializados Halal (FM 7.1.3.1) e produção especial (FM 7.1.8.x). Rastreia matérias-primas, ingredientes e dados do produto final.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `serialNumber` | String | Não | - | Número serial único (formato: SIF/ANO/SEQ) |
| `formNumber` | String | Não | - | Número do formulário FM (ex: "FM 7.1.3.1") |
| `plantId` | UUID | Não | - | FK para Plant - planta onde ocorreu a produção |
| `supervisorId` | UUID | Não | - | FK para SupervisorProfile - supervisor responsável |
| `productionStart` | DateTime | Não | - | Data/hora de início da produção |
| `productionEnd` | DateTime | Não | - | Data/hora de fim da produção |
| `shiftsCount` | Int | Não | - | Número de turnos utilizados na produção |
| `meatRawMaterials` | Json | Não | - | Tabela de matérias-primas cárneas (frigorífico, SIF, data abate, CSN, certificado Halal) |
| `approvedIngredients` | Json | Sim | - | Tabela de ingredientes aprovados não-carneos (fornecedor, lote, válidade) |
| `productName` | String | Não | - | Nome do produto final |
| `productCode` | String | Sim | - | Código do produto |
| `productBatch` | String | Sim | - | Número do lote do produto |
| `manufacturingDate` | Daté | Sim | - | Data de fabricação |
| `expiryDate` | Daté | Sim | - | Data de válidade |
| `totalProduced` | Decimal(12,3) | Sim | - | Quantidade total produzida (kg ou unidades) |
| `packageType` | String | Sim | - | Tipo de embalagem |
| `packageCount` | Int | Sim | - | Quantidade de embalagens |
| `netWeightKg` | Decimal(12,3) | Sim | - | Peso liquido total em kg |
| `grossWeightKg` | Decimal(12,3) | Sim | - | Peso bruto total em kg |
| `verificationItems` | Json | Não | - | 5 itens de verificação C/NC (conformidade) |
| `observations` | String | Sim | - | Observações gerais |
| `isSpecialProduction` | Boolean | Não | `false` | Flag para produção especial (FM 7.1.8.x) |
| `status` | ReportStatus | Não | `rascunho` | Status atual do relatório |
| `cancelledReason` | String | Sim | - | Motivo do cancelamento |
| `replacedBy` | String | Sim | - | Serial do relatório substituto |
| `reviewedBy` | UUID | Sim | - | ID do coordenador que revisou |
| `reviewedAt` | DateTime | Sim | - | Data/hora da revisão |
| `created_at` | DateTime | Não | `now()` | Data de criação do registro |
| `updated_at` | DateTime | Não | auto | Data da última atualização |

**Constraints:**
- `UNIQUE` em `serialNumber`

**Índices:**
- `(plantId, productionStart)` - Busca de relatórios por planta e período de produção
- `(supervisorId, productionStart)` - Busca de relatórios por supervisor e período
- `(status)` - Filtragem por status do relatório

---

### 3.4.5 ShippingReport (`shipping_reports`)

Relatório de embarque, venda ou transferência de produtos Halal. Unifica 3 tipos de movimentação (FM 7.1.7.1 Exportação, FM 7.1.7.4 Venda Interna, DCPOA Transferência) em um único modelo diferenciado por `shippingType`.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `serialNumber` | String | Não | - | Número serial único (formato: SIF/ANO/SEQ) |
| `formNumber` | String | Não | - | Número do formulário FM |
| `shippingType` | ShippingType | Não | - | Tipo de movimentação (exportação, venda_interna, transferência) |
| `plantId` | UUID | Não | - | FK para Plant - planta de origem |
| `supervisorId` | UUID | Não | - | FK para SupervisorProfile - supervisor responsável |
| `loadingDate` | Daté | Não | - | Data do carregamento |
| `exporter` | String | Sim | - | Razao social do exportador (exportação) |
| `slaughterhouseInfo` | String | Sim | - | Informações do abatedouro de origem |
| `productionUnitInfo` | String | Sim | - | Informações da unidade de produção |
| `loadingAddress` | String | Sim | - | Endereço de carregamento |
| `importer` | String | Sim | - | Razao social do importador (exportação) |
| `transportType` | TransportType | Sim | - | Tipo de transporte (terrestre, aereo, maritimo) |
| `embarkationPort` | String | Sim | - | Porto/ponto de embarque (exportação) |
| `vehicleId` | String | Sim | - | Identificação do veiculo (placa, número voo, etc.) |
| `containerNumber` | String | Sim | - | Número do container |
| `destinationPort` | String | Sim | - | Porto de destino (exportação) |
| `destinationCountry` | String | Sim | - | País de destino (exportação) |
| `orderNumber` | String | Sim | - | Número do pedido/ordem de compra |
| `csiNumber` | String | Sim | - | Número do CSI (Certificado Sanitario Internacional) |
| `sealNumber` | String | Sim | - | Número do lacre |
| `products` | Json | Não | - | Lista de produtos embarcados (produto, código, lote, datas, pesos, temperatura) |
| `verificationItems` | Json | Não | - | 2 itens de verificação C/NC (exclusividade Halal no container + selo) |
| `observations` | String | Sim | - | Observações gerais |
| `status` | ReportStatus | Não | `rascunho` | Status atual do relatório |
| `cancelledReason` | String | Sim | - | Motivo do cancelamento |
| `replacedBy` | String | Sim | - | Serial do relatório substituto |
| `reviewedBy` | UUID | Sim | - | ID do coordenador que revisou |
| `reviewedAt` | DateTime | Sim | - | Data/hora da revisão |
| `created_at` | DateTime | Não | `now()` | Data de criação do registro |
| `updated_at` | DateTime | Não | auto | Data da última atualização |

**Constraints:**
- `UNIQUE` em `serialNumber`

**Índices:**
- `(plantId, loadingDate)` - Busca de relatórios por planta e data de carregamento
- `(supervisorId, loadingDate)` - Busca de relatórios por supervisor e data
- `(shippingType, status)` - Filtragem por tipo de embarque e status

---

### 3.4.6 NonConformity (`non_conformities`)

Registro de não-conformidade identificada durante a supervisão Halal (FM 7.1.6.1). Pode ser vinculada a qualquer tipo de relatório (abate, produção, embarque) ou registrada de forma avulsa. Implementa o prazo de 7 dias corridos conforme PR 7.1.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `plantId` | UUID | Não | - | FK para Plant - planta onde a NC foi identificada |
| `supervisorId` | UUID | Não | - | FK para SupervisorProfile - supervisor que registrou |
| `slaughterReportId` | UUID | Sim | - | FK para SlaughterReport - vínculo opcional a relatório de abate |
| `productionReportId` | UUID | Sim | - | FK para ProductionReport - vínculo opcional a relatório de produção |
| `shippingReportId` | UUID | Sim | - | FK para ShippingReport - vínculo opcional a relatório de embarque |
| `description` | String | Não | - | Descrição detalhada da não-conformidade |
| `severity` | Severity | Não | - | Grau de severidade (crítica, maior, menor, observação) |
| `category` | String | Não | - | Categoria da NC (higiene, processo, equipamento, materia-prima, rotulagem, etc.) |
| `evidence` | Json | Sim | - | Evidencias (fotos, descricoes, referências documentais) |
| `correctiveAction` | String | Sim | - | Descrição da ação corretiva implementada |
| `preventiveAction` | String | Sim | - | Descrição da ação preventiva proposta |
| `deadline` | DateTime | Sim | - | Prazo para resolução (padrão: 7 dias corridos) |
| `resolvedAt` | DateTime | Sim | - | Data/hora em que a NC foi resolvida |
| `resolvedBy` | UUID | Sim | - | ID de quem registrou a resolução |
| `verifiedAt` | DateTime | Sim | - | Data/hora da verificação da resolução |
| `verifiedBy` | UUID | Sim | - | ID de quem verificou a resolução |
| `status` | NCStatus | Não | `aberta` | Status atual da NC |
| `created_at` | DateTime | Não | `now()` | Data de criação do registro |
| `updated_at` | DateTime | Não | auto | Data da última atualização |

**Constraints:** Nenhuma constraint unique adicional.

**Índices:**
- `(plantId, status)` - Busca de NCs por planta e status
- `(severity, status)` - Busca de NCs por severidade e status
- `(supervisorId)` - Busca de NCs por supervisor

---

### 3.4.7 SupervisorSchedule (`supervisor_schedules`)

Escala de trabalho dos supervisores nas plantas industriais. Permite ao coordenador alocar supervisores por planta, turno e dia.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `supervisorId` | UUID | Não | - | FK para SupervisorProfile - supervisor alocado |
| `plantId` | UUID | Não | - | FK para Plant - planta de alocação |
| `date` | Daté | Não | - | Data da escala |
| `shift` | Shift | Não | - | Turno alocado |
| `type` | ScheduleType | Não | `regular` | Tipo de alocação (regular, substituicao, extra, folga) |
| `notes` | String | Sim | - | Observações sobre a alocação |
| `created_at` | DateTime | Não | `now()` | Data de criação do registro |
| `updated_at` | DateTime | Não | auto | Data da última atualização |

**Constraints:**
- `UNIQUE` composto em `(supervisorId, plantId, date, shift)` - Garante que um supervisor não pode ter duplicidade de alocação na mesma planta, data e turno

**Índices:**
- `(supervisorId, date)` - Busca de escala por supervisor e data
- `(plantId, date)` - Busca de escala por planta e data

---

### 3.4.8 Collaborator (`collaborators`) — Implementado

Cadastro de colaboradores operacionais que atuam nas plantas mas não são usuários do sistema (degoladores, sheiks, auxiliares, veterinários). Campo `externalId` preparado para integração futura com HalalSphere.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `name` | String | Não | - | Nome completo do colaborador |
| `document` | String | Sim | - | Documento de identificação (RG, CPF ou passaporte) |
| `type` | CollaboratorType | Não | - | Tipo: degolador, sheik, auxiliar, veterinario, outro |
| `phone` | String | Sim | - | Telefone de contato |
| `photoUrl` | String | Sim | - | URL/path da foto do colaborador |
| `externalId` | String | Sim | - | ID no sistema externo (HalalSphere) para integração futura |
| `isActive` | Boolean | Não | `true` | Se o colaborador está ativo |
| `created_at` | DateTime | Não | `now()` | Data de criação do registro |
| `updated_at` | DateTime | Não | auto | Data da última atualização |

**Índices:**
- `(type, isActive)` — Busca por tipo de colaborador e status

**Endpoints:**
- CRUD completo via `GET/POST/PATCH /collaborators`
- Upload foto: `POST /collaborators/:id/photo`
- Vinculação plantas: `POST/DELETE /collaborators/:id/plants`

---

### 3.4.9 CollaboratorPlant (`collaborator_plants`) — Implementado

Tabela de ligação N:N entre colaboradores e plantas onde atuam.

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `collaboratorId` | UUID | Não | - | FK para Collaborator (CASCADE) |
| `plantId` | UUID | Não | - | FK para Plant (CASCADE) |
| `assignedAt` | DateTime | Não | `now()` | Data de vinculação |

**Constraints:**
- `UNIQUE` composta em `(collaboratorId, plantId)`

**Índices:**
- `(plantId)` — Busca de colaboradores por planta

---

### 3.4.10 ReportStaff (`report_staff`) — Implementado

Vinculação N:N entre colaboradores e relatórios, registrando a equipe que atuou em cada relatório. Usa FKs separadas para cada tipo de relatório (apenas uma preenchida por registro).

| Campo | Tipo | Nullable | Default | Descrição |
|-------|------|----------|---------|-----------|
| `id` | UUID | Não | `uuid_generate_v4()` | Chave primaria |
| `collaboratorId` | UUID | Não | - | FK para Collaborator (CASCADE) |
| `role` | String | Sim | - | Função específica naquele relatório |
| `slaughterReportId` | UUID | Sim | - | FK para SlaughterReport (CASCADE) |
| `productionReportId` | UUID | Sim | - | FK para ProductionReport (CASCADE) |
| `shippingReportId` | UUID | Sim | - | FK para ShippingReport (CASCADE) |

**Índices:**
- `(collaboratorId)` — Histórico de atuação do colaborador
- `(slaughterReportId)` — Equipe do relatório de abate
- `(productionReportId)` — Equipe do relatório de produção
- `(shippingReportId)` — Equipe do relatório de embarque

**Utilitário backend:** `syncReportStaff()` em `src/common/utils/report-staff.util.ts` gerencia a sincronização via delete+create nos 3 tipos de relatório.

---

## 3.5 Convencoes

### 3.5.1 Chaves Primarias

- Todas as tabelas utilizam **UUID v4** como chave primaria
- Gerado no banco via `uuid_generate_v4()` (extensão `uuid-ossp`)
- Tipo Postgres: `UUID`
- Motivo: evitar enumeração sequencial, segurança em APIs públicas, compatibilidade com sistemas distribuidos

### 3.5.2 Nomenclatura de Tabelas

- Prisma Models usam **PascalCase** (ex: `SlaughterReport`)
- Tabelas no Postgres usam **snake_case** via `@@map()` (ex: `slaughter_reports`)
- Mapeamento completo:

| Model (Prisma) | Tabela (Postgres) | Status |
|-----------------|-------------------|--------|
| SupervisorProfile | `supervisor_profiles` | Implementado |
| Plant | `plants` | Implementado |
| SlaughterReport | `slaughter_reports` | Implementado |
| ProductionReport | `production_reports` | Implementado |
| ShippingReport | `shipping_reports` | Implementado |
| NonConformity | `non_conformities` | Implementado |
| SupervisorSchedule | `supervisor_schedules` | Implementado |
| Collaborator | `collaborators` | Implementado |
| CollaboratorPlant | `collaborator_plants` | Implementado |
| ReportStaff | `report_staff` | Implementado |

### 3.5.3 Timestamps

- Todos os modelos possuem `created_at` e `updated_at`
- `created_at`: `DateTime @default(now())` - preenchido automáticamente na insercao
- `updated_at`: `DateTime @updatedAt` - atualizado automáticamente pelo Prisma em cada `UPDATE`
- Campos mapeados para snake_case no banco via `@map()`

### 3.5.4 Campos JSON

Campos JSON são usados para dados com estrutura flexível ou listas heterogêneas. Embora não tenham validação no banco, são validados pela camada de aplicação (DTOs + Zod).

| Campo | Modelo | Estrutura Esperada |
|-------|--------|--------------------|
| `qualifications` | SupervisorProfile | `{ certifications: string[], courses: string[], ... }` |
| `preferences` | SupervisorProfile | `{ theme: string, notifications: boolean, ... }` |
| `address` | Plant | `{ street: string, city: string, state: string, zip: string, lat?: number, lng?: number }` |
| `contact` | Plant | `{ phone: string, email: string, responsible: string }` |
| `verificationItems` | SlaughterReport | `[{ id: number, description: string, conformity: "C" \| "NC", observation?: string }]` |
| `stunningVerifications` | SlaughterReport | `[{ time: string, pressure: number, result: string }]` |
| `meatRawMaterials` | ProductionReport | `[{ slaughterhouse: string, sif: string, slaughterDate: string, csn: string, halalCert: string }]` |
| `approvedIngredients` | ProductionReport | `[{ name: string, supplier: string, batch: string, expiry: string }]` |
| `verificationItems` | ProductionReport | `[{ id: number, description: string, conformity: "C" \| "NC", observation?: string }]` |
| `products` | ShippingReport | `[{ name: string, code: string, batch: string, mfgDate: string, expDate: string, weight: number, temp: number }]` |
| `verificationItems` | ShippingReport | `[{ id: number, description: string, conformity: "C" \| "NC", observation?: string }]` |
| `evidence` | NonConformity | `{ photos: string[], description: string, references: string[] }` |

### 3.5.5 Índices e Performance

A estratégia de indexação segue o padrão de acesso mais frequente:

- **Índices compostos por entidade + data**: Todas as tabelas de relatório possuem índices `(plantId, date)` e `(supervisorId, date)` para queries de listagem páginada filtradas por planta ou supervisor em um período
- **Índices por status**: Permitem filtragem eficiente de relatórios pendentes, em revisão, etc.
- **Índices específicos**:
  - `ShippingReport(shippingType, status)` - Filtra por tipo de embarque + status
  - `NonConformity(severity, status)` - Prioriza NCs críticas abertas
  - `NonConformity(supervisorId)` - NCs por supervisor

### 3.5.6 Soft Delete vs. Hard Delete

O sistema utiliza **desativação lógica** (`isActive = false`) em vez de exclusão física para:
- `SupervisorProfile`: Supervisores desativados não aparecem em listagens ativas, mas seus relatórios históricos permanecem
- `Plant`: Plantas desativadas preservam histórico de relatórios e NCs

Relatórios e NCs **não são excluidos**; utilizam o workflow de status (`cancelado`/`encerrada`) para controle de ciclo de vida.
