# Canais de Entrega

## Hoje: WhatsApp

Na versão atual do ClickFlow, a experiência do consumidor final é entregue **exclusivamente via WhatsApp**. Cada step do Flow (formulário, autenticação, assinatura) é apresentado ao usuário dentro da conversa, com formulários e telas de autenticação abrindo em webview nativa para reduzir fricção.

O payload de início de execução (`POST /flows/{flow_id}/execute` no Runner) não recebe hoje um parâmetro de canal — o comportamento é implicitamente WhatsApp. Veja o contrato completo em [`../collections/openapi/clickflow-runner-v1.openapi.json`](../collections/openapi/clickflow-runner-v1.openapi.json).

## Em desenvolvimento: canal `api` (headless)

Está confirmada, em nível de arquitetura, a adição futura de um parâmetro `channel` na chamada de início de execução do Runner, com dois valores previstos:

- `whatsapp` — comportamento atual (default).
- `api` — modo headless: o Runner devolve, por step, o `id`, o `type` e a `url` a renderizar, sem enviar nada via WhatsApp. Nesse modo, a própria empresa integradora (ou a Clicksign, construindo uma casca própria) é responsável por renderizar a experiência.

Este parâmetro ainda não faz parte do contrato de API publicado — ele será documentado aqui, com exemplos de request/response, assim que o contrato estiver estável. Não existe hoje um canal "web" nativo operado pela Clicksign para o consumidor final interagir via navegador; qualquer experiência web é, ou será, uma camada construída sobre o canal `api`.

## Próximo passo

Para um passo a passo completo de integração — criar um Flow, publicá-lo, iniciar e acompanhar uma execução — siga para [`05-guia-de-integracao.md`](05-guia-de-integracao.md).
