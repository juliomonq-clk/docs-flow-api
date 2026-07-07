# Guia de Integração

Este guia percorre o ciclo completo de uma jornada: criar um Flow, publicá-lo, iniciar uma execução e acompanhar o progresso até a conclusão.

> Os exemplos usam `{{sequencer_base_url}}` e `{{runner_base_url}}` como placeholders. Use os hosts de **Sandbox** para testar (ver [`00-ambientes.md`](00-ambientes.md)):
> `sequencer_base_url = https://clickflow-sandbox.clicksign.com/api/v1`
> `runner_base_url = https://clickflow-runner-sandbox.clicksign.com/api/v1`

## 1. Criar um Flow (Sequencer)

```http
POST {{sequencer_base_url}}/flows
Authorization: <token>
Content-Type: application/json

{
  "name": "Onboarding de colaborador",
  "steps": [
    { "type": "form", "context": { "version_key": "<version_key do formulário ClickForm>" } },
    { "type": "verify", "context": { "authentication": "liveness" } },
    { "type": "signature", "context": { "folder_key": "<referência da pasta>", "documents": [ { "kind": "template", "template_key": "<uuid do modelo Távola>", "filename": "contrato.docx" } ] } }
  ]
}
```

O Flow é criado com `status: "draft"`. Nesse estado, ele pode ser editado livremente via `PUT /flows/{id}`.

## 2. Publicar o Flow (Sequencer)

```http
PATCH {{sequencer_base_url}}/flows/{flow_id}/publish
Authorization: <token>
```

A partir daqui, o Flow passa a `status: "published"` e sua definição fica imutável — uma tentativa de `PUT` retorna `403 Forbidden`.

## 3. Iniciar uma execução (Runner)

A execução é sempre iniciada pelo **Runner**, nunca diretamente pelo Sequencer. O Runner cria a execução no Sequencer internamente e mantém seu próprio registro.

```http
POST {{runner_base_url}}/flows/{flow_id}/execute
Authorization: <token>
Content-Type: application/json

{
  "contact": {
    "person_name": "Maria Silva",
    "person_documentation": "00000000000",
    "person_birthday": "1990-01-01",
    "phone_number": "+5511999999999"
  }
}
```

Resposta:

```json
{
  "execution_id": "<uuid>",
  "flow_id": "<uuid>",
  "status": "running"
}
```

> O casing exato de `status` tem uma divergência conhecida entre o spec e payloads reais — ver [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md#estados-de-uma-execution).

A partir deste ponto, o consumidor final (`Maria Silva`) recebe a primeira etapa da jornada pelo WhatsApp, no número informado.

## 4. Acompanhar o progresso

Para saber em qual step a execução está, consulte o histórico de steps no Runner:

```http
GET {{runner_base_url}}/executions/{execution_id}/steps
Authorization: <token>
```

Cada item retornado traz `step_id`, `attribute_type`, `context`, `status`, `external_id`, `started_at` e `updated_at` — o suficiente para montar uma visão de funil (em qual etapa cada transação está) sem precisar de um dashboard analítico.

Para o estado consolidado da execução, incluindo o step atual e o histórico já executado, consulte o Sequencer:

```http
GET {{sequencer_base_url}}/executions/{execution_id}
Authorization: <token>
```

A resposta traz `current_step` (`{ type, context }`) e `executed_steps[]` (`{ type, status, step_index, started_at, finished_at }`) — ver [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

## 5. Avançar manualmente (quando aplicável)

Na maior parte dos casos, o avanço entre steps acontece automaticamente conforme o consumidor final interage no WhatsApp. Quando for necessário forçar o avanço para o próximo passo pela API:

```http
POST {{sequencer_base_url}}/executions/{execution_id}/next
Authorization: <token>
```

## 6. Encerramento

A execução chega a `completed` quando todos os steps forem concluídos com sucesso, ou a `failed` caso algum step seja encerrado sem sucesso. Veja o detalhamento de estados em [`06-referencia-de-erros-e-status.md`](06-referencia-de-erros-e-status.md).

## Testando sem escrever código

As coleções Postman, Insomnia e Bruno com esses mesmos requests já configurados (mais o OpenAPI completo) estão em [`../collections/`](../collections/README.md).
