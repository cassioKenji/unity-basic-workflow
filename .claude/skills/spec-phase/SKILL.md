---
name: spec-phase
description: Entrevista socrática com score de ambiguidade que transforma um markdown (vazio, notas soltas ou spec antiga) numa spec com requisitos falseáveis, boundaries explícitos e acceptance pass/fail. Trata só do O QUÊ e do PORQUÊ, nunca do COMO. Use quando quiser fechar uma feature antes de escrever o design. Recebe o path do markdown, que é reescrito no lugar.
argument-hint: <path do markdown> [--auto]
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# spec-phase

Pergunta que a skill responde: "O que exatamente esta feature entrega, o que fica de fora e como sabemos que ficou pronta?"

Argumentos: `$ARGUMENTS` = path do markdown, opcionalmente seguido de `--auto`.

- O markdown **é a própria spec**: a skill lê o que houver nele como ponto de partida e reescreve no mesmo path.
- `--auto`: pula a entrevista. Claude escolhe os defaults recomendados, registra cada escolha no Interview Log como "auto" e escreve a spec.

Fluxo do repo: `/spec-phase <spec.md>` → `/spec-review <spec.md>` → `/design <spec.md>` → `/design-vs-spec` → `/tasks <design.md>` → `/tasks-vs-design`.

## Modelo de ambiguidade

Cada dimensão recebe uma nota de 0.0 (nada claro) a 1.0 (totalmente claro):

| Dimensão            | Peso | Mínimo | O que mede                                                      |
|---------------------|------|--------|-----------------------------------------------------------------|
| Goal Clarity        | 35%  | 0.75   | O resultado é específico e mensurável?                          |
| Boundary Clarity    | 25%  | 0.70   | O que está dentro e o que está fora do escopo?                  |
| Constraint Clarity  | 20%  | 0.65   | Performance, plataforma, input, dados, dependências?            |
| Acceptance Criteria | 20%  | 0.70   | Como sabemos que está pronto (Play Mode, teste, log, cena)?     |

**Ambiguidade** = 1.0 − (0.35×goal + 0.25×boundary + 0.20×constraint + 0.20×acceptance)

**Gate:** ambiguidade ≤ 0.20 E todas as dimensões ≥ mínimo → pronto para escrever a spec.

## Perspectivas da entrevista

Alterne as perspectivas: cada uma revela pontos cegos diferentes.

- **Researcher (rodadas 1–2):** o que existe hoje no projeto relacionado a isso? Qual o delta até o alvo? O que motiva o trabalho: o que está quebrado ou faltando?
- **Simplifier (rodada 2):** qual a versão mais simples que resolve o problema central? Se cortar 50%, o que sobra de irredutível?
- **Boundary Keeper (rodada 3):** o que explicitamente NÃO será feito? Que problemas vizinhos dá vontade de resolver junto, mas não deve? Qual o entregável final?
- **Failure Analyst (rodada 4):** como seria uma versão quebrada disso no jogo? O que faria um verificador reprovar a entrega? Que caso de borda (morte durante a ação, pausa, input simultâneo, objeto destruído) invalida um requisito?
- **Seed Closer (rodadas 5–6):** "[dimensão] está em [nota]. O que a deixaria totalmente clara?" / "A ambiguidade restante está em [área]. Dá para decidir agora?"

## Passo 1: Validar a entrada

1. Separe o path de `--auto`. Sem path, pare e peça o path. Não procure um arquivo "parecido".
2. Arquivo não existe: pergunte se deve criar nesse path. Com `--auto`, crie.
3. Leia o arquivo inteiro e classifique:
   - **Vazio** → spec nova.
   - **Notas soltas** (não tem `## Requisitos` nem `## Requirements`) → material bruto. Tudo o que as notas dizem precisa acabar na spec (requisito, constraint, fora do escopo ou Background). Nada some sem registro.
   - **Já é spec** (tem `## Requisitos` ou `## Requirements`) → use AskUserQuestion (header "Spec"): "Este arquivo já é uma spec. O que fazer?" com as opções "Atualizar" (revisar e re-pontuar), "Ver" (mostrar e depois oferecer Atualizar/Sair) e "Sair" (não alterar nada). Com `--auto`, escolha Atualizar.

## Passo 2: Investigar o projeto (antes de qualquer pergunta)

Leia e procure, nesta ordem:
- Os outros arquivos da mesma pasta do markdown (design.md, tasks, notas): contexto da mesma feature.
- `docs/README.md` e as specs vizinhas em `docs/specs/` que tocam o mesmo assunto.
- `.claude/rules/`: as constraints de projeto que já valem (arquitetura de módulos, performance, FSM). Não pergunte de novo o que uma regra já decide.
- `Assets/Scripts/` com Grep/Glob: módulos, atores, systems, ScriptableObjects e estados relacionados. Se `Assets/` não existir, o projeto é greenfield: registre isso.

Sintetize internamente, sem mostrar ao usuário ainda:
- O que existe hoje relacionado à feature.
- O gap entre o estado atual e o objetivo.
- O entregável principal: qual comportamento ou capacidade ainda NÃO existe.

## Passo 3: Primeira avaliação

Pontue as 4 dimensões só com base no markdown e no que a investigação achou, e mostre:

```
Goal Clarity:        [nota]
Boundary Clarity:    [nota]
Constraint Clarity:  [nota]
Acceptance Criteria: [nota]
Ambiguidade: [nota]
```

Com `--auto` e o gate já passando: pule a entrevista e vá ao Passo 5.

## Passo 4: Loop de entrevista

**Máximo de 6 rodadas, 2–3 perguntas por rodada.** A rodada termina quando o usuário responde. Use AskUserQuestion com a opção recomendada primeiro.

- Rodada 1: Researcher
- Rodada 2: Researcher + Simplifier
- Rodada 3: Boundary Keeper
- Rodada 4: Failure Analyst
- Rodadas 5–6: Seed Closer, focando nas dimensões com nota mais baixa

Depois de cada rodada, atualize as notas e mostre:

```
Após a rodada [N]:
  Goal Clarity:        [nota] (mín 0.75) [✓ ou ↑ falta]
  Boundary Clarity:    [nota] (mín 0.70) [✓ ou ↑ falta]
  Constraint Clarity:  [nota] (mín 0.65) [✓ ou ↑ falta]
  Acceptance Criteria: [nota] (mín 0.70) [✓ ou ↑ falta]
  Ambiguidade: [nota] (gate: ≤ 0.20)
```

**Gate passou:** AskUserQuestion (header "Gate"): "Ambiguidade em [nota]. Os requisitos estão claros o bastante. Escrever a spec?" com as opções "Sim, escrever" e "Mais uma rodada". Com `--auto`, escreva.

**6 rodadas sem passar no gate:** AskUserQuestion (header "Limite"): "Após 6 rodadas a ambiguidade está em [nota]. Abaixo do mínimo: [dimensões]. O que fazer?" com as opções:
- "Escrever mesmo assim, marcando as lacunas"
- "Continuar conversando" (sem limite de rodadas daqui em diante)
- "Abandonar" (sai sem escrever nada)

Com `--auto`: escreva marcando as dimensões abaixo do mínimo.

## Passo 5: Escrever a spec

Sobrescreva o path recebido com o template abaixo.

**Todo requisito tem:**
- Uma frase específica e testável.
- Atual: o que existe hoje (ou "não existe").
- Alvo: o que passa a existir.
- Acceptance: o check pass/fail que um verificador executa.

**Requisitos vagos são rejeitados:**
- ✗ "O pulo deve ser responsivo"
- ✗ "Melhorar a sensação do combate"
- ✓ "O pulo começa no mesmo frame do input quando o Player está no chão; no ar, o input é ignorado"
- ✓ "Com 200 zumbis ativos, a cena não aloca heap por frame (GC Alloc = 0 B no Profiler)"

**Boundaries** são duas listas explícitas, nunca prosa: Dentro do escopo, e Fora do escopo com o motivo de cada item.

**Acceptance criteria** são checkboxes pass/fail. Nada de "fica bom" ou "parece razoável".

Dimensão abaixo do mínimo: marque no Relatório de Ambiguidade com `⚠ Abaixo do mínimo — o design trata como suposição`.

### Template

```markdown
# [Nome da feature] — Spec

**Criada:** [data] | **Atualizada:** [data]
**Ambiguidade:** [nota] (gate: ≤ 0.20)
**Requisitos:** [N] fechados

## Objetivo

[Uma frase precisa e mensurável. Não "melhorar X": "X muda de A para B".]

## Background

[Estado atual, vindo da investigação: o que existe hoje, o que falta, o que motiva o trabalho. Ancorado no projeto real.]

## Requisitos

1. **[Rótulo curto]**: [Frase específica e testável.]
   - Atual: [o que existe ou NÃO existe hoje]
   - Alvo: [o que passa a existir]
   - Acceptance: [check pass/fail concreto]

## Boundaries

**Dentro do escopo:**
- [Entregável ou comportamento concreto]

**Fora do escopo:**
- [O que não será feito] — [motivo]

## Constraints

[Performance, plataforma, input, dados, dependências. As regras de `.claude/rules/` já valem e não precisam ser repetidas; liste só o que é específico desta feature. Se não houver: "Sem constraints além das convenções do projeto."]

## Acceptance Criteria

- [ ] [Critério pass/fail, sem ambiguidade]

## Relatório de Ambiguidade

| Dimensão            | Nota | Mín   | Status | Notas |
|---------------------|------|-------|--------|-------|
| Goal Clarity        |      | 0.75  |        |       |
| Boundary Clarity    |      | 0.70  |        |       |
| Constraint Clarity  |      | 0.65  |        |       |
| Acceptance Criteria |      | 0.70  |        |       |
| **Ambiguidade**     |      | ≤0.20 |        |       |

Status: ✓ = atingiu o mínimo, ⚠ = abaixo do mínimo (o design trata como suposição)

## Interview Log

| Rodada | Perspectiva | Resumo da pergunta | Decisão fechada |
|--------|-------------|--------------------|-----------------|
| 1      | Researcher  |                    |                 |

[Com --auto: marque as decisões como "auto" e dê o motivo da escolha.]
```

## Passo 6: Encerrar

Não faça commit. Mostre:

```
Spec escrita — [N] requisitos fechados.
  Arquivo: [path]
  Ambiguidade: [nota] (gate: ≤ 0.20)

Próximo: /spec-review [path]
```

## Regras críticas

- Todo requisito tem Atual, Alvo e Acceptance.
- Boundaries é obrigatório e nunca fica vazio; as duas listas são explícitas.
- Acceptance é sempre pass/fail.
- Com "Abandonar" ou "Sair", o arquivo não é alterado.
- NÃO pergunte sobre COMO implementar (módulos, patterns, classes, arquitetura): isso é trabalho do design.md.
- Investigue o projeto ANTES da primeira pergunta: só perguntas ancoradas.
- No máximo 2–3 perguntas por rodada.
- Conteúdo de notas soltas nunca some: entra na spec ou vai para Fora do escopo com motivo.
