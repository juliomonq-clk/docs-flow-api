# Ambientes

O ClickFlow tem dois ambientes, cada um com um host próprio para o Sequencer e para o Runner.

| Serviço | Sandbox | Produção |
|---|---|---|
| Sequencer (Orquestrador) | `https://clickflow-sandbox.clicksign.com/api/v1` | `https://clickflow.clicksign.com/api/v1` |
| Runner (Executor) | `https://clickflow-runner-sandbox.clicksign.com/api/v1` | `https://clickflow-runner.clicksign.com/api/v1` |

**Use Sandbox para testes e integração inicial.** É o ambiente recomendado para validar sua integração antes de ir para produção — sem impacto em dados reais.

## Status observado (07/07/2026)

| Host | Status |
|---|---|
| Sequencer Sandbox | ✅ saudável (`GET /health` → `{"status":"ok"}`) |
| Sequencer Produção | ✅ saudável |
| Runner Produção | ✅ saudável |
| Runner Sandbox | ⚠️ indisponível (`GET /health` → "no available server") |

O ambiente de Sandbox do Runner estava indisponível no momento da última verificação. Antes de basear um plano de testes nele, confirme o status atual com o time técnico do ClickFlow — pode ser uma indisponibilidade pontual ou um ambiente ainda não provisionado para uso externo.

## Próximo passo

Para o modelo de dados e o vocabulário usado nos dois serviços, siga para [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).
