-- GF Arena Mobile v3.0
-- Otimizado para Delta Executor
-- GUI 100% funcional

-- Verificação inicial
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- Serviços essenciais
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")

-- Remove GUIs anteriores
if CoreGui:FindFirstChild("GFAMobileGUI") then
    CoreGui.GFAMobileGUI:Destroy()
end

--- FUNÇÃO PRINCIPAL PARA CRIAR GUI ---
local function CreateMainGUI()
    -- Cria a GUI principal
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "GFAMobileGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.DisplayOrder = 999
    
 Frame principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 320, 0, 400)
    MainFrame.Position = UDim2.new(0.5, -160, 0.5, -200)
    MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    MainFrame.BackgroundTransparency = 0.15
    MainFrame.BorderSizePixel = 0
    MainFrame.Visible = true
    MainFrame.Parent = ScreenGui

 Efeitos visuais
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 12)
    UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
    UIStroke.Color = Color3.fromRGB(80, 80, 120)
    UIStroke.Thickness = 2
    UIStroke.Parent = MainFrame

 Barra de título
    local TitleBar = Instance.new("Frame")
    TitleBar.Name = "TitleBar"
    TitleBar.Size = UDim2.new(1, 0, 0, 40)
    TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
    TitleCorner.CornerRadius = UDim.new(0, 12)
    TitleCorner.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Name = "TitleLabel"
    TitleLabel.Size = UDim2.new(1, -50, 1, 0)
    TitleLabel.Position = UDim2.new(0, 15, 0, 0)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.Text = "GF Arena Mobile"
    TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    TitleLabel.TextSize = 18
    TitleLabel.Font = Enum.Font.GothamBold
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    TitleLabel.Parent = TitleBar

 Botão de fechar
    local CloseButton = Instance.new("TextButton")
    CloseButton.Name = "CloseButton"
    CloseButton.Size = UDim2.new(0, 30, 0, 30)
    CloseButton.Position = UDim2.new(1, -35, 0.5, -15)
    CloseButton.AnchorPoint = Vector2.new(0.5, 0.5)
    CloseButton.BackgroundColor3 = Color3.fromRGB(180, 60, 60)
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.Text = "×"
    CloseButton.TextSize = 24
    CloseButton.Font = Enum.Font.GothamBold
    CloseButton.Parent = TitleBar

 Efeito hover no botão
    CloseButton.MouseEnter:Connect(function()
        CloseButton.BackgroundColor3 = Color3.fromRGB(200, 80, 80)
    end)
    
CloseButton.MouseLeave:Connect(function()
        CloseButton.BackgroundColor3 = Color3.fromRGB(180, 60, 60)
    end)

 Corpo principal
    local BodyFrame = Instance.new("Frame")
    BodyFrame.Name = "BodyFrame"
    BodyFrame.Size = UDim2.new(1, -20, 1, -80)
    BodyFrame.Position = UDim2.new(0, 10, 0, 70)
    BodyFrame.BackgroundTransparency = 1
    BodyFrame.Parent = MainFrame

 Botão de teste (para confirmar funcionamento)
    local TestButton = Instance.new("TextButton")
    TestButton.Name = "TestButton"
    TestButton.Size = UDim2.new(1, 0, 0, 45)
    TestButton.Position = UDim2.new(0, 0, 0, 10)
    TestButton.BackgroundColor3 = Color3.fromRGB(60, 60, 90)
    TestButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    TestButton.Text = "GUI CARREGADA COM SUCESSO!"
    TestButton.TextSize = 14
    TestButton.Font = Enum.Font.GothamBold
    TestButton.Parent = BodyFrame

 Efeito no botão
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 8)
    ButtonCorner.Parent = TestButton

 Sistema de arrastar
    local UserInputService = game:GetService("UserInputService")
    local dragging, dragInput, dragStart, startPos

local function UpdateInput(input)
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(
            startPos.X.Scale, 
            startPos.X.Offset + delta.X,
            startPos.Y.Scale, 
            startPos.Y.Offset + delta.Y
        )
    end

TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
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
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            UpdateInput(input)
        end
    end)

 Fechar GUI
    CloseButton.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)

 Adiciona ao CoreGui
    ScreenGui.Parent = CoreGui

 Efeito de entrada
    MainFrame.Position = UDim2.new(0.5, -160, 0, -400)
    MainFrame.Visible = true
    
local tween = TweenService:Create(
        MainFrame,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Position = UDim2.new(0.5, -160, 0.5, -200)}
    )
    tween:Play()

 return ScreenGui
end

--- EXECUÇÃO PRINCIPAL ---
local success, err = pcall(function()
    -- Cria a GUI
    local GUI = CreateMainGUI()
    
 Verificação final
    wait(0.5)
    
if GUI and GUI.Parent then
        print("GUI criada com sucesso no Delta Executor!")
    else
        warn("Falha ao criar GUI")
    end
end)

if not success then
    warn("Erro ao executar script:", err)
    
 Fallback simples
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "GF Arena Mobile",
        Text = "Erro ao criar interface completa",
        Duration = 5
    })
end
