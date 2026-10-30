---
name: design-vs-spec
description: Compara o design.md contra o spec.md com um agente general-purpose e procedimento fixo, procurando gaps (requisito ou mecânica da spec esquecida na arquitetura técnica) e oportunidades de melhoria. Use quando quiser saber se o design técnico contemplou 100% do que a spec pediu, antes de gerar as tasks. Recebe o path do design.md e o path do spec.md.
argument-hint: <path do design.md> <path do spec.md>
---

# design-vs-spec

Pergunta que a skill responde: "O design técnico contemplou 100% do que a spec pediu? Alguma mecânica da spec foi esquecida na arquitetura técnica?"

Argumentos: `$ARGUMENTS` = path do design.md, depois path do spec.md.

## 1. Validar a entrada (antes de chamar o agente)

1. Confirme que os dois arquivos existem listando os cabeçalhos com `Grep` (padrão `^## `). Não leia os arquivos inteiros: quem lê é o agente. Se faltar um path ou um arquivo não existir, pare e peça o path. Não procure um "parecido".
2. Confira a ordem pelos cabeçalhos: o design costuma ter mapa de arquivos, tipos ou fluxo; a spec costuma ter requisitos, decisões, boundaries e acceptance. Se a ordem parecer trocada, pergunte antes de seguir.
3. Não edite nenhum dos dois arquivos em momento algum.

## 2. Chamar o agente

Chame o agente `general-purpose` com o `Agent` tool e o prompt abaixo, trocando `<DESIGN>` e `<SPEC>` pelos paths. Não resuma nem altere o prompt.

```text
Você é um auditor adversarial. Hipótese inicial: o design deixou algo da spec de fora, até a
evidência provar o contrário. Não dê crédito por intenção: só conta o que está escrito no
design. Não amacie severidade para evitar conflito.

Insumos. Leia os dois inteiros com Read antes de qualquer outra ação, e não leia nenhum outro
arquivo (código, outros docs, git):
- spec: <SPEC>
- design: <DESIGN>

Objetivo: dizer se o design cobre tudo o que a spec pede. Não julgue se o design é bom
tecnicamente; julgue só a cobertura.

PASSO 1. Extraia os itens da spec. Um item é uma unidade verificável:
- cada requisito numerado, e o critério de acceptance dele como item separado;
- cada linha de tabela de decisões;
- cada entrada de In scope e de Out of scope;
- cada constraint;
- cada regra de dado ou validação do modelo de dados (campo, padrão, faixa, erro esperado)
  que a spec trava: em requisito, constraint, decisão, ou em texto que diga que aquilo é fixo.
Exemplo não travado: campo ou elemento que só aparece em JSON ou código de exemplo, quando a
spec diz que o exemplo é ilustrativo ou deixa os campos para o design, NÃO é item. Liste em
Melhorias como "exemplo não travado: <elemento> (spec:linha)" e diga se o design o trata.
Use a numeração da própria spec como id (Req 3, Req 3.acc, Decisão 9). Sem numeração, use
seção + ordem (Boundaries.out.2). Se a spec não tiver essas seções, diga como extraiu os itens.

PASSO 2. Para cada item, procure no design onde ele é tratado e dê um status:
- COBERTO: o design diz como entrega o item (tipo, arquivo, fluxo, regra ou teste).
- PARCIAL: o design trata o item, mas falta uma parte. Diga qual parte.
- AUSENTE: nada no design trata o item. Antes de marcar, busque no design com Grep pelo menos
  2 termos-chave do item (nome de campo, classe, palavra do requisito) e liste os termos.
- CONTRADIZ: o design decide diferente da spec.
- NAO_VERIFICAVEL: o item da spec é vago demais para dizer se foi coberto.
Para um item de acceptance, COBERTO exige que o design diga como aquilo será verificado.

PASSO 3. Direção inversa: liste o que o design entrega e a spec não pede, e qualquer parte do
design que caia em um item de Out of scope.

PASSO 4. Melhorias: só as ancoradas em um trecho citado. Exemplos: ambiguidade que gera duas
leituras, risco que o design reconhece sem mitigar, acceptance sem forma de medir, nome ou tipo
diferente entre spec e design. Proibido: arquitetura alternativa sem âncora, preferência de
estilo, boas práticas genéricas. Lacuna da própria spec (requisito que parece faltar) entra
aqui, marcada como "lacuna da spec", nunca como gap do design.

Severidade (fixa, não ajuste caso a caso):
- BLOCKER: AUSENTE ou CONTRADIZ; design entrega algo de Out of scope.
- WARNING: PARCIAL; NAO_VERIFICAVEL.
- INFO: escopo extra que não bate em Out of scope; melhorias.

Regras de evidência:
- Todo achado e toda linha da matriz citam o texto literal (até 25 palavras) com
  arquivo:linha. Citação é texto exato copiado, nunca paráfrase. Sem citação, o achado não existe.
- Não invente requisito que a spec não escreve, nem trecho de design que não está lá.
- Se um documento citar outro arquivo como fonte, não o leia: liste em "Fora do alcance".
- Na dúvida entre dois status, escolha o mais severo e diga por quê em uma frase. Essa regra
  não decide se um trecho é item: na dúvida entre requisito e exemplo, aplique a regra do
  exemplo não travado do PASSO 1.

Saída, exatamente neste formato e nesta ordem:

## Resumo
Itens da spec: N | COBERTO a | PARCIAL b | AUSENTE c | CONTRADIZ d | NAO_VERIFICAVEL e
BLOCKER x | WARNING y | INFO z
Veredito: APROVADO (0 BLOCKER) ou REVISAR (1+ BLOCKER)

## Matriz de cobertura
| Id | Item da spec (spec:linha) | Status | Onde no design (design:linha) | Severidade |
(uma linha por item do PASSO 1, inclusive os COBERTO)

## Achados
### [SEVERIDADE] Id: título curto
- Spec (arquivo:linha): "trecho literal"
- Design (arquivo:linha): "trecho literal", ou "buscado: termo1, termo2 (sem resultado)"
- O que falta ou diverge: uma ou duas frases.
- Sugestão (exemplo de caminho, não prescrição): uma frase.
(só itens não COBERTO, BLOCKER primeiro)

## Escopo extra do design
(do PASSO 3, com citação; "nenhum" se vazio)

## Melhorias
(do PASSO 4, com citação; "nenhuma" se vazio)

## Fora do alcance
(arquivos citados e não lidos; "nenhum" se vazio)
```

## 3. Conferir o relatório antes de mostrar

1. Para cada citação do relatório, confirme com `Grep` (busca literal) que o trecho existe no arquivo citado. Se um trecho não existir, descarte o achado e liste-o no fim como "descartado: citação não encontrada".
2. Confira se a soma dos status no Resumo bate com o número de linhas da Matriz. Se não bater, devolva ao agente pedindo a correção, uma vez só.
3. Se o agente fugiu do formato, leu arquivos além dos dois insumos ou avaliou qualidade técnica em vez de cobertura, devolva o prompt uma vez, apontando o desvio. Se fugir de novo, mostre o que veio e avise o usuário.

## 4. Entregar

Mostre o relatório conferido. Não corrija spec nem design por conta própria. No fim, ofereça em uma linha: corrigir o design para os BLOCKER, ou registrar os achados em `docs/Backlog/`.
