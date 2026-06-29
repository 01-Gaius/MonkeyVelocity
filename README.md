-- 🔱 Titanium Hub | Speed Monkey Escape
-- v6 — UI reescrita, bugs corrigidos, Group ID real, posições atualizadas

local Players           = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService      = game:GetService("TweenService")
local UserInputService  = game:GetService("UserInputService")
local CoreGui           = game:GetService("CoreGui")
local RunService        = game:GetService("RunService")
local MPS               = game:GetService("MarketplaceService")
local VirtualUser       = game:GetService("VirtualUser")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")
local Remotes     = ReplicatedStorage:WaitForChild("Remotes")

-- ══════════════════════════════════════════
-- CONFIGS
-- ══════════════════════════════════════════
local AurasConfig = {
    ["Amber"]    = { Multi = 1.5,  Price = 5000           },
    ["Ice Cold"] = { Multi = 2,    Price = 150000         },
    ["Nature"]   = { Multi = 2.5,  Price = 4000000        },
    ["Rainbow"]  = { Multi = 3,    Price = 2500000000     },
    ["Lunar"]    = { Multi = 3.5,  Price = 90000000000    },
    ["Sparkle"]  = { Multi = 4,    Price = 5250000000000  },
}
local TrailsConfig = {
    ["Red"]     = { Multi = 1.5,  Price = 1000           },
    ["Blue"]    = { Multi = 2,    Price = 30000          },
    ["Green"]   = { Multi = 3,    Price = 750000         },
    ["Rainbow"] = { Multi = 5,    Price = 400000000      },
    ["Galaxy"]  = { Multi = 10,   Price = 18750000000    },
    ["Divine"]  = { Multi = 25,   Price = 600000000000   },
}
local MonkeyTailPriority = {
    { id = 6, name = "Divine Tail"   },
    { id = 5, name = "Galaxy Tail"   },
    { id = 4, name = "Thunder Tail"  },
    { id = 3, name = "Ice Tail"      },
    { id = 2, name = "Flame Tail"    },
    { id = 1, name = "Basic Tail"    },
}

-- GROUP ID real do jogo: 1008902725
-- Posições atualizadas via MCP (coordenadas reais do servidor)
local GAME_GROUP_ID = 1008902725

local TreadmillPriority = {
    { name = "TreadmillCelestial", pos = Vector3.new(-348, 24, -277), gamepassId = 0       },
    { name = "TreadmillVoid",      pos = Vector3.new(-384, 45, -304), gamepassId = 0       },
    { name = "TreadmillGalaxy",    pos = Vector3.new(-396, 30, -300), gamepassId = 0       },
    { name = "TreadmillDiamond",   pos = Vector3.new(-490, 23, -306), gamepassId = 0       },
    { name = "TreadmillPlaytime",  pos = Vector3.new(-456, 27, -294), groupId = GAME_GROUP_ID },
    { name = "TreadmillGold",      pos = Vector3.new(-416, 24, -288), gamepassId = 0       },
    { name = "TreadmillBasic",     pos = Vector3.new(-440, 24, -288), free = true          },
}

local REBIRTH_REQUIRED_LEVEL = 25
local MAX_REBIRTHS            = 11

-- ══════════════════════════════════════════
-- ESTADOS
-- ══════════════════════════════════════════
_G.AutoRebirth    = false
_G.AutoAura       = false
_G.AutoTrail      = false
_G.AutoMonkeyTail = false
_G.AutoEsteira    = false

-- ══════════════════════════════════════════
-- AUTO ESTEIRA
-- ══════════════════════════════════════════
local antiAfkConn   = nil
local charAddedConn = nil

local function hasAccess(t)
    if t.free then return true end
    if t.groupId and t.groupId ~= 0 then
        local ok, v = pcall(function() return LocalPlayer:IsInGroup(t.groupId) end)
        return ok and v
    end
    if t.gamepassId and t.gamepassId ~= 0 then
        local ok, v = pcall(function() return MPS:UserOwnsGamePassAsync(LocalPlayer.UserId, t.gamepassId) end)
        return ok and v
    end
    return false
end

local function getBest()
    for _, t in ipairs(TreadmillPriority) do
        if hasAccess(t) then return t end
    end
    return TreadmillPriority[#TreadmillPriority]
end

local function goTo(t)
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = CFrame.new(t.pos + Vector3.new(0, 3, 0))
    end
end

local function resetChar()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then hum.Health = 0 end
end

local function startAntiAFK()
    if antiAfkConn then antiAfkConn:Disconnect() end
    antiAfkConn = RunService.Heartbeat:Connect(function() end)
    task.spawn(function()
        while _G.AutoEsteira do
            task.wait(55)
            if not _G.AutoEsteira then break end
            pcall(function()
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
        end
    end)
end

local function stopAntiAFK()
    if antiAfkConn then antiAfkConn:Disconnect() antiAfkConn = nil end
end

local function hookCharAdded()
    if charAddedConn then charAddedConn:Disconnect() end
    charAddedConn = LocalPlayer.CharacterAdded:Connect(function(char)
        if not _G.AutoEsteira then return end
        task.wait(2.5)
        char:WaitForChild("HumanoidRootPart", 6)
        local best = getBest()
        goTo(best)
    end)
end

local function startAutoEsteira()
    startAntiAFK()
    hookCharAdded()
    task.spawn(function()
        local best = getBest()
        resetChar()
        task.wait(3)
        local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        char:WaitForChild("HumanoidRootPart", 6)
        task.wait(0.5)
        goTo(best)
    end)
end

local function stopAutoEsteira()
    stopAntiAFK()
    if charAddedConn then charAddedConn:Disconnect() charAddedConn = nil end
end

-- ══════════════════════════════════════════
-- GUI PARENT
-- ══════════════════════════════════════════
local targetParent
pcall(function()
    local t = Instance.new("Frame"); t.Parent = CoreGui; t:Destroy()
    targetParent = CoreGui
end)
if not targetParent then targetParent = PlayerGui end
if targetParent:FindFirstChild("TitaniumV6") then
    targetParent.TitaniumV6:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name           = "TitaniumV6"
ScreenGui.ResetOnSpawn   = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent         = targetParent

-- ══════════════════════════════════════════
-- PALETA
-- ══════════════════════════════════════════
local C = {
    bg        = Color3.fromRGB(8,  8,  14),
    surface   = Color3.fromRGB(14, 14, 22),
    card      = Color3.fromRGB(18, 18, 28),
    cardHov   = Color3.fromRGB(22, 22, 36),
    cardOn    = Color3.fromRGB(0,  22, 36),
    border    = Color3.fromRGB(36, 36, 56),
    borderAct = Color3.fromRGB(0,  160, 210),
    accent    = Color3.fromRGB(0,  200, 245),
    accentDim = Color3.fromRGB(0,  80,  110),
    text      = Color3.fromRGB(210,210,230),
    textDim   = Color3.fromRGB(65, 65, 95),
    textMid   = Color3.fromRGB(120,120,155),
    green     = Color3.fromRGB(0,  215,125),
    greenDim  = Color3.fromRGB(0,  50,  35),
    red       = Color3.fromRGB(215, 55, 75),
    redDim    = Color3.fromRGB(55,  12, 18),
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
    s.Color     = col   or C.border
    s.Thickness = thick or 1.2
    s.Parent    = parent
    return s
end

-- ══════════════════════════════════════════
-- DRAG — versão limpa, sem acumular listeners
-- Usa uma única conexão global em vez de por-objeto
-- ══════════════════════════════════════════
local dragState = {
    active = false,
    target = nil,
    startInput = Vector3.new(),
    originPos  = UDim2.new(),
}

-- Conexão global única (não acumula)
UserInputService.InputChanged:Connect(function(i)
    if not dragState.active then return end
    if i.UserInputType ~= Enum.UserInputType.MouseMovement
    and i.UserInputType ~= Enum.UserInputType.Touch then return end
    local d = i.Position - dragState.startInput
    dragState.target.Position = UDim2.new(
        dragState.originPos.X.Scale,
        dragState.originPos.X.Offset + d.X,
        dragState.originPos.Y.Scale,
        dragState.originPos.Y.Offset + d.Y
    )
end)

UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1
    or i.UserInputType == Enum.UserInputType.Touch then
        dragState.active = false
        dragState.target = nil
    end
end)

local function makeDraggable(handle, target)
    handle.InputBegan:Connect(function(i)
        if dragState.active then return end
        if i.UserInputType ~= Enum.UserInputType.MouseButton1
        and i.UserInputType ~= Enum.UserInputType.Touch then return end
        dragState.active    = true
        dragState.target    = target
        dragState.startInput = i.Position
        dragState.originPos  = target.Position
    end)
end

-- ══════════════════════════════════════════
-- MINI BUTTON (mostrado quando minimizado)
-- ══════════════════════════════════════════
local MiniBtn = Instance.new("TextButton")
MiniBtn.Size             = UDim2.new(0, 52, 0, 52)
MiniBtn.Position         = UDim2.new(0, 20, 0.5, -26)
MiniBtn.BackgroundColor3 = C.bg
MiniBtn.Text             = "🔱"
MiniBtn.TextSize         = 22
MiniBtn.Font             = Enum.Font.GothamBold
MiniBtn.TextColor3       = C.accent
MiniBtn.AutoButtonColor  = false
MiniBtn.Visible          = false
MiniBtn.ZIndex           = 20
MiniBtn.Parent           = ScreenGui
corner(MiniBtn, 14)
local miniStroke = stroke(MiniBtn, C.accentDim, 1.5)

MiniBtn.MouseEnter:Connect(function()
    tw(MiniBtn,    0.15, {BackgroundColor3 = C.accentDim})
    tw(miniStroke, 0.15, {Color = C.accent})
end)
MiniBtn.MouseLeave:Connect(function()
    tw(MiniBtn,    0.15, {BackgroundColor3 = C.bg})
    tw(miniStroke, 0.15, {Color = C.accentDim})
end)
makeDraggable(MiniBtn, MiniBtn)

-- ══════════════════════════════════════════
-- JANELA PRINCIPAL
-- ══════════════════════════════════════════
local Main = Instance.new("Frame")
Main.Name             = "Main"
Main.Size             = UDim2.new(0, 375, 0, 450)
Main.Position         = UDim2.new(0, 120, 0.5, -225)
Main.BackgroundColor3 = C.bg
Main.BorderSizePixel  = 0
Main.ClipsDescendants = true
Main.ZIndex           = 5
Main.Parent           = ScreenGui
corner(Main, 16)
stroke(Main, C.border, 1.5)

-- Gradiente de fundo
local BgGrad = Instance.new("UIGradient", Main)
BgGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(11, 11, 18)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(7,  7,  12)),
})
BgGrad.Rotation = 135

-- Barra topo decorativa
local TopBar = newFrame(Main, {
    Size             = UDim2.new(0.8, 0, 0, 2),
    Position         = UDim2.new(0.1, 0, 0, 0),
    BackgroundColor3 = C.accent,
    ZIndex           = 10,
})
corner(TopBar, 2)
local tg = Instance.new("UIGradient", TopBar)
tg.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0,   Color3.fromRGB(0, 140, 190)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 220, 255)),
    ColorSequenceKeypoint.new(1,   Color3.fromRGB(0, 140, 190)),
})

-- ══════════════════════════════════════════
-- TITLE BAR
-- ══════════════════════════════════════════
local TitleBar = newFrame(Main, { Size = UDim2.new(1, 0, 0, 58), ZIndex = 9 })

-- Ícone
local IconBg = Instance.new("Frame", TitleBar)
IconBg.Size             = UDim2.new(0, 38, 0, 38)
IconBg.Position         = UDim2.new(0, 14, 0.5, -19)
IconBg.BackgroundColor3 = C.accentDim
IconBg.BorderSizePixel  = 0
IconBg.ZIndex           = 10
corner(IconBg, 10)
stroke(IconBg, C.accent, 1)
newLabel(IconBg, {
    Size     = UDim2.new(1, 0, 1, 0),
    Text     = "🔱",
    TextSize = 18,
    Font     = Enum.Font.GothamBold,
    ZIndex   = 11,
})

-- Título
local TitleLbl = newLabel(TitleBar, {
    Size              = UDim2.new(0, 130, 0, 20),
    Position          = UDim2.new(0, 60, 0.5, -21),
    Text              = "TITANIUM HUB",
    TextColor3        = C.accent,
    TextSize          = 14,
    Font              = Enum.Font.GothamBold,
    TextXAlignment    = Enum.TextXAlignment.Left,
    ZIndex            = 10,
})
local tgrad = Instance.new("UIGradient", TitleLbl)
tgrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 230, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 155, 200)),
})

newLabel(TitleBar, {
    Size           = UDim2.new(0, 220, 0, 13),
    Position       = UDim2.new(0, 60, 0.5, 3),
    Text           = "Speed Monkey Escape  •  v6",
    TextColor3     = C.textDim,
    TextSize       = 10,
    Font           = Enum.Font.GothamMedium,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex         = 10,
})

-- Badge versão
local Badge = Instance.new("Frame", TitleBar)
Badge.Size             = UDim2.new(0, 40, 0, 18)
Badge.Position         = UDim2.new(1, -104, 0.5, -9)
Badge.BackgroundColor3 = Color3.fromRGB(0, 30, 45)
Badge.BorderSizePixel  = 0
Badge.ZIndex           = 10
corner(Badge, 5)
stroke(Badge, C.accentDim, 1)
newLabel(Badge, {
    Size       = UDim2.new(1, 0, 1, 0),
    Text       = "V6",
    TextColor3 = C.accent,
    TextSize   = 10,
    Font       = Enum.Font.GothamBold,
    ZIndex     = 11,
})

-- Botão minimizar
local MinBtn = Instance.new("TextButton", TitleBar)
MinBtn.Size             = UDim2.new(0, 26, 0, 26)
MinBtn.Position         = UDim2.new(1, -40, 0.5, -13)
MinBtn.BackgroundColor3 = C.surface
MinBtn.Text             = "–"
MinBtn.TextSize         = 13
MinBtn.Font             = Enum.Font.GothamBold
MinBtn.TextColor3       = C.textMid
MinBtn.AutoButtonColor  = false
MinBtn.ZIndex           = 11
corner(MinBtn, 7)
stroke(MinBtn, C.border, 1)
MinBtn.MouseEnter:Connect(function() tw(MinBtn, 0.1, {BackgroundColor3 = C.cardHov, TextColor3 = C.text}) end)
MinBtn.MouseLeave:Connect(function() tw(MinBtn, 0.1, {BackgroundColor3 = C.surface, TextColor3 = C.textMid}) end)

-- Divisor
newFrame(Main, {
    Size             = UDim2.new(1, -28, 0, 1),
    Position         = UDim2.new(0, 14, 0, 58),
    BackgroundColor3 = C.border,
    ZIndex           = 9,
})

-- ══════════════════════════════════════════
-- STATUS CARD
-- ══════════════════════════════════════════
local StatCard = Instance.new("Frame", Main)
StatCard.Size             = UDim2.new(1, -28, 0, 32)
StatCard.Position         = UDim2.new(0, 14, 0, 66)
StatCard.BackgroundColor3 = C.surface
StatCard.BorderSizePixel  = 0
StatCard.ZIndex           = 9
corner(StatCard, 8)
stroke(StatCard, C.border, 1)

local StatPulse = newFrame(StatCard, {
    Size             = UDim2.new(0, 3, 0.55, 0),
    Position         = UDim2.new(0, 9, 0.225, 0),
    BackgroundColor3 = C.accent,
    ZIndex           = 10,
})
corner(StatPulse, 2)

local StatusLbl = newLabel(StatCard, {
    Size           = UDim2.new(1, -24, 1, 0),
    Position       = UDim2.new(0, 18, 0, 0),
    Text           = "⏳  Aguardando dados...",
    TextColor3     = C.textDim,
    TextSize       = 10,
    Font           = Enum.Font.GothamMedium,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex         = 10,
})

-- Pulso do indicator
task.spawn(function()
    while true do
        tw(StatPulse, 0.8, {BackgroundColor3 = C.accent})
        task.wait(0.9)
        tw(StatPulse, 0.8, {BackgroundColor3 = C.accentDim})
        task.wait(0.9)
    end
end)

-- ══════════════════════════════════════════
-- SCROLL
-- ══════════════════════════════════════════
local ScrollFrame = Instance.new("ScrollingFrame", Main)
ScrollFrame.Size                 = UDim2.new(1, -28, 1, -110)
ScrollFrame.Position             = UDim2.new(0, 14, 0, 106)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.BorderSizePixel      = 0
ScrollFrame.ScrollBarThickness   = 3
ScrollFrame.ScrollBarImageColor3 = C.accentDim
ScrollFrame.CanvasSize           = UDim2.new(0, 0, 0, 0)
ScrollFrame.AutomaticCanvasSize  = Enum.AutomaticSize.Y
ScrollFrame.ZIndex               = 9
ScrollFrame.Parent               = Main

local ListLayout = Instance.new("UIListLayout", ScrollFrame)
ListLayout.SortOrder = Enum.SortOrder.LayoutOrder
ListLayout.Padding   = UDim.new(0, 5)

-- Cabeçalho seção
local SecFrame = newFrame(ScrollFrame, { Size = UDim2.new(1, 0, 0, 18), ZIndex = 9 })
SecFrame.LayoutOrder = 0
newLabel(SecFrame, {
    Size           = UDim2.new(1, 0, 1, 0),
    Position       = UDim2.new(0, 0, 0, 0),
    Text           = "AUTOMAÇÕES",
    TextColor3     = C.textDim,
    TextSize       = 9,
    Font           = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex         = 10,
})

-- ══════════════════════════════════════════
-- TOGGLE FACTORY — limpo e polido
-- ══════════════════════════════════════════
local function createToggle(cfg)
    local Card = Instance.new("TextButton", ScrollFrame)
    Card.Name             = cfg.name .. "Card"
    Card.LayoutOrder      = cfg.order
    Card.Size             = UDim2.new(1, 0, 0, 60)
    Card.BackgroundColor3 = C.card
    Card.Text             = ""
    Card.AutoButtonColor  = false
    Card.BorderSizePixel  = 0
    Card.ZIndex           = 10
    corner(Card, 10)
    local cStroke = stroke(Card, C.border, 1.2)

    -- Barra lateral (visível quando ON)
    local LeftBar = newFrame(Card, {
        Size             = UDim2.new(0, 3, 0.55, 0),
        Position         = UDim2.new(0, 0, 0.225, 0),
        BackgroundColor3 = C.accent,
        ZIndex           = 11,
    })
    corner(LeftBar, 2)
    LeftBar.BackgroundTransparency = 1

    -- Ícone
    local IB = Instance.new("Frame", Card)
    IB.Size             = UDim2.new(0, 36, 0, 36)
    IB.Position         = UDim2.new(0, 12, 0.5, -18)
    IB.BackgroundColor3 = C.surface
    IB.BorderSizePixel  = 0
    IB.ZIndex           = 11
    corner(IB, 9)
    local ibS = stroke(IB, C.border, 1)
    newLabel(IB, {
        Size     = UDim2.new(1, 0, 1, 0),
        Text     = cfg.icon,
        TextSize = 17,
        Font     = Enum.Font.GothamBold,
        ZIndex   = 12,
    })

    -- Labels
    local NL = newLabel(Card, {
        Size           = UDim2.new(0.5, 0, 0, 18),
        Position       = UDim2.new(0, 56, 0.5, -20),
        Text           = cfg.label,
        TextColor3     = C.text,
        TextSize       = 12,
        Font           = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex         = 11,
    })
    local DL = newLabel(Card, {
        Size           = UDim2.new(0.6, 0, 0, 13),
        Position       = UDim2.new(0, 56, 0.5, 1),
        Text           = cfg.desc,
        TextColor3     = C.textDim,
        TextSize       = 9,
        Font           = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex         = 11,
    })

    -- Toggle pill
    local PB = Instance.new("Frame", Card)
    PB.Size             = UDim2.new(0, 48, 0, 24)
    PB.Position         = UDim2.new(1, -60, 0.5, -12)
    PB.BackgroundColor3 = C.redDim
    PB.BorderSizePixel  = 0
    PB.ZIndex           = 11
    corner(PB, 12)
    stroke(PB, C.border, 1)

    local PD = Instance.new("Frame", PB)
    PD.Size             = UDim2.new(0, 16, 0, 16)
    PD.Position         = UDim2.new(0, 4, 0.5, -8)
    PD.BackgroundColor3 = C.red
    PD.BorderSizePixel  = 0
    PD.ZIndex           = 12
    corner(PD, 8)

    local PL = newLabel(PB, {
        Size       = UDim2.new(0, 22, 1, 0),
        Position   = UDim2.new(1, -24, 0, 0),
        Text       = "OFF",
        TextColor3 = C.red,
        TextSize   = 8,
        Font       = Enum.Font.GothamBold,
        ZIndex     = 12,
    })

    local isOn = false

    local function setVisual(on)
        if on then
            tw(Card,   0.22, {BackgroundColor3 = C.cardOn})
            tw(cStroke,0.22, {Color = C.borderAct})
            tw(NL,     0.18, {TextColor3 = C.accent})
            tw(DL,     0.18, {TextColor3 = C.accentDim})
            tw(IB,     0.18, {BackgroundColor3 = C.accentDim})
            tw(ibS,    0.18, {Color = C.accent})
            tw(PB,     0.22, {BackgroundColor3 = C.greenDim})
            tw(PD,     0.22, {BackgroundColor3 = C.green, Position = UDim2.new(1, -20, 0.5, -8)})
            tw(PL,     0.18, {TextColor3 = C.green, Position = UDim2.new(0, 4, 0, 0)})
            PL.Text = "ON"
            tw(LeftBar,0.18, {BackgroundTransparency = 0})
        else
            tw(Card,   0.22, {BackgroundColor3 = C.card})
            tw(cStroke,0.22, {Color = C.border})
            tw(NL,     0.18, {TextColor3 = C.text})
            tw(DL,     0.18, {TextColor3 = C.textDim})
            tw(IB,     0.18, {BackgroundColor3 = C.surface})
            tw(ibS,    0.18, {Color = C.border})
            tw(PB,     0.22, {BackgroundColor3 = C.redDim})
            tw(PD,     0.22, {BackgroundColor3 = C.red,  Position = UDim2.new(0, 4, 0.5, -8)})
            tw(PL,     0.18, {TextColor3 = C.red,  Position = UDim2.new(1, -24, 0, 0)})
            PL.Text = "OFF"
            tw(LeftBar,0.18, {BackgroundTransparency = 1})
        end
    end

    Card.Activated:Connect(function()
        isOn = not isOn
        _G[cfg.gvar] = isOn
        setVisual(isOn)
        -- Feedback de clique
        tw(Card, 0.07, {Size = UDim2.new(1, 0, 0, 56)})
        task.delay(0.09, function() tw(Card, 0.14, {Size = UDim2.new(1, 0, 0, 60)}) end)
        if cfg.cb then cfg.cb(isOn) end
    end)

    Card.MouseEnter:Connect(function()
        if not isOn then tw(Card, 0.14, {BackgroundColor3 = C.cardHov}) end
    end)
    Card.MouseLeave:Connect(function()
        if not isOn then tw(Card, 0.14, {BackgroundColor3 = C.card}) end
    end)
end

-- Cria os toggles
createToggle({
    name  = "AutoRebirth",
    icon  = "⚡",
    label = "Auto Rebirth",
    desc  = "Rebirth ao atingir Lv.25",
    gvar  = "AutoRebirth",
    order = 1,
})
createToggle({
    name  = "AutoAura",
    icon  = "✨",
    label = "Auto Aura",
    desc  = "Compra e equipa a melhor Aura",
    gvar  = "AutoAura",
    order = 2,
})
createToggle({
    name  = "AutoTrail",
    icon  = "🌊",
    label = "Auto Trail",
    desc  = "Compra e equipa a melhor Trail",
    gvar  = "AutoTrail",
    order = 3,
})
createToggle({
    name  = "AutoMonkeyTail",
    icon  = "🐒",
    label = "Auto Monkey Tail",
    desc  = "Seleciona a melhor Cauda desbloqueada",
    gvar  = "AutoMonkeyTail",
    order = 4,
})
createToggle({
    name  = "AutoEsteira",
    icon  = "🏃",
    label = "Auto Esteira",
    desc  = "Melhor esteira disponível  •  Anti AFK",
    gvar  = "AutoEsteira",
    order = 5,
    cb    = function(s)
        if s then startAutoEsteira() else stopAutoEsteira() end
    end,
})

-- ══════════════════════════════════════════
-- MINIMIZE / RESTORE — corrigido
-- Salva/restaura posição real sem calcular offset errado
-- ══════════════════════════════════════════
local isMin       = false
local savedMainPos = Main.Position  -- posição salva antes de minimizar

local function setMin(state)
    isMin = state
    dragState.active = false  -- cancela drag em andamento

    if state then
        savedMainPos = Main.Position

        -- Anima saída da janela
        tw(Main, 0.2, {
            Size     = UDim2.new(0, 0, 0, 0),
            Position = UDim2.new(
                savedMainPos.X.Scale,
                savedMainPos.X.Offset + 187,
                savedMainPos.Y.Scale,
                savedMainPos.Y.Offset + 225
            ),
        })

        task.delay(0.21, function()
            Main.Visible = false
            -- Posiciona MiniBtn próximo ao centro onde a janela estava
            MiniBtn.Position = UDim2.new(
                savedMainPos.X.Scale,
                savedMainPos.X.Offset,
                savedMainPos.Y.Scale,
                savedMainPos.Y.Offset + 180
            )
            MiniBtn.Visible = true
        end)
    else
        MiniBtn.Visible = false
        Main.Visible    = true

        -- Restaura janela a partir da posição do MiniBtn
        local restorePos = UDim2.new(
            MiniBtn.Position.X.Scale,
            MiniBtn.Position.X.Offset,
            MiniBtn.Position.Y.Scale,
            MiniBtn.Position.Y.Offset - 180
        )

        Main.Size     = UDim2.new(0, 0, 0, 0)
        Main.Position = UDim2.new(
            restorePos.X.Scale,
            restorePos.X.Offset + 187,
            restorePos.Y.Scale,
            restorePos.Y.Offset + 225
        )
        tw(Main, 0.28, {
            Size     = UDim2.new(0, 375, 0, 450),
            Position = restorePos,
        })
    end
end

MinBtn.Activated:Connect(function() setMin(true) end)
MiniBtn.Activated:Connect(function() setMin(false) end)
makeDraggable(TitleBar, Main)

-- ══════════════════════════════════════════
-- STATUS UPDATE
-- ══════════════════════════════════════════
local function fmt(n)
    if     n >= 1e12 then return ("%.1fT"):format(n / 1e12)
    elseif n >= 1e9  then return ("%.1fB"):format(n / 1e9)
    elseif n >= 1e6  then return ("%.1fM"):format(n / 1e6)
    elseif n >= 1e3  then return ("%.1fK"):format(n / 1e3)
    else                  return tostring(math.floor(n)) end
end

local function updateStatus()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then
        StatusLbl.Text      = "⏳  Aguardando dados..."
        StatusLbl.TextColor3 = C.textDim
        return
    end
    local lv  = data:FindFirstChild("Level")    and data.Level.Value    or 0
    local rb  = data:FindFirstChild("Rebirths") and data.Rebirths.Value or 0
    local win = data:FindFirstChild("Wins")     and data.Wins.Value     or 0
    local best = getBest()
    local estInfo = _G.AutoEsteira and ("  •  🏃 " .. best.name) or ""
    StatusLbl.Text       = ("Lv.%d  ·  %d/%d RB  ·  %s W%s"):format(lv, rb, MAX_REBIRTHS, fmt(win), estInfo)
    StatusLbl.TextColor3 = C.accent
end

-- ══════════════════════════════════════════
-- LÓGICAS AUTO
-- ══════════════════════════════════════════
local function autoRebirth()
    local d = LocalPlayer:FindFirstChild("Data")
    if not d then return end
    local rb = d:FindFirstChild("Rebirths") and d.Rebirths.Value or 0
    local lv = d:FindFirstChild("Level")    and d.Level.Value    or 0
    if rb >= MAX_REBIRTHS then return end
    if lv >= REBIRTH_REQUIRED_LEVEL then
        Remotes.Rebirth:FireServer()
    end
end

local function autoBuyEquip(cfg, dataKey, buyRemote, equipRemote, equippedKey)
    local d = LocalPlayer:FindFirstChild("Data")
    if not d then return end
    local wins     = d:FindFirstChild("Wins")      and d.Wins.Value      or 0
    local unlocked = d:FindFirstChild(dataKey)
    local equipped = d:FindFirstChild(equippedKey) and d[equippedKey].Value or ""

    -- Tenta comprar o melhor acessível ainda não possuído
    local bestBuy, bestPrice = nil, -1
    for name, c in pairs(cfg) do
        local owned = unlocked and unlocked:FindFirstChild(name) ~= nil
        if not owned and wins >= c.Price and c.Price > bestPrice then
            bestPrice = c.Price
            bestBuy   = name
        end
    end
    if bestBuy then
        pcall(function() Remotes[buyRemote]:FireServer(bestBuy) end)
        task.wait(0.2)
    end

    -- Equipa o de maior multiplicador possuído
    local bestEquip, bestMulti = "", -1
    if unlocked then
        for _, child in ipairs(unlocked:GetChildren()) do
            local c = cfg[child.Name]
            if c and c.Multi > bestMulti then
                bestMulti = c.Multi
                bestEquip = child.Name
            end
        end
    end
    if bestEquip ~= "" and bestEquip ~= equipped then
        pcall(function() Remotes[equipRemote]:FireServer(bestEquip) end)
    end
end

local function autoMonkeyTail()
    local d = LocalPlayer:FindFirstChild("Data")
    if not d then return end
    local unlocked   = d:FindFirstChild("UnlockedUpgrades")
    local equippedObj = d:FindFirstChild("SelectedUpgrade")
    local equippedId = equippedObj and equippedObj.Value or 0

    local bestId = 0
    for _, t in ipairs(MonkeyTailPriority) do
        if unlocked and unlocked:FindFirstChild(tostring(t.id)) then
            bestId = t.id
            break
        end
    end

    if bestId > 0 and bestId ~= equippedId then
        pcall(function() Remotes.SelectUpgrade:FireServer(bestId) end)
    end
end

-- ══════════════════════════════════════════
-- MAIN LOOP
-- ══════════════════════════════════════════
task.spawn(function()
    while true do
        task.wait(0.8)
        pcall(updateStatus)
        if _G.AutoRebirth    then pcall(autoRebirth)  end
        if _G.AutoAura       then pcall(autoBuyEquip, AurasConfig,  "UnlockedAuras",  "BuyAura",  "EquipAura",  "EquippedAura")  end
        if _G.AutoTrail      then pcall(autoBuyEquip, TrailsConfig, "UnlockedTrails", "BuyTrail", "EquipTrail", "EquippedTrail") end
        if _G.AutoMonkeyTail then pcall(autoMonkeyTail) end
    end
end)

print("🔱 Titanium Hub v6 carregado — Group ID: " .. GAME_GROUP_ID)
