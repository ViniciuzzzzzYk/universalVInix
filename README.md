-- Gunfight Arena Exploit v2.0
-- GUI Custom sem bibliotecas externas
-- Features completas: ESP, Hitbox Expander, Aimbot, AutoFarm

--[[
  INSTRUÇÕES:
  1. Copie todo este script
  2. Cole no seu executor mobile (Delta, Hydrogen, Fluxus)
  3. Execute
  4. A GUI aparecerá na tela com controles touch-friendly
]]

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Configurações
local Settings = {
    ESP = {
        Enabled = false,
        Color = Color3.fromRGB(255, 50, 50),
        TeamCheck = true
    },
    Hitbox = {
        Enabled = false,
        Size = Vector3.new(5, 5, 5),
        OriginalSize = Vector3.new(2, 2, 1)
    },
    Aimbot = {
        Enabled = false,
        Target = nil,
        Smoothness = 0.2,
        FOV = 100,
        TeamCheck = true
    },
    AutoFarm = {
        Enabled = false,
        Delay = 0.5
    }
}

-- Variáveis de controle
local GUIEnabled = true
local Dragging, DragInput, DragStart, StartPos
local ESPInstances = {}
local HitboxInstances = {}

-- Cria a GUI principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GFAMobileGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 300, 0, 400)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
MainFrame.BackgroundTransparency = 0.2
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui  -- Fix: Parent MainFrame to ScreenGui

-- Efeito de borda
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(80, 80, 100)
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "TitleLabel"
TitleLabel.Size = UDim2.new(1, -40, 1, 0)
TitleLabel.Position = UDim2.new(0, 10, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "GF Arena Mobile"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 18
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -35, 0.5, -15)
CloseButton.AnchorPoint = Vector2.new(0.5, 0.5)
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Text = "X"
CloseButton.TextSize = 16
CloseButton.Font = Enum.Font.GothamBold
CloseButton.Parent = TitleBar

-- Tabs
local Tabs = {"Visual", "Combate", "Farm"}
local CurrentTab = "Visual"

local TabButtons = {}
local TabFrames = {}

local TabButtonsFrame = Instance.new("Frame")
TabButtonsFrame.Name = "TabButtonsFrame"
TabButtonsFrame.Size = UDim2.new(1, 0, 0, 40)
TabButtonsFrame.Position = UDim2.new(0, 0, 0, 40)
TabButtonsFrame.BackgroundTransparency = 1
TabButtonsFrame.Parent = MainFrame

for i, tabName in ipairs(Tabs) do
    local TabButton = Instance.new("TextButton")
    TabButton.Name = tabName.."TabButton"
    TabButton.Size = UDim2.new(1/#Tabs, -5, 1, -5)
    TabButton.Position = UDim2.new((i-1)/#Tabs, 2, 0, 2)
    TabButton.BackgroundColor3 = i == 1 and Color3.fromRGB(60, 60, 80) or Color3.fromRGB(40, 40, 50)
    TabButton.Text = tabName
    TabButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    TabButton.TextSize = 14
    TabButton.Font = Enum.Font.Gotham
    TabButton.Parent = TabButtonsFrame
    
    local TabFrame = Instance.new("Frame")
    TabFrame.Name = tabName.."TabFrame"
    TabFrame.Size = UDim2.new(1, -20, 1, -90)
    TabFrame.Position = UDim2.new(0, 10, 0, 85)
    TabFrame.BackgroundTransparency = 1
    TabFrame.Visible = i == 1
    TabFrame.Parent = MainFrame
    
    TabButtons[tabName] = TabButton
    TabFrames[tabName] = TabFrame
    
    TabButton.MouseButton1Click:Connect(function()
        CurrentTab = tabName
        for name, frame in pairs(TabFrames) do
            frame.Visible = name == tabName
        end
        for name, button in pairs(TabButtons) do
            button.BackgroundColor3 = name == tabName and Color3.fromRGB(60, 60, 80) or Color3.fromRGB(40, 40, 50)
        end
    end)
end

-- Função para criar toggles
local function CreateToggle(parent, name, default, callback)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Name = name.."ToggleFrame"
    ToggleFrame.Size = UDim2.new(1, 0, 0, 40)
    ToggleFrame.BackgroundTransparency = 1
    ToggleFrame.Parent = parent
    
    local ToggleButton = Instance.new("TextButton")
    ToggleButton.Name = name.."ToggleButton"
    ToggleButton.Size = UDim2.new(0, 120, 0, 30)
    ToggleButton.Position = UDim2.new(0, 0, 0.5, -15)
    ToggleButton.BackgroundColor3 = default and Color3.fromRGB(60, 150, 60) or Color3.fromRGB(150, 60, 60)
    ToggleButton.Text = name .. ": " .. (default and "ON" or "OFF")
    ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    ToggleButton.TextSize = 14
    ToggleButton.Font = Enum.Font.Gotham
    ToggleButton.Parent = ToggleFrame
    
    local Value = default
    
    ToggleButton.MouseButton1Click:Connect(function()
        Value = not Value
        ToggleButton.BackgroundColor3 = Value and Color3.fromRGB(60, 150, 60) or Color3.fromRGB(150, 60, 60)
        ToggleButton.Text = name .. ": " .. (Value and "ON" or "OFF")
        callback(Value)
    end)
    
    return {
        SetValue = function(newValue)
            Value = newValue
            ToggleButton.BackgroundColor3 = Value and Color3.fromRGB(60, 150, 60) or Color3.fromRGB(150, 60, 60)
            ToggleButton.Text = name .. ": " .. (Value and "ON" or "OFF")
            callback(Value)
        end
    }
end

-- Função para criar sliders
local function CreateSlider(parent, name, min, max, default, callback)
    local SliderFrame = Instance.new("Frame")
    SliderFrame.Name = name.."SliderFrame"
    SliderFrame.Size = UDim2.new(1, 0, 0, 60)
    SliderFrame.BackgroundTransparency = 1
    SliderFrame.Parent = parent
    
    local SliderName = Instance.new("TextLabel")
    SliderName.Name = name.."SliderName"
    SliderName.Size = UDim2.new(1, 0, 0, 20)
    SliderName.Position = UDim2.new(0, 0, 0, 0)
    SliderName.BackgroundTransparency = 1
    SliderName.Text = name .. ": " .. default
    SliderName.TextColor3 = Color3.fromRGB(255, 255, 255)
    SliderName.TextSize = 14
    SliderName.TextXAlignment = Enum.TextXAlignment.Left
    SliderName.Font = Enum.Font.Gotham
    SliderName.Parent = SliderFrame
    
    local SliderTrack = Instance.new("Frame")
    SliderTrack.Name = name.."SliderTrack"
    SliderTrack.Size = UDim2.new(1, 0, 0, 10)
    SliderTrack.Position = UDim2.new(0, 0, 0, 30)
    SliderTrack.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    SliderTrack.BorderSizePixel = 0
    SliderTrack.Parent = SliderFrame
    
    local SliderThumb = Instance.new("Frame")
    SliderThumb.Name = name.."SliderThumb"
    SliderThumb.Size = UDim2.new(0, 20, 0, 20)
    SliderThumb.Position = UDim2.new((default - min)/(max - min), -5, 0, -5)
    SliderThumb.AnchorPoint = Vector2.new(0.5, 0.5)
    SliderThumb.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
    SliderThumb.BorderSizePixel = 0
    SliderThumb.Parent = SliderTrack
    
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(1, 0)
    UICorner.Parent = SliderThumb
    
    local Dragging = false
    
    local function UpdateValue(input)
        local relativeX = (input.Position.X - SliderTrack.AbsolutePosition.X) / SliderTrack.AbsoluteSize.X
        local value = math.clamp(min + (max - min) * relativeX, min, max)
        local rounded = math.floor(value * 10) / 10
        
        SliderThumb.Position = UDim2.new((value - min)/(max - min), -5, 0, -5)
        SliderName.Text = name .. ": " .. rounded
        callback(rounded)
    end
    
    SliderThumb.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Dragging = true
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    Dragging = false
                end
            end)
        end
    end)
    
    SliderTrack.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            UpdateValue(input)
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if Dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            UpdateValue(input)
        end
    end)
end

-- Cria os controles na GUI

-- Tab Visual
local ESPToggle = CreateToggle(TabFrames.Visual, "ESP", false, function(value)
    Settings.ESP.Enabled = value
end)

local HitboxToggle = CreateToggle(TabFrames.Visual, "Hitbox Expander", false, function(value)
    Settings.Hitbox.Enabled = value
end)

-- Tab Combate
local AimbotToggle = CreateToggle(TabFrames.Combate, "Aimbot", false, function(value)
    Settings.Aimbot.Enabled = value
end)

CreateSlider(TabFrames.Combate, "Suavidade", 0.1, 1, Settings.Aimbot.Smoothness, function(value)
    Settings.Aimbot.Smoothness = value
end)

CreateSlider(TabFrames.Combate, "FOV", 50, 300, Settings.Aimbot.FOV, function(value)
    Settings.Aimbot.FOV = value
end)

-- Tab Farm
local AutoFarmToggle = CreateToggle(TabFrames.Farm, "Auto Farm", false, function(value)
    Settings.AutoFarm.Enabled = value
end)

CreateSlider(TabFrames.Farm, "Intervalo", 0.1, 5, Settings.AutoFarm.Delay, function(value)
    Settings.AutoFarm.Delay = value
end)

-- Função para arrastar a janela
local function DragGUI(input)
    local delta = input.Position - DragStart
    local newPos = StartPos + UDim2.new(0, delta.X, 0, delta.Y)
    
    -- Limita a janela à área da tela
    local viewportSize = workspace.CurrentCamera.ViewportSize
    local newX = math.clamp(newPos.X.Offset, 0, viewportSize.X - MainFrame.AbsoluteSize.X)
    local newY = math.clamp(newPos.Y.Offset, 0, viewportSize.Y - MainFrame.AbsoluteSize.Y)
    MainFrame.Position = UDim2.new(0, newX, 0, newY)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Dragging = true
        DragStart = input.Position
        StartPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                Dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and Dragging then
        DragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == DragInput and Dragging then
        DragGUI(input)
    end
end)

-- Botão de fechar
CloseButton.MouseButton1Click:Connect(function()
    GUIEnabled = false
    ScreenGui:Destroy()
end)

-- Adiciona a GUI ao PlayerGui
if game:GetService("CoreGui"):FindFirstChild("GFAMobileGUI") then
    game:GetService("CoreGui").GFAMobileGUI:Destroy()
end
ScreenGui.Parent = game:GetService("CoreGui")

-- Implementação das funcionalidades

-- ESP
local function UpdateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character then
                local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                
                if Settings.ESP.Enabled then
                    if not ESPInstances[player] then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = player.Name.."_ESP"
                        highlight.FillColor = Settings.ESP.Color
                        highlight.OutlineColor = Settings.ESP.Color
                        highlight.FillTransparency = 0.5
                        highlight.OutlineTransparency = 0
                        highlight.Parent = character
                        ESPInstances[player] = highlight
                    else
                        ESPInstances[player].FillColor = Settings.ESP.Color
                        ESPInstances[player].OutlineColor = Settings.ESP.Color
                        ESPInstances[player].Adornee = character
                        ESPInstances[player].Parent = character
                    end
                else
                    if ESPInstances[player] then
                        ESPInstances[player]:Destroy()
                        ESPInstances[player] = nil
                    end
                end
            end
        end
    end
end

-- Hitbox Expander
local function UpdateHitboxes()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character then
                local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                
                if humanoidRootPart then
                    if Settings.Hitbox.Enabled then
                        humanoidRootPart.Size = Settings.Hitbox.Size
                        humanoidRootPart.Transparency = 0.7
                        humanoidRootPart.CanCollide = false
                        HitboxInstances[player] = true
                    else
                        if HitboxInstances[player] then
                            humanoidRootPart.Size = Settings.Hitbox.OriginalSize
                            humanoidRootPart.Transparency = 0
                            humanoidRootPart.CanCollide = true
                            HitboxInstances[player] = nil
                        end
                    end
                end
            end
        end
    end
end

-- Aimbot
local function FindClosestPlayer()
    local closestPlayer, closestDistance = nil, Settings.Aimbot.FOV
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
            if humanoidRootPart then
                local screenPoint = Camera:WorldToViewportPoint(humanoidRootPart.Position)
                if screenPoint.Z > 0 then -- Verifica se está na frente da câmera
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                    
                    if distance < closestDistance then
                        closestPlayer = player
                        closestDistance = distance
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

local function AimbotThread()
    while Settings.Aimbot.Enabled and GUIEnabled do
        task.wait()
        
        local targetPlayer = FindClosestPlayer()
        if targetPlayer and targetPlayer.Character then
            local targetPart = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
            if targetPart then
                local camCFrame = Camera.CFrame
                local targetPosition = targetPart.Position + Vector3.new(0, 1.5, 0) -- Ajuste para mirar no peito
                local newCFrame = camCFrame:Lerp(CFrame.lookAt(camCFrame.Position, targetPosition), Settings.Aimbot.Smoothness)
                Camera.CFrame = newCFrame
            end
        end
    end
end

-- AutoFarm
local function AutoFarmThread()
    while Settings.AutoFarm.Enabled and GUIEnabled do
        task.wait(Settings.AutoFarm.Delay)
        
        -- Implementação básica (ajuste para o jogo específico)
        local character = LocalPlayer.Character
        if character then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                -- Simula ações de farm (substitua pela lógica real do jogo)
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character then
                        local enemyRoot = player.Character:FindFirstChild("HumanoidRootPart")
                        if enemyRoot then
                            -- Simula ataque (ajuste para o sistema de combate do jogo)
                            humanoid:MoveTo(enemyRoot.Position)
                            break
                        end
                    end
                end
            end
        end
    end
end

-- Conexões de eventos
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if Settings.ESP.Enabled then UpdateESP() end
        if Settings.Hitbox.Enabled then UpdateHitboxes() end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    if ESPInstances[player] then
        ESPInstances[player]:Destroy()
        ESPInstances[player] = nil
    end
    HitboxInstances[player] = nil
end)

-- Inicia threads quando ativadas
spawn(function()
    while GUIEnabled do
        if Settings.Aimbot.Enabled then
            AimbotThread()
