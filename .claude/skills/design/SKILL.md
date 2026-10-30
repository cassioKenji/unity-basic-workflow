---
name: design
description: Gera o design.md técnico a partir de um spec.md fechado. Investiga o projeto, pergunta só as gray areas de implementação, avalia os game patterns e escreve, na mesma pasta da spec, o mapa de arquivos, os tipos, regras, fluxos, decisões, a ordem de implementação, a verificação e o rastreio de cada requisito. Trata do COMO; o O QUÊ já está travado na spec. Recebe o path do spec.md.
argument-hint: <path do spec.md> [--auto]
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# design

Pergunta que a skill responde: "Como vamos construir exatamente o que a spec pede, neste projeto, seguindo as regras dele?"

Argumentos: `$ARGUMENTS` = path do spec.md, opcionalmente seguido de `--auto`.

- Saída: `design.md` na mesma pasta da spec.
- `--auto`: pula as perguntas. Claude escolhe a opção recomendada em cada gray area, registra em §6 como "auto" com o motivo e escreve o design.

Fluxo do repo: `/spec-phase` → `/spec-review` → `/design <spec.md>` → `/design-vs-spec` → `/tasks <design.md>` → `/tasks-vs-design`.

## Passo 1: Validar a entrada

1. Separe o path de `--auto`. Sem path, ou o arquivo não existe: pare e peça o path. Não procure um arquivo "parecido".
2. Leia a spec inteira. Ela precisa ter `## Requisitos` ou `## Requirements`. Se não tiver, pare e sugira `/spec-phase <path>`.
3. Se o Relatório de Ambiguidade tiver alguma linha ⚠, ou a spec tiver itens pendentes ("TBD", "a definir", "talvez"), avise quais são. Sem `--auto`: pergunte se continua ou se volta para `/spec-review`. Com `--auto`: continue e registre cada um em §10 como suposição.
4. Se já existir `design.md` na pasta, use AskUserQuestion (header "Design"): "Já existe um design.md. O que fazer?" com as opções "Atualizar" (revisar contra a spec atual), "Ver" (mostrar e depois oferecer Atualizar/Sair) e "Sair" (não alterar nada). Com `--auto`, escolha Atualizar e preserve as decisões já fechadas que a spec não invalidou.
5. A spec não é editada em momento algum.

## Passo 2: Investigar o projeto (antes de qualquer pergunta)

- `.claude/rules/`: architecture, csharp, performance, state-machine. O que elas decidem não vira pergunta: o design segue as regras.
- `Assets/Scripts/` com Grep/Glob: módulos, atores, systems, ScriptableObjects e estados que dá para reaproveitar ou que serão alterados. Anote o path e os membros públicos de cada um. Se `Assets/` não existir, o projeto é greenfield: registre isso.
- Os outros arquivos da pasta da spec e as specs/designs vizinhos em `docs/specs/` que tocam o mesmo assunto.

Para cada requisito, monte internamente: que peça (Módulo, Ator, System, ScriptableObject, Estado) entrega o requisito, se ela já existe ou é nova, e como o acceptance será verificado.

## Passo 3: Avaliar os game patterns

Para cada problema que o design resolve, avalie Observer, Singleton, State, Object Pooling e Factory: qual resolve, qual não se aplica e por quê. Use um pattern só quando ele resolve um problema que a spec cria. Pattern sem problema é overengineering.

Encaixe nas regras do projeto:
- Observer → `event` C# do módulo, ligado pelo ator em `OnEnable`/`OnDisable`.
- Singleton → só para Systems (`public static X Instance`).
- State → `StateMachineComponent` + `IState`, com os estados ao lado do ator.
- Object Pooling → `UnityEngine.Pool.ObjectPool<T>` para projéteis, efeitos e spawn frequente.
- Factory → só se houver criação com variações que um ScriptableObject sozinho não resolve.

## Passo 4: Gray areas

Uma gray area é uma decisão de implementação que a spec não fixa, que as regras não decidem e que tem mais de uma opção válida com consequência diferente. Exemplos: reaproveitar um módulo existente ou criar outro, ChangeState ou PushState, qual System é dono de um evento, valor em ScriptableObject ou derivado em runtime, pool ou instância única.

Não é gray area: nome de variável, detalhe interno de um método, qualquer coisa que a spec ou `.claude/rules/` já decidem.

**Sem gray areas:** vá direto ao Passo 5.

**Com `--auto`:** escolha a recomendada em cada uma e vá ao Passo 5.

**Sem `--auto`:** no máximo 3 rodadas, 2–3 perguntas por rodada, com AskUserQuestion. Opção recomendada primeiro, e cada opção diz a consequência dela. Ordem: primeiro as que mudam o mapa de arquivos ou os tipos, depois as de fluxo, por último as de dado.

Se uma resposta mudar o O QUÊ (requisito novo, requisito cortado, mudança de acceptance), não absorva no design: avise que é mudança de spec e sugira `/spec-phase <spec.md>`. Sem `--auto`, pergunte se para aqui ou segue registrando em §10 como lacuna da spec. Com `--auto`, siga registrando.

## Passo 5: Escrever o design.md

Numere tudo como no template (§2.arquivo.N, §4.regra.N, D1, §7.passo.N): `/design-vs-spec` e `/tasks-vs-design` citam esses ids.

**Regras do conteúdo:**
- Todo requisito e todo acceptance da spec aparece em §9, apontando para onde é tratado e como é verificado.
- Nada de Fora do escopo da spec entra no design.
- Nomes, namespaces e pastas seguem `.claude/rules/architecture.md` (`Assets/Scripts/Modules|Actors|Systems/...`, namespace em bloco).
- Tipos com membros públicos e eventos como assinaturas. Sem corpo de método: código é trabalho das tasks.
- Valor de gameplay vai para um ScriptableObject, com o campo e o default.
- Hot path (`Update`, `FixedUpdate`, colisão) diz como fica sem alocação, quando o requisito toca nisso.
- Regra de comportamento precisa ser verificável: diz o gatilho, o efeito e o caso de borda.

### Template

```markdown
# [Nome da feature] — Design

**Spec:** [path do spec.md] ([N] requisitos)
**Criado:** [data] | **Atualizado:** [data]

## 1. Visão geral

[Um parágrafo: que peças entram, como se conectam, o que é reaproveitado do projeto.]

## 2. Mapa de arquivos

| Id | Arquivo | Ação | Namespace | Papel |
|----|---------|------|-----------|-------|
| §2.arquivo.1 | Assets/Scripts/... | criar / alterar / remover / mover | ProjectName.... | Módulo / Ator / System / SO / Estado |

## 3. Tipos

### §3.[NomeDoTipo]
- Papel: [Módulo / Ator / System / SO / Estado] — [arquivo §2.arquivo.N]
- Membros públicos: [assinaturas]
- Eventos: [assinaturas]
- Depende de: [tipos/módulos, e como recebe a referência: GetComponent no Awake, SerializeField, Instance, construtor]

## 4. Regras

1. **§4.regra.1 [rótulo]**: [gatilho] → [efeito]. Borda: [caso de borda e o que acontece].

## 5. Fluxos

**§5.fluxo.1 [nome]:** A → B → C
[Uma linha por passo quando o fluxo tiver evento, troca de estado ou ordem que importa.]

## 6. Decisões e patterns

| Id | Problema | Escolha | Descartado (motivo) | Origem |
|----|----------|---------|---------------------|--------|
| D1 | | [pattern ou decisão] | | usuário / auto / regra do projeto |

## 7. Ordem de implementação

1. **§7.passo.1** [o que é feito] — depende de: [nada / §7.passo.N]

## 8. Verificação

| Id | Como verificar | Tipo | Cobre |
|----|----------------|------|-------|
| §8.check.1 | [passo concreto e resultado esperado] | teste EditMode / PlayMode / cena / Profiler / build | Req N.acc |

## 9. Rastreio

| Requisito da spec | Onde no design | Verificação |
|-------------------|----------------|-------------|
| Req 1 | §3.X, §4.regra.1 | §8.check.1 |

## 10. Riscos e suposições

- [Risco ou suposição] — [mitigação, ou "aceito"]
[Inclui as linhas ⚠ da spec e as lacunas da spec achadas no Passo 4. Se não houver: "nenhum".]
```

## Passo 6: Conferir antes de entregar

1. Todo requisito e todo acceptance da spec tem linha em §9, e todo id citado em §9 existe no design.
2. Todo arquivo de §2 é usado por algum tipo de §3, e todo tipo de §3 tem arquivo em §2.
3. Todo passo de §7 depende só de passos anteriores.
4. Nenhum item de Fora do escopo da spec aparece no design.

Se algo falhar, corrija o design antes de entregar.

## Passo 7: Encerrar

Não faça commit. Mostre:

```
design.md escrito — [N] arquivos, [N] tipos, [N] decisões.
  Arquivo: [path]
  Rastreio: [N]/[N] requisitos cobertos
  Suposições: [N] (ver §10)

Próximo: /design-vs-spec [path do design.md] [path do spec.md]
```

## Regras críticas

- O design não muda o O QUÊ: mudança de requisito volta para `/spec-phase`.
- Investigue o projeto ANTES da primeira pergunta.
- Não pergunte o que a spec ou `.claude/rules/` já decidem.
- Todo pattern usado tem um problema da spec que o justifica.
- Sem corpo de método: assinaturas sim, implementação não.
- Com "Sair", nada é escrito.
