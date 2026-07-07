# ClickFlow API — Coleções

Coleções para testar e integrar com a API do ClickFlow (Sequencer + Runner) sem escrever código. Disponíveis em **Postman**, **Insomnia** e **Bruno**, além do spec **OpenAPI** de cada serviço.

## Conteúdo

```
collections/
├── openapi/    clickflow-sequencer-v1.openapi.json, clickflow-runner-v1.openapi.json
├── postman/    coleção + ambientes Sandbox/Produção
├── insomnia/   export v4 (workspace + sub-ambientes Sandbox/Produção)
└── bruno/      coleção em pastas .bru (abrir a pasta direto no Bruno)
```

## Autenticação

Todas as chamadas usam o header `Authorization: <token>` — **sem** prefixo `Bearer`. Veja [`../docs/03-autenticacao.md`](../docs/03-autenticacao.md) para como obter um token.

## Ambientes e variáveis usadas nas coleções

| Variável | Sandbox (recomendado para testes) | Produção |
|---|---|---|
| `sequencer_base_url` | `https://clickflow-sandbox.clicksign.com/api/v1` | `https://clickflow.clicksign.com/api/v1` |
| `runner_base_url` | `https://clickflow-runner-sandbox.clicksign.com/api/v1` ⚠️ observado indisponível em 07/07/2026, confirmar com o time técnico | `https://clickflow-runner.clicksign.com/api/v1` |

| Variável | Descrição |
|---|---|
| `access_token` | Seu token de acesso (vazio, preencha) |
| `flow_id`, `execution_id` | IDs usados nos paths (vazio, preencha conforme for testando o fluxo) |

Veja [`../docs/00-ambientes.md`](../docs/00-ambientes.md) para o detalhe de status de cada host.

## Como importar

### Postman
1. Import → arraste `postman/ClickFlow-API.postman_collection.json` e os dois arquivos `postman/ClickFlow-Sandbox.postman_environment.json` / `postman/ClickFlow-Producao.postman_environment.json`.
2. Selecione o ambiente "ClickFlow - Sandbox" no canto superior direito.
3. Preencha `access_token`.

### Insomnia
1. Application → Preferences → Import Data → From File → selecione `insomnia/ClickFlow-API.insomnia.json`.
2. Isso cria o workspace, o "Base Environment" e os sub-ambientes Sandbox/Produção. Edite o Base Environment para preencher `access_token`.

### Bruno
1. Open Collection → selecione a pasta `bruno/` inteira (ela já tem o `bruno.json` na raiz).
2. Escolha o ambiente "Sandbox" ou "Producao" no seletor de ambiente do Bruno.
3. Preencha `access_token` no ambiente.

## Fluxo sugerido de teste

Siga a ordem descrita em [`../docs/05-guia-de-integracao.md`](../docs/05-guia-de-integracao.md): criar flow → publicar → iniciar execução (Runner) → consultar steps → (opcional) avançar manualmente.

## Observação sobre o spec OpenAPI

Os dois arquivos em `openapi/` foram extraídos do endpoint de documentação viva de cada serviço (`/api/v1/docs/doc.json`, gerado via swaggo a partir do código Go) em 07/07/2026, e adaptados para um formato único (OpenAPI 3.0.3) — o do Runner era originalmente Swagger 2.0. `servers` foi adicionado por este repositório; o restante (paths, schemas, exemplos) reflete o contrato tal como publicado pelo serviço. Há uma divergência conhecida de *casing* no enum de status de execução — ver nota em `domain.ExecutionStatus` no spec do Sequencer e em [`../docs/02-conceitos-e-modelo-de-dados.md`](../docs/02-conceitos-e-modelo-de-dados.md).
