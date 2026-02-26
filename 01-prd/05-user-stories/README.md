---
title: User Stories
parent: PRD
nav_order: 5
has_children: true
permalink: /prd/user-stories/
---

# User Stories - SIH v1.0

---

## Resumo

| Épico | FM | User Stories | Story Points | Prioridade |
|-------|----|:------------:|:------------:|:----------:|
| [Epic 01: Relatórios de Abate](./epic-01-relatórios-abate.md) | FM 7.1.4.x | 7 | 47 | P0 |
| [Epic 02: Relatórios de Produção](./epic-02-relatórios-produção.md) | FM 7.1.3.x / FM 7.1.8.x | 6 | 40 | P0 |
| [Epic 03: Relatórios de Embarque](./epic-03-relatórios-embarque.md) | FM 7.1.7.x / DCPOA | 6 | 38 | P0 |
| [Epic 04: Não-Conformidades](./epic-04-não-conformidades.md) | FM 7.1.6.1 | 6 | 35 | P0 |
| [Epic 05: Escala de Supervisores](./epic-05-escala-supervisores.md) | - | 4 | 20 | P1 |
| [Epic 06: Dashboard e Relatórios](./epic-06-dashboard.md) | - | 4 | 22 | P1 |
| [Epic 07: Inventário](./epic-07-inventário.md) | FM 7.1.5.x / FM 7.1.3.6 | 6 | TBD | P2 (FUTURO) |
| **Total v1.0** | | **33** | **202** | |
| **Total (com futuro)** | | **39** | **202+** | |

---

## Convencoes

### Formato das User Stories

```
Como [persona],
Eu quero [acao],
Para que [beneficio].
```

### Priorização

| Nível | Descrição |
|-------|-----------|
| **P0 - Must Have** | Essencial para v1.0. Sistema não funciona sem isso. |
| **P1 - Should Have** | Importante para v1.0. Pode ser simplificado se necessário. |
| **P2 - Nice to Have** | Documentado para implementação futura. |

### Story Points (Fibonacci)

| Pontos | Complexidade |
|--------|-------------|
| 1-2 | Trivial (campo simples, CRUD básico) |
| 3-5 | Simples (formulário padrão, validações) |
| 8 | Medio (lógica de negócio, seções condicionais) |
| 13 | Complexo (workflow, calculos, integração) |
| 21 | Muito complexo (multiplos subsistemas) |

### Nomenclatura

- **SIH-XXX**: Identificador único da user story
- **FM X.X.X.X**: Referência ao formulário FAMBRAS
- **PR 7.1**: Referência ao procedimento Rev 22
