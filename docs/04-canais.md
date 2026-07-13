# Canais de Entrega

## `whatsapp` (padrão)

Na versão atual do ClickFlow, o canal padrão de entrega é **WhatsApp**. Cada step do Flow (formulário, autenticação, KYC, assinatura) é apresentado ao usuário dentro da conversa, com formulários e telas de autenticação abrindo em webview nativa para reduzir fricção. `contact.phone_number` é obrigatório.

Omitir o campo `channel` no `POST /flows/{flow_id}/execute` equivale a `"whatsapp"` — compatível com integrações feitas antes de 13/07/2026, quando este parâmetro ainda não existia.

## `api` (headless) — confirmado em 13/07/2026

O parâmetro `channel` no `POST /flows/{flow_id}/execute` (Runner) agora faz parte do contrato publicado, com dois valores:

- `whatsapp` — comportamento padrão, descrito acima.
- `api` — modo headless: o Runner **não envia mensagens automáticas via WhatsApp**. A resposta traz o objeto `current_step` (`{ id, type, url }`) com o passo em andamento, para a própria empresa integradora (ou a Clicksign, construindo uma casca própria) renderizar a experiência. `contact.phone_number` é opcional nesse modo — **exceto** para steps `acceptance`, que continuam exigindo interação via WhatsApp e falham sem `phone_number`.

Qualquer valor de `channel` diferente de `whatsapp`/`api` retorna `400 Bad Request`.

### Request

```http
POST {{runner_base_url}}/flows/{flow_id}/execute
Authorization: <token>
Content-Type: application/json

{
  "channel": "api",
  "contact": {
    "person_name": "Maria Silva",
    "person_documentation": "00000000000",
    "person_birthday": "1990-01-01",
    "phone_number": "+5511999999999"
  }
}
```

### Response

```json
{
  "execution_id": "<uuid>",
  "flow_id": "<uuid>",
  "status": "running",
  "channel": "api",
  "current_step": {
    "id": "3",
    "type": "signature",
    "url": "https://app.clicksign.com/notarial/widget/signatures/uuid/redirect"
  },
  "contact": { "person_name": "Maria Silva", "person_documentation": "00000000000", "person_birthday": "1990-01-01", "phone_number": "+5511999999999" }
}
```

`current_step.url` é `null` para steps `acceptance` (não há uma URL a renderizar — a interação é sempre via WhatsApp). Para acompanhar a mudança de step a step, consulte `GET /executions/{execution_id}` repetidamente ou `GET /executions/{execution_id}/steps` para o histórico completo.

> **⚠️ Enum incompleto observado.** O spec do Runner declara `current_step.type` com apenas `form`, `verify`, `signature`, `acceptance` — sem `kyc` (step novo, ver [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md)). Não está confirmado com o time técnico se um step `kyc` em andamento produz um `current_step` no canal `api`, ou se ele é resolvido automaticamente sem etapa visível ao contato. Valide isso antes de depender do canal `api` num Flow que use `kyc`.

Não existe hoje um canal "web" nativo operado pela Clicksign para o consumidor final interagir via navegador; qualquer experiência web é, ou será, uma camada construída sobre o canal `api`. Veja o contrato completo em [`../collections/openapi/clickflow-runner-v1.openapi.json`](../collections/openapi/clickflow-runner-v1.openapi.json).

## Próximo passo

Para um passo a passo completo de integração — criar um Flow, publicá-lo, iniciar e acompanhar uma execução — siga para [`05-guia-de-integracao.md`](05-guia-de-integracao.md).
