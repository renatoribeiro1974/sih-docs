---
title: PRD
nav_order: 2
has_children: true
permalink: /prd/
---

# Product Requirements Document v1.0

**SIH - Supervisão Industrial Halal**
**Sistema de Digitalização dos Formulários FM FAMBRAS Halal**

---

## Controle do Documento

| Campo | Valor |
|-------|-------|
| **Versão** | 1.0 |
| **Data** | 24 de Fevereiro de 2026 |
| **Autor** | Product Manager - SIH Team |
| **Status** | Em Desenvolvimento |

---

## Índice do PRD

### Parte 1: Fundamentos do Produto

1. **[Visão Geral do Produto](./01-overview.md)**
   - Problema que resolvemos
   - Solução proposta
   - Catálogo de formulários FM FAMBRAS
   - Estrutura padrão dos formulários

2. **[Objetivos e Métricas de Sucesso](./02-objectives.md)**
   - Objetivos do projeto
   - KPIs de negócio
   - Métricas operacionais

3. **[Personas e Jornadas](./03-personas.md)**
   - Supervisor Muçulmano (preenche e assina relatórios)
   - Operador (preenche relatórios, não assina)
   - Coordenador de Supervisores (gestão de equipe, read-only em relatórios)

4. **[Arquitetura de Features](./04-architecture.md)**
   - 7 Épicos organizados
   - Priorização e dependências
   - Mapeamento FM → Módulo

### Parte 2: User Stories

5. **[User Stories por Épico](./05-user-stories/)**
   - [Epic 01: Relatórios de Abate](./05-user-stories/epic-01-relatórios-abate.md) - FM 7.1.4.x
   - [Epic 02: Relatórios de Produção](./05-user-stories/epic-02-relatórios-produção.md) - FM 7.1.3.x / FM 7.1.8.x
   - [Epic 03: Relatórios de Embarque](./05-user-stories/epic-03-relatórios-embarque.md) - FM 7.1.7.x / DCPOA
   - [Epic 04: Não-Conformidades](./05-user-stories/epic-04-não-conformidades.md) - FM 7.1.6.1
   - [Epic 05: Escala de Supervisores](./05-user-stories/epic-05-escala-supervisores.md)
   - [Epic 06: Dashboard e Relatórios](./05-user-stories/epic-06-dashboard.md)
   - [Epic 07: Inventário](./05-user-stories/epic-07-inventário.md) - FM 7.1.5.x / FM 7.1.3.6 **[FUTURO]**

### Parte 3: Planejamento

6. **[Roadmap](./06-roadmap.md)**
   - Fases de implementação
   - Marcos e entregas

7. **[Requisitos Não-Funcionais](./07-non-functional.md)**
   - Performance e escalabilidade
   - Segurança e autenticação
   - Usabilidade e acessibilidade
   - PWA e preparação offline
