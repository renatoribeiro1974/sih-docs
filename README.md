---
nav_exclude: true
---

# SIH - Supervisão Industrial Halal

**Sistema de digitalização dos formulários FM FAMBRAS Halal para supervisão industrial em campo**

---

## Sobre o Projeto

O SIH substitui os formulários em papel (PDFs) utilizados pelos supervisores muçulmanos da FAMBRAS Halal por um sistema digital responsivo, otimizado para uso em tablets em campo.

### Formulários Digitalizados

| Família | FM | Descrição |
|---------|-----|-----------|
| Abate | FM 7.1.4.1 / FM 7.1.4.2 | Relatórios de Abate Halal (Aves e Bovinos) |
| Produção | FM 7.1.3.1 / FM 7.1.8.x | Relatórios de Produção Industrial |
| Embarque | FM 7.1.7.1 / FM 7.1.7.4 / DCPOA | Embarque, Venda e Transferência |
| NC | FM 7.1.6.1 | Checklist de Não-Conformidade |
| Inventário | FM 7.1.5.x / FM 7.1.3.6 | Inventário de Carne, Lotes e Rotulagem (futuro) |

### Tech Stack

- **Backend**: NestJS 11 + Prisma 7 + PostgreSQL 16
- **Frontend**: React 19 + Vite 7 + Tailwind CSS + shadcn/ui
- **Auth**: Self-contained (bcrypt + JWT HS256)
- **Plataforma**: PWA responsivo (tablet-first)

---

## Documentação

- [PRD - Product Requirements Document](./01-prd/README.md)
- [Documentação Técnica](./02-technical/README.md)
- [Guias](./GUIDES/)
- [Planejamento](./PLANNING/)
