-- 🔱 Titanium Hub | Speed Monkey Escape
-- v3 — UI completamente redesenhada
-- Minimizado: botão quadrado arredondado com tridente (clique abre a UI)

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

local REBIRTH_REQUIRED_LEVEL = 25
local MAX_REBIRTHS = 11

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
    local t = Instance.new("Frame")
    t.Parent = CoreGui
    t:Destroy()
    targetParent = CoreGui
end)
if not ok or not targetParent then
    targetParent = PlayerGui
end

if targetParent:FindFirstChild("TitaniumHubGui") then
    targetParent.TitaniumHubGui:Destroy()
end

-- ══════════════════════════════════════════
-- GUI ROOT
-- ══════════════════════════════════════════
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "TitaniumHubGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = targetParent

-- ══════════════════════════════════════════
-- BOTÃO MINIMIZADO (tridente quadrado)
-- Visível apenas quando minimizado
-- ══════════════════════════════════════════
local MiniBtn = Instance.new("TextButton")
MiniBtn.Name = "MiniBtn"
MiniBtn.Size = UDim2.new(0, 58, 0, 58)
MiniBtn.Position = UDim2.new(0, 24, 0.5, -29)
MiniBtn.BackgroundColor3 = Color3.fromRGB(14, 14, 20)
MiniBtn.Text = "🔱"
MiniBtn.TextSize = 26
MiniBtn.Font = Enum.Font.GothamBold
MiniBtn.TextColor3 = Color3.fromRGB(0, 210, 240)
MiniBtn.AutoButtonColor = false
MiniBtn.Visible = false
MiniBtn.ZIndex = 10
MiniBtn.Parent = ScreenGui

local MiniBtnCorner = Instance.new("UICorner")
MiniBtnCorner.CornerRadius = UDim.new(0, 16)
MiniBtnCorner.Parent = MiniBtn

local MiniBtnStroke = Instance.new("UIStroke")
MiniBtnStroke.Color = Color3.fromRGB(0, 180, 220)
MiniBtnStroke.Thickness = 1.5
MiniBtnStroke.Parent = MiniBtn

-- Efeito hover no botão minimizado
MiniBtn.MouseEnter:Connect(function()
    TweenService:Create(MiniBtn, TweenInfo.new(0.15), {
        BackgroundColor3 = Color3.fromRGB(0, 50, 70)
    }):Play()
    TweenService:Create(MiniBtnStroke, TweenInfo.new(0.15), {
        Color = Color3.fromRGB(0, 229, 255)
    }):Play()
end)
MiniBtn.MouseLeave:Connect(function()
    TweenService:Create(MiniBtn, TweenInfo.new(0.15), {
        BackgroundColor3 = Color3.fromRGB(14, 14, 20)
    }):Play()
    TweenService:Create(MiniBtnStroke, TweenInfo.new(0.15), {
        Color = Color3.fromRGB(0, 180, 220)
    }):Play()
end)

-- ══════════════════════════════════════════
-- JANELA PRINCIPAL
-- ══════════════════════════════════════════
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 360, 0, 300)
MainFrame.Position = UDim2.new(0, 100, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(14, 14, 20)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.ZIndex = 5
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 14)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = Color3.fromRGB(40, 40, 65)
MainStroke.Thickness = 1.5

-- Linha de acento cyan no topo
local TopAccent = Instance.new("Frame", MainFrame)
TopAccent.Size = UDim2.new(0.55, 0, 0, 2)
TopAccent.Position = UDim2.new(0.225, 0, 0, 0)
TopAccent.BackgroundColor3 = Color3.fromRGB(0, 200, 235)
TopAccent.BorderSizePixel = 0
TopAccent.ZIndex = 6
Instance.new("UICorner", TopAccent).CornerRadius = UDim.new(0, 2)

-- ══════════════════════════════════════════
-- TITLE BAR
-- ══════════════════════════════════════════
local TitleBar = Instance.new("Frame", MainFrame)
TitleBar.Size = UDim2.new(1, 0, 0, 52)
TitleBar.BackgroundTransparency = 1
TitleBar.ZIndex = 6

-- Ícone tridente
local TitleIcon = Instance.new("TextLabel", TitleBar)
TitleIcon.Size = UDim2.new(0, 36, 0, 36)
TitleIcon.Position = UDim2.new(0, 14, 0.5, -18)
TitleIcon.BackgroundTransparency = 1
TitleIcon.Text = "🔱"
TitleIcon.TextSize = 22
TitleIcon.Font = Enum.Font.GothamBold
TitleIcon.ZIndex = 6

-- Nome do hub
local TitleText = Instance.new("TextLabel", TitleBar)
TitleText.Size = UDim2.new(0, 150, 0, 22)
TitleText.Position = UDim2.new(0, 50, 0.5, -18)
TitleText.BackgroundTransparency = 1
TitleText.Text = "TITANIUM"
TitleText.TextColor3 = Color3.fromRGB(0, 220, 255)
TitleText.TextSize = 17
TitleText.Font = Enum.Font.GothamBold
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.ZIndex = 6

-- Sub-título
local TitleSub = Instance.new("TextLabel", TitleBar)
TitleSub.Size = UDim2.new(0, 200, 0, 14)
TitleSub.Position = UDim2.new(0, 50, 0.5, 4)
TitleSub.BackgroundTransparency = 1
TitleSub.Text = "Speed Monkey Escape"
TitleSub.TextColor3 = Color3.fromRGB(70, 70, 100)
TitleSub.TextSize = 11
TitleSub.Font = Enum.Font.Gotham
TitleSub.TextXAlignment = Enum.TextXAlignment.Left
TitleSub.ZIndex = 6

-- Botão minimizar (seta)
local MinimizeBtn = Instance.new("TextButton", TitleBar)
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Position = UDim2.new(1, -44, 0.5, -15)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(22, 22, 32)
MinimizeBtn.Text = "–"
MinimizeBtn.TextColor3 = Color3.fromRGB(120, 120, 160)
MinimizeBtn.TextSize = 16
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.AutoButtonColor = false
MinimizeBtn.ZIndex = 7
Instance.new("UICorner", MinimizeBtn).CornerRadius = UDim.new(0, 8)
local MinStroke = Instance.new("UIStroke", MinimizeBtn)
MinStroke.Color = Color3.fromRGB(45, 45, 70)
MinStroke.Thickness = 1

MinimizeBtn.MouseEnter:Connect(function()
    TweenService:Create(MinimizeBtn, TweenInfo.new(0.12), {
        BackgroundColor3 = Color3.fromRGB(0, 50, 70)
    }):Play()
    TweenService:Create(MinStroke, TweenInfo.new(0.12), {
        Color = Color3.fromRGB(0, 200, 235)
    }):Play()
    TweenService:Create(MinimizeBtn, TweenInfo.new(0.12), {
        TextColor3 = Color3.fromRGB(0, 220, 255)
    }):Play()
end)
MinimizeBtn.MouseLeave:Connect(function()
    TweenService:Create(MinimizeBtn, TweenInfo.new(0.12), {
        BackgroundColor3 = Color3.fromRGB(22, 22, 32)
    }):Play()
    TweenService:Create(MinStroke, TweenInfo.new(0.12), {
        Color = Color3.fromRGB(45, 45, 70)
    }):Play()
    TweenService:Create(MinimizeBtn, TweenInfo.new(0.12), {
        TextColor3 = Color3.fromRGB(120, 120, 160)
    }):Play()
end)

-- Divisor
local Divider = Instance.new("Frame", MainFrame)
Divider.Size = UDim2.new(1, -28, 0, 1)
Divider.Position = UDim2.new(0, 14, 0, 52)
Divider.BackgroundColor3 = Color3.fromRGB(35, 35, 55)
Divider.BorderSizePixel = 0
Divider.ZIndex = 6

-- ══════════════════════════════════════════
-- STATUS BAR (Level / Rebirths / Wins)
-- ══════════════════════════════════════════
local StatusBar = Instance.new("Frame", MainFrame)
StatusBar.Size = UDim2.new(1, -28, 0, 28)
StatusBar.Position = UDim2.new(0, 14, 0, 58)
StatusBar.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
StatusBar.BorderSizePixel = 0
StatusBar.ZIndex = 6
Instance.new("UICorner", StatusBar).CornerRadius = UDim.new(0, 8)

local StatusLabel = Instance.new("TextLabel", StatusBar)
StatusLabel.Size = UDim2.new(1, -16, 1, 0)
StatusLabel.Position = UDim2.new(0, 10, 0, 0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Carregando..."
StatusLabel.TextColor3 = Color3.fromRGB(80, 80, 120)
StatusLabel.TextSize = 11
StatusLabel.Font = Enum.Font.GothamMedium
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.ZIndex = 7

-- ══════════════════════════════════════════
-- CONTENT FRAME (toggles)
-- ══════════════════════════════════════════
local ContentFrame = Instance.new("Frame", MainFrame)
ContentFrame.Size = UDim2.new(1, -28, 1, -100)
ContentFrame.Position = UDim2.new(0, 14, 0, 94)
ContentFrame.BackgroundTransparency = 1
ContentFrame.ZIndex = 6

local ContentLayout = Instance.new("UIListLayout", ContentFrame)
ContentLayout.SortOrder = Enum.SortOrder.LayoutOrder
ContentLayout.Padding = UDim.new(0, 8)

-- ══════════════════════════════════════════
-- FACTORY: TOGGLE
-- ══════════════════════════════════════════
local function createToggle(name, icon, labelText, descText, globalVarName, order)
    local Btn = Instance.new("TextButton", ContentFrame)
    Btn.Name = name .. "Toggle"
    Btn.LayoutOrder = order
    Btn.Size = UDim2.new(1, 0, 0, 52)
    Btn.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    Btn.Text = ""
    Btn.AutoButtonColor = false
    Btn.ZIndex = 7
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 10)
    local BtnStroke = Instance.new("UIStroke", Btn)
    BtnStroke.Color = Color3.fromRGB(35, 35, 55)
    BtnStroke.Thickness = 1

    -- Ícone
    local IconLbl = Instance.new("TextLabel", Btn)
    IconLbl.Size = UDim2.new(0, 32, 1, 0)
    IconLbl.Position = UDim2.new(0, 10, 0, 0)
    IconLbl.BackgroundTransparency = 1
    IconLbl.Text = icon
    IconLbl.TextSize = 18
    IconLbl.Font = Enum.Font.GothamBold
    IconLbl.ZIndex = 8

    -- Label
    local Label = Instance.new("TextLabel", Btn)
    Label.Size = UDim2.new(0.55, 0, 0, 20)
    Label.Position = UDim2.new(0, 46, 0, 8)
    Label.BackgroundTransparency = 1
    Label.Text = labelText
    Label.TextColor3 = Color3.fromRGB(200, 200, 220)
    Label.TextSize = 13
    Label.Font = Enum.Font.GothamMedium
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.ZIndex = 8

    -- Desc
    local Desc = Instance.new("TextLabel", Btn)
    Desc.Size = UDim2.new(0.6, 0, 0, 14)
    Desc.Position = UDim2.new(0, 46, 0, 28)
    Desc.BackgroundTransparency = 1
    Desc.Text = descText
    Desc.TextColor3 = Color3.fromRGB(55, 55, 80)
    Desc.TextSize = 10
    Desc.Font = Enum.Font.Gotham
    Desc.TextXAlignment = Enum.TextXAlignment.Left
    Desc.ZIndex = 8

    -- Pill de status
    local Pill = Instance.new("Frame", Btn)
    Pill.Size = UDim2.new(0, 56, 0, 22)
    Pill.Position = UDim2.new(1, -66, 0.5, -11)
    Pill.BackgroundColor3 = Color3.fromRGB(40, 15, 15)
    Pill.BorderSizePixel = 0
    Pill.ZIndex = 8
    Instance.new("UICorner", Pill).CornerRadius = UDim.new(0, 6)

    local PillText = Instance.new("TextLabel", Pill)
    PillText.Size = UDim2.new(1, 0, 1, 0)
    PillText.BackgroundTransparency = 1
    PillText.Text = "OFF"
    PillText.TextColor3 = Color3.fromRGB(200, 60, 60)
    PillText.TextSize = 11
    PillText.Font = Enum.Font.GothamBold
    PillText.ZIndex = 9

    local function updateVisual(on)
        if on then
            TweenService:Create(Btn,      TweenInfo.new(0.2), { BackgroundColor3 = Color3.fromRGB(0, 35, 50)      }):Play()
            TweenService:Create(BtnStroke,TweenInfo.new(0.2), { Color             = Color3.fromRGB(0, 180, 220)    }):Play()
            TweenService:Create(Label,    TweenInfo.new(0.2), { TextColor3        = Color3.fromRGB(0, 220, 255)    }):Play()
            TweenService:Create(Desc,     TweenInfo.new(0.2), { TextColor3        = Color3.fromRGB(0, 120, 150)    }):Play()
            TweenService:Create(Pill,     TweenInfo.new(0.2), { BackgroundColor3  = Color3.fromRGB(0, 40, 55)      }):Play()
            TweenService:Create(PillText, TweenInfo.new(0.2), { TextColor3        = Color3.fromRGB(0, 220, 255)    }):Play()
        else
            TweenService:Create(Btn,      TweenInfo.new(0.2), { BackgroundColor3 = Color3.fromRGB(20, 20, 30)      }):Play()
            TweenService:Create(BtnStroke,TweenInfo.new(0.2), { Color             = Color3.fromRGB(35, 35, 55)     }):Play()
            TweenService:Create(Label,    TweenInfo.new(0.2), { TextColor3        = Color3.fromRGB(200, 200, 220)  }):Play()
            TweenService:Create(Desc,     TweenInfo.new(0.2), { TextColor3        = Color3.fromRGB(55, 55, 80)     }):Play()
            TweenService:Create(Pill,     TweenInfo.new(0.2), { BackgroundColor3  = Color3.fromRGB(40, 15, 15)     }):Play()
            TweenService:Create(PillText, TweenInfo.new(0.2), { TextColor3        = Color3.fromRGB(200, 60, 60)    }):Play()
        end
        PillText.Text = on and "ON" or "OFF"
    end

    Btn.Activated:Connect(function()
        _G[globalVarName] = not _G[globalVarName]
        updateVisual(_G[globalVarName])
    end)

    Btn.MouseEnter:Connect(function()
        if not _G[globalVarName] then
            TweenService:Create(BtnStroke, TweenInfo.new(0.1), { Color = Color3.fromRGB(60, 60, 90) }):Play()
        end
    end)
    Btn.MouseLeave:Connect(function()
        if not _G[globalVarName] then
            TweenService:Create(BtnStroke, TweenInfo.new(0.1), { Color = Color3.fromRGB(35, 35, 55) }):Play()
        end
    end)
end

createToggle("AutoRebirth", "⚡", "Auto Rebirth",      "Rebirth ao atingir Lv.25",      "AutoRebirth", 1)
createToggle("AutoAura",    "✨", "Auto Aura",          "Compra e equipa a melhor Aura", "AutoAura",    2)
createToggle("AutoTrail",   "🌊", "Auto Trail",         "Compra e equipa a melhor Trail","AutoTrail",   3)

-- ══════════════════════════════════════════
-- MINIMIZAR / RESTAURAR
-- ══════════════════════════════════════════
local isMinimized = false

local function setMinimized(state)
    isMinimized = state
    if state then
        -- Esconde a janela e mostra o botão tridente
        TweenService:Create(MainFrame, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
            Size = UDim2.new(0, 0, 0, 0),
            Position = UDim2.new(
                MainFrame.Position.X.Scale,
                MainFrame.Position.X.Offset + 180,
                MainFrame.Position.Y.Scale,
                MainFrame.Position.Y.Offset + 150
            )
        }):Play()
        task.delay(0.2, function()
            MainFrame.Visible = false
            MiniBtn.Visible = true
            MiniBtn.Position = UDim2.new(
                MainFrame.Position.X.Scale,
                MainFrame.Position.X.Offset - 9,
                MainFrame.Position.Y.Scale,
                MainFrame.Position.Y.Offset + 121
            )
        end)
    else
        -- Restaura a janela
        MiniBtn.Visible = false
        MainFrame.Visible = true
        MainFrame.Size = UDim2.new(0, 0, 0, 0)
        MainFrame.Position = UDim2.new(
            MainFrame.Position.X.Scale,
            MainFrame.Position.X.Offset + 180,
            MainFrame.Position.Y.Scale,
            MainFrame.Position.Y.Offset + 150
        )
        TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 360, 0, 300),
            Position = UDim2.new(
                MainFrame.Position.X.Scale,
                MainFrame.Position.X.Offset - 180,
                MainFrame.Position.Y.Scale,
                MainFrame.Position.Y.Offset - 150
            )
        }):Play()
    end
end

MinimizeBtn.Activated:Connect(function() setMinimized(true) end)
MiniBtn.Activated:Connect(function() setMinimized(false) end)

-- ══════════════════════════════════════════
-- DRAG SYSTEM (janela principal e mini-botão)
-- ══════════════════════════════════════════
local function makeDraggable(handle, target)
    local dragging, dragInput, dragStart, startPos

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging  = true
            dragStart = input.Position
            startPos  = target.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local d = input.Position - dragStart
            target.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + d.X,
                startPos.Y.Scale, startPos.Y.Offset + d.Y
            )
        end
    end)
end

makeDraggable(TitleBar, MainFrame)
makeDraggable(MiniBtn,  MiniBtn)

-- ══════════════════════════════════════════
-- STATUS LABEL HELPER
-- ══════════════════════════════════════════
local function fmtNum(n)
    if n >= 1e12 then return string.format("%.1fT", n/1e12)
    elseif n >= 1e9  then return string.format("%.1fB", n/1e9)
    elseif n >= 1e6  then return string.format("%.1fM", n/1e6)
    elseif n >= 1e3  then return string.format("%.1fK", n/1e3)
    else return tostring(n) end
end

local function updateStatus()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then
        StatusLabel.Text = "⏳ Aguardando dados..."
        return
    end
    local lv  = data:FindFirstChild("Level")    and data.Level.Value    or 0
    local rb  = data:FindFirstChild("Rebirths") and data.Rebirths.Value or 0
    local win = data:FindFirstChild("Wins")     and data.Wins.Value     or 0
    StatusLabel.Text = string.format(
        "Lv.%d  •  %d/%d Rebirths  •  %s Wins",
        lv, rb, MAX_REBIRTHS, fmtNum(win)
    )
    StatusLabel.TextColor3 = Color3.fromRGB(0, 160, 190)
end

-- ══════════════════════════════════════════
-- LÓGICA: AUTO REBIRTH
-- ══════════════════════════════════════════
local function runAutoRebirth()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end
    local level    = data:FindFirstChild("Level")    and data.Level.Value    or 0
    local rebirths = data:FindFirstChild("Rebirths") and data.Rebirths.Value or 0
    if rebirths >= MAX_REBIRTHS then return end
    if level >= REBIRTH_REQUIRED_LEVEL then
        Remotes.Rebirth:FireServer()
    end
end

-- ══════════════════════════════════════════
-- LÓGICA: AUTO AURA
-- ══════════════════════════════════════════
local function runAutoAura()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end
    local wins          = data:FindFirstChild("Wins")         and data.Wins.Value         or 0
    local unlockedAuras = data:FindFirstChild("UnlockedAuras")
    local equippedAura  = data:FindFirstChild("EquippedAura") and data.EquippedAura.Value or ""

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
        task.wait(0.15)
    end

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
-- ══════════════════════════════════════════
local function runAutoTrail()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end
    local wins           = data:FindFirstChild("Wins")          and data.Wins.Value          or 0
    local unlockedTrails = data:FindFirstChild("UnlockedTrails")
    local equippedTrail  = data:FindFirstChild("EquippedTrail") and data.EquippedTrail.Value or ""

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
        pcall(updateStatus)
        if _G.AutoRebirth then pcall(runAutoRebirth) end
        if _G.AutoAura    then pcall(runAutoAura)    end
        if _G.AutoTrail   then pcall(runAutoTrail)   end
    end
end)

print("🔱 Titanium Hub v3 carregado!")
