-- RLK HUB (LocalScript) - Exclusivo para executor PC
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- Verificação segura do LocalPlayer
repeat task.wait() until Players.LocalPlayer
local LocalPlayer = Players.LocalPlayer

-- Criação da GUI
local RLKHub = Instance.new("ScreenGui")
RLKHub.Name = "RLKHub_"..tostring(math.random(1,10000)) -- Nome único
RLKHub.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
RLKHub.ResetOnSpawn = false
RLKHub.Parent = CoreGui

-- Função para criar elementos com fallback
local function CreateInstance(className, props)
    local obj = Instance.new(className)
    for prop, value in pairs(props) do
        obj[prop] = value
    end
    return obj
end

-- Frame principal
local MainFrame = CreateInstance("Frame", {
    Name = "MainFrame",
    Size = UDim2.new(0, 400, 0, 450),
    Position = UDim2.new(0.5, -200, 0.5, -225),
    BackgroundColor3 = Color3.fromRGB(20, 20, 25),
    BackgroundTransparency = 0.15,
    BorderSizePixel = 0,
    ClipsDescendants = true,
    Parent = RLKHub
})

CreateInstance("UICorner", {
    CornerRadius = UDim.new(0, 12),
    Parent = MainFrame
})

-- Barra superior
local TopBar = CreateInstance("Frame", {
    Name = "TopBar",
    Size = UDim2.new(1, 0, 0, 40),
    BackgroundColor3 = Color3.fromRGB(30, 30, 40),
    BorderSizePixel = 0,
    Parent = MainFrame
})

CreateInstance("TextLabel", {
    Name = "Title",
    Size = UDim2.new(0, 200, 1, 0),
    Position = UDim2.new(0, 15, 0, 0),
    BackgroundTransparency = 1,
    Text = "RLK HUB v2.1",
    TextColor3 = Color3.fromRGB(255, 255, 255),
    TextXAlignment = Enum.TextXAlignment.Left,
    Font = Enum.Font.GothamBold,
    TextSize = 16,
    Parent = TopBar
})

local CloseButton = CreateInstance("TextButton", {
    Name = "CloseButton",
    Size = UDim2.new(0, 40, 0, 40),
    Position = UDim2.new(1, -40, 0, 0),
    BackgroundTransparency = 1,
    Text = "×",
    TextColor3 = Color3.fromRGB(255, 255, 255),
    Font = Enum.Font.GothamBold,
    TextSize = 24,
    Parent = TopBar
})

-- Tabs
local TabButtons = CreateInstance("Frame", {
    Name = "TabButtons",
    Size = UDim2.new(0, 120, 1, -40),
    Position = UDim2.new(0, 0, 0, 40),
    BackgroundTransparency = 1,
    Parent = MainFrame
})

local Tabs = CreateInstance("Frame", {
    Name = "Tabs",
    Size = UDim2.new(1, -120, 1, -40),
    Position = UDim2.new(0, 120, 0, 40),
    BackgroundTransparency = 1,
    Parent = MainFrame
})

-- Função para criar tabs
local function CreateTab(name)
    local button = CreateInstance("TextButton", {
        Name = name.."Tab",
        Size = UDim2.new(1, -10, 0, 45),
        Position = UDim2.new(0, 5, 0, (50 * (#TabButtons:GetChildren() - 1))),
        BackgroundColor3 = Color3.fromRGB(35, 35, 45),
        BackgroundTransparency = 0.7,
        BorderSizePixel = 0,
        Text = name,
        TextColor3 = Color3.fromRGB(255, 255, 255),
        Font = Enum.Font.GothamMedium,
        TextSize = 14,
        Parent = TabButtons
    })

    CreateInstance("UICorner", {CornerRadius = UDim.new(0, 6), Parent = button})

    local tab = CreateInstance("ScrollingFrame", {
        Name = name,
        Size = UDim2.new(1, 0, 1, 0),
        Position = UDim2.new(0, 0, 0, 0),
        BackgroundTransparency = 1,
        Visible = false,
        ScrollBarThickness = 4,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = Tabs
    })

    CreateInstance("UIListLayout", {
        Padding = UDim.new(0, 10),
        Parent = tab
    })

    return tab
end

-- Criar abas
local MovementTab = CreateTab("Movement")
local VisualTab = CreateTab("Visual")

-- Ativar primeira aba
MovementTab.Visible = true
TabButtons.MovementTab.BackgroundTransparency = 0.4

-- Controle de tabs
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
    local toggleFrame = CreateInstance("Frame", {
        Name = name.."Toggle",
        Size = UDim2.new(1, -20, 0, 40),
        Position = UDim2.new(0, 10, 0, 10 + (45 * (#parent:GetChildren() - 1))),
        BackgroundColor3 = Color3.fromRGB(30, 30, 40),
        BackgroundTransparency = 0.7,
        BorderSizePixel = 0,
        Parent = parent
    })

    CreateInstance("UICorner", {CornerRadius = UDim.new(0, 6), Parent = toggleFrame})

    CreateInstance("TextLabel", {
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

    local toggleButton = CreateInstance("TextButton", {
        Name = "ToggleButton",
        Size = UDim2.new(0, 60, 0, 25),
        Position = UDim2.new(1, -70, 0.5, -12),
        BackgroundColor3 = Color3.fromRGB(60, 60, 70),
        AutoButtonColor = false,
        BorderSizePixel = 0,
        Text = "",
        Parent = toggleFrame
    })

    local toggleCircle = CreateInstance("Frame", {
        Name = "Circle",
        Size = UDim2.new(0, 20, 0, 20),
        Position = UDim2.new(0, 3, 0.5, -10),
        BackgroundColor3 = Color3.fromRGB(220, 220, 220),
        BorderSizePixel = 0,
        Parent = toggleButton
    })

    CreateInstance("UICorner", {Parent = toggleButton})
    CreateInstance("UICorner", {Parent = toggleCircle})

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

-- Sistema de Speed
local SpeedEnabled = false
local SpeedValue = 50

local SpeedToggle = CreateToggle(MovementTab, "Speed Hack", function(state)
    SpeedEnabled = state
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = state and SpeedValue or 16
    end
end)

-- Infinite Jump (One-time toggle)
local InfiniteJumpEnabled = false
local InfiniteJumpConnection

local InfiniteJumpButton = CreateInstance("TextButton", {
    Name = "InfiniteJumpButton",
    Size = UDim2.new(1, -20, 0, 40),
    Position = UDim2.new(0, 10, 0, 10 + (65 * (#MovementTab:GetChildren() - 1))),
    BackgroundColor3 = Color3.fromRGB(0, 150, 200),
    BackgroundTransparency = 0.7,
    BorderSizePixel = 0,
    Text = "TOGGLE INFINITE JUMP",
    TextColor3 = Color3.fromRGB(255, 255, 255),
    Font = Enum.Font.GothamBold,
    TextSize = 14,
    Parent = MovementTab
})

CreateInstance("UICorner", {CornerRadius = UDim.new(0, 6), Parent = InfiniteJumpButton})

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

-- ESP System
local ESPEnabled = false
local ESPHighlights = {}

local function UpdateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if ESPEnabled then
                if player.Character and not ESPHighlights[player] then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "RLKESP"
                    highlight.FillColor = Color3.fromRGB(255, 50, 50)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineTransparency = 0
                    highlight.Parent = player.Character
                    ESPHighlights[player] = highlight
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

-- Handle character added
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

-- Handle players added
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

-- Fly movement handler
RunService.Heartbeat:Connect(function()
    if FlyEnabled and FlyBodyVelocity and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local root = LocalPlayer.Character.HumanoidRootPart
        local cam = workspace.CurrentCamera.CFrame
        
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

-- Initialize if character already exists
if LocalPlayer.Character then
    CharacterAdded(LocalPlayer.Character)
end

-- Initialize ESP for existing players
UpdateESP()

-- Auto-update canvas size
for _, tab in ipairs(Tabs:GetChildren()) do
    if tab:IsA("ScrollingFrame") then
        tab:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
            tab.CanvasSize = UDim2.new(0, 0, 0, tab.UIListLayout.AbsoluteContentSize.Y + 20)
        end)
    end
end

-- Mostra a GUI após 1 segundo (para garantir que tudo carregou)
task.wait(1)
ToggleGUI()
