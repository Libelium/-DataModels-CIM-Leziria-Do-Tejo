# Payment Analytics

Analítica do consumo com cartão (compras e levantamentos de numerário) nos territórios da Lezíria do Tejo, a partir dos dados agregados e anonimizados que a SIBS Analytics entrega à CIMLT.

## Modelo de dados

| Modelo | Referência | Schema | Exemplos |
|---|---|---|---|
| `CardConsumptionObserved` | Modelo próprio (proposta). Não existe no CNMD nem nos Smart Data Models. Segue o padrão das observações agregadas `*Observed` dos Smart Data Models (por exemplo `TrafficFlowObserved`). | [`schema/CardConsumptionObserved.json`](schema/CardConsumptionObserved.json) | mensal: [normalizado](exemplos/CardConsumptionObserved-monthly-normalized.jsonld) · [key-values](exemplos/CardConsumptionObserved-monthly-keyvalues.json)<br>semanal: [normalizado](exemplos/CardConsumptionObserved-weekly-normalized.jsonld) · [key-values](exemplos/CardConsumptionObserved-weekly-keyvalues.json) |

`CardConsumptionObserved` representa uma **observação agregada**: uma célula de um cubo estatístico com seis dimensões (período, zona, setor económico, tipo de dia, país e concelho de origem do cartão) e seis medidas (número de cartões, número de operações, valor e três médias). **Nunca é uma transação individual**: a SIBS entrega contagens e valores já consolidados e anonimizados. Cada linha do ficheiro de origem é uma entidade.

> Este tipo **não faz parte do CNMD**, que não cobre analítica económica. É proposto como modelo novo, a validar com a ARTE.

## Origem dos dados

| | |
|---|---|
| **Fonte** | SIBS Analytics, relatório «Dados de Consumo CIM» (Report 6): ficheiro CSV semanal ou mensal, com uma linha por célula do cubo, agregado e anonimizado na origem |
| **Integração** | Fluxo Node-RED: formulário de carregamento do CSV → validação e transformação → envio de cada célula como medição ao IoT Agent da plataforma iris360, que a publica no *context broker* NGSI-LD |
| **Frequência** | A cada entrega da SIBS (semanal ou mensal). Carga idempotente: o identificador deriva das dimensões da célula e reenviar o mesmo ficheiro atualiza as mesmas entidades |
| **Âmbito** | Uma entidade por célula, publicada no espaço de dados do município da zona observada. As zonas sem município identificado são publicadas no espaço de dados da CIM |

## Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:CardConsumptionObserved:<período>-<zona>-<setor>-<tipo de dia>-<país>-<concelho>` | `cod_week_monday` ou `cod_year_month`, `cod_zone`, `cod_sector`, `cod_day_type`, `cod_card_country`, `cod_card_town_hall` |
| `type` | — | Texto | — | Valor fixo `CardConsumptionObserved` | — |
| `dateObserved` | Property | Texto (intervalo ISO 8601) | — | Período observado, `início/fim` na hora legal de Lisboa: segunda-feira 00:00:00 a domingo 23:59:59, ou dia 1 00:00:00 ao último dia do mês 23:59:59 | `cod_week_monday` ou `cod_year_month` |
| `frequency` | Property | Texto | — | `weekly` ou `monthly`, conforme o ficheiro | tipo de ficheiro |
| `weekOfYear` | Property | Texto | — | Semana ISO 8601, `AAAASnn` (por exemplo `2026S10`). Só nas observações semanais | derivado de `cod_week_monday` |
| `refZone` | Property | Objeto `{geographyType, code, name}` | — | Zona observada: nível (`Freguesia`, `Concelho`), código administrativo e nome | `geography_type`, `cod_zone`, `dsc_zone` |
| `sector` | Property | Objeto `{level, code, name}` | — | Setor económico. Códigos especiais: `0` Todos, `999999` Levantamentos, `-1` Indefinido | `csa_level`, `cod_sector`, `dsc_sector` |
| `operationType` | Property | Texto | — | `withdrawal` se o setor é `999999`; `all` se o setor é `0` (Todos); `purchase` nos restantes | derivado de `cod_sector` |
| `dayType` | Property | Objeto `{code, label}` | — | Tipo de dia: `0` Todos, `1` Dia de semana, `2` Fim de semana | `cod_day_type`, `dsc_day_type` |
| `cardOrigin` | Property | Objeto `{country: {code, name}, townHall: {code, name}}` | — | País (código numérico ISO 3166-1) e concelho de origem do cartão. `0` Todos, `-1` Indefinido | `cod_card_country`, `dsc_card_country`, `cod_card_town_hall`, `dsc_crd_town_hall` |
| `isAggregate` | Property | Booleano | — | `true` se o setor, o tipo de dia, o país ou o concelho de origem têm o código `0` (subtotal do cubo) | derivado |
| `cardQuantity` | Property | Inteiro | — | Número de cartões com operações | `card_quantity` |
| `operationQuantity` | Property | Inteiro | — | Número de operações | `operation_quantity` |
| `operationValue` | Property | Número | EUR | Valor total das operações | `operation_value` |
| `avgOperationsPerCard` | Property | Número | — | Média de operações por cartão | `avg_qny_operation_by_card` |
| `avgValuePerCard` | Property | Número | EUR | Valor médio por cartão | `avg_val_operation_by_card` |
| `avgValuePerOperation` | Property | Número | EUR | Valor médio por operação | `avg_val_by_operation` |
| `dataProvider` | Property | Texto | — | Valor fixo `SIBS Analytics` | — |
| `organization` | Property | Texto | — | Entidade destinatária. Valor fixo `CIMLT` | — |

Todos os atributos levam `observedAt` com o início do período (segunda-feira ou dia 1 às 00:00 de Lisboa), em ISO 8601 UTC. Todos os códigos de dimensão são publicados como texto. Unidade em código UN/CEFACT: `EUR` euro.

**Atributos obrigatórios:** `id`, `type`, `dateObserved`, `refZone`, `sector`, `dayType`, `cardOrigin`, `cardQuantity`, `operationQuantity`, `operationValue`. Os restantes são opcionais, embora a carga publique sempre todos (exceto `weekOfYear` nas observações mensais).

> **Aviso para quem consome os dados.** O ficheiro mistura células de detalhe com subtotais pré-calculados pela SIBS (código `0`, «Todos», numa ou mais dimensões). Para somar valores sem contagem dupla, filtrar por `isAggregate = false`.

## `@context`

Os exemplos usam o contexto *core* do NGSI-LD. Os termos próprios do modelo (`refZone`, `sector`, `operationType`, `dayType`, `cardOrigin`, `isAggregate`, `weekOfYear`, `frequency`, as medidas e `organization`) não estão definidos em nenhum contexto publicado e expandem-se no vocabulário por omissão do NGSI-LD. Os termos genéricos (`dateObserved`, `dataProvider`) têm o nome e a semântica dos Smart Data Models; como o contexto SDM não é incluído, também se expandem no vocabulário por omissão. Se o modelo for aceite no CNMD, os termos próprios passam a ser definidos no contexto do catálogo.

## Desvios e decisões

- **Tipo próprio.** A alternativa do catálogo seria `KeyPerformanceIndicator`, mas tem um único valor (`kpiValue`) e nenhuma estrutura dimensional: cada célula de seis medidas daria seis entidades e perder-se-ia a unidade da célula. Optou-se por um tipo de observação agregada, como as restantes analíticas de contagem da plataforma.
- **Semanal e mensal.** A documentação de modelação previa só a periodicidade semanal. A integração aceita os dois ficheiros da SIBS, que só diferem na coluna do período; `frequency` indica qual é e `weekOfYear` só existe no semanal.
- **Identificador.** O primeiro segmento é o código do período tal como vem no ficheiro: `AAAAMMDD` (a segunda-feira da semana) no semanal e `AAAAMM` no mensal, pelo que uma semana e um mês nunca coincidem. A documentação de modelação usava a semana ISO (`2026S16`). O nível geográfico e o espaço de dados não entram no identificador: o primeiro deduz-se do número de dígitos da zona e o segundo é o contexto do pedido.
- **Dimensões como objeto.** Código e descrição de cada dimensão viajam juntos num único atributo de valor objeto, em vez de pares de atributos planos. Os códigos especiais da SIBS (`0` Todos, `999999` Levantamentos, `-1` Indefinido) são conservados e todos os códigos são texto, porque são identificadores administrativos (podem ter zeros à esquerda) e não quantidades.
- **Normalização de rótulos.** `sector.level` é encurtado (`CSA Nível 2` → `CSA 2`). Nos códigos especiais de origem do cartão, o nome é fixado pela carga: `Todos os Países do Cartão`, `Todos os Concelhos do Cartão` e `Unknown` (concelho `-1`).
- **`operationType`.** Torna explícita a separação entre compras e levantamentos, que a SIBS esconde no código de setor `999999`. O setor `0` (Todos) junta compras e levantamentos, pelo que recebe o valor `all`. A carga atual ainda publica `purchase` nesse caso e será alinhada com o modelo.
- **Subtotais e duplicados.** São carregadas todas as células, de detalhe e de subtotal, marcadas com `isAggregate`. Os levantamentos chegam repetidos nos níveis CSA 2 e CSA 3 com os mesmos valores e são publicados uma só vez.
- **Tempo.** `dateObserved` é um intervalo ISO 8601 numa única cadeia, com o desvio horário de Lisboa em cada extremo (pode mudar dentro do mesmo mês com a hora de verão). Como não é filtrável por intervalo no *broker*, cada atributo leva `observedAt` com o início do período, que permite consultas temporais.
- **`dateModified` não publicado.** A documentação de modelação previa-o com o instante da entrega, mas colide com a marca de modificação que o *broker* mantém. Não é publicado; o *broker* regista `modifiedAt`.
- **Unidade monetária.** Os valores estão em euros, mas a carga atual não envia `unitCode`. A unidade fica declarada no schema (`x-ngsi.units`); o exemplo normalizado reflete o que se publica hoje.
- **Sem `location`.** O ficheiro não traz coordenadas. A georreferenciação limita-se ao descritor `refZone`; quando existirem na plataforma entidades de freguesia e concelho, a proposta é acrescentar uma `Relationship` para elas.
- **Privacidade.** Os dados são agregados e anonimizados pela SIBS e não contêm operações nem titulares individuais. São publicadas todas as células, sem limiar mínimo de cartões; as células com muito poucos cartões são mais suscetíveis de reidentificação, pelo que esta decisão é revisável se a SIBS ou a ARTE recomendarem um limiar.
- **Pontos em aberto.** Ancorar `dateObserved` no contexto dos Smart Data Models; publicar `unitCode: "EUR"` nas medidas monetárias; alinhar a carga para publicar `operationType = all` no setor `0`; validar na carga a coerência das médias com as medidas base (hoje são publicadas tal como vêm da SIBS).

## Exemplos

Os exemplos usam dados fictícios (zona, valores e datas) com a estrutura real das entidades publicadas.

- [`exemplos/CardConsumptionObserved-monthly-normalized.jsonld`](exemplos/CardConsumptionObserved-monthly-normalized.jsonld): célula mensal de detalhe (restauração, dias de semana, cartões de um concelho português), em NGSI-LD normalizado tal como é publicada no *context broker*.
- [`exemplos/CardConsumptionObserved-monthly-keyvalues.json`](exemplos/CardConsumptionObserved-monthly-keyvalues.json): a mesma entidade em formato *key-values*, validável com o schema.
- [`exemplos/CardConsumptionObserved-weekly-normalized.jsonld`](exemplos/CardConsumptionObserved-weekly-normalized.jsonld): célula semanal de subtotal (levantamentos, todos os dias, todos os concelhos), com `weekOfYear` e `isAggregate = true`.
- [`exemplos/CardConsumptionObserved-weekly-keyvalues.json`](exemplos/CardConsumptionObserved-weekly-keyvalues.json): a mesma entidade em formato *key-values*.
