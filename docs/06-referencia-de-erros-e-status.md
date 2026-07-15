# Referência de Erros e Status

## Status de um Flow

| Status | Significado |
|---|---|
| `draft` | Editável via `PUT`. Não aceita novas execuções. |
| `published` | Imutável a `PUT` direto. Aceita novas execuções via `POST /executions`. Para editar, despublique antes (`PATCH /flows/{id}/unpublish`). |
| `unpublished` | Editável via `PUT` de novo, como `draft`. Não aceita novas execuções até ser republicado. |
| `deleted` | Soft delete, via `DELETE /flows/{id}`. Fora de uso corrente. |

> ⚠️ **Mudança de contrato (13/07/2026):** o valor `archived` foi substituído por `unpublished` + `deleted`. Detalhe em [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

## Status de uma Execution

| Status | Significado | Exposto por |
|---|---|---|
| `running` | A execução está avançando. | Sequencer e Runner |
| `waiting` | Aguardando ação externa (ex: resposta do usuário no WhatsApp). | Somente Sequencer |
| `completed` | Todos os steps concluídos com sucesso. | Sequencer e Runner |
| `failed` | Execução encerrada sem sucesso. | Sequencer e Runner |

> ⚠️ O spec do serviço declara os valores em minúsculo, mas já foi observado `"RUNNING"` em maiúsculo num payload real de produção (07/07/2026). Trate como *case-insensitive* até essa divergência ser esclarecida. O Runner, além disso, não expõe `waiting` no seu próprio `status` — só o Sequencer tem os 4 valores. Detalhe em [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

## Erros conhecidos do contrato

| Código | Onde ocorre | Causa |
|---|---|---|
| `400 Bad Request` | `POST /flows`, `PUT /flows/{id}`, `GET/POST /flows/{id}/executions`, `POST /flows/{flow_id}/execute` e `POST /webhooks/signature` (Runner) | Body ou parâmetros inválidos (inclui `channel` fora de `whatsapp`/`api`) |
| `401 Unauthorized` | Qualquer rota autenticada do Sequencer; `POST /webhooks/signature` (Runner) | Token ausente/inválido na introspecção; ou HMAC inválido no webhook |
| `403 Forbidden` | `PUT /flows/{id}` | Tentativa de editar um Flow `published` (despublique antes) |
| `404 Not Found` | `GET/PUT/DELETE /flows/{id}`, `GET/POST .../executions`, `GET /executions/{execution_id}` (Sequencer e Runner) | Recurso não encontrado |
| `409 Conflict` | `PUT /flows/{id}`, `PATCH /flows/{id}/publish`, `PATCH /flows/{id}/unpublish`, `POST /executions/{execution_id}/next` | Flow já está no status alvo, ou execução não está em um estado que permite avançar. **`PUT /flows/{id}` ganhou esse código numa geração de contrato mais recente (15/07/2026) sem detalhar a causa** — distinto do `403` (que já é conhecido: editar um Flow `published`); não confirmado com o time técnico o que dispara especificamente o `409` aqui |
| `500 Internal Server Error` | Rotas do Runner | Erro interno do serviço |
| `503 Service Unavailable` | `GET/POST /flows`, `GET /flows/{id}/executions`, `GET /health` (Sequencer) | Dependência essencial indisponível (ex: DynamoDB) |

### Webhook de assinatura (Runner)

`POST /webhooks/signature` recebe callbacks do Tavola e sempre responde `200` com `status: "processed"` (evento `sign` avançou o step) ou `status: "ignored"` (evento não é `sign`, ou `signer_key` não corresponde a nenhum step do flow) — isso evita retries desnecessários do lado do Tavola. Um `401` indica falha na validação do header `Content-Hmac`.

## Separação entre erro de negócio e erro técnico

Ao investigar uma falha, é importante distinguir:

- **Erro de negócio** (ex: usuário reprovado em uma checagem de liveness, resposta de aceite fora do esperado): faz parte do fluxo normal e é registrado como parte do histórico de steps da execução (`GET /executions/{execution_id}/steps`), não como uma falha de infraestrutura.
- **Erro técnico** (timeout, erro 5xx, falha de conectividade com um módulo): é uma falha de infraestrutura, não uma regra de negócio.

Se uma execução parecer travada sem motivo aparente, confira primeiro o histórico de steps antes de reportar como incidente — muitas vezes a execução está em `WAITING`, aguardando uma ação do consumidor final.

## Próximo passo

Veja perguntas comuns sobre integração em [`07-perguntas-frequentes.md`](07-perguntas-frequentes.md).
