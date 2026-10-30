---
paths:
  - "**/StateMachine/**"
  - "**/States/**"
  - "**/*State*.cs"
---

# State Machine Patterns

## Motor genérico e agnóstico

`StateMachineComponent` (`Assets/Scripts/Modules/StateMachine/StateMachineComponent.cs`) é um Pushdown Automaton: uma pilha de `IState`, onde só o topo recebe `Update`/`PhysicsUpdate`. Ele não conhece o ator que o contém nem os estados concretos — é um módulo reutilizável entre Player, elites e bosses.

**Não se autodirige.** `StateMachineComponent` não implementa `Update`/`FixedUpdate` do Unity — expõe `Tick(deltaTime)` e `PhysicsTick(deltaTime)` públicos, e é o ator dono quem chama isso explicitamente, no ponto certo do próprio ciclo. Isso existe porque ordem importa: o `PlayerController`, por exemplo, precisa aplicar gravidade/pulo, **depois** deixar o estado atual resolver a velocidade horizontal, e só então chamar `CharacterController.Move()` — se a FSM rodasse sozinha no próprio `Update`, a ordem entre ela e o ator dependeria da Script Execution Order.

Os métodos públicos se chamam `Tick`/`PhysicsTick`, não `Update`/`FixedUpdate`: num `MonoBehaviour`, um método chamado `Update` com parâmetro gera erro de mensagem do Unity.

```csharp
// PlayerController.Update
if (!_controller.isGrounded) _velocity.y += Physics.gravity.y * Time.deltaTime;
// ...jump...

_states.Tick(Time.deltaTime); // o estado atual escreve _velocity.x/_velocity.z aqui

_controller.Move(_velocity * Time.deltaTime);
```

Estados são classes C# puras (não `MonoBehaviour`):

```csharp
public interface IState
{
    void Enter();
    void Exit();
    void Update(float deltaTime);
    void PhysicsUpdate(float deltaTime);
}
```

Estados concretos ficam perto de quem os usa (`Assets/Scripts/Actors/Player/States/`, `Assets/Scripts/Actors/Enemies/Elite/States/`), não dentro do módulo genérico.

## Três operações, três intenções diferentes

A pilha existe pra resolver um problema específico: **retomar exatamente o contexto anterior sem recalculá-lo**. Se a transição não precisa desse "retomar", não deveria empilhar.

| Operação | Quando usar | Efeito |
|---|---|---|
| `ChangeState` | Transição lateral entre estados **irmãos, mutuamente exclusivos** (ex: Idle ↔ Walk ↔ Running) | `Exit()` do topo atual + `Push` do novo. Não guarda histórico. |
| `PushState` | Interrupção que precisa **devolver o controle exato de onde parou** (ex: Aim sobre Walk, Stagger/hitstun sobre qualquer locomoção, Pause) | Empilha sem chamar `Exit()` no estado de baixo — ele fica pausado, não encerrado. |
| `PopState` | Fim da interrupção | Chama `Exit()` no topo e retoma o estado de baixo automaticamente (sem `Enter()` de novo — ele nunca saiu). |

### Exemplo — locomoção (irmãos, `ChangeState`)

```csharp
// dentro de WalkState.Update
if (_sprintAction.IsPressed() && _player.Velocity.sqrMagnitude > 0.01f)
{
    _player.States.ChangeState(new RunningState(_player));
    return;
}
```

### Exemplo — interrupção (`PushState`/`PopState`)

```csharp
// algo externo detecta hitstun
_player.States.PushState(new StaggerState(_player));

// StaggerState.Exit() é chamado quando o stagger acaba, e o estado
// de locomoção que estava pausado (Walk, Running, Idle) retoma sozinho
_player.States.PopState();
```

## Quem decide a transição

Cada `IState` conhece as próprias condições de saída e chama `ChangeState`/`PushState`/`PopState` diretamente — a lógica de transição vive dentro do estado, não em um dispatcher externo nem em eventos. O construtor do estado recebe as referências que ele precisa (ator dono, `StateMachineComponent`, outros módulos), seguindo o mesmo padrão de injeção via construtor já usado no projeto.

## Evento `StateChanged` é só pra fora

`StateMachineComponent` expõe `public event Action<string> StateChanged`, disparado a cada transição. Esse evento é para sistemas **externos à FSM** que precisam reagir sem acoplamento direto — eles não decidem a transição, só reagem a ela depois de acontecida. Nunca use esse evento para orquestrar transições dentro da própria FSM — isso é responsabilidade dos estados, via chamada direta aos métodos da pilha.

`stateName` é derivado do nome da classe do estado com o sufixo `State` removido (`IdleState` → `"Idle"`, `RunningState` → `"Running"`) — assim o nome do state do Animator não fica acoplado a um detalhe de implementação C#.

### Locomoção não usa o evento — usa blend contínuo

Idle/Move não são estados "discretos" do ponto de vista de animação: são pontos num espectro contínuo de direção de input. Por isso a locomoção (`AnimationComponent`, ver `Assets/Scripts/Modules/Animation/AnimationComponent.cs`) não escuta `StateChanged` — ela lê o input bruto todo frame e alimenta um Blend Tree 1D/2D do `Animator` direto via `SetFloat` (com o hash cacheado por `Animator.StringToHash`), onde Idle é o ponto de origem (posição 0) e correr pra frente/trás são os extremos. Isso desacopla completamente animação de locomoção do nome da classe do estado de física.

`StateChanged` continua reservado pra estados **não-blendáveis** — ações discretas tipo Jump, Stagger, Death — que futuramente vão sair do Blend Tree de locomoção e tocar um clipe único via `Animator.CrossFade(stateName, ...)`, voltando ao Blend Tree ao sair do estado.
