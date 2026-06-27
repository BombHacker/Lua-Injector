# BOMBA Lua API

This guide explains how to use the **BOMBA Lua API** with hooks, IL2CPP method calls, raw address calls, MemoryPatch, logs, and ImGui.

The current Lua style is:

```lua
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")
```

## 1. Main rule

In Lua, use the global table:

```lua
BOMBA
```

So you write:

```lua
BOMBA.Class(...)
BOMBA.HOOK(...)
BOMBA.call_original(...)
BOMBA.Function(...)
BOMBA.MemoryPatch(...)
```

---

## 2. Minimal test script

```lua
LOGD("Lua works")
return "OK"
```

Logging:

```lua
LOGD("int=%i float=%f ptr=%p text=%s", 123, 1.5, 0x1234, "hello")
```

---

## 3. Signature types

A signature always looks like this:

```lua
{
    ret = "void",
    args = {"ptr", "int"}
}
```

### Return types and argument types

| Lua type | Meaning |
|---|---|
| `"void"` | no return |
| `"bool"`, `"boolean"` | bool |
| `"int"`, `"int32"`, `"i32"` | int32 |
| `"uint"`, `"uint32"`, `"u32"` | uint32 |
| `"long"`, `"int64"`, `"i64"` | int64 |
| `"ulong"`, `"uint64"`, `"u64"` | uint64 |
| `"short"`, `"int16"`, `"i16"` | int16 |
| `"ushort"`, `"uint16"`, `"u16"` | uint16 |
| `"byte"`, `"uchar"`, `"uint8"`, `"u8"` | uint8 |
| `"char"`, `"sbyte"`, `"int8"`, `"i8"` | int8 |
| `"float"`, `"single"` | float |
| `"double"` | double |
| `"ptr"`, `"pointer"`, `"void*"`, `"object"`, `"userdata"`, `"this"` | pointer |
| `"string"`, `"String"`, `"MonoString"`, `"cs_string"`, `"il2cpp_string"` | MonoString |
| `"cstring"`, `"char*"`, `"const char*"` | C string |

### Signature examples

```lua
{ ret = "void",  args = {"ptr"} }
{ ret = "void",  args = {"ptr", "int"} }
{ ret = "int",   args = {"ptr"} }
{ ret = "float", args = {"ptr"} }
{ ret = "ptr",   args = {"ptr"} }
{ ret = "bool",  args = {"ptr", "int", "float", "ptr"} }
{ ret = "string", args = {"ptr"} }
```

---

# 4. Classes and methods

## 4.1 Class by namespace and class name

```lua
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")
```

With namespace:

```lua
local WalletModel = BOMBA.Class("SYBO.Subway.Core.ProfileData", "WalletModel")
```

## 4.2 Class from instance

```lua
local cls = BOMBA.Class(instance)
LOGD("class=%s", cls:str())
```

Or easier:

```lua
LOGD("class=%s", BOMBA.ClassName(instance))
LOGD("class=%s", BOMBA.ObjectClassName(instance))
```

---

# 5. GetMethod

```lua
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")
local FixedUpdate = PlayerPhysics:GetMethod("FixedUpdate", 0)
```

Method with one argument:

```lua
local SetBodyType = PlayerPhysics:GetMethod("SetBodyType", 1)
```

Method with multiple arguments:

```lua
local SomeMethod = PlayerPhysics:GetMethod("SomeMethod", 4)
```

---

# 6. Cast and Call

## 6.1 Non-static method: `void(instance, int)`

C++ equivalent:

```cpp
void (*SetBodyType_NotStatic)(void *instance, int type);
SetBodyType_NotStatic(instance, 2);
```

Lua:

```lua
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")

local SetBodyType = PlayerPhysics:GetMethod("SetBodyType", 1)
    :cast({
        ret = "void",
        args = {"ptr", "int"}
    })

SetBodyType:Call(instance, 2)
```

## 6.2 Static method: `void(int)`

C++ equivalent:

```cpp
void (*SetBodyType_Static)(int type);
SetBodyType_Static(2);
```

Lua:

```lua
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")

local SetBodyTypeStatic = PlayerPhysics:GetMethod("SetBodyTypeStatic", 1)
    :cast({
        ret = "void",
        args = {"int"}
    })

SetBodyTypeStatic:CallStatic(2)
```

## 6.3 Return `float`

C++ equivalent:

```cpp
float (*get_TrueSpeedMethod)(void *instance);
float value = get_TrueSpeedMethod(instance);
```

Lua:

```lua
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")

local get_TrueSpeed = PlayerPhysics:GetMethod("get_TrueSpeed", 0)
    :cast({
        ret = "float",
        args = {"ptr"}
    })

local speed = get_TrueSpeed:Call(instance)
LOGD("speed=%f", speed)
```

## 6.4 Return pointer

C++ equivalent:

```cpp
void *(*get_cloths)(void *instance);
void *cloths = get_cloths(instance);
```

Lua:

```lua
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")

local get_cloths = PlayerPhysics:GetMethod("get_cloths", 0)
    :cast({
        ret = "ptr",
        args = {"ptr"}
    })

local cloths = get_cloths:Call(instance)

LOGD("cloths ptr=%p", cloths)
LOGD("cloths class=%s", BOMBA.ClassName(cloths))
```

## 6.5 Return `int`

```lua
local method = SomeClass:GetMethod("GetValue", 0)
    :cast({
        ret = "int",
        args = {"ptr"}
    })

local value = method:Call(instance)
LOGD("value=%i", value)
```

## 6.6 Return `bool`

```lua
local method = SomeClass:GetMethod("IsReady", 0)
    :cast({
        ret = "bool",
        args = {"ptr"}
    })

local ready = method:Call(instance)
LOGD("ready=%s", tostring(ready))
```

## 6.7 Return MonoString

```lua
local method = SomeClass:GetMethod("GetName", 0)
    :cast({
        ret = "string",
        args = {"ptr"}
    })

local name = method:Call(instance)
LOGD("name=%s", name)
```

## 6.8 MonoString argument

```lua
local setName = SomeClass:GetMethod("SetName", 1)
    :cast({
        ret = "void",
        args = {"ptr", "string"}
    })

setName:Call(instance, "HELLO WORLD")
```

Or manually:

```lua
local mono = BOMBA.CreateMonoString("HELLO WORLD")
setName:Call(instance, mono)
```

---

# 7. Hooking a method with `BOMBA.HOOK(method, sig, fn)`

## 7.1 Hook `FixedUpdate(void instance)`

```lua
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")

function PlayerPhysics_FixedUpdate(instance)
    LOGD("FixedUpdate instance=%p", instance)

    return BOMBA.call_original(instance)
end

local FixedUpdate = PlayerPhysics:GetMethod("FixedUpdate", 0)

BOMBA.HOOK(FixedUpdate, {
    ret = "void",
    args = {"ptr"}
}, PlayerPhysics_FixedUpdate)

return "hook registered"
```

## 7.2 Hook and call another method inside it

```lua
InfiniteCurrencies = true

local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")

function PlayerPhysics_FixedUpdate(instance)
    PlayerPhysics:GetMethod("SetBodyType", 1)
        :cast({
            ret = "void",
            args = {"ptr", "int"}
        })
        :Call(instance, 1)

    return BOMBA.call_original(instance)
end

local FixedUpdate = PlayerPhysics:GetMethod("FixedUpdate", 0)

BOMBA.HOOK(FixedUpdate, {
    ret = "void",
    args = {"ptr"}
}, PlayerPhysics_FixedUpdate)

return " "
```

## 7.3 Hook with `int` return

```lua
InfiniteCurrencies = true

local WalletModel = BOMBA.Class("SYBO.Subway.Core.ProfileData", "WalletModel")

function WalletModel_GetCurrency(instance, currencyType)
    if InfiniteCurrencies then
        return 999999
    end

    return BOMBA.call_original(instance, currencyType)
end

local GetCurrency = WalletModel:GetMethod("GetCurrency", 1)

BOMBA.HOOK(GetCurrency, {
    ret = "int",
    args = {"ptr", "int"}
}, WalletModel_GetCurrency)

return "currency hook registered"
```

## 7.4 Hook with `bool` return

```lua
function CanShoot(instance)
    LOGD("CanShoot instance=%p", instance)
    return true
end

local method = BOMBA.Class("", "PlayerPhysics"):GetMethod("get_canShoot", 0)

BOMBA.HOOK(method, {
    ret = "bool",
    args = {"ptr"}
}, CanShoot)
```

## 7.5 Hook with `float` return

```lua
function GetSpeed(instance)
    local original = BOMBA.call_original(instance)
    LOGD("original speed=%f", original)
    return original * 2.0
end

local method = BOMBA.Class("", "PlayerPhysics"):GetMethod("get_TrueSpeed", 0)

BOMBA.HOOK(method, {
    ret = "float",
    args = {"ptr"}
}, GetSpeed)
```

## 7.6 Hook with hidden `MethodInfo*` argument

If the hook crashes on a real phone, sometimes the function has an extra pointer argument:

```lua
function FixedUpdate(instance, methodInfo)
    LOGD("instance=%p methodInfo=%p", instance, methodInfo)
    return BOMBA.call_original(instance, methodInfo)
end

BOMBA.HOOK("libil2cpp.so", 0x123456, {
    ret = "void",
    args = {"ptr", "ptr"}
}, FixedUpdate)
```

---

# 8. Hook by address / offset

## 8.1 Hook by `lib + offset`

```lua
function PlayerPhysics_FixedUpdate(instance)
    LOGD("AWDAWD instance=%p", instance)
    return BOMBA.call_original(instance)
end

local id = BOMBA.HOOK("libil2cpp.so", 0x21c8ee4, {
    ret = "void",
    args = {"ptr"}
}, PlayerPhysics_FixedUpdate)

LOGD("hook id=%i", id)

return "queued hook"
```

## 8.2 Hook by resolved address

```lua
local addr = getLibraryAddress("libil2cpp.so", 0x21c8ee4)

function MyHook(instance)
    LOGD("hit instance=%p", instance)
    return BOMBA.call_original(instance)
end

BOMBA.HOOK(addr, {
    ret = "void",
    args = {"ptr"}
}, MyHook)
```

## 8.3 Check address

```lua
local info = BOMBA.CheckAddress("libil2cpp.so", 0x21c8ee4)

LOGD("ok=%s addr=%p perms=%s reason=%s",
    tostring(info.ok),
    info.address,
    info.perms,
    info.reason
)
```

---

# 9. Raw function pointer calls

## 9.1 C++ function pointer equivalent

C++:

```cpp
void (*SetBodyType_NotStatic)(void *instance, int type);
SetBodyType_NotStatic(instance, 2);
```

Lua:

```lua
local SetBodyType_NotStatic = BOMBA.Function("libil2cpp.so", 0x21c70e0, {
    ret = "void",
    args = {"ptr", "int"}
})

SetBodyType_NotStatic(instance, 2)
```

## 9.2 Static function

C++:

```cpp
void (*SetBodyType_Static)(int type);
SetBodyType_Static(2);
```

Lua:

```lua
local SetBodyType_Static = BOMBA.Function("libil2cpp.so", 0x21bf814, {
    ret = "void",
    args = {"int"}
})

SetBodyType_Static(2)
```

## 9.3 Raw return `float`

```lua
local get_TrueSpeedMethod = BOMBA.Function("libil2cpp.so", 0x21c9800, {
    ret = "float",
    args = {"ptr"}
})

local speed = get_TrueSpeedMethod(instance)
LOGD("get_TrueSpeed=%f", speed)
```

## 9.4 Raw return pointer

```lua
local get_cloths = BOMBA.Function("libil2cpp.so", 0x21c966c, {
    ret = "ptr",
    args = {"ptr"}
})

local cloths = get_cloths(instance)
LOGD("cloths=%p", cloths)
LOGD("cloths class=%s", BOMBA.ClassName(cloths))
```

## 9.5 Cast from resolved address

```lua
local addr = getLibraryAddress("libil2cpp.so", 0x21c70e0)

local fn = BOMBA.cast(addr, {
    ret = "void",
    args = {"ptr", "int"}
})

fn(instance, 2)
```

## 9.6 One-time call without saving the function

```lua
BOMBA.CallFunction("libil2cpp.so", 0x21bf814, {
    ret = "void",
    args = {"int"}
}, 2)
```

---

# 10. Field API

## 10.1 Non-static int field

Lua:

```lua
local cls = BOMBA.Class("a", "bn")

local field = cls:GetField("not_static")
    :cast("int")

field:Set(instance, 5)

local value = field:Get(instance)
LOGD("field value=%i", value)
```

## 10.2 Static int field
Lua:

```lua
local cls = BOMBA.Class("a", "bn")

local field = cls:GetField("static")
    :cast("int")

field:SetStatic(5)

local value = field:GetStatic()
LOGD("static field=%i", value)
```

## 10.3 Field cast types

```lua
field:cast("int")
field:cast("uint")
field:cast("bool")
field:cast("float")
field:cast("double")
field:cast("long")
field:cast("ulong")
field:cast("ptr")
field:cast("string")
```

---

# 11. Read / write memory

## 11.1 Int

```lua
local value = BOMBA.read_int(instance, 0x12)
BOMBA.write_int(instance, 0x12, 5)
```

## 11.2 Bool

```lua
local value = BOMBA.read_bool(instance, 0x20)
BOMBA.write_bool(instance, 0x20, true)
```

## 11.3 Float

Equivalent C++:

```cpp
*(float *)((uintptr_t)instance + 0x12) = 5.0f;
```

Lua:

```lua
BOMBA.write_float(instance, 0x12, 5.0)

local speed = BOMBA.read_float(instance, 0x12)
LOGD("speed=%f", speed)
```

## 11.4 Double

```lua
BOMBA.write_double(instance, 0x30, 10.5)
local v = BOMBA.read_double(instance, 0x30)
```

## 11.5 Pointer

```lua
local ptr = BOMBA.read_ptr(instance, 0x40)
BOMBA.write_ptr(instance, 0x40, ptr)
```

---

# 12. MonoString / String

## 12.1 Create MonoString

```lua
local mono = BOMBA.CreateMonoString("HELLO WORLD")
```

## 12.2 MonoString to Lua string

```lua
local text = BOMBA.MonoStringToString(mono)
LOGD("text=%s", text)
```

Alias:

```lua
local text = BOMBA.ToString(mono)
```

## 12.3 Use in method call

```lua
local cls = BOMBA.Class("", "Example")

local setName = cls:GetMethod("SetName", 1)
    :cast({
        ret = "void",
        args = {"ptr", "string"}
    })

setName:Call(instance, "HELLO WORLD")
```

---

# 13. MemoryPatch

## 13.1 Patch by `lib + offset`

C++:

```cpp
auto get_canShoot = MemoryPatch::createWithHex(
    getLibraryAddress("libil2cpp.so", 0x0),
    "01 00 A0 E3 1E FF 2F E1"
);

get_canShoot.Modify();
get_canShoot.Restore();
```

Lua:

```lua
local get_canShoot = BOMBA.MemoryPatch(
    "libil2cpp.so",
    0x0,
    "01 00 A0 E3 1E FF 2F E1"
)

get_canShoot:Modify()
get_canShoot:Restore()
```

## 13.2 Patch by method

C++:

```cpp
auto get_canShoot_2 = MemoryPatch::createWithHex(
    BNM::Class("", "PlayerPhysics").GetMethod("getsOEMTHIGN", 0).GetOffset(),
    "01 00 A0 E3 1E FF 2F E1"
);
```

Lua:

```lua
local method = BOMBA.Class("", "PlayerPhysics")
    :GetMethod("getsOEMTHIGN", 0)

local get_canShoot_2 = BOMBA.MemoryPatch(
    method,
    "01 00 A0 E3 1E FF 2F E1"
)

get_canShoot_2:Modify()
get_canShoot_2:Restore()
```

## 13.3 Patch by raw address

```lua
local addr = getLibraryAddress("libil2cpp.so", 0x0)

local patch = BOMBA.MemoryPatch(
    addr,
    "01 00 A0 E3 1E FF 2F E1"
)

patch:Modify()
```

## 13.4 Toggle patch

```lua
local patch = BOMBA.MemoryPatch("libil2cpp.so", 0x123456, "01 00 A0 E3 1E FF 2F E1")
local enabled = false

function TogglePatch()
    enabled = not enabled

    if enabled then
        patch:Modify()
        LOGD("patch enabled")
    else
        patch:Restore()
        LOGD("patch disabled")
    end
end
```

## 13.5 Patch methods

```lua
patch:Modify()
patch:Restore()
patch:IsModified()
patch:GetAddress()
```

Aliases:

```lua
BOMBA.MemoryPatch(...)
BOMBA.Patch(...)
BOMBA.CreatePatch(...)
MemoryPatch(...)
```

---

# 14. ImGui from Lua

You have a global table:

```lua
ImGui
```

Rendering can be done with the global function:

```lua
function DrawImGui(width, height)
    -- draw here
end
```

Or with a callback:

```lua
ImGui.SetDrawCallback(function(width, height)
    -- draw here
end)
```

Clear callback:

```lua
ImGui.ClearDrawCallback()
```

---

# 15. Minimal ImGui window

```lua
showMenu = true
speed = 1.0
godMode = false

function DrawImGui(width, height)
    ImGui.SetNextWindowSize({420, 320}, ImGui.Cond_Once)

    if ImGui.Begin("BOMBA Menu", ImGui.WindowFlags_NoCollapse) then
        ImGui.Text("Hello from Lua ImGui")
        ImGui.Separator()

        godMode = ImGui.Checkbox("God Mode", godMode)

        speed = ImGui.SliderFloat("Speed", speed, 0.1, 10.0, "%.2f")

        if ImGui.Button("Print") then
            LOGD("godMode=%s speed=%f", tostring(godMode), speed)
        end
    end

    ImGui.End()
end
```

---

# 16. ImGui return value rule

Many widgets return:

```lua
newValue, changed = ImGui.Widget(...)
```

Example:

```lua
speed, changed = ImGui.SliderFloat("Speed", speed, 0.1, 10.0)

if changed then
    LOGD("new speed=%f", speed)
end
```

Checkbox:

```lua
enabled, changed = ImGui.Checkbox("Enabled", enabled)
```

InputText:

```lua
name, changed = ImGui.InputText("Name", name, 128)
```

Combo:

```lua
selectedIndex, changed = ImGui.Combo("Mode", selectedIndex, {"Normal", "Fast", "Ultra"})
```

`Combo` uses Lua indexing, so it starts from `1`.

---

# 17. ImGui widgets — examples

## 17.1 Text

```lua
ImGui.Text("Normal text")
ImGui.TextWrapped("Long long long wrapped text")
```

## 17.2 Button

```lua
if ImGui.Button("Click me", {120, 40}) then
    LOGD("clicked")
end

if ImGui.SmallButton("Small") then
    LOGD("small clicked")
end
```

## 17.3 Checkbox

```lua
enabled, changed = ImGui.Checkbox("Enabled", enabled)
```

## 17.4 RadioButton

```lua
if ImGui.RadioButton("Option A", mode == 1) then
    mode = 1
end

if ImGui.RadioButton("Option B", mode == 2) then
    mode = 2
end
```

## 17.5 SliderInt

```lua
value, changed = ImGui.SliderInt("Value", value, 0, 100)
```

## 17.6 SliderFloat

```lua
speed, changed = ImGui.SliderFloat("Speed", speed, 0.1, 20.0, "%.2f")
```

## 17.7 DragInt

```lua
count, changed = ImGui.DragInt("Count", count, 1.0, 0, 999)
```

## 17.8 DragFloat

```lua
scale, changed = ImGui.DragFloat("Scale", scale, 0.05, 0.0, 10.0, "%.2f")
```

## 17.9 InputText

```lua
playerName, changed = ImGui.InputText("Player name", playerName, 128)
```

## 17.10 InputTextMultiline

```lua
bigText, changed = ImGui.InputTextMultiline("Script", bigText, 4096, {400, 200})
```

## 17.11 InputInt

```lua
amount, changed = ImGui.InputInt("Amount", amount)
```

## 17.12 InputFloat

```lua
gravity, changed = ImGui.InputFloat("Gravity", gravity)
```

## 17.13 Combo

```lua
mode, changed = ImGui.Combo("Mode", mode, {
    "Normal",
    "Fast",
    "Insane"
})
```

## 17.14 BeginCombo / Selectable

```lua
local names = {"Red", "Green", "Blue"}
local preview = names[colorIndex] or "Select"

if ImGui.BeginCombo("Color", preview) then
    for i, name in ipairs(names) do
        if ImGui.Selectable(name, colorIndex == i) then
            colorIndex = i
        end
    end

    ImGui.EndCombo()
end
```

## 17.15 ColorEdit4

```lua
menuColor, changed = ImGui.ColorEdit4("Menu color", menuColor)
```

`menuColor` is a table:

```lua
menuColor = {0.1, 0.2, 0.8, 1.0}
```

## 17.16 ColorPicker4

```lua
accentColor, changed = ImGui.ColorPicker4("Accent", accentColor)
```

---

# 18. ImGui layout

## 18.1 SameLine

```lua
ImGui.Button("A")
ImGui.SameLine()
ImGui.Button("B")
```

## 18.2 Separator / spacing

```lua
ImGui.Separator()
ImGui.Spacing()
ImGui.NewLine()
```

## 18.3 PushItemWidth

```lua
ImGui.PushItemWidth(180)
speed, changed = ImGui.SliderFloat("Speed", speed, 0.1, 10.0)
ImGui.PopItemWidth()
```

## 18.4 Style color

```lua
ImGui.PushStyleColor(ImGui.Col_Button, {0.0, 0.2, 0.9, 1.0})

if ImGui.Button("Blue button") then
    LOGD("clicked")
end

ImGui.PopStyleColor()
```

Multiple style colors:

```lua
ImGui.PushStyleColor(ImGui.Col_Button, {0.0, 0.2, 0.9, 1.0})
ImGui.PushStyleColor(ImGui.Col_ButtonHovered, {0.0, 0.3, 1.0, 1.0})

ImGui.Button("Styled")

ImGui.PopStyleColor(2)
```

---

# 19. ImGui tabs

```lua
function DrawImGui(width, height)
    if ImGui.Begin("BOMBA Menu", ImGui.WindowFlags_NoCollapse) then
        if ImGui.BeginTabBar("tabs") then

            if ImGui.BeginTabItem("Main") then
                ImGui.Text("Main tab")
                ImGui.EndTabItem()
            end

            if ImGui.BeginTabItem("Hooks") then
                ImGui.Text("Hooks tab")
                ImGui.EndTabItem()
            end

            if ImGui.BeginTabItem("Settings") then
                ImGui.Text("Settings tab")
                ImGui.EndTabItem()
            end

            ImGui.EndTabBar()
        end
    end

    ImGui.End()
end
```

---

# 20. ImGui tables

```lua
if ImGui.BeginTable("features", 3) then
    ImGui.TableNextRow()

    ImGui.TableSetColumnIndex(0)
    ImGui.Text("Feature")

    ImGui.TableSetColumnIndex(1)
    ImGui.Text("Value")

    ImGui.TableSetColumnIndex(2)
    ImGui.Text("Action")

    ImGui.TableNextRow()

    ImGui.TableSetColumnIndex(0)
    ImGui.Text("Speed")

    ImGui.TableSetColumnIndex(1)
    ImGui.Text(tostring(speed))

    ImGui.TableSetColumnIndex(2)
    if ImGui.Button("Reset") then
        speed = 1.0
    end

    ImGui.EndTable()
end
```

---

# 21. ImGui child windows

```lua
if ImGui.BeginChild("left", {180, 250}, true) then
    ImGui.Text("Left panel")
end
ImGui.EndChild()

ImGui.SameLine()

if ImGui.BeginChild("right", {300, 250}, true) then
    ImGui.Text("Right panel")
end
ImGui.EndChild()
```

---

# 22. ImGui collapsing headers and trees

```lua
if ImGui.CollapsingHeader("Advanced") then
    ImGui.Text("Advanced options")
end

if ImGui.TreeNode("Debug") then
    ImGui.Text("Debug content")
    ImGui.TreePop()
end
```

---

# 23. Touch rects for ImGui

The easiest way: the current window adds a touch rect automatically when `ImGui.End()` is called.

You can force it manually:

```lua
ImGui.TouchRectCurrentWindow()
```

Or set a manual touch area:

```lua
ImGui.SetTouchRect(100, 100, 400, 300)
```

With multiple windows, it is safest to call this at the end of each window:

```lua
ImGui.TouchRectCurrentWindow()
```

Full example:

```lua
function DrawImGui(width, height)
    if ImGui.Begin("Window 1", ImGui.WindowFlags_NoCollapse) then
        ImGui.Text("First window")
    end
    ImGui.TouchRectCurrentWindow()
    ImGui.End()

    if ImGui.Begin("Window 2", ImGui.WindowFlags_NoCollapse) then
        ImGui.Text("Second window")
    end
    ImGui.TouchRectCurrentWindow()
    ImGui.End()
end
```

---

# 24. Full ImGui menu example

```lua
menu = {
    god = false,
    speed = 1.0,
    amount = 100,
    name = "Player",
    color = {0.0, 0.25, 1.0, 1.0},
    mode = 1,
    patchEnabled = false
}

local modes = {
    "Normal",
    "Fast",
    "Ultra"
}

local patch = nil

function DrawImGui(width, height)
    ImGui.SetNextWindowSize({520, 420}, ImGui.Cond_Once)

    if ImGui.Begin("BOMBA Lua Menu", ImGui.WindowFlags_NoCollapse) then
        if ImGui.BeginTabBar("main_tabs") then

            if ImGui.BeginTabItem("Main") then
                menu.god = ImGui.Checkbox("God mode", menu.god)

                menu.speed = ImGui.SliderFloat("Speed", menu.speed, 0.1, 20.0, "%.2f")

                menu.amount = ImGui.InputInt("Amount", menu.amount)

                menu.name = ImGui.InputText("Name", menu.name, 128)

                menu.mode = ImGui.Combo("Mode", menu.mode, modes)

                if ImGui.Button("Print values", {160, 40}) then
                    LOGD("god=%s speed=%f amount=%i name=%s mode=%i",
                        tostring(menu.god),
                        menu.speed,
                        menu.amount,
                        menu.name,
                        menu.mode
                    )
                end

                ImGui.EndTabItem()
            end

            if ImGui.BeginTabItem("Style") then
                menu.color = ImGui.ColorPicker4("Accent", menu.color)

                ImGui.PushStyleColor(ImGui.Col_Button, menu.color)
                ImGui.Button("Preview button", {180, 40})
                ImGui.PopStyleColor()

                ImGui.EndTabItem()
            end

            if ImGui.BeginTabItem("Patch") then
                if patch == nil then
                    patch = BOMBA.MemoryPatch(
                        "libil2cpp.so",
                        0x0,
                        "01 00 A0 E3 1E FF 2F E1"
                    )
                end

                if ImGui.Button(menu.patchEnabled and "Disable patch" or "Enable patch", {180, 40}) then
                    menu.patchEnabled = not menu.patchEnabled

                    if menu.patchEnabled then
                        patch:Modify()
                    else
                        patch:Restore()
                    end
                end

                ImGui.EndTabItem()
            end

            ImGui.EndTabBar()
        end
    end

    ImGui.TouchRectCurrentWindow()
    ImGui.End()
end
```

---

# 25. Combining a hook with ImGui

```lua
feature = {
    forceBodyType = false,
    bodyType = 1
}

local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")

function PlayerPhysics_FixedUpdate(instance)
    if feature.forceBodyType then
        PlayerPhysics:GetMethod("SetBodyType", 1)
            :cast({
                ret = "void",
                args = {"ptr", "int"}
            })
            :Call(instance, feature.bodyType)
    end

    return BOMBA.call_original(instance)
end

local FixedUpdate = PlayerPhysics:GetMethod("FixedUpdate", 0)

BOMBA.HOOK(FixedUpdate, {
    ret = "void",
    args = {"ptr"}
}, PlayerPhysics_FixedUpdate)

function DrawImGui(width, height)
    if ImGui.Begin("PlayerPhysics", ImGui.WindowFlags_NoCollapse) then
        feature.forceBodyType = ImGui.Checkbox("Force body type", feature.forceBodyType)
        feature.bodyType = ImGui.SliderInt("Body type", feature.bodyType, 0, 5)
    end

    ImGui.End()
end

return "hook + imgui ready"
```

---

# 26. BOMBA function reference

## 26.1 Class

```lua
BOMBA.Class(namespace, className)
BOMBA.Class(instance)
```

## 26.2 Method

```lua
class:GetMethod(name, argsCount)
method:cast(signature)
method:Call(...)
method:CallStatic(...)
method:GetOffset()
method:GetAddress()
```

## 26.3 Field

```lua
class:GetField(name)
field:cast(type)
field:Get(instance)
field:Set(instance, value)
field:GetStatic()
field:SetStatic(value)
```

## 26.4 Hook

```lua
BOMBA.HOOK(method, signature, callback)
BOMBA.HOOK(address, signature, callback)
BOMBA.HOOK(libName, offset, signature, callback)
BOMBA.call_original(...)
```

## 26.5 Raw function

```lua
BOMBA.Function(address, signature)
BOMBA.Function(libName, offset, signature)
BOMBA.Func(...)
BOMBA.CastAddress(...)
BOMBA.castAddress(...)
BOMBA.cast(...)
BOMBA.CallFunction(address, signature, ...)
BOMBA.CallFunction(libName, offset, signature, ...)
NativeFunction(...)
```

## 26.6 Memory patch

```lua
BOMBA.MemoryPatch(address, hex)
BOMBA.MemoryPatch(method, hex)
BOMBA.MemoryPatch(libName, offset, hex)

patch:Modify()
patch:Restore()
patch:IsModified()
patch:GetAddress()
```

## 26.7 Strings

```lua
BOMBA.CreateMonoString(text)
BOMBA.MonoStringToString(ptr)
BOMBA.ToString(ptr)
```

## 26.8 Memory read/write

```lua
BOMBA.read_int(ptr, offset)
BOMBA.write_int(ptr, offset, value)

BOMBA.read_bool(ptr, offset)
BOMBA.write_bool(ptr, offset, value)

BOMBA.read_float(ptr, offset)
BOMBA.write_float(ptr, offset, value)

BOMBA.read_double(ptr, offset)
BOMBA.write_double(ptr, offset, value)

BOMBA.read_ptr(ptr, offset)
BOMBA.write_ptr(ptr, offset, value)
```

## 26.9 Address helpers

```lua
getLibraryAddress(libName, offset)

BOMBA.getLibraryAddress(libName, offset)
BOMBA.GetLibraryAddress(libName, offset)
BOMBA.Address(libName, offset)

BOMBA.CheckAddress(libName, offset)
BOMBA.getLibraryAddressRVA(libName, offset)
BOMBA.getLibraryAddressFileOffset(libName, offset)

BOMBA.GetMethodAddress(method)
BOMBA.MethodAddress(method)
BOMBA.DumpMethodAddress(method, "libil2cpp.so")
```

---

# 27. ImGui function reference

## 27.1 Frame callback

```lua
ImGui.SetDrawCallback(function(width, height)
end)

ImGui.ClearDrawCallback()

function DrawImGui(width, height)
end
```

## 27.2 Window

```lua
ImGui.Begin(title, flags)
ImGui.End()

ImGui.BeginChild(id, size, border, flags)
ImGui.EndChild()

ImGui.SetNextWindowSize({w, h}, cond)
ImGui.SetNextWindowPos({x, y}, cond)

ImGui.GetWindowPos()
ImGui.GetWindowSize()
```

## 27.3 Text / layout

```lua
ImGui.Text(text)
ImGui.TextWrapped(text)
ImGui.Separator()
ImGui.SameLine(offset, spacing)
ImGui.Spacing()
ImGui.NewLine()
```

## 27.4 Widgets

```lua
ImGui.Button(label, size)
ImGui.SmallButton(label)

value, changed = ImGui.Checkbox(label, value)
clicked = ImGui.RadioButton(label, active)

value, changed = ImGui.SliderInt(label, value, min, max)
value, changed = ImGui.SliderFloat(label, value, min, max, format)

value, changed = ImGui.DragInt(label, value, speed, min, max)
value, changed = ImGui.DragFloat(label, value, speed, min, max, format)

text, changed = ImGui.InputText(label, current, capacity)
text, changed = ImGui.InputTextMultiline(label, current, capacity, size)

value, changed = ImGui.InputInt(label, value)
value, changed = ImGui.InputFloat(label, value)

index, changed = ImGui.Combo(label, currentIndex, itemsTable)

opened = ImGui.BeginCombo(label, preview, flags)
ImGui.EndCombo()

clicked = ImGui.Selectable(label, selected, flags)

color, changed = ImGui.ColorEdit4(label, color)
color, changed = ImGui.ColorPicker4(label, color)
```

## 27.5 Tabs

```lua
ImGui.BeginTabBar(id, flags)
ImGui.EndTabBar()

ImGui.BeginTabItem(label, flags)
ImGui.EndTabItem()
```

## 27.6 Tree / collapsing

```lua
ImGui.CollapsingHeader(label, flags)
ImGui.TreeNode(label)
ImGui.TreePop()
```

## 27.7 Tables

```lua
ImGui.BeginTable(id, columns, flags)
ImGui.EndTable()

ImGui.TableNextRow()
ImGui.TableNextColumn()
ImGui.TableSetColumnIndex(index)
```

## 27.8 Input / hover

```lua
ImGui.IsItemHovered()
ImGui.IsItemClicked(button)
ImGui.IsWindowHovered(flags)
```

## 27.9 Style

```lua
ImGui.PushStyleColor(index, color)
ImGui.PopStyleColor(count)

ImGui.PushItemWidth(width)
ImGui.PopItemWidth()
```

## 27.10 Touch

```lua
ImGui.SetTouchRect(x, y, w, h)
ImGui.TouchRectCurrentWindow()
```

---

# 28. ImGui constants

## 28.1 Window flags

```lua
ImGui.WindowFlags_None
ImGui.WindowFlags_NoTitleBar
ImGui.WindowFlags_NoResize
ImGui.WindowFlags_NoMove
ImGui.WindowFlags_NoScrollbar
ImGui.WindowFlags_NoCollapse
ImGui.WindowFlags_AlwaysAutoResize
ImGui.WindowFlags_NoBackground
```

Combine flags:

```lua
local flags = ImGui.WindowFlags_NoCollapse + ImGui.WindowFlags_NoResize
ImGui.Begin("Menu", flags)
```

## 28.2 Cond

```lua
ImGui.Cond_Always
ImGui.Cond_Once
ImGui.Cond_FirstUseEver
ImGui.Cond_Appearing
```

## 28.3 Colors

```lua
ImGui.Col_Text
ImGui.Col_WindowBg
ImGui.Col_Button
ImGui.Col_ButtonHovered
ImGui.Col_ButtonActive
```

---

# 29. Ready-to-use starter template

```lua
-- state
local state = {
    enabled = false,
    speed = 1.0,
    mode = 1,
    color = {0.0, 0.2, 1.0, 1.0}
}

local modes = {"Normal", "Fast", "Ultra"}

-- classes
local PlayerPhysics = BOMBA.Class("", "PlayerPhysics")

-- hook
function PlayerPhysics_FixedUpdate(instance)
    if state.enabled then
        PlayerPhysics:GetMethod("SetBodyType", 1)
            :cast({
                ret = "void",
                args = {"ptr", "int"}
            })
            :Call(instance, state.mode)
    end

    return BOMBA.call_original(instance)
end

local FixedUpdate = PlayerPhysics:GetMethod("FixedUpdate", 0)

BOMBA.HOOK(FixedUpdate, {
    ret = "void",
    args = {"ptr"}
}, PlayerPhysics_FixedUpdate)

-- imgui
function DrawImGui(width, height)
    ImGui.SetNextWindowSize({520, 380}, ImGui.Cond_Once)

    if ImGui.Begin("BOMBA", ImGui.WindowFlags_NoCollapse) then
        state.enabled = ImGui.Checkbox("Enabled", state.enabled)
        state.speed = ImGui.SliderFloat("Speed", state.speed, 0.1, 10.0, "%.2f")
        state.mode = ImGui.Combo("Mode", state.mode, modes)

        state.color = ImGui.ColorEdit4("Color", state.color)

        ImGui.PushStyleColor(ImGui.Col_Button, state.color)
        if ImGui.Button("Debug print", {160, 40}) then
            LOGD("enabled=%s speed=%f mode=%i", tostring(state.enabled), state.speed, state.mode)
        end
        ImGui.PopStyleColor()
    end

    ImGui.TouchRectCurrentWindow()
    ImGui.End()
end

return "BOMBA script loaded"
```

---

# 30. Troubleshooting

## 30.1 Hook registered, but callback does not fire

Check if the method really executes:

```lua
LOGD("hook registered")
```

Check the address:

```lua
local info = BOMBA.CheckAddress("libil2cpp.so", 0xOFFSET)
LOGD("ok=%s addr=%p perms=%s", tostring(info.ok), info.address, info.perms)
```

If `BOMBA.HOOK(method, ...)` works but `BOMBA.HOOK("lib", offset, ...)` does not, use:

```lua
local dump = BOMBA.DumpMethodAddress(method, "libil2cpp.so")

LOGD("method addr=%p rva=0x%x file=0x%x",
    dump.address,
    dump.rva,
    dump.file_offset
)
```

## 30.2 Crash when hook runs

Check the signature.  
Most crashes happen because the argument count is wrong:

```lua
{ ret = "void", args = {"ptr"} }
```

or:

```lua
{ ret = "void", args = {"ptr", "ptr"} }
```

## 30.3 ImGui does not receive touch

At the end of the window, add:

```lua
ImGui.TouchRectCurrentWindow()
```

For multiple windows, do it for every window.

## 30.4 Logs lag

Do not spam `LOGD` every frame without a condition.  
Better:

```lua
counter = (counter or 0) + 1

if counter % 60 == 0 then
    LOGD("tick=%i", counter)
end
```

---

# 31. Suggested GitHub repo structure

```text
repo/
 ├─ README.md
 ├─ README_PL.md
 ├─ examples/
 │   ├─ 01_basic_hook.lua
 │   ├─ 02_imgui_menu.lua
 │   ├─ 03_raw_function.lua
 │   ├─ 04_memory_patch.lua
 │   ├─ 05_fields.lua
 │   └─ 06_full_template.lua
 └─ docs/
     ├─ BOMBA_API.md
     └─ IMGUI_API.md
```

---

# 32. Short cheat sheet

```lua
-- class
local cls = BOMBA.Class("", "PlayerPhysics")

-- method
local m = cls:GetMethod("FixedUpdate", 0)

-- hook
BOMBA.HOOK(m, { ret = "void", args = {"ptr"} }, function(instance)
    return BOMBA.call_original(instance)
end)

-- call method
cls:GetMethod("SetBodyType", 1)
    :cast({ ret = "void", args = {"ptr", "int"} })
    :Call(instance, 1)

-- raw function
local fn = BOMBA.Function("libil2cpp.so", 0x123456, {
    ret = "void",
    args = {"ptr", "int"}
})

fn(instance, 2)

-- patch
local patch = BOMBA.MemoryPatch("libil2cpp.so", 0x123456, "01 00 A0 E3 1E FF 2F E1")
patch:Modify()
patch:Restore()

-- imgui
function DrawImGui(w, h)
    if ImGui.Begin("Menu", ImGui.WindowFlags_NoCollapse) then
        ImGui.Text("Hello")
    end
    ImGui.End()
end
```
