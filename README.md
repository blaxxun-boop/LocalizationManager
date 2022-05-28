# Localization Manager

Can be used to easily add localization for your mod in Valheim.

## How to add localization

Add localization strings to your mod. For each string that can be localized, you should add a unique localization string. This localization string should be unique between all mods as well, so I recommend to mark them with some prefix that indicates your mod.

### Merging the DLL into your mod

Download the LocalizationManager.dll from the release section to the right.
Including the DLL is best done via ILRepack (https://github.com/ravibpatel/ILRepack.Lib.MSBuild.Task). You can load this package (ILRepack.Lib.MSBuild.Task) from NuGet.

If you have installed ILRepack via NuGet, simply create a file named `ILRepack.targets` in your project and copy the following content into the file.

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <Target Name="ILRepacker" AfterTargets="Build">
        <ItemGroup>
            <InputAssemblies Include="$(TargetPath)" />
            <InputAssemblies Include="$(OutputPath)\LocalizationManager.dll" />
        </ItemGroup>
        <ILRepack Parallel="true" DebugInfo="true" Internalize="true" InputAssemblies="@(InputAssemblies)" OutputFile="$(TargetPath)" TargetKind="SameAsPrimaryAssembly" LibraryPath="$(OutputPath)" />
    </Target>
</Project>
```

Make sure to set the LocalizationManager.dll in your project to "Copy to output directory" in the properties of the DLL and to add a reference to it.
After that, simply add `using LocalizationManager;` to your mod and use the `Localizer` class, to localize your mod.

## Example project

This adds a Warlock Hat, using the ItemManager and then adds localization for it. You can also use placeholders, to dynamically fill values into your strings.

```csharp
using BepInEx;
using BepInEx.Configuration;
using ItemManager;
using LocalizationManager;

namespace LocalizationTest;

[BepInPlugin(ModGUID, ModName, ModVersion)]
public class LocalizationTest : BaseUnityPlugin
{
	private const string ModName = "LocalizationTest";
	private const string ModVersion = "1.0.0";
	private const string ModGUID = "org.bepinex.plugins.localizationtest";

	private static ConfigEntry<float> warlockHatSmokeScreenSizeIncrease = null!;
	private static ConfigEntry<int> warlockHatSmokeScreenBlockIncrease = null!;
	private static ConfigEntry<int> warlockHatSmokeScreenDurationIncrease = null!;

	public void Awake()
	{
		Localizer.Load(); // Use this to initialize the LocalizationManager

		warlockHatSmokeScreenSizeIncrease = Config.Bind("Odins Warlock Hat", "Smoke Screen Size Increase", 2f, new ConfigDescription("Radius increase for the smoke screen ability of the Dragon Staff while wearing the Warlock hat.", new AcceptableValueRange<float>(0f, 5f)));
		warlockHatSmokeScreenDurationIncrease = Config.Bind("Odins Warlock Hat", "Smoke Screen Duration Increase", 120, new ConfigDescription("Duration increase for the smoke screen ability of the Dragon Staff while wearing the Warlock hat in seconds.", new AcceptableValueRange<int>(0, 300)));
		warlockHatSmokeScreenBlockIncrease = Config.Bind("Odins Warlock Hat", "Smoke Screen Block Chance Increase", 25, new ConfigDescription("Projectile block chance increase for the smoke screen ability of the Dragon Staff while wearing the Warlock hat.", new AcceptableValueRange<int>(0, 100)));

		Localizer.AddPlaceholder("pp_odins_warlock_hat_description", "power", warlockHatSmokeScreenBlockIncrease); // This will replace the {power} placeholder in your localization string with the value from the warlockHatSmokeScreenBlockIncrease
		Localizer.AddPlaceholder("pp_odins_warlock_hat_description", "radius", warlockHatSmokeScreenSizeIncrease);
		Localizer.AddPlaceholder("pp_odins_warlock_hat_description", "duration", warlockHatSmokeScreenDurationIncrease, duration => (duration / 60f).ToString("0.#")); // There is another parameter you can use to change the representation of a value. This will convert the seconds from the config entry to minutes for the display string
		
		Item alchemyEquipment = new("potions", "Odins_Warlock_Hat");
		alchemyEquipment.Crafting.Add(CraftingTable.Workbench, 5);
		alchemyEquipment.MaximumRequiredStationLevel = 5;
		alchemyEquipment.RequiredItems.Add("LinenThread", 20);
		alchemyEquipment.RequiredItems.Add("SurtlingCore", 5);
		alchemyEquipment.RequiredUpgradeItems.Add("LinenThread", 10);
		alchemyEquipment.RequiredUpgradeItems.Add("SurtlingCore", 2);
	}
}
```

You can use either YAML or JSON to localize your mod. All localization files go into a `translations` folder inside of your project.

**German.yml**
```yaml
pp_odins_warlock_hat: "Odins Hexenhut"
pp_odins_warlock_hat_description: "Ein Hut wie ihn mächtige Hexen tragen. Wenn du diesen Hut trägst, wird deine Nebelwand von Odins Drachenstab erheblich verstärkt.\n\nBlockchance um {power}% erhöht\nRadius um {radius} erhöht\nDauer um {duration} Minuten erhöht"
```

**English.json**
```json
{
  "pp_odins_warlock_hat": "Odins Warlock Hat",
  "pp_odins_warlock_hat_description": "A hat for your warlock self. Wearing this item greatly amplifies the smoke screen ability of your Odins Dragon Staff.\n\nBlock chance increased by {power}%\nRadius increased by {radius}\nDuration increased by {duration} minutes"
}
```

If your users want to add additional localizations, they just have to create a file with the name `ModName.Language.yml` or `ModName.Language.json` anywhere inside of the Bepinex folder.
For example, to add a French translation to this example mod, a user could create a LocalizationTest.French.yml file inside of the config folder and add French translations there.
