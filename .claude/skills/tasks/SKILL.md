---
name: tasks
description: Gera o tasks.md a partir de um design.md fechado, com 1 task por passo da ordem de implementação. Cada task diz o objetivo, os ids do design que entrega, passos concretos (código ou Editor), pontos de atenção das regras do projeto, critério de pronto pass/fail e dependências. Não inventa arquitetura: o que não está no design não vira task. Recebe o path do design.md.
argument-hint: <path do design.md>
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# tasks

Pergunta que a skill responde: "Em que passos, e em que ordem, o design vira código, cada um terminando com build verde e um critério de pronto verificável?"

Argumento: `$ARGUMENTS` = path do design.md.

- Saída: `tasks.md` na mesma pasta do design.
- Sem entrevista: o COMO já está decidido no design. Só pergunta quando o design tem uma lacuna.

Fluxo do repo: `/spec-phase` → `/spec-review` → `/design` → `/design-vs-spec` → `/tasks <design.md>` → `/tasks-vs-design`.

## Passo 1: Validar a entrada

1. Sem path, ou o arquivo não existe: pare e peça o path. Não procure um arquivo "parecido".
2. Leia o design inteiro. Ele precisa ter mapa de arquivos, tipos e ordem de implementação (§2, §3, §7). Se não tiver, pare e sugira `/design <spec.md>`.
3. O design precisa ter a avaliação de game patterns em §6 (Observer, Singleton, State, Object Pooling, Factory). Se não tiver, pare e sugira `/design`: a regra do projeto pede essa avaliação antes de qualquer task de implementação.
4. Se §10 tiver uma "lacuna da spec" aberta, avise. Pergunte se segue (a lacuna vira um ponto de atenção da task afetada) ou se para para fechar na spec.
5. Se já existir `tasks.md` na pasta, use AskUserQuestion (header "Tasks"): "Já existe um tasks.md. O que fazer?" com as opções "Atualizar" (regerar contra o design atual, mantendo o Status das tasks cujo conteúdo não mudou), "Ver" (mostrar e depois oferecer Atualizar/Sair) e "Sair" (não alterar nada).
6. O design e a spec não são editados em momento algum.

## Passo 2: Ler o contexto

- Leia `.claude/rules/` e anote quais regras cada task toca: arquitetura (módulo, ator, SO, namespace), C# (sem `var`, prefixo `_`, sem `?.`/`??` em objetos Unity), performance (sem alocação em hot path, pool) e state machine.
- Olhe `Assets/` com Glob para saber o que já existe. Um arquivo que o design manda "criar" e que já existe é um conflito: avise antes de seguir.

## Passo 3: Montar as tasks

**Uma task por `§7.passo.N`, na mesma ordem.** Só divida um passo se ele não couber numa sessão terminando com build verde. Nesse caso, as partes viram `T4a`, `T4b`, ..., e cada parte entrega ids diferentes. Nunca junte dois passos numa task.

Para cada task, levante no design:
- os ids que ela entrega: `§2.arquivo.N`, `§3.Tipo`, `§4.regra.N` e `§5.fluxo.N` (o passo do fluxo que pede código nesta task);
- os checks de §8 que ficam verdes ao fim dela;
- as decisões de §6 e os riscos de §10 que afetam como ela é feita.

**Passos concretos.** Cada passo é uma ação com o path e o nome do tipo. Marque o tipo de cada um:
- `[código]`: escrever ou alterar um arquivo `.cs` ou `.asmdef`, com as assinaturas de §3.
- `[Editor]`: ação manual no Unity Editor (criar um asset pelo menu `Create`, montar um prefab, ligar referências no Inspector, colocar uma cena no Build Settings).
- `[verificar]`: rodar testes no Test Runner ou executar um check manual de §8.

Nada de corpo de método: a task diz o quê e onde, com a regra de §4 a cumprir. O código é escrito na execução.

**Critério de pronto** são checkboxes pass/fail. Toda task tem pelo menos:
- [ ] Unity compila sem erros (Console sem erro de compilação).
- [ ] Os checks de §8 que a task fecha passam. Cite o id e o resultado esperado.

**O que não entra:**
- Tipo, arquivo, membro público, evento, pacote ou formato de dado que o design não lista. Se a task parecer precisar de algo assim, é lacuna do design: pare e pergunte se volta para `/design` ou se segue registrando a lacuna como ponto de atenção da task.
- Item de Fora do escopo da spec.
- Refactor ou melhoria de código que já existe e que o design não pede.

## Passo 4: Escrever o tasks.md

### Template

```markdown
# [Nome da feature] — Tasks

**Design:** [path do design.md]
**Criado:** [data] | **Atualizado:** [data]

## Índice

| Task | Título | Passo do design | Entrega | Depende de | Status |
|------|--------|-----------------|---------|------------|--------|
| T1 | | §7.passo.1 | §2.arquivo.1 | — | pendente |

Status: pendente / em andamento / feito. Só quem executa a task muda o Status.

## T1 — [Título] (§7.passo.1)

**Objetivo:** [uma frase: o que existe ao fim desta task que não existia antes]

**Entrega:** [ids do design: §2.arquivo.N, §3.Tipo, §4.regra.N, §5.fluxo.N]

**Passos:**
1. `[código]` [ação concreta com path e tipo]
2. `[Editor]` [ação concreta no Editor]
3. `[verificar]` [teste ou check, com o id de §8]

**Pontos de atenção:**
- [Regra de `.claude/rules/` que esta task toca, em uma linha]
- [Decisão Dn ou risco de §10 que afeta esta task]

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] [§8.check.N: resultado esperado]

**Depende de:** [— / T-N]
```

## Passo 5: Conferir antes de entregar

1. Todo `§7.passo` tem uma task (ou partes `Na`, `Nb`), na mesma ordem.
2. Todo `§2.arquivo`, todo `§3.Tipo`, toda `§4.regra` e todo `§8.check` aparece em **Entrega** ou no **Critério de pronto** de alguma task.
3. Nenhuma task cita um tipo, arquivo ou membro público que não está no design.
4. Toda dependência aponta para uma task anterior, e as dependências repetem as de §7.
5. Todo critério de pronto é pass/fail.

Se algo falhar, corrija o tasks.md antes de entregar.

## Passo 6: Encerrar

Não faça commit. Mostre:

```
tasks.md escrito — [N] tasks ([N] passos do design).
  Arquivo: [path]
  Cobertura: [N] arquivos, [N] tipos, [N] regras, [N] checks do design
  Passos manuais no Editor: [N]

Próximo: /tasks-vs-design [path do tasks.md] [path do design.md]
```

## Regras críticas

- As tasks não mudam o COMO: mudança de design volta para `/design`.
- Sem arquitetura nova: o que não está no design não vira task.
- Uma task por passo de §7; dividir pode, juntar não.
- Toda task termina com build verde e um critério de pronto verificável.
- Sem corpo de método.
- Com "Sair", nada é escrito.
