# Processo de produção dos design docs

## Sobre o desafio

O trabalho transforma a reunião técnica e o código existente do Order Management System (OMS) em um pacote de design docs para o Sistema de Webhooks de Notificação de Pedidos. A IA foi usada como apoio à produção e revisão, enquanto os requisitos e decisões foram tratados como rastreáveis às fontes do repositório.

Os documentos têm papéis complementares: o PRD registra problema e escopo; o RFC apresenta a proposta para revisão e mantém questões em aberto; os ADRs registram decisões arquiteturais; o FDD detalha a implementação; e o Tracker liga itens documentados às suas origens. O enunciado original foi preservado em [`DESAFIO.md`](DESAFIO.md).

## Ferramentas de IA utilizadas

- **Copilot no VS Code** — usado para gerar e revisar os prompts de cada etapa e apoiar a produção dos documentos, conforme o processo informado pelo autor.
- **Skill `prd-writer`** — utilizada na produção do PRD. É uma skill, não um modelo ou ferramenta independente.

O modelo específico utilizado não está identificado no histórico disponível.

## Workflow adotado

Os commits registram primeiro a atualização do PRD e seu prompt. Em uma alteração posterior foram atualizados o RFC e o FDD, adicionados os seis ADRs e organizados os prompts correspondentes. O Tracker foi atualizado depois. Assim, a ordem registrada não corresponde integralmente à sequência sugerida em [`DESAFIO.md`](DESAFIO.md): o PRD aparece antes dos ADRs, RFC e FDD. Como esses documentos técnicos aparecem agrupados no mesmo commit, não é possível afirmar a ordem exata em que foram redigidos entre si. O README do processo foi deixado para o fim.

O Copilot foi usado para gerar/adaptar prompts por etapa, e o `prd-writer` foi aplicado ao PRD. Os prompts delimitam fontes, escopo, dependências e validações específicas, em vez de pedir a geração indiscriminada de documentos.

## Prompts customizados

Os trechos abaixo foram transcritos de prompts preparados ou adaptados para o trabalho. São prompts reutilizáveis; a existência dos arquivos, por si só, não comprova que cada prompt individual tenha sido executado.

Origem: [`docs/prompts/prompt-adrs.md`](docs/prompts/prompt-adrs.md).

```text
Crie de **5 a 8 Architecture Decision Records (ADRs)** para o Sistema de Webhooks de Notificação de Pedidos descrito neste repositório. Salve um arquivo Markdown por decisão em `docs/adrs/`, sem modificar outros documentos ou arquivos de código.
```

Origem: [`docs/prompts/prompt-rfc.md`](docs/prompts/prompt-rfc.md).

```text
Crie o documento **RFC (Request for Comments)** para a feature **Sistema de Webhooks de Notificação de Pedidos** do Order Management System (OMS) descrito neste repositório. Salve somente o RFC em `docs/RFC.md`.
```

## Iterações e ajustes

Dois ajustes concretos confirmados durante a revisão foram feitos nas instruções dos prompts:

1. Foi explicitado que cada etapa deve verificar e utilizar uma skill somente quando ela for diretamente aplicável, distinguindo skills de PRD, RFC, ADR e especificação para evitar atribuições ou entregáveis fora de escopo. Essa orientação aparece, por exemplo, em [`docs/prompts/prompt-adrs.md`](docs/prompts/prompt-adrs.md) e [`docs/prompts/prompt-rfc.md`](docs/prompts/prompt-rfc.md).
2. O prompt do RFC passou a tratar os ADRs como documentos dependentes: deve verificá-los antes de criar referências e pedir orientação se não houver ADRs relevantes, em vez de assumir que estão disponíveis ou criar links quebrados. A regra está em [`docs/prompts/prompt-rfc.md`](docs/prompts/prompt-rfc.md).

Na revisão final dos documentos, foram corrigidos quatro problemas concretos:

1. `order.created` estava descrito como comportamento confirmado, embora a reunião não decidisse se a criação do pedido gera um evento separado. Foi removido do escopo baseline e mantido como questão em aberto.
2. A documentação tratava a política como um envio inicial mais cinco retries, mas a reunião resume a decisão como “cinco tentativas” e lista cinco atrasos. A contagem e o limite da DLQ agora ficam pendentes de esclarecimento.
3. O PRD afirmava uma meta de 95% em menos de 10 segundos sem origem na reunião. A meta foi alinhada ao limite de latência informado pelos clientes, sem inventar um percentual.
4. Vários detalhes propostos para o FDD eram descritos como decisões confirmadas, apesar de não constarem na transcrição nem no código. Foram marcados como propostas sujeitas à revisão, e as decisões em aberto foram explicitadas.

O número de ciclos principais de geração/revisão da produção inicial **não foi registrado**. Esta revisão final foi um ciclo de auditoria e correção; não é possível inferir quantos ciclos de geração ocorreram antes dela.

## Como navegar a entrega

Para entender a solução, sugere-se esta ordem:

1. [`TRANSCRICAO.md`](TRANSCRICAO.md) — fonte da reunião; [`DESAFIO.md`](DESAFIO.md) — enunciado e critérios do exercício.
2. [`docs/PRD.md`](docs/PRD.md) — problema, público, escopo, funcionalidades e critérios de aceitação.
3. [`docs/RFC.md`](docs/RFC.md) — proposta arquitetural submetida à revisão.
4. Decisões relacionadas: [`ADR-001`](docs/adrs/ADR-001-transactional-outbox-on-mysql.md), [`ADR-002`](docs/adrs/ADR-002-separate-polling-worker.md), [`ADR-003`](docs/adrs/ADR-003-bounded-retries-and-separate-dlq.md), [`ADR-004`](docs/adrs/ADR-004-endpoint-scoped-hmac-authentication.md), [`ADR-005`](docs/adrs/ADR-005-at-least-once-with-stable-event-id.md) e [`ADR-006`](docs/adrs/ADR-006-reuse-existing-project-patterns.md).
5. [`docs/FDD.md`](docs/FDD.md) — fluxos, contratos, persistência, integração, erros e critérios técnicos.
6. [`docs/TRACKER.md`](docs/TRACKER.md) — rastreabilidade dos itens documentados às fontes.

Os arquivos listados existem no repositório. O RFC está marcado como **Proposed — Submitted for Review**, mantém o autor como `TBD` e registra questões em aberto; o PRD e o FDD também identificam decisões que permanecem pendentes. Portanto, a presença dos documentos não significa que todos os pontos de revisão estejam encerrados.
