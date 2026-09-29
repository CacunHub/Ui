# Feral UI

A small, dark Roblox UI library with a sidebar, pages, sections and the usual controls, plus a config system (SaveManager) that saves and loads every setting.

- Toggles, buttons, labels, textboxes, sliders, dropdowns (single and multi-select) and keybinds
- Notifications and a changeable accent color
- Save, load, overwrite and delete configs, with autoload
- UI text is always English (Roblox auto-translation is turned off)

---

## Files

| File | What it is |
| --- | --- |
| `FeralLib.lua` | The UI library |
| `FeralSaveManager.lua` | Config saving/loading (needs `FeralLib.lua`) |
| `FeralExampleUI.lua` | Full example using everything |
| `FeralFull.lua` | Library + SaveManager + example in one file, nothing to host |

---

## Quick start

```lua
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/CacunHub/Ui/refs/heads/main/FeralLib.lua"))()

local Window = Library:CreateMain({
	Title = "My Hub",                  -- sidebar title
	Desc = "v1.0",                     -- shown next to the brand in the top bar
	Brand = "Feral",                   -- blue word in the top bar
	ToggleKey = Enum.KeyCode.RightShift,
})

local Main = Window:CreatePage("Main", "Main Tab")
local Section = Main:CreateSection("Combat")

Section:CreateToggle({ Title = "Auto Farm", Description = "Farms for you", Default = false }, function(state)
	print("Auto Farm:", state)
end)

Main:Select() -- open this page first
```

Press the `ToggleKey` (RightShift by default) to hide or show the window.

---

## Controls

Every control is created on a section and takes an options table plus a callback.

```lua
local Section = Page:CreateSection("Section name")
```

### Toggle
```lua
local toggle = Section:CreateToggle({ Title = "Speed", Description = "Optional text", Default = false }, function(state) end)
toggle:Set(true)
print(toggle:Get())
```

### Button
```lua
Section:CreateButton({ Title = "Teleport" }, function() end)
```

### Label
```lua
local label = Section:CreateLabel({ Title = "Status: idle" })
label:SetText("Status: running")
label:SetColor(Color3.fromRGB(0, 255, 0))
```

### Box (textbox)
```lua
local box = Section:CreateBox({ Title = "Webhook", Default = "", Placeholder = "Enter URL..." }, function(text) end)
```
The callback runs when the box loses focus.

### Slider
```lua
local slider = Section:CreateSlider({ Title = "Walk Speed", Min = 16, Max = 200, Default = 16, Decimals = 0 }, function(value) end)
```

### Dropdown
```lua
-- single choice
local sea = Section:CreateDropdown({ Title = "Sea", Options = { "First Sea", "Second Sea" }, Default = "First Sea" }, function(choice) end)

-- multi choice: callback gets an array of the selected names
local rarity = Section:CreateDropdown({
	Title = "Rarities",
	Options = { "Common", "Rare", "Legendary" },
	Default = { "Rare" },
	Multi = true,
}, function(list) end)

sea:Refresh({ "New", "Options" })   -- replace the option list (Refresh(list, true) keeps the current selection)
```
`Set(value)` changes the selection silently. `Set(value, true)` also fires the callback.

### Bind (keybind)
```lua
Section:CreateBind({ Title = "Aim Key", Default = Enum.KeyCode.E }, function(key) end)
```
The callback runs when the key is pressed. Click the bind to pick a new key: Escape cancels, Backspace clears it, mouse buttons work too.

---

## Methods

| Call | What it does |
| --- | --- |
| `Window:CreatePage(name, title)` | Adds a sidebar tab |
| `Page:CreateSection(name)` | Adds a section to a page |
| `Page:Select()` | Opens that page |
| `Window:Toggle()` | Shows or hides the window |
| `Window:SetToggleKey(key)` | Changes the hide/show key |
| `Window:Destroy()` | Removes the UI |
| `Library:CreateNoti({ Title, Desc, ShowTime })` | Shows a notification that closes itself |
| `Library:SetAccent(Color3)` | Changes the accent color |

Toggle, box, slider, dropdown and bind objects have `:Set()` and `:Get()`, so you can read or change them from your own code.

---

## Flags (ids for saving)

Every toggle, box, slider, dropdown and bind registers itself in `Library.Flags` so configs can save it. The id is:

- the `Flag` you give it: `Section:CreateToggle({ Title = "Speed", Flag = "SpeedHack" }, ...)`, or
- if you leave it out, `Page/Section/Title`, for example `Main/Speed/Speed`.

Give important controls a `Flag` yourself. If you later rename a title, an automatic id changes and old configs stop matching that control.

---

## Configs (SaveManager)

```lua
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/CacunHub/Ui/refs/heads/main/FeralSaveManager.lua"))()

SaveManager:SetLibrary(Library)
SaveManager:SetFolder("Feral/MyGame")       -- configs go in workspace/Feral/MyGame/settings
SaveManager:SetIgnoreIndexes({ "SpeedHack" }) -- optional: flags that should never be saved
SaveManager:BuildConfigSection(ConfigPage)  -- adds a "Configuration" section to a page

-- create all your other controls first, then:
SaveManager:LoadAutoloadConfig()
```

The Configuration section has:

- **Config Name** and **Create Config** to save the current settings under a new name
- **Config List** with **Load Config**, **Overwrite Config** and **Delete Config**
- **Refresh List**
- **Set As Autoload** to load a config automatically on start

Loading a config also runs each control's callback, so features switch on or off for real.

You can also call it from code: `SaveManager:Save("name")`, `SaveManager:Load("name")`, `SaveManager:Delete("name")`.

---

## Requirements and troubleshooting

- Saving needs an executor with file functions (`writefile`, `readfile`, `isfile`, `isfolder`, `makefolder`, `listfiles`, `delfile`).
- **"FeralLib is outdated (no Library.Flags)":** you are loading an old `FeralLib.lua`. Use the current one. GitHub raw links can serve an old copy for a few minutes after an upload.
- **Nothing happens when clicking a config button:** open the F9 console. Errors are also shown as notifications.
- Studio: use a ModuleScript and `require(path.to.FeralLib)` instead of `loadstring`. The SaveManager needs executor file functions, so it will not save in Studio.
- Prefer a single file with nothing to host? Use `FeralFull.lua`.
