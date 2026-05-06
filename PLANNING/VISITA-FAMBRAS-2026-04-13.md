# Visita Tecnica FAMBRAS — 13 a 16 de abril de 2026
## Relatorio para SIH (Supervisao Industrial Halal)

> Ecohalal x FAMBRAS Halal (FAMBRAS e socia da Ecohalal).
> Visita presencial de Renato Ribeiro (PJ contratado, PO).
> Relatorio reconstituido em 2026-05-03 a partir de debrief
> verbal — pode haver itens adicionais quando relatorio formal
> for redigido.

---

## 1. Contexto

| Dia | Data | Equipe FAMBRAS | Foco principal para SIH |
|---|---|---|---|
| Segunda | 13/04 | Controladoria de frigorifico | Varios itens de melhoria sobre o que ja foi escrito do SIH |
| Terca | 14/04 | Industrializados (Lina) | Detalhes do processo (mais relevante para HalalSphere — vide relatorio especifico) |
| Quarta | 15/04 | Industrializados (continuacao) | Inventario — interface com SIH para registro |
| Quinta | 16/04 | Qualidade (Elaine) | Registro de relatorios, conformidade/NC, reclamacoes, seguranca de assinatura |

Documentos repassados por **Elaine (Qualidade)** e **Lina (Industrializados)** durante a semana.

---

## 2. Visao macro da FAMBRAS sobre o SIH

O **SIH foi avaliado como muito util** para:
- Registro de relatorios em campo
- Acompanhamento e gestao do andamento de processos produtivos
- Controle de relatorios de conformidade e nao-conformidade
- Relatorios de reclamacao

Em resumo: a FAMBRAS valida o conceito do SIH, mas ha gaps a
fechar antes do produto ser confiavel para uso pleno.

---

## 3. Pontos identificados pela Controladoria de frigorifico (13/04)

A equipe de controladoria de frigorifico identificou **varios
itens de melhoria** em relacao ao que ja foi documentado/
implementado do SIH.

> **Pendencia:** lista detalhada desses itens nao foi capturada
> no debrief — precisa ser recuperada do relatorio formal a ser
> redigido.

---

## 4. Pedidos da equipe de Qualidade (16/04)

### 4.1 Seguranca na assinatura de relatorios (alta prioridade)

A equipe de Qualidade chegou a solicitar explicitamente:

- **Geolocalizacao do aparelho** usado pelo supervisor no
  momento da assinatura do relatorio — para evidencia de que
  a assinatura aconteceu no local da supervisao.
- **Foto do supervisor** no momento da assinatura — para
  evidencia de identidade do signatario.

Ambas as medidas sao de **seguranca de auditoria**: garantir
que o relatorio nao foi assinado remotamente ou por outra
pessoa que nao o supervisor responsavel.

**Implicacao tecnica (inferencia minha — validar):**
- Solicitacao de permissao do navegador para geolocalizacao
  no momento da assinatura.
- Captura de foto via camera do dispositivo no momento da
  assinatura, anexada ao relatorio.
- Armazenamento dessas evidencias junto com o relatorio
  (S3 + metadados).
- Possivel impacto na PWA atual (camera + geolocation APIs).

### 4.2 Anexacao de documentos nos relatorios

- Permitir anexacao de documentos diretamente nos relatorios
  do SIH.
- Caso de uso: documentos comprobatorios, fotos de campo
  adicionais, planilhas de apoio, etc.
- Implica armazenamento (S3), UI de upload e auditoria.
- **Cruzamento com HalalSphere:** demanda similar foi feita
  para HalalSphere (anexacao por fase do processo de
  certificacao). Vale modelar uma solucao consistente entre
  os dois produtos.

### 4.3 Gestao de relatorios de conformidade e nao-conformidade

- A Qualidade gerencia **relatorios de NC** ja documentados no
  PR 7.1 e parcialmente implementados no SIH (FM 7.1.6.1
  Checklist NC).
- Tambem gerencia **relatorios de reclamacao** — esses
  provavelmente ainda nao estao no SIH.
- Validar se os relatorios de reclamacao sao um novo modulo
  ou uma extensao da estrutura atual de NC.

---

## 5. Inventario (industrializados, 14-15/04)

A equipe de Industrializados aprofundou a discussao sobre
inventario:
- Funcionamento atual: bem estruturado, mesmo manualmente;
  controle de documentacao adequado.
- Necessidade: digitalizacao do inventario (Carne, Lotes,
  Rotulagem) — ja existe em parte no SIH (Fatia 1 e 2 deployed
  em 12/04, Fatia 3 ainda nao iniciada).
- **Decisao operacional ventilada:** visitar uma planta da
  **JBS em Lins** para ver inventario funcionando em escala
  industrial maior.

---

## 6. Pendencias e promessas

| Item | Origem | Prioridade |
|---|---|---|
| Itens de melhoria da Controladoria de frigorifico | Dia 13/04 | alta — recuperar lista no relatorio formal |
| Geolocalizacao na assinatura | Qualidade, 16/04 | alta |
| Foto do supervisor na assinatura | Qualidade, 16/04 | alta |
| Anexacao de documentos nos relatorios | Qualidade, 16/04 | media-alta |
| Relatorios de reclamacao (validar como modulo) | Qualidade, 16/04 | media |
| Decisao sobre visita JBS Lins | Industrializados, 15/04 | media |
| Conclusao da Fatia 2 (validacao end-to-end) | Anterior a visita; ainda em aberto | alta — divida em prod |

---

## 7. Decisoes estrategicas necessarias com CEO+CTO Ecohalal

1. **Visita JBS Lins:** vale priorizar? Quando? Recurso de
   tempo do Renato.
2. **Reabrir vs estender escopo do SIH:** com seguranca de
   assinatura + anexacoes + relatorios de reclamacao +
   sprints 1-5 ja contratados, ha sinal de re-priorizacao do
   roadmap.
3. **Possivel re-visita a FAMBRAS:** alguns processos
   precisam imersao mais profunda.

---

## 8. Proximos passos sugeridos (rascunho — validar)

### Esta semana (2026-05-04 a 2026-05-10)
- [ ] Recuperar lista detalhada de melhorias da Controladoria de
      frigorifico (ler notas do dia 13/04 ou pedir a Elaine/Lina)
- [ ] Fechar testes da Fatia 2 (Inventario de Lotes) — divida em
      prod desde 12/04
- [ ] Decidir prioridade: continuar Sprints 1-5 ou inserir
      seguranca de assinatura na frente?

### Proximas 2-3 semanas
- [ ] Implementar geolocalizacao + foto na assinatura
- [ ] Implementar anexacao de documentos nos relatorios
- [ ] Validar com Qualidade FAMBRAS

### Mes seguinte
- [ ] Revisitar roadmap do SIH a luz das demandas da visita
- [ ] Definir modulo de relatorios de reclamacao (se for novo
      modulo)
- [ ] Decisao final sobre Fatia 3 (Rotulagem) e visita JBS

---

## 9. Itens nao cobertos por este relatorio

- Lista detalhada dos itens levantados pela Controladoria de
  frigorifico (13/04) — recuperar.
- Documentos especificos repassados por Elaine e Lina — listar
  quando consolidados.
- Estimativa de esforço por demanda — fazer no proximo
  planejamento.

---

## 10. Itens deste relatorio que sao do HalalSphere

Para evitar duplicacao, os pontos da visita que sao escopo do
HalalSphere (cadastro de produtos com ingredientes, fluxo de
certificacao, QR code com prazo 20/05, escopo "ERP halal")
estao em relatorio separado:
`halalsphere-docs/PLANNING/VISITA-FAMBRAS-2026-04-13.md`.
