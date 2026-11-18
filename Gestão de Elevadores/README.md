# Gestão de Elevadores

Cadastro das instalações de elevação (ascensores e monta-cargas) dos municípios da Lezíria do Tejo, tal como é mantido pela CIMLT.

## Modelo de dados

| Modelo | Referência | Schema | Exemplos |
|---|---|---|---|
| `LiftRegistration` | Modelo próprio (proposta). Não existe no CNMD nem nos Smart Data Models. | [`schema/LiftRegistration.json`](schema/LiftRegistration.json) | [normalizado](exemplos/LiftRegistration-normalized.jsonld) · [key-values](exemplos/LiftRegistration-keyvalues.json) |

`LiftRegistration` representa o **registo cadastral** de uma instalação de elevação, ou seja, o seu processo administrativo e não o dispositivo físico. O nome segue o padrão do CNMD para cadastros (`BuildingRegistration`, `UnitRegistration`): tal como `BuildingRegistration` é o registo de um edifício, `LiftRegistration` é o registo de um equipamento de elevação.

> Este tipo **não faz parte do CNMD**. É proposto como modelo novo, a validar com a ARTE.

## Origem dos dados

| | |
|---|---|
| **Fonte** | API de equipamentos do cadastro de elevadores da CIMLT (`GET /api/v1/equipamentos`, paginada, autenticação por chave) |
| **Integração** | Processo ETL em Apache Airflow (extração → encaminhamento por município → carga através do IoT Agent da plataforma iris360) |
| **Frequência** | Semanal. Carga completa e idempotente: o número de processo identifica a entidade |
| **Âmbito** | Uma entidade por equipamento, publicada no espaço de dados do município a que pertence |

## Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Campo de origem |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:LiftRegistration:<processNumber>` | `process_number` |
| `type` | — | Texto | Valor fixo `LiftRegistration` | — |
| `name` | Property | Texto | Nome legível: `"<liftType> — <morada>"` | derivado de `type.name` e `address` |
| `processNumber` | Property | Inteiro | Número de processo oficial, único e imutável | `process_number` |
| `liftType` | Property | Texto | Tipo de equipamento (`Ascensores`, `Monta-cargas`) | `type.name` |
| `registrationStatus` | Property | Objeto `{value, label}` | Estado do registo (ver tabela abaixo) | `status` |
| `address` | Property | Objeto (schema.org `PostalAddress`) | `addressCountry`, `addressLocality` (município), `streetAddress`, `postalCode` | `address`, `postal_code`, `municipality.name` |
| `location` | GeoProperty | GeoJSON `Point` | Coordenadas WGS84 `[longitude, latitude]`. Só é publicado quando o cadastro tem coordenadas | `lat`, `lon` |
| `geocodeStatus` | Property | Texto | Confiança das coordenadas: `pending_confirmation` ou `confirmed`. Acompanha `location` | `geocode_status.value` |
| `maintenanceCompany` | Property | Objeto `{name, license}` | Empresa de manutenção (EMIE). Omitido quando o cadastro não indica EMIE | `emie.name`, `emie.ref` |
| `dataProvider` | Property | Texto | Valor fixo `CIMLT` | — |

**Estados do registo (`registrationStatus.value`)**

| Código | Rótulo |
|---|---|
| 0 | A Aguardar Aprovação |
| 1 | Operacional |
| 60 | Manutenção |
| 99 | Selado |
| 100 | Arquivado |

**Atributos obrigatórios:** `id`, `type`, `name`, `processNumber`, `liftType`, `registrationStatus`, `address`, `dataProvider`.

## `@context`

Os exemplos usam o contexto *core* do NGSI-LD. Os termos próprios do modelo (`processNumber`, `liftType`, `registrationStatus`, `geocodeStatus`, `maintenanceCompany`) não estão definidos em nenhum contexto publicado e expandem-se no vocabulário por omissão do NGSI-LD. Os termos genéricos (`name`, `address`, `location`, `dataProvider`) correspondem aos do contexto *core* e dos Smart Data Models. Se o modelo for aceite no CNMD, os termos próprios passam a ser definidos no contexto do catálogo.

## Desvios e decisões

- **Tipo próprio.** Foram avaliados os modelos do CNMD e dos Smart Data Models, e nenhum representa um registo cadastral de equipamentos de elevação. O tipo segue a nomenclatura `*Registration` do CNMD.
- **Empresa de manutenção como `Property`.** A CIMLT fornece a EMIE como texto (nome e referência) e não como entidade. Uma `Relationship` para uma `Organization` inexistente seria uma referência pendente. No futuro poderá passar a `Relationship` sem alterar o resto do modelo.
- **`registrationStatus`** mantém o objeto `{value, label}` do cadastro, porque é o estado do registo e não o estado de serviço do equipamento.
- **`liftType`** é publicado como texto (o nome do tipo), sem o identificador numérico do cadastro.
- **`location` condicionado.** Os registos sem coordenadas são publicados sem `location` e completados quando são georreferenciados. `geocodeStatus` indica se as coordenadas foram confirmadas por uma pessoa.
- **Relações futuras.** `refBuilding` (para `BuildingRegistration`) não é incluída enquanto não existir o cadastro de edifícios na plataforma.

## Exemplos

Os exemplos usam dados fictícios (número de processo, morada, empresa e coordenadas aproximadas) com a estrutura real das entidades publicadas.

- [`exemplos/LiftRegistration-normalized.jsonld`](exemplos/LiftRegistration-normalized.jsonld): entidade NGSI-LD normalizada, tal como é publicada no *context broker*.
- [`exemplos/LiftRegistration-keyvalues.json`](exemplos/LiftRegistration-keyvalues.json): a mesma entidade em formato *key-values*, validável com o schema.
