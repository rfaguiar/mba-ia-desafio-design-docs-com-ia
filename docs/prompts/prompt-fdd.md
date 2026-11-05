# Prompt para criar o FDD do sistema de webhooks

> Prompt reutilizável para executar no agente dentro do repositório do desafio.

## Instrução principal

Crie o **Feature Design Document (FDD)** de implementação para a feature **Sistema de Webhooks de Notificação de Pedidos** do Order Management System (OMS) descrito neste repositório. Salve somente o documento em `docs/FDD.md`.

Antes de começar, verifique as skills disponíveis e acione uma skill específica de FDD/Feature Design Document, caso exista. Siga suas instruções sem deixar de cumprir este prompt e os requisitos do `README.md`.

Nas skills atualmente disponíveis, `prd-writer` gera PRDs, enquanto `spec-writer` gera uma especificação e um plano de implementação em dois arquivos próprios. Elas não produzem o FDD exigido aqui: não as acione para esta tarefa nem gere seus entregáveis. Não acione skills de implementação; esta entrega é documental e o README proíbe alterações no código da aplicação. Se uma skill específica de FDD estiver disponível no ambiente em que este prompt for executado, use-a mantendo o escopo exclusivo de `docs/FDD.md`.

## Fontes e preparação

Leia integralmente o `README.md` e use o requisito **“FDD da feature”** e seus critérios de aceite como contrato obrigatório. Antes de redigir:

1. Leia `TRANSCRICAO.md` por inteiro. Ela é a fonte primária para decisões, requisitos, restrições, fluxos, alternativas descartadas e pontos adiados.
2. Leia `docs/PRD.md` para identificar os requisitos, escopo, IDs de funcionalidades, critérios de aceitação e dependências aprovados para a feature.
3. Leia `docs/RFC.md` e os ADRs disponíveis em `docs/adrs/` para manter consistência com a proposta e as decisões arquiteturais. Inspecione as fontes desses documentos quando necessário.
4. Leia os documentos existentes que possam afetar contratos ou integração, incluindo a estrutura de pastas e os padrões pertinentes do projeto.
5. Explore o código real para mapear arquitetura, transações, autenticação/autorização, validação, erros, logging, persistência, testes e convenções. Confirme a existência de cada caminho de arquivo antes de citá-lo.

Não trate um documento placeholder, comentário de template ou conteúdo sem evidência como especificação aprovada. Se documentos divergirem, use a transcrição para verificar o que foi efetivamente acordado, identifique a divergência e pergunte ao usuário quando ela impedir uma especificação segura e acionável. Não resolva divergências por suposição.

Toda afirmação sobre decisões, comportamentos existentes, requisitos, números, contratos ou restrições deve ser rastreável à transcrição, ao código existente ou aos documentos de produto/arquitetura confirmados. Diferencie o que foi decidido do que permanece aberto. Não invente nomes de arquivos, métodos, tabelas, endpoints existentes, regras, prazos, códigos de erro, participantes ou decisões.

Conduza uma entrevista técnica apenas para lacunas que a transcrição, o PRD, os ADRs e o código não resolvam e que precisem de decisão para tornar o FDD implementável. Faça uma pergunta de cada vez, ofereça recomendação fundamentada quando apropriado e não repita perguntas já respondidas pelas fontes. Após as respostas, resuma as decisões técnicas e os pontos ainda em aberto antes de redigir. Não registre uma recomendação como decisão sem confirmação.

## Objetivo e nível de detalhe

O FDD responde **“como construir, em detalhe?”** e deve ser acionável para que um engenheiro implemente a feature. Diferentemente do RFC, que apresenta a proposta para revisão, o FDD detalha fluxos, contratos, erros, integração e validação. Não copie o RFC nem transforme o FDD em uma repetição de PRD/ADRs.

Escreva o documento integralmente em **inglês**, seguindo os termos e padrões reais do projeto. Use títulos claros, tabelas para matrizes e contratos quando ajudarem a leitura, e exemplos consistentes entre as seções. Não acrescente informações sem fonte ou confirmação.

## Estrutura e conteúdo obrigatório

Inclua, no mínimo, as seguintes seções. Organize-as com títulos inequívocos e sem omitir requisitos do README:

1. **Context and Technical Motivation** — problema técnico, contexto do OMS e motivação para a solução.
2. **Technical Objectives** — objetivos técnicos verificáveis, derivados das fontes.
3. **Scope and Exclusions** — comportamento incluído nesta versão e exclusões explicitamente descartadas ou adiadas. Não reintroduza como requisito itens fora de escopo.
4. **Architecture and Component Overview** — componentes e responsabilidades da solução, limites entre API, persistência e worker, e relações com componentes existentes. Nomeie apenas componentes confirmados ou propostos de forma explícita.
5. **Detailed Flows** — fluxos passo a passo, com estados e resultados relevantes, cobrindo obrigatoriamente:
   - criação de evento/outbox dentro da transação do pedido, incluindo commit, rollback e filtragem por endpoint;
   - leitura e processamento pelo worker, entrega HTTP e registro do resultado;
   - timeout, falha de conexão, respostas não-2xx, agendamento de retry, backoff e limite de tentativas;
   - transferência para a DLQ e replay administrativo, incluindo autorização e auditoria quando definidas.
6. **Public Contracts** — pelo menos quatro endpoints HTTP da feature. Para cada endpoint, especifique método e caminho, propósito, autenticação/autorização, parâmetros, headers relevantes, request e response JSON de exemplo, status codes, validações e semântica. Use a semântica e os caminhos estabelecidos pelas fontes; não invente contrato já decidido de outra forma. Se o caminho ou comportamento necessário não estiver definido, pergunte antes de fechar o contrato.
7. **Data Model and Persistence** — entidades/registros necessários, campos, relações, estados, constraints e índices relevantes para o comportamento. Descreva transações e imutabilidade do payload quando exigidas. Separe com clareza o que já existe no código e o que é proposta para a feature. Não escreva migrations nem código.
8. **Error Matrix** — matriz com cenário, código, condição, resposta/comportamento e recuperação. Códigos de erro da feature devem usar o prefixo `WEBHOOK_` e seguir o padrão de erros existente. Inclua somente cenários confirmados ou necessários para os fluxos definidos; não prometa mensagens ou status HTTP sem base.
9. **Resilience and Security** — timeouts, retry/backoff, DLQ, consistência transacional, duplicação, idempotência, HMAC, segredos, HTTPS, rotação e demais controles efetivamente definidos. Para todo ponto de segurança ainda aberto, registre-o como aberto; não invente proteção ou garantia.
10. **Observability** — métricas, logs e tracing: o que medir/registrar, em qual evento do fluxo, e dimensões úteis sem expor segredos ou dados sensíveis. Reutilize os padrões encontrados no projeto. Se tracing ou uma métrica específica não estiver definida nas fontes, apresente-a como proposta e identifique-a como tal, ou pergunte se a decisão afetar escopo.
11. **Integration with the Existing System** — seção obrigatória específica deste desafio. Nomeie pelo menos **quatro caminhos de arquivo reais** do código base e explique concretamente como o módulo de webhooks se integrará com cada um. Inclua os pontos pertinentes indicados no README, como transação/método `changeStatus`, classes e códigos de erro, middleware de autenticação/autorização, error middleware centralizado e logger. Use somente caminhos e símbolos verificados; não presuma que nomes de exemplo do README correspondem literalmente aos caminhos existentes.
12. **Dependencies and Compatibility** — dependências internas/externas, impacto em deploy/configuração e compatibilidade com a aplicação existente. Não adicione tecnologias ou serviços não aprovados sem registrar como proposta em aberto.
13. **Technical Acceptance Criteria** — critérios verificáveis para os fluxos bem-sucedidos, falhas, consistência e integração.
14. **Risks and Mitigations** — riscos técnicos e operacionais relevantes, com probabilidade, impacto e mitigação quando essas informações puderem ser sustentadas pelas fontes. Identifique lacunas em vez de inventar probabilidades ou mitigações acordadas.

## Requisitos específicos do desafio

Valide explicitamente os seguintes critérios do README:

- Há pelo menos quatro endpoints HTTP, cada qual com exemplo de request e response e status codes.
- A matriz de erros utiliza códigos com prefixo `WEBHOOK_`.
- A seção **Integration with the Existing System** cita pelo menos quatro caminhos reais e explica a integração com cada um.
- A seção **Observability** cobre métricas, logs e tracing.
- Os fluxos de outbox, worker, retry e DLQ estão descritos em detalhe suficiente para implementação.
- Os conteúdos do FDD são consistentes com a transcrição e com o escopo do PRD; itens descartados/adiados não aparecem como funcionalidade ativa.

Quando o PRD explicitar limites, formatos, estados, payloads, headers ou prazos, preserve exatamente os valores. Por exemplo, não altere timeout, frequência de polling, limites de payload, política de retry, filtros de status, cabeçalhos ou garantias de entrega. Se uma informação estiver marcada como decisão em aberto, não a feche silenciosamente: descreva o ponto em aberto e solicite orientação se a implementação depender dele.

## Limites de escopo

- Crie ou atualize somente `docs/FDD.md`.
- Não altere `src/`, `prisma/`, `tests/`, configurações, `README.md`, `TRANSCRICAO.md`, PRD, RFC, ADRs ou tracker.
- Não crie ADRs, endpoints implementados, migrations, código, plano de implementação separado ou outros documentos.
- Referencie o código como contexto e integração, sem modificá-lo.

## Validação antes de salvar

Antes de gravar, valide:

- [ ] O FDD cobre todos os tópicos obrigatórios do README e as seções necessárias deste prompt.
- [ ] Há pelo menos quatro contratos HTTP, cada qual com request, response e status codes.
- [ ] A matriz de erros usa o prefixo `WEBHOOK_` e é coerente com o padrão existente.
- [ ] A seção de integração lista e explica pelo menos quatro caminhos de arquivo existentes, conferidos no repositório.
- [ ] Observabilidade inclui métricas, logs e tracing.
- [ ] Fluxos de transação/rollback, entrega, tentativas, DLQ e replay são coerentes entre si.
- [ ] Exemplos de payload, headers, códigos, estados e respostas não se contradizem entre seções.
- [ ] Decisões abertas permanecem explicitamente abertas; nenhuma lacuna foi preenchida com fatos inventados.
- [ ] O documento não contradiz a transcrição, o PRD, os ADRs relevantes ou o código existente.
- [ ] Nenhum arquivo fora do escopo foi alterado.

Se algum critério não puder ser atendido sem inventar informação, explique a lacuna e peça orientação ao usuário antes de salvar. Após salvar, releia `docs/FDD.md`, confira os contratos e caminhos citados e informe o caminho exato do arquivo.
