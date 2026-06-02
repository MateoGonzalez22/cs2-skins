# Cómo actualizar desde el repo original de WeaponPaints

Este fork tiene cambios propios sobre el repo original (`Nereziel/cs2-WeaponPaints`):
- Comando `!skin <query>` con fuzzy matching (FuzzySharp)
- `CommandSkinSearch` en la configuración

Cuando el repo original publique una actualización importante (offsets, bug fixes, nueva versión de CSS) seguí estos pasos.

---

## 1. Configurar el upstream (solo la primera vez)

```bash
git remote add upstream https://github.com/Nereziel/cs2-WeaponPaints.git
git remote -v
# Debe aparecer tanto origin (tu fork) como upstream (el original)
```

---

## 2. Traer los cambios del upstream

```bash
git fetch upstream
```

---

## 3. Mergear en tu rama main

```bash
git checkout main
git merge upstream/main
```

---

## 4. Resolver conflictos (si los hay)

Los únicos archivos donde casi seguro va a haber conflicto son los que modificaste:

### `WeaponPaints.csproj`
Asegurate de que quede la línea de FuzzySharp junto a las otras dependencias:
```xml
<PackageReference Include="FuzzySharp" Version="2.0.2" />
```

### `Config.cs`
Asegurate de que quede el bloque `CommandSkinSearch` dentro de la clase `Additional`:
```csharp
[JsonPropertyName("CommandSkinSearch")]
public List<string> CommandSkinSearch { get; set; } = ["skin"];
```

### `Commands.cs`
Tres cosas que deben estar:

**Al inicio del archivo:**
```csharp
using FuzzySharp;
```

**Dentro de `RegisterCommands()`**, después del bloque de `CommandSkin`:
```csharp
_config.Additional.CommandSkinSearch.ForEach(c =>
{
    AddCommand($"css_{c}", "Search and apply skin by name", (player, info) =>
    {
        if (!Utility.IsPlayerValid(player)) return;
        OnCommandSkinSearch(player, info);
    });
});
```

**Al final de la clase**, el método `OnCommandSkinSearch` completo (ver el archivo actual).

Para resolver un conflicto en Git, abrí el archivo conflictuado, buscá los marcadores `<<<<<<<`, `=======`, `>>>>>>>` y dejá el contenido correcto combinando ambas versiones.

---

## 5. Marcar el conflicto como resuelto y commitear

```bash
git add WeaponPaints.csproj Config.cs Commands.cs
git commit -m "Merge upstream + keep !skin command"
```

---

## 6. Recompilar

```bash
dotnet restore
dotnet build -c Release
```

Si el upstream actualizó la versión de `CounterStrikeSharp.API` en el `.csproj`, el restore descarga la nueva versión automáticamente.

---

## 7. Desplegar al servidor

Copiá estos archivos a `addons/counterstrikesharp/plugins/WeaponPaints/`:

```
bin/Release/net8.0/WeaponPaints.dll
bin/Release/net8.0/FuzzySharp.dll   ← siempre incluirla, es propia de este fork
```

Si el upstream actualizó gamedata o archivos de `data/` (skins JSON, etc.), copialos también desde la carpeta `bin/Release/net8.0/`.

---

## Referencia rápida

| Situación | Qué hacer |
|---|---|
| Solo cambiaron offsets en `gamedata/` | Merge + rebuild + copiar DLL + gamedata |
| Actualizaron versión de CSS API | Merge + `dotnet restore` + rebuild + copiar DLL |
| Cambiaron `Commands.cs` o `Config.cs` | Merge con cuidado, verificar los 3 puntos del paso 4 |
| Agregaron nuevas features | Merge normal, conflictos poco probables |
