# Conceitos e Modelo de Dados

## Flow

Um **Flow** é a definição (a "receita") de uma jornada de formalização: uma lista ordenada de *steps* que serão executados em sequência para cada transação.

| Campo | Descrição |
|---|---|
| `flow_id` | Identificador único do Flow |
| `name` | Nome do Flow |
| `status` | `draft` \| `published` \| `unpublished` \| `deleted` |
| `steps[]` | Lista ordenada de `FlowStep` |
| `account` | Conta proprietária do Flow |
| `user_id` | Usuário que criou/mantém o Flow |
| `created_at` / `updated_at` | Timestamps |

> **⚠️ Mudança de contrato (13/07/2026).** O valor `archived` não existe mais. Em seu lugar, o ciclo de vida ganhou dois estados: `unpublished` (via `PATCH /flows/{id}/unpublish`) e `deleted` (soft delete via `DELETE /flows/{id}`).

### Ciclo de vida do status

```
                    ┌──────────────(unpublish)──────────────┐
                    ▼                                        │
draft ──(publish)──► published                        unpublished
  │                     │                                    │
  │                     └──────────────(publish)──────────────┘
  │
  └──(delete)──► deleted            (delete também é possível a partir de qualquer status)
```

- Um Flow nasce em `draft`.
- Só é possível **editar** (`PUT`) um Flow enquanto ele está em `draft` ou `unpublished`. Um Flow `published` retorna `403 Forbidden` a uma tentativa de `PUT` — despublique antes (`PATCH /flows/{id}/unpublish`) para voltar a editá-lo, depois publique de novo. Isso garante que transações já em andamento não sejam afetadas por mudanças no meio do caminho.
- Só é possível **iniciar execuções** (`POST /executions`) de um Flow com status `published`.
- `DELETE /flows/{id}` é um soft delete — o Flow passa para `deleted`. Não há hard delete no contrato público.

## FlowStep

Cada item da lista `steps[]` de um Flow tem:

| Campo | Descrição |
|---|---|
| `type` | `acceptance` \| `form` \| `verify` \| `kyc` \| `signature` |
| `context` | Objeto de configuração específico do tipo (schema livre por tipo) |

### Regras de validação e campos por tipo

- **`acceptance`**: `context = { template_key, accept_replies[] }`. A estrutura é validada no **Sequencer**; o *match* da resposta do usuário (comparação com `accept_replies`) é validado no **Runner**.
  ```json
  { "type": "acceptance", "context": { "template_key": "chave-do-template", "accept_replies": ["accept", "aceitar"] } }
  ```
- **`form`**: `context = { version_key, contact?, context_map_keys? }`. `version_key` identifica a versão do formulário (módulo ClickForm) a ser exibido.
  ```json
  { "type": "form", "context": { "version_key": "sua-form-version-key" } }
  ```
  - `context_map_keys` faz "forward-fill" — injeta valores de campos de um form anterior (ou do contato recebido na abertura da execução) em campos deste form. É um mapa `campo_destino → { read_only, value }`, onde `value` é a chave do campo de origem (ou um placeholder de contato, como `{{person_name}}`, `{{person_documentation}}`, `{{person_birthday}}`, `{{phone_number}}`):
    ```json
    { "type": "form", "context": {
      "version_key": "versao-formulario-confirmacao",
      "context_map_keys": {
        "nome-completo": { "read_only": true, "value": "{{person_name}}" },
        "cpf": { "read_only": true, "value": "{{person_documentation}}" }
      }
    } }
    ```
- **`verify`**: `context = { authentication, contact_fields_map? }`, com `authentication: "liveness"` ou `"biometric_behavior"`. Quando combinado com um `form` anterior, `contact_fields_map` casa o campo de identidade com a chave do campo do form onde ele foi coletado — mapa `campo_identidade → chave_do_campo_no_form`:
  ```json
  { "type": "verify", "context": {
    "authentication": "biometric_behavior",
    "contact_fields_map": { "person_name": "field-key-do-nome", "person_documentation": "field-key-do-documento" }
  } }
  ```
- **`kyc`** *(novo, 13/07/2026)*: `context = { type }`, com `type: "business"` (CNPJ) ou `"customer"` (CPF). Tipicamente encadeado logo após um `verify`, usando os dados já coletados (do `contact` da execução ou de um `form`/`verify` anterior) para rodar a checagem de conhecimento de cliente.
  ```json
  { "type": "kyc", "context": { "type": "customer" } }
  ```
- **`signature`**: `context = { folder_key, documents[] }`, validado no **Runner**, não no Sequencer. Cada item de `documents[]` é um arquivo já existente no S3 (`kind: "file"`, com `s3_bucket`, `s3_key`, `filename`) ou gerado a partir de um modelo Távola (`kind: "template"`, com `template_key`, `filename`) — os dois tipos podem ser combinados na mesma lista. `filename` aceita interpolação de placeholders (ex: `doc_{{person_name}}.docx`).
  ```json
  { "type": "signature", "context": {
    "folder_key": "1fb29ab8-84c4-4856-be37-5fce0f17c11a",
    "documents": [ { "kind": "template", "template_key": "uuid-do-modelo-tavola", "filename": "contrato-proposta.docx" } ]
  } }
  ```

Mais exemplos (aceite, form encadeado, verify com form, assinatura mista) estão nos `examples` de `POST /flows` em [`../collections/openapi/clickflow-sequencer-v1.openapi.json`](../collections/openapi/clickflow-sequencer-v1.openapi.json).

## Execution

Uma **Execution** é uma transação em andamento (ou concluída) de um Flow publicado — uma instância real da jornada, para uma pessoa específica.

O Sequencer expõe duas formas de representar uma execução:

- **Formato resumido** (`domain.Execution`) — devolvido pela listagem `GET /flows/{id}/executions`:

  | Campo | Descrição |
  |---|---|
  | `execution_id` | Identificador único da execução |
  | `flow_id` / `flow_name` | Referência ao Flow que originou a execução |
  | `account` | Snapshot da conta no momento em que a execução foi criada |
  | `status` | Ver seção de estados, abaixo |
  | `current_step_index` | Índice do step atual dentro de `steps[]` |
  | `created_at` / `updated_at` | Timestamps |

- **Formato detalhado** (`handler.ExecutionDetailResponse`) — devolvido por `POST /flows/{id}/executions`, `GET /executions/{execution_id}` e `POST /executions/{execution_id}/next`. Além dos campos acima, traz:

  | Campo | Descrição |
  |---|---|
  | `current_step` | `{ type, context }` do step em andamento agora |
  | `executed_steps[]` | Histórico: `{ type, status, step_index, started_at, finished_at }` de cada step já executado |

- **Formato do Runner** (`http.executionResponse`) — devolvido por `POST /flows/{flow_id}/execute` e `GET /executions/{execution_id}` no **Runner** *(schema unificado em 13/07/2026 — antes, o `POST` devolvia um formato reduzido próprio)*:

  | Campo | Descrição |
  |---|---|
  | `execution_id` / `flow_id` | Referências ao Flow e à execução |
  | `status` | Ver seção de estados, abaixo — **atenção**: o enum do Runner só tem `running`, `completed`, `failed` (sem `waiting`) |
  | `channel` | `whatsapp` \| `api` — canal definido no `POST /execute`, ver [`04-canais.md`](04-canais.md) |
  | `current_step` | `{ id, type, url }` do passo em andamento. Presente só quando `channel = "api"` e há um step `RUNNING`. `url` é `null` para `acceptance` |
  | `contact` | Contato informado (ou atualizado) na execução |
  | `presentation_runner_error` | Objeto livre — presente quando houve falha ao apresentar o step atual ao contato |
  | `whatsapp_presentation_outbound` / `whatsapp_conclusion_outbound` | Objetos livres com metadados do envio da mensagem de apresentação/conclusão via WhatsApp (quando `channel = "whatsapp"`) |
  | `created_at` / `updated_at` | Timestamps |

### Estados de uma Execution

```
running ──► waiting ──► running ──► completed
   │                                    ▲
   └──────────────► failed ─────────────┘
```

- `running`: a execução está avançando (ex: acabou de ser criada, ou acabou de avançar de step).
- `waiting`: a execução está aguardando uma ação externa (ex: resposta do usuário no WhatsApp).
- `completed`: todos os steps foram concluídos com sucesso.
- `failed`: a execução foi encerrada sem sucesso.

> **⚠️ Divergência de casing observada.** O spec publicado pelo próprio serviço (`/api/v1/docs/doc.json`) declara os quatro valores em minúsculo (`running`, `waiting`, `completed`, `failed`). Um payload real de produção capturado em 07/07/2026, porém, trouxe o valor em maiúsculo (`"status": "RUNNING"`). Não assuma um casing fixo — trate a comparação de `status` como *case-insensitive* no seu código até essa divergência ser esclarecida com o time técnico do ClickFlow.

> **⚠️ Divergência entre serviços.** O enum acima (`domain.ExecutionStatus`, 4 valores) é o do **Sequencer**. O **Runner** expõe só 3 valores no seu próprio `status` (`running`, `completed`, `failed` — sem `waiting`). Se você só integra com o Runner (fluxo recomendado, ver [`05-guia-de-integracao.md`](05-guia-de-integracao.md)), não espere ver `waiting` na resposta dele; para o estado `waiting`, consulte o Sequencer diretamente (`GET /executions/{execution_id}`).

## Como as três entidades se relacionam

```
Flow (draft) ──publish──► Flow (published)
                                │
                                ├── POST /executions ──► Execution #1 (running → ... → completed)
                                ├── POST /executions ──► Execution #2
                                └── POST /executions ──► Execution #N
```

Um mesmo Flow publicado pode originar múltiplas execuções independentes — uma por consumidor final.

## Próximo passo

Para autenticar suas chamadas contra o Sequencer e o Runner, siga para [`03-autenticacao.md`](03-autenticacao.md).
