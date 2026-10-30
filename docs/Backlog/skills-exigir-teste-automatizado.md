# Backlog — Skills design/tasks: exigir teste automatizado

**Criado:** 2026-10-30 | **Status:** pendente

## Problema

Nenhuma skill do fluxo cobra teste automatizado:

- `.claude/skills/design/SKILL.md` (§8 Verificação): a coluna "Tipo" aceita `teste EditMode / PlayMode / cena / Profiler / build`. Um design só com checks manuais de cena passa sem aviso.
- `.claude/skills/tasks/SKILL.md`: o `[verificar]` aceita "testes no Test Runner **ou** check manual de §8". O critério de pronto mínimo é "compila + checks de §8 passam", sem dizer o tipo do check.

Hoje o fluxo é: design §8 (qualquer tipo) → task cita o check → pronto.

## Melhoria proposta

A regra entra no **design**, porque a skill tasks não pode inventar o que o design não lista.

1. **design**: toda `§4.regra` com lógica pura (sem depender de cena ou de feel) exige pelo menos um check em §8 do tipo `teste EditMode` ou `PlayMode`. Check manual fica para o que é visual, de feel ou de performance.
2. **design**: um novo item em "Conferir antes de entregar": toda regra de lógica pura tem um teste automatizado em §8.
3. **tasks**: um novo item no Passo 5: toda task que entrega `[código]` com uma regra de §4 tem um `[verificar]` com teste automatizado. Se faltar, é lacuna do design: avisar e sugerir `/design`.
4. **design-vs-spec / tasks-vs-design** (opcional): sinalizar uma regra de lógica sem teste automatizado como gap.

## Critério de pronto

- [ ] Um design com uma regra de lógica pura e só check manual é barrado pela conferência do `/design`.
- [ ] O `/tasks` avisa quando uma task de `[código]` com regra de §4 não tem teste automatizado.
