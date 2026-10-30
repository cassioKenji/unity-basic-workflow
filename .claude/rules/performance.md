---
paths:
  - "**/*.cs"
---

# Performance Patterns

## Regra geral: zero alocações por frame

`Update`, `FixedUpdate` e `LateUpdate` não devem alocar heap. Toda coleção, buffer ou objeto temporário deve ser pré-alocado em `Awake` ou como campo da classe.

## Timers em vez de contadores de frame

Nunca use `Time.frameCount % 30 == 0` para espaçar lógica. Acumule `Time.deltaTime`.

```csharp
// ERRADO
private void Update()
{
    if (Time.frameCount % 30 == 0) UpdateAggro();
}

// CORRETO
private float _aggroTimer;
private const float AggroInterval = 0.5f;

private void Update()
{
    _aggroTimer += Time.deltaTime;
    if (_aggroTimer >= AggroInterval)
    {
        _aggroTimer = 0f;
        UpdateAggro();
    }
}
```

## Distância sem `sqrt`

Use `sqrMagnitude` para comparações de distância. Só use `Vector3.Distance`/`magnitude` quando o valor real for exibido ou usado em fórmulas que exijam a raiz.

```csharp
// ERRADO
if (Vector3.Distance(transform.position, target) < 5f) { }

// CORRETO
if ((transform.position - target).sqrMagnitude < 25f) { }
```

## Layers e triggers em vez de varreduras

Prefira configurar a Layer Collision Matrix corretamente e usar colliders `isTrigger` com `OnTriggerEnter`/`OnTriggerExit`. Evite `Physics.OverlapSphere` em `FixedUpdate`; se for inevitável, use `Physics.OverlapSphereNonAlloc` com buffer pré-alocado e `LayerMask`.

## Pooling de projéteis e efeitos

Projéteis, partículas e inimigos de spawn frequente devem usar object pool (`UnityEngine.Pool.ObjectPool<T>`). Nunca use `Instantiate`/`Destroy` nesses casos — reutilize objetos desativados.

## Visibilidade e culling

Desative a lógica de objetos fora de câmera usando `OnBecameVisible`/`OnBecameInvisible` (setando `enabled`). Inimigos fora de range não precisam rodar AI a cada frame.

## String em hot paths

Nunca use `Debug.Log`, string interpolation ou `name`/`tag` em `Update` em builds de release. Envolva em `#if UNITY_EDITOR || DEVELOPMENT_BUILD`. Para tags, use `CompareTag`.

```csharp
#if UNITY_EDITOR || DEVELOPMENT_BUILD
Debug.Log($"[Health] took {amount} damage");
#endif
```
