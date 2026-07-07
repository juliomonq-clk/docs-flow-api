# Visão Geral

## O que é o ClickFlow

O ClickFlow é o orquestrador de jornadas de formalização digital da Clicksign. Em vez de uma empresa integrar separadamente um fornecedor de biometria, um fornecedor de coleta de dados e a Clicksign para assinatura, o ClickFlow compõe esses módulos em uma única esteira configurável, com um único ponto de integração.

## Problema que resolve

Processos de formalização (contratação, abertura de conta, aceite de termos, etc.) tipicamente exigem várias etapas independentes — coletar dados, validar identidade, coletar assinatura — cada uma com seu próprio fornecedor e sua própria interface. O resultado comum é:

- Jornada fragmentada para o consumidor final, com múltiplas trocas de contexto/aplicativo.
- Falta de visibilidade operacional sobre em qual etapa cada transação está.
- Alto custo de manutenção de integrações redundantes.

O ClickFlow resolve isso orquestrando as etapas como uma sequência única, entregue hoje inteiramente dentro do WhatsApp.

## Como o ClickFlow é dividido

| Componente | Responsabilidade | O que NÃO faz |
|---|---|---|
| **Sequencer** (Orquestrador) | Guarda a definição do fluxo, controla o estado da transação, decide o próximo passo | Não executa módulos, não conhece o canal de entrega |
| **Runner** (Executor) | Recebe a instrução do Sequencer, aciona o módulo correto, entrega a experiência ao consumidor final no canal configurado | Não decide a ordem das etapas |
| **Módulos** (Form, Verify, Assinatura, Aceite) | Executam uma capacidade de negócio específica de forma autônoma | Não dependem uns dos outros nem do canal |

Essa separação é uma decisão de arquitetura deliberada: qualquer módulo pode, em tese, ser usado de forma independente, fora da esteira completa.

## Tipos de step disponíveis hoje

Um Flow é uma sequência de *steps*. Os tipos suportados atualmente são:

- **`acceptance`** — aceite rápido de um termo/condição (ex: aceite de novos termos), sem geração de documento completo.
- **`form`** — coleta de dados estruturados (módulo ClickForm).
- **`verify`** — autenticação do usuário (liveness e/ou biometria comportamental).
- **`signature`** — coleta de assinatura eletrônica no documento gerado para a transação.

Detalhe de cada tipo (estrutura de `context`, regras de validação) está em [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).

## Próximo passo

Para entender o vocabulário exato (Flow, Step, Execution) antes de olhar o contrato de API, siga para [`02-conceitos-e-modelo-de-dados.md`](02-conceitos-e-modelo-de-dados.md).
