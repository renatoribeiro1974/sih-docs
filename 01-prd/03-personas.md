---
title: Personas
parent: PRD
nav_order: 3
---

# 3. Personas e Jornadas

---

## 3.1 Personas do Sistema

O SIH atende 4 personas principais, mapeadas a partir dos papéis definidos no **PR 7.1 Rev 22, Seção 10.10**.

> **Nota**: O role `gestor` existia no enum original mas foi **removido na v1.0**. Gestores utilizam o HalalSphere para visão executiva. O SIH foca nos papéis operacionais.

---

### Persona 1: Supervisor Muçulmano

**Role no sistema**: `supervisor`

| Atributo | Descrição |
|----------|-----------|
| **Quem e** | Profissional muçulmano qualificado, designado pela FAMBRAS Halal para acompanhar processos industriais em plantas certificadas |
| **Onde trabalha** | Em campo - plantas de abate, produção industrial e embarque |
| **Dispositivo** | Tablet (uso principal) ou celular |
| **Conectividade** | Variavel - pode estar em areas com internet limitada |
| **Frequência de uso** | Diário, durante todo o turno de trabalho |
| **Volume** | 3-10 relatórios por dia, dependendo do tipo de planta |

#### Responsabilidades (PR 7.1 Sec. 10.10)

- Supervisionar processos de abate conforme requisitos Halal
- Acompanhar produção industrial de produtos certificados
- Verificar embarques para exportação e mercado interno
- Registrar não-conformidades e acompanhar ações corretivas
- Garantir rastreabilidade da carne Halal (origem, CSN, certificado)
- Assinar e declarar conformidade de cada relatório

#### Necessidades

- Preenchimento rápido e intuitivo (esta em campo, muitas vezes de pe)
- Interface grande e legivel (tela de tablet, uso com luvas possível)
- Formulários que guiem o preenchimento (campos condicionais, validações)
- Acesso rápido ao histórico de relatórios da planta
- Possibilidade de salvar rascunho e continuar depois

#### Dores Atuais

- Preencher formulários em papel e demorado e propenso a erros
- Números seriais precisam ser controlados manualmente
- PDFs se perdem ou ficam ilegives
- Sem feedback imediato sobre erros de preenchimento
- Demora para informar NCs a gestão

---

### Persona 2: Coordenador de Supervisores

**Role no sistema**: `coordenador`

| Atributo | Descrição |
|----------|-----------|
| **Quem e** | Profissional que coordena a equipe de supervisores, distribui escalas e revisa relatórios |
| **Onde trabalha** | Escritório + visitas a campo |
| **Dispositivo** | Desktop (escritório) e tablet (campo) |
| **Conectividade** | Estavel no escritório |
| **Frequência de uso** | Diário |
| **Volume** | Revisa 10-30 relatórios por dia |

#### Responsabilidades

- Distribuir supervisores entre as plantas (escala)
- Criar, editar e desativar supervisores e operadores
- Visualizar todos os relatórios das plantas sob sua coordenação (somente leitura)
- Cancelar relatórios quando necessário
- Gerênciar não-conformidades (verificar, encerrar)
- Garantir cobertura de todas as plantas
- **NÃO assina relatórios** — a assinatura é exclusiva do supervisor

#### Necessidades

- Visão consolidada de todos os relatórios (somente leitura)
- Gestão de escala com visualização por calendário
- CRUD de usuários (supervisores e operadores)
- Alertas de NCs próximas do vencimento (7 dias)
- Filtros e busca rápida por planta, supervisor, período

#### Dores Atuais

- Relatórios chegam com dias de atraso
- Dificuldade de montar escala otimizada
- Sem visibilidade de NCs pendentes em tempo real
- Consolidação manual de dados para reportar a gestão

---

### Persona 3: Operador

**Role no sistema**: `operador`

| Atributo | Descrição |
|----------|-----------|
| **Quem e** | Profissional de apoio que preenche relatórios sob orientação de um supervisor, mas não tem autoridade para assinar |
| **Onde trabalha** | Em campo - plantas de abate, produção industrial e embarque |
| **Dispositivo** | Tablet (uso principal) ou celular |
| **Conectividade** | Variavel |
| **Frequência de uso** | Diário |
| **Volume** | 3-10 relatórios por dia |

#### Responsabilidades

- Preencher relatórios de abate, produção e embarque
- Registrar dados operacionais com precisao
- Registrar não-conformidades
- **NÃO pode assinar relatórios** — salvam como rascunho para assinatura do supervisor

#### Necessidades

- Interface rápida e intuitiva (igual ao supervisor)
- Acesso a histórico de relatórios para referência
- Possibilidade de salvar rascunho

#### Dores Atuais

- Mesmas dores do supervisor quanto ao preenchimento em papel

---

## 3.2 Matriz de Permissões por Persona

| Funcionalidade | Supervisor | Operador | Coordenador | Admin |
|----------------|:----------:|:--------:|:-----------:|:-----:|
| Criar relatórios | Sim | Sim | - | Sim |
| Editar relatórios (rascunho) | Próprios | Próprios | - | Todos |
| **Assinar relatórios** | **Próprios** | **NÃO** | **NÃO** | **NÃO** |
| Cancelar relatórios | - | - | Sim | Sim |
| Visualizar relatórios | Próprios | Próprios | Todos (read-only) | Todos |
| Registrar NCs | Sim | Sim | Sim | Sim |
| Resolver NCs | Sim | Sim | Sim | Sim |
| Verificar/Encerrar NCs | - | - | Sim | Sim |
| Ver dashboard | Básico | Básico | Completo | Completo |
| Gerênciar escala | Ver própria | Ver própria | Sim | Sim |
| Gerênciar plantas | - | - | - | Sim |
| Gerênciar usuários | - | - | Sim (sup+oper) | Sim (todos) |
| Gerar PDF relatório | Sim | Sim | Sim | Sim |

---

## 3.3 Jornadas Tipicas

### Jornada do Supervisor - Dia Tipico em Planta de Abate

```
06:00 - Chega na planta, abre SIH no tablet
06:05 - Seleciona planta + turno, inicia novo relatorio de abate (FM 7.1.4.x)
06:10 - Preenche cabecalho (dados do degolador, especie)
06:15 - Inicia supervisao do abate
       → Verifica itens C/NC durante o processo
       → Registra contagens (total, aprovados, rejeitados)
       → Para bovinos: avalia insensibilizacao
11:00 - Primeira conferencia de stunning (bovino)
12:00 - Pausa - salva rascunho
13:00 - Retoma preenchimento
14:00 - Segunda conferencia de stunning
15:00 - Finaliza contagens e camaras de resfriamento
15:10 - Revisa, assina e envia relatorio
15:15 - Se NC identificada: registra NC com foto e descricao
15:30 - Fim do turno
```

### Jornada do Supervisor - Dia Tipico em Planta de Produção Industrial

```
07:00 - Chega na planta, abre SIH
07:05 - Inicia relatorio de producao (FM 7.1.3.1)
07:10 - Registra materias-primas carneas (SIF origem, CSN, certificado Halal)
07:20 - Registra ingredientes aprovados (nao-carneos)
07:30 - Inicia acompanhamento da producao
       → Verifica 5 itens C/NC (limpeza, materias-primas, documentacao, armazenamento, rotulagem)
12:00 - Registra dados do produto final (codigo, lote, quantidades)
12:10 - Envia relatorio de producao
14:00 - Acompanha embarque: inicia relatorio de embarque (FM 7.1.7.1)
14:10 - Preenche dados de exportacao (importador, container, lacre)
14:20 - Adiciona tabela de produtos
14:25 - Verifica 2 itens C/NC (exclusividade Halal, selo)
14:30 - Envia relatorio de embarque
```

### Jornada do Coordenador - Gestão Diária

```
08:00 - Abre dashboard, ve resumo geral dos relatorios
08:10 - Filtra relatorios assinados do dia anterior → revisa visualmente
08:15 - Identifica relatorio com NC pendente → abre NC para verificacao
08:30 - Ve 3 NCs proximas do prazo de 7 dias → contacta supervisores
09:00 - Monta escala da proxima semana no calendario
09:30 - Cadastra novo operador que iniciara na planta
10:00 - Gera PDFs de relatorios assinados para arquivo da auditoria
```
