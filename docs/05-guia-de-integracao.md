# Guia de Integração

Este guia percorre o ciclo completo de uma jornada: criar um Flow, publicá-lo, iniciar uma execução e acompanhar o progresso até a conclusão.

> Os exemplos usam `{{sequencer_base_url}}` e `{{runner_base_url}}` como placeholders — substitua pelos hosts fornecidos no seu onboarding. O host confirmado do Runner é `clickflow-runner.clicksign.com`; o host do Sequencer é fornecido por conta durante o provisionamento.

## 1. Criar um Flow (Sequencer)

```http
POST {{sequencer_base_url}}/api/v1/flows
Authorization: <token>
Content-Type: application/json

{
  "name": "Onboarding de colaborador",
  "steps": [
    { "type": "form", "context": { "form_id": "<id do formulário ClickForm>" } },
    { "type": "verify", "context": { "authentication": "liveness" } },
    { "type": "signature", "context": { "folder_key": "<referência do documento/pasta>" } }
  ]
}
```

O Flow é criado com `status: "draft"`. Nesse estado, ele pode ser editado livremente via `PUT /flows/{id}`.

## 2. Publicar o Flow (Sequencer)

```http
PATCH {{sequencer_base_url}}/api/v1/flows/{flow_id}/publish
Authorization: <token>
```

A partir daqui, o Flow passa a `status: "published"` e sua definição fica imutável — uma tentativa de `PUT` retorna `403 Forbidden`.

## 3. Iniciar uma execução (Runner)

A execução é sempre iniciada pelo **Runner**, nunca diretamente pelo Sequencer. O Runner cria a execução no Sequencer internamente e mantém seu próprio registro.

```http
POST {{runner_base_url}}/api/v1/flows/{flow_id}/execute
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
  "status": "RUNNING"
}
```

A partir deste ponto, o consumidor final (`Maria Silva`) recebe a primeira etapa da jornada pelo WhatsApp, no número informado.

## 4. Acompanhar o progresso

Para saber em qual step a execução está, consulte o histórico de steps no Runner:

```http
GET {{runner_base_url}}/api/v1/executions/{execution_id}/steps
Authorization: <token>
```

Cada item retornado traz `attribute_type`, `context`, `status`, `external_id`, `started_at` e `updated_at` — o suficiente para montar uma visão de funil (em qual etapa cada transação está) sem precisar de um dashboard analítico.

Para o estado consolidado da execução (status geral, step atual), consulte o Sequencer:

```http
GET {{sequencer_base_url}}/api/v1/executions/{execution_id}
Authorization: <token>
```

## 5. Avançar manualmente (quando aplicável)

Na maior parte dos casos, o avanço entre steps acontece automaticamente conforme o consumidor final interage no WhatsApp. Quando for necessário forçar o avanço para o próximo passo pela API:

```http
POST {{sequencer_base_url}}/api/v1/executions/{execution_id}/next
Authorization: <token>
```

## 6. Encerramento

A execução chega a `COMPLETED` quando todos os steps forem concluídos com sucesso, ou a `FAILED` caso algum step seja encerrado sem sucesso. Veja o detalhamento de estados em [`06-referencia-de-erros-e-status.md`](06-referencia-de-erros-e-status.md).

## Testando sem escrever código

As coleções Postman, Insomnia e Bruno com esses mesmos requests já configurados (mais o OpenAPI completo) estão em [`../collections/`](../collections/README.md).
