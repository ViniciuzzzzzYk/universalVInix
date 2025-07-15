-- RLK HUB v3.0 (LocalScript) - Para executor PC
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")

-- Configurações iniciais
repeat task.wait() until Players.LocalPlayer
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = Workspace.CurrentCamera

-- Criação da GUI
local RLKHub = Instance.new("ScreenGui")
RLKHub.Name = "RLKHub_"..tostring(math.random(1,10000))
RLKHub.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
RLKHub.ResetOnSpawn = false
RLKHub.Parent = CoreGui

-- Frame principal (arrastável)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 450, 0, 500)
MainFrame.Position = UDim2.new(0.5, -225, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
MainFrame.BackgroundTransparency = 0.15
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Visible = false
MainFrame.Parent = RLKHub

-- Função para arrastar a GUI
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
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

MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Elementos da GUI
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

local TopBar = Instance.new("Frame")
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "RLK HUB v3.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.Parent = TopBar

local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 40, 0, 40)
CloseButton.Position = UDim2.new(1, -40, 0, 0)
CloseButton.BackgroundTransparency = 1
CloseButton.Text = "×"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 24
CloseButton.Parent = TopBar

-- Sistema de abas
local TabButtons = Instance.new("Frame")
TabButtons.Name = "TabButtons"
TabButtons.Size = UDim2.new(0, 120, 1, -40)
TabButtons.Position = UDim2.new(0, 0, 0, 40)
TabButtons.BackgroundTransparency = 1
TabButtons.Parent = MainFrame

local Tabs = Instance.new("Frame")
Tabs.Name = "Tabs"
Tabs.Size = UDim2.new(1, -120, 1, -40)
Tabs.Position = UDim2.new(0, 120, 0, 40)
Tabs.BackgroundTransparency = 1
Tabs.Parent = MainFrame

-- Função para criar abas
local function CreateTab(name)
    local button = Instance.new("TextButton")
    button.Name = name.."Tab"
    button.Size = UDim2.new(1, -10, 0, 45)
    button.Position = UDim2.new(0, 5, 0, (50 * (#TabButtons:GetChildren() - 1)))
    button.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    button.BackgroundTransparency = 0.7
    button.BorderSizePixel = 0
    button.Text = name
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.GothamMedium
    button.TextSize = 14
    button.Parent = TabButtons

    Instance.new("UICorner", {CornerRadius = UDim.new(0, 6), Parent = button})

    local tab = Instance.new("ScrollingFrame")
    tab.Name = name
    tab.Size = UDim2.new(1, 0, 1, 0)
    tab.Position = UDim2.new(0, 0, 0, 0)
    tab.BackgroundTransparency = 1
    tab.Visible = false
    tab.ScrollBarThickness = 4
    tab.AutomaticCanvasSize = Enum.AutomaticSize.Y
    tab.Parent = Tabs

    Instance.new("UIListLayout", {
        Padding = UDim.new(0, 10),
        Parent = tab
    })

    return tab
end

-- Criar abas
local MovementTab = CreateTab("Movement")
local VisualTab = CreateTab("Visual")
local CombatTab = CreateTab("Combat")

-- Ativar primeira aba
MovementTab.Visible = true
TabButtons.MovementTab.BackgroundTransparency = 0.4

-- Controle de abas
for _, button in ipairs(TabButtons:GetChildren()) do
    if button:IsA("TextButton") then
        button.MouseButton1Click:Connect(function()
            for _, tab in ipairs(Tabs:GetChildren()) do
                if tab:IsA("ScrollingFrame") then tab.Visible = false end
            end
            for _, btn in ipairs(TabButtons:GetChildren()) do
                if btn:IsA("TextButton") then btn.BackgroundTransparency = 0.7 end
            end
            Tabs[button.Name:gsub("Tab", "")].Visible = true
            button.BackgroundTransparency = 0.4
        end)
    end
end

-- Toggle GUI
local GUIEnabled = false
local function ToggleGUI()
    GUIEnabled = not GUIEnabled
    MainFrame.Visible = GUIEnabled
end

CloseButton.MouseButton1Click:Connect(ToggleGUI)
UIS.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.RightShift and not gameProcessed then
        ToggleGUI()
    end
end)

-- Função para criar toggles
local function CreateToggle(parent, name, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Name = name.."Toggle"
    toggleFrame.Size = UDim2.new(1, -20, 0, 40)
    toggleFrame.Position = UDim2.new(0, 10, 0, 10 + (45 * (#parent:GetChildren() - 1)))
    toggleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    toggleFrame.BackgroundTransparency = 0.7
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = parent

    Instance.new("UICorner", {CornerRadius = UDim.new(0, 6), Parent = toggleFrame})

    Instance.new("TextLabel", {
        Name = "Label",
        Size = UDim2.new(0.7, 0, 1, 0),
        Position = UDim2.new(0, 15, 0, 0),
        BackgroundTransparency = 1,
        Text = name,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        TextXAlignment = Enum.TextXAlignment.Left,
        Font = Enum.Font.Gotham,
        TextSize = 14,
        Parent = toggleFrame
    })

    local toggleButton = Instance.new("TextButton")
    toggleButton.Name = "ToggleButton"
    toggleButton.Size = UDim2.new(0, 60, 0, 25)
    toggleButton.Position = UDim2.new(1, -70, 0.5, -12)
    toggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    toggleButton.AutoButtonColor = false
    toggleButton.BorderSizePixel = 0
    toggleButton.Text = ""
    toggleButton.Parent = toggleFrame

    local toggleCircle = Instance.new("Frame")
    toggleCircle.Name = "Circle"
    toggleCircle.Size = UDim2.new(0, 20, 0, 20)
    toggleCircle.Position = UDim2.new(0, 3, 0.5, -10)
    toggleCircle.BackgroundColor3 = Color3.fromRGB(220, 220, 220)
    toggleCircle.BorderSizePixel = 0
    toggleCircle.Parent = toggleButton

    Instance.new("UICorner", {Parent = toggleButton})
    Instance.new("UICorner", {Parent = toggleCircle})

    local state = false
    
    local function UpdateToggle()
        if state then
            TweenService:Create(toggleCircle, TweenInfo.new(0.2), {
                Position = UDim2.new(1, -23, 0.5, -10),
                BackgroundColor3 = Color3.fromRGB(0, 200, 255)
            }):Play()
            TweenService:Create(toggleButton, TweenInfo.new(0.2), {
                BackgroundColor3 = Color3.fromRGB(0, 100, 150)
            }):Play()
        else
            TweenService:Create(toggleCircle, TweenInfo.new(0.2), {
                Position = UDim2.new(0, 3, 0.5, -10),
                BackgroundColor3 = Color3.fromRGB(220, 220, 220)
            }):Play()
            TweenService:Create(toggleButton, TweenInfo.new(0.2), {
                BackgroundColor3 = Color3.fromRGB(60, 60, 70)
            }):Play()
        end
        callback(state)
    end

    toggleButton.MouseButton1Click:Connect(function()
        state = not state
        UpdateToggle()
    end)

    return {
        Set = function(value)
            state = value
            UpdateToggle()
        end,
        Get = function()
            return state
        end
    }
end

-- Função para criar sliders
local function CreateSlider(parent, name, min, max, default, callback)
    local sliderFrame = Instance.new("Frame")
    sliderFrame.Name = name.."Slider"
    sliderFrame.Size = UDim2.new(1, -20, 0, 60)
    sliderFrame.Position = UDim2.new(0, 10, 0, 10 + (65 * (#parent:GetChildren() - 1)))
    sliderFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    sliderFrame.BackgroundTransparency = 0.7
    sliderFrame.BorderSizePixel = 0
    sliderFrame.Parent = parent

    Instance.new("UICorner", {CornerRadius = UDim.new(0, 6), Parent = sliderFrame})

    local sliderLabel = Instance.new("TextLabel")
    sliderLabel.Name = "Label"
    sliderLabel.Size = UDim2.new(1, -20, 0, 20)
    sliderLabel.Position = UDim2.new(0, 10, 0, 5)
    sliderLabel.BackgroundTransparency = 1
    sliderLabel.Text = name..": "..default
    sliderLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    sliderLabel.TextXAlignment = Enum.TextXAlignment.Left
    sliderLabel.Font = Enum.Font.Gotham
    sliderLabel.TextSize = 14
    sliderLabel.Parent = sliderFrame

    local sliderTrack = Instance.new("Frame")
    sliderTrack.Name = "Track"
    sliderTrack.Size = UDim2.new(1, -20, 0, 6)
    sliderTrack.Position = UDim2.new(0, 10, 0, 35)
    sliderTrack.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    sliderTrack.BorderSizePixel = 0
    sliderTrack.Parent = sliderFrame

    local sliderFill = Instance.new("Frame")
    sliderFill.Name = "Fill"
    sliderFill.Size = UDim2.new((default - min)/(max - min), 0, 1, 0)
    sliderFill.Position = UDim2.new(0, 0, 0, 0)
    sliderFill.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
    sliderFill.BorderSizePixel = 0
    sliderFill.Parent = sliderTrack

    local sliderButton = Instance.new("TextButton")
    sliderButton.Name = "Button"
    sliderButton.Size = UDim2.new(0, 16, 0, 16)
    sliderButton.Position = UDim2.new((default - min)/(max - min), -8, 0.5, -8)
    sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    sliderButton.AutoButtonColor = false
    sliderButton.Text = ""
    sliderButton.Parent = sliderTrack

    Instance.new("UICorner", {Parent = sliderTrack})
    Instance.new("UICorner", {Parent = sliderFill})
    Instance.new("UICorner", {Parent = sliderButton})

    local dragging = false
    local currentValue = default
    
    local function UpdateSlider(value)
        value = math.clamp(value, min, max)
        currentValue = value
        sliderFill.Size = UDim2.new((value - min)/(max - min), 0, 1, 0)
        sliderButton.Position = UDim2.new((value - min)/(max - min), -8, 0.5, -8)
        sliderLabel.Text = name..": "..math.floor(value)
        callback(value)
    end
    
    sliderButton.MouseButton1Down:Connect(function()
        dragging = true
    end)
    
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    UIS.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local xScale = (input.Position.X - sliderTrack.AbsolutePosition.X) / sliderTrack.AbsoluteSize.X
            UpdateSlider(min + (max - min) * math.clamp(xScale, 0, 1))
        end
    end)
    
    return {
        Set = function(value)
            UpdateSlider(value)
        end,
        Get = function()
            return currentValue
        end
    }
end

-- Sistema de Fly
local FlyEnabled = false
local FlySpeed = 50
local FlyBodyVelocity

local FlyToggle = CreateToggle(MovementTab, "Fly", function(state)
    FlyEnabled = state
    
    if state then
        if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            FlyToggle.Set(false)
            return
        end
        
        FlyBodyVelocity = Instance.new("BodyVelocity")
        FlyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        FlyBodyVelocity.MaxForce = Vector3.new(0, 0, 0)
        FlyBodyVelocity.Parent = LocalPlayer.Character.HumanoidRootPart
        
        LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Physics)
    else
        if FlyBodyVelocity then
            FlyBodyVelocity:Destroy()
            FlyBodyVelocity = nil
        end
        
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
        end
    end
end)

local FlySpeedSlider = CreateSlider(MovementTab, "Fly Speed", 20, 150, 50, function(value)
    FlySpeed = value
end)

-- Sistema de Speed
local SpeedEnabled = false
local SpeedValue = 50

local SpeedToggle = CreateToggle(MovementTab, "Speed Hack", function(state)
    SpeedEnabled = state
    
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = state and SpeedValue or 16
    end
end)

local SpeedSlider = CreateSlider(MovementTab, "WalkSpeed", 16, 100, 50, function(value)
    SpeedValue = value
    if SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = value
    end
end)

-- Infinite Jump (One-time toggle)
local InfiniteJumpEnabled = false
local InfiniteJumpConnection

local InfiniteJumpButton = Instance.new("TextButton")
InfiniteJumpButton.Name = "InfiniteJumpButton"
InfiniteJumpButton.Size = UDim2.new(1, -20, 0, 40)
InfiniteJumpButton.Position = UDim2.new(0, 10, 0, 10 + (65 * (#MovementTab:GetChildren() - 1)))
InfiniteJumpButton.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
InfiniteJumpButton.BackgroundTransparency = 0.7
InfiniteJumpButton.BorderSizePixel = 0
InfiniteJumpButton.Text = "TOGGLE INFINITE JUMP"
InfiniteJumpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
InfiniteJumpButton.Font = Enum.Font.GothamBold
InfiniteJumpButton.TextSize = 14
InfiniteJumpButton.Parent = MovementTab

Instance.new("UICorner", {CornerRadius = UDim.new(0, 6), Parent = InfiniteJumpButton})

InfiniteJumpButton.MouseButton1Click:Connect(function()
    InfiniteJumpEnabled = not InfiniteJumpEnabled
    
    if InfiniteJumpEnabled then
        InfiniteJumpButton.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
        InfiniteJumpButton.Text = "INFINITE JUMP: ON"
        
        if InfiniteJumpConnection then
            InfiniteJumpConnection:Disconnect()
        end
        
        InfiniteJumpConnection = UIS.JumpRequest:Connect(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    else
        InfiniteJumpButton.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
        InfiniteJumpButton.Text = "TOGGLE INFINITE JUMP"
        
        if InfiniteJumpConnection then
            InfiniteJumpConnection:Disconnect()
            InfiniteJumpConnection = nil
        end
    end
end)

-- Sistema de ESP
local ESPEnabled = false
local ESPHighlights = {}

local function UpdateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if ESPEnabled then
                if player.Character then
                    if not ESPHighlights[player] then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = "RLKESP"
                        highlight.FillColor = Color3.fromRGB(255, 50, 50)
                        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                        highlight.FillTransparency = 0.5
                        highlight.OutlineTransparency = 0
                        highlight.Parent = player.Character
                        ESPHighlights[player] = highlight
                    end
                end
            else
                if ESPHighlights[player] then
                    ESPHighlights[player]:Destroy()
                    ESPHighlights[player] = nil
                end
            end
        end
    end
end

local ESPToggle = CreateToggle(VisualTab, "ESP", function(state)
    ESPEnabled = state
    UpdateESP()
end)

-- Sistema de Aimbot
local AimbotEnabled = false
local AimbotSmoothness = 0.5
local AimbotRange = 100
local AimbotTeamCheck = false
local AimbotTarget = nil

local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = AimbotRange
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not AimbotTeamCheck or player.Team ~= LocalPlayer.Team then
                local distance = (player.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                if distance < shortestDistance then
                    local screenPoint = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
                    if screenPoint.Z > 0 then -- Verifica se está na frente da câmera
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

local AimbotToggle = CreateToggle(CombatTab, "Aimbot", function(state)
    AimbotEnabled = state
end)

local AimbotSmoothSlider = CreateSlider(CombatTab, "Aimbot Smoothness", 0.1, 1, 0.5, function(value)
    AimbotSmoothness = value
end)

local AimbotRangeSlider = CreateSlider(CombatTab, "Aimbot Range", 10, 500, 100, function(value)
    AimbotRange = value
end)

local AimbotTeamToggle = CreateToggle(CombatTab, "Ignore Team", function(state)
    AimbotTeamCheck = state
end)

-- Loop do Aimbot
RunService.RenderStepped:Connect(function()
    if AimbotEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        AimbotTarget = GetClosestPlayer()
        
        if AimbotTarget and AimbotTarget.Character and AimbotTarget.Character:FindFirstChild("HumanoidRootPart") then
            local targetPos = AimbotTarget.Character.HumanoidRootPart.Position
            local cameraPos = Camera.CFrame.Position
            local direction = (targetPos - cameraPos).Unit
            
            -- Suavização do movimento
            local currentLook = Camera.CFrame.LookVector
            local newLook = currentLook:Lerp(direction, 1 - AimbotSmoothness)
            
            Camera.CFrame = CFrame.new(cameraPos, cameraPos + newLook)
        end
    end
end)

-- Atualizações quando o personagem spawna
local function CharacterAdded(character)
    if FlyEnabled then
        FlyToggle.Set(true)
    end
    
    if SpeedEnabled then
        character:WaitForChild("Humanoid").WalkSpeed = SpeedValue
    end
    
    if InfiniteJumpEnabled then
        InfiniteJumpConnection = UIS.JumpRequest:Connect(function()
            character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end)
    end
    
    character.Humanoid.Died:Connect(function()
        if InfiniteJumpConnection then
            InfiniteJumpConnection:Disconnect()
            InfiniteJumpConnection = nil
        end
    end)
end

LocalPlayer.CharacterAdded:Connect(CharacterAdded)

-- Atualiza quando jogadores entram
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if ESPEnabled then
            local highlight = Instance.new("Highlight")
            highlight.Name = "RLKESP"
            highlight.FillColor = Color3.fromRGB(255, 50, 50)
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.FillTransparency = 0.5
            highlight.OutlineTransparency = 0
            highlight.Parent = character
            ESPHighlights[player] = highlight
        end
    end)
end)

-- Loop do Fly
RunService.Heartbeat:Connect(function()
    if FlyEnabled and FlyBodyVelocity and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local root = LocalPlayer.Character.HumanoidRootPart
        local cam = Camera.CFrame
        
        local forward = (cam.LookVector * Vector3.new(1, 0, 1)).Unit
        local right = (cam.RightVector * Vector3.new(1, 0, 1)).Unit
        
        local velocity = Vector3.new(0, 0, 0)
        
        if UIS:IsKeyDown(Enum.KeyCode.W) then velocity = velocity + forward end
        if UIS:IsKeyDown(Enum.KeyCode.S) then velocity = velocity - forward end
        if UIS:IsKeyDown(Enum.KeyCode.D) then velocity = velocity + right end
        if UIS:IsKeyDown(Enum.KeyCode.A) then velocity = velocity - right end
        if UIS:IsKeyDown(Enum.KeyCode.Space) then velocity = velocity + Vector3.new(0, 1, 0) end
        if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then velocity = velocity - Vector3.new(0, 1, 0) end
        
        if velocity.Magnitude > 0 then
            velocity = velocity.Unit * FlySpeed
        end
        
        FlyBodyVelocity.Velocity = velocity
        FlyBodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
    end
end)

-- Inicialização
if LocalPlayer.Character then
    CharacterAdded(LocalPlayer.Character)
end

-- Atualiza ESP para jogadores existentes
UpdateESP()

-- Auto-update canvas size
for _, tab in ipairs(Tabs:GetChildren()) do
    if tab:IsA("ScrollingFrame") then
        tab:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
            tab.CanvasSize = UDim2.new(0, 0, 0, tab.UIListLayout.AbsoluteContentSize.Y + 20)
        end)
    end
end

-- Mostra a GUI após carregar
task.wait(1)
ToggleGUI()
