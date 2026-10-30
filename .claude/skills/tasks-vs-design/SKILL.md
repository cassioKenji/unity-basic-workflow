---
name: tasks-vs-design
description: Compara as tasks (tasks.md e os arquivos de task que ele lista) contra o design.md com um agente general-purpose e procedimento fixo, procurando parte do design sem task (tipo, arquivo, regra ou teste que ninguém cria), task que inventa arquitetura fora do design e ordem divergente. Use antes de executar as tasks, para saber se elas implementam exatamente o que foi desenhado. Recebe o path do tasks.md e o path do design.md.
argument-hint: <path do tasks.md> <path do design.md>
---

# tasks-vs-design

Pergunta que a skill responde: "As tasks implementam exatamente o que foi desenhado? Nenhuma task inventou arquitetura nova fora do design.md? Nenhuma struct planejada ficou sem task de criação?"

Argumentos: `$ARGUMENTS` = path do tasks.md, depois path do design.md.

## 1. Validar a entrada (antes de chamar o agente)

1. Confirme que os dois arquivos existem listando os cabeçalhos com `Grep` (padrão `^#`). Não leia os arquivos inteiros: quem lê é o agente. Se faltar um path ou um arquivo não existir, pare e peça o path. Não procure um "parecido".
2. Confira a ordem pelos cabeçalhos: o design costuma ter mapa de arquivos, tipos, fluxo ou ordem de implementação; o tasks.md costuma ter uma tabela de tasks ou seções por task. Se a ordem parecer trocada, pergunte antes de seguir.
3. Monte a lista de arquivos de task:
   - Se o tasks.md tem links para arquivos de task (tabela `| Task | Arquivo | ...` ou links `.md`), resolva cada link relativo à pasta do tasks.md e confirme que cada arquivo existe. Se algum link não resolver, pare e mostre quais faltam.
   - Se o tasks.md não tem links, as tasks são as seções do próprio arquivo. A lista é vazia.
4. Não edite nenhum dos arquivos em momento algum.

## 2. Chamar o agente

Chame o agente `general-purpose` com o `Agent` tool e o prompt abaixo, trocando `<TASKS>`, `<DESIGN>` e `<TASK_FILES>` (um path por linha, ou "nenhum") pelos valores do passo 1. Não resuma nem altere o prompt.

```text
Você é um auditor adversarial. Hipótese inicial: as tasks deixaram parte do design de fora ou
inventaram algo que o design não pede, até a evidência provar o contrário. Não dê crédito por
intenção: só conta o que está escrito nas tasks. Não amacie severidade para evitar conflito.

Insumos. Leia todos inteiros com Read antes de qualquer outra ação, e não leia nenhum outro
arquivo (código, spec, outros docs, git):
- design: <DESIGN>
- índice de tasks: <TASKS>
- arquivos de task (listados pelo índice):
<TASK_FILES>

Objetivo: dizer se as tasks implementam exatamente o design, nem menos nem mais. Não julgue se
o design é bom, nem se a task é bem escrita; julgue só a correspondência.

O que conta numa task: objetivo, passos, pontos de atenção, critério de pronto e dependências.
Ignore seções de resultado ou execução já feita ("Resultado", "Status", "Arquivos criados",
"Correções da revisão"): elas dizem o que aconteceu, não o que a task pede. Se uma task só
existir como resultado, sem objetivo, diga isso no achado.

PASSO 1. Extraia os itens do design. Um item é uma unidade que alguma task precisa entregar:
- cada tipo novo ou alterado (classe, struct, enum, record, interface, DTO), com os membros
  públicos que o design lista;
- cada arquivo do mapa de arquivos, com a ação (criar, alterar, remover, mover);
- cada regra de validação ou de comportamento que o design fixa;
- cada passo de fluxo que pede código novo (chamada, sinal, evento, troca de estado);
- cada passo da ordem de implementação, com as dependências entre eles;
- cada verificação que o design pede (teste, smoke test, cena, build).
NÃO é item: histórico de revisão, log de decisões passadas, alternativas descartadas,
riscos sem ação pedida. Use a numeração do próprio design como id (§3.BossPhase, §4.regra.2,
§7.passo.3, D5). Sem numeração, use seção + ordem. Diga como extraiu os itens se o design não
tiver essas seções.

PASSO 2. Para cada item, procure nas tasks onde ele é entregue e dê um status:
- COBERTO: uma task cria, altera ou verifica o item de forma explícita (nome do tipo, do
  arquivo ou da regra no objetivo, nos passos ou no critério de pronto).
- PARCIAL: uma task trata o item, mas falta parte (membro do tipo, regra da validação, caso do
  teste). Diga qual parte.
- AUSENTE: nenhuma task entrega o item. Antes de marcar, busque com Grep em todos os arquivos
  de task pelo menos 2 termos-chave (nome do tipo, do arquivo, do campo) e liste os termos.
- CONTRADIZ: a task faz diferente do design (outro tipo, outro nome, outra regra, outra ordem).
- NAO_VERIFICAVEL: o item do design é vago demais para dizer se uma task o entrega.
Para um item de verificação, COBERTO exige o caso no critério de pronto de alguma task.
Para um item de ordem, CONTRADIZ quando a dependência das tasks inverte a do design.

PASSO 3. Direção inversa: liste o que as tasks pedem e o design não tem. Separe em:
- ARQUITETURA NOVA: tipo, arquivo, API pública, dependência de pacote, sinal ou evento,
  estado de FSM, formato de dado que o design não lista.
- DETALHE: helper privado, ordem interna de passos, caso extra de teste, comentário, nome
  local. Na dúvida entre os dois, é ARQUITETURA NOVA.

PASSO 4. Melhorias: só as ancoradas em um trecho citado. Exemplos: task grande demais para um
passo com build verde que o design pede; critério de pronto que não dá para medir; nome
diferente entre task e design para a mesma coisa; dependência entre tasks que o índice não
declara. Proibido: reescrever a task, preferência de estilo, boas práticas genéricas. Lacuna
do próprio design entra aqui, marcada como "lacuna do design", nunca como gap das tasks.

Severidade (fixa, não ajuste caso a caso):
- BLOCKER: AUSENTE ou CONTRADIZ; ARQUITETURA NOVA.
- WARNING: PARCIAL; NAO_VERIFICAVEL.
- INFO: DETALHE; melhorias.

Regras de evidência:
- Todo achado e toda linha da matriz citam o texto literal (até 25 palavras) com
  arquivo:linha. Citação é texto exato copiado, nunca paráfrase. Sem citação, o achado não existe.
- Não invente item que o design não escreve, nem trecho de task que não está lá.
- Se um documento citar outro arquivo como fonte (spec, código), não o leia: liste em
  "Fora do alcance".
- Na dúvida entre dois status, escolha o mais severo e diga por quê em uma frase. Essa regra
  não decide se um trecho é item: aplique a lista de "NÃO é item" do PASSO 1.

Saída, exatamente neste formato e nesta ordem:

## Resumo
Itens do design: N | COBERTO a | PARCIAL b | AUSENTE c | CONTRADIZ d | NAO_VERIFICAVEL e
Arquitetura nova nas tasks: f | Detalhe extra: g
BLOCKER x | WARNING y | INFO z
Veredito: APROVADO (0 BLOCKER) ou REVISAR (1+ BLOCKER)

## Matriz de cobertura
| Id | Item do design (design:linha) | Status | Task que entrega (arquivo:linha) | Severidade |
(uma linha por item do PASSO 1, inclusive os COBERTO)

## Achados
### [SEVERIDADE] Id: título curto
- Design (arquivo:linha): "trecho literal"
- Task (arquivo:linha): "trecho literal", ou "buscado: termo1, termo2 (sem resultado)"
- O que falta ou diverge: uma ou duas frases.
- Sugestão (exemplo de caminho, não prescrição): uma frase.
(só itens não COBERTO e ARQUITETURA NOVA, BLOCKER primeiro)

## Extra nas tasks
(do PASSO 3, com citação e a classe ARQUITETURA NOVA ou DETALHE; "nenhum" se vazio)

## Melhorias
(do PASSO 4, com citação; "nenhuma" se vazio)

## Fora do alcance
(arquivos citados e não lidos; seções de resultado ignoradas, por task; "nenhum" se vazio)
```

## 3. Conferir o relatório antes de mostrar

1. Para cada citação do relatório, confirme com `Grep` (busca literal) que o trecho existe no arquivo citado. Se um trecho não existir, descarte o achado e liste-o no fim como "descartado: citação não encontrada".
2. Confira se a soma dos status no Resumo bate com o número de linhas da Matriz, e se "Arquitetura nova" bate com os itens dessa classe em "Extra nas tasks". Se não bater, devolva ao agente pedindo a correção, uma vez só.
3. Se o agente fugiu do formato, leu arquivos além dos insumos, usou seções de resultado como entrega ou avaliou qualidade técnica em vez de correspondência, devolva o prompt uma vez, apontando o desvio. Se fugir de novo, mostre o que veio e avise o usuário.

## 4. Entregar

Mostre o relatório conferido. Não corrija tasks nem design por conta própria. No fim, ofereça em uma linha: corrigir as tasks para os BLOCKER, ou registrar os achados em `docs/Backlog/`.
