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
| `RUNNING` | A execução está avançando. |
| `WAITING` | Aguardando ação externa (ex: resposta do usuário no WhatsApp). |
| `COMPLETED` | Todos os steps concluídos com sucesso. |
| `FAILED` | Execução encerrada sem sucesso. |

Os valores são sempre em maiúsculo.

## Erros conhecidos do contrato

| Código | Onde ocorre | Causa |
|---|---|---|
| `403 Forbidden` | `PUT /flows/{id}` | Tentativa de editar um Flow que não está em `draft` (ex: já `published`) |

## Separação entre erro de negócio e erro técnico

Ao investigar uma falha, é importante distinguir:

- **Erro de negócio** (ex: usuário reprovado em uma checagem de liveness, resposta de aceite fora do esperado): faz parte do fluxo normal e é registrado como parte do histórico de steps da execução (`GET /executions/{execution_id}/steps`), não como uma falha de infraestrutura.
- **Erro técnico** (timeout, erro 5xx, falha de conectividade com um módulo): é uma falha de infraestrutura, não uma regra de negócio.

Se uma execução parecer travada sem motivo aparente, confira primeiro o histórico de steps antes de reportar como incidente — muitas vezes a execução está em `WAITING`, aguardando uma ação do consumidor final.

## Próximo passo

Veja perguntas comuns sobre integração em [`07-perguntas-frequentes.md`](07-perguntas-frequentes.md).
