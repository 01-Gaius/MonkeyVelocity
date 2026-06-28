--[[
    🔱 Titanium Hub - +1 Fuga de Macaco de Velocidade
    Criado com design premium, responsivo, minimizável e com automações completas.
    Sem dependências externas para garantir compatibilidade com qualquer executor.
--]]
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
-- Configurações Inlining para compatibilidade máxima (sem usar require)
local AurasConfig = {
    ["Amber"] = { Multi = 1.5, Price = 5000 },
    ["Ice Cold"] = { Multi = 2, Price = 150000 },
    ["Nature"] = { Multi = 2.5, Price = 4000000 },
    ["Rainbow"] = { Multi = 3, Price = 2500000000 },
    ["Lunar"] = { Multi = 3.5, Price = 90000000000 },
    ["Sparkle"] = { Multi = 4, Price = 5250000000000 }
}
local TrailsConfig = {
    ["Red"] = { Multi = 1.5, Price = 1000 },
    ["Blue"] = { Multi = 2, Price = 30000 },
    ["Green"] = { Multi = 3, Price = 750000 },
    ["Rainbow"] = { Multi = 5, Price = 400000000 },
    ["Galaxy"] = { Multi = 10, Price = 18750000000 },
    ["Divine"] = { Multi = 25, Price = 600000000000 }
}
local function GetMaxLevel(rebirths)
    return math.min(25 * (rebirths + 1), 250)
end
-- Estados globais das automações
_G.AutoRebirth = false
_G.AutoAura = false
_G.AutoTrail = false
-- 1. Determinar o Parent adequado para a GUI (CoreGui ou PlayerGui como fallback)
local targetParent = nil
local success, _ = pcall(function()
    targetParent = CoreGui
end)
if not success or not targetParent then
    targetParent = PlayerGui
end
-- Remover instâncias anteriores se existirem
if targetParent:FindFirstChild("GaiusHubGui") then
    targetParent.GaiusHubGui:Destroy()
end
-- 2. Criação da Interface do Usuário (Sleek Dark UI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GaiusHubGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = targetParent
-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 350, 0, 270)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -135)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui
local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame
local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(60, 60, 80)
MainStroke.Thickness = 1.5
MainStroke.Parent = MainFrame
local MainGradient = Instance.new("UIGradient")
MainGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(30, 30, 38)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(15, 15, 20))
})
MainGradient.Rotation = 45
MainGradient.Parent = MainFrame
-- Barra de Título (Title Bar)
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundTransparency = 1
TitleBar.Parent = MainFrame
-- Logotipo e Nome "🔱 Titanium"
local TitleText = Instance.new("TextLabel")
TitleText.Name = "TitleText"
TitleText.Size = UDim2.new(0.7, 0, 1, 0)
TitleText.Position = UDim2.new(0, 15, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "🔱 Titanium | Speed Monkey Escape"
TitleText.TextColor3 = Color3.fromRGB(0, 229, 255) -- Ciano Neon
TitleText.TextSize = 20
TitleText.Font = Enum.Font.GothamBold
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.Parent = TitleBar
-- Botão de Minimizar
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Name = "MinimizeBtn"
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Position = UDim2.new(1, -40, 0.5, -15)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
MinimizeBtn.Text = "▲"
MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeBtn.TextSize = 14
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.Parent = TitleBar
local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 6)
MinCorner.Parent = MinimizeBtn
local MinStroke = Instance.new("UIStroke")
MinStroke.Color = Color3.fromRGB(60, 60, 80)
MinStroke.Thickness = 1
MinStroke.Parent = MinimizeBtn
-- Divisor entre Título e Conteúdo
local Divider = Instance.new("Frame")
Divider.Name = "Divider"
Divider.Size = UDim2.new(1, 0, 0, 1)
Divider.Position = UDim2.new(0, 0, 0, 45)
Divider.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
Divider.BorderSizePixel = 0
Divider.Parent = MainFrame
-- Container de Conteúdo (Content Frame)
local ContentFrame = Instance.new("Frame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -30, 1, -60)
ContentFrame.Position = UDim2.new(0, 15, 0, 55)
ContentFrame.BackgroundTransparency = 1
ContentFrame.Parent = MainFrame
local ContentLayout = Instance.new("UIListLayout")
ContentLayout.SortOrder = Enum.SortOrder.LayoutOrder
ContentLayout.Padding = UDim.new(0, 12)
ContentLayout.Parent = ContentFrame
-- Função auxiliar para criar botões de alternância (Toggles) elegantes
local function createToggle(name, labelText, globalVarName)
    local Button = Instance.new("TextButton")
    Button.Name = name .. "Toggle"
    Button.Size = UDim2.new(1, 0, 0, 50)
    Button.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
    Button.Text = ""
    Button.AutoButtonColor = false
    Button.Parent = ContentFrame
    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 8)
    BtnCorner.Parent = Button
    local BtnStroke = Instance.new("UIStroke")
    BtnStroke.Color = Color3.fromRGB(50, 50, 70)
    BtnStroke.Thickness = 1
    BtnStroke.Parent = Button
    -- Label do botão
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.6, 0, 1, 0)
    Label.Position = UDim2.new(0, 15, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = labelText
    Label.TextColor3 = Color3.fromRGB(220, 220, 230)
    Label.TextSize = 14
    Label.Font = Enum.Font.GothamMedium
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Button
    -- Indicador visual de estado (Switch/Status)
    local StatusText = Instance.new("TextLabel")
    StatusText.Size = UDim2.new(0.3, 0, 1, 0)
    StatusText.Position = UDim2.new(0.7, -15, 0, 0)
    StatusText.BackgroundTransparency = 1
    StatusText.Text = "DESATIVADO"
    StatusText.TextColor3 = Color3.fromRGB(244, 67, 54) -- Vermelho suave
    StatusText.TextSize = 12
    StatusText.Font = Enum.Font.GothamBold
    StatusText.TextXAlignment = Enum.TextXAlignment.Right
    StatusText.Parent = Button
    -- Efeitos visuais de clique
    local function updateVisualState(isActive)
        local targetBg = isActive and Color3.fromRGB(0, 229, 255) or Color3.fromRGB(30, 30, 38)
        local targetTextCol = isActive and Color3.fromRGB(10, 10, 15) or Color3.fromRGB(220, 220, 230)
        local targetStatusCol = isActive and Color3.fromRGB(10, 10, 15) or Color3.fromRGB(244, 67, 54)
        local targetStrokeCol = isActive and Color3.fromRGB(0, 229, 255) or Color3.fromRGB(50, 50, 70)
        local statusString = isActive and "ATIVADO" or "DESATIVADO"
        TweenService:Create(Button, TweenInfo.new(0.2), {BackgroundColor3 = targetBg}):Play()
        TweenService:Create(Label, TweenInfo.new(0.2), {TextColor3 = targetTextCol}):Play()
        TweenService:Create(StatusText, TweenInfo.new(0.2), {TextColor3 = targetStatusCol}):Play()
        TweenService:Create(BtnStroke, TweenInfo.new(0.2), {Color = targetStrokeCol}):Play()
        StatusText.Text = statusString
    end
    Button.Activated:Connect(function()
        _G[globalVarName] = not _G[globalVarName]
        updateVisualState(_G[globalVarName])
    end)
    -- Efeito Hover
    Button.MouseEnter:Connect(function()
        if not _G[globalVarName] then
            TweenService:Create(BtnStroke, TweenInfo.new(0.15), {Color = Color3.fromRGB(0, 229, 255)}):Play()
        end
    end)
    Button.MouseLeave:Connect(function()
        if not _G[globalVarName] then
            TweenService:Create(BtnStroke, TweenInfo.new(0.15), {Color = Color3.fromRGB(50, 50, 70)}):Play()
        end
    end)
end
-- Instanciar os Toggles de automação
createToggle("AutoRebirth", "Auto Rebirth", "AutoRebirth")
createToggle("AutoAura", "Auto Aura (Melhor)", "AutoAura")
createToggle("AutoTrail", "Auto Trail (Melhor)", "AutoTrail")
-- 3. Funcionalidade de Minimizar Janela
local isMinimized = false
MinimizeBtn.Activated:Connect(function()
    isMinimized = not isMinimized
    local targetHeight = isMinimized and 45 or 270
    MinimizeBtn.Text = isMinimized and "▼" or "▲"
    ContentFrame.Visible = not isMinimized
    Divider.Visible = not isMinimized
    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 350, 0, targetHeight)
    }):Play()
end)
-- 4. Sistema para Arrastar Janela (Drag system)
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end
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
        update(input)
    end
end)
-- 5. Lógica Interna das Automações (Executadas em Loops seguros)
-- Lógica: Auto Rebirth
local function runAutoRebirth()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end
    local level = data:FindFirstChild("Level") and data.Level.Value or 0
    local rebirths = data:FindFirstChild("Rebirths") and data.Rebirths.Value or 0
    local maxLevel = GetMaxLevel(rebirths)
    if level >= maxLevel then
        ReplicatedStorage.Remotes.Rebirth:FireServer()
    end
end
-- Lógica: Auto Aura (Compra a melhor possível e equipa a melhor desbloqueada)
local function runAutoAura()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end
    local wins = data:FindFirstChild("Wins") and data.Wins.Value or 0
    local unlockedAuras = data:FindFirstChild("UnlockedAuras")
    local equippedAura = data:FindFirstChild("EquippedAura") and data.EquippedAura.Value or ""
    -- Encontrar a melhor aura que podemos comprar com as Wins atuais
    local bestAuraToBuy = nil
    local maxAffordablePrice = -1
    for name, config in pairs(AurasConfig) do
        local alreadyUnlocked = unlockedAuras and unlockedAuras:FindFirstChild(name) ~= nil
        if not alreadyUnlocked and wins >= config.Price then
            if config.Price > maxAffordablePrice then
                maxAffordablePrice = config.Price
                bestAuraToBuy = name
            end
        end
    end
    if bestAuraToBuy then
        ReplicatedStorage.Remotes.BuyAura:FireServer(bestAuraToBuy)
        task.wait(0.1)
    end
    -- Equipar a aura desbloqueada com o maior multiplicador
    local bestUnlockedAura = ""
    local maxMulti = -1
    if unlockedAuras then
        for _, child in ipairs(unlockedAuras:GetChildren()) do
            local name = child.Name
            local config = AurasConfig[name]
            if config and config.Multi > maxMulti then
                maxMulti = config.Multi
                bestUnlockedAura = name
            end
        end
    end
    if bestUnlockedAura ~= "" and bestUnlockedAura ~= equippedAura then
        ReplicatedStorage.Remotes.EquipAura:FireServer(bestUnlockedAura)
    end
end
-- Lógica: Auto Trail (Compra a melhor possível e equipa a melhor desbloqueada)
local function runAutoTrail()
    local data = LocalPlayer:FindFirstChild("Data")
    if not data then return end
    local wins = data:FindFirstChild("Wins") and data.Wins.Value or 0
    local unlockedTrails = data:FindFirstChild("UnlockedTrails")
    local equippedTrail = data:FindFirstChild("EquippedTrail") and data.EquippedTrail.Value or ""
    -- Encontrar a melhor trilha que podemos comprar com as Wins atuais
    local bestTrailToBuy = nil
    local maxAffordablePrice = -1
    for name, config in pairs(TrailsConfig) do
        local alreadyUnlocked = unlockedTrails and unlockedTrails:FindFirstChild(name) ~= nil
        if not alreadyUnlocked and wins >= config.Price then
            if config.Price > maxAffordablePrice then
                maxAffordablePrice = config.Price
                bestTrailToBuy = name
            end
        end
    end
    if bestTrailToBuy then
        ReplicatedStorage.Remotes.BuyTrail:FireServer(bestTrailToBuy)
        task.wait(0.1)
    end
    -- Equipar a trilha desbloqueada com o maior multiplicador
    local bestUnlockedTrail = ""
    local maxMulti = -1
    if unlockedTrails then
        for _, child in ipairs(unlockedTrails:GetChildren()) do
            local name = child.Name
            local config = TrailsConfig[name]
            if config and config.Multi > maxMulti then
                maxMulti = config.Multi
                bestUnlockedTrail = name
            end
        end
    end
    if bestUnlockedTrail ~= "" and bestUnlockedTrail ~= equippedTrail then
        ReplicatedStorage.Remotes.EquipTrail:FireServer(bestUnlockedTrail)
    end
end
-- 6. Loop de execução assíncrono para as funções habilitadas
task.spawn(function()
    while true do
        task.wait(0.8) -- Delay seguro para evitar sobrecarga de requisições de rede
        
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
print("🔱 gaius Hub carregado com sucesso!")
