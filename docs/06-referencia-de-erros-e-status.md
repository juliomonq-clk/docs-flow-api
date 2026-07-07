# Referência de Erros e Status

## Status de um Flow

| Status | Significado |
|---|---|
| `draft` | Editável via `PUT`. Não aceita novas execuções. |
| `published` | Imutável. Aceita novas execuções via `POST /executions`. |
| `archived` | Fora de uso corrente. |

## Status de uma Execution

| Status | Significado |
|---|---|
| `running` | A execução está avançando. |
| `waiting` | Aguardando ação externa (ex: resposta do usuário no WhatsApp). |
| `completed` | Todos os steps concluídos com sucesso. |
| `failed` | Execução encerrada sem sucesso. |

> ⚠️ O spec do serviço declara os valores em minúsculo, mas já foi observado `"RUNNING"` em maiúsculo num payload real de produção (07/07/2026). Trate como *case-insensitive* até essa divergência ser esclarecida. Detalhe em [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

## Erros conhecidos do contrato

| Código | Onde ocorre | Causa |
|---|---|---|
| `400 Bad Request` | `POST /flows`, `PUT /flows/{id}`, `GET/POST /flows/{id}/executions`, `POST /flows/{flow_id}/execute` (Runner) | Body ou parâmetros inválidos |
| `401 Unauthorized` | Qualquer rota autenticada do Sequencer | Token ausente/inválido na introspecção |
| `403 Forbidden` | `PUT /flows/{id}` | Tentativa de editar um Flow que não está em `draft` (ex: já `published`) |
| `404 Not Found` | `GET/PUT /flows/{id}`, `GET/POST .../executions`, `GET /executions/{execution_id}` (Sequencer e Runner) | Recurso não encontrado |
| `409 Conflict` | `PATCH /flows/{id}/publish`, `POST /executions/{execution_id}/next` | Flow já publicado/arquivado, ou execução não está em um estado que permite avançar |
| `500 Internal Server Error` | Rotas do Runner | Erro interno do serviço |
| `503 Service Unavailable` | `GET/POST /flows`, `GET /flows/{id}/executions`, `GET /health` (Sequencer) | Dependência essencial indisponível (ex: DynamoDB) |

## Separação entre erro de negócio e erro técnico

Ao investigar uma falha, é importante distinguir:

- **Erro de negócio** (ex: usuário reprovado em uma checagem de liveness, resposta de aceite fora do esperado): faz parte do fluxo normal e é registrado como parte do histórico de steps da execução (`GET /executions/{execution_id}/steps`), não como uma falha de infraestrutura.
- **Erro técnico** (timeout, erro 5xx, falha de conectividade com um módulo): é uma falha de infraestrutura, não uma regra de negócio.

Se uma execução parecer travada sem motivo aparente, confira primeiro o histórico de steps antes de reportar como incidente — muitas vezes a execução está em `WAITING`, aguardando uma ação do consumidor final.

## Próximo passo

Veja perguntas comuns sobre integração em [`07-perguntas-frequentes.md`](07-perguntas-frequentes.md).
