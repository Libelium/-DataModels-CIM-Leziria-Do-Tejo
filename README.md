# Modelos de Dados — Plataforma de Gestão Urbana da Lezíria do Tejo

<p align="center">
  <img src="https://img.shields.io/badge/estado-em%20revis%C3%A3o%20ARTE-orange" alt="Estado: em revisão ARTE">
  <img src="https://img.shields.io/badge/NGSI--LD-ETSI%20GS%20CIM%20009-blue" alt="NGSI-LD">
  <img src="https://img.shields.io/badge/formato-JSON%20Schema-000000?logo=json" alt="Formato: JSON Schema">
  <img src="https://img.shields.io/badge/idioma-PT--PT-green" alt="Idioma: PT-PT">
</p>

Este repositório reúne os **modelos de dados utilizados na Plataforma de Gestão Urbana (PGU) da Comunidade Intermunicipal da Lezíria do Tejo (CIMLT)**, desenvolvida no âmbito do concurso público **CP/08/2025** e implementada sobre a plataforma **iris360** da Libelium.

O objetivo do repositório é disponibilizar à **[ARTE — Territórios Inteligentes](https://territoriosinteligentes.gov.pt)** os esquemas e exemplos de cada modelo de dados, de forma a permitir a sua revisão e a verificação do alinhamento com o **[Catálogo Nacional de Modelos de Dados (CNMD)](https://territoriosinteligentes.gov.pt)** e, quando aplicável, com os **[Smart Data Models](https://smartdatamodels.org)** da FIWARE Foundation.

## Âmbito

A PGU integra, para a CIMLT e para os seus 11 municípios (Almeirim, Alpiarça, Azambuja, Benavente, Cartaxo, Chamusca, Coruche, Golegã, Rio Maior, Salvaterra de Magos e Santarém), os dados provenientes das seguintes áreas temáticas:

| Área temática | Modelo(s) de dados | Referência |
|---|---|---|
| [Gestão de Arvoredo](Gest%C3%A3o%20de%20Arvoredo/) | `MunicipalTree` | Modelo próprio iris360 (o mais próximo nos Smart Data Models é `FloweringTree`, que não cobre o inventário) |
| [Gestão de Elevadores](Gest%C3%A3o%20de%20Elevadores/) | `LiftRegistration` | Modelo próprio (proposta), segundo o padrão `*Registration` do CNMD |
| [Gestão de Ocorrências](Gest%C3%A3o%20de%20Ocorr%C3%AAncias/) | `Incident`, `AssetIntervention`, `OperatorsTeam`, `SpecialAnnouncement`, `PointOfInterest`, `CitizenFeedback`, `SurveyResponse` | Modelos próprios; Smart Data Models `PointOfInterest` (adaptado); schema.org `SpecialAnnouncement` |
| [Monitorização do Consumo de Água](Monitoriza%C3%A7%C3%A3o%20do%20Consumo%20de%20%C3%81gua/) | `Device`, `WaterConsumptionObserved` | Smart Data Models `dataModel.Device`, `dataModel.WaterConsumption` |
| [Monitorização Energética](Monitoriza%C3%A7%C3%A3o%20Energ%C3%A9tica/) | `ThreePhaseAcMeasurement` | Smart Data Models `dataModel.Energy` |
| [Payment Analytics](Payment%20Analytics/) | `CardConsumptionObserved` | Modelo próprio (proposta), segundo o padrão `*Observed` dos Smart Data Models |
| [Qualidade do Ar e Som](Qualidade%20do%20Ar%20e%20Som/) | `AirQualityObserved`, `NoiseLevelObserved` | Smart Data Models `dataModel.Environment` (modelo previsto; ainda sem integração ativa) |
| [Regas Inteligentes](Regas%20Inteligentes/) | `WeatherObserved`, `Device`, `AgriParcel`, `AgriParcelOperation`, `WaterConsumptionObserved` | Smart Data Models `dataModel.Weather`, `dataModel.Device`, `dataModel.AgriFood`, `dataModel.WaterConsumption` |

> Sempre que existe um modelo equivalente no CNMD, foi esse o modelo adotado. Nos restantes casos foi utilizado o modelo correspondente dos Smart Data Models ou, na sua ausência, um modelo próprio da plataforma iris360. Eventuais atributos adicionais ou adaptações estão identificados no `README.md` de cada área temática.

## Organização

O repositório está organizado por áreas temáticas, correspondentes às verticais da PGU. Para cada área temática existe:

- um ficheiro `README.md`, que descreve o modelo (ou modelos) usado na área temática, os seus atributos, a origem dos dados e as eventuais diferenças em relação ao modelo de referência (CNMD ou Smart Data Models);
- uma diretoria `exemplos`, que contém um ou mais exemplos de entidades tal como são publicadas na PGU (anonimizados quando necessário);
- uma diretoria `schema`, que contém os JSON Schemas que permitem validar o(s) modelo(s).

```
DataModels-CIM-Leziria-Do-Tejo/
├── README.md
├── Gestão de Arvoredo/
│   ├── README.md
│   ├── exemplos/
│   └── schema/
├── Gestão de Elevadores/
│   ├── README.md
│   ├── exemplos/
│   └── schema/
├── Gestão de Ocorrências/
├── Monitorização do Consumo de Água/
├── Monitorização Energética/
├── Payment Analytics/
├── Qualidade do Ar e Som/
└── Regas Inteligentes/
```

## Convenções

- **Protocolo:** os modelos são publicados através da API NGSI-LD (ETSI GS CIM 009) do _context broker_ da plataforma iris360.
- **Identificadores:** as entidades são identificadas por URN, no formato `urn:ngsi-ld:<Tipo>:<identificador>`.
- **Contexto JSON-LD:** cada exemplo inclui o `@context` utilizado, com referência ao contexto do modelo de origem.
- **Datas e horas:** formato ISO 8601 em UTC.
- **Georreferenciação:** os atributos `location` são publicados em GeoJSON (WGS84), conforme a especificação NGSI-LD.
- **Unidades:** sempre que possível, indicadas através de `unitCode` (códigos UN/CEFACT).
