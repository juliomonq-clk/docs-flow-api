# Perguntas Frequentes

**O ClickFlow tem uma interface visual para montar o fluxo (builder)?**
Não, hoje não. A definição de um Flow é feita via API (JSON), com apoio do time de Professional Services da Clicksign no onboarding.

**Consigo editar um Flow depois de publicado?**
Não diretamente — uma tentativa de `PUT` num Flow `published` retorna `403`. Mas desde 13/07/2026 dá para despublicar (`PATCH /flows/{id}/unpublish`), editar e publicar de novo, sem precisar criar um Flow novo. Isso garante que execuções já em andamento não sejam afetadas por uma mudança no meio do caminho. Veja [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**Qual a diferença entre os steps `acceptance` e `consent`?**
`acceptance` é avanço de jornada: uma mensagem no WhatsApp com um botão de continuar. Serve para confirmar que a pessoa viu algo e seguir adiante — **não registra aceite de nada**. `consent` *(disponível desde 18/08/2026)* é aceite formal: apresenta o termo, oferece aceitar ou recusar, e registra o desfecho (aceito, recusado ou expirado), sendo que recusa e expiração encerram a jornada. Os dois convivem e nenhum substitui o outro. Regra prática: se o passo só precisa que a pessoa siga, use `acceptance`; se existe um termo cujo aceite ou recusa precisa ficar registrado, use `consent`. Veja [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**O step `consent` gera um PDF do termo aceito?**
Não. A evidência do aceite é o retorno estruturado do módulo (com o registro do desfecho e do momento), não um documento anexo. Se o seu fluxo também precisa de um documento assinado, isso vem do step `signature`, que é outra etapa — aceite e assinatura eletrônica são coisas distintas, com valor jurídico distinto.

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

**Quais opções de autenticação o step `verify` aceita?** *(atualizado 29/07/2026)*
Três: `liveness` (prova de vida facial), `biometric_behavior` (biometria comportamental, com alerta de fraude `identity_fraudsters_result`) e `identity_biometrics` (mesmo provedor Único, sem o alerta de fraude — retorna `risk_score` de 0 a 100 quando o resultado é `inconclusive`). Veja [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**Como remover um Flow que não uso mais?**
`DELETE /flows/{id}` no Sequencer — é um soft delete (`status: "deleted"`), não afeta execuções já criadas a partir dele.

**Um módulo (ex: assinatura) pode ser usado fora da esteira completa?**
Sim, essa é uma decisão de arquitetura: os módulos são desacoplados e autônomos. Fale com a Clicksign sobre o modelo de contratação para o seu caso.

**Onde consigo credenciais para testar?**
Com o time de Professional Services da Clicksign: `professionalservices@clicksign.com`.

**Onde estão as coleções para testar sem escrever código?**
Em [`../collections/`](../collections/README.md) — OpenAPI, Postman, Insomnia e Bruno.

**Dá para colocar mais de um documento no mesmo envelope de assinatura?**
Sim. O `context` do step `signature` tem um campo `documents` (array) — cada item é um documento, com `kind: "file"` (referência S3, via `s3_bucket`+`s3_key`), `kind: "template"` (modelo do motor de assinatura legado, via `template_key`) ou `kind: "runner_files"` (**novo, 21/08/2026** — slot preenchido no disparo, ver pergunta abaixo), e `filename` **opcional** nos dois primeiros (confirmado por Julio, 22/07/2026 — se omitido, o sistema aplica um nome padrão) e **obrigatório** em `runner_files`. Um mesmo `documents[]` pode misturar os `kind`, e todos compartilham as mesmas configurações de `settings` (ver pergunta abaixo). Veja os exemplos `signature_file`/`signature_template`/`signature_mixed`/`signature_runner_files` em [`clickflow-sequencer-v1.openapi.json`](../collections/openapi/clickflow-sequencer-v1.openapi.json) e detalhe em [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**Dá para mandar o documento em base64, sem subir nada em bucket S3 nem cadastrar modelo?** *(novo, 21/08/2026)*
Sim — mudou. Até 20/08/2026 a resposta era não: as duas origens de documento (`file` e `template`) eram referências a algo que já existia antes da jornada, e o princípio de arquitetura era que API e banco nunca trafegam base64. Esse princípio continua valendo, mas agora existe uma porta de entrada na borda: declare no step `signature` um slot `{ "kind": "runner_files", "key": "1", "filename": "contrato.pdf" }` e mande o conteúdo no disparo, em `files[]` do `POST /flows/{flow_id}/execute` — `{ "key": "1", "content_base_64": "data:application/pdf;base64,..." }`. O `key` correlaciona os dois lados, o nome do arquivo vem do Flow, o `content_base_64` é a **data-URL completa** (com o prefixo `data:<mime>;base64,`) e o Runner materializa o arquivo antes de seguir — do Sequencer para frente, só referência. Serve exatamente ao caso do documento gerado pelo seu sistema no momento do disparo. **Limite de tamanho não está publicado no contrato** — confirme com o time técnico antes de contar com essa via para documento grande.

**Dá para configurar o envelope de assinatura (pasta, prazo, lembrete, idioma, mensagem)?** *(novo, 22/07/2026)*
Sim, via `context.settings` no step `signature` — objeto opcional, irmão de `documents`/`signers`, com: `folder_key` (pasta do Távola — opcional, se omitido o documento vai para a pasta raiz da conta; **aceita também a forma antiga `context.folder_key`, no nível do `context`, por retrocompatibilidade — prefira `settings.folder_key` em fluxo novo**, confirmado 03/08/2026), `auto_close` (fecha o envelope automaticamente ao concluir todas as assinaturas), `block_after_refusal` (bloqueia o envelope após uma recusa), `deadline_at` (prazo de assinatura — **desde 21/08/2026 aceita um inteiro de dias**, ex. `30`, com o backend resolvendo a data absoluta no disparo e teto de 90 dias conferido na publicação; a data fixa em ISO 8601 continua aceita), `default_message`/`default_subject` (mensagem/assunto padrão da notificação), `locale` (idioma da experiência de assinatura, ex. `en-US`) e `remind_interval` (intervalo entre lembretes automáticos). **Atenção — mudança de estrutura:** `folder_key` costumava ser campo direto de `context` (documentado assim até 20/07/2026); agora vive dentro de `settings`. Semântica dos 7 campos novos inferida do nome, não confirmada com engenharia. Veja o exemplo `signature_envelope_settings` em [`clickflow-sequencer-v1.openapi.json`](../collections/openapi/clickflow-sequencer-v1.openapi.json) e detalhe em [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**Com que nome o envelope criado pela esteira aparece no motor de assinatura?** *(novo, 06/08/2026 — comportamento mudou em 04/08/2026)*
Automaticamente, no formato `{nome do Flow} - {identificação do contato} - {dd-mm-aaaa HHhMM}` (ex.: `Proposta Crédito - Maria Silva - 31-07-2026 14h32`). A identificação do contato é o `person_name` do `contact` da execução quando presente e o `phone_number` quando não; o horário é o da criação do envelope, no fuso `America/Sao_Paulo`. **Não é configurável** — não existe campo de nome de envelope no `context` do step `signature`. **Atenção a integrações existentes:** o formato anterior era `Esteira {FlowID} | Execução {ExecutionID}`; quem parseava aquele padrão para correlacionar envelope e execução precisa se ajustar (o `execution_id` continua disponível pelas rotas de consulta de execução, que são a forma correta de fazer essa correlação). Duas execuções da mesma esteira, para o mesmo contato, criadas dentro do mesmo minuto ainda produzem nomes idênticos — limitação conhecida e aceita.

**Tamanho:** ao importar esses valores para o nome do envelope, o nome do Flow é truncado em **45 caracteres** e o nome do contato em **55**, mantendo o nome final em no máximo 122 caracteres. **Não é um limite do campo `name` do Flow** — ele continua íntegro no payload e nas respostas da API; o corte vale só na composição do nome do envelope. Passar desses limites não gera erro, mas o excedente não aparece no envelope — vale escolher um nome de Flow curto e distintivo, já que é a única parte do nome do envelope sob seu controle.

**Dá para adicionar uma testemunha, fiador ou outro signatário com identidade fixa na etapa de assinatura, além do contato da execução?** *(novo, 16/07/2026)*
Sim. O `context` do step `signature` aceita um campo `signers` (array, opcional, irmão de `documents`/`settings`) — cada item pode ter identidade fixa (`name`/`documentation`/`birthday`/`phone_number`/`email` hardcoded) ou dinâmica (via placeholder `{{person_name}}` etc., puxando do `contact` da execução), método de autenticação próprio (`auths` — exemplo usa `handwritten`/`liveness`/`whatsapp`, universo do Távola é maior) e papel (`roles` — exemplo usa `witness`/`sign`, Távola tem ~70 valores possíveis). Um campo `group` **confirmado** controla a ordem de assinatura: signatários de `group` maior só assinam depois que todos os de `group` menor tiverem assinado. Esses campos são nativos do motor de assinatura legado (Távola); a semântica de `group`/`communicate_events`/`refusable`/`has_documentation`/`location_required_enabled` foi confirmada em [developers.clicksign.com](https://developers.clicksign.com/reference/signatario-campos-e-regras-de-negocio). **Notificar por e-mail funciona** — basta informar `email` no próprio item de `signers[]` (o exemplo do contrato já faz isso: `email: "signatario.fixo@mail.com"`). Isso é direto para um signatário fixo/extra; para o signatário dinâmico (dados vindos do `contact` da execução), **hoje não há** — confirmado com o time de engenharia da Clicksign: o `contact` não tem campo `email`, e não existe hoje um mecanismo de puxar dado de um `form` anterior para dentro de `signers[]` (é melhoria conhecida, ainda não implementada). Veja o exemplo `signature_signers_extras` em [`clickflow-sequencer-v1.openapi.json`](../collections/openapi/clickflow-sequencer-v1.openapi.json) e detalhe em [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

**Atenção — só existe um slot de identidade dinâmica por execução, confirmado com o time de engenharia da Clicksign:** os placeholders `{{person_name}}`/`{{person_documentation}}`/`{{person_birthday}}`/`{{phone_number}}` sempre resolvem pro `contact` único da execução, e a orientação da própria engenharia é interpolá-los em um único item de `signers[]`. Se o segundo signatário precisar de identidade que varia por execução (não é fixa), **hoje não há** essa opção — os demais signatários são tratados como fixos, sem mecanismo de puxar dado de um `form`. É melhoria conhecida (mapear campo de formulário → campo do signatário via placeholder), ainda não implementada.

**Consigo ter etapas da mesma esteira preenchidas por pessoas diferentes, cada uma no seu WhatsApp?**
Sim, sem precisar criar execuções separadas. O `context` de um step `form` aceita um campo `contact` (`person_name`, `person_documentation`, `person_birthday`, `phone_number`) que sobrescreve, a partir daquele step, o contato definido na abertura da execução — o Runner passa a entregar aquele formulário para o novo `phone_number`. No spec, esse exemplo é rotulado `form_contact` / "Form — atualização de contato". **Atenção:** não confundir com o exemplo `form_forward_fill_contact` / "Injeção do contato no formulário" — nome parecido, mas esse outro só pré-preenche campos do formulário com o contato já existente (via `context_map_keys` e placeholders `{{person_name}}` etc.), sem trocar quem recebe a mensagem. Veja [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).
