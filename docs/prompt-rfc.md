# Prompt para criar o RFC do sistema de webhooks

> Prompt reutilizável para executar no agente dentro do repositório do desafio.

## Instrução principal

Crie o documento **RFC (Request for Comments)** para a feature **Sistema de Webhooks de Notificação de Pedidos** do Order Management System (OMS) descrito neste repositório. Salve somente o RFC em `docs/RFC.md`.

Antes de começar, verifique as skills disponíveis e use-as somente se forem diretamente adequadas à tarefa. Não há uma skill específica para escrever RFC entre as skills atuais:

- Se `docs/PRD.md` não existir ou estiver ausente, pare e peça autorização para produzi-lo com a skill **`prd-writer`**. Não invente contexto de produto nem gere um PRD sem autorização.
- A skill **`spec-writer`** produz uma especificação de implementação e um plano, não um RFC. Não a acione para esta tarefa, pois isso criaria entregáveis fora do escopo e duplicaria o nível de detalhe do FDD.

Se uma nova skill específica para RFC estiver disponível no ambiente de execução, acione-a e siga suas instruções, preservando todos os requisitos deste prompt e do README.

## Contexto e fontes

Leia integralmente o `README.md` e use como contrato obrigatório os requisitos e critérios de aceite da seção **“RFC da feature”**. Antes de redigir:

1. Leia `TRANSCRICAO.md` por inteiro. Ela é a fonte primária para decisões, alternativas debatidas, trade-offs, questões adiadas, restrições e participantes.
2. Leia `docs/PRD.md` para entender o problema, o escopo e a terminologia da feature.
3. Leia os ADRs disponíveis em `docs/adrs/` para refletir decisões já formalizadas e preparar referências corretas.
4. Leia `docs/FDD.md` e demais documentos relevantes, se existirem, apenas para manter consistência e não duplicar o detalhamento de implementação.
5. Inspecione o código existente para verificar contexto arquitetural e compatibilidade da proposta. Cite somente arquivos e padrões que confirmar que existem.

Toda afirmação factual deve ser sustentada pela transcrição, pelo código ou pelos documentos existentes. Diferencie decisões fechadas, alternativas rejeitadas e assuntos adiados ou ainda em aberto. Não invente requisitos, resultados de discussões, trade-offs, nomes, métricas ou limitações. Se fontes divergirem, não resolva a divergência por suposição: apresente-a como questão a esclarecer.

Se faltar uma fonte essencial, se houver contradição que impeça uma proposta coerente ou se não for possível identificar os elementos mínimos abaixo, faça perguntas objetivas ao usuário, uma por vez, antes de redigir.

## Conteúdo obrigatório do RFC

Escreva o RFC em inglês, mantendo o estilo dos documentos existentes no pacote. Ele deve ser uma proposta concisa, com extensão aproximada de **2 a 4 páginas**, submetida à revisão da equipe. Cubra as seções abaixo, sem acrescentar detalhe de implementação que pertença ao FDD:

1. **Metadata** — author, status, date e reviewers. Use como revisores os participantes identificados na transcrição. Para o autor ou outros metadados não encontrados nas fontes, use `TBD` em vez de inventar. Para a data, use a data atual do ambiente se disponível; caso contrário, `TBD`. Use um status coerente com uma proposta submetida à revisão.
2. **Executive Summary (TL;DR)** — problema, proposta em alto nível e benefício esperado.
3. **Context and Problem** — contexto do OMS, problema que motiva a feature e limitações relevantes verificadas no código.
4. **Technical Proposal** — arquitetura proposta em nível geral, fluxo entre os componentes principais e justificativa da abordagem. Explique como a proposta se integra ao sistema existente usando evidências, sem prescrever contratos, esquemas, algoritmos ou passos detalhados de implementação que pertencem ao FDD.
5. **Alternatives Considered** — pelo menos duas alternativas reais discutidas e descartadas na reunião. Para cada alternativa, explique o trade-off que levou ao descarte, com base na transcrição. Não crie alternativas fictícias para atingir a quantidade mínima.
6. **Open Questions** — pelo menos dois pontos levantados na reunião que permaneceram sem decisão ou foram adiados. Descreva o que falta decidir, sem apresentá-los como decisões fechadas.
7. **Impact and Risks** — impactos esperados, riscos técnicos/operacionais relevantes e mitigação sustentados pelas fontes. Identifique claramente riscos que ainda não tenham mitigação acordada.
8. **Related Decisions** — links relativos para pelo menos dois ADRs correspondentes do pacote, com uma frase explicando a relação de cada ADR com a proposta.

## Referências aos ADRs

Verifique se existem pelo menos dois ADRs relevantes e use links Markdown relativos corretos a partir de `docs/RFC.md`, apontando para os arquivos em `docs/adrs/`. Não invente nomes de arquivos, números de ADR ou decisões.

Se não houver dois ADRs relevantes disponíveis, não crie ADRs como parte desta tarefa e não grave links quebrados como se fossem válidos. Informe ao usuário que a sequência sugerida no README produz os ADRs antes do RFC e peça que ele os disponibilize ou autorize a criação deles em uma tarefa separada. Retome o RFC depois que essa dependência for resolvida.

## Fronteira do documento

- O RFC responde **“como pretendemos resolver e o que ainda está em aberto?”**. Apresenta a proposta e abre espaço para revisão.
- Os ADRs registram decisões arquiteturais individuais já tomadas.
- O FDD descreve como construir, com fluxos detalhados, contratos, erros e integração de implementação.
- Não copie seções inteiras do PRD, dos ADRs ou do FDD. Resuma apenas o contexto necessário para apresentar e avaliar a proposta.
- Não gere nem altere PRD, FDD, ADRs, tracker, README ou arquivos de código. Não execute mudanças em `src/`, `prisma/`, `tests/` ou configurações.

## Validação e gravação

Antes de salvar, confira:

- [ ] O RFC existe em `docs/RFC.md`, está em Markdown e tem extensão aproximada de 2 a 4 páginas.
- [ ] Inclui metadados de author, status, date e reviewers; participantes usados como revisores foram identificados na transcrição.
- [ ] Inclui resumo executivo, contexto/problema e proposta técnica em nível de arquitetura.
- [ ] Apresenta pelo menos duas alternativas efetivamente discutidas e descartadas, cada uma com seu trade-off respaldado pela transcrição.
- [ ] Apresenta pelo menos duas questões efetivamente adiadas ou não decididas, sem convertê-las em decisões.
- [ ] Inclui impacto e riscos e links funcionais para pelo menos dois ADRs existentes e relevantes.
- [ ] Não contradiz a transcrição, o PRD, os ADRs ou o código; não contém afirmações sem origem identificável.
- [ ] Não duplica o nível de detalhe de implementação reservado ao FDD.
- [ ] Nenhum arquivo além de `docs/RFC.md` foi criado ou modificado.

Se um critério não puder ser atendido sem inventar informação, não o preencha por suposição: explique a lacuna e solicite orientação. Ao concluir, releia `docs/RFC.md`, confira os títulos das seções e os destinos dos links, e informe o caminho exato do arquivo.
