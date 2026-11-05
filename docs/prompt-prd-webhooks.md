# Prompt para criar o PRD do sistema de webhooks

> Prompt reutilizável para executar no agente dentro do repositório do desafio.

## Instrução principal

Ative e siga a skill disponível **`prd-writer`** para conduzir a criação do PRD. Comece pela Fase 1 da skill: confirme em uma frase o entendimento do produto e explore o contexto do repositório. Siga as fases e os procedimentos da skill, incluindo a entrevista iterativa, a validação antes de salvar e a conferência do arquivo final.

Não acione `spec-writer` nem `implement-feature`: esta tarefa é exclusivamente a produção do PRD, não a especificação técnica detalhada nem a implementação de código.

## Contexto e objetivo

Estamos trabalhando no desafio descrito no `README.md`: documentar o **Sistema de Webhooks de Notificação de Pedidos** para um Order Management System (OMS) existente. Produza somente o PRD desta feature, no caminho **`docs/PRD.md`**.

Antes de entrevistar o usuário:

1. Leia o `README.md` completo e use seus requisitos e critérios de aceite como referência obrigatória.
2. Leia `TRANSCRICAO.md` por inteiro. Trate a transcrição como fonte primária dos requisitos, decisões, restrições, alternativas descartadas e itens adiados.
3. Explore o código e a documentação existente para entender o produto, a arquitetura e as convenções relevantes ao PRD. O código é fonte de contexto, não autorização para propor mudanças.
4. Verifique se já existe documentação de produto que deva ser considerada.
5. Não altere arquivos de aplicação, incluindo `src/`, `prisma/`, `tests/` ou configurações. A única saída desta tarefa é `docs/PRD.md`; não crie RFC, FDD, ADRs, tracker ou outros documentos.

Se algum arquivo de contexto essencial não existir ou não puder ser lido, informe isso explicitamente e peça orientação. Não substitua uma fonte ausente por suposições.

## Regras de evidência e entrevista

- Toda afirmação factual do PRD deve ser rastreável à transcrição, ao código existente ou a uma resposta explícita do usuário durante a entrevista. Não invente requisitos, números, decisões, restrições, métricas, nomes de participantes ou comportamento do produto.
- Ao identificar um requisito ou decisão, registre para seu próprio controle a origem precisa: timestamp e falante da transcrição ou caminho real do arquivo de código. Não inclua esse controle como seção adicional no PRD.
- Diferencie claramente o que foi decidido, o que foi explicitamente descartado, o que foi adiado e o que permanece em aberto. Não transforme alternativas rejeitadas ou adiadas em requisitos desta versão.
- Se a informação necessária para completar um critério do README não estiver nas fontes, pergunte ao usuário durante a Fase 2, uma pergunta por vez, conforme a skill. Priorize esclarecer metas quantitativas, pelo menos oito requisitos funcionais, dois itens fora de escopo, riscos e critérios de aceitação.
- A skill permite inferir detalhes razoáveis quando o usuário não os especifica. Neste desafio, **não apresente inferências como fatos ou decisões**: pergunte quando a lacuna afetar escopo ou comportamento. Se uma inferência menor for indispensável, identifique-a explicitamente como hipótese e obtenha confirmação do usuário antes de registrá-la como requisito.
- Não encerre a entrevista até esclarecer os aspectos essenciais e as lacunas que possam levar a contradições com a transcrição. Em seguida, resuma o entendimento e peça confirmação antes de redigir.

## Requisitos para o documento

Escreva o PRD integralmente em **inglês**, conforme a skill `prd-writer`. O arquivo deve iniciar com o título do produto em H1 e conter **exatamente nove seções numeradas**, na ordem e com os títulos abaixo. Não acrescente seções de validação, próximos passos, data, versão ou cabeçalho com ID.

1. **Executive Summary** — resumo e contexto da feature, público, valor e funcionamento em alto nível.
2. **Problem and Opportunity** — problema e motivação, dores identificadas nas fontes e como a feature responde a elas.
3. **Target Audience** — público-alvo e cenários de uso; use somente personas que correspondam a grupos de usuários sustentados pelas fontes.
4. **Objectives** — objetivos verificáveis e métricas de sucesso com meta e condição de medição. O README exige pelo menos um objetivo com meta quantitativa. Se não houver base para um número, pergunte ao usuário; não fabrique uma meta.
5. **User Stories** — histórias concretas agrupadas por funcionalidade, usando IDs sequenciais `F01`, `F02` etc.
6. **Functionalities** — requisitos funcionais e não funcionais, regras e limites específicos, fluxos, decisões e trade-offs pertinentes, e dependências entre funcionalidades. Use os blocos de `Consumes`/`Provides` quando houver dependência funcional de dados, seguindo as regras da skill. Cada funcionalidade deve ter ID e blocos de `Capabilities` e `Experience`; inclua `Error Handling` apenas quando aplicável pelos critérios da skill.
7. **Out of Scope** — itens explicitamente descartados ou adiados durante a reunião. Inclua pelo menos dois itens confirmados na transcrição e deixe claro se foram descartados ou adiados; não deduza itens a partir do silêncio.
8. **Dependency Graph** — tabela topológica com todas as funcionalidades uma única vez, prioridades, dependências, ondas calculadas mecanicamente, legenda e diagrama Mermaid. Inclua `Foundation Features` somente se os critérios da skill forem atendidos.
9. **Acceptance Criteria** — critérios verificáveis de sucesso e falha por funcionalidade, mais uma estratégia de testes e validação compatível com o nível de produto do PRD. Inclua critérios de integração para cada declaração `Consumes`.

Além dos requisitos estruturais da skill, o PRD deve cobrir explicitamente: resumo e contexto, problema e motivação, público e cenários de uso, objetivos e métricas, escopo incluído e excluído, requisitos funcionais e não funcionais, decisões e trade-offs principais, dependências, riscos e mitigação, critérios de aceitação e estratégia de testes e validação.

O README exige pelo menos **dois riscos**, cada qual com probabilidade, impacto e mitigação. Use apenas riscos evidenciados pelas fontes ou confirmados pelo usuário. Não invente percentuais: use a escala qualitativa discutida ou peça esclarecimento. Garanta que cada problema tenha uma resposta no escopo e que o conteúdo do PRD não contradiga a transcrição ou o código.

## Validação antes de salvar

Siga a checklist e o limite de iterações da Fase 4 da skill. Além disso, confirme todos estes critérios de aceite do README:

- `docs/PRD.md` existe e está em Markdown, com as nove seções obrigatórias.
- Há pelo menos oito requisitos funcionais discutidos na reunião e identificáveis no PRD.
- Existe pelo menos um objetivo com métrica e meta quantitativa confirmadas.
- “Out of Scope” contém pelo menos dois itens descartados ou adiados explicitamente na reunião.
- “Risks” cobre pelo menos dois riscos, cada um com probabilidade, impacto e mitigação.
- O PRD inclui requisitos não funcionais, decisões/trade-offs, dependências e estratégia de testes e validação.
- IDs de funcionalidades são únicos, sequenciais e consistentes entre as seções 5, 6, 8 e 9.
- Nenhum conteúdo contradiz as fontes ou trata uma alternativa descartada/adiada como requisito ativo.
- A tabela de dependências é topológica; ondas e arestas do Mermaid correspondem exatamente às dependências.
- Toda dependência em `Consumes` aparece também na tabela de dependências; os contratos `Consumes`/`Provides` e os critérios de integração são consistentes.
- Não foram alterados código, testes, configurações ou documentos além de `docs/PRD.md`.

Se algum critério não puder ser atendido sem inventar informação, não o satisfaça por suposição. Explique a lacuna ao usuário e solicite a informação necessária antes de salvar. Após salvar, releia `docs/PRD.md`, confirme as seções 1 e 9 e informe ao usuário o caminho exato do arquivo.
