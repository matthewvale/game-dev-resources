# Unity C# Code Standards

The C# standards I use on my Unity game Space Nomads. Names such as `GameService`, `GameServices`, `ServiceLocator` and `RPSLib.Debug` come from my own framework, so swap in your own equivalents.

## File header

Every script starts with the author block, then usings, then the namespace:

```csharp
/// ------------------------------
/// Original Author: Matthew Vale
/// ------------------------------

using RPSCore;
using System.Collections.Generic;
using UnityEngine;

namespace GameCore
{
    public class Example : GameService
    {
    }
}
```

- Usings sorted alphabetically, no blank lines between them.
- Remove unused `using` directives.
- Game code lives in `namespace GameCore`. File-scoped namespaces are not used.
- Services derive from `GameService`; plain components from `MonoBehaviour`.

## Regions and ordering

Every member lives inside a region. Regions appear in this order, omitting any that are empty:

1. `#region Nested Types`
2. `#region Serialized Fields`
3. `#region Public Properties`
4. `#region Private Properties`
5. `#region Events`
6. `#region Unity Flow`
7. `#region Public Methods`
8. `#region Private Methods`
9. `#region <InterfaceName> Implementation` (one per interface, e.g. `#region IDragHandler Implementation`)
10. `#region Debug` (always the very last region)

Rules:
- Region names are Title Case (not `VARIABLES`, `SETUP`).
- One blank line after `#region` and before `#endregion`; one blank line between regions.
- Interface implementations always go in their own region, separate from other public methods.
- Nested classes, structs, enums and delegates go in `Nested Types`. Their own members do not need regions.
- Do not split large regions with suffixes (no `#region Public Methods - Dragging`). Keep one region per category.
- `Unity Flow` holds `Awake`, `OnEnable`, `Start`, `Update`, `OnDisable`, `OnDestroy` in lifecycle order. `GameService` subclasses keep the `Awake`/`OnDestroy` overrides calling `base`.

## Fields and properties

- Inspector fields: `[SerializeField] private` with `_camelCase` names. No public fields.
- Exception: save data classes (`ISaveData` types and the classes they contain, e.g. `ShipLoadout`, `SystemEconomyState`) keep public PascalCase fields. Newtonsoft writes the save file from public members and uses their names as keys, so making them private or renaming them breaks saves.
- Group inspector fields with `[Header("...")]`; explain non-obvious ones with `[Tooltip("...")]` on the line above.
- Attribute and field on the same line: `[SerializeField] private Button _saveButton;`
- Private state: `_camelCase`, `readonly` where possible, target-typed `new()` for collections.
- If a value could be a `const`, make it a `const`. `const` doesn't change naming: name it as you would any member with that access level, so `private const float _centralSearchFraction = 0.12f;` and `public const int MaxSockets = 32;`. Use `static readonly` only where `const` isn't allowed.
- Explicit defaults are written out: `private bool _isEditorOpen = false;`
- Public properties: PascalCase. Prefer expression-bodied read-only accessors (`public bool IsValid => _failures.Count == 0;`) or `{ get; private set; }`. Use a full `get { }` block only when it needs several lines.
- Expose collections as `IReadOnlyList<T>`.
- Events: `public event Action OnSomethingHappened;`

## Methods and naming

- PascalCase method names; British spelling (`Initialise`, `Visualiser`, `Randomise`).
- UI handlers named `On<Thing>Pressed` (e.g. `OnSavePressed`).
- Paired operations are named as pairs: `SuspendGameplay` / `RestoreGameplay`, `HideActivityFeed` / `RestoreActivityFeed`.

## Formatting

- Allman braces, 4-space indentation.
- Always use braces, even for single-line `if` bodies and early `return`/`continue`.
- Early returns for guard clauses, followed by a blank line.
- A blank line between a local variable declaration and the logic that uses it.
- Use `var` only when the type is visible in the same statement (`var builder = new StringBuilder(256);`). Otherwise write the explicit type (`ShipData shipData = PlayerShipProperties_Persistent.GetData();`).
- `foreach` and `for` loops are both fine.
- Keep LINQ to a minimum because it allocates garbage. Never use it in per-frame or hot paths; write the loop instead.
- Null-check serialized references before use (`if (_saveButton != null)`); use `?.` for one-line service calls.
- Long strings split with `+` onto an indented continuation line.

## Logging

Always through `RPSLib.Debug.Log`, prefixed with the class name and `::`, with an explicit style:

```csharp
RPSLib.Debug.Log($"ShipEditorLauncher :: '{shipData.ShipName}' has no loadout to edit.", RPSLib.Debug.Style.Warning);
```

Styles in use: `Warning`, `CriticalError`.

## Services

- Find services with `GameServices.X` or `ServiceLocator.Find<T>()`, and handle a null result.

## Comments

- Comments explain *why*, not *what* – especially non-obvious Unity behaviour, ordering constraints and bugs being avoided (e.g. "GivePlayerControl is not idempotent…").
- Put the comment directly above the method it explains. Trailing comments at the end of a line are fine for short notes about that line.
- `/// <summary>` only on non-obvious public or complex private methods.
- `// TODO` for known follow-ups.
