-- 🔱 Titanium Hub | Speed Monkey Escape
-- V2 — UI Definitiva

local Players           = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService      = game:GetService("TweenService")
local UserInputService  = game:GetService("UserInputService")
local CoreGui           = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")
local Remotes     = ReplicatedStorage:WaitForChild("Remotes")

local AurasConfig = {
    ["Amber"]    = { Multi = 1.5,  Price = 5000            },
    ["Ice Cold"] = { Multi = 2,    Price = 150000          },
    ["Nature"]   = { Multi = 2.5,  Price = 4000000         },
    ["Rainbow"]  = { Multi = 3,    Price = 2500000000      },
    ["Lunar"]    = { Multi = 3.5,  Price = 90000000000     },
    ["Sparkle"]  = { Multi = 4,    Price = 5250000000000   },
}
local TrailsConfig = {
    ["Red"]     = { Multi = 1.5,  Price = 1000            },
    ["Blue"]    = { Multi = 2,    Price = 30000           },
    ["Green"]   = { Multi = 3,    Price = 750000          },
    ["Rainbow"] = { Multi = 5,    Price = 400000000       },
    ["Galaxy"]  = { Multi = 10,   Price = 18750000000     },
    ["Divine"]  = { Multi = 25,   Price = 600000000000    },
}
local MonkeyTailsConfig = {
    ["Basic Tail"]   = { Multi = 1.5,  Price = 500          },
    ["Flame Tail"]   = { Multi = 2,    Price = 25000        },
    ["Ice Tail"]     = { Multi = 3,    Price = 500000       },
    ["Thunder Tail"] = { Multi = 4,    Price = 20000000     },
    ["Galaxy Tail"]  = { Multi = 6,    Price = 800000000    },
    ["Divine Tail"]  = { Multi = 12,   Price = 50000000000  },
}
local REBIRTH_REQUIRED_LEVEL = 25
local MAX_REBIRTHS           = 11

_G.AutoRebirth    = false
_G.AutoAura       = false
_G.AutoTrail      = false
_G.AutoMonkeyTail = false

-- ══════════════════════════════════════════
-- GUI PARENT
-- ══════════════════════════════════════════
local targetParent
pcall(function()
    local t = Instance.new("Frame"); t.Parent = CoreGui; t:Destroy()
    targetParent = CoreGui
end)
if not targetParent then targetParent = PlayerGui end
if targetParent:FindFirstChild("TitaniumV2") then targetParent.TitaniumV5:Destroy() end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name            = "TitaniumV5"
ScreenGui.ResetOnSpawn    = false
ScreenGui.ZIndexBehavior  = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset  = true
ScreenGui.Parent          = targetParent

-- ══════════════════════════════════════════
-- PALETA
-- ══════════════════════════════════════════
local C = {
    bg        = Color3.fromRGB(10, 10, 16),
    surface   = Color3.fromRGB(16, 16, 26),
    card      = Color3.fromRGB(20, 20, 32),
    cardHov   = Color3.fromRGB(24, 24, 40),
    border    = Color3.fromRGB(38, 38, 60),
    borderAct = Color3.fromRGB(0, 170, 210),
    accent    = Color3.fromRGB(0, 200, 245),
    accentDim = Color3.fromRGB(0, 100, 130),
    text      = Color3.fromRGB(210, 210, 230),
    textDim   = Color3.fromRGB(70, 70, 100),
    textMid   = Color3.fromRGB(130, 130, 160),
    green     = Color3.fromRGB(0, 220, 130),
    greenDim  = Color3.fromRGB(0, 60, 40),
    red       = Color3.fromRGB(220, 60, 80),
    redDim    = Color3.fromRGB(60, 15, 20),
}

local function tw(obj, t, props)
    TweenService:Create(obj, TweenInfo.new(t, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), props):Play()
end

-- ══════════════════════════════════════════
-- HELPERS
-- ══════════════════════════════════════════
local function newFrame(parent, props)
    local f = Instance.new("Frame")
    f.BackgroundTransparency = 1
    f.BorderSizePixel = 0
    for k,v in pairs(props or {}) do f[k] = v end
    f.Parent = parent
    return f
end
local function newLabel(parent, props)
    local l = Instance.new("TextLabel")
    l.BackgroundTransparency = 1
    l.BorderSizePixel = 0
    l.TextScaled = false
    for k,v in pairs(props or {}) do l[k] = v end
    l.Parent = parent
    return l
end
local function corner(parent, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 10)
    c.Parent = parent
end
local function stroke(parent, col, thick)
    local s = Instance.new("UIStroke")
    s.Color = col or C.border
    s.Thickness = thick or 1.2
    s.Parent = parent
    return s
end

-- ══════════════════════════════════════════
-- DRAG (sem drift)
-- ══════════════════════════════════════════
local function makeDraggable(handle, target)
    local drag, start, origin = false, Vector3.new(), UDim2.new()
    handle.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag = true; start = i.Position; origin = target.Position
        end
    end)
    handle.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag = false
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if not drag then return end
        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then
            local d = i.Position - start
            target.Position = UDim2.new(origin.X.Scale, origin.X.Offset+d.X, origin.Y.Scale, origin.Y.Offset+d.Y)
        end
    end)
end

-- ══════════════════════════════════════════
-- MINI BUTTON
-- ══════════════════════════════════════════
local MiniBtn = Instance.new("TextButton")
MiniBtn.Size             = UDim2.new(0, 54, 0, 54)
MiniBtn.Position         = UDim2.new(0, 20, 0.5, -27)
MiniBtn.BackgroundColor3 = C.bg
MiniBtn.Text             = "🔱"
MiniBtn.TextSize         = 24
MiniBtn.Font             = Enum.Font.GothamBold
MiniBtn.TextColor3       = C.accent
MiniBtn.AutoButtonColor  = false
MiniBtn.Visible          = false
MiniBtn.ZIndex           = 20
MiniBtn.Parent           = ScreenGui
corner(MiniBtn, 14)
local miniStroke = stroke(MiniBtn, C.accentDim, 1.5)

local miniGlow = Instance.new("ImageLabel", MiniBtn)
miniGlow.Size                   = UDim2.new(1.8,0,1.8,0)
miniGlow.Position               = UDim2.new(-0.4,0,-0.4,0)
miniGlow.BackgroundTransparency = 1
miniGlow.Image                  = "rbxassetid://5028857084"
miniGlow.ImageColor3            = C.accent
miniGlow.ImageTransparency      = 0.85
miniGlow.ZIndex                 = 19

MiniBtn.MouseEnter:Connect(function()
    tw(MiniBtn, 0.15, {BackgroundColor3 = C.accentDim})
    tw(miniStroke, 0.15, {Color = C.accent})
    tw(miniGlow, 0.15, {ImageTransparency = 0.6})
end)
MiniBtn.MouseLeave:Connect(function()
    tw(MiniBtn, 0.15, {BackgroundColor3 = C.bg})
    tw(miniStroke, 0.15, {Color = C.accentDim})
    tw(miniGlow, 0.15, {ImageTransparency = 0.85})
end)
makeDraggable(MiniBtn, MiniBtn)

-- ══════════════════════════════════════════
-- MAIN WINDOW
-- ══════════════════════════════════════════
local Main = Instance.new("Frame")
Main.Name             = "Main"
Main.Size             = UDim2.new(0, 380, 0, 380)
Main.Position         = UDim2.new(0, 120, 0.5, -190)
Main.BackgroundColor3 = C.bg
Main.BorderSizePixel  = 0
Main.ClipsDescendants = false   -- false: evita cortar filhos e não bloqueia input
Main.ZIndex           = 5
Main.Parent           = ScreenGui
corner(Main, 16)
stroke(Main, C.border, 1.5)

-- Clip interno separado (só o visual, não o input)
local ClipInner = Instance.new("Frame", Main)
ClipInner.Size             = UDim2.new(1,0,1,0)
ClipInner.BackgroundColor3 = C.bg
ClipInner.BorderSizePixel  = 0
ClipInner.ZIndex           = 5
ClipInner.ClipsDescendants = true
corner(ClipInner, 16)

local BgGrad = Instance.new("UIGradient", ClipInner)
BgGrad.Color    = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(12,12,20)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(8,8,14)),
})
BgGrad.Rotation = 135

-- Shadow
local Shadow = Instance.new("ImageLabel", Main)
Shadow.Size                   = UDim2.new(1,60,1,60)
Shadow.Position               = UDim2.new(0,-30,0,-30)
Shadow.BackgroundTransparency = 1
Shadow.Image                  = "rbxassetid://5028857084"
Shadow.ImageColor3            = Color3.fromRGB(0,0,0)
Shadow.ImageTransparency      = 0.55
Shadow.ZIndex                 = 4

-- Top accent bar
local TopBar = newFrame(Main, {
    Size             = UDim2.new(0,0,0,2),
    Position         = UDim2.new(0.1,0,0,0),
    BackgroundColor3 = C.accent,
    ZIndex           = 10,
})
corner(TopBar, 2)
local topGrad = Instance.new("UIGradient", TopBar)
topGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0,   Color3.fromRGB(0,150,200)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0,220,255)),
    ColorSequenceKeypoint.new(1,   Color3.fromRGB(0,150,200)),
})
task.delay(0.1, function() tw(TopBar, 0.6, {Size = UDim2.new(0.8,0,0,2)}) end)

-- ══════════════════════════════════════════
-- TITLE BAR
-- ══════════════════════════════════════════
local TitleBar = newFrame(Main, {
    Size   = UDim2.new(1,0,0,60),
    ZIndex = 9,
})

local IconBg = Instance.new("Frame", TitleBar)
IconBg.Size             = UDim2.new(0,40,0,40)
IconBg.Position         = UDim2.new(0,16,0.5,-20)
IconBg.BackgroundColor3 = C.accentDim
IconBg.BorderSizePixel  = 0
IconBg.ZIndex           = 10
corner(IconBg, 10)
stroke(IconBg, C.accent, 1)
newLabel(IconBg, { Size=UDim2.new(1,0,1,0), Text="🔱", TextSize=20, Font=Enum.Font.GothamBold, ZIndex=11 })

local TitleLbl = newLabel(TitleBar, {
    Size           = UDim2.new(0,120,0,22),
    Position       = UDim2.new(0,64,0.5,-22),
    Text           = "TITANIUM",
    TextColor3     = C.accent,
    TextSize       = 16,
    Font           = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex         = 10,
})
local TitleGrad = Instance.new("UIGradient", TitleLbl)
TitleGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0,230,255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0,160,200)),
})

newLabel(TitleBar, {
    Size           = UDim2.new(0,200,0,14),
    Position       = UDim2.new(0,64,0.5,2),
    Text           = "Speed Monkey Escape  •  v2",
    TextColor3     = C.textDim,
    TextSize       = 10,
    Font           = Enum.Font.GothamMedium,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex         = 10,
})

-- Badge V2
local Badge = Instance.new("Frame", TitleBar)
Badge.Size             = UDim2.new(0,46,0,20)
Badge.Position         = UDim2.new(1,-120,0.5,-10)
Badge.BackgroundColor3 = Color3.fromRGB(0,35,50)
Badge.BorderSizePixel  = 0
Badge.ZIndex           = 10
corner(Badge, 6)
stroke(Badge, C.accentDim, 1)
newLabel(Badge, { Size=UDim2.new(1,0,1,0), Text="V2", TextColor3=C.accent, TextSize=10, Font=Enum.Font.GothamBold, ZIndex=11 })

-- Minimize button
local MinBtn2 = Instance.new("TextButton", TitleBar)
MinBtn2.Size             = UDim2.new(0,28,0,28)
MinBtn2.Position         = UDim2.new(1,-44,0.5,-14)
MinBtn2.BackgroundColor3 = C.surface
MinBtn2.Text             = "–"
MinBtn2.TextSize         = 13
MinBtn2.Font             = Enum.Font.GothamBold
MinBtn2.TextColor3       = C.textMid
MinBtn2.AutoButtonColor  = false
MinBtn2.ZIndex           = 11
corner(MinBtn2, 7)
stroke(MinBtn2, C.border, 1)
MinBtn2.MouseEnter:Connect(function() tw(MinBtn2,0.1,{BackgroundColor3=C.cardHov}) end)
MinBtn2.MouseLeave:Connect(function() tw(MinBtn2,0.1,{BackgroundColor3=C.surface}) end)

-- Divider
newFrame(Main, {
    Size             = UDim2.new(1,-32,0,1),
    Position         = UDim2.new(0,16,0,60),
    BackgroundColor3 = C.border,
    ZIndex           = 9,
})

-- ══════════════════════════════════════════
-- STATUS CARD
-- ══════════════════════════════════════════
local StatCard = Instance.new("Frame", Main)
StatCard.Size             = UDim2.new(1,-32,0,34)
StatCard.Position         = UDim2.new(0,16,0,68)
StatCard.BackgroundColor3 = C.surface
StatCard.BorderSizePixel  = 0
StatCard.ZIndex           = 9
corner(StatCard, 10)
stroke(StatCard, C.border, 1)

local StatAccent = newFrame(StatCard, {
    Size             = UDim2.new(0,3,0.6,0),
    Position         = UDim2.new(0,10,0.2,0),
    BackgroundColor3 = C.accent,
    ZIndex           = 10,
})
corner(StatAccent, 2)

local StatusLbl = newLabel(StatCard, {
    Size           = UDim2.new(1,-28,1,0),
    Position       = UDim2.new(0,20,0,0),
    Text           = "⏳  Aguardando dados do player...",
    TextColor3     = C.textDim,
    TextSize       = 11,
    Font           = Enum.Font.GothamMedium,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex         = 10,
})

task.spawn(function()
    while true do
        tw(StatAccent,0.8,{BackgroundColor3=C.accent})     task.wait(0.9)
        tw(StatAccent,0.8,{BackgroundColor3=C.accentDim})  task.wait(0.9)
    end
end)

-- ══════════════════════════════════════════
-- SCROLL FRAME
-- ══════════════════════════════════════════
local ScrollFrame = Instance.new("ScrollingFrame", Main)
ScrollFrame.Size                  = UDim2.new(1,-32,1,-112)
ScrollFrame.Position              = UDim2.new(0,16,0,108)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.BorderSizePixel       = 0
ScrollFrame.ScrollBarThickness    = 3
ScrollFrame.ScrollBarImageColor3  = C.accentDim
ScrollFrame.CanvasSize            = UDim2.new(0,0,0,0)
ScrollFrame.AutomaticCanvasSize   = Enum.AutomaticSize.Y
ScrollFrame.ZIndex                = 9

-- FIX: ScrollingFrame precisa estar acima do ClipInner em ZIndex
-- e o Main com ClipsDescendants=false para não cortar o input
ScrollFrame.Parent = Main

local ListLayout = Instance.new("UIListLayout", ScrollFrame)
ListLayout.SortOrder = Enum.SortOrder.LayoutOrder
ListLayout.Padding   = UDim.new(0,6)

-- Section label
local SecFrame = newFrame(ScrollFrame, {
    Size      = UDim2.new(1,0,0,20),
    ZIndex    = 9,
})
SecFrame.LayoutOrder = 0
newFrame(SecFrame, { Size=UDim2.new(0,16,0,1), Position=UDim2.new(0,0,0.5,0), BackgroundColor3=C.border, ZIndex=10 })
newLabel(SecFrame, {
    Size=UDim2.new(1,0,1,0), Position=UDim2.new(0,22,0,0),
    Text="AUTOMAÇÕES", TextColor3=C.textDim, TextSize=9,
    Font=Enum.Font.GothamBold, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=10
})
newFrame(SecFrame, { Size=UDim2.new(1,-80,0,1), Position=UDim2.new(0,80,0.5,0), BackgroundColor3=C.border, ZIndex=10 })

-- ══════════════════════════════════════════
-- TOGGLE FACTORY
-- ══════════════════════════════════════════
local function createToggle(cfg)
    local Card = Instance.new("TextButton", ScrollFrame)
    Card.Name             = cfg.name.."Card"
    Card.LayoutOrder      = cfg.order
    Card.Size             = UDim2.new(1,0,0,64)
    Card.BackgroundColor3 = C.card
    Card.Text             = ""
    Card.AutoButtonColor  = false
    Card.BorderSizePixel  = 0
    Card.ZIndex           = 10
    corner(Card, 12)
    local cStroke = stroke(Card, C.border, 1.2)

    local LeftBar = newFrame(Card, {
        Size             = UDim2.new(0,3,0.6,0),
        Position         = UDim2.new(0,0,0.2,0),
        BackgroundColor3 = C.accent,
        ZIndex           = 11,
    })
    corner(LeftBar, 2)
    LeftBar.BackgroundTransparency = 1

    local IconBubble = Instance.new("Frame", Card)
    IconBubble.Size             = UDim2.new(0,40,0,40)
    IconBubble.Position         = UDim2.new(0,14,0.5,-20)
    IconBubble.BackgroundColor3 = C.surface
    IconBubble.BorderSizePixel  = 0
    IconBubble.ZIndex           = 11
    corner(IconBubble, 10)
    local iBubStroke = stroke(IconBubble, C.border, 1)
    newLabel(IconBubble, { Size=UDim2.new(1,0,1,0), Text=cfg.icon, TextSize=20, Font=Enum.Font.GothamBold, ZIndex=12 })

    local NameL = newLabel(Card, {
        Size=UDim2.new(0.5,0,0,20), Position=UDim2.new(0,62,0.5,-22),
        Text=cfg.label, TextColor3=C.text, TextSize=13,
        Font=Enum.Font.GothamBold, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=11,
    })
    local DescL = newLabel(Card, {
        Size=UDim2.new(0.55,0,0,14), Position=UDim2.new(0,62,0.5,0),
        Text=cfg.desc, TextColor3=C.textDim, TextSize=10,
        Font=Enum.Font.Gotham, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=11,
    })

    local PillBg = Instance.new("Frame", Card)
    PillBg.Size             = UDim2.new(0,52,0,26)
    PillBg.Position         = UDim2.new(1,-66,0.5,-13)
    PillBg.BackgroundColor3 = C.redDim
    PillBg.BorderSizePixel  = 0
    PillBg.ZIndex           = 11
    corner(PillBg, 13)
    stroke(PillBg, C.border, 1)

    local PillDot = Instance.new("Frame", PillBg)
    PillDot.Size             = UDim2.new(0,18,0,18)
    PillDot.Position         = UDim2.new(0,4,0.5,-9)
    PillDot.BackgroundColor3 = C.red
    PillDot.BorderSizePixel  = 0
    PillDot.ZIndex           = 12
    corner(PillDot, 9)

    local PillLbl = newLabel(PillBg, {
        Size=UDim2.new(0,24,1,0), Position=UDim2.new(1,-26,0,0),
        Text="OFF", TextColor3=C.red, TextSize=9,
        Font=Enum.Font.GothamBold, ZIndex=12,
    })

    local isOn = false

    local function setVisual(on)
        if on then
            tw(Card,       0.25, {BackgroundColor3 = Color3.fromRGB(0,28,42)})
            tw(cStroke,    0.25, {Color            = C.borderAct})
            tw(NameL,      0.2,  {TextColor3       = C.accent})
            tw(DescL,      0.2,  {TextColor3       = C.accentDim})
            tw(IconBubble, 0.2,  {BackgroundColor3 = C.accentDim})
            tw(iBubStroke, 0.2,  {Color            = C.accent})
            tw(PillBg,     0.25, {BackgroundColor3 = C.greenDim})
            tw(PillDot,    0.25, {BackgroundColor3 = C.green, Position=UDim2.new(1,-22,0.5,-9)})
            tw(PillLbl,    0.2,  {TextColor3=C.green, Position=UDim2.new(0,4,0,0)})
            PillLbl.Text = "ON"
            tw(LeftBar,    0.2,  {BackgroundTransparency=0})
        else
            tw(Card,       0.25, {BackgroundColor3 = C.card})
            tw(cStroke,    0.25, {Color            = C.border})
            tw(NameL,      0.2,  {TextColor3       = C.text})
            tw(DescL,      0.2,  {TextColor3       = C.textDim})
            tw(IconBubble, 0.2,  {BackgroundColor3 = C.surface})
            tw(iBubStroke, 0.2,  {Color            = C.border})
            tw(PillBg,     0.25, {BackgroundColor3 = C.redDim})
            tw(PillDot,    0.25, {BackgroundColor3 = C.red, Position=UDim2.new(0,4,0.5,-9)})
            tw(PillLbl,    0.2,  {TextColor3=C.red, Position=UDim2.new(1,-26,0,0)})
            PillLbl.Text = "OFF"
            tw(LeftBar,    0.2,  {BackgroundTransparency=1})
        end
    end

    Card.Activated:Connect(function()
        isOn = not isOn
        _G[cfg.gvar] = isOn
        setVisual(isOn)
        tw(Card, 0.08, {Size=UDim2.new(1,0,0,60)})
        task.delay(0.1, function() tw(Card, 0.15, {Size=UDim2.new(1,0,0,64)}) end)
        if cfg.cb then cfg.cb(isOn) end
    end)

    Card.MouseEnter:Connect(function()
        if not isOn then tw(Card,0.15,{BackgroundColor3=C.cardHov}) end
    end)
    Card.MouseLeave:Connect(function()
        if not isOn then tw(Card,0.15,{BackgroundColor3=C.card}) end
    end)
end

createToggle({ name="AutoRebirth",    icon="⚡", label="Auto Rebirth",    desc="Rebirth automático ao atingir nivel necessário",       gvar="AutoRebirth",    order=1 })
createToggle({ name="AutoAura",       icon="✨", label="Auto Aura",        desc="Compra e equipa a melhor Aura disponível",  gvar="AutoAura",       order=2 })
createToggle({ name="AutoTrail",      icon="🌊", label="Auto Trail",       desc="Compra e equipa a melhor Trail disponível", gvar="AutoTrail",      order=3 })
createToggle({ name="AutoMonkeyTail", icon="🐒", label="Auto Monkey Tail", desc="Compra e equipa a melhor Cauda de Macaco",  gvar="AutoMonkeyTail", order=4 })

-- ══════════════════════════════════════════
-- MINIMIZE / RESTORE — sem drift, input correto
-- Fix: Main.ClipsDescendants=false e usamos
--      Main.Visible para esconder tudo junto
-- ══════════════════════════════════════════
local isMin    = false
local savedPos = Main.Position

local function setMin(state)
    isMin = state
    if state then
        savedPos = Main.Position
        tw(Main, 0.2, {
            Size     = UDim2.new(0,0,0,0),
            Position = UDim2.new(savedPos.X.Scale, savedPos.X.Offset+190,
                                 savedPos.Y.Scale, savedPos.Y.Offset+190)
        })
        task.delay(0.21, function()
            Main.Visible     = false
            MiniBtn.Position = UDim2.new(0,20, savedPos.Y.Scale, savedPos.Y.Offset+80)
            MiniBtn.Visible  = true
        end)
    else
        MiniBtn.Visible  = false
        Main.Visible     = true
        Main.Size        = UDim2.new(0,0,0,0)
        Main.Position    = UDim2.new(savedPos.X.Scale, savedPos.X.Offset+190,
                                     savedPos.Y.Scale, savedPos.Y.Offset+190)
        tw(Main, 0.28, { Size=UDim2.new(0,380,0,380), Position=savedPos })
    end
end

MinBtn2.Activated:Connect(function() setMin(true) end)
MiniBtn.Activated:Connect(function() setMin(false) end)
makeDraggable(TitleBar, Main)

-- ══════════════════════════════════════════
-- STATUS UPDATE
-- ══════════════════════════════════════════
local function fmt(n)
    if     n>=1e12 then return ("%.1fT"):format(n/1e12)
    elseif n>=1e9  then return ("%.1fB"):format(n/1e9)
    elseif n>=1e6  then return ("%.1fM"):format(n/1e6)
    elseif n>=1e3  then return ("%.1fK"):format(n/1e3)
    else                return tostring(math.floor(n)) end
end

local function updateStatus()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then StatusLbl.Text = "⏳  Aguardando dados..." return end
    local lv  = data:FindFirstChild("Level")    and data.Level.Value    or 0
    local rb  = data:FindFirstChild("Rebirths") and data.Rebirths.Value or 0
    local win = data:FindFirstChild("Wins")     and data.Wins.Value     or 0
    StatusLbl.Text      = ("Lv.%d   ·   %d/%d Rebirths   ·   %s Wins"):format(lv,rb,MAX_REBIRTHS,fmt(win))
    StatusLbl.TextColor3 = C.accent
end

-- ══════════════════════════════════════════
-- LÓGICAS AUTO
-- ══════════════════════════════════════════
local function autoRebirth()
    local d = LocalPlayer:FindFirstChild("Data"); if not d then return end
    if (d:FindFirstChild("Rebirths") and d.Rebirths.Value or 0) >= MAX_REBIRTHS then return end
    if (d:FindFirstChild("Level") and d.Level.Value or 0) >= REBIRTH_REQUIRED_LEVEL then
        Remotes.Rebirth:FireServer()
    end
end

local function autoBuyEquip(cfg, dataKey, buyRemote, equipRemote, equippedKey)
    local d = LocalPlayer:FindFirstChild("Data"); if not d then return end
    local wins     = d:FindFirstChild("Wins")     and d.Wins.Value     or 0
    local unlocked = d:FindFirstChild(dataKey)
    local equipped = d:FindFirstChild(equippedKey) and d[equippedKey].Value or ""
    local bestBuy, bestPrice = nil, -1
    for name, c in pairs(cfg) do
        local owned = unlocked and unlocked:FindFirstChild(name) ~= nil
        if not owned and wins >= c.Price and c.Price > bestPrice then
            bestPrice = c.Price; bestBuy = name
        end
    end
    if bestBuy then pcall(function() Remotes[buyRemote]:FireServer(bestBuy) end) task.wait(0.15) end
    local bestEquip, bestMulti = "", -1
    if unlocked then
        for _, child in ipairs(unlocked:GetChildren()) do
            local c = cfg[child.Name]
            if c and c.Multi > bestMulti then bestMulti=c.Multi; bestEquip=child.Name end
        end
    end
    if bestEquip ~= "" and bestEquip ~= equipped then
        pcall(function() Remotes[equipRemote]:FireServer(bestEquip) end)
    end
end

-- ══════════════════════════════════════════
-- MAIN LOOP
-- ══════════════════════════════════════════
task.spawn(function()
    while true do
        task.wait(0.8)
        pcall(updateStatus)
        if _G.AutoRebirth    then pcall(autoRebirth) end
        if _G.AutoAura       then pcall(autoBuyEquip, AurasConfig,       "UnlockedAuras",  "BuyAura",  "EquipAura",  "EquippedAura")  end
        if _G.AutoTrail      then pcall(autoBuyEquip, TrailsConfig,      "UnlockedTrails", "BuyTrail", "EquipTrail", "EquippedTrail") end
        if _G.AutoMonkeyTail then pcall(autoBuyEquip, MonkeyTailsConfig, "UnlockedTails",  "BuyTail",  "EquipTail",  "EquippedTail")  end
    end
end)

print("🔱 Titanium Hub V2 carregado!")
