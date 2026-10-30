# Loadout por slots — Design

**Spec:** docs/specs/example-spec/spec.md (10 requisitos)
**Criado:** 2026-10-29 | **Atualizado:** 2026-10-29

## 1. Visão geral

Projeto greenfield: nada é reaproveitado. As regras do loadout ficam numa classe C# pura (`Loadout`), testável em EditMode sem cena, alimentada por dois ScriptableObjects: `LoadoutConfig` (custos e capacidades) e `WeaponCatalog` (as 3 `WeaponData`). O Player (ator) compõe três módulos autônomos:
- `LoadoutModule`: dono do `Loadout`, expõe a API e emite `ValidLoadoutChanged`;
- `LoadoutPersistenceModule`: lê e grava um JSON em `persistentDataPath`;
- `LoadoutViewModule`: instancia um placeholder `WeaponView` por arma.

O Player liga `ValidLoadoutChanged` → salvar e mostrar, e no `Start` faz ler o save → aplicar no loadout. Um Custom Editor do `LoadoutModule` dá ao Inspector os botões Adicionar/Remover e o campo de bônus, chamando a mesma API.

## 2. Mapa de arquivos

| Id | Arquivo | Ação | Namespace | Papel |
|----|---------|------|-----------|-------|
| §2.arquivo.1 | Assets/Scripts/UnityBasicWorkflow.asmdef | criar | — | Assembly runtime |
| §2.arquivo.2 | Assets/Scripts/Modules/Loadout/WeaponCategory.cs | criar | UnityBasicWorkflow.Modules.Loadout | Enum |
| §2.arquivo.3 | Assets/Scripts/Modules/Loadout/WeaponData.cs | criar | UnityBasicWorkflow.Modules.Loadout | SO |
| §2.arquivo.4 | Assets/Scripts/Modules/Loadout/WeaponCatalog.cs | criar | UnityBasicWorkflow.Modules.Loadout | SO |
| §2.arquivo.5 | Assets/Scripts/Modules/Loadout/LoadoutConfig.cs | criar | UnityBasicWorkflow.Modules.Loadout | SO |
| §2.arquivo.6 | Assets/Scripts/Modules/Loadout/Loadout.cs | criar | UnityBasicWorkflow.Modules.Loadout | Classe C# pura (regras) |
| §2.arquivo.7 | Assets/Scripts/Modules/Loadout/LoadoutModule.cs | criar | UnityBasicWorkflow.Modules.Loadout | Módulo |
| §2.arquivo.8 | Assets/Scripts/Modules/Loadout/Persistence/LoadoutSaveData.cs | criar | UnityBasicWorkflow.Modules.Loadout.Persistence | DTO serializável |
| §2.arquivo.9 | Assets/Scripts/Modules/Loadout/Persistence/LoadoutPersistenceModule.cs | criar | UnityBasicWorkflow.Modules.Loadout.Persistence | Módulo |
| §2.arquivo.10 | Assets/Scripts/Modules/Loadout/View/WeaponView.cs | criar | UnityBasicWorkflow.Modules.Loadout.View | Componente do placeholder |
| §2.arquivo.11 | Assets/Scripts/Modules/Loadout/View/LoadoutViewModule.cs | criar | UnityBasicWorkflow.Modules.Loadout.View | Módulo |
| §2.arquivo.12 | Assets/Scripts/Actors/Player/Player.cs | criar | UnityBasicWorkflow.Actors.Player | Ator |
| §2.arquivo.13 | Assets/Scripts/Editor/UnityBasicWorkflow.Editor.asmdef | criar | — | Assembly de Editor (só plataforma Editor) |
| §2.arquivo.14 | Assets/Scripts/Editor/Loadout/LoadoutModuleEditor.cs | criar | UnityBasicWorkflow.Editor.Loadout | Custom Editor |
| §2.arquivo.15 | Assets/Data/Weapons/Rifle.asset | criar | — | WeaponData (id `rifle`, Rifle) |
| §2.arquivo.16 | Assets/Data/Weapons/SawedOffShotgun.asset | criar | — | WeaponData (id `sawed_off_shotgun`, SawedOffShotgun) |
| §2.arquivo.17 | Assets/Data/Weapons/Pistol.asset | criar | — | WeaponData (id `pistol`, Pistol) |
| §2.arquivo.18 | Assets/Data/Weapons/WeaponCatalog.asset | criar | — | WeaponCatalog com §2.arquivo.15–17 |
| §2.arquivo.19 | Assets/Data/Loadout/LoadoutConfig.asset | criar | — | LoadoutConfig com os defaults de §3.LoadoutConfig |
| §2.arquivo.20 | Assets/Prefabs/Loadout/WeaponPlaceholder.prefab | criar | — | Cubo + WeaponView |
| §2.arquivo.21 | Assets/Prefabs/Player/Player.prefab | criar | — | Player + os 3 módulos, com config, catálogo e placeholder atribuídos |
| §2.arquivo.22 | Assets/Scenes/LoadoutSandbox.unity | criar | — | Cena com o Player.prefab, para o check manual do Inspector |
| §2.arquivo.23 | Assets/Tests/EditMode/UnityBasicWorkflow.Tests.EditMode.asmdef | criar | — | Assembly de teste EditMode |
| §2.arquivo.24 | Assets/Tests/EditMode/CatalogTests.cs | criar | UnityBasicWorkflow.Tests.EditMode | Teste EditMode |
| §2.arquivo.25 | Assets/Tests/EditMode/LoadoutTests.cs | criar | UnityBasicWorkflow.Tests.EditMode | Teste EditMode |
| §2.arquivo.26 | Assets/Tests/PlayMode/UnityBasicWorkflow.Tests.PlayMode.asmdef | criar | — | Assembly de teste PlayMode (Editor) |
| §2.arquivo.27 | Assets/Tests/PlayMode/PlayerLoadoutTests.cs | criar | UnityBasicWorkflow.Tests.PlayMode | Teste PlayMode |

## 3. Tipos

### §3.WeaponCategory
- Papel: enum — §2.arquivo.2
- Membros: `Rifle`, `SawedOffShotgun`, `Pistol`

### §3.WeaponData
- Papel: SO (`[CreateAssetMenu(menuName = "Data/Weapon")]`) — §2.arquivo.3
- Campos: `[SerializeField] private string _id`, `[SerializeField] private WeaponCategory _category`
- Membros públicos: `string Id { get; }`, `WeaponCategory Category { get; }`

### §3.WeaponCatalog
- Papel: SO (`[CreateAssetMenu(menuName = "Data/Weapon Catalog")]`) — §2.arquivo.4
- Campos: `[SerializeField] private List<WeaponData> _weapons`
- Membros públicos: `IReadOnlyList<WeaponData> Weapons { get; }`, `bool TryGet(string id, out WeaponData weapon)`

### §3.LoadoutConfig
- Papel: SO (`[CreateAssetMenu(menuName = "Data/Loadout Config")]`) — §2.arquivo.5
- Campos e defaults: `_baseCapacity = 4`, `_maxCapacity = 6`, `_maxWeapons = 2`, `_rifleCost = 3`, `_sawedOffShotgunCost = 2`, `_pistolCost = 1` (todos `[SerializeField] private int`)
- Membros públicos: `int BaseCapacity`, `int MaxCapacity`, `int MaxWeapons` (get), `int GetCost(WeaponCategory category)`

### §3.Loadout
- Papel: classe C# pura, `sealed` — §2.arquivo.6
- Membros públicos:
  - `Loadout(LoadoutConfig config)`
  - `IReadOnlyList<WeaponData> Weapons { get; }`
  - `int Bonus { get; }`, `int Capacity { get; }`, `int UsedSlots { get; }`, `bool IsValid { get; }`
  - `bool TryAdd(WeaponData weapon)`
  - `bool TryRemove(WeaponData weapon)`
  - `bool TrySetBonus(int bonus)`
  - `bool TryReplace(IReadOnlyList<WeaponData> weapons)`
  - `void Clear()`
- Eventos: nenhum. Quem notifica é o `LoadoutModule`.
- Depende de: `LoadoutConfig`, pelo construtor.

### §3.LoadoutModule
- Papel: Módulo — §2.arquivo.7
- Campos: `[SerializeField] private LoadoutConfig _config`, `[SerializeField] private WeaponCatalog _catalog`, `[SerializeField] private int _bonus` (default 0)
- Membros públicos:
  - `IReadOnlyList<WeaponData> Weapons`, `int Bonus`, `int Capacity`, `int UsedSlots`, `bool IsValid` (repassam do `Loadout`)
  - `WeaponCatalog Catalog { get; }` (usado pelo Custom Editor)
  - `bool TryAdd(WeaponData weapon)`, `bool TryRemove(WeaponData weapon)`, `bool TrySetBonus(int bonus)`
  - `void ApplySaved(IReadOnlyList<string> weaponIds)`: `null` significa save ilegível
- Eventos: `public event Action<IReadOnlyList<WeaponData>> ValidLoadoutChanged`
- Depende de: `LoadoutConfig` e `WeaponCatalog` via `[SerializeField]`. Cria o `Loadout` no `Awake` e aplica `_bonus` com `TrySetBonus`.

### §3.LoadoutSaveData
- Papel: DTO `[Serializable]` para `JsonUtility` — §2.arquivo.8
- Campos: `public List<string> WeaponIds`

### §3.LoadoutPersistenceModule
- Papel: Módulo — §2.arquivo.9
- Campos: `[SerializeField] private string _fileName = "loadout.json"`
- Membros públicos:
  - `string FileName { get; set; }`: os testes trocam o arquivo antes do `Start` do Player
  - `void Save(IReadOnlyList<WeaponData> weapons)`: grava os ids em `Path.Combine(Application.persistentDataPath, FileName)`
  - `bool TryRead(out List<string> weaponIds)`: `false` = não existe arquivo; `true` com `weaponIds == null` = arquivo ilegível
- Depende de: nada além de `WeaponData`.

### §3.WeaponView
- Papel: componente do prefab placeholder — §2.arquivo.10
- Membros públicos: `string WeaponId { get; }`, `void SetWeapon(WeaponData weapon)`
- O id fica visível no Inspector (campo serializado só-leitura no fluxo de jogo).

### §3.LoadoutViewModule
- Papel: Módulo — §2.arquivo.11
- Campos: `[SerializeField] private WeaponView _placeholderPrefab`, `[SerializeField] private Transform _anchor`
- Membros públicos: `IReadOnlyList<WeaponView> Views { get; }`, `void Show(IReadOnlyList<WeaponData> weapons)`
- Depende de: o prefab e a âncora via `[SerializeField]`.

### §3.Player
- Papel: Ator — §2.arquivo.12
- Membros públicos: `LoadoutModule Loadout { get; }` (usado pelos testes PlayMode)
- Depende de: `LoadoutModule`, `LoadoutPersistenceModule` e `LoadoutViewModule` via `GetComponent` no `Awake`.
- Liga em `OnEnable` e desliga em `OnDisable`: `_loadout.ValidLoadoutChanged += _persistence.Save` e `_loadout.ValidLoadoutChanged += _view.Show`.
- `Start`: `if (_persistence.TryRead(out List<string> ids)) _loadout.ApplySaved(ids);`

### §3.LoadoutModuleEditor
- Papel: Custom Editor `[CustomEditor(typeof(LoadoutModule))]`, herda `UnityEditor.Editor` (nome qualificado, por causa do namespace `...Editor.Loadout`) — §2.arquivo.14
- Desenha:
  - os campos `_config` e `_catalog`;
  - um IntField de bônus, que nunca usa o campo cru `_bonus`;
  - em Play Mode, um dropdown com `Catalog.Weapons` e os botões Adicionar/Remover;
  - só leitura: armas atuais, `UsedSlots/Capacity` e `IsValid`.

## 4. Regras

1. **§4.regra.1 Custo**: custo de uma arma = `config.GetCost(weapon.Category)`. `UsedSlots` = soma dos custos de `Weapons`.
2. **§4.regra.2 Capacidade**: `Capacity = Mathf.Min(BaseCapacity + Bonus, MaxCapacity)`.
3. **§4.regra.3 Bônus**: `TrySetBonus(v)` com `v < 0` → `false` e `Bonus` não muda. Com `v ≥ 0`: `Bonus = v` e `true`, mesmo que o loadout fique inválido. Borda: se o `_bonus` serializado for negativo (YAML editado à mão), o `Awake` rejeita e o bônus fica 0.
4. **§4.regra.4 Adicionar**: `TryAdd(w)` → `false` se `w == null`, se `Weapons.Count >= MaxWeapons` ou se `UsedSlots + custo(w) > Capacity`. Caso contrário, adiciona no fim e devolve `true`. Borda: num loadout inválido, sempre `false` (já passou da capacidade).
5. **§4.regra.5 Remover**: `TryRemove(w)` remove a primeira arma com `Id == w.Id` e devolve `true`. Sem nenhuma com esse id, devolve `false` e nada muda. Borda: com 2 Pistolas, sai 1.
6. **§4.regra.6 Validade**: `IsValid = UsedSlots <= Capacity && Weapons.Count <= MaxWeapons`. Só o §4.regra.3 consegue deixar o loadout inválido.
7. **§4.regra.7 Replace**: `TryReplace(list)` só aceita se `list` não tiver `null`, se `list.Count <= MaxWeapons` e se a soma dos custos for `<= Capacity`. Aceita → substitui tudo e devolve `true`. Rejeita → `false` e nada muda.
8. **§4.regra.8 Notificação**: depois de um `TryAdd`, `TryRemove` ou `TrySetBonus` do `LoadoutModule` bem-sucedido, `ValidLoadoutChanged(Weapons)` dispara se `IsValid`. Inválido → não dispara. Borda: remover uma arma de um loadout inválido que volta a caber dispara.
9. **§4.regra.9 ApplySaved**: `ApplySaved(ids)` é inválido se `ids == null`, se algum id não está no catálogo ou se `TryReplace` falha. Inválido → `Clear()`, exatamente 1 `Debug.LogWarning` e `ValidLoadoutChanged(vazio)`, o que salva o vazio pelo §4.regra.10. Válido → `ValidLoadoutChanged(Weapons)`.
10. **§4.regra.10 Save**: o save só acontece pelo `ValidLoadoutChanged`, então um loadout inválido nunca é salvo. O arquivo guarda só os ids, sem o bônus. Sem arquivo e sem mudança, nada é escrito.
11. **§4.regra.11 Leitura**: `TryRead` sem arquivo → `false`. Com arquivo: `JsonUtility.FromJson` lança exceção ou devolve `WeaponIds == null` → `true` com `weaponIds = null`. A exceção é capturada e vira o sinal "ilegível", e o aviso é logado no §4.regra.9, então nada é engolido em silêncio.
12. **§4.regra.12 View**: `Show(weapons)` destrói os `WeaponView` atuais e instancia 1 placeholder por arma, com `SetWeapon`. `Views` é atualizado na hora, sem esperar o `Destroy` do fim do frame. Borda: lista vazia → 0 views.
13. **§4.regra.13 Inspector**: o IntField de bônus chama `TrySetBonus` em Play Mode. Fora do Play Mode, grava `_bonus` só se o valor for `≥ 0`. Os botões Adicionar/Remover só aparecem em Play Mode e chamam `TryAdd`/`TryRemove` com a arma do dropdown.

## 5. Fluxos

**§5.fluxo.1 Início:** `Player.Awake` (pega os módulos) → `LoadoutModule.Awake` cria o `Loadout` e aplica `_bonus` → `Player.OnEnable` liga os eventos → `Player.Start` → `TryRead` → `false`: fica vazio, 0 views, nada salvo | `true`: `ApplySaved(ids)` → `ValidLoadoutChanged` → `Save` + `Show`

**§5.fluxo.2 Mudança:** Inspector ou API → `LoadoutModule.TryAdd`/`TryRemove`/`TrySetBonus` → `Loadout` aplica o §4.regra.3–5 → sucesso e `IsValid` → `ValidLoadoutChanged` → `LoadoutPersistenceModule.Save` + `LoadoutViewModule.Show`

**§5.fluxo.3 Bônus cai:** `TrySetBonus(0)` com Rifle + Shotgun → `true`, `IsValid == false` → sem evento → o save e as views ficam como estavam → `TryRemove(Shotgun)` → `IsValid == true` → evento → `Save` + `Show`

**§5.fluxo.4 Save inválido:** `TryRead` → `true` (ids inválidos ou `null`) → `ApplySaved` → `Clear` + 1 aviso → `ValidLoadoutChanged(vazio)` → `Save(vazio)` + `Show(vazio)`

## 6. Decisões e patterns

| Id | Problema | Escolha | Descartado (motivo) | Origem |
|----|----------|---------|---------------------|--------|
| D1 | Save e view reagirem às mudanças sem acoplar os módulos | Observer: `ValidLoadoutChanged`, ligado pelo Player em `OnEnable`/`OnDisable` | Módulo chamar o outro direto (acopla) | regra do projeto |
| D2 | Regras testáveis em EditMode sem cena (Constraint) | Classe C# pura `Loadout`, com o `LoadoutModule` como casca | Regras no MonoBehaviour (exigiria GameObject no teste) | spec |
| D3 | Onde salvar | Arquivo JSON (`JsonUtility`) em `persistentDataPath`, nome configurável | PlayerPrefs (no registro, difícil de corromper no teste) | usuário |
| D4 | Quem é dono do save | `LoadoutPersistenceModule` próprio, ligado pelo Player | Dentro do `LoadoutModule` (mistura regra e I/O) | usuário |
| D5 | Inspector disparar adicionar/remover | Custom Editor com dropdown e botões | `[ContextMenu]` (menos direto) | usuário |
| D6 | Montar o Player nos testes PlayMode | Instanciar `Player.prefab` via `AssetDatabase` | Montar em código (API só para o teste; não testa o prefab) | usuário |
| D7 | Visual da arma | Prefab placeholder único no `LoadoutViewModule` | Prefab por `WeaponData` (prepara um visual que está fora do escopo) | usuário |
| D8 | Namespace raiz | `UnityBasicWorkflow` | `ProjectName` (placeholder) | usuário |
| D9 | Custos e capacidades | `LoadoutConfig` SO com os defaults de §3.LoadoutConfig | Constantes no código | regra do projeto |
| D10 | Identidade da arma | `WeaponData.Id` (string); remover e o load comparam por id | Referência do asset (o save precisa de um id estável) | spec |
| D11 | Singleton/System | Não usado: há um Player só, e nada precisa de acesso global | `LoadoutSystem.Instance` (sem problema que justifique) | avaliação de patterns |
| D12 | Pooling | Não usado: `Instantiate`/`Destroy` nas views; a mudança é rara (edição manual) | `ObjectPool<WeaponView>` (pool só vale para spawn frequente) | regra do projeto |
| D13 | State | Não usado: "inválido" é um bool derivado, sem comportamento por estado | FSM Válido/Inválido (overengineering) | avaliação de patterns |
| D14 | Factory | Não usado: o catálogo SO já define as armas | `WeaponFactory` (sem variação de criação) | avaliação de patterns |
| D15 | Onde nasce o aviso do save ruim | O `LoadoutModule.ApplySaved` loga 1 aviso; a persistência só sinaliza `null` | A persistência logar (daria 2 avisos no save com id fora do catálogo) | spec (exatamente 1 aviso) |

## 7. Ordem de implementação

1. **§7.passo.1** Criar o asmdef runtime (§2.arquivo.1) — depende de: nada
2. **§7.passo.2** Criar `WeaponCategory`, `WeaponData`, `WeaponCatalog` e `LoadoutConfig`, e os assets §2.arquivo.15–19 — depende de: §7.passo.1
3. **§7.passo.3** Criar `Loadout` (§4.regra.1–7) — depende de: §7.passo.2
4. **§7.passo.4** Criar o asmdef EditMode, `CatalogTests` e `LoadoutTests` (§8.check.1–6) — depende de: §7.passo.3
5. **§7.passo.5** Criar `LoadoutModule` (§4.regra.3, 8, 9) — depende de: §7.passo.3
6. **§7.passo.6** Criar `LoadoutSaveData` e `LoadoutPersistenceModule` (§4.regra.10, 11) — depende de: §7.passo.2
7. **§7.passo.7** Criar `WeaponView`, `LoadoutViewModule` e `WeaponPlaceholder.prefab` (§4.regra.12) — depende de: §7.passo.2
8. **§7.passo.8** Criar `Player` e `Player.prefab` (§5.fluxo.1–4) — depende de: §7.passo.5, §7.passo.6, §7.passo.7
9. **§7.passo.9** Criar o asmdef de Editor e o `LoadoutModuleEditor` (§4.regra.13) — depende de: §7.passo.5
10. **§7.passo.10** Criar o asmdef PlayMode e `PlayerLoadoutTests` (§8.check.7, 8, 10, 12) — depende de: §7.passo.8
11. **§7.passo.11** Criar `LoadoutSandbox.unity` com o Player — depende de: §7.passo.8, §7.passo.9
12. **§7.passo.12** Executar o check manual §8.check.9 — depende de: §7.passo.11

## 8. Verificação

| Id | Como verificar | Tipo | Cobre |
|----|----------------|------|-------|
| §8.check.1 | Carregar `WeaponCatalog.asset` e `LoadoutConfig.asset`: 3 armas, ids distintos, `GetCost` = 3 (Rifle), 2 (SawedOffShotgun), 1 (Pistol) | teste EditMode | Req 1.acc |
| §8.check.2 | `new Loadout(config).Bonus == 0`; `TrySetBonus(0/1/2/5)` → `Capacity` 4/5/6/6; `TrySetBonus(-1)` → `false` e `Bonus` anterior | teste EditMode | Req 2.acc |
| §8.check.3 | Vazio → `IsValid`; `TryAdd(pistol)` ×2 → `true`; 3ª → `false` com `UsedSlots == 2` | teste EditMode | Req 3.acc |
| §8.check.4 | Capacidade 4: Rifle + Pistol → `true`; Rifle + Shotgun → `false`, com `Weapons` e `UsedSlots` iguais aos de antes; `TrySetBonus(1)`: Rifle + Shotgun → `true` | teste EditMode | Req 4.acc |
| §8.check.5 | Rifle + Pistol, `TryRemove(rifle)` → `UsedSlots == 1`; 2 Pistols, `TryRemove(pistol)` → 1 Pistol; `TryRemove(shotgun)` ausente → `false`, sem mudança | teste EditMode | Req 5.acc |
| §8.check.6 | Bônus 1, Rifle + Shotgun, `TrySetBonus(0)` → `IsValid == false` e `Weapons.Count == 2`; `TryRemove(shotgun)` → `IsValid == true` | teste EditMode | Req 6.acc (regra) |
| §8.check.7 | Player.prefab com arquivo de teste: bônus 1, Rifle + Shotgun, `TrySetBonus(0)` → o conteúdo do arquivo continua `[rifle, sawed_off_shotgun]` e `Views.Count == 2` | teste PlayMode | Req 6.acc (save e Player) |
| §8.check.8 | `TryAdd(pistol)` → `Views.Count == 1` e `Views[0].WeaponId == "pistol"`; `TryRemove(pistol)` → `Views.Count == 0`; com o arquivo `[rifle, pistol]` pré-escrito, logo após o `Start` os `Views` têm os ids `rifle` e `pistol` | teste PlayMode | Req 7.acc |
| §8.check.9 | Em Play Mode na `LoadoutSandbox`: (a) com um Rifle e bônus 0, Adicionar uma Shotgun → rejeitado, a lista não muda; (b) com 2 Pistols, Adicionar uma Pistol → rejeitado; (c) bônus −1 no IntField em Play Mode → o valor volta ao anterior; (d) bônus −1 no IntField fora do Play Mode (Edit Mode) → o valor volta ao anterior | check manual | Req 8.acc, Req 2 |
| §8.check.10 | Rifle + Pistol montado → destruir o Player e instanciar outro com o mesmo arquivo → `Weapons` = Rifle + Pistol; o caso do bônus caindo é coberto por §8.check.7 | teste PlayMode | Req 9.acc |
| §8.check.12 | Sem arquivo → `Weapons.Count == 0`; arquivo `[rifle, sawed_off_shotgun]` com bônus 0 → vazio, arquivo regravado vazio, `LogAssert.Expect(LogType.Warning, …)` 1 vez; o mesmo para `[machete]` e para um arquivo com `{{{` | teste PlayMode | Req 10.acc |

Os testes PlayMode trocam `FileName` para um arquivo `test_*.json` logo após o `Instantiate` (antes do `Start`) e apagam esse arquivo no `TearDown`.

A numeração salta de `§8.check.10` para `§8.check.12`: o `§8.check.11` (verificação em build) foi removido numa revisão anterior, e o id ficou vago de propósito para não remunerar os checks já citados em §9.

## 9. Rastreio

| Requisito da spec | Onde no design | Verificação |
|-------------------|----------------|-------------|
| Req 1 | §3.WeaponData, §3.WeaponCatalog, §3.LoadoutConfig, §4.regra.1, §2.arquivo.15–18, D9, D10 | §8.check.1 |
| Req 1.acc | §8.check.1 | §8.check.1 |
| Req 2 | §3.Loadout, §3.LoadoutModule, §4.regra.2, §4.regra.3, §4.regra.13 | §8.check.2, §8.check.9 |
| Req 2.acc | §8.check.2 (API); a via do Inspector fica no Req 8.acc | §8.check.2 |
| Req 3 | §4.regra.4, §4.regra.6, D10 | §8.check.3 |
| Req 3.acc | §8.check.3 | §8.check.3 |
| Req 4 | §4.regra.4 | §8.check.4 |
| Req 4.acc | §8.check.4 | §8.check.4 |
| Req 5 | §4.regra.5, D10 | §8.check.5 |
| Req 5.acc | §8.check.5 | §8.check.5 |
| Req 6 | §4.regra.3, §4.regra.6, §4.regra.8, §4.regra.10, §5.fluxo.3 | §8.check.6, §8.check.7 |
| Req 6.acc | §8.check.6 (regra), §8.check.7 (save e Player) | §8.check.6, §8.check.7 |
| Req 7 | §3.WeaponView, §3.LoadoutViewModule, §4.regra.12, §5.fluxo.1, §5.fluxo.2, D7 | §8.check.8 |
| Req 7.acc | §8.check.8 | §8.check.8 |
| Req 8 | §3.LoadoutModuleEditor, §4.regra.13, §5.fluxo.2, D5 | §8.check.9 |
| Req 8.acc | §8.check.9 | §8.check.9 |
| Req 9 | §3.LoadoutPersistenceModule, §3.LoadoutSaveData, §4.regra.10, §5.fluxo.2, D3, D4 | §8.check.7, §8.check.10 |
| Req 9.acc | §8.check.10, §8.check.7 | §8.check.7, §8.check.10 |
| Req 10 | §3.Player (`Start`), §4.regra.9, §4.regra.11, §5.fluxo.1, §5.fluxo.4, D15 | §8.check.12 |
| Req 10.acc | §8.check.12 | §8.check.12 |

## 10. Riscos e suposições

- `Destroy` é adiado para o fim do frame — mitigação: `Views` é a lista própria do módulo, atualizada na hora, e os testes contam `Views`, não os filhos do transform.
- `FileName` tem setter público usado só pelos testes, e precisa ser trocado antes do `Start` — mitigação: os testes trocam logo após o `Instantiate`, no mesmo frame, e apagam o arquivo no `TearDown`.
- A unicidade dos ids do catálogo é conferida só pelo §8.check.1, não em runtime — aceito: o catálogo é editado à mão e tem 3 itens.
- Os testes PlayMode usam `AssetDatabase` e só rodam no Editor, não num player build — aceito: verificar o save num build está fora do escopo da spec.
- Suposição: o Unity Test Framework (`com.unity.test-framework`) já vem no template do Unity 6. A constraint "sem pacote novo" vale para a persistência, e o `JsonUtility` é nativo.
- Suposição: o namespace `UnityBasicWorkflow.Editor.Loadout` deixa `Editor` ambíguo dentro do arquivo — mitigação: herdar de `UnityEditor.Editor` com o nome qualificado.
- A borda do §4.regra.3 (bônus negativo gravado à mão no YAML do asset) não tem check: a spec não pede esse caso, e a regra existe só como defesa — aceito, sem verificação.
