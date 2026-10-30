---
name: spec-review
description: Revisa um spec.md sozinho, antes do design, com um agente general-purpose e procedimento fixo, procurando requisito sem acceptance ou impossível de verificar, ambiguidade com duas leituras, contradição interna, regra de dado incompleta, exemplo que viola a própria regra e pendência aberta. Use quando a spec estiver pronta para virar design.md e você quiser saber o que ainda precisa ser fechado. Recebe o path do spec.md.
argument-hint: <path do spec.md>
---

# spec-review

Pergunta que a skill responde: "Esta spec está fechada o bastante para virar design? O que ainda tem duas leituras, não dá para verificar ou se contradiz?"

Argumento: `$ARGUMENTS` = path do spec.md.

## 1. Validar a entrada (antes de chamar o agente)

1. Confirme que o arquivo existe listando os cabeçalhos com `Grep` (padrão `^#`). Não leia o arquivo inteiro: quem lê é o agente. Se faltar o path ou o arquivo não existir, pare e peça o path. Não procure um "parecido".
2. Confira pelos cabeçalhos que é uma spec (requisitos, decisões, boundaries, acceptance). Se parecer um design (mapa de arquivos, tipos, ordem de implementação) ou uma lista de tasks, pergunte antes de seguir.
3. Não edite o arquivo em momento algum.

## 2. Chamar o agente

Chame o agente `general-purpose` com o `Agent` tool e o prompt abaixo, trocando `<SPEC>` pelo path. Não resuma nem altere o prompt.

```text
Você é um revisor adversarial de spec. Hipótese inicial: esta spec ainda não está pronta para
virar design, até a evidência provar o contrário. Não dê crédito por intenção: só conta o que
está escrito. Não amacie severidade para evitar conflito.

Insumo. Leia o arquivo inteiro com Read antes de qualquer outra ação, e não leia nenhum outro
arquivo (código, outros docs, git): <SPEC>

Objetivo: dizer o que o autor ainda precisa fechar para alguém escrever o design sem adivinhar.
Não proponha arquitetura nem solução técnica; julgue só se a spec é clara, completa,
verificável e coerente consigo mesma.

PASSO 1. Extraia os itens da spec. Um item é:
- cada requisito numerado, e o critério de acceptance dele como item separado;
- cada linha de tabela de decisões;
- cada entrada de In scope e de Out of scope;
- cada constraint;
- cada campo ou regra do modelo de dados (tipo, padrão, faixa, erro esperado).
NÃO é item, mas pode ser citado como evidência: contexto, interview log, ambiguity report,
histórico e seções de resultado de implementação. Use a numeração da própria spec como id
(Req 3, Req 3.acc, Decisão 9). Sem numeração, use seção + ordem. Diga como extraiu os itens.

PASSO 2. Aplique a cada item as checagens que valem para ele:
- C1 VERIFICÁVEL (requisito): tem acceptance? O acceptance é observável (número, estado,
  log, teste, cena) e dá para dizer "passou" ou "falhou" sem opinião?
- C2 SEM AMBIGUIDADE (todos): o texto admite duas leituras que levariam a designs diferentes?
  Só marque se conseguir escrever as duas leituras concretas.
- C3 COERENTE (todos): contradiz outro item da spec (decisão vs requisito, in scope vs out of
  scope, regra vs exemplo, número vs número)? Cite os dois trechos.
- C4 DADO COMPLETO (campo ou regra de dado): falta tipo, padrão quando ausente, faixa válida
  ou o que acontece com valor inválido?
- C5 FECHADO (todos): o item está marcado como pendente, aberto, a definir, "TBD", "talvez",
  ou a seção de ambiguidade aponta pendência nele?
- C6 EXEMPLO CONSISTENTE (exemplos de JSON, código ou tabela): o exemplo viola uma regra da
  própria spec (campo inexistente, valor fora da faixa, referência que não existe)? Diga se a
  spec chama o exemplo de ilustrativo ou de contrato.
- C7 TERMO DEFINIDO (todos): usa termo de domínio, nome de tipo ou mecânica que a spec não
  define nem atribui a outro documento citado?
Item que passa em todas as checagens que valem para ele fica OK.

PASSO 3. Checagens globais:
- G1: requisito sem nenhum acceptance, ou acceptance sem requisito.
- G2: decisão que nenhum requisito aplica (decisão órfã), ou requisito que depende de uma
  decisão que não está na spec.
- G3: a spec se declara fechada ("status", "ambiguidade", "pronta") mas tem item com C5.

PASSO 4. Melhorias: só as ancoradas em um trecho citado e que não sejam achados das
checagens. Proibido: arquitetura, preferência de estilo, boas práticas genéricas, requisito
novo que o autor não pediu. Requisito que parece faltar entra aqui como "possível lacuna", com
o trecho que sugere a falta.

Severidade (fixa, não ajuste caso a caso):
- BLOCKER: C1 falha (sem acceptance ou acceptance não verificável); C3 falha; C5 em requisito
  ou decisão; C6 em exemplo que a spec trata como contrato; G1.
- WARNING: C2; C4; C5 em item que não é requisito nem decisão; C6 em exemplo ilustrativo; C7;
  G2; G3.
- INFO: melhorias.

Regras de evidência:
- Todo achado cita o texto literal (até 25 palavras) com arquivo:linha. Citação é texto exato
  copiado, nunca paráfrase. Sem citação, o achado não existe.
- Ambiguidade (C2) só vale com as duas leituras escritas. Contradição (C3) só vale com os dois
  trechos citados.
- Não invente requisito, decisão ou trecho que a spec não tem.
- Se a spec citar outro arquivo como fonte, não o leia: liste em "Fora do alcance". Termo
  definido num arquivo citado não falha em C7.
- Na dúvida entre duas severidades, escolha a mais severa e diga por quê em uma frase.

Saída, exatamente neste formato e nesta ordem:

## Resumo
Itens da spec: N | OK a | com achado b
BLOCKER x | WARNING y | INFO z
Veredito: PRONTA PARA DESIGN (0 BLOCKER) ou REVISAR (1+ BLOCKER)

## Matriz
| Id | Item (spec:linha) | Checagens que falharam | Severidade máxima |
(uma linha por item do PASSO 1, inclusive os OK, com "—" quando nada falhou)

## Achados
### [SEVERIDADE] Id / Cn: título curto
- Spec (arquivo:linha): "trecho literal"
- Segundo trecho, quando C3 ou C6 (arquivo:linha): "trecho literal"
- Leitura A / Leitura B, quando C2: uma frase cada.
- O que falta: uma ou duas frases.
- Pergunta que fecha o achado: a pergunta que o autor responde para resolver, sem sugerir
  a resposta.
(BLOCKER primeiro; achados globais G1–G3 no fim desta seção)

## Melhorias
(do PASSO 4, com citação; "nenhuma" se vazio)

## Fora do alcance
(arquivos citados e não lidos; seções lidas só como evidência; "nenhum" se vazio)
```

## 3. Conferir o relatório antes de mostrar

1. Para cada citação do relatório, confirme com `Grep` (busca literal) que o trecho existe no arquivo. Se um trecho não existir, descarte o achado e liste-o no fim como "descartado: citação não encontrada".
2. Confira se "OK + com achado" no Resumo bate com o número de linhas da Matriz, e se a contagem de severidades bate com os achados. Se não bater, devolva ao agente pedindo a correção, uma vez só.
3. Confira que todo C2 tem as duas leituras e todo C3 tem os dois trechos. Achado sem isso é descartado como "descartado: sem evidência".
4. Se o agente fugiu do formato, leu outros arquivos ou propôs arquitetura em vez de revisar a spec, devolva o prompt uma vez, apontando o desvio. Se fugir de novo, mostre o que veio e avise o usuário.

## 4. Entregar

Mostre o relatório conferido, com as perguntas dos BLOCKER em destaque. Não corrija a spec por conta própria. No fim, ofereça em uma linha: responder as perguntas junto com o usuário e atualizar a spec, ou registrar os achados em `docs/Backlog/`.
