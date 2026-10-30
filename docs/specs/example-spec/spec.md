# Loadout por slots — Spec

**Criada:** 2026-10-29 | **Atualizada:** 2026-10-29 (respostas do /spec-review; "fechar o jogo" removido no /design)
**Ambiguidade:** 0.10 (gate: ≤ 0.20)
**Requisitos:** 10 fechados

## Objetivo

O Player passa a ter um loadout de até 2 armas de um catálogo, cujo custo em slots (Rifle 3, Shotgun de cano serrado 2, Pistola 1) cabe numa capacidade de 4 + bônus (máx. 6), validado em toda mudança, salvo entre sessões e mostrado no Player em cena.

## Background

O projeto é greenfield: não existe `Assets/`, nem armas, inventário ou Player. A feature nasce de notas sobre um sistema de loadout por slots no estilo de extraction shooter: 6 slots no total, rifles usam 3, pistolas 1, shotguns de cano serrado 2, o jogador usa 4 e um perk adiciona +1, totalizando 5. Na entrevista, os "6 slots" foram definidos como o teto absoluto da capacidade, e o perk como um bônus numérico de capacidade, não como um sistema de perks.

## Requisitos

1. **Catálogo e custo por categoria**: existe um catálogo de armas; cada arma tem um id único e exatamente uma categoria, e o custo em slots é o da categoria: Rifle = 3, Shotgun de cano serrado = 2, Pistola = 1. O catálogo desta feature tem 1 arma por categoria (3 armas).
   - Atual: não existem armas nem categorias.
   - Alvo: o catálogo tem 3 armas com ids distintos, uma de cada categoria, e o custo de cada arma vem da categoria.
   - Acceptance: um teste EditMode confere que o catálogo tem 3 armas com ids distintos e que elas custam 3 (Rifle), 2 (Shotgun) e 1 (Pistola).

2. **Capacidade e bônus**: a capacidade do loadout é `min(4 + bônus, 6)`. O bônus é um inteiro ≥ 0 e vale 0 quando ninguém o configura. Ele muda pelo Inspector do componente (antes e durante o Play Mode) e pela API. Pelas duas vias, um valor negativo é rejeitado e o bônus anterior é mantido.
   - Atual: não existe capacidade.
   - Alvo: com bônus 0, 1, 2 e 5, a capacidade é 4, 5, 6 e 6.
   - Acceptance: um teste EditMode confere que o bônus sem configuração é 0, que os quatro valores dão essas capacidades e que definir o bônus −1 pela API deixa o bônus anterior intacto. A via do Inspector é verificada no Req 8.

3. **Limite de armas**: o loadout tem 0, 1 ou 2 armas. "Mesma arma" é o mesmo id do catálogo, e duas cópias da mesma arma são permitidas.
   - Atual: não existe loadout.
   - Alvo: um loadout vazio é válido; 2 Pistolas iguais são válidas; uma 3ª arma nunca entra.
   - Acceptance: um teste EditMode confere que o loadout vazio é válido, que 2 Pistolas iguais entram e que uma 3ª Pistola é rejeitada mesmo com slots livres.

4. **Adicionar arma**: adicionar só tem sucesso se o loadout tiver menos de 2 armas e `slots usados + custo da arma ≤ capacidade`. Caso contrário, é rejeitado e o loadout não muda.
   - Atual: não existe.
   - Alvo: com capacidade 4, Rifle + Pistola (4) entra e Rifle + Shotgun (5) é rejeitado; com capacidade 5, Rifle + Shotgun entra.
   - Acceptance: um teste EditMode confere os três casos e confere que, após uma rejeição, as armas e os slots usados são os mesmos de antes.

5. **Remover arma**: remover uma arma presente sempre tem sucesso e tira uma única cópia dela. Remover uma arma que não está no loadout é rejeitado e o loadout não muda.
   - Atual: não existe.
   - Alvo: remover libera os slots de uma cópia da arma.
   - Acceptance: um teste EditMode confere que remover o Rifle de Rifle + Pistola deixa 1 slot usado, que remover uma Pistola de 2 Pistolas deixa 1 Pistola, e que remover uma Shotgun ausente não altera nada.

6. **Loadout inválido**: o loadout é inválido quando os slots usados passam da capacidade, o que só acontece quando o bônus cai. Nenhuma arma é removida sozinha. Um loadout inválido não é salvo e não atualiza as armas mostradas no Player até voltar a caber.
   - Atual: não existe.
   - Alvo: Rifle + Shotgun (5) com o bônus caindo de 1 para 0 fica inválido, com as 2 armas mantidas; remover a Shotgun deixa o loadout válido de novo.
   - Acceptance: um teste EditMode confere o estado inválido, as 2 armas mantidas e a volta a válido após a remoção. Um teste PlayMode confere que, nesse estado inválido, o save não muda e o Player continua mostrando as 2 armas.

7. **Mostrar no Player**: um Player mínimo e estático em cena mostra cada arma do loadout como um objeto. Cada objeto informa o id da arma que representa, e o visual pode ser o mesmo placeholder para todas. Sempre que o loadout é carregado ou muda e continua válido, os objetos mostrados batem com as armas do loadout. As armas não atiram.
   - Atual: não existe Player.
   - Alvo: o Player mostra exatamente 1 objeto por arma do loadout, cada um com o id da arma.
   - Acceptance: um teste PlayMode confere que adicionar uma Pistola cria 1 objeto que informa o id da Pistola, que removê-la deixa 0 objetos, e que logo após o load os objetos batem com o loadout carregado.

8. **Montagem via Inspector e código**: em Play Mode, o loadout é montado pelo Inspector do componente ou pela API pública usada pelos testes. O Inspector só dispara as operações de adicionar (Req 4) e remover (Req 5), uma de cada vez; trocar uma arma é remover e depois adicionar. As duas vias passam pela mesma validação dos Req 2, 3, 4 e 5. Não há controle in-game para o jogador.
   - Atual: não existe.
   - Alvo: uma ação no Inspector que quebre uma regra é rejeitada como seria pela API.
   - Acceptance: check manual em Play Mode, com estes passos. (a) Com capacidade 4 e um Rifle, adicionar uma Shotgun pelo Inspector é rejeitado e o loadout fica intacto. (b) Com 2 Pistolas, adicionar uma 3ª pelo Inspector é rejeitado. (c) Definir o bônus −1 pelo Inspector mantém o bônus anterior.

9. **Salvar**: toda mudança bem-sucedida que deixa o loadout válido salva o loadout na hora. Um loadout inválido nunca é salvo. O save sobrevive a sair e voltar ao Play Mode. O bônus não faz parte do save.
   - Atual: não existe persistência.
   - Alvo: o que foi salvo é o que é carregado na próxima sessão.
   - Acceptance: um teste PlayMode confere que, ao montar Rifle + Pistola e recarregar, o loadout carregado é Rifle + Pistola; e que, com Rifle + Shotgun salvo e o bônus caindo para 0, o save continua Rifle + Shotgun.

10. **Carregar**: ao iniciar, o Player carrega o loadout salvo e o valida com o bônus em vigor no início. Sem save, o loadout começa vazio. Um save inválido é descartado inteiro: o loadout começa vazio, o save é sobrescrito com o loadout vazio e 1 aviso vai para o log. Um save é inválido quando tem um id que não está no catálogo, mais de 2 armas, armas que não cabem na capacidade, ou quando não pode ser lido (corrompido ou num formato inesperado).
   - Atual: não existe.
   - Alvo: o loadout inicial é sempre válido, e o save sempre bate com ele.
   - Acceptance: um teste PlayMode confere que sem save o loadout é vazio; que um save com Rifle + Shotgun carregado com bônus 0 resulta em loadout vazio, save vazio e 1 aviso no log; e que um save com id fora do catálogo e um save ilegível dão o mesmo resultado.

## Boundaries

**Dentro do escopo:**
- Catálogo de armas com 1 arma por categoria.
- Regras e dados do loadout: categorias, custo, capacidade, bônus, limite de 2 armas, adicionar, remover, estado inválido.
- Persistência do loadout entre sessões, com o load validado.
- Player mínimo e estático em cena que mostra as armas do loadout como objetos que informam o id da arma.
- Montagem pelo Inspector (adicionar e remover) e pela API pública, com a mesma validação.
- Testes EditMode das regras, testes PlayMode do Player, do save e do load, e check manual do Inspector.

**Fora do escopo:**
- Munição, consumíveis e ferramentas — não são armas e teriam regras de slot próprias.
- Preço e economia de compra — feature separada.
- Sistema de perks (desbloquear, escolher, equipar perk) — aqui o perk é só o valor do bônus.
- Tela de loadout e qualquer UI — a montagem nesta feature é por Inspector e código.
- Atirar, recarregar e trocar a arma ativa — combate é outra feature; as armas aqui são só visuais.
- Pegar ou trocar arma no mundo durante a partida — o entregável escolhido foi o loadout, não o pickup.
- Controle in-game (teclas de debug) para montar o loadout — descartado em favor do Inspector.
- Movimento, câmera e controle do Player — o Player desta feature é estático.
- Visual distinto por arma ou categoria — a identidade é o id informado pelo objeto.
- Mais de uma arma por categoria no catálogo — o catálogo cresce em outra feature.
- Capacidade acima de 6 — 6 é o teto absoluto do sistema.
- Verificar o save num build (fechar e abrir o jogo) — sem Inspector nem controle in-game, não há como montar o loadout num build.

## Constraints

- O teto de 6 slots é fixo e não depende de nenhum bônus.
- As regras dos Req 1 a 6 (custo, capacidade, limite, adicionar, remover, validade) precisam ser testáveis em EditMode, sem cena. Os efeitos do Req 6 no save e no Player são verificados em PlayMode.
- Sem dependência de pacote novo para a persistência.

## Acceptance Criteria

- [ ] O catálogo tem 3 armas com ids distintos, que custam 3 (Rifle), 2 (Shotgun de cano serrado) e 1 (Pistola).
- [ ] O bônus sem configuração é 0; bônus 0, 1, 2 e 5 dão capacidade 4, 5, 6 e 6; bônus −1 pela API é rejeitado e o bônus anterior fica.
- [ ] Loadout vazio é válido; 2 Pistolas iguais entram; uma 3ª arma é rejeitada.
- [ ] Com capacidade 4, Rifle + Pistola entra e Rifle + Shotgun é rejeitado sem alterar o loadout; com capacidade 5, Rifle + Shotgun entra.
- [ ] Remover o Rifle de Rifle + Pistola deixa 1 slot usado; remover uma Pistola de 2 Pistolas deixa 1 Pistola; remover uma arma ausente não altera nada.
- [ ] Rifle + Shotgun com o bônus caindo de 1 para 0 fica inválido, mantém as 2 armas e volta a válido ao remover a Shotgun (EditMode); nesse estado, o save não muda e o Player continua mostrando as 2 armas (PlayMode).
- [ ] Adicionar uma Pistola cria 1 objeto com o id da Pistola, removê-la deixa 0 objetos, e logo após o load os objetos batem com o loadout carregado (PlayMode).
- [ ] Pelo Inspector, adicionar uma Shotgun a um Rifle com capacidade 4 é rejeitado, uma 3ª Pistola é rejeitada e o bônus −1 é rejeitado (check manual).
- [ ] Rifle + Pistola montado é o mesmo loadout depois de recarregar; com Rifle + Shotgun salvo e o bônus caindo para 0, o save continua Rifle + Shotgun (PlayMode).
- [ ] Sem save, o loadout começa vazio; um save que não cabe, com id fora do catálogo ou ilegível resulta em loadout vazio, save vazio e 1 aviso no log (PlayMode).

## Relatório de Ambiguidade

| Dimensão            | Nota | Mín   | Status | Notas |
|---------------------|------|-------|--------|-------|
| Goal Clarity        | 0.92 | 0.75  | ✓      | Entregável: catálogo + regras + save + Player visual |
| Boundary Clarity    | 0.90 | 0.70  | ✓      | 12 itens explícitos fora do escopo |
| Constraint Clarity  | 0.80 | 0.65  | ✓      | Formato do save fica para o design |
| Acceptance Criteria | 0.90 | 0.70  | ✓      | 10 critérios pass/fail, cada um com o tipo de verificação |
| **Ambiguidade**     | 0.10 | ≤0.20 | ✓      |       |

Status: ✓ = atingiu o mínimo, ⚠ = abaixo do mínimo (o design trata como suposição)

## Interview Log

| Rodada | Perspectiva | Resumo da pergunta | Decisão fechada |
|--------|-------------|--------------------|-----------------|
| 1 | Researcher | O que é "6 slots no total" ao lado de 4 utilizáveis? | 6 é o teto absoluto da capacidade |
| 1 | Researcher | Qual o entregável? | Regras + dados testáveis em EditMode (ampliado na rodada 3) |
| 1 | Researcher | Existe limite de quantidade de armas? | Máximo de 2 armas, além do orçamento de slots |
| 2 | Simplifier | O que é o perk? | Só um bônus de capacidade; sistema de perks fora |
| 2 | Simplifier | Custo por categoria ou por arma? | Por categoria |
| 2 | Simplifier | Duplicata da mesma arma? | Permitida |
| 3 | Boundary Keeper | Loadout com 0 ou 1 arma é válido? | Sim: 0, 1 ou 2 armas |
| 3 | Boundary Keeper | Valores do bônus? | Inteiro ≥ 0; capacidade = min(4 + bônus, 6) |
| 3 | Boundary Keeper | O que entra no escopo? | Salvar e equipar no Player entram; munição e preço ficam fora |
| 4 | Failure Analyst | O que "equipar no Player" entrega, sem Player no projeto? | Player mínimo mostra as armas como objetos, sem atirar |
| 4 | Failure Analyst | Load de save inválido? | Descarta tudo: loadout vazio + aviso no log |
| 4 | Failure Analyst | Bônus cai com loadout acima da capacidade? | Fica inválido até o ajuste; nada é removido sozinho; não salva |
| 5 | Seed Closer | Como montar o loadout sem UI? | Inspector + código/testes, com a mesma validação |
| 5 | Seed Closer | Loadout sem save? | Vazio |
| 5 | Seed Closer | Quando salva? | A cada mudança bem-sucedida que deixa o loadout válido |
| — | derivado | Bônus negativo | Rejeitado, mantém o anterior (confirmado na revisão: vale para Inspector e API) |
| — | derivado | Player com loadout inválido | Mantém as armas mostradas até voltar a válido (consequência de "não pode ser equipado") |
| — | derivado | Bônus no save | Não é salvo; o load valida com o bônus em vigor no início |
| R | spec-review | O que é uma arma? | Item de catálogo com id e categoria; 1 por categoria nesta feature |
| R | spec-review | Como provar que o objeto é "de Pistola"? (BLOCKER) | O objeto informa o id da arma; o visual pode ser igual para todas |
| R | spec-review | Remover com duplicata? | Tira uma cópia |
| R | spec-review | Que edições o Inspector permite? | Só adicionar e remover; trocar = remover + adicionar |
| R | spec-review | Origem e vias do bônus? | Default 0; Inspector + API, as duas rejeitam negativo |
| R | spec-review | O descarte apaga o save? | Sim: o save é sobrescrito com vazio |
| R | spec-review | Save ilegível? | Mesma regra do save inválido |
| R | spec-review | Player mínimo tem outra capacidade? | Não: é estático |
| R | spec-review | Como verificar os Req 7–10? | Testes PlayMode automáticos; Inspector com check manual |
| R | spec-review | O que do Req 6 é EditMode? | Só a regra; os efeitos no save e no Player vão para PlayMode |
| R | spec-review | "Fechar o jogo" no Req 9 | Coberto por check manual num build (revogado abaixo) |
| D | design | "Fechar o jogo" é verificável num build sem controle in-game? | Não: sai do Req 9 e vai para Fora do escopo |
| R | derivado | Player logo após o load | Mostra o loadout carregado (lacuna apontada nas melhorias) |
