# Regas Inteligentes

Rega inteligente dos espaços verdes municipais da Lezíria do Tejo: estações meteorológicas, controladores de rega e caudalímetros da plataforma Werrigate, com as observações meteorológicas, os programas de rega e as medições de consumo de água.

## Modelo de dados

| Modelo | Referência | Schema | Exemplos |
|---|---|---|---|
| `WeatherObserved` | Smart Data Models: `WeatherObserved` (`dataModel.Weather`) | [`schema/WeatherObserved.json`](schema/WeatherObserved.json) | [normalizado](exemplos/WeatherObserved-normalized.jsonld) · [key-values](exemplos/WeatherObserved-keyvalues.json) |
| `Device` | Smart Data Models: `Device` (`dataModel.Device`) | [`schema/Device.json`](schema/Device.json) | Estação: [normalizado](exemplos/Device-estacao-normalized.jsonld) · [key-values](exemplos/Device-estacao-keyvalues.json)<br>Controlador: [normalizado](exemplos/Device-controlador-normalized.jsonld) · [key-values](exemplos/Device-controlador-keyvalues.json)<br>Caudalímetro: [normalizado](exemplos/Device-caudalimetro-normalized.jsonld) · [key-values](exemplos/Device-caudalimetro-keyvalues.json) |
| `AgriParcel` | Smart Data Models: `AgriParcel` (`dataModel.AgriFood`) | [`schema/AgriParcel.json`](schema/AgriParcel.json) | [normalizado](exemplos/AgriParcel-normalized.jsonld) · [key-values](exemplos/AgriParcel-keyvalues.json) |
| `AgriParcelOperation` | Smart Data Models: `AgriParcelOperation` (`dataModel.AgriFood`) | [`schema/AgriParcelOperation.json`](schema/AgriParcelOperation.json) | [normalizado](exemplos/AgriParcelOperation-normalized.jsonld) · [key-values](exemplos/AgriParcelOperation-keyvalues.json) |
| `WaterConsumptionObserved` | Smart Data Models: `WaterConsumptionObserved` (`dataModel.WaterConsumption`) | [`schema/WaterConsumptionObserved.json`](schema/WaterConsumptionObserved.json) | [normalizado](exemplos/WaterConsumptionObserved-normalized.jsonld) · [key-values](exemplos/WaterConsumptionObserved-keyvalues.json) |

Nenhum destes tipos tem equivalente no CNMD; foram adotados os modelos dos Smart Data Models, com os atributos de extensão indicados em cada tabela.

Cada jardim com rega inteligente é um `AgriParcel` que agrupa os seus equipamentos (`Device`). Os dados mestres de cada equipamento ficam no `Device`; as observações e o estado operacional ficam em entidades próprias, ligadas ao `Device` e com a mesma raiz no identificador:

```
urn:ngsi-ld:AgriParcel:<NomeDoJardim>                              ← jardim (hasDevice → Device)
urn:ngsi-ld:Device:ControladorRega_<n>                             ← controlador de rega
urn:ngsi-ld:AgriParcelOperation:ControladorRega_<n>_prog_<m>       ← programa <m> do controlador (operatedBy, hasAgriParcel)
urn:ngsi-ld:Device:Caudalimetro_<n>                                ← caudalímetro
urn:ngsi-ld:WaterConsumptionObserved:Caudalimetro_<n>_flow         ← medição do caudalímetro (refDevice)
urn:ngsi-ld:Device:Estacao_Meteorologica_<n>                       ← estação meteorológica
urn:ngsi-ld:WeatherObserved:Estacao_Meteorologica_<município>_<n>_WTO ← observação da estação
```

O estado atual é mantido no *context broker*; o histórico das observações é conservado na base de dados de séries temporais da plataforma.

> Os contadores de água das entidades gestoras usam os mesmos tipos `Device` e `WaterConsumptionObserved` com atributos próprios e estão documentados em [Monitorização do Consumo de Água](../Monitoriza%C3%A7%C3%A3o%20do%20Consumo%20de%20%C3%81gua/).

## Origem dos dados

| | |
|---|---|
| **Fonte** | Plataforma de rega Werrigate (Visualforma), com uma instância por município e autenticação por *token*. API de meteorologia (estações com consola da família Davis Vantage): lista de estações e leitura atual de cada estação. API de rega (equipamentos LoRaWAN): lista de equipamentos e detalhe de cada um, com os programas, o estado das saídas e o bloco do caudalímetro |
| **Integração** | Quatro processos ETL em Apache Airflow (extração por município → transformação em medidas → carga através do IoT Agent da plataforma iris360) |
| **Frequência** | `Device` (dados mestres): semanal. `WeatherObserved` e estado da estação: 15 minutos. `WaterConsumptionObserved` e bateria do caudalímetro: 30 minutos. `AgriParcelOperation`: 30 minutos. `AgriParcel`: carga pontual, sem processo periódico |
| **Âmbito** | 11 municípios. Cada entidade é publicada no espaço de dados do município a que pertence |

**Famílias de equipamentos.** O campo `model` da API de rega distingue os controladores de rega (`CMBATT6`, `CMIRRI16`, `CMIRRI24`, com 6, 16 e 24 saídas) dos caudalímetros (`CCBATT1`). As estações meteorológicas vêm da API de meteorologia.

**Segurança.** O detalhe dos equipamentos de rega inclui as chaves criptográficas LoRaWAN (APPKEY, APPSKEY, NWKSKEY). O processo ETL descarta-as e nunca são publicadas; só o identificador de hardware (DevEUI) é conservado.

**Inventário.** Na carga inicial foram publicados 33 jardins e 192 equipamentos de rega (controladores e caudalímetros):

| Município | Jardins | Equipamentos de rega |
|---|---|---|
| Almeirim | 3 | 24 |
| Alpiarça | 1 | 4 |
| Azambuja | 3 | 35 |
| Benavente | 3 | 6 |
| Cartaxo | 3 | 21 |
| Chamusca | 3 | 8 |
| Coruche | 4 | 14 |
| Golegã | 1 | 6 |
| Rio Maior | 3 | 8 |
| Salvaterra de Magos | 3 | 21 |
| Santarém | 6 | 45 |
| **Total** | **33** | **192** |

## Atributos

Nas tabelas, **Extensão** indica um atributo que não existe no modelo de referência dos Smart Data Models.

### `WeatherObserved`

Uma entidade por estação meteorológica. Cada leitura inválida (valor sentinela com que a consola indica «sem leitura», ou valor fora do intervalo físico indicado no schema) é omitida atributo a atributo, sem descartar o resto da observação.

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:WeatherObserved:Estacao_Meteorologica_<município>_<n>_WTO` | município, `id` |
| `type` | — | Texto | — | Valor fixo `WeatherObserved` | — |
| `temperature` | Property | Número | °C (`CEL`) | Temperatura do ar | `outsideTemperature` |
| `relativeHumidity` | Property | Número | % (`P1`) | Humidade relativa, 0 a 100 (ver «Desvios e decisões») | `outsideHumidity` |
| `dewPoint` | Property | Número | °C (`CEL`) | Ponto de orvalho | `dewPoint` |
| `feelsLikeTemperature` | Property | Número | °C (`CEL`) | Temperatura aparente (índice de calor) | `heatIndex` |
| `atmosphericPressure` | Property | Número | hPa (`A97`) | Pressão atmosférica | `barometer` |
| `pressureTendency` | Property | Texto | — | Tendência barométrica (`Steady`, `Rising Slowly`…) | `barometricPressure` |
| `windSpeed` | Property | Número | km/h (`KMH`) | Velocidade do vento | `windSpeed` |
| `windDirection` | Property | Número | graus (`DD`) | Direção do vento, 0 a 360 | `windDirection` |
| `gustSpeed` | Property | Número | km/h (`KMH`) | Rajada máxima dos últimos 10 minutos | `windGust10m` |
| `precipitation` | Property | Número | mm (`MMT`) | Precipitação da última hora | `last1hRain` |
| `solarRadiation` | Property | Número | W/m² (`D54`) | Radiação solar global | `solarRadiation` |
| `uVIndexMax` | Property | Número | — | Índice UV. Só com leitura real | `uvLevel` |
| `windChillTemperature` | Property | Número | °C (`CEL`) | Sensação térmica pelo vento. **Extensão** | `windChill` |
| `thswIndex` | Property | Número | °C (`CEL`) | Índice THSW da consola. **Extensão** | `thswIndex` |
| `windSpeedAverage2m` | Property | Número | km/h (`KMH`) | Vento médio dos últimos 2 minutos. **Extensão** | `avgWindSpeed2m` |
| `windSpeedAverage10m` | Property | Número | km/h (`KMH`) | Vento médio dos últimos 10 minutos. **Extensão** | `avgWindSpeed10m` |
| `windDirection10m` | Property | Número | graus (`DD`) | Direção da rajada máxima dos últimos 10 minutos. **Extensão** | `windDirection10m` |
| `precipitationRate` | Property | Número | mm/h (`H67`) | Intensidade de precipitação. **Extensão** | `rainRate` |
| `rainAccumulatedLast15m` | Property | Número | mm (`MMT`) | Chuva dos últimos 15 minutos. **Extensão** | `last15mRain` |
| `rainAccumulatedLast24h` | Property | Número | mm (`MMT`) | Chuva das últimas 24 horas. **Extensão** | `last24hRain` |
| `rainAccumulatedDay` | Property | Número | mm (`MMT`) | Chuva do dia. **Extensão** | `dayRain` |
| `rainAccumulatedMonth` | Property | Número | mm (`MMT`) | Chuva do mês. **Extensão** | `monthRain` |
| `rainAccumulatedYear` | Property | Número | mm (`MMT`) | Chuva do ano. **Extensão** | `yearRain` |
| `rainStorm` | Property | Número | mm (`MMT`) | Chuva da tempestade em curso. **Extensão** | `stormRain` |
| `stormStartDate` | Property | Número | — | Início da tempestade em curso, no formato numérico da consola. Omitido sem tempestade ativa. **Extensão** | `startDateOfCurrentStorm` |
| `evapotranspirationDay` | Property | Número | mm (`MMT`) | Evapotranspiração do dia. **Extensão** | `dayET` |
| `evapotranspirationMonth` | Property | Número | mm (`MMT`) | Evapotranspiração do mês. **Extensão** | `monthET` |
| `evapotranspirationYear` | Property | Número | mm (`MMT`) | Evapotranspiração do ano. **Extensão** | `yearET` |
| `sunrise` | Property | Texto | — | Hora local do nascer do sol (`HH:MM`). **Extensão** | `sunRise` |
| `sunset` | Property | Texto | — | Hora local do pôr do sol (`HH:MM`). **Extensão** | `sunSet` |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas WGS84 `[longitude, latitude]` da estação | `latitude`, `longitude` (lista de estações) |

Os atributos levam `observedAt` em ISO 8601 UTC: a hora da última comunicação da estação (`lastContact`) quando a fonte a indica, ou a hora de receção pela plataforma.

**Atributos obrigatórios:** `id`, `type`, `location`. As grandezas são todas condicionais, porque cada uma é omitida quando a leitura é inválida.

### `Device`

Uma entidade por equipamento, em três famílias. Os dados mestres são publicados pelo processo semanal; a bateria e o último contacto são atualizados pelos processos de observação.

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:Device:Estacao_Meteorologica_<n>`, `…:ControladorRega_<n>` ou `…:Caudalimetro_<n>` | `id` |
| `type` | — | Texto | — | Valor fixo `Device` | — |
| `name` | Property | Texto | — | Nome do equipamento. Nos equipamentos de rega segue o padrão `<Jardim> - Caixa <n>` ou `<Jardim> - Caudalímetro <n>` | `alias` |
| `category` | Property | Lista de texto | — | `sensor` (estação), `actuator` + `irrSystem` (controlador), `meter` (caudalímetro) | família |
| `controlledProperty` | Property | Lista de texto | — | Só estações e caudalímetros. Estação: `temperature`, `humidity`, `atmosphericPressure`, `windSpeed`, `windDirection`, `precipitation`, `solarRadiation`. Caudalímetro: `waterConsumption` | família |
| `deviceModelName` | Property | Texto | — | Modelo: `CMBATT6`, `CMIRRI16`, `CMIRRI24`, `CCBATT1`, ou `meteo` nas estações. **Extensão** | `model` |
| `loraWanDEVEUI` | Property | Texto | — | Só controladores e caudalímetros. Identificador de hardware LoRaWAN (DevEUI, 16 dígitos hexadecimais). **Extensão** | `loraWanDEVEUI` |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas WGS84 `[longitude, latitude]` | `latitude`, `longitude` |
| `batteryLevel` | Property | Número | fração 0 a 1 | Só caudalímetros. Nível de bateria (percentagem da fonte ÷ 100) | `battery` |
| `batteryVoltage` | Property | Número | V (`VLT`) | Só estações. Tensão da bateria da consola. **Extensão** | `consoleBattery` |
| `dateLastValueReported` | Property | Texto ou número | — | Só estações. Último contacto da estação, tal como devolvido pela fonte (ver «Desvios e decisões») | `lastContact` |

**Atributos obrigatórios:** `id`, `type`, `name`, `category`, `deviceModelName`.

### `AgriParcel`

Uma entidade por jardim, carregada uma vez a partir do inventário dos equipamentos de rega.

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:AgriParcel:<NomeDoJardim>`: nome do jardim em PascalCase, sem acentos nem espaços | derivado de `alias` |
| `type` | — | Texto | — | Valor fixo `AgriParcel` | — |
| `name` | Property | Texto | — | Nome do jardim | derivado de `alias` |
| `addressLocality` | Property | Texto | — | Município. **Extensão** (ver «Desvios e decisões») | instância do município |
| `addressCountry` | Property | Texto | — | Valor fixo `PT`. **Extensão** | — |
| `hasDevice` | Relationship | Lista de URI | — | Controladores e caudalímetros do jardim (`Device`) | inventário |
| `location` | GeoProperty | GeoJSON `Point` | — | Ponto representativo do jardim, calculado a partir das coordenadas dos seus equipamentos | `latitude`, `longitude` dos equipamentos |

**Atributos obrigatórios:** `id`, `type`, `name`, `location`, `addressLocality`, `addressCountry`, `hasDevice`.

### `AgriParcelOperation`

Uma entidade por programa habilitado de cada controlador de rega. Os programas desabilitados não são publicados.

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:AgriParcelOperation:ControladorRega_<n>_prog_<m>` | `id`, `programs[].number` |
| `type` | — | Texto | — | Valor fixo `AgriParcelOperation` | — |
| `operationType` | Property | Texto | — | Valor fixo `water` | — |
| `status` | Property | Texto | — | `ongoing` se o programa está a regar; `scheduled` nos restantes casos | `programs[].running` |
| `hasAgriParcel` | Relationship | URI | — | Jardim (`AgriParcel`), derivado do nome do controlador. Omitido se não for possível derivá-lo | `alias` |
| `operatedBy` | Relationship | URI | — | Controlador de rega (`Device`) que executa o programa. **Extensão** | `id` |
| `irrigationSchedule` | Property | Objeto | — | Programação (ver tabela abaixo). **Extensão** | `programs[]` |
| `valveStatus` | Property | Lista de objetos `{valve, alias, open}` | — | Estado em tempo real de cada saída física do controlador (6, 16 ou 24 conforme o modelo). **Extensão** | `outputStatus`, `outputAlias` |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas do controlador | `latitude`, `longitude` |

**Estrutura de `irrigationSchedule`**

| Campo | Tipo de dado | Descrição | Campo de origem |
|---|---|---|---|
| `programNumber` | Inteiro | Número do programa no controlador | `number` |
| `enabled` | Booleano | Programa habilitado (sempre `true` nas entidades publicadas) | `enabled` |
| `running` | Booleano | Programa a regar neste momento | `running` |
| `allDays` | Booleano | Ativo todos os dias | `enableOnAllDays` |
| `daysOfWeek` | Objeto `{monday … sunday}` de booleanos | Dias da semana em que o programa está ativo | `daysOfWeek` |
| `startTimes` | Lista de texto `HH:MM` | Horas de arranque (hora local), sem as posições vazias | `startTimes` |
| `startTimesAreIntervals` | Booleano | `startTimes` são intervalos de repetição e não horas de arranque | `startTimesAreIntervals` |
| `durationUnit` | Texto | Unidade das durações. Valor fixo `minutes` | — |
| `valveSequence` | Lista de objetos `{order, valve, duration}` | Válvulas por ordem de abertura e tempo de rega de cada uma; só as posições com válvula atribuída | `sequence`, `time` |

**Atributos obrigatórios:** `id`, `type`, `operationType`, `status`, `operatedBy`, `irrigationSchedule`, `valveStatus`.

### `WaterConsumptionObserved`

Uma entidade por caudalímetro de rega.

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:WaterConsumptionObserved:Caudalimetro_<n>_flow` | `id` |
| `type` | — | Texto | — | Valor fixo `WaterConsumptionObserved` | — |
| `refDevice` | Relationship | URI | — | Caudalímetro (`urn:ngsi-ld:Device:Caudalimetro_<n>`) | `id` |
| `waterConsumption` | Property | Número | m³ (`MTQ`) | Volume total acumulado (totalizador). Ver unidade em «Verificações pendentes» | `FlowMeter_TotalVolume` |
| `alarmInProgress` | Property | Inteiro (0/1) | — | 1 se algum alarme ou aviso do caudalímetro está ativo | derivado dos seis alarmes |
| `flow` | Property | Número | da fonte | Caudal instantâneo. **Extensão** | `FlowMeter_InstantFlow` |
| `totalVolumeWithoutOffset` | Property | Número | m³ (`MTQ`) | Volume total sem o desvio de calibração do equipamento. **Extensão** | `FlowMeter_TotalVolumeWithoutOffset` |
| `maxFlowAlarm` | Property | Booleano | — | Caudal acima do limiar de alarme. **Extensão** | `FlowMeter_MaxFlowAlarmExceed` |
| `maxFlowWarning` | Property | Booleano | — | Caudal acima do limiar de aviso. **Extensão** | `FlowMeter_MaxFlowWarningExceed` |
| `maxDayVolumeAlarm` | Property | Booleano | — | Volume diário acima do máximo de alarme. **Extensão** | `FlowMeter_MaxDayVolumeAlarmValueExceed` |
| `maxDayVolumeWarning` | Property | Booleano | — | Volume diário acima do máximo de aviso. **Extensão** | `FlowMeter_MaxDayVolumeWarningValueExceed` |
| `minDayVolumeAlarm` | Property | Booleano | — | Volume diário abaixo do mínimo de alarme. **Extensão** | `FlowMeter_MinDayVolumeAlarmValueExceed` |
| `minDayVolumeWarning` | Property | Booleano | — | Volume diário abaixo do mínimo de aviso. **Extensão** | `FlowMeter_MinDayVolumeWarningValueExceed` |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas do caudalímetro | `latitude`, `longitude` |

**Atributos obrigatórios:** `id`, `type`, `refDevice`, `waterConsumption`, `alarmInProgress`, `maxFlowAlarm`, `maxFlowWarning`, `maxDayVolumeAlarm`, `maxDayVolumeWarning`, `minDayVolumeAlarm`, `minDayVolumeWarning`.

## `@context`

Os exemplos usam o contexto *core* do NGSI-LD e o contexto do modelo de referência de cada tipo:

- `WeatherObserved`: [`dataModel.Weather/context.jsonld`](https://raw.githubusercontent.com/smart-data-models/dataModel.Weather/master/context.jsonld).
- `Device`: [`dataModel.Device/context.jsonld`](https://raw.githubusercontent.com/smart-data-models/dataModel.Device/master/context.jsonld).
- `AgriParcel` e `AgriParcelOperation`: [`dataModel.AgriFood/context.jsonld`](https://raw.githubusercontent.com/smart-data-models/dataModel.AgriFood/master/context.jsonld).
- `WaterConsumptionObserved`: [`dataModel.WaterConsumption/context.jsonld`](https://raw.githubusercontent.com/smart-data-models/dataModel.WaterConsumption/master/context.jsonld).

Os termos genéricos (`name`, `location`) correspondem aos do contexto *core* e dos Smart Data Models. Os atributos marcados como **Extensão** não constam dos contextos publicados e expandem-se no vocabulário por omissão do NGSI-LD.

## Desvios e decisões

**Comuns**

- **Identificadores.** A parte final do `id` é `<Família>_<n>`, em que `<n>` é o identificador do equipamento na instância Werrigate do município. Cada município numera os seus equipamentos de forma independente, pelo que o mesmo `id` pode existir em espaços de dados de municípios diferentes; é único dentro de cada espaço de dados. As observações acrescentam um sufixo que as distingue do `Device`: `_WTO` (meteorologia), `_flow` (caudal) e `_prog_<m>` (programa de rega).
- **Observação separada do equipamento.** As grandezas medidas e o estado de rega são publicados em entidades próprias; a bateria e o último contacto ficam no `Device`, porque descrevem o equipamento e não a observação.
- **Unidades.** Os exemplos normalizados indicam a unidade com `unitCode` (UN/CEFACT). Na carga atual através do IoT Agent o `unitCode` não é acrescentado às entidades; a unidade é sempre a indicada neste documento.
- **Leituras inválidas.** Os valores sentinela das estações, os valores fora do intervalo físico e as coordenadas nulas (0,0) são omitidos atributo a atributo; a entidade mantém o último valor válido de cada atributo.

**`WeatherObserved`**

- **`relativeHumidity` em percentagem.** É publicada de 0 a 100, tal como a fonte a entrega. O modelo de referência usa uma fração de 0 a 1; os consumidores devem dividir por 100.
- **Vento em km/h.** A fonte entrega a velocidade do vento já convertida para km/h. O modelo de referência indica m/s.
- **`pressureTendency` como texto da consola** (`Steady`, `Rising Slowly`, `Falling Rapidly`…), em vez do vocabulário do modelo de referência (`raising`, `falling`, `steady`).
- **Sem `dateObserved`.** O modelo de referência exige `dateObserved`; a hora da observação é publicada como `observedAt` de cada atributo.
- **Extensões.** Os acumulados de chuva e de evapotranspiração são a base do cálculo de rega (balanço evapotranspiração menos chuva) e não têm equivalente no modelo de referência. Os índices próprios da consola (`windChillTemperature`, `thswIndex`), o vento médio, a direção da rajada, a intensidade de precipitação, a tempestade em curso e as horas do nascer e do pôr do sol também são extensões.
- **Atributos não publicados.** As leituras internas da consola (temperatura e humidade interiores), o estado da bateria do transmissor e o endereço IP da estação não são publicados. A estação não tem `refDevice`: o `Device` correspondente identifica-se pelo número `<n>` do identificador.
- **Previsão da consola.** A consola fornece uma previsão textual. O modelo `WeatherForecast` previsto na documentação do projeto **não está implementado** e não é publicado.

**`Device`**

- **`controlledProperty` só em estações e caudalímetros.** O modelo de referência exige-o; nos controladores de rega não é publicado, porque são atuadores e não medem grandezas.
- **`category` e `controlledProperty` com um só valor.** O modelo de referência define-os como listas; quando têm um único valor (`sensor`, `meter`, `waterConsumption`) são publicados como texto simples. O schema aceita as duas formas.
- **Extensões.** `deviceModelName` (o modelo de referência remete o modelo para uma entidade `DeviceModel`, que não é publicada), `loraWanDEVEUI` e `batteryVoltage` (a consola meteorológica indica volts, não percentagem). O valor `meteo` de `deviceModelName` nas estações indica a família, porque a fonte não identifica o modelo da consola.
- **Estado das estações noutro identificador.** O processo de meteorologia publica `batteryVoltage`, `dateLastValueReported`, `name` e `location` da estação no identificador `urn:ngsi-ld:Device:Estacao_Meteorologica_<município>_<n>`, diferente do dos dados mestres (`…:Estacao_Meteorologica_<n>`). Ver «Verificações pendentes».
- **`dateLastValueReported` sem conversão.** É publicado tal como a fonte o entrega, e não como data ISO 8601 (o tipo do modelo de referência).
- **Atributos não publicados.** Nos controladores não são publicados o nível de bateria, o estado de comunicação, o estado de funcionamento (`value`) nem o fator de espalhamento LoRa. A ligação do equipamento ao jardim é publicada no `AgriParcel` (`hasDevice`), e não no `Device` (`controlledAsset`).
- **Atributos residuais.** Algumas entidades `Device` criadas na fase de arranque conservam atributos de medição (meteorologia ou caudal) anteriores à separação das observações em entidades próprias. Não fazem parte do modelo; as medições válidas estão em `WeatherObserved` e `WaterConsumptionObserved`.

**`AgriParcel`**

- **Jardim urbano como `AgriParcel`.** Não existe no CNMD nem nos Smart Data Models um tipo para espaços verdes regados; `AgriParcel` é o tipo que agrupa equipamentos de rega (`hasDevice`) e é o alvo de `AgriParcelOperation`.
- **Sem `area`, `hasAgriCrop` nem `category`.** O modelo de referência exige `area` e `hasAgriCrop`; não se dispõe do polígono dos jardins, e a cultura e a categoria agrícola não se aplicam. `location` é um ponto representativo e não o perímetro do jardim.
- **`addressLocality` e `addressCountry` no primeiro nível.** O município e o país são publicados como atributos de texto e não dentro de `address`, porque são os únicos campos de morada disponíveis.
- **Identificador derivado do nome.** O `id` é o nome do jardim em PascalCase. O mesmo critério é aplicado ao nome do controlador para obter `hasAgriParcel`, pelo que ambos coincidem.

**`AgriParcelOperation`**

- **Programa recorrente em vez de operação pontual.** O modelo de referência descreve operações com início e fim (`plannedStartAt`, `plannedEndAt`, obrigatórios). Um programa de rega repete-se segundo um calendário; o calendário é publicado em `irrigationSchedule` e as datas planeadas não são publicadas.
- **Extensões.** `operatedBy` (o modelo de referência só prevê o operador humano, `hasOperator`), `irrigationSchedule` e `valveStatus`.
- **Saídas físicas.** A fonte devolve sempre 32 nomes de saída genéricos; `valveStatus` só inclui as saídas reais do modelo do controlador (6, 16 ou 24).
- **`hasAgriParcel` condicionado.** É derivado do nome do controlador (`<Jardim> - Caixa <n>`). Se o nome não segue o padrão, o atributo é omitido.
- **`status`** usa apenas os valores `scheduled` e `ongoing` do vocabulário do modelo de referência.

**`WaterConsumptionObserved`**

- **`flow` (extensão).** O modelo de referência só tem caudais mínimo e máximo (`minFlow`, `maxFlow`); o caudal instantâneo é publicado como atributo próprio.
- **Alarmes do equipamento (extensão).** Os seis alarmes e avisos de limiar do caudalímetro são publicados como booleanos. `alarmInProgress` (modelo de referência) resume-os: vale 1 se algum estiver ativo. Os alarmes enumerados do modelo de referência (persistência, fuga, manipulação, qualidade da água) não são fornecidos pela fonte e não são publicados.
- **`waterConsumption` como número.** O modelo de referência define-o como inteiro; é publicado como número porque o totalizador pode ter casas decimais.
- **Configuração não publicada.** Os limiares de alarme, o desvio de calibração e o fator de impulsos do caudalímetro são configuração e não telemetria.

**Verificações pendentes**

- Confirmar a unidade do volume do caudalímetro na fonte. O valor é publicado sem conversão até essa confirmação; o modelo exige m³. A unidade de `flow` também está por confirmar.
- Unificar o identificador do `Device` das estações meteorológicas: os dados mestres e o estado (bateria, último contacto) estão publicados em duas entidades distintas.
- Normalizar `dateLastValueReported` para ISO 8601 UTC.
- Em produção, `irrigationSchedule` e `valveStatus` ficam atualmente armazenados como texto (serialização do objeto) e não como objeto JSON. O schema documenta a estrutura de objeto, que é a que o processo ETL envia.
- Confirmar que `refDevice`, `operatedBy`, `hasAgriParcel` e `hasDevice` ficam registados como `Relationship` no *context broker*: o processo ETL envia os URN como medidas e o tipo final depende do aprovisionamento do IoT Agent.

## Exemplos

Os exemplos usam dados fictícios (identificadores, nomes de jardins, identificadores LoRaWAN, medições e coordenadas aproximadas) com a estrutura real das entidades publicadas.

- [`exemplos/WeatherObserved-normalized.jsonld`](exemplos/WeatherObserved-normalized.jsonld) · [`exemplos/WeatherObserved-keyvalues.json`](exemplos/WeatherObserved-keyvalues.json): observação de uma estação meteorológica.
- [`exemplos/Device-estacao-normalized.jsonld`](exemplos/Device-estacao-normalized.jsonld) · [`exemplos/Device-estacao-keyvalues.json`](exemplos/Device-estacao-keyvalues.json): dados mestres de uma estação meteorológica.
- [`exemplos/Device-controlador-normalized.jsonld`](exemplos/Device-controlador-normalized.jsonld) · [`exemplos/Device-controlador-keyvalues.json`](exemplos/Device-controlador-keyvalues.json): controlador de rega de 6 saídas.
- [`exemplos/Device-caudalimetro-normalized.jsonld`](exemplos/Device-caudalimetro-normalized.jsonld) · [`exemplos/Device-caudalimetro-keyvalues.json`](exemplos/Device-caudalimetro-keyvalues.json): caudalímetro, com o nível de bateria.
- [`exemplos/AgriParcel-normalized.jsonld`](exemplos/AgriParcel-normalized.jsonld) · [`exemplos/AgriParcel-keyvalues.json`](exemplos/AgriParcel-keyvalues.json): jardim com dois controladores e um caudalímetro.
- [`exemplos/AgriParcelOperation-normalized.jsonld`](exemplos/AgriParcelOperation-normalized.jsonld) · [`exemplos/AgriParcelOperation-keyvalues.json`](exemplos/AgriParcelOperation-keyvalues.json): programa 1 do controlador do exemplo.
- [`exemplos/WaterConsumptionObserved-normalized.jsonld`](exemplos/WaterConsumptionObserved-normalized.jsonld) · [`exemplos/WaterConsumptionObserved-keyvalues.json`](exemplos/WaterConsumptionObserved-keyvalues.json): medição do caudalímetro do exemplo.

Os ficheiros `*-normalized.jsonld` são entidades NGSI-LD normalizadas, tal como são publicadas no *context broker*, com `unitCode` nas grandezas com unidade; os ficheiros `*-keyvalues.json` são as mesmas entidades em formato *key-values*, validáveis com o schema do tipo.
