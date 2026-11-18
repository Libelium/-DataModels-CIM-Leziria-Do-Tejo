# Gestão de Ocorrências

Ocorrências urbanas reportadas pelos cidadãos dos municípios da Lezíria do Tejo, a sua resolução pelos serviços municipais e os conteúdos de participação e informação associados (avisos, serviços municipais, sugestões e inquéritos de satisfação).

## Modelo de dados

| Modelo | Referência | Schema | Exemplos |
|---|---|---|---|
| `Incident` | Modelo próprio. Próximo do Smart Data Model `Open311_ServiceRequest` (dataModel.IssueTracking), sem o adotar. Equivalência no CNMD por confirmar com a ARTE. | [`schema/Incident.json`](schema/Incident.json) | [normalizado](exemplos/Incident-normalized.jsonld) · [key-values](exemplos/Incident-keyvalues.json) |
| `AssetIntervention` | Modelo próprio. | [`schema/AssetIntervention.json`](schema/AssetIntervention.json) | [normalizado](exemplos/AssetIntervention-normalized.jsonld) · [key-values](exemplos/AssetIntervention-keyvalues.json) |
| `OperatorsTeam` | Modelo próprio. | [`schema/OperatorsTeam.json`](schema/OperatorsTeam.json) | [normalizado](exemplos/OperatorsTeam-normalized.jsonld) · [key-values](exemplos/OperatorsTeam-keyvalues.json) |
| `SpecialAnnouncement` | schema.org [`SpecialAnnouncement`](https://schema.org/SpecialAnnouncement) (subconjunto, com extensões). | [`schema/SpecialAnnouncement.json`](schema/SpecialAnnouncement.json) | [normalizado](exemplos/SpecialAnnouncement-normalized.jsonld) · [key-values](exemplos/SpecialAnnouncement-keyvalues.json) |
| `PointOfInterest` | Smart Data Model [`PointOfInterest`](https://github.com/smart-data-models/dataModel.PointOfInterest) (com adaptações). | [`schema/PointOfInterest.json`](schema/PointOfInterest.json) | [normalizado](exemplos/PointOfInterest-normalized.jsonld) · [key-values](exemplos/PointOfInterest-keyvalues.json) |
| `CitizenFeedback` | Modelo próprio. | [`schema/CitizenFeedback.json`](schema/CitizenFeedback.json) | [normalizado](exemplos/CitizenFeedback-normalized.jsonld) · [key-values](exemplos/CitizenFeedback-keyvalues.json) |
| `SurveyResponse` | Modelo próprio. | [`schema/SurveyResponse.json`](schema/SurveyResponse.json) | [normalizado](exemplos/SurveyResponse-normalized.jsonld) · [key-values](exemplos/SurveyResponse-keyvalues.json) |

`Incident` é a entidade central: uma ocorrência georreferenciada com categoria, severidade, urgência e estado. Uma `AssetIntervention` agrupa uma ou mais ocorrências numa unidade de trabalho (`refersTo`) e é atribuída a um operador ou a uma `OperatorsTeam` (`assignedTeam`); o seu estado é propagado às ocorrências agrupadas. Uma ocorrência pode também referir o ativo urbano afetado (`relatedAsset`), que pertence a outra área temática (por exemplo uma árvore, `MunicipalTree`). `SpecialAnnouncement` e `PointOfInterest` são conteúdos publicados pelo município para os cidadãos (avisos e serviços municipais). `CitizenFeedback` e `SurveyResponse` recolhem a participação dos cidadãos.

> `Incident`, `AssetIntervention`, `OperatorsTeam`, `CitizenFeedback` e `SurveyResponse` **não fazem parte do CNMD** nem dos Smart Data Models. São propostos como modelos próprios, a validar com a ARTE.

## Origem dos dados

| | |
|---|---|
| **Fonte** | Aplicação do cidadão da PGU (web e móvel) e *backoffice* municipal (triagem, operadores, conteúdos) |
| **Integração** | As entidades são criadas e atualizadas diretamente pela PGU no *context broker* da plataforma iris360. Não há processo ETL nem fonte externa |
| **Frequência** | Em tempo real: cada ação (reporte, triagem, atribuição, mudança de estado, mensagem, publicação) atualiza a entidade |
| **Âmbito** | Cada entidade é publicada no espaço de dados do município (ou da CIMLT) em que é criada |
| **Identificadores** | `urn:ngsi-ld:<Tipo>:<prefixo>-<ano>-<número>`, com os prefixos `OC` (`Incident`), `AI` (`AssetIntervention`), `TEAM` (`OperatorsTeam`), `ANN` (`SpecialAnnouncement`), `POI` (`PointOfInterest`), `FB` (`CitizenFeedback`) e `SR` (`SurveyResponse`) |

## Incident

Ocorrência urbana reportada por um cidadão (ou gerada automaticamente por um sensor).

### Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Obrigatório |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:Incident:<ref>` | Sim |
| `type` | — | Texto | Valor fixo `Incident` | Sim |
| `ref` | Property | Texto | Referência legível `OC-<ano>-<número>`; é a parte final do `id` | Sim |
| `name` | Property | Texto | Nome canónico; espelha `title` | Sim |
| `title` | Property | Texto | Título visível. Se não for indicado, deriva da descrição ou da referência | Sim |
| `description` | Property | Texto | Descrição livre (10 a 2000 caracteres) | Sim |
| `category` | Property | Texto | Código da categoria, segundo a taxonomia de ocorrências do município (alinhada com o CNMD) | Sim |
| `subCategory` | Property | Texto | Código da subcategoria | Não |
| `status` | Property | Texto | Estado do ciclo de vida (ver tabela abaixo). Criada como `open` | Sim |
| `privacy` | Property | Texto | `public` (visível para os cidadãos) ou `private` (interna) | Sim |
| `source` | Property | Texto | Canal de origem (por exemplo `web`, aplicação móvel, sensor) | Sim |
| `dateCreated` | Property | Data/hora ISO 8601 | Data de criação | Sim |
| `location` | Property | GeoJSON `Point` | Coordenadas WGS84 `[longitude, latitude]`. Sem localização não é possível criar a ocorrência | Sim |
| `severity` | Property | Texto | `low`, `moderate`, `high` ou `critical` | Não |
| `urgencyLevel` | Property | Texto | `P1` (máxima) a `P4`. `critical` corresponde a `P1` | Não |
| `address` | Property | Texto | Morada legível, obtida por geocodificação inversa | Não |
| `locationValidated` | Property | Booleano | `true` se a posição provém de um GPS válido ou de um marcador colocado manualmente | Não |
| `locationAccuracy` | Property | Número (metros) | Precisão da posição GPS | Não |
| `reportedBy` | Property | Texto | Identificador interno do utilizador que reportou | Não |
| `notifyConsent` | Property | Booleano | Consentimento do autor para receber notificações de mudança de estado (por omissão `true`) | Sim |
| `images` | Property | Lista de texto | Chaves de armazenamento das fotografias (máximo 6) | Não |
| `publicNote` | Property | Texto | Nota pública apresentada quando a ocorrência fica `resolved` ou `closed` | Não |
| `resolvedAt` | Property | Data/hora ISO 8601 | Momento em que o operador resolveu ou encerrou a ocorrência | Não |
| `expectedResponseTime` | Property | Texto | Tempo de resposta esperado (informativo) | Não |
| `workStatus` | Property | Texto | Estado operacional da ordem de trabalho (ver tabela abaixo) | Não |
| `relatedAsset` | Property | URI | URN do ativo urbano associado (por exemplo uma `MunicipalTree`) | Não |
| `relatedAssetType` | Property | Texto | Tipo de entidade do ativo associado | Não |
| `relatedAssetName` | Property | Texto | Nome legível do ativo associado | Não |
| `relatedAssetId` | Property | Texto | Identificador local do ativo associado na plataforma | Não |
| `chatMessage` | Property | Objeto `{message, sender: {username, role}}` | Última mensagem entre o cidadão e o município. `username` é pseudonimizado; `role` é `citizen` ou `municipality`. O histórico é a série temporal do atributo | Não |
| `assignedTo` | Property | Texto | Identificador interno do operador atribuído | Não |
| `assignedTeam` | Property | Texto | Identificador da equipa (`OperatorsTeam`) atribuída, ou destino em texto livre quando a ocorrência é reencaminhada ou escalada | Não |
| `assignedTeamName` | Property | Texto | Nome legível da equipa atribuída | Não |
| `startedAt` | Property | Data/hora ISO 8601 | Início da intervenção: registado na primeira passagem a `inProgress` | Não |
| `slaDue` | Property | Data/hora ISO 8601 | Prazo de resolução. Quando não existe, é calculado a partir da urgência | Não |
| `internalNotes` | Property | Texto | Nota interna do operador (motivo do reencaminhamento, do escalamento ou da derivação). Não é mostrada ao cidadão | Não |
| `externalEntity` | Property | Texto | Nome da entidade externa (por exemplo forças de segurança ou bombeiros) para a qual a ocorrência foi derivada (`workStatus` `external`) | Não |
| `externalEntityId` | Property | Inteiro | Identificador interno do destino de derivação configurado na plataforma | Não |
| `followUpSlaDue` | Property | Data/hora ISO 8601 | Prazo de acompanhamento de uma derivação externa. Quando expira, a ocorrência é encerrada | Não |
| `resolvedBy` | Property | Texto | Operador que resolveu ou encerrou a ocorrência (nome ou identificador interno) | Não |
| `resolutionNotes` | Property | Texto | Notas internas de resolução. Não são mostradas ao cidadão | Não |

**Estados (`status`) e estados operacionais (`workStatus`)**

| `status` | Significado | `workStatus` correspondentes |
|---|---|---|
| `open` | Recebida, por tratar | `queued` |
| `assigned` | Atribuída a um operador ou equipa | `assigned` |
| `inProgress` | Em tratamento | `accepted`, `inProgress`, `paused`, `forwarded`, `escalated`, `external` |
| `resolved` | Resolvida pelo município | `resolved` |
| `closed` | Encerrada | `noAction` |
| `cancelled` | Anulada | `noAction` |

`category`, `severity`, `urgencyLevel` e `status` resultam da triagem municipal: o cidadão não os altera depois de criar a ocorrência.

**Atributos obrigatórios:** `id`, `type`, `ref`, `name`, `title`, `description`, `category`, `status`, `privacy`, `source`, `dateCreated`, `location`, `notifyConsent`.

## AssetIntervention

Unidade de trabalho que agrupa uma ou mais ocorrências (um grupo de uma só ocorrência é admitido) e é atribuída a um operador e/ou a uma equipa. Cada ocorrência pertence no máximo a uma intervenção. Ao agrupar as ocorrências, estas passam a `assigned`, e cada mudança de estado da intervenção é propagada a todas elas.

### Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Obrigatório |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:AssetIntervention:AI-<ano>-<número>` | Sim |
| `type` | — | Texto | Valor fixo `AssetIntervention` | Sim |
| `name` | Property | Texto | Nome da intervenção | Sim |
| `status` | Property | Texto | `open`, `inProgress`, `assigned`, `resolved`, `closed` ou `cancelled`. Criada como `open` | Sim |
| `operatorId` | Property | Texto | Identificador interno do operador que criou a intervenção | Sim |
| `refersTo` | Property | Lista de URI | URN das ocorrências (`Incident`) agrupadas | Sim |
| `location` | Property | GeoJSON `Point` | Centroide das localizações das ocorrências agrupadas, calculado pela plataforma | Não |
| `category` | Property | Texto | Categoria (normalmente herdada das ocorrências) | Não |
| `assignedTo` | Property | Texto | Identificador interno do operador atribuído | Não |
| `assignedToName` | Property | Texto | Nome de apresentação do operador atribuído | Não |
| `assignedTeam` | Property | Texto | Identificador da equipa (`OperatorsTeam`) atribuída | Não |
| `assignedTeamName` | Property | Texto | Nome de apresentação da equipa atribuída | Não |
| `assignmentType` | Property | Texto | `individual` (operador) ou `team` (equipa) | Não |
| `assignedAt` | Property | Data/hora ISO 8601 | Momento da atribuição | Não |

**Atributos obrigatórios:** `id`, `type`, `name`, `status`, `operatorId`, `refersTo`.

## OperatorsTeam

Equipa ou piquete de operadores municipais, usada para atribuir intervenções a um grupo. Não tem localização.

### Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Obrigatório |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:OperatorsTeam:TEAM-<ano>-<número>` | Sim |
| `type` | — | Texto | Valor fixo `OperatorsTeam` | Sim |
| `name` | Property | Texto | Nome da equipa | Sim |
| `members` | Property | Lista de texto | Identificadores internos dos operadores membros | Sim |

O número de ordens de uma equipa não é guardado: calcula-se contando as intervenções com `assignedTeam` igual à equipa.

**Atributos obrigatórios:** `id`, `type`, `name`, `members`.

## SpecialAnnouncement

Aviso municipal operacional e temporário, bilingue, apresentado aos cidadãos e usado nas notificações.

### Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Obrigatório |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:SpecialAnnouncement:ANN-<ano>-<número>` | Sim |
| `type` | — | Texto | Valor fixo `SpecialAnnouncement` | Sim |
| `name` | Property | Objeto `{pt, en}` | Título bilingue | Sim |
| `text` | Property | Objeto `{pt, en}` | Conteúdo bilingue | Sim |
| `category` | Property | Texto | `urgente`, `transito` ou `evento`. A cor e o ícone derivam da categoria e não são guardados | Sim |
| `datePosted` | Property | Data/hora ISO 8601 | Data de publicação (por omissão, o momento da criação) | Sim |
| `source` | Property | Texto | Canal de origem (`backoffice`) | Sim |
| `expires` | Property | Data/hora ISO 8601 | Fim de vigência; os avisos expirados deixam de ser apresentados | Não |
| `address` | Property | Texto | Localização legível | Não |
| `announcementLocation` | Property | GeoJSON `Point` | Ponto no mapa `[longitude, latitude]` | Não |
| `hidden` | Property | Booleano | Arquivado: oculto para os cidadãos, visível para os administradores (por omissão `false`) | Não |
| `images` | Property | Lista de texto | Chaves de armazenamento das imagens anexadas | Não |

**Atributos obrigatórios:** `id`, `type`, `name`, `text`, `category`, `datePosted`, `source`.

## PointOfInterest

Serviço municipal de referência apresentado aos cidadãos (águas, resíduos, iluminação, espaços verdes, emergência, farmácias), com localização, contacto e horário.

### Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Obrigatório |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:PointOfInterest:POI-<ano>-<número>` | Sim |
| `type` | — | Texto | Valor fixo `PointOfInterest` | Sim |
| `name` | Property | Texto | Nome do serviço | Sim |
| `category` | Property | Texto | `aguas`, `residuos`, `iluminacao`, `verdes`, `emergencia` ou `pharmacies` | Sim |
| `location` | Property | GeoJSON `Point` | Coordenadas WGS84 `[longitude, latitude]` | Sim |
| `source` | Property | Texto | Canal de origem (`backoffice`) | Sim |
| `address` | Property | Texto | Morada legível | Não |
| `telephone` | Property | Texto | Telefone de contacto | Não |
| `openingHours` | Property | Texto ou lista de texto | Horário na sintaxe schema.org, em hora local: `24/7`, uma regra (`Mo-Fr 09:00-18:00`) ou uma lista de regras | Não |
| `description` | Property | Texto | Descrição do serviço | Não |
| `images` | Property | Lista de texto | Chaves de armazenamento das imagens | Não |

O estado «aberto agora» ou «24 horas» não é guardado: calcula-se a partir de `openingHours`.

**Atributos obrigatórios:** `id`, `type`, `name`, `category`, `location`, `source`.

## CitizenFeedback

Sugestão ou comentário de um cidadão. O mesmo modelo é usado para as denúncias de conteúdo enviadas para moderação, que se distinguem pelo valor de `topic`.

### Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Obrigatório |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:CitizenFeedback:FB-<ano>-<número>` | Sim |
| `type` | — | Texto | Valor fixo `CitizenFeedback` | Sim |
| `topic` | Property | Texto | `suggestion`, `praise`, `complaint`, `app_issue`; `content_report` para as denúncias de conteúdo | Sim |
| `message` | Property | Texto | Texto do feedback ou da denúncia | Sim |
| `anonymous` | Property | Booleano | `true` salvo se o cidadão indicar o nome | Sim |
| `submittedAt` | Property | Data/hora ISO 8601 | Data de envio | Sim |
| `channel` | Property | Texto | Canal de origem (por exemplo `citizen-app`) | Sim |
| `status` | Property | Texto | Estado de triagem; criado como `new` | Sim |
| `submitterName` | Property | Texto | Nome do remetente, apenas se se identificar | Não |
| `contactEmail` | Property | Texto (correio eletrónico) | Endereço para resposta, indicado voluntariamente e apenas em envios não anónimos | Não |

**Atributos obrigatórios:** `id`, `type`, `topic`, `message`, `anonymous`, `submittedAt`, `channel`, `status`.

## SurveyResponse

Resposta de um cidadão a um inquérito de satisfação municipal, identificada ou anónima.

### Atributos

| Atributo | Tipo NGSI-LD | Tipo de dado | Descrição | Obrigatório |
|---|---|---|---|---|
| `id` | — | URI | `urn:ngsi-ld:SurveyResponse:SR-<ano>-<número>` | Sim |
| `type` | — | Texto | Valor fixo `SurveyResponse` | Sim |
| `survey` | Property | Texto | Identificador do inquérito respondido | Sim |
| `answers` | Property | Lista de objetos `{q, value}` | Uma resposta por pergunta: identificador da pergunta e valor numérico | Sim |
| `anonymous` | Property | Booleano | `true` salvo se o cidadão indicar nome ou correio eletrónico | Sim |
| `submittedAt` | Property | Data/hora ISO 8601 | Data de envio | Sim |
| `channel` | Property | Texto | Canal de origem (por exemplo `citizen-app`) | Sim |
| `respondentName` | Property | Texto | Nome do respondente, apenas se se identificar | Não |
| `respondentEmail` | Property | Texto (correio eletrónico) | Correio eletrónico do respondente, apenas se se identificar | Não |

**Atributos obrigatórios:** `id`, `type`, `survey`, `answers`, `anonymous`, `submittedAt`, `channel`.

## `@context`

Os exemplos usam o contexto *core* do NGSI-LD. Os termos próprios dos modelos (`ref`, `subCategory`, `urgencyLevel`, `workStatus`, `refersTo`, `members`, `topic`, `answers`, etc.) não estão definidos em nenhum contexto publicado e expandem-se no vocabulário por omissão do NGSI-LD. Os termos genéricos (`name`, `description`, `address`, `location`, `dateCreated`, `source`) correspondem aos do contexto *core* e dos Smart Data Models.

- `PointOfInterest` inclui também o contexto do Smart Data Model ([`dataModel.PointOfInterest/context.jsonld`](https://raw.githubusercontent.com/smart-data-models/dataModel.PointOfInterest/master/context.jsonld)), antes do contexto *core*.
- `SpecialAnnouncement` usa os nomes de propriedade do tipo schema.org (`name`, `text`, `category`, `datePosted`, `expires`, `announcementLocation`), mas não carrega o contexto schema.org: esses termos expandem-se no vocabulário por omissão.

Se os modelos próprios forem aceites no CNMD, os seus termos passam a ser definidos no contexto do catálogo.

## Desvios e decisões

### Convenções transversais

- **Localização como `Property`, não `GeoProperty`.** A camada de publicação da plataforma não aceita atributos `GeoProperty`. Por isso `location` (e `announcementLocation`) é publicada como `Property` cujo valor é um GeoJSON `Point` com coordenadas `[longitude, latitude]`. O conteúdo é GeoJSON válido, idêntico ao de uma `GeoProperty`, mas o *context broker* não o indexa como geometria; as consultas geográficas fazem-se na plataforma. Quando a limitação for resolvida, basta mudar o tipo do atributo para `GeoProperty`, sem alterar o valor.
- **Textos bilingues como objeto `{pt, en}`, não `LanguageProperty`.** Pela mesma razão, os textos bilingues (`name` e `text` de `SpecialAnnouncement`) são publicados como `Property` com o valor `{"pt": "…", "en": "…"}`, em vez de `LanguageProperty` com `languageMap`. A estrutura é a mesma do `languageMap`, pelo que a conversão é direta. O schema aceita também um texto simples, usado nos avisos publicados numa só língua.
- **Referências entre entidades como `Property` com URN.** A camada de publicação também não propaga o `object` das `Relationship`. As referências a outras entidades (`refersTo` de `AssetIntervention`, `relatedAsset` de `Incident`) são publicadas como `Property` cujo valor é o URN (ou a lista de URN) da entidade referida. Semanticamente são relações e deverão passar a `Relationship` quando a limitação for resolvida.
- **Identificadores de utilizadores e equipas.** `reportedBy`, `operatorId`, `assignedTo`, `members` e `assignedTeam` contêm identificadores internos da plataforma e não URN NGSI-LD, porque os utilizadores não são entidades do *context broker*.
- **Dados pessoais.** A identidade do autor de uma ocorrência não é guardada na entidade (apenas `reportedBy`), e o autor das mensagens é pseudonimizado. Os únicos dados pessoais de cidadãos são `submitterName`, `contactEmail`, `respondentName` e `respondentEmail`, publicados apenas quando o cidadão opta por se identificar. Do lado municipal, `resolvedBy` pode conter o nome do operador.

### Incident

- **Modelo próprio.** O Smart Data Model `Open311_ServiceRequest` cobre o mesmo domínio (pedido de serviço do cidadão) e há correspondência entre os principais atributos: `ref` ↔ `service_request_id`, `category`/`subCategory` ↔ `service_code`, `description` ↔ `description`, `status` ↔ `status`, `dateCreated` ↔ `requested_datetime`, `address` ↔ `address_string`, `images` ↔ `media_url`, `publicNote` ↔ `status_notes`. Não foi adotado porque reproduz os campos da API Open311 (nomes em *snake_case* e só dois estados, `open` e `closed`) e não contempla a triagem (severidade, urgência), a privacidade, o consentimento de notificação nem a ligação a ativos.
- **Categorias.** `category` e `subCategory` são códigos da taxonomia de ocorrências do município, alinhada com a taxonomia do CNMD. A taxonomia (incluindo o prazo de resposta de cada subcategoria) é configuração da plataforma e não faz parte da entidade.
- **Estados.** São usados seis estados; `assigned` é uma extensão aos cinco estados do dicionário canónico de referência. `workStatus` detalha o estado operacional e cada valor corresponde a um `status` (ver tabela).
- **Prazo de resposta.** Quando o prazo é fixado, fica em `slaDue`. Sem `slaDue`, o prazo calcula-se a partir da taxonomia e da urgência (P1 2 h, P2 6 h, P3 24 h, P4 72 h). `expectedResponseTime` é apenas informativo.
- **Atributos operacionais.** A atribuição (`assignedTo`, `assignedTeam`, `assignedTeamName`), o início e os prazos (`startedAt`, `slaDue`, `followUpSlaDue`), a derivação para entidades externas (`externalEntity`, `externalEntityId`) e a resolução (`resolvedBy`, `resolutionNotes`, `internalNotes`) são escritos pela aplicação do operador ao longo do ciclo de vida. São publicados na entidade, mas as notas internas não são mostradas ao cidadão.
- **`resolvedAt`.** É o atributo usado para a data de resolução (não é usado `dateResolved`).
- **`images`.** Lista de chaves de armazenamento. Algumas ocorrências mais antigas contêm uma única chave como texto, e o schema aceita as duas formas.
- **`relatedAssetType`, `relatedAssetName` e `relatedAssetId`** duplicam informação do ativo referido em `relatedAsset`, para apresentação sem consultar a outra entidade.
- **`chatMessage`.** Cada mensagem substitui o valor do atributo; o histórico da conversa é a série temporal do atributo.

### AssetIntervention

- **Modelo próprio.** Não existe no CNMD nem nos Smart Data Models um modelo para agrupar pedidos de cidadãos numa ordem de trabalho.
- **`location`** não é indicada pelo operador: a plataforma calcula-a como o centroide das ocorrências agrupadas e atualiza-a quando o grupo muda.
- **Nota pública de encerramento.** Ao encerrar a intervenção, a nota pública é escrita no `publicNote` de cada ocorrência agrupada, onde o cidadão a vê. Não é guardada na própria intervenção.
- **`assignedToName` e `assignedTeamName`** são cópias de apresentação; a referência é `assignedTo` / `assignedTeam`.

### OperatorsTeam

- **Modelo próprio.** Foi considerado o Smart Data Model `Organization`, mas uma equipa de operadores é um agrupamento interno de utilizadores, sem identidade jurídica, morada nem contactos próprios.

### SpecialAnnouncement

- **Base schema.org.** São usados `name`, `text`, `category`, `datePosted`, `expires` e `announcementLocation` de schema.org `SpecialAnnouncement`. No schema.org, `category` é um URL ou `Thing`; aqui é um código de texto (`urgente`, `transito`, `evento`). `announcementLocation` é um ponto GeoJSON em vez de um `Place`.
- **Extensões:** `source`, `address`, `hidden` e `images`.

### PointOfInterest

- **Base Smart Data Models**, com estas adaptações:
  - `category` é um texto único em vez de uma lista;
  - `address` é um texto em vez de um objeto `PostalAddress`;
  - o telefone é publicado em `telephone` em vez de `contactPoint.telephone`;
  - `openingHours` (de `DateTime-Commons`) pode ser uma lista de regras, uma por grupo de dias;
  - `images` (lista de chaves de armazenamento) é usado em vez de `image` (URL).

### CitizenFeedback e SurveyResponse

- **Modelos próprios.** Não existe no CNMD nem nos Smart Data Models um modelo para sugestões de cidadãos ou respostas a inquéritos de satisfação.
- **Denúncias de conteúdo.** As denúncias para moderação usam `CitizenFeedback` com `topic` = `content_report`, em vez de um modelo próprio.
- **`answers`** guarda só o identificador da pergunta e o valor numérico; o texto das perguntas pertence à definição do inquérito, identificada por `survey`.

## Exemplos

Os exemplos usam dados fictícios (referências, moradas, nomes, contactos e coordenadas aproximadas) com a estrutura real das entidades publicadas.

| Modelo | Normalizado (publicado no *context broker*) | *Key-values* (validável com o schema) |
|---|---|---|
| `Incident` | [`exemplos/Incident-normalized.jsonld`](exemplos/Incident-normalized.jsonld) | [`exemplos/Incident-keyvalues.json`](exemplos/Incident-keyvalues.json) |
| `AssetIntervention` | [`exemplos/AssetIntervention-normalized.jsonld`](exemplos/AssetIntervention-normalized.jsonld) | [`exemplos/AssetIntervention-keyvalues.json`](exemplos/AssetIntervention-keyvalues.json) |
| `OperatorsTeam` | [`exemplos/OperatorsTeam-normalized.jsonld`](exemplos/OperatorsTeam-normalized.jsonld) | [`exemplos/OperatorsTeam-keyvalues.json`](exemplos/OperatorsTeam-keyvalues.json) |
| `SpecialAnnouncement` | [`exemplos/SpecialAnnouncement-normalized.jsonld`](exemplos/SpecialAnnouncement-normalized.jsonld) | [`exemplos/SpecialAnnouncement-keyvalues.json`](exemplos/SpecialAnnouncement-keyvalues.json) |
| `PointOfInterest` | [`exemplos/PointOfInterest-normalized.jsonld`](exemplos/PointOfInterest-normalized.jsonld) | [`exemplos/PointOfInterest-keyvalues.json`](exemplos/PointOfInterest-keyvalues.json) |
| `CitizenFeedback` | [`exemplos/CitizenFeedback-normalized.jsonld`](exemplos/CitizenFeedback-normalized.jsonld) | [`exemplos/CitizenFeedback-keyvalues.json`](exemplos/CitizenFeedback-keyvalues.json) |
| `SurveyResponse` | [`exemplos/SurveyResponse-normalized.jsonld`](exemplos/SurveyResponse-normalized.jsonld) | [`exemplos/SurveyResponse-keyvalues.json`](exemplos/SurveyResponse-keyvalues.json) |
