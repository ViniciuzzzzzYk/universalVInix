--[[
  Gunfight Arena Exploit v1.3
  Interface mobile-friendly
  Funciona em Delta, Hydrogen, Fluxus
]]

-- Verifica se o jogo carregou
repeat wait() until game:IsLoaded()

-- Carrega a biblioteca de interface
local success, library = pcall(function()
    return loadstring(game:HttpGet("https://raw.githubusercontent.com/violin-suzutsuki/LinoriaLib/main/Library.lua"))()
end)

if not success then
    warn("Falha ao carregar a biblioteca de interface")
    return
end

-- Cria a janela principal
local Window = library:CreateWindow({
    Title = "GF Arena Mobile",
    Center = true, 
    AutoShow = true,
    TabPadding = 8,
    MenuFadeTime = 0.2
})

-- Cria as abas
local VisualTab = Window:AddTab("Visual")
local CombatTab = Window:AddTab("Combate")
local FarmTab = Window:AddTab("Farm")

-- Variáveis de configuração
local Config = {
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
        Smoothness = 0.2,
        FOV = 120,
        TeamCheck = true
    },
    AutoFarm = {
        Enabled = false,
        Delay = 0.5
    }
}

-- Elementos da interface

-- Tab Visual
local ESPToggle = VisualTab:AddToggle("ESP", {
    Text = "ESP (Visualização)",
    Default = false,
    Tooltip = "Mostra jogadores através das paredes",
    Callback = function(value)
        Config.ESP.Enabled = value
        library:Notify("ESP " .. (value and "ativado" or "desativado"))
    end
})

local ESPColor = VisualTab:AddColorPicker("ESPColor", {
    Title = "Cor do ESP",
    Default = Config.ESP.Color,
    Callback = function(value)
        Config.ESP.Color = value
    end
})

local HitboxToggle = VisualTab:AddToggle("HitboxExpander", {
    Text = "Hitbox Expander",
    Default = false,
    Tooltip = "Aumenta a hitbox dos inimigos",
    Callback = function(value)
        Config.Hitbox.Enabled = value
        library:Notify("Hitbox Expander " .. (value and "ativado" or "desativado"))
    end
})

-- Tab Combate
local AimbotToggle = CombatTab:AddToggle("Aimbot", {
    Text = "Aimbot",
    Default = false,
    Tooltip = "Mira automática nos inimigos",
    Callback = function(value)
        Config.Aimbot.Enabled = value
        library:Notify("Aimbot " .. (value and "ativado" or "desativado"))
    end
})

local AimbotSmoothness = CombatTab:AddSlider("AimbotSmoothness", {
    Text = "Suavidade",
    Default = Config.Aimbot.Smoothness * 10,
    Min = 1,
    Max = 10,
    Rounding = 1,
    Callback = function(value)
        Config.Aimbot.Smoothness = value / 10
    end
})

local AimbotFOV = CombatTab:AddSlider("AimbotFOV", {
    Text = "Campo de Visão",
    Default = Config.Aimbot.FOV,
    Min = 50,
    Max = 300,
    Rounding = 0,
    Callback = function(value)
        Config.Aimbot.FOV = value
    end
})

-- Tab Farm
local AutoFarmToggle = FarmTab:AddToggle("AutoFarm", {
    Text = "Auto Farm",
    Default = false,
    Tooltip = "Farm automático de pontos",
    Callback = function(value)
        Config.AutoFarm.Enabled = value
        library:Notify("Auto Farm " .. (value and "ativado" or "desativado"))
    end
})

local AutoFarmDelay = FarmTab:AddSlider("AutoFarmDelay", {
    Text = "Intervalo",
    Default = Config.AutoFarm.Delay * 10,
    Min = 1,
    Max = 50,
    Rounding = 1,
    Callback = function(value)
        Config.AutoFarm.Delay = value / 10
    end
})

-- Botão de fechar
local UnloadButton = Window:AddButton("Descarregar", function()
    library:Unload()
end)

-- Atualiza a interface para mobile
library:SetWatermarkVisibility(true)
library:SetWatermark("GF Arena Mobile v1.3")

-- Aplica configurações mobile
library:OnUnload(function()
    print("Interface descarregada")
end)

-- Notificação inicial
library:Notify("Interface carregada com sucesso!", 5)

-- Inicializa a interface
library:Init()
