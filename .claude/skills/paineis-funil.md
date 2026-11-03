---
name: paineis-funil
description: Gera e atualiza os painéis do funil Pipefy PT-BR (fechamento mensal e monitor semanal). Use com `/paineis-funil mensal` ou `/paineis-funil semanal`.
---

# Painéis do funil Pipefy PT-BR

Quem pede: Tha (Thays Vitória, CRO & Web Analytics, responde à Ana Morisson, líder de Growth).

## Regras gerais

- Português do Brasil, acentuação correta, registro direto e informal. Sem travessão longo. Sem elogios.
- Todo número vem de consulta rodada nesta sessão. Nunca use número de memória.
- Não afirme causa. Separe "dá para afirmar", "não dá para afirmar" e "hipótese".
- Painéis são breves. Uma linha de método no rodapé basta.
- Use os nomes de campo do HubSpot exatamente como são.
- HTML autocontido, tema claro (igual aos templates em `claude/templates/`). Reconstrua do zero a cada vez.
- Texto para a Ana: ultra conciso, primeira pessoa, um próximo passo definido.

## Fontes e filtros

### HubSpot (MCP, ferramenta query_crm_data)

Chame `tool_guidance` antes da primeira consulta de cada sessão.

Pipelines de deals:
- Inside Sales Inbound: `924876629`. Etapas: Lead `1418175839`, MQL `1416613216`, SAL `1416613217`, SAL Connected `1430168675`, No Show `1416613231`, SQL Converted `1416613221`, Disqualified `1416613222`.
- Inside Sales Outbound: `924877607`. Etapas: Prospect, SAL `1416667205`, Connected `1430155913`, SQL `1416667209`. Sem MQL.

Filtro de time: `team = 'BR'` (o rótulo no HubSpot é "BR/RoW").

Propriedades: `lead_source`, `team`, `createdate`, `company_size__c`, `enrich_harvest_api_company_size`, `ldr_source`, `migration`.
- Entrada na etapa: `hs_v2_date_entered_<id da etapa>`.
- Enriquecido (definição da Ana): `ldr_source IS NOT NULL`.

Lead sources que importam: App Signup, Web Request, Qualified, Meta, LinkedIn Lead Gen, Call Cup, Content, Trade Show, Integrations, Employee Referral, Linkedin, Network Referral, Pipefy Event.

Limites do query_crm_data:
- Sem alias, CASE, DISTINCT ou HAVING.
- Filtro de data casa por dia.
- O filtro `migration = 'false'` junto com GROUP BY devolve vazio. Omita.
- Padrão que funciona: `SELECT DATE_TRUNC(campo, 'DAY'), team, COUNT(*) FROM DEAL WHERE pipeline = '...' AND lead_source = '...' AND campo BETWEEN 'AAAA-MM-DD' AND 'AAAA-MM-DD' GROUP BY DATE_TRUNC(campo, 'DAY'), team`. Some as linhas de team BR.

Eventos que contaminam comparações:
- 31/08: migração do HubSpot criou 1.289 deals de Signup e 302 de Web Request. Janelas começam em 07/09 por isso.
- 15/09: 716 deals de Call Cup de uma vez. Fora de todas as contas.
- Antes de qualquer janela nova, rode contagem de deals por dia. Dia com mais de 3x a mediana e lote ou migração: investigue e pergunte a Tha.

### GA4 pelo Supermetrics (fonte GAWA)

Propriedade: Pipefy - Official, id `224213526`. Fuso: `America/Sao_Paulo`.

**Filtro PT-BR (reproduz o Looker Studio da Tha, validado em 03/11/2026 com diferenca de 13 sessoes):**
```
fullPageUrl =@ use.pipefy.com OR fullPageUrl =@ www.pipefy.com/pt-br
```

Campos uteis: `date`, `hostName`, `pagePath`, `fullPageUrl`, `sessions`, `activeUsers`, `newUsers`, `eventName`, `eventCount`, `sessionDefaultChannelGrouping`.

Eventos: `sign_up` (Signup), `begin_sign_up`, `signupSteps`, `book_a_demo_pageview`, `book_a_demo_step_1` a `book_a_demo_step_5_typ`, `book_a_demo_step_5_skip`, `book_a_demo_submit_lp`, `form_submit`, `qualified_event`.

Book a Demo = `book_a_demo_submit_lp` + `book_a_demo_step_2`.

Nota: `activeUsers` e non-aggregatable. Nao some valores diarios para obter o total semanal; consulte a semana inteira em uma unica query.

Calibracao validada (semana 28/09 a 03/10, segunda a sabado): 48.392 sessoes (planilha: 48.405).

## Tarefa A: fechamento mensal

Argumento: `/paineis-funil mensal` ou `/paineis-funil mensal nov` (para novembro).

### Janelas
- Mes atual contra mes anterior, dia 07 ao dia 26 de cada mes (20 dias).
- Etapas (MQL, SAL, Conectado, SQL) contam deals criados na janela que entraram na etapa ate 8 dias depois do fim da janela.
- Para o proximo mes, use as janelas de dia 07 a 26 a menos que a Tha defina um evento.

### Consultas a rodar (nesta ordem)

1. Verificacao de anomalias: deals por dia na janela, ambos os pipelines, team BR. Dia com mais de 3x a mediana e alerta.
2. Funil por lead source, Inbound + Outbound somados: deals, SAL, Conectado, SQL.
3. Funil por lead source, so Inbound: deals, MQL, SAL, Conectado, SQL.
4. Signup e Web Request por Company Size (Inbound): deals, MQL, SAL, Conectado, SQL.
5. Enriquecidos (ldr_source preenchido), Signup: deals, MQL, SAL, Conectado, SQL.
6. GA4: sessoes, usuarios, sign_up, Book a Demo, qualified_event por dia na janela.

### Paineis a gerar

1. `claude/paineis/painel-funil-MES_A-vs-MES_B.html`: funil por Lead Source (template: `claude/templates/painel-funil-set-vs-out.html`).
2. `claude/paineis/painel-funil-signup-web-request-MES_A-vs-MES_B.html`: Signup e Web Request por Company Size e enriquecimento (template: `claude/templates/painel-funil-signup-web-request.html`).

### Regras de analise
- Compare deals e SQL somando os dois pipelines. Um canal pode mudar de pipeline entre meses.
- Confira cada percentual antes de escrever.
- Quando o numero for pequeno, calcule a chance de ver o resultado por acaso (Poisson).
- Nao escreva "nao foi demanda" sem separar volume.
- Concilie numeros que nao batem com a planilha da Tha por escrito no painel.

### Entrega
- HTMLs reconstruidos do zero.
- Tres linhas de resumo para a Ana (o que mudou, o que e incerto, o proximo passo).
- Commit e push no repositorio.

## Tarefa B: monitor semanal

Argumento: `/paineis-funil semanal` ou `/paineis-funil semanal 2026-11-03` (para a semana que termina nessa data).

### Rotina

1. Defina a semana fechada: segunda a domingo.
2. GA4 por dia: sessions, activeUsers, sign_up, Book a Demo, qualified_event. Filtro PT-BR (fullPageUrl).
3. HubSpot por dia, Inbound, team BR:
   - Deals criados por createdate: lead_source App Signup, Web Request e Qualified.
   - Entradas em MQL, SAL e SQL Converted por data de entrada na etapa, separando App Signup e Web Request.
   - Para Signup, entradas em MQL e SAL com ldr_source IS NOT NULL (enriquecidos).
4. Agregue por semana, acrescente ao CSV `claude/data/historico-semanal.csv`.
5. Calcule comparativos: (a) semana anterior, (b) mesma semana 4 semanas atras, (c) mediana das 4 semanas anteriores.
6. Gere o HTML e o resumo para a Ana.

### Colunas do CSV

`semana_inicio,semana_fim,sessoes,usuarios,signup_ga,demo_ga,qualified_ga,signup_deals,wr_deals,qualified_deals,signup_mql,wr_mql,signup_sal,wr_sal,signup_sql,wr_sql,signup_mql_enriq,signup_sal_enriq`

### Metricas e taxas

- Volumes: todas as colunas acima.
- Taxas: signup_ga/sessao, demo_ga/sessao, signup_mql/signup_deals, signup_sal/signup_mql (separada em enriquecido e nao enriquecido).
- Ligacao GA4-HubSpot: signup_deals/signup_ga, wr_deals/demo_ga. Variacao de 15%+ em qualquer sentido e alerta.

### Regra de situacao

- Vermelho: queda de 15%+ contra a semana anterior E de 10%+ contra a mediana.
- Atencao: queda de 15%+ em uma das duas comparacoes.
- So monitorar: contagem abaixo de 40 na semana (SQL, qualified_ga). Percentual em numero pequeno e ruido.
- Ligacoes GA4-HubSpot: alerta em variacao de 15%+ nos dois sentidos.
- Feriado: marque a semana e nao compare sem avisar.

### Entrega
- HTML em `claude/paineis/monitor-semanal-AAAA-MM-DD.html`.
- CSV atualizado.
- Resumo para a Ana.
- Commit e push.

## Verificacoes antes de entregar

1. Regressao do HubSpot: confira os totais contra os numeros de referencia abaixo.
2. Semanal: soma dos dias bate com a semana, semana tem 7 dias, historico e consistente.
3. HTML: tags balanceadas, nenhum "undefined" ou "NaN", nenhuma secao duplicada, datas em dd/mm.
4. Releia cada afirmacao contra a tabela. Afirmacao sem numero que a sustente sai.

### Numeros de referencia (resultados de 03/11/2026)

Janela 07/09 a 26/09 (corte 04/10): Signup 3.722 deals, 1.093 MQL; Web Request 459 deals, 326 MQL, 76 SQL; Qualified 374 deals, 278 MQL, 28 SQL; LinkedIn Lead Gen Outbound 613 deals, 38 SQL.

Janela 07/10 a 26/10 (corte 03/11): Signup 3.261 deals, 2.045 MQL; Web Request 62 SQL; LinkedIn Lead Gen Inbound 629 deals, 15 SQL. MQL total Inbound: 3.662 out, 1.830 set.

Enriquecidos (ldr_source preenchido), Signup, outubro: 1.253 deals, 884 MQL, 331 SAL, 33 conectados, 1 SQL.

GA4 calibracao (filtro fullPageUrl, semana 28/09-03/10, seg-sab): 48.392 sessoes.
