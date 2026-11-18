# Gestão de Arvoredo

Inventário do arvoredo urbano dos municípios da Lezíria do Tejo, tal como é mantido pela CIMLT na plataforma Lezíria Verde: uma ficha por árvore, com taxonomia, medidas, localização, estado e gestão.

## Modelo de dados

| Modelo | Referência | Schema | Exemplos |
|---|---|---|---|
| `MunicipalTree` | Modelo próprio (iris360). Não existe nos Smart Data Models um modelo de inventário de arvoredo; o mais próximo, `dataModel.ParksAndGardens/FloweringTree`, não cobre a ficha de inventário. Segue as convenções dos Smart Data Models. | [`schema/MunicipalTree.json`](schema/MunicipalTree.json) | [normalizado](exemplos/MunicipalTree-normalized.jsonld) · [key-values](exemplos/MunicipalTree-keyvalues.json) |

`MunicipalTree` representa **uma árvore inventariada**. Ao contrário dos modelos de telemetria, a origem é um inventário: os dados mudam devagar e não há medições em tempo real. A entidade conserva todos os campos do inventário de origem com o seu nome original e acrescenta, por cima, uma camada de atributos canónicos (com `unitCode`) e de etiquetas legíveis dos campos codificados, para que possa ser explorada sem conhecer a API de origem.

> Este tipo **não faz parte do CNMD**. É um modelo próprio da plataforma, a validar com a ARTE.

## Origem dos dados

| | |
|---|---|
| **Fonte** | API REST da plataforma Lezíria Verde (inventário de arvoredo da CIMLT): listagem paginada de árvores, dicionário de traduções e catálogo de espécies. Só leitura, autenticação por chave |
| **Integração** | Processo ETL em Apache Airflow (extração → transformação para NGSI-LD → *upsert* por lotes no *context broker* NGSI-LD da plataforma iris360) |
| **Frequência** | Diária e incremental: só são reenviadas as árvores criadas ou alteradas desde a última passagem completa (por `dateUpdate` ou, na sua falta, `dateCreation`). O *upsert* pelo identificador torna a carga idempotente |
| **Âmbito** | Uma entidade por árvore (cerca de 23 400 em 11 municípios), publicada no espaço de dados do município a que pertence (`municipalityId`) |

## Atributos

Os atributos opcionais só são publicados quando a origem os preenche: os valores nulos e os textos vazios são omitidos, enquanto `0` e `false` são publicados, porque num inventário são informação (uma árvore sem caldeira, um risco nulo).

| Atributo | Tipo NGSI-LD | Tipo de dado | Unidade | Descrição | Campo de origem |
|---|---|---|---|---|---|
| `id` | — | URI | — | `urn:ngsi-ld:MunicipalTree:leziriaverde:<id>`, normalizado para ASCII | `id` |
| `type` | — | Texto | — | Valor fixo `MunicipalTree` | — |
| `name` | Property | Texto | — | Nome legível: nome comum; na sua falta, o científico; em último recurso, `Árvore <id>` | `commonName`, `scientificName` |
| `treeId` | Property | Inteiro | — | Identificador numérico da árvore na origem | `id` |
| `category` | Property | Lista de texto | — | Valor fixo `["MunicipalTree", "tree", "leziriaverde"]` | — |
| `source` | Property | Texto | — | Sistema de origem. Valor fixo `leziriaverde` | — |
| `sourceModule` | Property | Texto | — | Módulo da integração. Valor fixo `leziriaverde-trees` | — |
| `dataProvider` | Property | Texto | — | Valor fixo `cimlt` | — |
| `fetchedAt` | Property | Data e hora (ISO 8601 UTC) | — | Instante da leitura na origem | — |
| `species` | Property | Texto | — | Espécie (atributo canónico): nome científico; na sua falta, o comum; em último recurso, o nome do catálogo de espécies | `scientificName(Other)`, `commonName(Other)`, `treeSpeciesId` |
| `scientificName` | Property | Texto | — | Nome científico tal como vem da origem (pode valer `Outro`) | `scientificName` |
| `scientificNameOther` | Property | Texto | — | Nome científico alternativo, quando o principal é `Outro` | `scientificNameOther` |
| `commonName` | Property | Texto | — | Nome comum tal como vem da origem (pode valer `Outro`) | `commonName` |
| `commonNameOther` | Property | Texto | — | Nome comum alternativo, quando o principal é `Outro` | `commonNameOther` |
| `family` | Property | Texto | — | Família botânica | `family` |
| `treeSpeciesId` | Property | Inteiro | — | Identificador da espécie no catálogo da origem | `treeSpeciesId` |
| `location` | GeoProperty | GeoJSON `Point` | — | Coordenadas WGS84 `[longitude, latitude]`. Omitido se a árvore não tiver coordenadas | `coordinates.longitude`, `coordinates.latitude` |
| `address` | Property | Objeto (schema.org `PostalAddress`) | — | `addressLocality` (município), `postalCode`, `addressCountry` | `municipality.name`, `.postalCode`, `.country` |
| `addressLocality` | Property | Texto | — | Município (repete `address.addressLocality`) | `municipality.name` |
| `municipalityId` | Property | Inteiro | — | Identificador do município na origem | `municipalityId` |
| `parish` | Property | Texto | — | Freguesia | `parish` |
| `trunkDiameter` | Property | Número | CMT | Diâmetro à altura do peito (DAP), atributo canónico | `dbh` |
| `dbh` | Property | Número | — | Mesmo valor que `trunkDiameter`, com o nome de origem e sem unidade | `dbh` |
| `canopyDiameter` | Property | Número | MTR | Diâmetro da copa, atributo canónico | `crownDiameter` |
| `crownDiameter` | Property | Número | — | Mesmo valor que `canopyDiameter`, com o nome de origem e sem unidade | `crownDiameter` |
| `estimatedAge` | Property | Número | ANN | Idade estimada, atributo canónico | `age` |
| `age` | Property | Número | — | Mesmo valor que `estimatedAge`, com o nome de origem e sem unidade | `age` |
| `height` | Property | Número | não declarada | Altura total | `height` |
| `trunkHeight` | Property | Número | não declarada | Altura do fuste | `trunkHeight` |
| `baseDiameter` | Property | Número | não declarada | Diâmetro na base | `baseDiameter` |
| `pap` | Property | Número | não declarada | Perímetro à altura do peito (PAP) | `pap` |
| `treeType` | Property | Inteiro (código) | — | Tipo de árvore | `treeType` |
| `treeTypeLabel` | Property | Texto | — | Etiqueta de `treeType` (`Plantada`, `Espontânea`…) | dicionário da origem |
| `treeStatus` | Property | Texto (código) | — | Estado da árvore (por exemplo `alive`) | `treeStatus` |
| `treeStatusLabel` | Property | Texto | — | Etiqueta de `treeStatus` (por exemplo `Viva`) | dicionário da origem |
| `soilCondition` | Property | Inteiro (código) | — | Condição do solo | `soilCondition` |
| `soilConditionLabel` | Property | Texto | — | Etiqueta de `soilCondition` (`Normal`, `Deformado`…) | dicionário da origem |
| `soilType` | Property | Texto | — | Tipo de solo ou revestimento (texto livre) | `soilType` |
| `treeLocation` | Property | Inteiro (código) | — | Tipologia do local (sem etiqueta) | `treeLocation` |
| `treeOrigin` | Property | Inteiro (código) | — | Origem da árvore (sem etiqueta) | `treeOrigin` |
| `conservationStatus` | Property | Inteiro (código) | — | Estado de conservação (sem etiqueta) | `conservationStatus` |
| `riskLevel` | Property | Inteiro (código) | — | Nível de risco (sem etiqueta) | `riskLevel` |
| `riskNotes` | Property | Texto | — | Notas sobre o risco | `riskNotes` |
| `isHealthy` | Property | Booleano | — | Árvore sã | `isHealthy` |
| `hasTreePit` | Property | Booleano | — | Tem caldeira | `hasTreePit` |
| `pitShape` | Property | Texto | — | Forma da caldeira | `pitShape` |
| `hasAutomaticIrrigation` | Property | Booleano | — | Tem rega automática | `hasAutomaticIrrigation` |
| `ownerEntity` | Property | Inteiro | — | Entidade titular (identificador da origem) | `ownerEntity` |
| `ownerEntityInfo` | Property | Texto | — | Informação sobre a entidade titular | `ownerEntityInfo` |
| `maintenanceEntity` | Property | Inteiro | — | Entidade de manutenção (identificador da origem) | `maintenanceEntity` |
| `maintenanceEntityInfo` | Property | Texto | — | Informação sobre a entidade de manutenção | `maintenanceEntityInfo` |
| `observations` | Property | Texto | — | Observações livres | `observations` |
| `image` | Property | URL | — | Fotografia principal (a marcada como principal ou, na sua falta, a primeira) | `images[].url` |
| `imageList` | Property | Lista de URL | — | Todas as fotografias. Só quando há mais do que uma | `images[].url` |
| `datePlanted` | Property | Data e hora (ISO 8601 UTC) | — | Data de plantação | `datePlanting` |
| `dateRegistered` | Property | Data e hora (ISO 8601 UTC) | — | Data de registo no inventário | `dateRegister` |
| `dateCreated` | Property | Data e hora (ISO 8601 UTC) | — | Criação do registo na origem | `dateCreation` |
| `dateModified` | Property | Data e hora (ISO 8601 UTC) | — | Última alteração do registo na origem | `dateUpdate` |
| `externalId` | Property | Inteiro | — | Identificador externo do registo | `externalId` |
| `globalId` | Property | Texto | — | Identificador global (GUID) do registo | `globalId` |
| `status` | Property | Texto (código) | — | Estado do registo na origem (por exemplo `"1"`) | `status` |
| `dynamicFields` | Property | Lista | — | Campos livres do fornecedor. Só quando não está vazia | `dynamicFields` |

Unidades em códigos UN/CEFACT: `CMT` centímetro, `MTR` metro, `ANN` ano.

**Atributos obrigatórios:** `id`, `type`, `name`, `treeId`, `category`, `source`, `sourceModule`, `dataProvider`, `fetchedAt`.

## `@context`

Os exemplos usam o contexto *core* do NGSI-LD na versão 1.7 (`https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.7.jsonld`), que é o que a integração envia no corpo de cada entidade. Os termos próprios do modelo (`treeId`, `trunkDiameter`, `canopyDiameter`, `estimatedAge`, `fetchedAt`, os `*Label` e os campos de origem) não estão definidos em nenhum contexto publicado e expandem-se no vocabulário por omissão do NGSI-LD. Os termos genéricos (`name`, `address`, `location`, `source`, `dataProvider`, `dateCreated`, `dateModified`, `category`) correspondem aos do contexto *core* e dos Smart Data Models. Se o modelo for aceite no CNMD, os termos próprios passam a ser definidos no contexto do catálogo.

## Desvios e decisões

- **Tipo próprio.** Não existe no CNMD nem nos Smart Data Models um modelo para a ficha de inventário de uma árvore municipal. `FloweringTree` (`dataModel.ParksAndGardens`) descreve a floração de árvores ornamentais e não inclui dendrometria, risco nem caldeira. O tipo segue as convenções dos Smart Data Models: nomes canónicos, unidades UN/CEFACT, geometria como `GeoProperty` e atributos comuns (`name`, `address`, `source`, `dataProvider`).
- **Fidelidade à origem.** Todos os campos escalares do inventário são publicados com o seu nome original. As três medidas com atributo canónico são publicadas duas vezes: com o nome de origem, sem unidade (`dbh`, `crownDiameter`, `age`), e com o nome canónico e `unitCode` (`trunkDiameter`, `canopyDiameter`, `estimatedAge`). O original nunca é eliminado.
- **Unidades inferidas.** A origem não documenta unidades. `CMT`, `MTR` e `ANN` são inferências coerentes com os valores observados. `height`, `trunkHeight`, `baseDiameter` e `pap` são publicados sem `unitCode` até o fornecedor confirmar as suas unidades.
- **Códigos e etiquetas.** Os campos codificados são publicados sempre com o código em bruto. Só `treeType`, `soilCondition` e `treeStatus` têm etiqueta (`*Label`), resolvida em português contra o dicionário de traduções da origem e omitida quando não há tradução. `conservationStatus`, `treeOrigin`, `treeLocation` e `riskLevel` são publicados só como código.
- **Sem `observedAt`.** O inventário não é telemetria. O instante da leitura é publicado em `fetchedAt`.
- **Datas da origem.** Chegam sem fuso horário (`AAAA-MM-DD HH:MM:SS`) e são publicadas em ISO 8601 assumindo UTC, com nome canónico (`dateCreation` → `dateCreated`, etc.). `dateCreated` e `dateModified` referem-se ao registo na origem e não à entidade no *broker*, que mantém as suas próprias marcas de sistema (`createdAt`, `modifiedAt`).
- **Endereço.** O município é publicado em `address` e repetido em `addressLocality` para facilitar filtros. `addressCountry` leva o nome do país tal como vem da origem (`Portugal`), não o código ISO 3166.
- **`source` como texto.** Os Smart Data Models recomendam um URL; aqui é o identificador do sistema de origem (`leziriaverde`) para não expor o endereço da API.
- **`location` condicionado.** As árvores sem coordenadas são publicadas sem `location`; não se gera um ponto aproximado.
- **Diferenças com a documentação de modelação.** O modelo foi documentado com a API v1 pública e um único espaço de dados para toda a CIM. A integração atual usa a API com chave de acesso e publica cada árvore no espaço de dados do seu município. Campos que na amostra inicial chegavam sempre vazios (`parish`, `observations`, `datePlanted`, `dateRegistered`, `ownerEntity`, `maintenanceEntity`) já chegam preenchidos em parte do inventário e são publicados.
- **Pontos em aberto.** Confirmar com o fornecedor as unidades das medidas e o significado dos códigos sem etiqueta; verificar se `dateUpdate` é atualizado em todas as edições (é a base do incremental); rever `ownerEntityInfo` e `maintenanceEntityInfo` antes de chegarem preenchidos, porque podem conter nomes de pessoas. A relação com o espaço verde que agrupa as árvores fica para quando a origem expuser jardins e parques.

## Exemplos

Os exemplos usam dados fictícios (identificadores, nomes, fotografia e coordenadas aproximadas) com a estrutura real das entidades publicadas.

- [`exemplos/MunicipalTree-normalized.jsonld`](exemplos/MunicipalTree-normalized.jsonld): entidade NGSI-LD normalizada, tal como é publicada no *context broker*.
- [`exemplos/MunicipalTree-keyvalues.json`](exemplos/MunicipalTree-keyvalues.json): a mesma entidade em formato *key-values*, validável com o schema.
