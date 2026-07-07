# ClickFlow API — Coleções

Coleções para testar e integrar com a API do ClickFlow (Sequencer + Runner) sem escrever código. Disponíveis em **Postman**, **Insomnia** e **Bruno**, além do spec **OpenAPI** de cada serviço.

## Conteúdo

```
collections/
├── openapi/    clickflow-sequencer-v1.openapi.json, clickflow-runner-v1.openapi.json
├── postman/    coleção + ambiente
├── insomnia/   export v4 (workspace + ambiente)
└── bruno/      coleção em pastas .bru (abrir a pasta direto no Bruno)
```

## Autenticação

Todas as chamadas usam o header `Authorization: <token>` — **sem** prefixo `Bearer`. Veja [`../docs/03-autenticacao.md`](../docs/03-autenticacao.md) para como obter um token.

## Variáveis usadas nas coleções

| Variável | Descrição | Default |
|---|---|---|
| `sequencer_base_url` | Host do Sequencer (Orquestrador) | Não há host público único — solicite ao seu contato Clicksign durante o onboarding |
| `runner_base_url` | Host do Runner (Executor) | `https://clickflow-runner.clicksign.com/api/v1` (confirmado) |
| `access_token` | Seu token de acesso | (vazio, preencha) |
| `flow_id`, `execution_id` | IDs usados nos paths | (vazio, preencha conforme for testando o fluxo) |

> Diferente de outras APIs da Clicksign, o ClickFlow ainda não tem uma separação pública confirmada entre ambiente Sandbox e Produção — por isso as coleções trazem um único ambiente (`ClickFlow`), não `Sandbox`/`Produção`. Atualize aqui assim que essa separação for publicada.

## Como importar

### Postman
1. Import → arraste `postman/ClickFlow-API.postman_collection.json` e `postman/ClickFlow.postman_environment.json`.
2. Selecione o ambiente "ClickFlow" no canto superior direito.
3. Preencha `sequencer_base_url` e `access_token`.

### Insomnia
1. Application → Preferences → Import Data → From File → selecione `insomnia/ClickFlow-API.insomnia.json`.
2. Isso cria o workspace e o "Base Environment". Edite-o para preencher `sequencer_base_url` e `access_token`.

### Bruno
1. Open Collection → selecione a pasta `bruno/` inteira (ela já tem o `bruno.json` na raiz).
2. Escolha o ambiente "ClickFlow" no seletor de ambiente do Bruno.
3. Preencha `sequencer_base_url` e `access_token` no ambiente.

## Fluxo sugerido de teste

Siga a ordem descrita em [`../docs/05-guia-de-integracao.md`](../docs/05-guia-de-integracao.md): criar flow → publicar → iniciar execução (Runner) → consultar steps → (opcional) avançar manualmente.

## Observação sobre o spec OpenAPI

Os dois arquivos em `openapi/` foram escritos a partir do contrato documentado internamente para o Sequencer e o Runner (endpoints, domínio e regras de validação) — não são exportados diretamente de um schema Swagger/Go do time técnico. Path, verbos, status codes e enums refletem o contrato confirmado; tipos de campo (`string`, `format`) são inferidos e devem ser tratados como referência, não como fonte definitiva byte-a-byte.
