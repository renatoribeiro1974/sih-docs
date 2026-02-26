---
title: Fase A — Colaboradores + Formulários Especializados
nav_order: 2
---

# Fase A — Colaboradores + Formulários Especializados

**Projeto**: SIH - Supervisão Industrial Halal
**Status**: PLANEJADO
**Pré-requisito**: v1.0 em produção
**Última Atualização**: 2026-02-26

---

## Objetivo

Expandir o SIH com:
1. **Cadastro de Colaboradores** — profissionais não-usuários (degoladores, sheiks, auxiliares, veterinários) com foto via AWS S3, vinculação N:N com plantas e equipe por relatório.
2. **Formulários Especializados** — expandir de 6 para 20 variantes de FM usando arquitetura Discriminador + JSON nos modelos existentes.

---

## Parte 1: Colaboradores

### Novos Modelos Prisma

#### Collaborator
| Campo | Tipo | Descrição |
|-------|------|-----------|
| id | UUID | PK |
| name | String | Nome completo |
| document | String? | RG ou CPF |
| type | CollaboratorType | degolador, sheik, auxiliar, veterinario, outro |
| phone | String? | Telefone |
| photoUrl | String? | URL da foto no S3 |
| externalId | String? | ID no HalalSphere (integração futura) |
| isActive | Boolean | Ativo/inativo |

#### CollaboratorPlant (N:N)
| Campo | Tipo | Descrição |
|-------|------|-----------|
| collaboratorId | UUID | FK → Collaborator |
| plantId | UUID | FK → Plant |
| assignedAt | DateTime | Data de vinculação |

#### ReportStaff (N:N — Equipe por Relatório)
| Campo | Tipo | Descrição |
|-------|------|-----------|
| collaboratorId | UUID | FK → Collaborator |
| role | String? | Função no relatório |
| slaughterReportId | UUID? | FK opcional → SlaughterReport |
| productionReportId | UUID? | FK opcional → ProductionReport |
| shippingReportId | UUID? | FK opcional → ShippingReport |

### Upload de Fotos — AWS S3

- Serviço S3 com pre-signed URLs
- Validação: JPEG/PNG, max 2MB
- Key: `collaborators/{id}/photo.{ext}`
- Dependências: `@aws-sdk/client-s3`, `@aws-sdk/s3-request-presigner`

### Backend — Módulo Collaborator

Endpoints:
```
GET    /collaborators              — listar (filtro: type, plantId, isActive, search)
GET    /collaborators/:id          — detalhe
POST   /collaborators              — criar
PATCH  /collaborators/:id          — editar
PATCH  /collaborators/:id/deactivate — desativar
POST   /collaborators/:id/photo    — upload foto
DELETE /collaborators/:id/photo    — remover foto
POST   /collaborators/:id/plants   — vincular a planta(s)
DELETE /collaborators/:id/plants/:plantId — desvincular
```

Roles: `@Roles('admin', 'coordenador')` para todos os endpoints.

### Frontend — Páginas

| Página | Descrição |
|--------|-----------|
| CollaboratorList | Tabela paginada + filtros (tipo, planta, ativo) + busca |
| CollaboratorForm | Form criar/editar + upload foto + vinculação plantas |

### Equipe por Relatório (ReportStaff)

- Componente `ReportStaffSelector` — multi-select de colaboradores filtrados pela planta
- Integrado nos 3 forms de relatório (Abate, Produção, Embarque)
- PDF inclui seção "Equipe / Staff" antes da assinatura

---

## Parte 2: Formulários Especializados (14 FMs)

### Arquitetura: Discriminador + JSON

| Modelo | Discriminador | Campo JSON | FMs Novos |
|--------|--------------|------------|-----------|
| ProductionReport | `productionType` (enum novo) | `customFields` (Json) | 7 tipos novos |
| ShippingReport | `shippingType` (enum expandido) | `customFields` (Json) | 6 novos |

### Produção — ProductionType Enum

| Valor | FM | Descrição |
|-------|-----|-----------|
| fabricacao | 7.1.3.1 | Fabricação (existente) |
| tripas | 7.1.3.3 | Tripas calibradas e salgadas |
| fracionamento | 7.1.3.5 | Fracionamento |
| couro | 7.1.4.5 | Couro verde / Pele bovina |
| mucosa | 7.1.4.6 | Produção de mucosa |
| heparina | 7.1.8.2 | Extração de heparina |
| raspa | 7.1.8.5 | Raspa/aparas |
| gelatina | 7.1.8.6 | Processamento de gelatina |

### Embarque — ShippingType Expandido

| Valor | FM | Descrição |
|-------|-----|-----------|
| exportacao | 7.1.7.1 | Exportação in natura (existente) |
| venda_interna | 7.1.7.4 | Venda mercado interno (existente) |
| transferencia | 7.1.7.3 | Transferência (existente) |
| exportacao_industrializados | 7.1.7.2 | Exportação industrializados |
| venda_industrializados | 7.1.7.5 | Venda industrializados |
| transferencia_industrializados | 7.1.7.9 | Transferência industrializados |
| embarque_subprodutos | 7.1.7.12 | Embarque subprodutos |
| transferencia_in_natura | 7.1.4.8.B | Transferência in natura |
| transferencia_subprodutos | 7.1.8.4 | Transferência subprodutos |

### Sidebar Expandida

```
── In Natura
│   ├── Abate
│   ├── Emb. Exportação
│   ├── Venda Merc. Int.
│   ├── Transferência
│   └── Transf. In Natura (NOVO)
── Industrializados
│   ├── Fabricação
│   ├── Tripas (NOVO)
│   ├── Fracionamento (NOVO)
│   ├── Emb. Export. Ind. (NOVO)
│   ├── Venda Ind. (NOVO)
│   └── Transf. Ind. (NOVO)
── Subprodutos
│   ├── Couro (NOVO)
│   ├── Mucosa (NOVO)
│   ├── Heparina (NOVO)
│   ├── Raspa/Aparas (NOVO)
│   ├── Gelatina (NOVO)
│   ├── Emb. Subprod. (NOVO)
│   └── Transf. Subprod. (NOVO)
── Gestão
│   ├── Colaboradores (NOVO)
│   └── ...
```

### PDF Templates

7 novos templates de produção + expansão do template de embarque para 6 novos subtipos.
Cada template reutiliza `pdf-helpers.ts` (cabeçalho FAMBRAS, assinatura, verificações).

---

## Ordem de Implementação

| Passo | Escopo |
|-------|--------|
| 1 | Schema Prisma (3 modelos + 2 enums) + Migration |
| 2 | Backend Collaborator (CRUD + S3) |
| 3 | Frontend Collaborator (List + Form) |
| 4 | ReportStaff integration |
| 5 | Backend Production specialization (DTOs + filtro) |
| 6 | Frontend Production (8 componentes de tipo) |
| 7 | Backend/Frontend Shipping (6 novos subtipos) |
| 8 | Sidebar + routing |
| 9 | PDF templates (13 novos) |
| 10 | Seed data + verificação |

---

## Referência: Documentos FM Originais

Localizados em `C:\SIH\`:
- FM 7.1.3.3 — Tripas calibradas e salgadas
- FM 7.1.3.5 — Fracionamento
- FM 7.1.4.5 — Couro verde
- FM 7.1.4.6 — Mucosa
- FM 7.1.8.2 — Heparina
- FM 7.1.8.5 — Raspa/aparas
- FM 7.1.8.6 — Gelatina
- FM 7.1.7.2, 7.1.7.5, 7.1.7.9, 7.1.7.12 — Embarques especializados
- FM 7.1.4.8.B, 7.1.8.4 — Transferências
