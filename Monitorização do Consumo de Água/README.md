# Monitorização do Consumo de Água

Contadores de água com telecontagem das entidades gestoras dos sistemas de abastecimento da Lezíria do Tejo: dados mestres de cada contador e respetiva leitura diária de consumo.

## Modelo de dados

| Modelo | Referência | Schema | Exemplos |
|---|---|---|---|
| `Device` | Smart Data Models: `Device` (`dataModel.Device`) | [`schema/Device.json`](schema/Device.json) | [normalizado](exemplos/Device-normalized.jsonld) · [key-values](exemplos/Device-keyvalues.json) |
| `WaterConsumptionObserved` | Smart Data Models: `WaterConsumptionObserved` (`dataModel.WaterConsumption`) | [`schema/WaterConsumptionObserved.json`](schema/WaterConsumptionObserved.json) | [normalizado](exemplos/WaterConsumptionObserved-normalized.jsonld) · [key-values](exemplos/WaterConsumptionObserved-keyvalues.json) |

Cada contador é representado por **duas entidades ligadas**, separando o que é estável do que varia:

- `Device` contém a identidade e a localização do contador (número de série, coordenadas, município, entidade gestora). É atualizada com baixa frequência.
- `WaterConsumptionObserved` contém a observação de consumo (índice acumulado e consumo do período). Aponta para o `Device` através de `refDevice` e partilha com ele a raiz do identificador:

```
urn:ngsi-ld:Device:AW-<n>                       ← dados mestres
urn:ngsi-ld:WaterConsumptionObserved:AW-<n>_WCO ← observação (sufixo _WCO)
```

O estado atual é mantido no *context broker*; o histórico das observações é conservado na base de dados de séries temporais da plataforma.

> Os caudalímetros do sistema de rega usam os mesmos tipos com atributos próprios e estão documentados em [Regas Inteligentes](../Regas%20Inteligentes/).

## Origem dos dados

| | |
|---|---|
| **Fonte** | API REST da plataforma de telecontagem Flowision, com um acesso por entidade gestora: `GetDeviceLocations` (número de série e coordenadas) e `GetDeviceConsumptions` (índice acumulado diário) |
| **Integração** | Dois processos ETL em Apache Airflow (extração → cálculo do município → transformação → carga através do IoT Agent da plataforma iris360) |
| **Frequência** | `Device`: semanal, carga completa e idempotente. `WaterConsumptionObserved`: diária, com uma janela móvel de 3 dias para cobrir falhas de comunicação dos contadores |
| **Âmbito** | Uma entidade de cada tipo por contador com coordenadas, publicada no espaço de dados do município onde está instalado |

**Município.** A API de origem não indica o município. O processo ETL calcula-o por interseção das coordenadas do contador com os limites administrativos oficiais dos 11 concelhos (CAOP). Os contadores sem coordenadas ou fora dos concelhos da CIMLT não são publicados.

**Consumo.** O índice acumulado (`lastreadingtotal`, em m³) é o dado de referência: é monótono e reconcilia entre leituras. O consumo do período é calculado pela diferença entre o índice do dia e o do dia civil anterior. O consumo diário fornecido pela API (`deltaconsumption`, em litros) não é utilizado, porque não reconcilia com a diferença de índices.

## Atributos

### `Device`

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Campo de origem |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:Device:AW-<n>` (número de série com `/` substituído por `-`) | `sn` |
| `type` | — | Texto | Valor fixo `Device` | — |
| `name` | Property | Texto | Igual ao número de série (`AW/<n>`) | `sn` |
| `serialNumber` | Property | Texto | Número de série do contador na plataforma de telecontagem (`AW/<n>`) | `sn` |
| `location` | GeoProperty | GeoJSON `Point` | Coordenadas WGS84 `[longitude, latitude]` | `gpslon`, `gpslat` |
| `addressLocality` | Property | Texto | Município (concelho) com o nome por extenso, por exemplo `Salvaterra de Magos`. **Extensão** (ver «Desvios e decisões») | calculado (CAOP) |
| `provider` | Property | Texto | Entidade gestora: `Águas do Ribatejo`, `Águas de Santarém`, `Cartágua`, `Águas de Azambuja`, `Município de Golegã` ou `Município de Rio Maior` | acesso da API utilizado |
| `category` | Property | Texto | Valor fixo `meter` | — |
| `controlledProperty` | Property | Texto | Valor fixo `waterConsumption` | — |

**Atributos obrigatórios:** `id`, `type`, `name`, `serialNumber`, `location`, `addressLocality`, `provider`, `category`, `controlledProperty`.

### `WaterConsumptionObserved`

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:WaterConsumptionObserved:AW-<n>_WCO` | `sn` |
| `type` | — | Texto | — | Valor fixo `WaterConsumptionObserved` | — |
| `name` | Property | Texto | — | `AW/<n>_WCO` | `sn` |
| `waterConsumption` | Property | Número | m³ (`MTQ`) | Índice acumulado do contador no dia de leitura | `lastreadingtotal` |
| `consumptionDelta` | Property | Número | m³ (`MTQ`) | Consumo do período: índice do dia menos índice do dia civil anterior. Omitido se falta o dia anterior ou se a diferença é negativa (troca ou reinicialização do contador). **Extensão** | calculado |
| `refDevice` | Relationship | URI | — | `Device` do contador (`urn:ngsi-ld:Device:AW-<n>`) | `sn` |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas do contador, iguais às do `Device` | `gpslon`, `gpslat` |

Todos os atributos levam `observedAt` com a data da leitura (`readingdate`, um registo por dia às 00:00:00), convertida para ISO 8601.

**Atributos obrigatórios:** `id`, `type`, `name`, `waterConsumption`, `refDevice`, `location`.

## `@context`

Os exemplos usam o contexto *core* do NGSI-LD e o contexto do modelo de referência:

- `Device`: [`dataModel.Device/context.jsonld`](https://raw.githubusercontent.com/smart-data-models/dataModel.Device/master/context.jsonld), que define `serialNumber`, `provider`, `category`, `controlledProperty` e `addressLocality`.
- `WaterConsumptionObserved`: [`dataModel.WaterConsumption/context.jsonld`](https://raw.githubusercontent.com/smart-data-models/dataModel.WaterConsumption/master/context.jsonld), que define `waterConsumption`.

Os termos `name` e `location` correspondem aos do contexto *core* e dos Smart Data Models. Os termos que não constam dos contextos publicados (`consumptionDelta`, `refDevice`) expandem-se no vocabulário por omissão do NGSI-LD.

## Desvios e decisões

- **Duas entidades por contador.** Os dados mestres (`Device`) e as observações (`WaterConsumptionObserved`) são carregados por processos e com frequências diferentes. A observação não repete o município nem a entidade gestora, que se obtêm através de `refDevice`. Repete apenas `location`, para que as observações possam ser representadas no mapa sem consultar o `Device`.
- **`consumptionDelta` (extensão).** O modelo de referência só tem o índice acumulado. O consumo do período é publicado como atributo próprio, em m³ (a mesma unidade do índice), para que os painéis não tenham de calcular diferenças. Não se usa o consumo diário da fonte (em litros) porque não reconcilia com os índices.
- **`addressLocality` no primeiro nível (extensão).** O município é publicado como atributo de texto e não dentro de `address` (`address.addressLocality` no modelo de referência), porque é o único campo de morada disponível. O valor é o nome do concelho por extenso.
- **`category` e `controlledProperty` como texto.** O modelo de referência define-os como listas; são publicados como texto com um único valor (`meter`, `waterConsumption`).
- **`waterConsumption` como número.** O modelo de referência define-o como inteiro; é publicado como número porque o índice da fonte pode ter casas decimais.
- **Atributos do modelo de referência não publicados.** A fonte não fornece alarmes, caudais mínimo e máximo nem o tipo de água, pelo que estes atributos não são publicados.
- **Unidades.** Os exemplos normalizados indicam a unidade com `unitCode` (`MTQ`, metro cúbico). Na carga atual através do IoT Agent o `unitCode` não é acrescentado às entidades; a unidade é sempre a indicada neste documento.
- **Identificadores.** O número de série da fonte (`AW/<n>`) não é válido num URN; no `id` a `/` é substituída por `-`. O sufixo `_WCO` distingue a observação do `Device`.

**Verificações pendentes**

- Confirmar que `refDevice` fica registado como `Relationship` no *context broker*: o processo ETL envia o URN do `Device` como medida e o tipo final depende do aprovisionamento do IoT Agent.
- A data de leitura da fonte não indica o fuso horário e é publicada como UTC sem conversão.
- A ligação entre contador e consumidor (faturação) não está disponível na fonte e não é publicada.

## Exemplos

Os exemplos usam dados fictícios (número de série, índices e coordenadas aproximadas) com a estrutura real das entidades publicadas.

- [`exemplos/Device-normalized.jsonld`](exemplos/Device-normalized.jsonld): entidade `Device` NGSI-LD normalizada, tal como é publicada no *context broker*.
- [`exemplos/Device-keyvalues.json`](exemplos/Device-keyvalues.json): a mesma entidade em formato *key-values*, validável com o schema.
- [`exemplos/WaterConsumptionObserved-normalized.jsonld`](exemplos/WaterConsumptionObserved-normalized.jsonld): observação diária do mesmo contador, normalizada, com `unitCode` nas grandezas medidas.
- [`exemplos/WaterConsumptionObserved-keyvalues.json`](exemplos/WaterConsumptionObserved-keyvalues.json): a mesma observação em formato *key-values*, validável com o schema.
