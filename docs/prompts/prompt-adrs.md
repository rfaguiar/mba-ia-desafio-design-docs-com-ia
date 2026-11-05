# Prompt para criar os ADRs do sistema de webhooks

> Prompt reutilizável para executar no agente dentro do repositório do desafio.

## Instrução principal

Crie de **5 a 8 Architecture Decision Records (ADRs)** para o Sistema de Webhooks de Notificação de Pedidos descrito neste repositório. Salve um arquivo Markdown por decisão em `docs/adrs/`, sem modificar outros documentos ou arquivos de código.

Antes de começar, verifique as skills disponíveis no ambiente e acione uma skill específica de ADR ou decisões arquiteturais, caso exista. Siga suas instruções sem deixar de cumprir este prompt e os requisitos do `README.md`.

As skills que apenas produzem PRDs ou especificações e planos de implementação não são skills de ADR: não acione `prd-writer` nem `spec-writer` para produzir este conjunto de documentos. Se o `docs/PRD.md` obrigatório estiver ausente, incompleto ou inconsistente a ponto de não permitir identificar as decisões, interrompa e peça orientação; não gere outro tipo de documento como substituto. Não acione skills de implementação: este desafio é documental e proíbe alterar o código da aplicação.

## Fontes e preparação

Leia integralmente o `README.md` e use o requisito **“ADRs”** e os critérios de aceite correspondentes como contrato. Antes de escrever:

1. Leia `TRANSCRICAO.md` por inteiro. É a fonte primária para identificar quais decisões foram efetivamente tomadas, por quem e com quais alternativas e trade-offs.
2. Leia `docs/PRD.md` para alinhar os ADRs ao problema, ao escopo e às decisões e questões em aberto registrados no PRD.
3. Leia `docs/RFC.md` e `docs/FDD.md`, se disponíveis, para manter consistência e não confundir decisões arquiteturais com proposta geral ou detalhes de implementação.
4. Inspecione os ADRs existentes em `docs/adrs/` antes de nomear ou criar arquivos. Preserve documentos existentes; não os sobrescreva nem duplique decisões já registradas.
5. Se um ADR referenciar o sistema existente, confirme os caminhos, módulos ou padrões citados no código antes de incluí-los.

Toda decisão deve ser rastreável à transcrição ou ao código existente. Para cada evidência da transcrição, anote sua localização com timestamp e falante no formato `[hh:mm] Nome`. Para evidência do código, use o caminho real do arquivo. Não invente decisões, alternativas debatidas, justificativas, trade-offs, nomes, localizações ou conteúdo de código. Uma alternativa plausível, mas não mencionada na reunião, pode ser incluída para cumprir o formato do ADR, desde que seja identificada como alternativa de análise, não como algo debatido na reunião.

Diferencie decisões fechadas de assuntos adiados ou em aberto. Registre como decisão apenas o que foi acordado. Não converta questões em aberto em ADRs com status `Accepted`; se uma questão for importante para contextualizar uma decisão, descreva-a como uma restrição ou ponto pendente, sem decidir por conta própria.

Se as fontes divergirem ou não sustentarem uma decisão necessária, faça perguntas objetivas ao usuário antes de redigir. Não preencha lacunas com suposições.

## Decisões que o conjunto deve cobrir

O README exige de 5 a 8 ADRs e cobertura de pelo menos 5 destas 6 decisões principais discutidas na reunião:

1. Padrão **Transactional Outbox no MySQL**.
2. **Política de retry com backoff e Dead-Letter Queue (DLQ)**.
3. **Autenticação HMAC-SHA256 com secret por endpoint**.
4. Garantia **at-least-once** com identificador estável `X-Event-Id`.
5. **Worker em processo separado**, usando polling.
6. **Reuso dos padrões existentes do projeto**.

Procure registrar cada decisão em um ADR próprio, se ela for realmente independente e isso não criar duplicação. Cubra pelo menos cinco decisões confirmadas na reunião; cubra as seis quando as fontes sustentarem cada uma. Se uma das seis não tiver sido decidida, não a apresente como aceita: explique a lacuna e peça orientação ou deixe-a como questão aberta fora dos ADRs de decisões fechadas.

Decisões secundárias só devem virar ADRs adicionais quando forem decisões arquiteturais independentes e relevantes, sustentadas pelas fontes. Não crie ADRs apenas para atingir uma contagem maior. A quantidade final deve estar entre 5 e 8.

## Formato e conteúdo dos arquivos

Escreva os ADRs em inglês para manter consistência com `docs/PRD.md`. Use o formato MADR ou uma variante padrão, incluindo em **cada** arquivo pelo menos estas seções:

- **Status** — normalmente `Accepted` para decisão fechada na reunião; use outro status apenas quando sustentado pelos documentos.
- **Context** — problema, forças e restrições que tornam a decisão necessária.
- **Decision** — uma decisão clara e específica, sem misturar múltiplas decisões independentes.
- **Alternatives Considered** — pelo menos uma alternativa real discutida na reunião ou uma alternativa plausível, identificada corretamente, com seu trade-off.
- **Consequences** — consequências positivas e negativas; explicite o trade-off e impactos relevantes.

Inclua em cada ADR referências de rastreabilidade às fontes, preferencialmente numa seção breve **References** ou junto ao contexto: timestamps e falantes da transcrição e/ou caminhos reais do código. Não invente timestamps. Pelo menos um ADR deve citar explicitamente arquivos, módulos ou padrões do código existente, confirmados por inspeção.

Use nomes sequenciais, sem lacunas, no formato:

`docs/adrs/ADR-NNN-titulo-em-kebab-case.md`

Exemplos de nomes, a usar apenas se corresponderem ao conteúdo real:

- `ADR-001-outbox-no-mysql.md`
- `ADR-002-worker-separado-com-polling.md`
- `ADR-003-entrega-at-least-once.md`

Se houver ADRs numerados existentes, continue a sequência e evite duplicar decisões. O nome da pasta existente pode conter documentação auxiliar; preserve-a.

## Limites de escopo

- Produza somente os arquivos ADR necessários em `docs/adrs/`.
- Não altere `README.md`, `TRANSCRICAO.md`, PRD, RFC, FDD, tracker ou arquivos de código.
- Não escreva a implementação, plano de implementação, contratos HTTP detalhados ou especificação de esquema de banco; esses detalhes pertencem ao FDD.
- Não crie RFC, PRD, FDD ou tracker.
- Não use afirmações genéricas sem uma decisão concreta, contexto, alternativas e trade-offs.

## Validação antes de concluir

Confira o conjunto completo contra esta lista e os critérios do README:

- [ ] Existem entre 5 e 8 arquivos ADR individuais em `docs/adrs/`.
- [ ] Os nomes seguem `ADR-NNN-titulo-em-kebab-case.md`, com numeração sequencial e sem lacunas.
- [ ] Cada arquivo contém Status, Context, Decision, Alternatives Considered e Consequences.
- [ ] Cada ADR registra uma decisão arquitetural identificável e não duplica outro ADR.
- [ ] O conjunto cobre pelo menos 5 das 6 decisões principais listadas neste prompt, sem tratar questões abertas como decisões aceitas.
- [ ] Cada alternativa e trade-off está identificada corretamente como debatida na reunião ou plausível, sem falsa atribuição à transcrição.
- [ ] Pelo menos um ADR referencia caminhos, módulos ou padrões reais do código, verificados no repositório.
- [ ] As referências à transcrição incluem timestamps e falantes existentes, no formato `[hh:mm] Nome`.
- [ ] ADRs não contradizem `TRANSCRICAO.md`, `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md` ou o código.
- [ ] Nenhum arquivo fora do escopo foi criado ou modificado.

Se não for possível satisfazer um critério com as fontes disponíveis, pare e informe a lacuna ao usuário em vez de inventar conteúdo. Ao concluir, releia os ADRs criados, confirme seus caminhos e a consistência entre decisões, e informe a lista de arquivos produzidos.
