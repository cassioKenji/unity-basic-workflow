# Flow de specs

Cada feature tem uma pasta própria em `docs/specs/<feature>/`. O flow leva um rascunho até tasks prontas para executar, passando por revisões que travam o O QUÊ antes do COMO.

```
docs/specs/<feature>/
├── spec.md     → O QUÊ e PORQUÊ (requisitos, boundaries, acceptance)
├── design.md   → COMO (arquivos, tipos, regras, fluxos, decisões, verificação)
└── tasks.md    → passos de implementação
```

## Flow

```
rascunho (vazio ou notas soltas em spec.md)
  → /spec-phase      → spec.md fechada
  → /spec-review     → BLOCKER? volta para /spec-phase ou corrige junto
  → /design          → design.md
  → /design-vs-spec  → gap? corrige o design (ou a spec, se for lacuna dela)
  → /tasks           → tasks.md
  → /tasks-vs-design → gap? corrige as tasks
  → executar as tasks
```

## Skills

| Skill | Entrada | Saída | O que faz |
|-------|---------|-------|-----------|
| `/spec-phase <spec.md> [--auto]` | spec.md vazio, com notas ou já uma spec | o mesmo spec.md, reescrito | Entrevista em até 6 rodadas, com score de ambiguidade e gate ≤ 0.20. Cada requisito sai com Atual, Alvo e Acceptance pass/fail. |
| `/spec-review <spec.md>` | spec.md | relatório (não edita) | Um agente revisor procura requisito sem acceptance, ambiguidade, contradição, dado incompleto e pendência aberta. Veredito: PRONTA ou REVISAR. |
| `/design <spec.md> [--auto]` | spec.md fechada | design.md na mesma pasta | Investiga o projeto, avalia os game patterns, pergunta só as gray areas de implementação e escreve o design numerado (§2.arquivo.N, §4.regra.N, D1, §8.check.N), com rastreio de cada requisito. |
| `/design-vs-spec <design.md> <spec.md>` | design.md + spec.md | relatório (não edita) | Um agente confere se o design cobre 100% da spec: itens COBERTO, PARCIAL, AUSENTE e CONTRADIZ, mais o escopo extra. |
| `/tasks <design.md>` | design.md fechado | tasks.md na mesma pasta | 1 task por passo de §7, com objetivo, ids do design entregues, passos `[código]`/`[Editor]`/`[verificar]`, pontos de atenção das regras, critério de pronto pass/fail e dependências. Sem arquitetura nova. |
| `/tasks-vs-design <tasks.md> <design.md>` | tasks.md + design.md | relatório (não edita) | Um agente confere se as tasks entregam exatamente o design: parte sem task, task que inventa arquitetura e ordem divergente. |

`--auto` pula as perguntas: Claude escolhe a opção recomendada e registra cada escolha como "auto" no Interview Log (spec) ou em §6 (design).

## Regras do flow

- **O QUÊ só muda na spec.** Se o design ou as tasks pedirem uma mudança de requisito, ela volta para `/spec-phase` antes de seguir.
- **As revisões não editam nada.** `/spec-review`, `/design-vs-spec` e `/tasks-vs-design` só reportam. No fim, oferecem corrigir junto com você ou registrar os achados em `docs/Backlog/`.
- **Nenhuma skill faz commit.**
- **Decisões ficam registradas no documento.** Na spec, no Interview Log, onde o que foi deduzido sem perguntar aparece como "derivado". No design, na tabela §6, com a origem de cada decisão (usuário, auto, regra do projeto, spec).
- **As regras de `.claude/rules/` já valem.** Nenhuma skill pergunta de novo o que elas decidem.

## Exemplo

[docs/specs/example-spec/](docs/specs/example-spec/) mostra o flow aplicado a um loadout por slots, com a spec entrevistada e revisada e o design.
