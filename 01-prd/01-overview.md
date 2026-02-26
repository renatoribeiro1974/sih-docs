---
title: Visão Geral
parent: PRD
nav_order: 1
---

# 1. Visão Geral do Produto

**SIH - Supervisão Industrial Halal v1.0**

---

## 1.1 Problema que Resolvemos

A FAMBRAS Halal utiliza um conjunto de **formulários padronizados (FM)** para registrar as atividades de supervisão industrial em plantas de abate, produção e embarque. Esses formulários são a base documental que comprova a conformidade Halal dos produtos.

### Situação Atual

- **Formulários em papel/PDF**: Supervisores preenchem manualmente em campo, gerando problemas de legibilidade, extravio e demora na consolidação
- **Sem rastreabilidade em tempo real**: Gestores só tem visibilidade dos dados quando os formulários físicos chegam ao escritório
- **Dados não estruturados**: Informações ficam presas em PDFs, impossibilitando análises, dashboards e cruzamento de dados
- **Risco de inconsistência**: Sem validação automática, erros de preenchimento passam despercebidos
- **Dificuldade de auditoria**: Localizar um formulário específico entre centenas de papéis é lento é impreciso
- **Não-conformidades sem prazo**: Sem tracking sistemático, prazos de 7 dias corridos (PR 7.1) podem ser perdidos

### Impacto

| Métrica | Atual (Papel) | Esperado (SIH) |
|---------|---------------|-----------------|
| Tempo de preenchimento | 15-25 min | 5-10 min |
| Tempo de consolidação | 2-5 dias | Tempo real |
| Erros de preenchimento | ~15% | <2% |
| Rastreabilidade | Manual/parcial | 100% digital |
| Tracking de NCs | Planilha/email | Automático com alertas |

---

## 1.2 Solução Proposta

O **SIH (Supervisão Industrial Halal)** e um sistema web responsivo (PWA) que digitaliza todos os formulários FM da FAMBRAS Halal, otimizado para uso em **tablets em campo**.

### Diferenciais

- **Formulários digitais fiéis ao original**: Cada tipo de formulário mantém seus campos e itens de verificação C/NC específicos, garantindo conformidade com o PR 7.1
- **Validação em tempo real**: Campos obrigatórios, formatos e regras de negócio validados no momento do preenchimento
- **Número serial automático**: Formato `SIF/ANO/SEQUENCIAL` gerado automáticamente sem risco de duplicidade
- **Rastreabilidade completa**: Cada relatório vinculado a planta, supervisor, data e turno
- **Workflow de não-conformidades**: Registro, acompanhamento e verificação com prazos automáticos de 7 dias
- **Dashboard em tempo real**: Visão consolidada para gestores e coordenadores
- **Autenticação própria**: Login self-contained com bcrypt + JWT HS256 (independente do HalalSphere)
- **PWA preparado para offline**: Estrutura pronta para funcionar sem internet no futuro

---

## 1.3 Catálogo de Formulários FM FAMBRAS

O SIH digitaliza os seguintes formulários da FAMBRAS Halal, baseados no **PR 7.1 Rev 22**:

### Formulários v1.0 (Implementados)

| Família | FM | Nome Oficial | Módulo no SIH | Tipo |
|---------|-----|-------------|----------------|------|
| Abate | FM 7.1.4.1 | Rel. Abate Halal - Aves | SlaughterReport | species=ave |
| Abate | FM 7.1.4.2 | Rel. Abate e Controle de Carcaça - Bovino | SlaughterReport | species=bovino |
| Produção | FM 7.1.3.1 | Rel. Acomp. Fabricação Prod. Industrializados (Carne) | ProductionReport | regular |
| Produção | FM 7.1.8.x | Rel. Produção Especial | ProductionReport | specialProduction=true |
| Embarque | FM 7.1.7.1 | Rel. Acomp. Embarque - Exportação | ShippingReport | type=exportação |
| Venda | FM 7.1.7.4 | Rel. Venda Mercado Interno | ShippingReport | type=venda_interna |
| Transferência | DCPOA | Documento de Transferência | ShippingReport | type=transferência |
| NC | FM 7.1.6.1 | Checklist de Não-Conformidade | NonConformity | - |

### Formulários Futuros (Modelo de dados documentado)

| Família | FM | Nome Oficial | Módulo no SIH | Observação |
|---------|-----|-------------|----------------|------------|
| Inventário | FM 7.1.5.1 | Inventário Carne Halal (Conta Corrente) | MeatInventory | Recebido vs. utilizado |
| Inventário | FM 7.1.5.6 | Inventário Lotes de Produção (Batch) | BatchInventory | Gerado vs. transferido |
| Rotulagem | FM 7.1.3.6 | Inventário Acomp. Rotulagem | LabelingInventory | Sem rótulo → rotulado → embarcado |

---

## 1.4 Estrutura Padrão dos Formulários FM

Todos os formulários FAMBRAS compartilham uma estrutura padrão:

### Cabeçalho

1. **Identificação do formulário**: Número FM, revisão, data de emissão
2. **Logo FAMBRAS Halal**: Identidade visual institucional
3. **Número serial**: Formato `SIF/ANO/SEQUENCIAL` (ex: `451/2026/000001`)

### Identificação

4. **Planta industrial**: Nome, endereço, número SIF/SIE/SIM
5. **Supervisor**: Nome completo, registro/documento

### Corpo (específico por tipo FM)

6. **Dados operacionais**: Campos específicos de cada tipo de formulário
7. **Itens de verificação C/NC**: Checklist de conformidade/não-conformidade
   - **NÃO são genéricos** - cada tipo FM tem seus próprios itens fixos
   - Abate bovino: 14 itens (bem-estar animal, degola, sangria, insensibilização, etc.)
   - Produção industrial: 5 itens (limpeza, matérias-primas, documentação, armazenamento, rotulagem)
   - Embarque exportação: 2 itens (exclusividade Halal no container, selo de garantia)

### Encerramento

8. **Observações**: Campo texto livre
9. **Declaração do supervisor**: Texto padrão bilíngue (PT/EN)
10. **Assinatura**: Supervisor muçulmano + data

### Cancelamento

- Relatório cancelado recebe selo "CANCELADO" e motivo
- Novo relatório e emitido com sufixo "A" no serial (ex: `451/2026/000001A`)

---

## 1.5 Descobertas-Chave da Análise de Domínio

A análise detalhada dos formulários FM preenchidos e das planilhas de inventário revelou:

### Sobre os Relatórios

1. **Checklists NÃO são genéricos**: Cada tipo de relatório tem seus próprios itens C/NC fixos, definidos pelo PR 7.1. Não há templates configuráveis - os itens são constantes do sistema.

2. **Seção de insensibilização (stunning)** é exclusiva do abate bovino (FM 7.1.4.2):
   - Avaliação da pressão/parâmetros do equipamento
   - 2 conferências por turno: animal vivo no momento do abate + sem lesões craniais

3. **Tabela de produtos** é padrão em embarque/venda/produção:
   - Produto, código, lote, datas (abate/produção/válidade), embalagem, pesos, temperatura

4. **Matérias-primas cárneas** (exclusivo produção industrial):
   - Rastreabilidade completa: frigorífico de origem, SIF, data abate, CSN, certificado Halal

5. **Ingredientes aprovados** (exclusivo produção industrial):
   - Lista de insumos não-carneos verificados: fornecedor, lote, válidade

### Sobre o Inventário (Futuro)

6. **Conta corrente** é o conceito central do inventário:
   - Saldo = Entrada - Saída (recebido - usado / gerado - transferido / sem rótulo - rotulado)
   - Aplica-se aos 3 tipos de inventário (carne, lotes, rotulagem)

7. **Alto volume de dados**: Planilhas reais mostram 1.000+ linhas/mês em inventário de lotes, 8.000+ linhas por produto em rotulagem

8. **Referências cruzadas**: O inventário conecta toda a cadeia - do abate ao embarque - via certificados Halal, CSN, códigos de produto e números de lote

---

## 1.6 Ecossistema FAMBRAS e Autenticação

O SIH faz parte de um ecossistema de 3 sistemas que cobrem a cadeia Halal da FAMBRAS:

| Sistema | Função | Stack |
|---------|--------|-------|
| **HalalSphere** | Gestão de Certificações (upstream) — certifica empresas, auditorias, propostas | NestJS 11 + React 19, porta 3333 |
| **SIH** | Supervisão Industrial Halal (operacional) — relatórios de abate, produção, embarque | NestJS 11 + React 19, porta 3334 |
| **SysHalal** | Emissão de Certificados de Exportação (downstream) — PDFs, QR Code, cartas de correcao | NestJS 10 + Next.js 14 |

### Autenticação Própria (v1.0)

Na v1.0, o SIH opera com **autenticação self-contained** — NÃO depende do HalalSphere para autenticar:

- Backend SIH possui endpoint `POST /auth/login` próprio
- Senhas armazenadas com **bcrypt** (salt rounds 10)
- JWT emitido pelo próprio backend SIH com **HS256** + `JWT_SECRET`
- JWT contém: `sub` (userId), `email`, `name`, `role`
- Token válido por 7 dias (`JWT_EXPIRES_IN`)

### Dados Compartilhados (Futuro)

- Plantas industriais: `Plant.externalCompanyId` → Company do HalalSphere
- Colaboradores: `Collaborator.externalId` → futuro cadastro no HalalSphere
- Usuários: `SupervisorProfile.externalUserId` → User do HalalSphere

### Independência

- Banco de dados próprio (PostgreSQL separado, porta 5433)
- Backend próprio (NestJS 11, porta 3334)
- Frontend próprio (React 19, porta 5174)
- Deploy independente
