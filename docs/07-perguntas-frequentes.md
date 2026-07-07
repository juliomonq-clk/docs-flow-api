# Perguntas Frequentes

**O ClickFlow tem uma interface visual para montar o fluxo (builder)?**
Não, hoje não. A definição de um Flow é feita via API (JSON), com apoio do time de Professional Services da Clicksign no onboarding.

**Consigo editar um Flow depois de publicado?**
Não. Depois de `publish`, a definição fica imutável. Isso garante que execuções já em andamento não sejam afetadas por uma mudança posterior na receita. Para mudar o fluxo, crie um novo Flow.

**Preciso integrar direto com o Sequencer?**
Não necessariamente. O início e o acompanhamento do dia a dia de uma execução acontecem pelo **Runner**. O Sequencer é consultado quando você precisa do estado consolidado da execução ou quer forçar o avanço manual de um step.

**Tem ambiente de testes (sandbox)?**
Sim — veja os hosts em [`00-ambientes.md`](00-ambientes.md). Use sempre o Sandbox para testar antes de ir para produção. O Sandbox do Runner foi observado indisponível em 07/07/2026; confirme o status atual com o time técnico antes de depender dele para um teste.

**Existe um canal além do WhatsApp?**
Hoje não. Um canal `api` (headless, para a empresa integradora construir sua própria interface) está confirmado como direção de arquitetura, mas ainda não faz parte do contrato publicado. Veja [`04-canais.md`](04-canais.md).

**Como sei em qual etapa uma transação específica está?**
Consulte `GET /executions/{execution_id}/steps` no Runner. Cada item retorna o tipo, o status e os timestamps do step.

**Um módulo (ex: assinatura) pode ser usado fora da esteira completa?**
Sim, essa é uma decisão de arquitetura: os módulos são desacoplados e autônomos. Fale com a Clicksign sobre o modelo de contratação para o seu caso.

**Onde consigo credenciais para testar?**
Com o time de Professional Services da Clicksign: `professionalservices@clicksign.com`.

**Onde estão as coleções para testar sem escrever código?**
Em [`../collections/`](../collections/README.md) — OpenAPI, Postman, Insomnia e Bruno.
