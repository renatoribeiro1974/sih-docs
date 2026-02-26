---
title: Requisitos Não-Funcionais
parent: PRD
nav_order: 7
---

# 7. Requisitos Não-Funcionais

---

## 7.1 Performance

| Requisito | Meta | Justificativa |
|-----------|------|---------------|
| Tempo de resposta API (P95) | < 500ms | Uso em campo com possível 3G/4G |
| Tempo de resposta API (P99) | < 2s | Picos de uso no início de turno |
| First Contentful Paint (3G) | < 3s | Carregamento rápido em mobile |
| Time to Interactive (3G) | < 5s | Interação rápida após abertura |
| Tamanho do bundle (gzip) | < 500KB | Economia de dados moveis |
| Queries de listagem | < 200ms | Páginação eficiente |

### Estratégias

- **Páginação** em todas as listagens (20 itens/página padrão)
- **Índices de banco** nos campos de busca mais frequentes (plantId, date, status, supervisorId)
- **Lazy loading** de páginas no frontend (code splitting por rota)
- **Cache** de dados de referência (plantas, perfil) no React Query (staleTime: 5min)

---

## 7.2 Segurança

### Autenticação (v1.0 — Self-Contained)

| Requisito | Implementação |
|-----------|---------------|
| Algoritmo JWT | HS256 (self-contained — SIH emite e válida tokens) |
| Hash de senha | bcrypt (salt rounds 10) |
| Válidade do token | 7 dias (estendida para uso em campo) |
| Armazenamento do token | localStorage (v1.0), IndexedDB (futuro offline) |
| CORS | Configurado por ambiente (localhost:5174 / domínio prod) |

### Autorização (RBAC)

| Role | Permissões |
|------|-----------|
| `supervisor` | Cria/edita/assina relatórios (próprios), registra NCs, vê escala própria |
| `operador` | Cria/edita relatórios (próprios), registra NCs — NÃO pode assinar |
| `coordenador` | Visualiza relatórios (read-only), cancela relatórios, gerência escalas/usuários/NCs — NÃO assina |
| `admin` | Acesso total: CRUD completo, gerênciamento de usuários e plantas |

### Protecoes

| Proteção | Implementação |
|----------|---------------|
| Injeção SQL | Prisma ORM (parametrized queries) |
| XSS | React (escape automático) + Content-Security-Policy |
| CSRF | Cookie-based tokens (quando aplicavel) |
| Raté limiting | Configurável por endpoint (via middleware) |
| Dados sensiveis | Campos de senha nunca retornados na API |
| Audit trail | `createdAt`/`updatedAt` em todas as tabelas |

---

## 7.3 Usabilidade

### Responsividade

| Dispositivo | Breakpoint | Prioridade |
|-------------|-----------|:----------:|
| Tablet (landscape) | >= 1024px | **Principal** |
| Tablet (portrait) | >= 768px | Alta |
| Desktop | >= 1280px | Media |
| Mobile | < 768px | Baixa (funcional) |

### Acessibilidade em Campo

| Requisito | Justificativa |
|-----------|---------------|
| Touch targets >= 44x44px | Uso com luvas ou maos umidas |
| Fontes legiveis (min 16px body) | Leitura em ambientes com iluminação variavel |
| Contraste alto (WCAG AA) | Uso ao ar livre / sob sol |
| Feedback visual de ações | Confirmação clara de envio/salvamento |
| Formulários com validação inline | Erros visíveis sem scroll |
| Autosave de rascunho | Prevenir perda de dados (intervalo 30s) |

### Navegação

| Requisito | Implementação |
|-----------|---------------|
| Sidebar responsiva | Colapsavel em tablet, drawer em mobile |
| Breadcrumbs | Navegação contextual em páginas internas |
| Atalhos de ação | Botão flutuante "Novo Relatório" sempre visivel |
| Busca rápida | Campo de busca por serial no header |

---

## 7.4 PWA e Preparação Offline

### v1.0 (Preparado)

| Item | Status |
|------|--------|
| `manifest.json` | Incluso (nome, icones, theme-color, display: standalone) |
| Service Worker básico | Via vite-plugin-pwa (cache de assets estaticos) |
| Meta tags PWA | theme-color, apple-mobile-web-app-capable, viewport tablet |
| Icones PWA | 192x192, 512x512 (identidade FAMBRAS) |
| Instalavel no tablet | Sim (manifest + service worker) |
| Pasta `src/lib/offline/` | Stubs criados (db.ts, sync-queue.ts) |

### Futuro (A implementar)

| Item | Descrição |
|------|-----------|
| Cache de dados (IndexedDB) | Via Dexie.js - relatórios, plantas, perfil |
| Background Sync | Envio automático de relatórios pendentes |
| Indicador online/offline | Badge visual no header |
| Fila de sincronização | Relatórios pendentes com contador |
| Resolução de conflitos | Timestamp-based ou merge manual |
| Service Worker avancado | Workbox com estratégias de cache por rota |

---

## 7.5 Escalabilidade

| Requisito | Meta |
|-----------|------|
| Usuários simultaneos | 50+ supervisores |
| Relatórios/mês | 5.000+ |
| Tamanho do banco (1 ano) | < 5GB |
| NCs ativas simultaneas | 500+ |

### Estratégias

- **Páginação em todas as listagens** (nunca retornar datasets completos)
- **Índices compostos** nos campos mais filtrados (plantId + date, status + date)
- **JSON para dados variavels** (verification items, products, raw materials) - evita joins complexos
- **Particionamento futuro** para inventário de alto volume (FM 7.1.5.6)

---

## 7.6 Disponibilidade

| Requisito | Meta |
|-----------|------|
| Uptime | > 99.5% |
| RTO (Recovery Time Objective) | < 1 hora |
| RPO (Recovery Point Objective) | < 15 minutos |
| Backup do banco | Diário automático (AWS RDS) |

---

## 7.7 Observabilidade

| Item | Implementação |
|------|---------------|
| Logs estruturados | `LoggingInterceptor` em todas as requisicoes |
| Health check | `GET /health` (banco, redis, app) |
| Métricas de erro | Exception filter global com logging |
| Swagger/OpenAPI | Documentação automática de todos os endpoints |

---

## 7.8 Compatibilidade de Navegadores

| Navegador | Versão Mínima | Prioridade |
|-----------|:------------:|:----------:|
| Chrome (Android/Desktop) | 100+ | **Principal** |
| Safari (iOS/iPadOS) | 15+ | Alta |
| Samsung Internet | 20+ | Media |
| Firefox | 100+ | Media |
| Edge | 100+ | Baixa |

---

## 7.9 Infraestrutura

### Portas (sem conflito com HalalSphere)

| Serviço | Porta SIH | Porta HalalSphere |
|---------|:---------:|:-----------------:|
| Backend API | 3334 | 3333 |
| Frontend Dev | 5174 | 5173 |
| PostgreSQL | 5433 | 5432 |
| Redis | 6380 | 6379 |

### Banco de Dados

| Item | Valor |
|------|-------|
| SGBD | PostgreSQL 16+ |
| Nome do banco | `sih` |
| Extensões | uuid-ossp, pgcrypto, pg_trgm |
| ORM | Prisma 7.x |

### Cache

| Item | Valor |
|------|-------|
| Engine | Redis 7+ |
| Uso principal | Cache de sessão, raté limiting |

### Design System

| Item | Valor |
|------|-------|
| Cores primarias | Verde FAMBRAS #0A6847, Dourado #D4A853 |
| Fontes | Playfair Display (títulos), DM Sans (corpo), JetBrains Mono (código) |
| Componentes | shadcn/ui com tema FAMBRAS |
| Icones | Lucide React |
