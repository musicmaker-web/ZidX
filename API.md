# ZidX UI Library — API Reference

Complete reference for every function, element, and option in the ZidX library (`TESTEN.lua`).

## Contents

- [Loading the Library](#loading-the-library)
- [Window](#window-createwindow)
- [Tabs & Sections](#tabs--sections)
- [Basic Elements](#basic-elements)
- [Premium System](#premium-system)
- [Premium Elements](#premium-elements)
- [Notifications & Dialogs](#notifications--dialogs)
- [Config System](#config-system)
- [Asset System (custom icons)](#asset-system-custom-icons)
- [Themes](#themes)
- [Multi-Window Support](#multi-window-support)

---

## Loading the Library

```lua
local UILibrary = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/musicmaker-web/ZidX/refs/heads/main/ZidX-O.lua"
))()
```

---

## Window (`CreateWindow`)

```lua
local Window = UILibrary.CreateWindow({
    Theme = "Frost",                    -- optional, see Themes below
    Size = UDim2.new(0, 560, 0, 380),   -- optional, default window size
})
```

The window title is fixed to **"ZidX"** (rendered as the `ZidX.png` image, falls back to text) and **cannot** be overridden. The logo inside the round open/close button is likewise fixed to `Logo.png` (fallback: the letter "Z") — `options.Icon`/`options.LogoText` are not read anymore.

### Window methods

| Method | Description |
|---|---|
| `Window:CreateTab(name, icon)` | Creates a normal tab. `icon` is an optional emoji/character. |
| `Window:CreatePremiumTab(name, icon)` | Creates a tab with a "NEED KEY" overlay, Copy Link/Insert Key/Check Key at the top. |
| `Window:SetTheme(themeKey)` | Switches the theme live. |
| `Window:OnThemeChanged(fn)` | Callback fired on every theme switch. |
| `Window:GetAccent()` | Returns the current theme's accent color (`Color3`). |
| `Window:ToggleMenu()` | Opens/closes the window (same as clicking the round button). |
| `Window:CloseMenu()` | Closes the window. |
| `Window:ToggleMinimize()` | Minimizes/restores. |
| `Window:ToggleMaximize()` | Maximizes/restores. |
| `Window:SelectTab(tab)` | Selects a specific tab (tab object, not a name string). |
| `Window:SetSidebarCollapsed(bool)` | Collapses/expands the sidebar. |

**Example:**

```lua
local Window = UILibrary.CreateWindow({ Theme = "Frost" })

Window:OnThemeChanged(function()
    print("Theme changed to:", Window.CurrentTheme)
end)
```

### Window chrome controls

- **Minimize / Maximize / Close** — top right, use `Minimize.png` / `Maximize.png` / `Close.png`.
- **Resize toggle (R)**, **drag toggle for the open/close button (T)**, **drag toggle for the window (W)** — use `ResizeUI.png` / `Toggle-Drag.png` / `UI-Drag.png`.
- **Resize handle** at the bottom right (only visible while resize is enabled) — drags the window bigger/smaller, top-left corner stays fixed.
- **Round open/close button** — freely draggable (if its drag toggle is on), opens/closes the window on click.

All window settings (size, position, toggle states, theme, sidebar state) are saved/restored automatically with `SaveConfig`/`LoadConfig` — see [Config System](#config-system).

---

## Tabs & Sections

```lua
local MyTab = Window:CreateTab("Settings", "⚙️")
local MySection = MyTab:CreateSection("General")
```

| Method | Description |
|---|---|
| `Tab:CreateSection(title)` | Creates a normal section inside a tab. |
| `Tab:CreatePremiumSection(title)` | Same as `CreateSection`, but with a "NEED KEY" overlay over the whole section. |

Every section instance exposes all element methods listed in the next two chapters.

---

## Basic Elements

All elements are created via `Section:CreateX({ ... })` and optionally accept `Tooltip` (string), `TooltipDuration` (seconds), and `Flag` (string, used by the config system).

### `CreateLabel(text)`

```lua
Section:CreateLabel("Just a plain info line.")
```

### `CreateParagraph({ Title, Text })`

```lua
Section:CreateParagraph({
    Title = "About",
    Text = "Longer body text that wraps automatically.",
})
```

### `CreateSeparator(label)`

```lua
Section:CreateSeparator("Advanced")
```

### `CreateButton({ Text, Tooltip, Callback })`

```lua
Section:CreateButton({
    Text = "Click me",
    Tooltip = "Runs an example action.",
    Callback = function()
        print("Button pressed")
    end,
})
```

### `CreateToggle({ Text, Default, Flag, Tooltip, Callback })`

```lua
local MyToggle = Section:CreateToggle({
    Text = "Example toggle",
    Default = false,
    Flag = "MyToggle",
    Callback = function(state)
        print("Toggle is now:", state)
    end,
})

MyToggle:Set(true)
print(MyToggle:Get())
```

Returns: `{ Instance, SetCallback, Set(value), Get() }`

### `CreateSlider({ Text, Min, Max, Default, Increment, Suffix, Flag, Tooltip, Callback })`

```lua
local VolumeSlider = Section:CreateSlider({
    Text = "Volume",
    Min = 0, Max = 100, Default = 50, Suffix = "%",
    Callback = function(value)
        print("Volume:", value)
    end,
})

VolumeSlider:Set(75)
```

Returns: `{ Instance, Set(value), Get() }`

### `CreateDualSlider({ Text, Min, Max, DefaultMin, DefaultMax, Increment, Suffix, Flag, Tooltip, Callback })`

```lua
Section:CreateDualSlider({
    Text = "Price range",
    Min = 0, Max = 1000, DefaultMin = 100, DefaultMax = 800, Suffix = "$",
    Callback = function(lo, hi)
        print("Range:", lo, "-", hi)
    end,
})
```

### `CreateDropdown({ Text, Options, Default, Flag, Tooltip, Callback })`

```lua
Section:CreateDropdown({
    Text = "Select one",
    Options = { "Option A", "Option B", "Option C" },
    Default = "Option A",
    Callback = function(value)
        print("Selected:", value)
    end,
})
```

### `CreateMultiDropdown({ Text, Options, Default, Flag, Tooltip, Callback })`

```lua
Section:CreateMultiDropdown({
    Text = "Filter",
    Options = { "Red", "Green", "Blue" },
    Default = { "Red" },
    Callback = function(selected)
        print("Selected:", table.concat(selected, ", "))
    end,
})
```

### `CreateSearchDropdown({ Text, Options, Flag, Tooltip, Callback })`

Same as `CreateDropdown`, with a search box for filtering long option lists.

### `CreateSearchMultiDropdown({ Text, Options, Flag, Tooltip, Callback })`

Same as `CreateMultiDropdown`, with a search box.

All four dropdown variants return: `{ Instance, SetOptions(list), Get(), Set(value) }`

### `CreateTextbox({ Text, Placeholder, Default, Flag, Tooltip, Callback })`

```lua
Section:CreateTextbox({
    Text = "Name",
    Placeholder = "Type something...",
    Callback = function(text, enterPressed)
        print("Text:", text, "Enter pressed:", enterPressed)
    end,
})
```

### `CreateKeybind({ Text, Default, Flag, Tooltip, Callback })`

```lua
Section:CreateKeybind({
    Text = "Toggle key",
    Default = Enum.KeyCode.RightControl,
    Callback = function(key)
        print("Bound to:", key.Name)
    end,
})
```

### `CreateColorPicker({ Text, Default, Flag, Tooltip, Callback })`

```lua
Section:CreateColorPicker({
    Text = "Accent color",
    Default = Color3.fromRGB(60, 235, 255),
    Callback = function(color)
        print("Color:", color)
    end,
})
```

### `CreateProgressBar({ Text, Min, Max, Default, Tooltip })`

Display-only, no callback — update it externally.

```lua
local Bar = Section:CreateProgressBar({
    Text = "Loading",
    Min = 0, Max = 100, Default = 0,
})

Bar:Set(65)
```

---

## Premium System

Local or server-based unlocking — **no code is ever loaded/executed from a server** in either mode.

### `Library.Premium` methods

| Method | Description |
|---|---|
| `Library.Premium:SetKey(key)` | Sets exactly one valid key (local mode). |
| `Library.Premium:SetValidKeys({...})` | Sets multiple valid keys (local mode). |
| `Library.Premium:SetLink(url)` | The link copied by "Copy Link" (e.g. a gated link service). |
| `Library.Premium:SetVerifyEndpoint(url)` | Enables server mode — keys are checked against `<url>` instead of locally. |
| `Library.Premium:IsPremium()` | Returns `true`/`false`. |
| `Library.Premium:GetRemainingSeconds()` | Remaining validity of the active key, in seconds (server mode only). |
| `Library.Premium:OnPremiumActivated(fn)` | Callback fired on every status change (activated/deactivated). |
| `Library.Premium:Activate(key)` | Verifies and activates a key. Returns `true`/`false`. |
| `Library.Premium:Recheck()` | Re-verifies the last used key (refreshes remaining time). |
| `Library.Premium:Deactivate()` | Resets Premium back to locked. |
| `Library.Premium:CopyLink()` | Copies the link (plus own Roblox UserId as `&puid=`, if server mode is active). |
| `Library.Premium:CheckForKey()` | Asks the server once whether a key is ready for the current UserId. |

**Example:**

```lua
Library.Premium:OnPremiumActivated(function()
    if Library.Premium:IsPremium() then
        print("Premium unlocked, expires in", Library.Premium:GetRemainingSeconds(), "seconds")
    else
        print("Premium locked")
    end
end)

-- Manually activating a key the player typed in:
local success = Library.Premium:Activate(enteredKey)
```

Locally cached keys are stored in `ZidX/Key.txt` and re-verified automatically on every script start.

---

## Premium Elements

Every basic element (except pure display elements like Label/Paragraph/Separator/ProgressBar) has a Premium variant with the exact same options table as its base element, plus a lock icon (🔒/🔓, swaps both symbol **and** color once unlocked):

- `CreatePremiumButton`
- `CreatePremiumToggle`
- `CreatePremiumSlider`
- `CreatePremiumDualSlider`
- `CreatePremiumDropdown`
- `CreatePremiumMultiDropdown`
- `CreatePremiumSearchDropdown`
- `CreatePremiumSearchMultiDropdown`
- `CreatePremiumTextbox`
- `CreatePremiumKeybind`

**Example:**

```lua
local PremiumTab = Window:CreatePremiumTab("Premium", "💎")
local PremiumSection = PremiumTab:CreateSection("Premium features")

PremiumSection:CreatePremiumButton({
    Text = "Premium action",
    Callback = function()
        print("Only runs when Premium is unlocked")
    end,
})
```

**Important:** the lock is enforced inside each element's own click/drag logic (not just visibility) — even if an element is made visible/active from the outside (e.g. via a property explorer), nothing happens without a valid key. Elements only work normally once `Library.Premium:IsPremium()` is `true`.

`ColorPicker` and `ProgressBar` currently have no Premium variant.

---

## Notifications & Dialogs

### `Library:Notify({ Title, Content, Type, Duration })`

```lua
UILibrary:Notify({
    Title = "Saved",
    Content = "Your settings have been applied.",
    Type = "Success", -- "Info" | "Success" | "Warning" | "Error"
    Duration = 4,      -- seconds, default: 4
})
```

Shows a toast notification in the bottom right with a countdown progress bar and a close button (`Close.png`).

### `Library:CreateDialog({ Title, Content, Buttons })`

```lua
UILibrary:CreateDialog({
    Title = "Are you sure?",
    Content = "This action cannot be undone.",
    Buttons = {
        { Text = "Cancel" },
        { Text = "Confirm", Accent = true, Callback = function()
            print("Confirmed")
        end },
    },
})
```

### `Library:AttachTooltip(instance, text, duration)`

Manually attaches a tooltip info button to any GUI element.

```lua
Library:AttachTooltip(someButton, "Extra info about this button.")
```

---

## Config System

```lua
UILibrary:SaveConfig("my_config")
UILibrary:LoadConfig("my_config")
UILibrary:ListConfigs()
```

- Saves/loads **every** value registered with a `Flag` (Toggle, Slider, Dropdown, Textbox, Keybind, ColorPicker, ...).
- Also automatically saves/loads, **without needing a `Flag`**: window size & position, open/close button position, resize/drag toggle states, current theme, sidebar state.
- **Premium values are only applied on load if a valid key is already active at load time** — without an active key they're silently skipped (everything else still loads normally).
- Files are stored under `UILibraryConfigs/<name>.json`.

**Example:**

```lua
Section:CreateButton({
    Text = "Save config",
    Callback = function()
        UILibrary:SaveConfig("my_config")
    end,
})

Section:CreateButton({
    Text = "Load config",
    Callback = function()
        UILibrary:LoadConfig("my_config")
    end,
})
```

---

## Asset System (custom icons)

```lua
Library.AssetBaseUrl = "https://raw.githubusercontent.com/musicmaker-web/ZidX/refs/heads/main/"

Library.Assets = {
    Logo       = "Logo.png",        -- round logo inside the open/close button
    Title      = "ZidX.png",        -- title wordmark in the title bar
    Close      = "Close.png",       -- close button + notification close
    Maximize   = "Maximize.png",
    Minimize   = "Minimize.png",
    Resize     = "ResizeUI.png",    -- resize toggle (R)
    ToggleDrag = "Toggle-Drag.png", -- drag toggle for the open/close button (T)
    UIDrag     = "UI-Drag.png",     -- drag toggle for the window (W)
    ArrowLeft  = "Left.png",        -- collapse sidebar
    ArrowRight = "Right.png",       -- expand sidebar
}
```

- Images are downloaded on first run and cached locally under `ZidX/Assets/` (no re-download on later runs).
- Uses `getcustomasset()` — only works in executor environments that support it.
- **Never hard-fails**: if the download or `getcustomasset` doesn't work, the original text/emoji is used automatically (no crash, no blank button).
- For the sidebar arrows (`ArrowLeft`/`ArrowRight`), images are only used if **both** load successfully — otherwise both fall back to text arrows (`‹`/`›`), never a mix of image and text.

---

## Themes

Built-in themes (via `Theme = "..."` in `CreateWindow` or `Window:SetTheme(...)`) — see `Library.Themes` / `Library.ThemeOrder` in the code for the full, current list including each theme's `DisplayName`.

```lua
local themeOptions = {}
local nameToKey = {}
for _, key in ipairs(UILibrary.ThemeOrder) do
    local displayName = UILibrary.Themes[key].DisplayName
    table.insert(themeOptions, displayName)
    nameToKey[displayName] = key
end

Section:CreateDropdown({
    Text = "Theme",
    Options = themeOptions,
    Default = UILibrary.Themes.Frost.DisplayName,
    Callback = function(displayName)
        local key = nameToKey[displayName]
        if key then
            Window:SetTheme(key)
        end
    end,
})
```

---

## Multi-Window Support

`UILibrary.CreateWindow({...})` can be called more than once — each window runs fully independently (its own tabs, theme, and config flags with a unique internal prefix).

```lua
local WindowOne = UILibrary.CreateWindow({ Theme = "Frost" })
local WindowTwo = UILibrary.CreateWindow({ Theme = "Cyberwave" })
```
