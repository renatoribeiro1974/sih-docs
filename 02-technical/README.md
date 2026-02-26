---
title: Documentação Técnica
nav_order: 3
has_children: true
permalink: /técnico/
---

# Documentação Técnica

**SIH - Supervisão Industrial Halal**

---

## Controle do Documento

| Campo | Valor |
|-------|-------|
| **Versão** | 1.0 |
| **Data** | 24 de Fevereiro de 2026 |
| **Status** | Em Desenvolvimento |

---

## Índice da Documentação Técnica

| # | Documento | Descrição |
|---|-----------|-----------|
| 1 | [Stack Tecnológica](./01-stack.md) | Tecnologias utilizadas no backend (NestJS 11, Prisma 7, PostgreSQL 16) e frontend (React 19, Vite 7, Tailwind CSS 4). Tabela completa de dependências com versões. |
| 2 | [Arquitetura do Sistema](./02-system-architecture.md) | Diagrama de alto nível, estrutura de módulos backend e frontend, padrões de comúnicação REST/JWT, mapeamento de portas e arquitetura de deploy. |
| 3 | [Banco de Dados](./03-database/) | Modelo entidade-relacionamento (ERD), dicionario de dados com todas as tabelas, campos e relacionamentos do schema Prisma. |
| 4 | [APIs](./04-apis.md) | Documentação dos endpoints REST, padrões de request/response, páginação, filtros e códigos de erro. |
| 5 | [Segurança](./05-security.md) | Autenticação self-contained (bcrypt + JWT HS256), autorização RBAC com 4 roles (admin, coordenador, supervisor, operador), protecoes contra ataques. |
| 6 | [Integração](./06-integration.md) | Ecossistema FAMBRAS (HalalSphere + SIH + SysHalal), entidades compartilhaveis, modelo Collaborator, estratégia de integração futura. |

---

## Stack Resumida

```
Frontend:  React 19 + Vite 7 + Tailwind CSS 4 + shadcn/ui
Backend:   NestJS 11 + Prisma 7 + PostgreSQL 16 + Redis 7
Auth:      JWT HS256 self-contained (bcrypt + Passport.js)
Infra:     Docker + AWS (ECS, S3, CloudFront)
```

---

## Como Navegar

- **Desenvolvedor novo no projeto**: Comece pelo [Stack Tecnológica](./01-stack.md) e depois [Arquitetura do Sistema](./02-system-architecture.md)
- **Precisa entender o banco**: Veja [Banco de Dados](./03-database/)
- **Vai implementar endpoints**: Consulte [APIs](./04-apis.md) e [Segurança](./05-security.md)
- **Integração com HalalSphere**: Veja [Integração](./06-integration.md)

---

## Documentos Relacionados

- [PRD - Product Requirements Document](../01-prd/README.md) - Requisitos do produto
- [Requisitos Não-Funcionais](../01-prd/07-non-functional.md) - Performance, escalabilidade, usabilidade
