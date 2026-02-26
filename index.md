---
layout: home
title: Home
nav_order: 1
permalink: /
---

# SIH - Supervisão Industrial Halal

**Sistema de digitalização dos formulários FM FAMBRAS Halal para supervisão industrial em campo**

![Version](https://img.shields.io/badge/version-1.0-green)
![Status](https://img.shields.io/badge/status-em_desenvolvimento-yellow)

---

## Sobre

O SIH digitaliza os formulários FM da FAMBRAS Halal utilizados por supervisores muçulmanos em plantas industriais. O sistema é otimizado para uso em tablets em campo e opera com autenticação própria (self-contained). Faz parte do ecossistema FAMBRAS junto com HalalSphere (certificações) e SysHalal (emissão de certificados).

---

## Documentação

### PRD - Product Requirements Document

| Documento | Descrição |
|-----------|-----------|
| [Visão Geral](./01-prd/01-overview.md) | Problema, solução e catálogo de formulários FM |
| [Objetivos e KPIs](./01-prd/02-objectives.md) | Métricas de sucesso do projeto |
| [Personas](./01-prd/03-personas.md) | Supervisor, operador, coordenador, admin |
| [Arquitetura de Features](./01-prd/04-architecture.md) | Épicos e priorização |
| [User Stories](./01-prd/05-user-stories/) | 7 épicos com user stories detalhadas |
| [Roadmap](./01-prd/06-roadmap.md) | Fases de desenvolvimento |
| [Requisitos Não-Funcionais](./01-prd/07-non-functional.md) | Performance, segurança, usabilidade |

### Documentação Técnica

| Documento | Descrição |
|-----------|-----------|
| [Stack Tecnológica](./02-technical/01-stack.md) | NestJS 11 + React 19 + PostgreSQL 16 |
| [Arquitetura do Sistema](./02-technical/02-system-architecture.md) | Componentes e integração |
| [Banco de Dados](./02-technical/03-database/) | ERD e dicionario de dados |
| [APIs](./02-technical/04-apis.md) | Endpoints REST |
| [Segurança](./02-technical/05-security.md) | JWT, RBAC, estratégia offline |
| [Integração](./02-technical/06-integration.md) | Gestão de Certificações |

### Guias e Planejamento

| Documento | Descrição |
|-----------|-----------|
| [Setup Ambiente Local](./GUIDES/SETUP-AMBIENTE-LOCAL.md) | Configuração do ambiente de desenvolvimento |
| [Roadmap Detalhado](./PLANNING/ROADMAP.md) | Cronograma de implementação |
