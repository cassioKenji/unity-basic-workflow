---
paths:
  - "**/*.cs"
---

# C# Patterns

## Nunca use `var`

Sempre declare o tipo explícito.

```csharp
// ERRADO
var health = GetComponent<HealthModule>();

// CORRETO
HealthModule health = GetComponent<HealthModule>();
```

## Cache de componentes e referências

Nunca chame `GetComponent`, `GameObject.Find`, `FindFirstObjectByType` ou qualquer lookup dentro de `Update`, `FixedUpdate` ou `LateUpdate`.
Faça o cache em `Awake` em campos privados da classe (ou atribua via `[SerializeField]` no Inspector).

```csharp
// ERRADO
private void Update()
{
    GetComponent<HealthModule>().TakeDamage(1f);
}

// CORRETO
private HealthModule _health;

private void Awake()
{
    _health = GetComponent<HealthModule>();
}

private void Update()
{
    _health.TakeDamage(1f);
}
```

## Campos privados com prefixo `_`

Campos privados usam prefixo `_` + camelCase. Propriedades e métodos públicos usam PascalCase. Campos expostos ao Inspector usam `[SerializeField] private`, nunca `public`.

```csharp
[SerializeField] private float _maxSpeed = 5f;
private float _currentSpeed;
private HealthModule _health;

public float MaxSpeed => _maxSpeed;
public void TakeDamage(float amount) { }
```

## Sem `?.` e `??` em objetos Unity

`UnityEngine.Object` (MonoBehaviour, GameObject, ScriptableObject, etc.) sobrescreve `==` para detectar objetos destruídos. `?.` e `??` ignoram essa sobrescrita. Use comparação explícita.

```csharp
// ERRADO — passa mesmo se _target foi destruído
_target?.TakeDamage(1f);

// CORRETO
if (_target != null) _target.TakeDamage(1f);
```

## Sem alocações em hot paths

Dentro de `Update`, `FixedUpdate`, `LateUpdate` e callbacks de colisão/trigger: sem `new` de classes, sem LINQ, sem string interpolation, sem lambdas.

```csharp
// ERRADO — aloca a cada frame
private void Update()
{
    List<Enemy> nearby = _enemies.Where(e => e.IsAlive).ToList();
    Debug.Log($"Enemies nearby: {nearby.Count}");
}

// CORRETO — reutiliza coleção pré-alocada
private readonly List<Enemy> _nearbyCache = new List<Enemy>(32);

private void Update()
{
    _nearbyCache.Clear();
    foreach (Enemy e in _enemies)
    {
        if (e.IsAlive) _nearbyCache.Add(e);
    }
}
```

## Tipos numéricos

Use `float` para coordenadas, velocidades, tempo e valores de gameplay — é o que a API do Unity usa (`Time.deltaTime`, `Vector3`). Use `double` apenas quando precisão longa for necessária (ex: `Time.timeAsDouble` em sessões muito longas).
