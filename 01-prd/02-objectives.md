---
title: Objetivos e KPIs
parent: PRD
nav_order: 2
---

# 2. Objetivos e Métricas de Sucesso

---

## 2.1 Objetivo Principal

**Digitalizar 100% dos formulários FM da FAMBRAS Halal utilizados em campo**, eliminando o preenchimento em papel e permitindo rastreabilidade em tempo real das atividades de supervisão industrial.

---

## 2.2 Objetivos Específicos

### O1: Eficiencia Operacional
Reduzir o tempo de preenchimento e consolidação dos relatórios de supervisão.

| Métrica | Atual | Meta | Reducao |
|---------|-------|------|---------|
| Tempo de preenchimento por relatório | 15-25 min | 5-10 min | 60% |
| Tempo de consolidação (campo → escritório) | 2-5 dias | Tempo real | 100% |
| Tempo de busca de relatório específico | 10-30 min | < 10 seg | 99% |

### O2: Qualidade dos Dados
Eliminar erros de preenchimento e garantir consistencia dos registros.

| Métrica | Atual | Meta |
|---------|-------|------|
| Taxa de erros de preenchimento | ~15% | < 2% |
| Campos obrigatórios em branco | ~10% | 0% |
| Seriais duplicados | Ocorrem | 0 |
| Dados estruturados e pesquisaveis | 0% | 100% |

### O3: Rastreabilidade e Conformidade
Garantir rastreabilidade completa da cadeia de supervisão conforme PR 7.1 Rev 22.

| Métrica | Atual | Meta |
|---------|-------|------|
| Relatórios rastreávels por planta/data | Parcial (papel) | 100% digital |
| NCs com prazo monitorado (7 dias) | Não monitorado | 100% com alertas |
| Vínculo relatório ↔ supervisor ↔ planta | Manual | Automático |
| Histórico de alterações | Inexistente | Audit trail completo |

### O4: Visibilidade para Gestão
Fornecer dashboards e indicadores em tempo real para tomada de decisao.

| Métrica | Atual | Meta |
|---------|-------|------|
| Visibilidade de relatórios pendentes | Nenhuma | Tempo real |
| Indicadores de NCs ativas por planta | Inexistente | Dashboard |
| Produtividade por supervisor | Não medida | Métricas automáticas |
| Status de relatórios (rascunho/enviado/aprovado) | Não existe | Workflow completo |

---

## 2.3 KPIs do Sistema

### KPIs Operacionais

| KPI | Descrição | Meta |
|-----|-----------|------|
| **Relatórios/dia** | Media de relatórios preenchidos por dia no sistema | Baseline + monitoramento |
| **Tempo medio de preenchimento** | Tempo entre abertura e envio do relatório | < 10 min |
| **Taxa de aprovação** | % de relatórios aprovados sem rejeicao | > 90% |
| **NCs resolvidas no prazo** | % de NCs resolvidas dentro dos 7 dias | > 85% |
| **Cobertura digital** | % de relatórios feitos no SIH vs. papel | 100% em 3 meses |

### KPIs Técnicos

| KPI | Descrição | Meta |
|-----|-----------|------|
| **Uptime** | Disponibilidade do sistema | > 99.5% |
| **Tempo de resposta** | P95 de latencia das APIs | < 500ms |
| **Tempo de carregamento (mobile)** | First Contentful Paint em 3G | < 3s |
| **Erros de sistema** | Taxa de erros 5xx | < 0.1% |

---

## 2.4 Critérios de Sucesso do Projeto

### Curto Prazo (Lançamento)
- [ ] Todos os 8 tipos de formulário FM digitalizados e funcionais
- [ ] Login via Gestão de Certificações funcionando
- [ ] Pelo menos 1 planta piloto usando o sistema
- [ ] Zero erros críticos em produção

### Medio Prazo (3 meses após lançamento)
- [ ] 100% das plantas migraram do papel para o SIH
- [ ] Dashboard operacional sendo usado pela gestão
- [ ] Reducao mensurável no tempo de consolidação
- [ ] NCs com tracking automático de prazos

### Longo Prazo (6-12 meses)
- [ ] Módulo de inventário implementado (FM 7.1.5.x)
- [ ] Modo offline funcional para uso em campo sem internet
- [ ] Integração completa com Gestão de Certificações (dados compartilhados)
- [ ] Relatórios e analytics avancados
