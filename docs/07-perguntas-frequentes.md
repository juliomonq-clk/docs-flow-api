# Perguntas Frequentes

**O ClickFlow tem uma interface visual para montar o fluxo (builder)?**
Não, hoje não. A definição de um Flow é feita via API (JSON), com apoio do time de Professional Services da Clicksign no onboarding.

**Consigo editar um Flow depois de publicado?**
Não diretamente — uma tentativa de `PUT` num Flow `published` retorna `403`. Mas desde 13/07/2026 dá para despublicar (`PATCH /flows/{id}/unpublish`), editar e publicar de novo, sem precisar criar um Flow novo. Isso garante que execuções já em andamento não sejam afetadas por uma mudança no meio do caminho. Veja [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**Preciso integrar direto com o Sequencer?**
Não necessariamente. O início e o acompanhamento do dia a dia de uma execução acontecem pelo **Runner**. O Sequencer é consultado quando você precisa do estado consolidado da execução ou quer forçar o avanço manual de um step.

**Tem ambiente de testes (sandbox)?**
Sim — veja os hosts em [`00-ambientes.md`](00-ambientes.md). Use sempre o Sandbox para testar antes de ir para produção. O Sandbox do Runner foi observado indisponível em 07/07/2026; confirme o status atual com o time técnico antes de depender dele para um teste.

**Existe um canal além do WhatsApp?**
Sim, desde 13/07/2026. O canal `api` (headless) já faz parte do contrato publicado: informe `"channel": "api"` no `POST /flows/{flow_id}/execute` e a resposta traz `current_step.url` para você entregar ao contato, sem o Runner enviar mensagens automáticas pelo WhatsApp. Veja [`04-canais.md`](04-canais.md).

**Como sei em qual etapa uma transação específica está?**
Consulte `GET /executions/{execution_id}/steps` no Runner. Cada item retorna o tipo, o status e os timestamps do step.

**O que é o step `kyc`?**
Checagem de conhecimento de cliente, adicionada ao contrato em 13/07/2026. `context = { type: "business" | "customer" }` — CNPJ ou CPF, respectivamente. Normalmente vem depois de um `verify`. Veja [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**Como remover um Flow que não uso mais?**
`DELETE /flows/{id}` no Sequencer — é um soft delete (`status: "deleted"`), não afeta execuções já criadas a partir dele.

**Um módulo (ex: assinatura) pode ser usado fora da esteira completa?**
Sim, essa é uma decisão de arquitetura: os módulos são desacoplados e autônomos. Fale com a Clicksign sobre o modelo de contratação para o seu caso.

**Onde consigo credenciais para testar?**
Com o time de Professional Services da Clicksign: `professionalservices@clicksign.com`.

**Onde estão as coleções para testar sem escrever código?**
Em [`../collections/`](../collections/README.md) — OpenAPI, Postman, Insomnia e Bruno.

**Dá para colocar mais de um documento no mesmo envelope de assinatura?**
Sim. O `context` do step `signature` tem um campo `documents` (array) — cada item é um documento, com `kind: "file"` (referência S3, via `s3_bucket`+`s3_key`) ou `kind: "template"` (modelo do motor de assinatura legado, via `template_key`). Um mesmo `documents[]` pode misturar os dois `kind`, e todos compartilham o mesmo `folder_key` (campo irmão de `documents`, não aninhado em cada item). Veja os exemplos `signature_file`/`signature_template`/`signature_mixed` em [`clickflow-sequencer-v1.openapi.json`](../collections/openapi/clickflow-sequencer-v1.openapi.json) e detalhe em [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**Consigo ter etapas da mesma esteira preenchidas por pessoas diferentes, cada uma no seu WhatsApp?**
Sim, sem precisar criar execuções separadas. O `context` de um step `form` aceita um campo `contact` (`person_name`, `person_documentation`, `person_birthday`, `phone_number`) que sobrescreve, a partir daquele step, o contato definido na abertura da execução — o Runner passa a entregar aquele formulário para o novo `phone_number`. No spec, esse exemplo é rotulado `form_contact` / "Form — atualização de contato". **Atenção:** não confundir com o exemplo `form_forward_fill_contact` / "Injeção do contato no formulário" — nome parecido, mas esse outro só pré-preenche campos do formulário com o contato já existente (via `context_map_keys` e placeholders `{{person_name}}` etc.), sem trocar quem recebe a mensagem. Veja [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).
