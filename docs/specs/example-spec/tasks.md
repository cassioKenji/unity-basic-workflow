# Loadout por slots — Tasks

**Design:** docs/specs/example-spec/design.md
**Criado:** 2026-10-29 | **Atualizado:** 2026-10-29

## Índice

| Task | Título | Passo do design | Entrega | Depende de | Status |
|------|--------|-----------------|---------|------------|--------|
| T1 | Assembly runtime | §7.passo.1 | §2.arquivo.1 | — | pendente |
| T2 | Catálogo e config (dados) | §7.passo.2 | §2.arquivo.2–5, 15–19 | T1 | pendente |
| T3 | Regras do loadout | §7.passo.3 | §2.arquivo.6, §4.regra.1–7 | T2 | pendente |
| T4 | Testes EditMode | §7.passo.4 | §2.arquivo.23–25, §8.check.1–6 | T3 | pendente |
| T5 | LoadoutModule | §7.passo.5 | §2.arquivo.7, §4.regra.3, 8, 9 | T3 | pendente |
| T6 | Persistência | §7.passo.6 | §2.arquivo.8–9, §4.regra.10, 11 | T2 | pendente |
| T7 | View do loadout | §7.passo.7 | §2.arquivo.10–11, 20, §4.regra.12 | T2 | pendente |
| T8 | Player e prefab | §7.passo.8 | §2.arquivo.12, 21, §5.fluxo.1–4 | T5, T6, T7 | pendente |
| T9 | Custom Editor | §7.passo.9 | §2.arquivo.13–14, §4.regra.13 | T5 | pendente |
| T10 | Testes PlayMode | §7.passo.10 | §2.arquivo.26–27, §8.check.7, 8, 10, 12 | T8 | pendente |
| T11 | Cena de teste manual | §7.passo.11 | §2.arquivo.22 | T8, T9 | pendente |
| T12 | Check manual do Inspector | §7.passo.12 | §8.check.9 | T11 | pendente |

Status: pendente / em andamento / feito. Só quem executa a task muda o Status.

## T1 — Assembly runtime (§7.passo.1)

**Objetivo:** Existe um assembly definition próprio para o código runtime do projeto, para que os testes EditMode/PlayMode (T4, T10) possam referenciá-lo sem puxar o assembly padrão inteiro.

**Entrega:** §2.arquivo.1

**Passos:**
1. `[Editor]` Criar `Assets/Scripts/UnityBasicWorkflow.asmdef` pelo menu Assets > Create > Assembly Definition, nome `UnityBasicWorkflow`.

**Pontos de atenção:**
- Sem código ainda: esta task só cria o assembly vazio que T2–T9 vão popular.

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] O asmdef aparece em `Assets/Scripts/` e nenhum outro assembly do projeto referencia ele ainda (normal, nada o usa até T2).

**Depende de:** —

## T2 — Catálogo e config (dados) (§7.passo.2)

**Objetivo:** Existem os ScriptableObjects de dado (categoria, arma, catálogo e config) e os 5 assets preenchidos: 3 armas, o catálogo e a config do loadout.

**Entrega:** §2.arquivo.2, §2.arquivo.3, §2.arquivo.4, §2.arquivo.5, §2.arquivo.15, §2.arquivo.16, §2.arquivo.17, §2.arquivo.18, §2.arquivo.19, §3.WeaponCategory, §3.WeaponData, §3.WeaponCatalog, §3.LoadoutConfig

**Passos:**
1. `[código]` Criar `Assets/Scripts/Modules/Loadout/WeaponCategory.cs`: enum `Rifle`, `SawedOffShotgun`, `Pistol` (§3.WeaponCategory).
2. `[código]` Criar `Assets/Scripts/Modules/Loadout/WeaponData.cs`: SO com `_id` (string) e `_category` (WeaponCategory), propriedades `Id` e `Category` (§3.WeaponData).
3. `[código]` Criar `Assets/Scripts/Modules/Loadout/WeaponCatalog.cs`: SO com `_weapons` (List<WeaponData>), `Weapons` e `TryGet(string id, out WeaponData)` (§3.WeaponCatalog).
4. `[código]` Criar `Assets/Scripts/Modules/Loadout/LoadoutConfig.cs`: SO com os 6 campos e defaults de §3.LoadoutConfig, e `GetCost(WeaponCategory)` (§4.regra.1).
5. `[Editor]` Criar `Assets/Data/Weapons/Rifle.asset`, `SawedOffShotgun.asset` e `Pistol.asset` via Create > Data > Weapon, com os ids `rifle`, `sawed_off_shotgun`, `pistol` e a categoria correspondente (§2.arquivo.15–17).
6. `[Editor]` Criar `Assets/Data/Weapons/WeaponCatalog.asset` via Create > Data > Weapon Catalog, com as 3 armas acima em `_weapons` (§2.arquivo.18).
7. `[Editor]` Criar `Assets/Data/Loadout/LoadoutConfig.asset` via Create > Data > Loadout Config, com os defaults de §3.LoadoutConfig (§2.arquivo.19).

**Pontos de atenção:**
- Sem `var`: tipo explícito em todo campo e propriedade (regra csharp).
- Campos privados com prefixo `_`; `[SerializeField] private`, nunca `public` (regra csharp).
- Namespace em bloco `UnityBasicWorkflow.Modules.Loadout`, alinhado à pasta (regra architecture).
- D9: custos e capacidades ficam no SO, não em constante no código.
- D10: a identidade da arma é o `Id` (string), não a referência do asset.

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] Os 5 assets existem com os valores acima; abrir cada `.asset` no Inspector confirma os campos.

**Depende de:** T1

## T3 — Regras do loadout (§7.passo.3)

**Objetivo:** Existe a classe `Loadout`, pura em C#, com todas as regras de capacidade, adicionar, remover, bônus e validade, sem depender de nenhum GameObject.

**Entrega:** §2.arquivo.6, §3.Loadout, §4.regra.1, §4.regra.2, §4.regra.3, §4.regra.4, §4.regra.5, §4.regra.6, §4.regra.7

**Passos:**
1. `[código]` Criar `Assets/Scripts/Modules/Loadout/Loadout.cs`: classe `sealed`, construtor `Loadout(LoadoutConfig config)`, propriedades `Weapons`, `Bonus`, `Capacity`, `UsedSlots`, `IsValid`, e os métodos `TryAdd`, `TryRemove`, `TrySetBonus`, `TryReplace`, `Clear` (§3.Loadout).
2. `[código]` Implementar `Capacity` pelo §4.regra.2 (`Mathf.Min(BaseCapacity + Bonus, MaxCapacity)`).
3. `[código]` Implementar `TrySetBonus` pelo §4.regra.3 (rejeita negativo, mantém `Bonus` anterior).
4. `[código]` Implementar `TryAdd` pelo §4.regra.4 (null, `MaxWeapons`, capacidade).
5. `[código]` Implementar `TryRemove` pelo §4.regra.5 (remove a primeira com o mesmo `Id`).
6. `[código]` Implementar `IsValid` pelo §4.regra.6.
7. `[código]` Implementar `TryReplace` pelo §4.regra.7 (tudo ou nada).

**Pontos de atenção:**
- D2: esta classe é C# pura (sem `MonoBehaviour`), exatamente para ser testável em EditMode sem cena (Constraint da spec).
- Sem `var`, tipo explícito (regra csharp).
- Sem alocação por chamada: `TryAdd`/`TryRemove`/`TryReplace` não são hot path de frame, mas evite LINQ se uma lista simples resolver (regra performance, por hábito).

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] `Loadout` existe sem referenciar `UnityEngine.MonoBehaviour`.

**Depende de:** T2

## T4 — Testes EditMode (§7.passo.4)

**Objetivo:** Os testes EditMode do catálogo e das regras do loadout existem e passam, fechando os Req 1 a 6 (parte de regra).

**Entrega:** §2.arquivo.23, §2.arquivo.24, §2.arquivo.25, §8.check.1, §8.check.2, §8.check.3, §8.check.4, §8.check.5, §8.check.6

**Passos:**
1. `[Editor]` Criar `Assets/Tests/EditMode/UnityBasicWorkflow.Tests.EditMode.asmdef`, referenciando `UnityBasicWorkflow.asmdef` e `UnityEngine.TestRunner`/`UnityEditor.TestRunner`.
2. `[código]` Criar `Assets/Tests/EditMode/CatalogTests.cs`: carrega `WeaponCatalog.asset` e `LoadoutConfig.asset` e confere o §8.check.1.
3. `[código]` Criar `Assets/Tests/EditMode/LoadoutTests.cs`: um teste por §8.check.2 a §8.check.6.
4. `[verificar]` Rodar no Test Runner (EditMode): todos os testes verdes.

**Pontos de atenção:**
- Sem `var` nos testes também (regra csharp vale para todo `.cs`).
- §8.check.1 cita o risco de §10: a unicidade do id não é garantida em runtime, só testada contra o asset fixo — não é bug, é a suposição aceita.

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] §8.check.1: catálogo com 3 armas, ids distintos, custos 3/2/1.
- [ ] §8.check.2: bônus 0/1/2/5 → capacidade 4/5/6/6; bônus −1 rejeitado.
- [ ] §8.check.3: vazio válido; 2 Pistols entram; 3ª rejeitada.
- [ ] §8.check.4: Rifle+Pistol entra, Rifle+Shotgun rejeitado (cap. 4) e aceito (cap. 5).
- [ ] §8.check.5: remover Rifle, remover 1 de 2 Pistols, remover ausente sem efeito.
- [ ] §8.check.6: bônus cai → inválido com as 2 armas mantidas; remover 1 → válido de novo.

**Depende de:** T3

## T5 — LoadoutModule (§7.passo.5)

**Objetivo:** Existe o módulo `LoadoutModule` que expõe o `Loadout` como componente Unity, aplica o bônus serializado no `Awake` e notifica `ValidLoadoutChanged` nas mudanças válidas.

**Entrega:** §2.arquivo.7, §3.LoadoutModule, §4.regra.3 (borda do YAML), §4.regra.8, §4.regra.9

**Passos:**
1. `[código]` Criar `Assets/Scripts/Modules/Loadout/LoadoutModule.cs`: `MonoBehaviour` com `[SerializeField] private LoadoutConfig _config`, `_catalog`, `_bonus`, e o evento `public event Action<IReadOnlyList<WeaponData>> ValidLoadoutChanged` (§3.LoadoutModule).
2. `[código]` `Awake`: criar o `Loadout` com `_config`; se `_bonus < 0`, usar 0 (borda de §4.regra.3); senão `TrySetBonus(_bonus)`.
3. `[código]` Expor `Weapons`, `Bonus`, `Capacity`, `UsedSlots`, `IsValid` e `Catalog` repassando do `Loadout`.
4. `[código]` Implementar `TryAdd`, `TryRemove`, `TrySetBonus`: chamam o `Loadout` e, se `true` e `IsValid`, disparam `ValidLoadoutChanged(Weapons)` (§4.regra.8).
5. `[código]` Implementar `ApplySaved(IReadOnlyList<string> weaponIds)` pelo §4.regra.9: resolve cada id com `Catalog.TryGet(id, out WeaponData weapon)`; `null`, algum `TryGet` falhando (id fora do catálogo) ou `TryReplace` falhando → `Clear()` + 1 `Debug.LogWarning` + `ValidLoadoutChanged(vazio)`; senão `ValidLoadoutChanged(Weapons)`.

**Pontos de atenção:**
- D15: o aviso de save ruim nasce aqui, não na persistência — só 1 `Debug.LogWarning` por `ApplySaved` inválido.
- Cache de componentes no `Awake`, nunca em `Update` (regra csharp) — este módulo não tem `Update`.
- Sem `?.`/`??` em objetos Unity (regra csharp): não se aplica aqui, pois `_config`/`_catalog` são verificados por referência de SO, não por null-coalescing em runtime.
- Sem `var` (regra csharp).

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] `LoadoutModule` com bônus serializado negativo inicializa com `Bonus == 0` (cobre a borda do §4.regra.3; sem teste automatizado, ver §10).
- [ ] `TryAdd`/`TryRemove`/`TrySetBonus`/`ApplySaved` existem com as assinaturas de §3.LoadoutModule.

**Depende de:** T3

## T6 — Persistência (§7.passo.6)

**Objetivo:** Existe o módulo que grava e lê o loadout como JSON em `persistentDataPath`, distinguindo "sem arquivo" de "arquivo ilegível".

**Entrega:** §2.arquivo.8, §2.arquivo.9, §3.LoadoutSaveData, §3.LoadoutPersistenceModule, §4.regra.10, §4.regra.11

**Passos:**
1. `[código]` Criar `Assets/Scripts/Modules/Loadout/Persistence/LoadoutSaveData.cs`: `[Serializable] class` com `public List<string> WeaponIds` (§3.LoadoutSaveData).
2. `[código]` Criar `Assets/Scripts/Modules/Loadout/Persistence/LoadoutPersistenceModule.cs`: `MonoBehaviour` com `[SerializeField] private string _fileName = "loadout.json"` e a propriedade `FileName` (§3.LoadoutPersistenceModule).
3. `[código]` Implementar `Save(IReadOnlyList<WeaponData> weapons)`: monta `LoadoutSaveData` com os `Id` das armas e grava com `JsonUtility.ToJson` em `Path.Combine(Application.persistentDataPath, FileName)` (§4.regra.10).
4. `[código]` Implementar `TryRead(out List<string> weaponIds)`: arquivo ausente → `false`, `weaponIds = null`; arquivo presente → tentar `JsonUtility.FromJson`, capturando exceção; sucesso com `WeaponIds != null` → `true, weaponIds = WeaponIds`; exceção ou `WeaponIds == null` → `true, weaponIds = null` (§4.regra.11).

**Pontos de atenção:**
- D3: `JsonUtility`, sem pacote novo (Constraint da spec).
- D4: este módulo só faz I/O; não decide se o loadout é válido, não loga aviso (isso é do §4.regra.9 no T5).
- Namespace `UnityBasicWorkflow.Modules.Loadout.Persistence` (regra architecture).

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] `Save` grava um JSON com `WeaponIds`; `TryRead` devolve `false` sem arquivo, e `true` com `weaponIds = null` para um arquivo corrompido (verificado manualmente ou no T10).

**Depende de:** T2

## T7 — View do loadout (§7.passo.7)

**Objetivo:** Existe o placeholder visual de arma e o módulo que instancia 1 por arma do loadout, com o id visível.

**Entrega:** §2.arquivo.10, §2.arquivo.11, §2.arquivo.20, §3.WeaponView, §3.LoadoutViewModule, §4.regra.12

**Passos:**
1. `[código]` Criar `Assets/Scripts/Modules/Loadout/View/WeaponView.cs`: `MonoBehaviour` com `WeaponId` (get) e `SetWeapon(WeaponData weapon)` (§3.WeaponView).
2. `[código]` Criar `Assets/Scripts/Modules/Loadout/View/LoadoutViewModule.cs`: `MonoBehaviour` com `[SerializeField] private WeaponView _placeholderPrefab`, `_anchor`, a lista `Views` e `Show(IReadOnlyList<WeaponData> weapons)` (§3.LoadoutViewModule).
3. `[código]` Implementar `Show`: destrói os `WeaponView` atuais, instancia 1 `_placeholderPrefab` por arma em `_anchor`, chama `SetWeapon`, e atualiza `Views` na hora, sem esperar o `Destroy` (§4.regra.12).
4. `[Editor]` Criar um Cube como `Assets/Prefabs/Loadout/WeaponPlaceholder.prefab`, com `WeaponView` adicionado (§2.arquivo.20).

**Pontos de atenção:**
- D7: 1 prefab único para todas as armas; visual distinto por categoria está Fora do escopo da spec.
- §4.regra.12, borda: lista vazia → 0 views.
- Namespace `UnityBasicWorkflow.Modules.Loadout.View` (regra architecture).

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] `Show` com 2 armas resulta em `Views.Count == 2`, cada um com o `WeaponId` certo (verificado no T10).
- [ ] `Show` com lista vazia resulta em `Views.Count == 0` (§4.regra.12, borda; verificado no T10).

**Depende de:** T2

## T8 — Player e prefab (§7.passo.8)

**Objetivo:** Existe o ator `Player`, que compõe os 3 módulos, liga os eventos em `OnEnable`/`OnDisable` e aplica o save no `Start`. O prefab monta tudo com as referências atribuídas.

**Entrega:** §2.arquivo.12, §2.arquivo.21, §3.Player, §5.fluxo.1, §5.fluxo.2, §5.fluxo.3, §5.fluxo.4

**Passos:**
1. `[código]` Criar `Assets/Scripts/Actors/Player/Player.cs`: `MonoBehaviour` com a propriedade `Loadout` (get) (§3.Player).
2. `[código]` `Awake`: `GetComponent` de `LoadoutModule`, `LoadoutPersistenceModule` e `LoadoutViewModule`, cacheados em campos privados.
3. `[código]` `OnEnable`/`OnDisable`: ligar/desligar `_loadout.ValidLoadoutChanged += _persistence.Save` e `+= _view.Show`.
4. `[código]` `Start`: `if (_persistence.TryRead(out List<string> ids)) _loadout.ApplySaved(ids);` (§5.fluxo.1).
5. `[Editor]` Criar `Assets/Prefabs/Player/Player.prefab`: adicionar os 4 componentes (`Player`, `LoadoutModule`, `LoadoutPersistenceModule`, `LoadoutViewModule`) e atribuir `_config`, `_catalog`, `_placeholderPrefab` e `_anchor` (§2.arquivo.21).

**Pontos de atenção:**
- Módulo não conhece o ator; é o `Player` que pega as referências e liga os eventos (regra architecture — "Atores apenas compõem").
- Cache no `Awake`, nunca `GetComponent` em `Update` (regra csharp).
- §5.fluxo.3: a passagem do bônus de 1 para 0 com Rifle+Shotgun não dispara `ValidLoadoutChanged`, então nada se conecta além do já ligado em `OnEnable` — nenhum código extra aqui, é consequência de T3+T5.
- Namespace `UnityBasicWorkflow.Actors.Player` (regra architecture).

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] `Player.prefab` tem os 4 componentes com as referências de SO/prefab atribuídas (sem campo vazio no Inspector).

**Depende de:** T5, T6, T7

## T9 — Custom Editor (§7.passo.9)

**Objetivo:** O Inspector do `LoadoutModule` ganha um IntField de bônus e, em Play Mode, um dropdown com as armas do catálogo e os botões Adicionar/Remover, todos passando pela mesma validação da API.

**Entrega:** §2.arquivo.13, §2.arquivo.14, §3.LoadoutModuleEditor, §4.regra.13

**Passos:**
1. `[Editor]` Criar `Assets/Scripts/Editor/UnityBasicWorkflow.Editor.asmdef`: `Platforms: Editor`, referenciando `UnityBasicWorkflow.asmdef`.
2. `[código]` Criar `Assets/Scripts/Editor/Loadout/LoadoutModuleEditor.cs`: `[CustomEditor(typeof(LoadoutModule))] class LoadoutModuleEditor : UnityEditor.Editor` (nome qualificado) (§3.LoadoutModuleEditor).
3. `[código]` Desenhar `_config` e `_catalog` com os campos padrão do Inspector.
4. `[código]` Desenhar o IntField de bônus: em Play Mode chama `TrySetBonus`; fora do Play Mode grava `_bonus` só se `≥ 0` (§4.regra.13).
5. `[código]` Em Play Mode, desenhar um dropdown com `Catalog.Weapons` e os botões Adicionar/Remover, chamando `TryAdd`/`TryRemove` com a arma escolhida.
6. `[código]` Fora de Play Mode, esconder os botões; sempre mostrar, só leitura, as armas atuais, `UsedSlots/Capacity` e `IsValid`.

**Pontos de atenção:**
- D5: Custom Editor, não `[ContextMenu]`.
- §4.regra.13: o IntField nunca escreve direto em `_bonus` quando está em Play Mode — sempre via `TrySetBonus`.
- Assembly de Editor isolado do runtime (regra architecture, por extensão — Editor nunca entra em build de player).

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] O Inspector do `LoadoutModule` mostra os campos e o IntField de bônus fora do Play Mode, e o dropdown + botões aparecem ao entrar em Play Mode.

**Depende de:** T5

## T10 — Testes PlayMode (§7.passo.10)

**Objetivo:** Os testes PlayMode do Player (save, load e view) existem e passam, fechando os Req 6 (parte de save/Player), 7, 9 e 10.

**Entrega:** §2.arquivo.26, §2.arquivo.27, §8.check.7, §8.check.8, §8.check.10, §8.check.12

**Passos:**
1. `[Editor]` Criar `Assets/Tests/PlayMode/UnityBasicWorkflow.Tests.PlayMode.asmdef`, referenciando `UnityBasicWorkflow.asmdef`, `UnityEngine.TestRunner`/`UnityEditor.TestRunner`, e com acesso a `UnityEditor.AssetDatabase` (D6).
2. `[código]` Criar `Assets/Tests/PlayMode/PlayerLoadoutTests.cs`: no `SetUp`, instanciar `Player.prefab` via `AssetDatabase.LoadAssetAtPath` + `Object.Instantiate`, e trocar `FileName` (do `LoadoutPersistenceModule`) para um `test_*.json` único, antes do `Start` rodar (D6).
3. `[código]` Implementar o teste do §8.check.7: bônus 1, Rifle+Shotgun, `TrySetBonus(0)` → arquivo continua `[rifle, sawed_off_shotgun]`, `Views.Count == 2`.
4. `[código]` Implementar o teste do §8.check.8: `TryAdd(pistol)` → 1 view com `WeaponId == "pistol"`; `TryRemove(pistol)` → 0 views (cobre também a borda "lista vazia" do §4.regra.12); arquivo pré-escrito `[rifle, pistol]` → após `Start`, os `Views` têm esses 2 ids.
5. `[código]` Implementar o teste do §8.check.10: Rifle+Pistol montado → destruir o Player, instanciar outro com o mesmo arquivo → `Weapons` = Rifle+Pistol.
6. `[código]` Implementar o teste do §8.check.12: sem arquivo → vazio; arquivo `[rifle, sawed_off_shotgun]` com bônus 0 → vazio + arquivo regravado vazio + 1 `LogAssert.Expect(LogType.Warning, …)`; o mesmo para `[machete]` (id fora do catálogo) e para um arquivo com `{{{` (ilegível).
7. `[código]` No `TearDown`, apagar o arquivo `test_*.json` usado.

**Pontos de atenção:**
- D6: o prefab real é instanciado, não montado em código — testa a liga ação real dos módulos.
- §8.check.12 cobre 3 variações de save inválido na mesma task: vazio original, id fora do catálogo, JSON ilegível.
- §10: `Destroy` é adiado para o fim do frame, mas os testes contam `Views` (a lista do módulo), não os filhos do transform.

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] §8.check.7: bônus cai → save e view não mudam.
- [ ] §8.check.8: adicionar/remover atualiza a view; load aplica a view correta.
- [ ] §8.check.10: save sobrevive a recriar o Player.
- [ ] §8.check.12: sem save, id inválido e JSON ilegível resultam em loadout vazio + 1 aviso cada.

**Depende de:** T8

## T11 — Cena de teste manual (§7.passo.11)

**Objetivo:** Existe uma cena com o `Player.prefab` em que o check manual do Inspector (T12) pode ser executado.

**Entrega:** §2.arquivo.22

**Passos:**
1. `[Editor]` Criar `Assets/Scenes/LoadoutSandbox.unity` e arrastar `Player.prefab` para a cena.

**Pontos de atenção:**
- Esta cena serve só ao check manual do Req 8; não precisa entrar no Build Settings (decisão já tomada ao remover o check de build do design).

**Critério de pronto:**
- [ ] Unity compila sem erros.
- [ ] A cena abre no Editor e mostra o Player com o loadout vazio (sem save prévio).

**Depende de:** T8, T9

## T12 — Check manual do Inspector (§7.passo.12)

**Objetivo:** Confirmar manualmente que o Custom Editor rejeita exatamente o que a API rejeita.

**Entrega:** §8.check.9

**Passos:**
1. `[verificar]` Entrar em Play Mode na `LoadoutSandbox`. (a) Com um Rifle no loadout e bônus 0, clicar Adicionar numa Shotgun → rejeitado, lista não muda. (b) Com 2 Pistols no loadout, clicar Adicionar numa Pistol → rejeitado. (c) Digitar −1 no IntField de bônus em Play Mode → o campo volta ao valor anterior.
2. `[verificar]` Saindo do Play Mode, digitar −1 no IntField de bônus (Edit Mode) → o campo volta ao valor anterior.

**Pontos de atenção:**
- §8.check.9 cobre Req 8.acc (a, b, c) e Req 2 (d, fora do Play Mode) — veio da correção do `/design-vs-spec`.

**Critério de pronto:**
- [ ] §8.check.9 (a): Adicionar Shotgun com Rifle+bônus 0 é rejeitado.
- [ ] §8.check.9 (b): Adicionar 3ª Pistol é rejeitado.
- [ ] §8.check.9 (c): bônus −1 em Play Mode é rejeitado.
- [ ] §8.check.9 (d): bônus −1 fora do Play Mode é rejeitado.

**Depende de:** T11
