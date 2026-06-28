-- 🔱 Gaius Hub | Speed Monkey Escape
-- Corrigido com base nos dados reais do jogo via MCP
-- Remotes, configs e lógica de rebirth verificados in-game

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Remotes = ReplicatedStorage:WaitForChild("Remotes")

-- ══════════════════════════════════════════
-- CONFIGS REAIS (verificadas via decompile)
-- ══════════════════════════════════════════
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

-- FIX: Rebirth requer level 25 fixo (Config.Rebirths.RequiredLevel = 25)
-- Não é 25 * (rebirths + 1) — isso estava ERRADO no script original
local REBIRTH_REQUIRED_LEVEL = 25
local MAX_REBIRTHS = 11 -- Config.Rebirths.MaxRebirths

-- ══════════════════════════════════════════
-- ESTADOS GLOBAIS
-- ══════════════════════════════════════════
_G.AutoRebirth = false
_G.AutoAura    = false
_G.AutoTrail   = false

-- ══════════════════════════════════════════
-- PARENT DA GUI
-- ══════════════════════════════════════════
local targetParent
local ok = pcall(function()
    -- Teste se temos acesso de escrita ao CoreGui
    local t = Instance.new("Frame")
    t.Parent = CoreGui
    t:Destroy()
    targetParent = CoreGui
end)
if not ok or not targetParent then
    targetParent = PlayerGui
end

-- Limpar instância anterior
local existing = targetParent:FindFirstChild("GaiusHubGui")
if existing then existing:Destroy() end

-- ══════════════════════════════════════════
-- GUI
-- ══════════════════════════════════════════
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GaiusHubGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = targetParent

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 360, 0, 290)
MainFrame.Position = UDim2.new(0.5, -180, 0.5, -145)
MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = Color3.fromRGB(55, 55, 80)
MainStroke.Thickness = 1.5

local MainGradient = Instance.new("UIGradient", MainFrame)
MainGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(28, 28, 38)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(14, 14, 20)),
})
MainGradient.Rotation = 45

-- Título
local TitleBar = Instance.new("Frame", MainFrame)
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 48)
TitleBar.BackgroundTransparency = 1

local TitleText = Instance.new("TextLabel", TitleBar)
TitleText.Size = UDim2.new(0.75, 0, 1, 0)
TitleText.Position = UDim2.new(0, 14, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "🔱 Titanium | Monkey Escape"
TitleText.TextColor3 = Color3.fromRGB(0, 229, 255)
TitleText.TextSize = 17
TitleText.Font = Enum.Font.GothamBold
TitleText.TextXAlignment = Enum.TextXAlignment.Left

-- Status de Rebirths no título
local RebirthStatus = Instance.new("TextLabel", TitleBar)
RebirthStatus.Size = UDim2.new(1, -14, 0, 16)
RebirthStatus.Position = UDim2.new(0, 14, 1, -20)
RebirthStatus.BackgroundTransparency = 1
RebirthStatus.Text = "Carregando dados..."
RebirthStatus.TextColor3 = Color3.fromRGB(130, 130, 160)
RebirthStatus.TextSize = 11
RebirthStatus.Font = Enum.Font.Gotham
RebirthStatus.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Minimizar
local MinimizeBtn = Instance.new("TextButton", TitleBar)
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Position = UDim2.new(1, -42, 0.5, -15)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(32, 32, 44)
MinimizeBtn.Text = "▲"
MinimizeBtn.TextColor3 = Color3.fromRGB(200, 200, 220)
MinimizeBtn.TextSize = 13
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.AutoButtonColor = false
Instance.new("UICorner", MinimizeBtn).CornerRadius = UDim.new(0, 6)
local MinStroke = Instance.new("UIStroke", MinimizeBtn)
MinStroke.Color = Color3.fromRGB(60, 60, 85)
MinStroke.Thickness = 1

-- Divisor
local Divider = Instance.new("Frame", MainFrame)
Divider.Size = UDim2.new(1, -24, 0, 1)
Divider.Position = UDim2.new(0, 12, 0, 48)
Divider.BackgroundColor3 = Color3.fromRGB(55, 55, 80)
Divider.BorderSizePixel = 0

-- Content Frame
local ContentFrame = Instance.new("Frame", MainFrame)
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -28, 1, -64)
ContentFrame.Position = UDim2.new(0, 14, 0, 58)
ContentFrame.BackgroundTransparency = 1

local ContentLayout = Instance.new("UIListLayout", ContentFrame)
ContentLayout.SortOrder = Enum.SortOrder.LayoutOrder
ContentLayout.Padding = UDim.new(0, 10)

-- ══════════════════════════════════════════
-- FACTORY: TOGGLE BUTTON
-- ══════════════════════════════════════════
local function createToggle(name, labelText, descText, globalVarName, order)
    local Button = Instance.new("TextButton", ContentFrame)
    Button.Name = name .. "Toggle"
    Button.LayoutOrder = order
    Button.Size = UDim2.new(1, 0, 0, 54)
    Button.BackgroundColor3 = Color3.fromRGB(26, 26, 36)
    Button.Text = ""
    Button.AutoButtonColor = false

    Instance.new("UICorner", Button).CornerRadius = UDim.new(0, 8)
    local BtnStroke = Instance.new("UIStroke", Button)
    BtnStroke.Color = Color3.fromRGB(48, 48, 68)
    BtnStroke.Thickness = 1

    -- Label principal
    local Label = Instance.new("TextLabel", Button)
    Label.Size = UDim2.new(0.65, 0, 0, 20)
    Label.Position = UDim2.new(0, 14, 0, 9)
    Label.BackgroundTransparency = 1
    Label.Text = labelText
    Label.TextColor3 = Color3.fromRGB(215, 215, 228)
    Label.TextSize = 14
    Label.Font = Enum.Font.GothamMedium
    Label.TextXAlignment = Enum.TextXAlignment.Left

    -- Descrição
    local Desc = Instance.new("TextLabel", Button)
    Desc.Size = UDim2.new(0.65, 0, 0, 16)
    Desc.Position = UDim2.new(0, 14, 0, 30)
    Desc.BackgroundTransparency = 1
    Desc.Text = descText
    Desc.TextColor3 = Color3.fromRGB(90, 90, 115)
    Desc.TextSize = 11
    Desc.Font = Enum.Font.Gotham
    Desc.TextXAlignment = Enum.TextXAlignment.Left

    -- Status
    local StatusText = Instance.new("TextLabel", Button)
    StatusText.Size = UDim2.new(0.32, 0, 1, 0)
    StatusText.Position = UDim2.new(0.68, 0, 0, 0)
    StatusText.BackgroundTransparency = 1
    StatusText.Text = "OFF"
    StatusText.TextColor3 = Color3.fromRGB(220, 70, 60)
    StatusText.TextSize = 13
    StatusText.Font = Enum.Font.GothamBold
    StatusText.TextXAlignment = Enum.TextXAlignment.Right

    local function updateVisual(isActive)
        local bg     = isActive and Color3.fromRGB(0, 180, 210)   or Color3.fromRGB(26, 26, 36)
        local lCol   = isActive and Color3.fromRGB(8, 8, 14)      or Color3.fromRGB(215, 215, 228)
        local dCol   = isActive and Color3.fromRGB(8, 8, 14)      or Color3.fromRGB(90, 90, 115)
        local sCol   = isActive and Color3.fromRGB(0, 229, 255)   or Color3.fromRGB(48, 48, 68)
        local stCol  = isActive and Color3.fromRGB(8, 8, 14)      or Color3.fromRGB(220, 70, 60)
        local stTxt  = isActive and "ON" or "OFF"

        TweenService:Create(Button,     TweenInfo.new(0.18), { BackgroundColor3 = bg   }):Play()
        TweenService:Create(Label,      TweenInfo.new(0.18), { TextColor3 = lCol       }):Play()
        TweenService:Create(Desc,       TweenInfo.new(0.18), { TextColor3 = dCol       }):Play()
        TweenService:Create(BtnStroke,  TweenInfo.new(0.18), { Color = sCol            }):Play()
        TweenService:Create(StatusText, TweenInfo.new(0.18), { TextColor3 = stCol      }):Play()
        StatusText.Text = stTxt
    end

    Button.Activated:Connect(function()
        _G[globalVarName] = not _G[globalVarName]
        updateVisual(_G[globalVarName])
    end)

    Button.MouseEnter:Connect(function()
        if not _G[globalVarName] then
            TweenService:Create(BtnStroke, TweenInfo.new(0.12), { Color = Color3.fromRGB(0, 180, 210) }):Play()
        end
    end)
    Button.MouseLeave:Connect(function()
        if not _G[globalVarName] then
            TweenService:Create(BtnStroke, TweenInfo.new(0.12), { Color = Color3.fromRGB(48, 48, 68) }):Play()
        end
    end)
end

createToggle("AutoRebirth", "⚡ Auto Rebirth", "Rebirths ao atingir Lv.25",  "AutoRebirth", 1)
createToggle("AutoAura",    "✨ Auto Aura",    "Compra e equipa a melhor Aura", "AutoAura",    2)
createToggle("AutoTrail",   "🌊 Auto Trail",   "Compra e equipa a melhor Trail","AutoTrail",   3)

-- ══════════════════════════════════════════
-- MINIMIZAR
-- ══════════════════════════════════════════
local isMinimized = false
MinimizeBtn.Activated:Connect(function()
    isMinimized = not isMinimized
    local h = isMinimized and 48 or 290
    MinimizeBtn.Text = isMinimized and "▼" or "▲"
    ContentFrame.Visible = not isMinimized
    Divider.Visible = not isMinimized
    RebirthStatus.Visible = not isMinimized
    TweenService:Create(MainFrame, TweenInfo.new(0.28, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 360, 0, h)
    }):Play()
end)

-- ══════════════════════════════════════════
-- DRAG SYSTEM
-- ══════════════════════════════════════════
local dragging, dragInput, dragStart, startPos

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

-- ══════════════════════════════════════════
-- HELPER: ATUALIZAR STATUS NO TÍTULO
-- ══════════════════════════════════════════
local function updateStatusLabel()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then
        RebirthStatus.Text = "Aguardando dados..."
        return
    end
    local level    = data:FindFirstChild("Level")    and data.Level.Value    or 0
    local rebirths = data:FindFirstChild("Rebirths") and data.Rebirths.Value or 0
    local wins     = data:FindFirstChild("Wins")     and data.Wins.Value     or 0
    RebirthStatus.Text = string.format(
        "Lv.%d | %d Rebirths | %s Wins",
        level, rebirths,
        wins >= 1e12 and string.format("%.1fT", wins/1e12)
        or wins >= 1e9 and string.format("%.1fB", wins/1e9)
        or wins >= 1e6 and string.format("%.1fM", wins/1e6)
        or wins >= 1e3 and string.format("%.1fK", wins/1e3)
        or tostring(wins)
    )
end

-- ══════════════════════════════════════════
-- LÓGICA: AUTO REBIRTH
-- FIX: RequiredLevel = 25 (fixo), não baseado em rebirths
-- FIX: Verifica MaxRebirths = 11 para não tentar além do limite
-- ══════════════════════════════════════════
local function runAutoRebirth()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end

    local level    = data:FindFirstChild("Level")    and data.Level.Value    or 0
    local rebirths = data:FindFirstChild("Rebirths") and data.Rebirths.Value or 0

    -- Não rebirth se já no máximo
    if rebirths >= MAX_REBIRTHS then return end

    if level >= REBIRTH_REQUIRED_LEVEL then
        Remotes.Rebirth:FireServer()
    end
end

-- ══════════════════════════════════════════
-- LÓGICA: AUTO AURA
-- FIX: Estrutura real = UnlockedAuras (Folder com filhos por nome)
-- FIX: EquippedAura é StringValue
-- ══════════════════════════════════════════
local function runAutoAura()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end

    local wins         = data:FindFirstChild("Wins")         and data.Wins.Value         or 0
    local unlockedAuras = data:FindFirstChild("UnlockedAuras")
    local equippedAura  = data:FindFirstChild("EquippedAura") and data.EquippedAura.Value or ""

    -- Comprar a aura mais cara que podemos pagar e ainda não temos
    local bestToBuy, bestPrice = nil, -1
    for name, cfg in pairs(AurasConfig) do
        local owned = unlockedAuras and unlockedAuras:FindFirstChild(name) ~= nil
        if not owned and wins >= cfg.Price and cfg.Price > bestPrice then
            bestPrice = cfg.Price
            bestToBuy = name
        end
    end

    if bestToBuy then
        Remotes.BuyAura:FireServer(bestToBuy)
        task.wait(0.15) -- pequeno wait para o servidor processar antes de equipar
    end

    -- Equipar a aura desbloqueada com maior Multi
    local bestToEquip, bestMulti = "", -1
    if unlockedAuras then
        for _, child in ipairs(unlockedAuras:GetChildren()) do
            local cfg = AurasConfig[child.Name]
            if cfg and cfg.Multi > bestMulti then
                bestMulti   = cfg.Multi
                bestToEquip = child.Name
            end
        end
    end

    if bestToEquip ~= "" and bestToEquip ~= equippedAura then
        Remotes.EquipAura:FireServer(bestToEquip)
    end
end

-- ══════════════════════════════════════════
-- LÓGICA: AUTO TRAIL
-- FIX: Mesma estrutura que Auras — Folder com filhos por nome
-- FIX: EquippedTrail é StringValue
-- ══════════════════════════════════════════
local function runAutoTrail()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end

    local wins          = data:FindFirstChild("Wins")          and data.Wins.Value          or 0
    local unlockedTrails = data:FindFirstChild("UnlockedTrails")
    local equippedTrail  = data:FindFirstChild("EquippedTrail") and data.EquippedTrail.Value or ""

    -- Comprar a trail mais cara que podemos pagar e ainda não temos
    local bestToBuy, bestPrice = nil, -1
    for name, cfg in pairs(TrailsConfig) do
        local owned = unlockedTrails and unlockedTrails:FindFirstChild(name) ~= nil
        if not owned and wins >= cfg.Price and cfg.Price > bestPrice then
            bestPrice = cfg.Price
            bestToBuy = name
        end
    end

    if bestToBuy then
        Remotes.BuyTrail:FireServer(bestToBuy)
        task.wait(0.15)
    end

    -- Equipar a trail desbloqueada com maior Multi
    local bestToEquip, bestMulti = "", -1
    if unlockedTrails then
        for _, child in ipairs(unlockedTrails:GetChildren()) do
            local cfg = TrailsConfig[child.Name]
            if cfg and cfg.Multi > bestMulti then
                bestMulti   = cfg.Multi
                bestToEquip = child.Name
            end
        end
    end

    if bestToEquip ~= "" and bestToEquip ~= equippedTrail then
        Remotes.EquipTrail:FireServer(bestToEquip)
    end
end

-- ══════════════════════════════════════════
-- LOOP PRINCIPAL
-- ══════════════════════════════════════════
task.spawn(function()
    while true do
        task.wait(0.8)

        -- Atualiza label de status sempre
        pcall(updateStatusLabel)

        if _G.AutoRebirth then
            pcall(runAutoRebirth)
        end
        if _G.AutoAura then
            pcall(runAutoAura)
        end
        if _G.AutoTrail then
            pcall(runAutoTrail)
        end
    end
end)

print("🔱 Gaius Hub carregado! | Monkey Escape v2")
