# ClickFlow API — Documentação de Integração

O **ClickFlow** é o orquestrador de jornadas de formalização digital da Clicksign. Ele combina módulos de captura de dados, autenticação e assinatura eletrônica em uma única esteira, configurável via API, e entrega a experiência final ao consumidor pelo WhatsApp.

Este repositório é o ponto de entrada para qualquer empresa que precise **entender como o ClickFlow funciona e como plugar sua operação nele**: conceitos, contrato de API e coleções prontas para importar no Postman/Insomnia.

> Este repositório documenta exclusivamente o **ClickFlow** (orquestração). Para o módulo de captura de formulários (**ClickForm**), consulte a documentação irmã em [`docs-form-api`](https://github.com/juliomonq-clk/docs-form-api).

---

## Por onde começar

| Se você quer... | Vá para |
|---|---|
| Saber os hosts de Sandbox e Produção | [`docs/00-ambientes.md`](docs/00-ambientes.md) |
| Entender o que o ClickFlow é e resolve | [`docs/01-visao-geral.md`](docs/01-visao-geral.md) |
| Conhecer o modelo de dados (Flow, Step, Execution) | [`docs/02-conceitos-e-modelo-de-dados.md`](docs/02-conceitos-e-modelo-de-dados.md) |
| Autenticar suas chamadas | [`docs/03-autenticacao.md`](docs/03-autenticacao.md) |
| Saber quais canais de entrega existem hoje | [`docs/04-canais.md`](docs/04-canais.md) |
| Seguir um passo a passo de ponta a ponta | [`docs/05-guia-de-integracao.md`](docs/05-guia-de-integracao.md) |
| Interpretar erros e estados de execução | [`docs/06-referencia-de-erros-e-status.md`](docs/06-referencia-de-erros-e-status.md) |
| Ver perguntas comuns | [`docs/07-perguntas-frequentes.md`](docs/07-perguntas-frequentes.md) |
| Testar a API sem escrever código | [`collections/`](collections/README.md) (OpenAPI + Postman + Insomnia) |

---

## Arquitetura em 30 segundos

O ClickFlow é dividido em dois serviços com responsabilidades estritamente separadas:

- **Sequencer (Orquestrador)** — o "cérebro". Guarda a definição do fluxo (a "receita", em JSON) e o estado de cada transação. Decide qual é o próximo passo, mas não executa nada e não conhece o canal de entrega.
- **Runner (Executor)** — o "músculo". Recebe a instrução do Sequencer, aciona o módulo correto (formulário, autenticação, assinatura) e entrega a experiência ao consumidor final.

```
Empresa integradora → Runner (inicia execução) → Sequencer (decide o próximo passo)
                            ↓
                    Módulo correto é acionado
                            ↓
              Consumidor final recebe a experiência
```

Detalhes de cada peça, do modelo de dados e do contrato completo estão em [`docs/`](docs/) e [`collections/`](collections/).

## Como hoje se configura um fluxo

Na versão atual, a criação e a configuração de um Flow são feitas **via API** (não há builder visual). O fluxo típico de acesso para uma empresa nova é:

1. Contato comercial/técnico com a Clicksign para provisionar credenciais e o Flow inicial.
2. Definição da "receita" do Flow (sequência de steps) em conjunto com o time de Professional Services da Clicksign.
3. Integração da empresa contratante com o Runner para iniciar execuções e acompanhar o progresso.

Dúvidas de acesso/credenciais: `professionalservices@clicksign.com`.

## Status deste documento

Este repositório documenta apenas o que já está confirmado em contrato de API ou validado com o time técnico do ClickFlow. Funcionalidades em desenvolvimento (ex: novos canais de entrega) são publicadas aqui somente depois de estáveis.
