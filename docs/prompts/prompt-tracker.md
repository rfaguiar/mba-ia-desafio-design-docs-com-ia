# Prompt para criar o Tracker de rastreabilidade

> Prompt reutilizável para executar no agente dentro do repositório do desafio.

## Instrução principal

Crie o documento **Tracker de Rastreabilidade** do pacote de design docs para o Sistema de Webhooks de Notificação de Pedidos. Salve somente o arquivo `docs/TRACKER.md`.

Verifique se há uma skill especificamente voltada à rastreabilidade ou geração de trackers e use-a se estiver disponível. Não use skills de PRD, RFC, FDD, ADR ou implementação como substituto: elas produzem outros entregáveis. Se documentos necessários para montar o tracker ainda forem placeholders ou não estiverem disponíveis, não tente completá-los por conta própria; informe ao usuário quais fontes faltam e peça orientação.

## Fontes e preparação

Leia integralmente o `README.md`, especialmente o requisito **“Tracker de Rastreabilidade”**, seu formato obrigatório, os critérios de aceite e as orientações contra alucinações. Depois:

1. Leia `TRANSCRICAO.md` por inteiro, identificando timestamps e falantes reais para cada afirmação usada como fonte.
2. Leia os documentos preenchidos que serão rastreados: `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md` e todos os ADRs existentes em `docs/adrs/`.
3. Ignore placeholders, comentários de template e texto não preenchido como evidência de requisito ou decisão.
4. Verifique no repositório os caminhos de código citados pelos documentos. Só use `CODIGO` quando o arquivo existir e sustentar o item.
5. Não altere os documentos-fonte nem o código. O único arquivo que pode criar ou editar é `docs/TRACKER.md`.

Se um documento-fonte ainda estiver em produção, não tiver conteúdo substancial ou houver contradições que impeçam determinar a origem dos itens, interrompa e informe ao usuário o que precisa ser concluído ou esclarecido. Não invente evidências para completar percentuais.

## Objetivo e regras de rastreabilidade

O Tracker é uma tabela de referência cruzada: cada linha representa um requisito, decisão, restrição, trade-off ou outro item identificável registrado nos documentos e aponta para sua **origem primária** na transcrição ou no código.

- Examine os documentos de forma sistemática e registre itens concretos e atômicos. Evite uma única linha genérica para vários requisitos independentes.
- Inclua itens de PRD, RFC, FDD e ADRs. A coluna `Documento` identifica onde o item aparece; a coluna `Fonte` aponta para a evidência original que o sustenta.
- Não use outro documento como valor da coluna `Fonte`: os únicos valores permitidos são `TRANSCRICAO` e `CODIGO`.
- Para itens originados na reunião, use `Fonte = TRANSCRICAO` e cite na coluna `Localização` o timestamp e o falante exatos no formato `[hh:mm] Nome`, como `[09:17] Diego`. Confira o horário e o nome diretamente em `TRANSCRICAO.md`; não estime timestamps nem atribua falas a participantes por inferência.
- Para itens sustentados pelo código, use `Fonte = CODIGO` e informe o caminho relativo exato do arquivo existente, por exemplo `src/modules/orders/order.service.ts`. Não inclua arquivos inexistentes, diretórios vagos ou caminhos supostos.
- Quando um item aparecer em mais de um documento, crie uma linha por ocorrência documental, mantendo o mesmo item identificável e indicando o respectivo documento. Evite duplicar a mesma ocorrência.
- Use IDs claros, únicos e consistentes, por exemplo `PRD-FR-01`, `RFC-ALT-02`, `FDD-CONTRATO-03` e `ADR-002`. Preserve IDs existentes nos documentos quando apropriado; não atribua IDs de ADR incorretos.
- A coluna `Tipo` deve descrever a natureza do item, como Requisito Funcional, Requisito Não Funcional, Decisão, Restrição, Trade-off, Contrato, Risco ou Critério de Aceitação.
- A coluna `Conteúdo (resumo)` deve resumir o item em uma linha, sem alterar seu sentido.
- Se não houver evidência rastreável, não invente uma origem nem force a linha para cumprir uma meta. Registre a lacuna para o usuário separadamente e não declare o tracker completo.

## Formato obrigatório

Escreva o conteúdo do Tracker em inglês para manter consistência com os documentos do pacote. Preserve exatamente o formato da tabela estabelecido no README, sem adicionar colunas:

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-FR-01 | docs/PRD.md | Functional Requirement | Example summary | TRANSCRICAO | [09:17] Diego |

Use uma linha por item rastreado. Não acrescente ao `docs/TRACKER.md` seções de resumo, metodologia, validação, contagem, observações ou itens sem fonte. Se precisar reportar problemas ou métricas de cobertura, faça isso na resposta ao usuário, não como colunas ou seções adicionais no Tracker.

## Critérios de aceite e validação

Antes de salvar, confira os requisitos do README:

- [ ] `docs/TRACKER.md` existe em Markdown e a tabela segue exatamente as seis colunas exigidas.
- [ ] Pelo menos 80% dos itens identificáveis nos documentos preenchidos têm uma linha correspondente no Tracker. Calcule a cobertura comparando itens concretos dos documentos com linhas rastreadas; não estime nem reduza artificialmente o denominador agrupando itens independentes.
- [ ] Pelo menos 70% das linhas têm `Fonte = TRANSCRICAO`, com localização válida no formato `[hh:mm] Nome`.
- [ ] Há pelo menos cinco linhas com `Fonte = CODIGO`, cada uma apontando para um caminho de arquivo existente e relevante.
- [ ] Todo timestamp e nome de falante foram conferidos diretamente na transcrição.
- [ ] Cada caminho marcado como `CODIGO` foi verificado no repositório e sustenta o resumo indicado.
- [ ] IDs são únicos; documentos e tipos estão corretamente identificados.
- [ ] Nenhuma linha contradiz as fontes ou trata como decidido algo que foi descartado, adiado ou permaneceu em aberto.
- [ ] Nenhum arquivo além de `docs/TRACKER.md` foi criado ou modificado.

Se qualquer meta não puder ser cumprida com as fontes reais, explique qual requisito ficou sem evidência e por quê. Não crie timestamps, falantes, requisitos, caminhos ou ocorrências fictícias para atingir os percentuais. Após salvar, releia `docs/TRACKER.md`, valide a tabela e os caminhos citados e informe o caminho exato do arquivo junto com as métricas de cobertura calculadas.
