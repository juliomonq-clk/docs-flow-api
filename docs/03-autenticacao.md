# Autenticação

Todas as chamadas ao Sequencer e ao Runner (exceto `GET /health`) usam o mesmo esquema de autenticação:

```
Authorization: <token>
```

- O valor do header **não** leva prefixo `Bearer` — é o token puro.
- O token é validado por introspecção em um serviço central de autenticação da Clicksign, que retorna:
  - `user_id`
  - `account` (`id` numérico, `key` em UUID, `name`)
- O Runner repassa o mesmo header `Authorization` recebido do cliente para as chamadas internas que faz ao Sequencer — ou seja, um único token é válido para os dois serviços.

## Como obter um token

O provisionamento de credenciais faz parte do processo de onboarding de uma empresa no ClickFlow, conduzido junto com o time de Professional Services da Clicksign (`professionalservices@clicksign.com`). Não há hoje um fluxo de self-service para geração de token.

## Endpoints sem autenticação

- `GET /health` (Sequencer e Runner) — health check simples, sem header de autenticação.

## Próximo passo

Para saber por qual canal a experiência do consumidor final é entregue, siga para [`04-canais.md`](04-canais.md).
