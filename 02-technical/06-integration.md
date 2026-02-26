---
title: Integração
parent: Documentação Técnica
nav_order: 6
---

# 6. Integração e Ecossistema FAMBRAS

---

## 6.1 Visão Geral do Ecossistema

O SIH faz parte de um ecossistema de 3 sistemas que cobrem toda a cadeia Halal da FAMBRAS:

```
┌─────────────────────────────┐    ┌─────────────────────────────┐    ┌─────────────────────────────┐
│  HalalSphere                │    │  SIH                        │    │  SysHalal                   │
│  Gestao de Certificacoes    │    │  Supervisao Industrial Halal│    │  Emissao de Certificados    │
│  (UPSTREAM)                 │    │  (OPERACIONAL)              │    │  (DOWNSTREAM)               │
│                             │    │                             │    │                             │
│  - Certifica empresas       │    │  - Relatorios de abate      │    │  - Emite certificados Halal │
│  - Gerencia auditorias      │    │  - Relatorios de producao   │    │    de exportacao            │
│  - Emite certificados       │    │  - Relatorios de embarque   │    │  - Gera PDFs certificados   │
│  - Gerencia propostas       │    │  - Nao-conformidades        │    │  - QR Code de validacao     │
│  - Workflow 17 fases        │    │  - Escalas de supervisores  │    │  - Gestao de operacoes      │
│  - AI/Chatbot               │    │  - Colaboradores            │    │  - Cartas de correcao       │
│                             │    │  - Geracao de PDF FAMBRAS   │    │                             │
│  NestJS 11 + React 19       │    │  NestJS 11 + React 19      │    │  NestJS 10 + Next.js 14     │
│  Prisma 7 + PostgreSQL 16   │    │  Prisma 7 + PostgreSQL 16  │    │  Prisma 6 + PostgreSQL 16   │
│  Porta: 3333                │    │  Porta: 3334               │    │  Porta: configuravel        │
└─────────────────────────────┘    └─────────────────────────────┘    └─────────────────────────────┘
```

### Fluxo de Dados entre Sistemas

```
HalalSphere (upstream)          SIH (operacional)           SysHalal (downstream)

  Empresa certificada ─────────> Planta (SIF) ─────────────> Empresa (SIF)
  Escopo de produtos             Relatorio de abate          Produtos (JSON)
  Auditoria realizada            Relatorio de producao       Pesos bruto/liquido
  Certificado emitido            Relatorio de embarque       Operacoes (abate/proc)
                                 Colaboradores               BL, Invoice, CSI
                                 NCs                         Porto orig/dest
```

---

## 6.2 Repositorios

| Sistema | Backend | Frontend |
|---------|---------|----------|
| **HalalSphere** | `C:\Projetos\halalsphere-backend` | `C:\Projetos\halalsphere-frontend` |
| **SIH** | `C:\Projetos\sih-backend` | `C:\Projetos\sih-frontend` |
| **SysHalal** | `C:\Projetos\syshalal-api` | `C:\Projetos\syshalal-web` |

---

## 6.3 Entidades Compartilhaveis — Mapeamento Cruzado

### 6.3.1 Plantas/Empresas

| Campo | HalalSphere (`Company`) | SysHalal (`tb_empresa`) | SIH (`Plant`) |
|-------|------------------------|------------------------|---------------|
| ID | UUID | Snowflake BigInt | UUID |
| Nome | `razaoSocial` | `razao_social` | `name` |
| Código SIF | `plantCode` + `plantCodeType` | campo `sif` | `sifCode` |
| CNPJ | `cnpj` (14 chars) | via `tb_cliente` | — (não cadastra) |
| Tipo | `tipoEmpresa` | flags (`typeSlaughterer`, `typeProcessor`, etc.) | `type` (enum: frigorífico, processamento) |
| Grupo | `groupId` → `CompanyGroup` | `cliente_id` → `tb_cliente` | — |
| Endereço | `address` (JSON) | `endereco_id` → `tb_endereco` | `address` (JSON) |
| Especies | — | — | `species[]` (bovino, ave) |
| Vínculo externo | — | — | `externalCompanyId` (preparado para HalalSphere) |

### 6.3.2 Pessoas/Colaboradores

| Aspecto | HalalSphere | SysHalal | SIH |
|---------|-------------|----------|-----|
| Usuários sistema | 12 roles (admin, empresa, analista, auditor, etc.) | RBAC dinamico | 4 roles (admin, coordenador, supervisor, operador) |
| Colaboradores não-usuários | — | `tb_pessoas` (contatos comerciais) | `Collaborator` (equipe operacional: degoladores, sheiks, auxiliares) |
| Auditores | `AuditorCompetency` (completo) | — | — |
| Vínculo com relatório | — | `cx_certificado_operacao` | `ReportStaff` (N:N) |

### 6.3.3 Certificados

| Campo | HalalSphere (`Certificate`) | SysHalal (`tb_certificado`) | SIH |
|-------|----------------------------|----------------------------|-----|
| Num certificado | `certificateNumber` | `nr_certificado` | — (gera relatórios, não certificados) |
| Produtos | `ScopeProduct` | `json_produtos` (JSON) | produtos por relatório (JSON) |
| Pesos | — | `vl_peso_bruto/liquido` | nos relatórios de embarque |
| Operações | — | `cx_certificado_operacao` | relatórios de abate/produção |
| CSI/CSN | — | `ds_certificado_sanitario` | `csiNumber` no shipping report |

---

## 6.4 Autenticação — v1.0 (Atual)

Na v1.0, o SIH opera com **autenticação própria (self-contained)**:

```
Frontend SIH ──(POST /auth/login)──> Backend SIH ──(bcrypt + JWT HS256)──> Token
```

- **Não depende** do HalalSphere para autenticar
- Backend gera JWT com HS256 usando `JWT_SECRET`
- Senha armazenada com bcrypt (salt rounds 10)
- JWT contém: `sub` (userId), `email`, `name`, `role`
- Token válido por 7 dias (`JWT_EXPIRES_IN`)
- Guards globais: `JwtAuthGuard` + `RolesGuard`
- Decorators: `@Public()` para pular auth, `@Roles('admin', 'coordenador')` para restringir

### Variaveis de Ambiente (v1.0)

| Variavel | Descrição | Exemplo |
|----------|-----------|---------|
| `JWT_SECRET` | Chave secreta HS256 | `dev-secret-key-minimum-32-chars...` |
| `JWT_EXPIRES_IN` | Válidade do token | `7d` |

---

## 6.5 Estratégia de Integração Futura

### 6.5.1 Fase 1 — Cadastros Compartilhados (HalalSphere como Master)

Quando a integração for implementada, o HalalSphere será o **master** dos cadastros de:
- **Plantas/Empresas**: `Company` do HalalSphere (com `plantCode` SIF) → vinculado via `Plant.externalCompanyId` no SIH
- **Colaboradores**: Futuro módulo no HalalSphere → vinculado via `Collaborator.externalId` no SIH

O SIH mantém copias locais com `externalId` para:
- Funcionar independente (sem dependência de rede)
- Preservar dados históricos mesmo se o master mudar

### 6.5.2 Fase 2 — Dados de Supervisão para SysHalal

O SIH alimenta o SysHalal com dados operacionais para emissão de certificados de exportação:
- Datas de abate, produção, válidades
- Pesos bruto/liquido por produto
- Números SIF de origem
- CSI/CSN associados
- Operações (abatedouro, processadora)

### 6.5.3 Mecanismo de Sincronização

| Direcao | Mecanismo | Dados |
|---------|-----------|-------|
| HalalSphere → SIH | API REST (sob demanda) | Plantas, colaboradores, certificados |
| SIH → SysHalal | API REST ou webhook | Relatórios assinados, dados de produto |
| SIH → HalalSphere | Webhook (evento) | NCs críticas, resumos de supervisão |

---

## 6.6 Modelo Collaborator — Decisao Arquitetural

### 6.6.1 Problema

Os relatórios de abate/produção envolvem diversos profissionais além do supervisor:
- Degoladores (realizam o abate Halal)
- Sheiks (autoridade religiosa)
- Auxiliares de supervisão
- Veterinarios da planta
- Supervisores da planta

Atualmente o SIH registra apenas `slaughtererName` e `slaughtererDoc` como texto livre no relatório de abate. Não há cadastro centralizado nem vínculo estruturado.

### 6.6.2 Solução

Criar as entidades `Collaborator` e `ReportStaff` no SIH:

```prisma
model Collaborator {
  id            String   @id @default(uuid())
  externalId    String?  @unique          // Vinculo futuro com HalalSphere
  name          String
  document      String                    // RG, CPF ou passaporte
  documentType  String   @default("rg")   // rg | cpf | passaporte
  role          String                    // degolador | sheik | auxiliar | supervisor_planta | veterinario
  photoUrl      String?                   // URL da foto do colaborador (S3/local)
  phone         String?
  email         String?
  isActive      Boolean  @default(true)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  plants        CollaboratorPlant[]       // Plantas onde atua (N:N)
  reportStaff   ReportStaff[]

  @@map("collaborators")
}

model CollaboratorPlant {
  collaboratorId String
  plantId        String
  collaborator   Collaborator @relation(fields: [collaboratorId], references: [id])
  plant          Plant        @relation(fields: [plantId], references: [id])

  @@id([collaboratorId, plantId])
  @@map("collaborator_plants")
}

model ReportStaff {
  id             String       @id @default(uuid())
  collaboratorId String
  reportId       String                    // ID do relatorio (abate/producao/embarque)
  reportType     String                    // slaughter | production | shipping
  roleInReport   String                    // Funcao naquele dia especifico
  date           DateTime                  // Data do servico

  collaborator   Collaborator @relation(fields: [collaboratorId], references: [id])

  @@index([reportId, reportType])
  @@index([collaboratorId, date])
  @@map("report_staff")
}
```

### 6.6.3 Foto do Colaborador

- Upload via endpoint `POST /collaborators/:id/photo` (multipart/form-data)
- Armazenamento: pasta local em dev (`uploads/collaborators/`), S3 em produção
- Formato aceito: JPEG, PNG (max 2MB)
- `photoUrl` armazena o path relativo ou URL S3
- Foto exibida no cadastro e opcionalmente nos relatórios impressos

### 6.6.4 Compartilhamento Futuro

- `externalId` permite vincular ao HalalSphere quando integrar
- SIH e o **master** deste cadastro na v1.0 (único sistema que gerência colaboradores operacionais)
- Na integração futura, avalia-se se HalalSphere absorve este cadastro ou se SIH continua como master

### 6.6.5 Impacto nos Relatórios

- Campos `slaughtererName` / `slaughtererDoc` no SlaughterReport passam a ser preenchidos automáticamente a partir do `Collaborator` selecionado
- Tela de relatório ganha seção "Equipe do dia" com selecao multipla de colaboradores
- PDF impresso pode listar a equipe envolvida

---

## 6.7 Comparativo Técnico dos Sistemas

| Aspecto | HalalSphere | SIH | SysHalal |
|---------|-------------|-----|----------|
| Framework backend | NestJS 11 | NestJS 11 | NestJS 10 |
| ORM | Prisma 7 | Prisma 7 | Prisma 6 |
| ID strategy | UUID | UUID | Snowflake BigInt |
| Auth | JWT HS256/RS256 + bcrypt + cookies | JWT HS256 + bcrypt | JWT RS256 + bcrypt |
| Frontend | React 19 + Vite 7 | React 19 + Vite 7 | Next.js 14 + React 18 |
| UI | shadcn/ui + Tailwind v4 | shadcn/ui + Tailwind v4 | shadcn/ui + Tailwind v3 |
| State mgmt | React Query | React (local) | Zustand + React Query |
| Database | PostgreSQL 16 + pgvector | PostgreSQL 16 | PostgreSQL 16 |
| PDF | PDFKit | PDFKit | jsPDF + html2canvas |
| Modelos | 42 entidades | 7+3 entidades (v1.0+Collaborator) | ~20 entidades |
| Deploy | AWS (S3, SES, Secrets Manager) | Docker/local | AWS (S3, SES, SNS) |
