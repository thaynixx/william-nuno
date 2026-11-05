# Como a análise "Blog como gerador de demanda" foi gerada

Data da extração: 04/11/2026. Janela principal: 01/11/2024 a 04/11/2026 (24 meses). Novembro de 2026 é parcial.

Este documento lista o que foi consultado, com que filtros, o que cada resultado devolveu, como cada número da página foi derivado e o que é hipótese minha e não dado medido. Nada foi escrito ou alterado no HubSpot, no GA4, no Canva ou no Miro como parte desta análise (só leitura). Consultas ao HubSpot e ao GA4 foram feitas pelas integrações conectadas à sua conta.

A página publicada: https://claude.ai/artifact/VPjECUivSHCYJAghyaRfaw (versão 3).

---

## 1. Premissas que eu assumi (confirme se alguma está errada)

| Premissa | Por quê |
|---|---|
| "Análise de 2" = 24 meses | Você disse que os 90 dias não bastavam e pediu "análise de 2". Não sabia se eram 2 meses, 2 anos ou as duas versões da página de demo. Usei 24 meses e abri por mês, para qualquer recorte derivar daí |
| As três fontes de captação são os valores **Web Request**, **Qualified** e **App Signup** da propriedade `leadsource` do contato | Esses três valores existem na lista de opções de `leadsource` |
| "Veio do blog" = a URL contém `/blog` | Cobre `pipefy.com/blog/...` (inglês) e `pipefy.com/pt-br/blog/...` (português). Não filtrei por domínio no HubSpot, então qualquer URL com `/blog` entra |
| First Page Seen = `hs_analytics_first_url` e Last Page Seen = `hs_analytics_last_url` | São os campos nativos do HubSpot com esses rótulos |
| "Conversão" no HubSpot = o contato existir com uma dessas três fontes e avançar de estágio | Não achei outra definição na conta |
| O padrão de apresentação é o do arquivo `painel-inbound-set-vs-out.html` que você enviou | Copiei a estrutura (resposta em uma frase, resumo, passos numerados, tabelas por etapa, insights, método) |

---

## 2. O que foi consultado

### 2.1 HubSpot (somente leitura)

Ferramentas usadas: `search_properties`, `get_properties`, `discover_hubspot_schema`, `query_crm_data` (SQL do HubSpot). Não usei nenhuma ferramenta `manage_*`.

**Descoberta de campos**

- `search_properties` em CONTACT por first url, last url, first page, last page, original source. Achei:
  - `hs_analytics_first_url` = "First Page Seen"
  - `hs_analytics_last_url` = "Last Page Seen"
  - `hs_analytics_source` = "Original Traffic Source"
  - `leadsource` = "Lead Source"
  - Existem também `first_page` ("desativar"), `user_first_page` e `previous_page`, que **não usei**.
- `get_properties` de `leadsource`: confirmou os valores `Web Request`, `Qualified` e `App Signup`, e também `Blog`, `Content`, `Meta`, `Call Cup`, entre muitos outros.
- `search_properties` por datas de entrada em estágio. Usei:
  - `hs_v2_date_entered_marketingqualifiedlead` (MQL)
  - `sales_accepted_stage_date` (SAL, campo customizado do contato)
  - `hs_v2_date_entered_salesqualifiedlead` (SQL)
  - `hs_v2_date_entered_opportunity` (Oportunidade)
  - `hs_v2_date_entered_customer` (Cliente)
- `discover_hubspot_schema` (GET_PIPELINES, DEAL): pipeline **Inside Sales Inbound (id 924876629)**, com estágios Lead (1418175839), MQL (1416613216), SAL (1416613217), SAL Connected (1430168675), No Show (1416613231), SQL Converted (1416613221) e Disqualified (1416613222).
- Para negócios usei as propriedades `hs_v2_date_entered_<id do estágio>`. Esse padrão de nome eu **deduzi**, não confirmei em `search_properties`. A consulta aceitou e devolveu dados coerentes.

**Consultas de contato que entram na página** (todas com `createdate >= '2024-11-01'`, `leadsource IN ('Web Request','Qualified','App Signup')`, `hs_analytics_first_url LIKE '%/blog%'`):

| O que mede | Agrupamento | Resultado |
|---|---|---|
| Contatos com blog como First Page Seen | `leadsource` | Web Request 192, App Signup 474, Qualified 10 (total 676) |
| Com MQL (`hs_v2_date_entered_marketingqualifiedlead IS NOT NULL`) | `leadsource` | WR 139, AS 156, Q 9 |
| Com SAL (`sales_accepted_stage_date IS NOT NULL`) | `leadsource` | WR 31, AS 67, Q 7 |
| Com SQL (`hs_v2_date_entered_salesqualifiedlead IS NOT NULL`) | `leadsource` | WR 9, AS 3, Q 2 |
| Com Oportunidade | `leadsource` | WR 50, AS 20, Q 3 |
| Com Cliente | `leadsource` | WR 4, AS 6, Q 0 |
| Original Traffic Source | `hs_analytics_source` | Organic Search 445, Offline 95, Direct 63, Paid Search 37, AI Referrals 14, Referrals 10, Social 8, Email 2, Other Campaigns 2 |
| Idioma (`LIKE '%/pt-br/blog%'`) | `leadsource` | Qualified 9, WR 125, AS 375 |
| E-mail de nutrição (`LIKE '%hs_automation%'`) | `leadsource` | WR 1, AS 73 |
| Clique pago (`LIKE '%gclid%'`) | `leadsource` | WR 36, AS 1 |
| Estágio atual dos 36 WR com `gclid` | `lifecyclestage` | Desqualificado 33, Lead 2, MQL 1 |
| Temas: `LIKE '%compras%'`, `'%agent%'`, `'%tutorial%'`, `'%bpm%'`, cada um combinado com `LIKE '%/blog%'` | `leadsource` | compras 4+23+32=59; agent 1+5+49=55; tutorial 1+7+18=26; bpm 0+3+13=16 |
| Mês de criação | `DATE_TRUNC(createdate,'MONTH')`, `leadsource` | Série mensal do gráfico |
| URLs distintas | `hs_analytics_first_url` | WR: 153 distintas (110 sem `?`); AS: 288 (202 sem `?`); nenhuma URL limpa com mais de 1 contato |
| Lista de URLs dos Qualified | `hs_analytics_first_url` | 10 URLs (9 em PT, 1 em EN) |

**Consultas de negócio que entram na página** (`FROM DEAL WHERE pipeline = '924876629' AND CONTACT.hs_analytics_first_url LIKE '%/blog%' AND createdate >= '2024-11-01'`, agrupadas por `lead_source` do negócio):

| Etapa | Filtro adicional | Resultado |
|---|---|---|
| Deals | nenhum | WR 13, AS 39, Q 5, Call Cup 4, Unassigned 1, Blog 1, Content 1, Meta 1 |
| MQL | `hs_v2_date_entered_1416613216 IS NOT NULL` | WR 7, AS 23, Q 3, Content 1, Call Cup 4 |
| SAL | `..._1416613217` | WR 9, AS 21, Q 1, Unassigned 1, Blog 1, Content 1, Meta 1, Call Cup 4 |
| Conectado | `..._1430168675` | WR 3, AS 3, Q 1, Blog 1 |
| SQL Converted | `..._1416613221` | WR 1, AS 1, Q 1 |
| No Show | `..._1416613231` | vazio |
| Desqualificado | `..._1416613222` | WR 8, AS 30, Q 4, Unassigned 1, Content 1, Meta 1, Call Cup 4 |
| Por trimestre (`DATE_TRUNC(createdate,'QUARTER')`) | nenhum | 3º tri/2026: 41; 4º tri/2026: 24 |
| SQL por trimestre | `..._1416613221` | 3º tri/2026: 3 |

O filtro `CONTACT.xxx` no `WHERE` de uma consulta de DEAL funcionou (devolveu um subconjunto, não o total). O resultado é "negócios em que algum contato associado tem blog como First Page Seen".

**Consultas feitas, mas que não aparecem na versão 3 da página** (existiam na versão 2):

- Total de contatos por fonte, sem filtro de blog: App Signup 313.734, Web Request 15.484, Qualified 1.981.
- Estágio atual do ciclo de vida, blog e média da fonte, por fonte.
- **Last Page Seen** com `/blog`: Qualified 4, Web Request 101, App Signup 323.
- Blog como primeira **e** última página: WR 16, AS 83.
- Blog só como última página (`hs_analytics_first_url NOT LIKE '%/blog%'`): Q 4, WR 85, AS 240.

Você pediu First e Last Page Seen. **A versão 3 não mostra Last Page Seen.** Tirei porque a página estava confusa e o HubSpot não diferencia "voltou ao blog antes de converter" de "leu depois de converter". Os números acima estão aqui para você decidir se voltam.

### 2.2 GA4 (via Supermetrics, somente leitura)

Fonte: `GAWA` (Google Analytics 4), property **224213526 "Pipefy - Official"**. Existem outras três properties conectadas (Guidelines Bricks, Pipefy Wiki, use.pipefy.com Webflow LPs) que não usei.

**Descoberta**

- `data_source_discovery` e `accounts_discovery`: GA4 autenticado, 4 properties.
- `field_discovery`: achei `pageReferrer`, `pagePathWithoutQueryString`, `screenPageViews`, `yearMonth`, `eventName`, `eventCount` e as dimensões customizadas `c_customevent_previous_page` e `c_customevent_previous_page_url`.
- Páginas encontradas por busca de caminho: demo em `/pt-br/agendar-uma-demonstracao`, `/pt-br/agendar-uma-demo` e `/book-a-demo` (com e sem barra final); cadastro em `/pt-br/cadastro`.

**Consultas que entram na página**

| O que mede | Parâmetros principais | Resultado |
|---|---|---|
| Pageviews de demo vindos do blog, por mês | `yearMonth`, `screenPageViews`, filtro `pageReferrer =@ /blog` e caminho = demo | 270 no total, 17 deles em jul/2025 |
| Pageviews de cadastro vindos do blog, por mês | idem, caminho `/pt-br/cadastro` | 27 no total (jul/26: 9, os demais entre 1 e 5) |
| Páginas do blog de origem | `pageReferrer`, `screenPageViews`, `max_rows 120` | Lista dos posts (ver abaixo) |
| Origem de desenvolvimento | `pageReferrer =@ lndo.site` | 17 pageviews de `pipefy.lndo.site/blog/...` |
| Pageviews mensais de demo (todas as origens) | `yearMonth`, caminho = demo | De 14–22 mil/mês no início de 2025 (pico 68.756 em jun/2025) a 3.630–4.960/mês em jul–out/2026 |

**Consultas feitas, mas que não aparecem na versão 3**

- `previous_page` e `previous_page_url` nos últimos 90 dias (06/08 a 04/11/2026). Com `previous_page` do blog, nas páginas de demo: `book_a_demo_pageview` 38, `step_1` 4, `step_2` 5, `step_3` 5, `step_4` 28, `step_5_typ` 3. Total do site no mesmo período: 5.503 pageviews de demo e 281 `step_5_typ`.
- Divisão por página de demo (você pediu para não ter essa granularidade, então saiu).
- A mesma consulta de `previous_page` na janela de 24 meses devolveu **só** `book_a_demo_pageview` (70) e, em outra variação, resultado vazio. Os 90 dias devolveram o funil completo. Não consegui explicar a divergência e por isso não uso conversão do GA4 na versão 3.

---

## 3. Como cada número da página foi derivado

| Número na página | Derivação |
|---|---|
| 676 contatos | 192 + 474 + 10 |
| 280 pageviews em formulários | (270 − 17 de dev) = 253 de demo, + 27 de cadastro |
| 93% entre maio e outubro/26 | (21+45+52+34+43+65) = 260; 260 ÷ 280 |
| 43 pageviews por mês | 260 ÷ 6 meses |
| 61 negócios | 13 + 39 + 5 + 4 (Call Cup excluído) |
| 34 MQL / 35 SAL / 8 conectados / 3 SQL / 45 desqualificados | Soma por linha da tabela de negócios, sem Call Cup |
| 56% / 57% / 13% / 4,9% | 34, 35, 8 e 3 divididos por 61 |
| 74% desqualificados | 45 ÷ 61 |
| Porcentagens do funil do contato (72%, 26% etc.) | Cada contagem ÷ contatos da própria linha |
| 304 / 105 / 14 / 73 / 10 | Soma das três fontes em cada etapa |
| 156 Web Requests "sem P-max" (versão 2) | 192 − 36 |
| 74 contatos com `hs_automation` | 73 App Signup + 1 Web Request |
| 25% de atribuição ruim | (74 + 95 Offline) ÷ 676. **Pode haver sobreposição** entre os dois grupos, que não verifiquei |
| 17 posts com 3 pageviews; "mais de 70 posts com 1 ou 2" | Contei **à mão** na lista do GA4. A consulta foi cortada em 120 linhas, então o fim da cauda é maior do que aparece |
| Tabela dos 10 posts | Os 10 maiores da lista do GA4, sem os 16 de `pipefy.lndo.site/blog/ai-business-process-automation/` |
| 14–22 mil → ~4 mil pageviews de demo | Série mensal do GA4; "queda de cerca de 75%" é aproximação |

---

## 4. Consultas que falharam ou que descartei

| O que tentei | O que aconteceu | O que fiz |
|---|---|---|
| `associations.DEAL IS NOT NULL` em CONTACT | Devolveu o total sem filtrar (era um filtro sem efeito) | Descartei. Passei a filtrar DEAL pelo contato associado |
| `num_associated_deals` | "Propriedade oculta, não pode ser consultada" | Descartei |
| Padrões com `%` no meio (`'/blog/%compras%'`) | Não suportado em consultas agregadas | Troquei por dois `LIKE` com coringa só nas pontas |
| Regex em `pageReferrer` no GA4 | Retornou vazio | Usei `=@` (contém) |
| Primeira tentativa de filtrar o blog no GA4 com `=@ pipefy.com/pt-br/blog` | Funcionou, mas só para português; `=@ /blog` pegou também `pipefy.lndo.site` | Descobri o ambiente de dev e o descontei |
| Duas buscas de propriedades com JSON malformado | Erro de entrada | Refiz com a sintaxe correta |

---

## 5. O que é hipótese minha, não dado medido

Estas frases estão na página e **não foram verificadas**:

1. **"O pico de 58 App Signups em nov/2025 coincide com e-mails de nutrição."** Vi 74 contatos com `hs_automation` no First Page Seen, mas não cruzei por mês. É uma suposição.
2. **"Alguma coisa mudou em maio de 2026, provavelmente CTAs nos posts."** O que medi é o salto de pageviews de formulário vindos do blog (0–3/mês até abril, 21 em maio). A causa é palpite.
3. **"Anúncio P-max."** Os 36 Web Requests têm `gclid` na URL. Em uma amostra das URLs apareciam `utm_content=p-max` e a campanha `US_INB_Google_DEC_PMax...`. **Não confirmei que os 36 são todos P-max.** Podem ser outros anúncios Google.
4. **"Os 74 contatos com `hs_automation` já existiam e clicaram em e-mail de nutrição."** É a leitura mais provável do parâmetro `utm_source=hs_automation` com campanhas "welcome nurturing", mas não confirmei que cada um já existia antes do clique.
5. **"Posts de modelo, diagrama e símbolo tendem a ter intenção de ferramenta."** Observação de padrão nos títulos, sem teste.
6. **"Offline Sources = criados por API ou importação."** É a definição do HubSpot, mas não verifiquei como esses 95 contatos foram criados.
7. **"O tráfego da demo caiu ~75% sem causa identificada."** Os números vêm do GA4. Não investiguei se é queda real, mudança de URL, de medição ou bots.

---

## 6. Limites que afetam a leitura

- **Estágio do ciclo de vida é o atual**, não o de um prazo fixo. Contatos recentes tiveram menos tempo para avançar.
- **Contato e negócio usam definições diferentes de lead source.** A tabela de negócios agrupa pelo `lead_source` do **negócio**. A tabela de contatos agrupa pelo `leadsource` do **contato**. Os dois nem sempre coincidem.
- **Os negócios do pipeline Inbound existem só a partir de julho de 2026** nessa consulta, embora o filtro comece em nov/2024. Não investiguei se o pipeline é novo ou se negócios antigos foram migrados.
- **Não filtrei por time (BR).** O seu painel Inbound BR filtra. O meu não. Não achei o campo de time.
- **Qualified tem 10 contatos e 5 negócios.** Qualquer taxa nessa linha é indicativa.
- **As etapas não são sequenciais.** Há mais oportunidades que SAL no Web Request (50 contra 31), porque o comercial cria o negócio direto. Por isso as colunas de funil são "passou pela etapa", não "restou na etapa".
- **GA4 e HubSpot medem coisas diferentes.** Pageviews vindos do blog (GA4) e contatos com blog como First Page Seen (HubSpot) não devem ser divididos entre si.
- **A ferramenta de consulta do HubSpot respondeu "report section created successfully" em cada `query_crm_data`.** Não sei se isso gera algum objeto visível na sua conta HubSpot. Se aparecer algo estranho em relatórios, foi isso.
- O GA4 contém tráfego de ambientes de desenvolvimento (`pipefy.lndo.site`, `pipefy-old.lndo.site`, `smoke-www.pipefy.com`). Descontei só os que apareceram na origem blog → formulário. O ambiente de dev não foi filtrado nas demais consultas de pageview.

---

## 7. Fora do escopo desta análise

- O fluxograma do Canva ("Fluxograma - Blog") foi lido e replicado no Miro antes desta análise, no board `uXjVERtDFv0=`.
- O deck de slides (https://claude.ai/artifact/Mio1Bd186VQBkjGjTY3UhN) é uma versão antiga, só com a janela de 90 dias do GA4. Não reflete o conteúdo atual.
