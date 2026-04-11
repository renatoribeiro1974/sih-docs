---
title: Fase A.2 — Planejamento Detalhado (A.2.5, A.2.6, A.2.7)
nav_order: 4
---

# Fase A.2.5, A.2.6, A.2.7 — Planejamento Detalhado

**Ultima Atualizacao**: 2026-04-10
**Pre-requisito**: Fases A.2.1–A.2.4 completas

---

## A.2.5 — Sidebar Expandida + Routing

### Escopo

Expandir a sidebar de 4 grupos / ~10 itens para 4 grupos / ~25 itens.
A sidebar atual esta em `sih-frontend/src/components/layout/Sidebar.tsx` (285 linhas).
As rotas estao em `App.tsx` — NAO precisam mudar (producao e embarque ja usam query params para tipo).

### Estrutura Nova da Sidebar

```
Dashboard
Analytics

In Natura
  Abate                              → /slaughter-reports
  Emb. Exportacao                    → /shipping-reports?shippingType=exportacao
  Venda Merc. Int.                   → /shipping-reports?shippingType=venda_interna
  Transferencia                      → /shipping-reports?shippingType=transferencia
  Transf. In Natura (NOVO)           → /shipping-reports?shippingType=transferencia_in_natura

Industrializados
  Fabricacao                         → /production-reports?productionType=fabricacao
  Tripas (NOVO)                      → /production-reports?productionType=tripas
  Fracionamento (NOVO)               → /production-reports?productionType=fracionamento
  Emb. Export. Ind. (NOVO)           → /shipping-reports?shippingType=exportacao_industrializados
  Venda Ind. (NOVO)                  → /shipping-reports?shippingType=venda_industrializados
  Transf. Ind. (NOVO)                → /shipping-reports?shippingType=transferencia_industrializados

Subprodutos (NOVO grupo)
  Couro (NOVO)                       → /production-reports?productionType=couro
  Mucosa (NOVO)                      → /production-reports?productionType=mucosa
  Heparina Bruta (NOVO)              → /production-reports?productionType=heparina_bruta
  Heparina Purif. (NOVO)             → /production-reports?productionType=heparina_purificacao
  Raspa/Aparas (NOVO)                → /production-reports?productionType=raspa
  Gelatina (NOVO)                    → /production-reports?productionType=gelatina
  Venda Subprod. (NOVO)              → /shipping-reports?shippingType=venda_subprodutos
  Transf. Subprod. (NOVO)            → /shipping-reports?shippingType=transferencia_subprodutos
  Transf. Generica (NOVO)            → /shipping-reports?shippingType=transferencia_generica
  Venda Couro (NOVO)                 → /shipping-reports?shippingType=venda_subprod_couro

Nao-Conformidades
Escalas

Gestao (admin + coordenador)
  Usuarios
  Colaboradores
  Plantas
```

### Decisoes de Design

1. **Routing**: NAO criar rotas novas. Usar query params (`?productionType=X` e `?shippingType=X`).
   As listas ja filtram por esses params (implementado em A.2.3/A.2.4).

2. **Icones**: Reutilizar icones Lucide existentes + adicionar novos:
   - Subprodutos grupo: `FlaskConical` ou `Microscope`
   - Couro: `Shirt` (skin/leather)
   - Mucosa: `Droplets`
   - Heparina: `Syringe` ou `Pill`
   - Raspa: `Scissors`
   - Gelatina: `Sparkles`
   - Tripas: `CircleDot`
   - Fracionamento: `PackageOpen`

3. **Grupos colapsados por default**: Todos exceto o primeiro (In Natura).
   Quando a rota atual pertence a um grupo, ele abre automaticamente.

4. **Sidebar longa**: Adicionar `overflow-y-auto` com scroll (ja existe).
   Em telas pequenas, considerar agrupar "Subprodutos" em subgrupo colapsavel.

### Tarefas

- [ ] Atualizar array `navigation` em Sidebar.tsx com os novos itens
- [ ] Adicionar icones Lucide para os novos itens
- [ ] Atualizar `openGroups` initial state para incluir 'subprodutos'
- [ ] Atualizar link de "Fabricacao" para incluir `?productionType=fabricacao`
- [ ] Testar highlight ativo com query params (funcao `isLinkActive` ja suporta)
- [ ] Verificar responsividade com sidebar colapsada (tooltips nos novos itens)

### Impacto em Rotas

**ZERO mudancas em App.tsx**. A sidebar usa query params que as listas ja leem.

A unica mudanca necessaria e garantir que `ProductionReportList` leia o query param `productionType` da URL e o use como filtro default (atualmente vem do state local).

Tarefa adicional:
- [ ] `ProductionReportList`: ler `productionType` do URL param e sincronizar com o state
- [ ] `ShippingReportList`: ler `shippingType` do URL param (ja existe no filtro)

---

## A.2.6 — PDF Templates

### Escopo

Criar templates PDF para os 8 tipos de producao especializados + expandir o template de embarque para os 8 novos tipos.

Os PDFs existentes estao em `sih-backend/src/pdf-export/templates/`:
- `slaughter-report-pdf.ts` (nao muda)
- `production-report-pdf.ts` (precisa de dispatch por tipo)
- `shipping-report-pdf.ts` (precisa de dispatch por tipo)
- `pdf-helpers.ts` (reutilizado, nao muda)

### Estrategia de Implementacao

**NAO criar 17 arquivos de template separados.** Em vez disso:

1. **production-report-pdf.ts**: Manter o gerador existente para `fabricacao`, e adicionar uma funcao `generateSpecializedProductionPdf()` que faz dispatch por `productionType`, renderizando as secoes apropriadas a partir do `customFields`.

2. **shipping-report-pdf.ts**: O template existente ja suporta 3 tipos. Expandir o `HEADER_INFO` map e adicionar condicional para os novos tipos. A maioria dos novos tipos de embarque sao variantes leves dos 3 existentes.

### Producao — Mapeamento de Templates

| Tipo | Base reutilizavel? | Secoes especificas |
|------|-------------------|-------------------|
| `fabricacao` | Existente (nao muda) | — |
| `tripas` | Parcial (cabecalho sim) | Tabela casings com bombonas, produto final com macos/bombonas |
| `fracionamento` | Nao | Tabela ENTRADA + tabela SAIDA |
| `couro` | Nao | Linha unica producao + secao venda embutida |
| `mucosa` | Nao | Tabela diaria (20+ linhas, horarios, regua) |
| `heparina_bruta` | Parcial (secao final sim) | Mat-primas animais (6 cols), insumos quimicos |
| `heparina_purificacao` | Parcial (secao final sim) | Mat-primas (4 cols), insumos, etapa do processo |
| `raspa` | Parcial | Checkbox produto, mat-primas animais (6 cols DCPOA), multi-produto |
| `gelatina` | Parcial (secao final sim) | Mat-primas simplificada (2 cols), qtd descartada |

### Producao — Funcoes a Criar

```
src/pdf-export/templates/
  production-report-pdf.ts         ← existente, adicionar dispatch
  production-specialized/
    tripas-pdf.ts                  ← Padrao B
    fracionamento-pdf.ts           ← Padrao C
    couro-pdf.ts                   ← Padrao D
    mucosa-pdf.ts                  ← Padrao E
    heparina-bruta-pdf.ts          ← Padrao F
    heparina-purificacao-pdf.ts    ← Padrao F (variante)
    raspa-pdf.ts                   ← Padrao G
    gelatina-pdf.ts                ← Padrao F (variante)
```

Cada arquivo exporta uma funcao `drawSpecializedSections(doc, report, y)` que renderiza
SOMENTE as secoes especificas do tipo (mat-primas, produto final). As secoes compartilhadas
(cabecalho FAMBRAS, identificacao da unidade, verificacoes, equipe/staff, assinatura) sao
chamadas pelo dispatcher em `production-report-pdf.ts`.

### Embarque — Mapeamento

| Novo tipo | Base reutilizavel | Diferencas |
|-----------|------------------|------------|
| `exportacao_industrializados` | `exportacao` | Sem frigorifico abate, sem CSI |
| `venda_industrializados` | `venda_interna` | +INDEA code, +SIF destino, sem CSI |
| `transferencia_industrializados` | `transferencia` | Tabela simplificada (3 cols), +serial Halal |
| `venda_subprodutos` | `venda_interna` | Produto unico (nao tabela), verificacao tanque |
| `transferencia_in_natura` | `transferencia` | Tabela 6 cols |
| `transferencia_subprodutos` | `transferencia` | Tabela 6 cols |
| `transferencia_generica` | `transferencia` | Tabela 7 cols (com codigo) |
| `venda_subprod_couro` | `venda_interna` | Campos transferencia, tabela 6 cols |

**Abordagem para embarque**: Expandir o template existente com condicionais por tipo.
Todos usam o mesmo layout base (informacoes da transferencia/exportacao/venda + tabela produto + verificacoes + assinatura). As diferencas sao:
- Quais campos de info aparecem
- Formato da tabela de produtos (3–11 colunas)
- Texto da declaracao de assinatura

### Tarefas

**Producao:**
- [ ] Criar diretorio `production-specialized/`
- [ ] Implementar `tripas-pdf.ts` — tabela casings + produto final com bombonas
- [ ] Implementar `fracionamento-pdf.ts` — tabela entrada/saida
- [ ] Implementar `couro-pdf.ts` — producao + venda hibrida
- [ ] Implementar `mucosa-pdf.ts` — log diario com horarios
- [ ] Implementar `heparina-bruta-pdf.ts` — mat-primas animais + insumos quimicos
- [ ] Implementar `heparina-purificacao-pdf.ts` — variante com etapa do processo
- [ ] Implementar `raspa-pdf.ts` — checkbox produto + multi-produto
- [ ] Implementar `gelatina-pdf.ts` — mat-primas simplificada
- [ ] Atualizar `production-report-pdf.ts` — dispatch por productionType
- [ ] Atualizar `pdf-export.service.ts` — passar productionType ao gerador

**Embarque:**
- [ ] Expandir `HEADER_INFO` em shipping-report-pdf.ts (8 novos tipos)
- [ ] Adicionar condicional para campos por tipo (INDEA, SIF destino, etc.)
- [ ] Adicionar variantes de tabela de produto (3, 6, 7 colunas)
- [ ] Adicionar variante "produto unico" para venda_subprodutos
- [ ] Atualizar declaracao de assinatura por tipo

### Dependencias

- Os templates PDF leem `report.customFields` — que so tera dados quando
  os formularios especializados do frontend forem preenchidos (placeholders
  implementados em A.2.3/A.2.4).
- Para testes, usar seed data (A.2.7) com customFields populados.

### Risco

**Maior risco**: Layout pixel-perfect dos PDFs para FMs com tabelas complexas
(tripas com 8+ colunas, mucosa com 20+ linhas diarias). Testar gerando PDFs
e comparando visualmente com os FMs originais em `C:\SIH\`.

---

## A.2.7 — Seed + Testes + Build

### Escopo

Criar dados de exemplo para cada tipo de producao e embarque, garantindo que
todo o pipeline (create → findAll → findOne → sign → PDF) funciona.

### Seed Data

**Arquivo**: `sih-backend/prisma/seed.ts` (existente, ja tem users, plants, reports, collaborators)

#### Producao — 8 registros de exemplo

| Tipo | Produto exemplo | Planta |
|------|----------------|--------|
| `fabricacao` | Corned Beef (existente) | Processamento |
| `tripas` | 1o Corte (calibrada salgada) | Abatedouro |
| `fracionamento` | Corned Beef 40g EU/UK | Processamento |
| `couro` | Pele Fresca de Bovino | Frigorifico |
| `mucosa` | Mucosa Intestinal de Bovina | Frigorifico |
| `heparina_bruta` | Salt Cake Heparina (Torta) | Processamento |
| `heparina_purificacao` | Heparina Sodica Crua (Massa) | Processamento |
| `raspa` | Aparas de Pele Conservada de Bovino | Processamento |
| `gelatina` | Gelatina de Origem Bovina Halal | Processamento |

Cada registro precisa de `customFields` populado com dados realistas baseados nos FMs analisados.

#### Embarque — 8 registros de exemplo (novos tipos)

| Tipo | Produto exemplo |
|------|----------------|
| `exportacao_industrializados` | Colageno Hidrolisado Halal |
| `venda_industrializados` | Pele Bovina Tratada (caleirada) |
| `transferencia_industrializados` | Carne Mecanicamente Recuperada |
| `venda_subprodutos` | Mucosa Intestinal de Bovina |
| `transferencia_in_natura` | Aparas de Couros Depilado |
| `transferencia_subprodutos` | Heparina |
| `transferencia_generica` | Colageno Hidrolisado Halal |
| `venda_subprod_couro` | Aparas de Pele Conservada |

### Verificacoes de Build

- [ ] `npm run build` sem erros no backend
- [ ] `npm run build` sem erros no frontend
- [ ] `npx prisma migrate deploy` aplica migration sem erros (com DB rodando)
- [ ] `npx prisma db seed` roda seed sem erros
- [ ] Swagger mostra novos valores de enum
- [ ] `GET /fm-metadata` retorna todos os 20 FMs
- [ ] `GET /production-reports?productionType=tripas` retorna filtrado
- [ ] `GET /shipping-reports?shippingType=exportacao_industrializados` retorna filtrado

### Testes Manuais

- [ ] Login como supervisor → criar relatorio de producao tipo `mucosa` → salvar rascunho → assinar
- [ ] Login como supervisor → criar relatorio de embarque tipo `venda_subprodutos` → salvar → assinar
- [ ] Download PDF de relatorio assinado (tipo `fabricacao` — existente, deve continuar funcionando)
- [ ] Sidebar mostra novos itens, navega corretamente
- [ ] Filtros na lista funcionam com novos tipos
- [ ] Coordenador ve relatorios read-only (sem botao criar)

### Tarefas

- [ ] Adicionar PlantType `quimico` (para Kin Master, Gelprime — empresas quimicas com CNPJ em vez de SIF)
  OU usar `processamento` com SIF opcional (decisao pendente — verificar se o modelo atual suporta CNPJ)
- [ ] Atualizar seed.ts com 8+ registros de producao especializada
- [ ] Atualizar seed.ts com 8+ registros de embarque novos tipos
- [ ] Popular `customFields` com dados realistas por tipo
- [ ] Popular `verificationItems` com verificacoes corretas por tipo (usar fm-metadata)
- [ ] Build completo backend + frontend
- [ ] Testes manuais se o DB estiver rodando

### Ordem de Execucao Recomendada

```
A.2.5 (Sidebar)  →  A.2.7 (Seed parcial + Build)  →  A.2.6 (PDFs)
```

**Justificativa**: A sidebar e rapida de implementar e permite testar navegacao.
O seed permite validar o pipeline de dados. Os PDFs sao a parte mais demorada
e dependem de dados de seed para testar — fazemos por ultimo.

Alternativa mais segura:
```
A.2.5 (Sidebar)  →  A.2.6 (PDFs fabricacao dispatch)  →  A.2.7 (Seed + teste completo)
```

---

## Estimativa de Complexidade

| Sub-fase | Arquivos | Complexidade | Notas |
|----------|----------|-------------|-------|
| A.2.5 | 1-2 (Sidebar + maybe List sync) | **Baixa** | Apenas editar array de navegacao |
| A.2.6 Producao | 9 novos + 2 editados | **Alta** | 8 templates PDF com layout distinto |
| A.2.6 Embarque | 1 editado | **Media** | Expandir template existente com condicionais |
| A.2.7 | 1 editado (seed.ts) + verificacao | **Media** | customFields realistas por tipo |

**Total estimado**: A.2.5 e rapida. A.2.6 e o maior esforco (PDF layout per FM). A.2.7 depende de DB rodando.
