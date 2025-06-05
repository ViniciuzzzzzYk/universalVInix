-- Verifica se o jogo carregou
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- Biblioteca UI otimizada para mobile
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()

-- Cria a janela principal
local Window = Rayfield:CreateWindow({
    Name = "GF Arena Mobile",
    LoadingTitle = "Carregando exploit...",
    LoadingSubtitle = "by Test Scripts",
    ConfigurationSaving = { Enabled = false },
    Discord = { Enabled = false }
})

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")

-- Configurações
local Settings = {
    ESP = {
        Enabled = false,
        Color = Color3.fromRGB(255, 50, 50),
        TeamCheck = true
    },
    Hitbox = {
        Enabled = false,
        Size = Vector3.new(5, 5, 5)
    },
    Aimbot = {
        Enabled = false,
        Smoothness = 0.1,
        FOV = 100,
        TeamCheck = true
    },
    AutoFarm = {
        Enabled = false,
        Delay = 1
    }
}

-- Tabelas de armazenamento
local ESPInstances = {}
local Hitboxes = {}

-- Função para criar ESP
local function UpdateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character then
                if Settings.ESP.Enabled then
                    if not ESPInstances[player] then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = player.Name.."_ESP"
                        highlight.FillColor = Settings.ESP.Color
                        highlight.OutlineColor = Settings.ESP.Color
                        highlight.FillTransparency = 0.5
                        highlight.Parent = character
                        ESPInstances[player] = highlight
                    end
                    ESPInstances[player].Adornee = character
                    ESPInstances[player].Parent = character
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

-- Função para Hitbox Expander
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
                        Hitboxes[player] = true
                    else
                        if Hitboxes[player] then
                            humanoidRootPart.Size = Vector3.new(2, 2, 1)
                            humanoidRootPart.Transparency = 0
                            Hitboxes[player] = nil
                        end
                    end
                end
            end
        end
    end
end

-- Função para Aimbot
local function AimbotThread()
    while Settings.Aimbot.Enabled do
        task.wait()
        local closestPlayer, closestDistance = nil, math.huge
        
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
                if humanoidRootPart then
                    local screenPoint = Camera:WorldToViewportPoint(humanoidRootPart.Position)
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                    
                    if distance < closestDistance and distance < Settings.Aimbot.FOV then
                        closestPlayer = player
                        closestDistance = distance
                    end
                end
            end
        end
        
        if closestPlayer and closestPlayer.Character then
            local targetPart = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
            if targetPart then
                local camCFrame = Camera.CFrame
                local targetPosition = targetPart.Position + Vector3.new(0, 1.5, 0)
                local newCFrame = camCFrame:Lerp(CFrame.lookAt(camCFrame.Position, targetPosition), Settings.Aimbot.Smoothness)
                Camera.CFrame = newCFrame
            end
        end
    end
end

-- Função para AutoFarm
local function AutoFarmThread()
    while Settings.AutoFarm.Enabled do
        task.wait(Settings.AutoFarm.Delay)
        -- Implemente sua lógica de farm aqui
    end
end

-- Cria as abas
local VisualTab = Window:CreateTab("Visual", 4483362458)
local CombatTab = Window:CreateTab("Combate", 4483362458)
local FarmTab = Window:CreateTab("Farm", 4483362458)

-- Seção Visual
VisualTab:CreateToggle({
    Name = "ESP",
    CurrentValue = false,
    Callback = function(value)
        Settings.ESP.Enabled = value
        UpdateESP()
    end
})

VisualTab:CreateColorPicker({
    Name = "Cor do ESP",
    Color = Settings.ESP.Color,
    Callback = function(value)
        Settings.ESP.Color = value
        for _, esp in pairs(ESPInstances) do
            esp.FillColor = value
            esp.OutlineColor = value
        end
    end
})

VisualTab:CreateToggle({
    Name = "Hitbox Expander",
    CurrentValue = false,
    Callback = function(value)
        Settings.Hitbox.Enabled = value
        UpdateHitboxes()
    end
})

-- Seção Combate
CombatTab:CreateToggle({
    Name = "Aimbot",
    CurrentValue = false,
    Callback = function(value)
        Settings.Aimbot.Enabled = value
        if value then
            coroutine.wrap(AimbotThread)()
        end
    end
})

CombatTab:CreateSlider({
    Name = "Suavidade do Aimbot",
    Range = {0.1, 1},
    Increment = 0.1,
    Suffix = "x",
    CurrentValue = Settings.Aimbot.Smoothness,
    Callback = function(value)
        Settings.Aimbot.Smoothness = value
    end
})

CombatTab:CreateSlider({
    Name = "FOV do Aimbot",
    Range = {50, 500},
    Increment = 10,
    Suffix = "px",
    CurrentValue = Settings.Aimbot.FOV,
    Callback = function(value)
        Settings.Aimbot.FOV = value
    end
})

-- Seção Farm
FarmTab:CreateToggle({
    Name = "AutoFarm",
    CurrentValue = false,
    Callback = function(value)
        Settings.AutoFarm.Enabled = value
        if value then
            coroutine.wrap(AutoFarmThread)()
        end
    end
})

FarmTab:CreateSlider({
    Name = "Delay do AutoFarm",
    Range = {0.1, 5},
    Increment = 0.1,
    Suffix = "s",
    CurrentValue = Settings.AutoFarm.Delay,
    Callback = function(value)
        Settings.AutoFarm.Delay = value
    end
})

-- Conexões de eventos
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if Settings.ESP.Enabled then UpdateESP() end
        if Settings.Hitbox.Enabled then UpdateHitboxes() end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    if ESPInstances[player] then
        ESPInstances[player]:Destroy()
        ESPInstances[player] = nil
    end
    Hitboxes[player] = nil
end)

-- Loop principal para atualizar features
RunService.Heartbeat:Connect(function()
    if Settings.ESP.Enabled then UpdateESP() end
    if Settings.Hitbox.Enabled then UpdateHitboxes() end
end)

Rayfield:Notify({
    Title = "GF Arena Mobile",
    Content = "Script carregado com sucesso!",
    Duration = 5,
    Image = 4483362458
})
