---
paths:
  - "**/*.cs"
---

# Architecture Patterns

## Módulos são autônomos

Um módulo é um `MonoBehaviour` que não conhece o ator que o contém. Ele expõe uma API pública e emite eventos C#. O ator conecta os módulos entre si.

```csharp
// ERRADO — módulo acessa o ator
public class HealthModule : MonoBehaviour
{
    private void Awake()
    {
        Player owner = GetComponent<Player>(); // acoplamento
    }
}

// CORRETO — módulo é agnóstico
public class HealthModule : MonoBehaviour
{
    public event Action Died;
    public event Action<float> Damaged;

    public void TakeDamage(float amount) { }
}
```

## Atores apenas compõem

Atores não implementam lógica de gameplay. Eles obtêm referências aos módulos no `Awake` e conectam eventos em `OnEnable`, desconectando em `OnDisable`.

```csharp
public class Zombie : MonoBehaviour
{
    private HealthModule _health;
    private NavigationModule _navigation;
    private TargetingModule _targeting;

    private void Awake()
    {
        _health = GetComponent<HealthModule>();
        _navigation = GetComponent<NavigationModule>();
        _targeting = GetComponent<TargetingModule>();
    }

    private void OnEnable()
    {
        _health.Died += OnDied;
        _targeting.TargetAcquired += _navigation.SetTarget;
    }

    private void OnDisable()
    {
        _health.Died -= OnDied;
        _targeting.TargetAcquired -= _navigation.SetTarget;
    }

    private void OnDied() { }
}
```

## Systems são singletons de cena ou injetados

Systems (WaveDirector, SpawnDirector, etc.) são acessados via singleton (`public static X Instance`) ou injetados por `[SerializeField]`. Nunca use `GameObject.Find`, `FindFirstObjectByType` ou `FindObjectsByType` para localizar um system dentro de um módulo.

## Sem herança entre atores

`Player`, `Zombie`, `Boss` não herdam uns dos outros. A reutilização acontece via módulos compartilhados.

## ScriptableObjects como dados

Configurações de gameplay (stats de inimigo, definição de onda, parâmetros de arma) devem ser `ScriptableObject` atribuídos via `[SerializeField]` — não constantes hardcoded no script.

```csharp
[CreateAssetMenu(menuName = "Data/Enemy")]
public class EnemyData : ScriptableObject { /* HP, speed, loot table, etc. */ }

[SerializeField] private EnemyData _data;
```

## Namespaces

Todo arquivo fonte declara seu namespace alinhado à pasta, prefixado pelo namespace raiz do projeto (`ProjectName` nos exemplos abaixo — substitua pelo nome do projeto). Unity 6 usa C# 9: use namespace em bloco, não file-scoped.

```csharp
// Assets/Scripts/Modules/Combat/Health/HealthModule.cs
namespace ProjectName.Modules.Combat.Health
{
}

// Assets/Scripts/Actors/Enemies/Zombie.cs
namespace ProjectName.Actors.Enemies
{
}

// Assets/Scripts/Systems/WaveDirector/WaveDirector.cs
namespace ProjectName.Systems.WaveDirector
{
}
```
