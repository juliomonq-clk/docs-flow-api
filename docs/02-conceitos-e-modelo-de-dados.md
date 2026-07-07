# Conceitos e Modelo de Dados

## Flow

Um **Flow** é a definição (a "receita") de uma jornada de formalização: uma lista ordenada de *steps* que serão executados em sequência para cada transação.

| Campo | Descrição |
|---|---|
| `flow_id` | Identificador único do Flow |
| `name` | Nome do Flow |
| `status` | `draft` \| `published` \| `archived` |
| `steps[]` | Lista ordenada de `FlowStep` |
| `account` | Conta proprietária do Flow |
| `user_id` | Usuário que criou/mantém o Flow |
| `created_at` / `updated_at` | Timestamps |

### Ciclo de vida do status

```
draft ──(publish)──► published ──► archived
```

- Um Flow nasce em `draft`.
- Só é possível **editar** (`PUT`) um Flow enquanto ele está em `draft`. Depois de publicado, a definição é imutável — isso garante que transações já iniciadas não sejam afetadas por mudanças posteriores na receita.
- Só é possível **iniciar execuções** (`POST /executions`) de um Flow com status `published`.

## FlowStep

Cada item da lista `steps[]` de um Flow tem:

| Campo | Descrição |
|---|---|
| `type` | `acceptance` \| `form` \| `verify` \| `signature` |
| `context` | Objeto de configuração específico do tipo (schema livre por tipo) |

### Regras de validação por tipo

- **`acceptance`**: a estrutura do `context` (`template_key`, `accept_replies`) é validada no **Sequencer**. O *match* da resposta do usuário é validado no **Runner**.
- **`form`**: aceita `context_map_keys` para "forward-fill" — injeta respostas de steps/forms anteriores (inclusive dados de contato recebidos na abertura da execução) em campos de um form seguinte, via placeholders como `{{person_name}}` e `{{person_documentation}}`. Relevante para jornadas com múltiplos formulários encadeados.
- **`verify`**: suporta `authentication: liveness` ou `biometric_behavior`. Quando combinado com um `form` anterior, usa `contact_fields_map` para casar campos do form com os dados de identidade a validar.
- **`signature`**: o `context` (documentos via referência S3 ou template, `folder_key`) é validado no **Runner**, não no Sequencer.

## Execution

Uma **Execution** é uma transação em andamento (ou concluída) de um Flow publicado — uma instância real da jornada, para uma pessoa específica.

| Campo | Descrição |
|---|---|
| `execution_id` | Identificador único da execução |
| `flow_id` / `flow_name` | Referência ao Flow que originou a execução |
| `account` | Snapshot da conta no momento em que a execução foi criada |
| `status` | `RUNNING` \| `WAITING` \| `COMPLETED` \| `FAILED` |
| `current_step_index` | Índice do step atual dentro de `steps[]` |
| `created_at` / `updated_at` | Timestamps |

### Estados de uma Execution

```
RUNNING ──► WAITING ──► RUNNING ──► COMPLETED
   │                                    ▲
   └──────────────► FAILED ─────────────┘
```

- `RUNNING`: a execução está avançando (ex: acabou de ser criada, ou acabou de avançar de step).
- `WAITING`: a execução está aguardando uma ação externa (ex: resposta do usuário no WhatsApp).
- `COMPLETED`: todos os steps foram concluídos com sucesso.
- `FAILED`: a execução foi encerrada sem sucesso.

> Os quatro valores são sempre em maiúsculo (`RUNNING`, não `Running` ou `In Progress`).

## Como as três entidades se relacionam

```
Flow (draft) ──publish──► Flow (published)
                                │
                                ├── POST /executions ──► Execution #1 (RUNNING → ... → COMPLETED)
                                ├── POST /executions ──► Execution #2
                                └── POST /executions ──► Execution #N
```

Um mesmo Flow publicado pode originar múltiplas execuções independentes — uma por consumidor final.

## Próximo passo

Para autenticar suas chamadas contra o Sequencer e o Runner, siga para [`03-autenticacao.md`](03-autenticacao.md).
