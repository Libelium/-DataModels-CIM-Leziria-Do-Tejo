# Qualidade do Ar e Som

Monitorização da qualidade do ar (gases e partículas) e do ruído ambiente nos municípios da Lezíria do Tejo.

> **Estado: modelo previsto.** Esta área temática ainda não tem nenhuma integração ativa na PGU, e não há entidades publicadas. Os modelos abaixo são os que serão usados quando a fonte de dados for integrada. Os exemplos são ilustrativos, com valores fictícios.

## Modelo de dados

| Modelo | Referência | Schema | Exemplos |
|---|---|---|---|
| `AirQualityObserved` | Smart Data Models `dataModel.Environment/AirQualityObserved` (perfil) | [`schema/AirQualityObserved.json`](schema/AirQualityObserved.json) | [normalizado](exemplos/AirQualityObserved-normalized.jsonld) · [key-values](exemplos/AirQualityObserved-keyvalues.json) |
| `NoiseLevelObserved` | Smart Data Models `dataModel.Environment/NoiseLevelObserved` (perfil, com extensões) | [`schema/NoiseLevelObserved.json`](schema/NoiseLevelObserved.json) | [normalizado](exemplos/NoiseLevelObserved-normalized.jsonld) · [key-values](exemplos/NoiseLevelObserved-keyvalues.json) |

- **`AirQualityObserved`** é uma observação da qualidade do ar num ponto e num instante: concentrações de gases e de partículas, e o índice de qualidade do ar.
- **`NoiseLevelObserved`** é uma observação do nível de ruído num ponto, agregada num intervalo de tempo (por exemplo uma hora).

Os dois modelos são os que a plataforma iris360 já usa nas suas visualizações de qualidade do ar (gases, partículas e índice europeu) e de ruído (níveis regulamentares e índices estatísticos). São perfis dos Smart Data Models: usam um subconjunto dos atributos padrão e, no ruído, acrescentam índices estatísticos.

## Origem dos dados

| | |
|---|---|
| **Fonte** | A definir. Estações de qualidade do ar e sonómetros ainda por integrar na PGU |
| **Integração** | Prevista através do IoT Agent da plataforma iris360 (ou de um processo ETL em Apache Airflow, se a fonte for uma API externa), tal como nas restantes áreas temáticas |
| **Frequência** | A definir com a fonte. Qualidade do ar: tipicamente horária. Ruído: um registo por intervalo de agregação |
| **Âmbito** | Uma entidade por estação ou sonómetro, publicada no espaço de dados do município a que pertence |

## Atributos

### `AirQualityObserved`

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição |
|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:AirQualityObserved:<estação>` |
| `type` | — | Texto | — | Valor fixo `AirQualityObserved` |
| `name` | Property | Texto | — | Nome da estação |
| `dateObserved` | Property | Data/hora ISO 8601 | — | Data e hora da observação (UTC) |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas WGS84 `[longitude, latitude]` da estação |
| `address` | Property | Objeto (schema.org `PostalAddress`) | — | Morada da estação |
| `areaServed` | Property | Texto | — | Município |
| `typeofLocation` | Property | Texto | — | `outdoor` (exterior) ou `indoor` (interior) |
| `refDevice` | Relationship | URI | — | Dispositivo (`Device`) que fez a medição |
| `o3` | Property | Número | µg/m³ (`GQ`) | Ozono |
| `no2` | Property | Número | µg/m³ (`GQ`) | Dióxido de azoto |
| `no` | Property | Número | µg/m³ (`GQ`) | Monóxido de azoto |
| `so2` | Property | Número | µg/m³ (`GQ`) | Dióxido de enxofre |
| `co` | Property | Número | mg/m³ (`GP`) | Monóxido de carbono |
| `co2` | Property | Número | ppm (`59`) | Dióxido de carbono |
| `sh2` | Property | Número | µg/m³ (`GQ`) | Sulfureto de hidrogénio |
| `volatileOrganicCompoundsTotal` | Property | Número | ppb (`61`) | Compostos orgânicos voláteis totais |
| `pm10` | Property | Número | µg/m³ (`GQ`) | Partículas PM10 |
| `pm25` | Property | Número | µg/m³ (`GQ`) | Partículas PM2,5 |
| `pm1` | Property | Número | µg/m³ (`GQ`) | Partículas PM1 |
| `airQualityIndex` | Property | Inteiro | — | Índice de qualidade do ar |
| `airQualityLevel` | Property | Texto | — | Classificação qualitativa: `good`, `moderate`, `unhealthyForSensitiveGroups`, `unhealthy`, `veryUnhealthy`, `hazardous` |
| `temperature` | Property | Número | ºC (`CEL`) | Temperatura do ar |
| `relativeHumidity` | Property | Número | fração (`C62`) | Humidade relativa, entre 0 e 1 |
| `dataProvider` | Property | Texto | — | Entidade fornecedora dos dados |

**Atributos obrigatórios:** `id`, `type`, `dateObserved`, `location` (os mesmos do Smart Data Model).

Cada estação publica apenas os poluentes que mede; os restantes atributos são omitidos.

### `NoiseLevelObserved`

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição |
|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:NoiseLevelObserved:<sonómetro>` |
| `type` | — | Texto | — | Valor fixo `NoiseLevelObserved` |
| `name` | Property | Texto | — | Nome do ponto de medição |
| `dateObservedFrom` | Property | Data/hora ISO 8601 | — | Início do período de observação (UTC) |
| `dateObservedTo` | Property | Data/hora ISO 8601 | — | Fim do período de observação (UTC) |
| `dateObserved` | Property | Intervalo ISO 8601 | — | Período de observação no formato `<início>/<fim>` |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas WGS84 `[longitude, latitude]` do sonómetro |
| `address` | Property | Objeto (schema.org `PostalAddress`) | — | Morada do ponto de medição |
| `areaServed` | Property | Texto | — | Município |
| `sonometerClass` | Property | Texto | — | Classe do sonómetro: `0`, `1` ou `2` |
| `refDevice` | Relationship | URI | — | Dispositivo (`Device`) que fez a medição |
| `LAeq` | Property | Número | dB(A) (`2N`) | Nível sonoro contínuo equivalente, ponderado A |
| `LAmax` | Property | Número | dB(A) (`2N`) | Nível sonoro máximo, ponderado A |
| `LAS` | Property | Número | dB(A) (`2N`) | Nível sonoro ponderado A, constante de tempo lenta |
| `LAeq_d` | Property | Número | dB(A) (`2N`) | Nível sonoro contínuo equivalente do período diurno |
| `LAmin` | Property | Número | dB(A) (`2N`) | **Extensão.** Nível sonoro mínimo, ponderado A |
| `LA10` | Property | Número | dB(A) (`2N`) | **Extensão.** Nível excedido em 10 % do período |
| `LA50` | Property | Número | dB(A) (`2N`) | **Extensão.** Nível excedido em 50 % do período |
| `LA90` | Property | Número | dB(A) (`2N`) | **Extensão.** Nível excedido em 90 % do período (ruído de fundo) |
| `dataProvider` | Property | Texto | — | Entidade fornecedora dos dados |

**Atributos obrigatórios:** `id`, `type`, `dateObservedFrom`, `dateObservedTo`, `location` (os mesmos do Smart Data Model).

## `@context`

Os exemplos usam o contexto do domínio `dataModel.Environment` dos Smart Data Models e o contexto *core* do NGSI-LD. Os atributos padrão de `AirQualityObserved` e `NoiseLevelObserved` estão definidos no contexto dos Smart Data Models. As extensões de ruído (`LAmin`, `LA10`, `LA50`, `LA90`) não estão definidas em nenhum contexto publicado e expandem-se no vocabulário por omissão do NGSI-LD.

## Desvios e decisões

- **Modelo previsto.** A área temática ainda não tem fonte de dados integrada. O modelo será confirmado, e se necessário ajustado, quando a fonte for definida; os desvios que surgirem nessa altura serão documentados aqui.
- **Perfil dos Smart Data Models.** Foram escolhidos os atributos que a plataforma usa nas suas visualizações de qualidade do ar e de ruído. Os restantes atributos padrão (por exemplo metais pesados, `nox`, `c6h6`, `pm4`, `reliability`, `obstacles`) podem ser publicados se a fonte os medir, sem alterar o modelo.
- **Índices estatísticos de ruído.** `LAmin`, `LA10`, `LA50` e `LA90` não fazem parte do Smart Data Model e são acrescentados como extensões, porque são os índices habituais na avaliação de ruído ambiente.
- **Nomes dos poluentes.** São usados os nomes padrão dos Smart Data Models: `sh2` para o sulfureto de hidrogénio e `volatileOrganicCompoundsTotal` para os compostos orgânicos voláteis.
- **Unidades.** As concentrações de gases e partículas usam µg/m³, exceto o monóxido de carbono (mg/m³) e o dióxido de carbono (ppm), como no exemplo de referência dos Smart Data Models.
- **Humidade relativa** em fração entre 0 e 1, como no Smart Data Model.

## Exemplos

Os exemplos são ilustrativos: identificadores, localizações e valores são fictícios.

- [`exemplos/AirQualityObserved-normalized.jsonld`](exemplos/AirQualityObserved-normalized.jsonld) e [`exemplos/AirQualityObserved-keyvalues.json`](exemplos/AirQualityObserved-keyvalues.json): observação horária de uma estação de qualidade do ar.
- [`exemplos/NoiseLevelObserved-normalized.jsonld`](exemplos/NoiseLevelObserved-normalized.jsonld) e [`exemplos/NoiseLevelObserved-keyvalues.json`](exemplos/NoiseLevelObserved-keyvalues.json): observação de ruído agregada numa hora.
