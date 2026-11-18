# Monitorização Energética

Medição elétrica dos circuitos trifásicos dos edifícios municipais da Lezíria do Tejo, a partir de analisadores de rede instalados nos quadros elétricos.

## Modelo de dados

| Modelo | Referência | Schema | Exemplos |
|---|---|---|---|
| `ThreePhaseAcMeasurement` | Smart Data Models: `ThreePhaseAcMeasurement` (`dataModel.Energy`) | [`schema/ThreePhaseAcMeasurement.json`](schema/ThreePhaseAcMeasurement.json) | [normalizado](exemplos/ThreePhaseAcMeasurement-normalized.jsonld) · [key-values](exemplos/ThreePhaseAcMeasurement-keyvalues.json) |

`ThreePhaseAcMeasurement` representa a medição de um sistema de corrente alternada trifásica. Cada entidade corresponde a um circuito medido (um analisador de rede) e é atualizada a cada nova leitura. As grandezas por fase não se desdobram em atributos independentes: são um único atributo cujo valor é um objeto com uma chave por fase (`L1`, `L2`, `L3` e, na corrente, `N` para o neutro), de modo que as três fases partilham o mesmo instante.

O identificador combina o local configurado no datalogger, o número de série do datalogger e o número do analisador no barramento:

```
urn:ngsi-ld:ThreePhaseAcMeasurement:<local>_<númeroDeSérieDatalogger>_<analisador>
```

## Origem dos dados

| | |
|---|---|
| **Fonte** | Analisadores de rede trifásicos (Carlo Gavazzi EM530) ligados a um datalogger (Carlo Gavazzi UWP 4.0) no edifício. O datalogger publica as leituras por MQTT |
| **Integração** | O FIWARE Manager da plataforma iris360 recebe as mensagens, converte cada sinal no atributo correspondente do modelo e envia as leituras ao IoT Agent da plataforma |
| **Frequência** | Contínua. Cada mensagem traz um lote de leituras consecutivas (cadência configurada no datalogger, da ordem do minuto); cada leitura atualiza a entidade com o seu `dateObserved` |
| **Âmbito** | Uma entidade por analisador de rede (circuito medido), publicada no espaço de dados do município a que pertence o edifício |

## Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:ThreePhaseAcMeasurement:<local>_<sn>_<analisador>` | chave de local, `sn`, `devices[].id` |
| `type` | — | Texto | — | Valor fixo `ThreePhaseAcMeasurement` | — |
| `totalActiveEnergyImport` | Property | Número | kWh (`KWH`) | Energia ativa importada desde o início da contagem, nas três fases. Contador totalizador crescente | `kWh (+)TOT` |
| `totalActivePower` | Property | Número | W (`WTT`) | Potência ativa instantânea das três fases. O equipamento entrega kW; a integração converte para W | `kW SYS` |
| `totalPowerFactor` | Property | Número | adimensional (-1 a +1) | Fator de potência do conjunto das fases | `PF SYS` |
| `current` | Property | Objeto `{L1, L2, L3, N}` | A (`AMP`) | Corrente por fase e, se o equipamento a entregar, no neutro | `A L1`, `A L2`, `A L3`, `A N` |
| `thdCurrent` | Property | Objeto `{L1, L2, L3}` | % (`P1`) | Distorção harmónica total da corrente por fase, em percentagem | `THDA L1`, `THDA L2`, `THDA L3` |
| `dateObserved` | Property | Data e hora (ISO 8601 UTC) | — | Instante da leitura indicado pelo equipamento | `signals[].time` |
| `serialNumber` | Property | Texto | — | Número de série do datalogger. **Extensão** | `sn` |
| `municipality` | Property | Texto | — | Município do edifício, tal como está configurado no datalogger (por exemplo `AZAMBUJA`). **Extensão** | chave de local |
| `circuit` | Property | Texto | — | Nome do circuito ou quadro medido, configurado no datalogger (por exemplo `Geral Group3P-1`). **Extensão** | chave de local |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas WGS84 `[longitude, latitude]` do edifício. Não é enviada pelo datalogger: é atribuída na plataforma quando o edifício está georreferenciado | — |

Todas as grandezas levam `observedAt`, atribuído pelo IoT Agent no momento da receção da leitura.

**Atributos obrigatórios:** `id`, `type`, `dateObserved`, `serialNumber`. As grandezas dependem dos sinais configurados no datalogger; os equipamentos atualmente instalados entregam todas as da tabela.

**Outras grandezas suportadas.** A integração converte também os sinais seguintes quando o datalogger os entrega (atualmente não são publicados):

| Atributos | Unidade |
|---|---|
| `phaseVoltage` `{L1, L2, L3}`, `phaseToPhaseVoltage` `{L12, L23, L31}` | V |
| `activePower`, `apparentPower`, `reactivePower` `{L1, L2, L3}`; `totalApparentPower`, `totalReactivePower` | W, VA, var |
| `powerFactor` `{L1, L2, L3}` | adimensional |
| `frequency` | Hz |
| `totalActiveEnergyExport`, `activeEnergyImport` `{L1, L2, L3}` | kWh |
| `totalReactiveEnergyImport`, `totalReactiveEnergyExport`, `totalApparentEnergyImport` | kvarh, kVAh |
| `thdVoltage` `{L1, L2, L3}` | % |
| **Extensões:** `averagePhaseVoltage`, `averagePhaseToPhaseVoltage` (V), `activePowerDemand` (W), `apparentPowerDemand` (VA), `operatingHours`, `operatingHoursPartial` (h) | |

## `@context`

Os exemplos usam o contexto *core* do NGSI-LD e o contexto do modelo de referência, [`dataModel.Energy/context.jsonld`](https://raw.githubusercontent.com/smart-data-models/dataModel.Energy/master/context.jsonld), que define todas as grandezas do modelo, as chaves de fase (`L1`, `L2`, `L3`, `N`) e também `serialNumber` e `dateObserved`. Os termos de extensão `municipality` e `circuit` não constam de nenhum contexto publicado e expandem-se no vocabulário por omissão do NGSI-LD.

## Desvios e decisões

- **`totalActivePower` em W.** O equipamento entrega as potências em kW, kVA e kvar. A integração multiplica-as por 1000 para usar a unidade do modelo de referência (W, VA, var), a mesma de outros analisadores da plataforma. As energias mantêm-se em kWh. As leituras anteriores à introdução desta conversão estão em kW.
- **`thdCurrent` em percentagem.** O modelo de referência limita a distorção harmónica ao intervalo de 0 a 1. A integração publica o valor tal como o equipamento o entrega, em percentagem (0 a 100). Para obter a escala do modelo, divide-se por 100.
- **`dateObserved` e `observedAt`.** `dateObserved` é o instante da leitura no equipamento e é a referência temporal da série. `observedAt` é atribuído pelo IoT Agent na receção e pode diferir alguns minutos; as leituras do mesmo lote recebem praticamente o mesmo `observedAt`.
- **Atributos de extensão.** `serialNumber` (número de série do datalogger, para rastrear o equipamento físico), `municipality` e `circuit` (configuração do datalogger) não fazem parte do modelo de referência. São conservados porque identificam o ponto de medição enquanto não existe uma entidade `Device` associada. O valor de `municipality` é texto livre configurado no equipamento e não um código normalizado.
- **Identificador dependente da configuração.** O local que entra no identificador é configurado no datalogger. Se essa configuração mudar (por exemplo, a capitalização do nome), passa a ser publicada uma entidade nova e a anterior deixa de ser atualizada.
- **Relações não publicadas.** `refDevice` (analisador de rede) e `refTargetDevice` (equipamento alimentado) não são publicadas, porque os analisadores ainda não estão registados como entidades `Device`. `name`, `address` e `dataProvider` também não são publicados.
- **Unidades.** Os exemplos normalizados indicam a unidade com `unitCode`. Na publicação atual o IoT Agent não acrescenta `unitCode`; a unidade de cada atributo é a indicada neste documento.

**Verificações pendentes**

- Usar `dateObserved` como `observedAt`, para que a marca temporal normativa seja o instante da leitura e não o da receção.
- Normalizar `thdCurrent` (e `thdVoltage`, quando for entregue) para a escala de 0 a 1 do modelo de referência.
- Registar os analisadores como entidades `Device` e publicar `refDevice`.
- Incorporar tensões, frequência e energia reativa, necessárias para supervisionar a qualidade da alimentação e a energia reativa faturada.

## Exemplos

Os exemplos usam dados fictícios (local, número de série, circuito, valores e coordenadas aproximadas) com a estrutura real das entidades publicadas.

- [`exemplos/ThreePhaseAcMeasurement-normalized.jsonld`](exemplos/ThreePhaseAcMeasurement-normalized.jsonld): entidade NGSI-LD normalizada, tal como é publicada no *context broker*, com `unitCode` nas grandezas com unidade.
- [`exemplos/ThreePhaseAcMeasurement-keyvalues.json`](exemplos/ThreePhaseAcMeasurement-keyvalues.json): a mesma entidade em formato *key-values*, validável com o schema.
