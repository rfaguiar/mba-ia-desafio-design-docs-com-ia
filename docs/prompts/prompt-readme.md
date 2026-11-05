# Prompt para documentar o processo no README

> Prompt reutilizável para executar no agente dentro do repositório do desafio.

## Instrução principal

Atualize o `README.md` da raiz para documentar o processo real de produção do pacote de design docs para o Sistema de Webhooks de Notificação de Pedidos. Use `DESAFIO.md` como fonte obrigatória dos requisitos. A saída desta tarefa é somente o README atualizado.

Antes de redigir, verifique se existe uma skill de documentação/README diretamente aplicável e use-a se estiver disponível. Não acione skills de PRD, RFC, FDD, ADR ou implementação para esta tarefa. O objetivo é documentar o processo já realizado, não gerar novamente os documentos do pacote.

## Fontes e levantamento

Leia integralmente `DESAFIO.md`. Use a seção **“README com o processo”**, os critérios de aceite do README e as regras gerais como contrato obrigatório. Em seguida:

1. Leia os documentos entregues para saber o que foi produzido: `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md`, `docs/TRACKER.md` e os ADRs disponíveis em `docs/adrs/`.
2. Leia os prompts customizados existentes em `docs/prompts/` e escolha pelo menos dois que tenham sido realmente escritos ou adaptados durante o trabalho e sejam relevantes para descrever o processo.
3. Examine o histórico e o contexto de trabalho disponíveis para identificar as ferramentas realmente utilizadas, a ordem de produção dos documentos e exemplos reais de iterações ou correções. Não presuma que uma ferramenta disponível foi efetivamente usada.
4. Confira a árvore de arquivos para registrar caminhos de entrega corretos e distinguir arquivos concluídos de placeholders ou ausentes.

O README atual contém o enunciado original. Substitua-o pela documentação do processo conforme `DESAFIO.md`. Preserve `DESAFIO.md`; não apague, mova ou copie seu conteúdo integral para o README. Se útil, inclua no README um link relativo para `DESAFIO.md` como referência ao enunciado.

Não fabrique ferramentas, papéis, revisões, problemas encontrados, quantidade de ciclos, resultados, decisões ou contribuições humanas. Se o histórico não comprovar algum fato exigido — especialmente ferramentas utilizadas e pelo menos duas iterações concretas — faça perguntas curtas ao usuário, uma por vez, e aguarde as respostas antes de afirmar esses fatos. Se não for possível confirmar um requisito, explique a lacuna e peça orientação em vez de inventar.

## Conteúdo obrigatório do README

Escreva em português, de forma clara e objetiva, usando exatamente as seções obrigatórias abaixo (os títulos podem incluir pequenas adaptações que preservem seu significado):

1. **Sobre o desafio** — um ou dois parágrafos descrevendo a tarefa com suas próprias palavras: transformar a reunião e o código do OMS em um pacote de design docs com apoio de IA, mantendo rastreabilidade e respeitando a fronteira entre PRD, RFC, ADRs, FDD e Tracker.
2. **Ferramentas de IA utilizadas** — liste somente ferramentas efetivamente usadas e descreva brevemente o papel real de cada uma. Diferencie agente, modelo, skill ou plugin quando necessário; não credite trabalho a uma ferramenta que apenas estava disponível.
3. **Workflow adotado** — explique a sequência real de trabalho e como os documentos foram produzidos e revisados. Compare com a ordem sugerida em `DESAFIO.md` sem alegar que seguiu uma etapa que não executou.
4. **Prompts customizados** — apresente pelo menos dois prompts relevantes que foram realmente escritos ou adaptados neste trabalho. Use blocos de código Markdown e transcreva trechos fiéis dos arquivos em `docs/prompts/`, identificando o prompt de origem. Não atribua ao agente execução de um prompt apenas porque o arquivo foi criado; se a execução não for conhecida, diga que foi preparado como prompt reutilizável.
5. **Iterações e ajustes** — descreva pelo menos dois exemplos concretos de conteúdo superficial, omisso ou incorreto que foi identificado e corrigido durante a produção. Informe a quantidade de ciclos principais somente se estiver registrada ou confirmada pelo usuário. Não invente erros ou melhorias genéricas para satisfazer a contagem; se não houver evidência suficiente, pergunte ao usuário.
6. **Como navegar a entrega** — liste os caminhos existentes dos documentos finais e sugira uma ordem de leitura que ajude o leitor a entender a solução. Inclua somente arquivos confirmados no repositório e marque claramente qualquer documento pendente.

O texto deve documentar o processo, não repetir o conteúdo técnico dos documentos. Evite afirmar que os requisitos foram cumpridos se ainda houver placeholders ou arquivos ausentes. Se apropriado, explique o papel complementar dos documentos: PRD para problema/escopo, RFC para proposta e questões abertas, ADRs para decisões, FDD para implementação e Tracker para rastreabilidade.

## Limites de escopo

- Edite somente o `README.md` da raiz.
- Não altere `DESAFIO.md`, `TRANSCRICAO.md`, documentos em `docs/`, código, testes ou configurações.
- Não crie novos prompts ou outros documentos nesta tarefa.
- Não copie o enunciado inteiro para o README; o foco é a jornada real de produção.

## Validação antes de concluir

Confira o README contra os critérios de `DESAFIO.md`:

- [ ] Contém todas as seis seções obrigatórias.
- [ ] “Sobre o desafio” tem um ou dois parágrafos.
- [ ] Lista pelo menos uma ferramenta de IA realmente utilizada e seu papel.
- [ ] Mostra pelo menos dois prompts customizados em blocos de código.
- [ ] Descreve pelo menos duas iterações ou ajustes concretos e verdadeiros.
- [ ] Explica o workflow realmente adotado.
- [ ] Lista os caminhos existentes dos documentos entregues e uma ordem sugerida de leitura.
- [ ] Não inventa fatos, ferramentas, iterações, etapas ou resultados.
- [ ] Preserva `DESAFIO.md` e não altera nenhum arquivo além do README da raiz.

Após a edição, releia o README, verifique os caminhos mencionados e confirme que os exemplos de prompts correspondem aos arquivos de origem. Informe ao usuário que o `README.md` foi atualizado e sinalize qualquer critério que não pôde ser confirmado.
