-- RLK HUB v2.0 (LocalScript) - Paste into StarterPlayerScripts
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Modern GUI Setup
local RLKHub = Instance.new("ScreenGui")
RLKHub.Name = "RLKHub"
RLKHub.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
RLKHub.ResetOnSpawn = false
RLKHub.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 400, 0, 450)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -225)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
MainFrame.BackgroundTransparency = 0.15
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Visible = false -- Starts hidden
MainFrame.Parent = RLKHub

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
Title.Text = "RLK HUB v2.0"
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

-- Create Tabs
local function createTab(name)
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
    
    local tab = Instance.new("ScrollingFrame")
    tab.Name = name
    tab.Size = UDim2.new(1, 0, 1, 0)
    tab.Position = UDim2.new(0, 0, 0, 0)
    tab.BackgroundTransparency = 1
    tab.Visible = false
    tab.ScrollBarThickness = 4
    tab.AutomaticCanvasSize = Enum.AutomaticSize.Y
    tab.CanvasSize = UDim2.new(0, 0, 0, 0)
    tab.Parent = Tabs
    
    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.Padding = UDim.new(0, 10)
    UIListLayout.Parent = tab
    
    UICorner:Clone().Parent = button
    
    return tab
end

-- Create Tabs
local MovementTab = createTab("Movement")
local VisualTab = createTab("Visual")

-- Make first tab visible
MovementTab.Visible = true
TabButtons:FindFirstChild("MovementTab").BackgroundTransparency = 0.4

-- Tab Switching
for _, button in ipairs(TabButtons:GetChildren()) do
    if button:IsA("TextButton") then
        button.MouseButton1Click:Connect(function()
            for _, tab in ipairs(Tabs:GetChildren()) do
                if tab:IsA("ScrollingFrame") then
                    tab.Visible = false
                end
            end
            for _, btn in ipairs(TabButtons:GetChildren()) do
                if btn:IsA("TextButton") then
                    btn.BackgroundTransparency = 0.7
                end
            end
            
            Tabs[button.Name:gsub("Tab", "")].Visible = true
            button.BackgroundTransparency = 0.4
        end)
    end
end

-- Toggle GUI Visibility
local guiVisible = false
local function toggleGUI()
    guiVisible = not guiVisible
    MainFrame.Visible = guiVisible
end

CloseButton.MouseButton1Click:Connect(toggleGUI)

UIS.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.RightShift and not gameProcessed then
        toggleGUI()
    end
end)

-- Modern Toggle Function
local function createToggle(parent, name, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Name = name.."Toggle"
    toggleFrame.Size = UDim2.new(1, -20, 0, 40)
    toggleFrame.Position = UDim2.new(0, 10, 0, 10 + (45 * (#parent:GetChildren() - 1)))
    toggleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    toggleFrame.BackgroundTransparency = 0.7
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = parent
    
    UICorner:Clone().Parent = toggleFrame
    
    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Name = "Label"
    toggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    toggleLabel.Position = UDim2.new(0, 15, 0, 0)
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.Text = name
    toggleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    toggleLabel.Font = Enum.Font.Gotham
    toggleLabel.TextSize = 14
    toggleLabel.Parent = toggleFrame
    
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
    
    UICorner:Clone().Parent = toggleButton
    UICorner:Clone().Parent = toggleCircle
    
    local state = false
    
    local function updateToggle()
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
        updateToggle()
    end)
    
    return {
        Set = function(value)
            state = value
            updateToggle()
        end,
        Get = function()
            return state
        end
    }
end

-- Slider Function
local function createSlider(parent, name, min, max, default, callback)
    local sliderFrame = Instance.new("Frame")
    sliderFrame.Name = name.."Slider"
    sliderFrame.Size = UDim2.new(1, -20, 0, 60)
    sliderFrame.Position = UDim2.new(0, 10, 0, 10 + (65 * (#parent:GetChildren() - 1)))
    sliderFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    sliderFrame.BackgroundTransparency = 0.7
    sliderFrame.BorderSizePixel = 0
    sliderFrame.Parent = parent
    
    UICorner:Clone().Parent = sliderFrame
    
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
    
    UICorner:Clone().Parent = sliderTrack
    UICorner:Clone().Parent = sliderFill
    UICorner:Clone().Parent = sliderButton
    
    local dragging = false
    local currentValue = default
    
    local function updateSlider(value)
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
            updateSlider(min + (max - min) * math.clamp(xScale, 0, 1))
        end
    end)
    
    return {
        Set = function(value)
            updateSlider(value)
        end,
        Get = function()
            return currentValue
        end
    }
end

-- Fly Feature
local flyEnabled = false
local flySpeed = 50
local flyBodyVelocity

local flyToggle = createToggle(MovementTab, "Fly", function(state)
    flyEnabled = state
    
    if state then
        if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            flyToggle.Set(false)
            return
        end
        
        flyBodyVelocity = Instance.new("BodyVelocity")
        flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        flyBodyVelocity.MaxForce = Vector3.new(0, 0, 0)
        flyBodyVelocity.Parent = LocalPlayer.Character.HumanoidRootPart
        
        LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Physics)
    else
        if flyBodyVelocity then
            flyBodyVelocity:Destroy()
            flyBodyVelocity = nil
        end
        
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
        end
    end
end)

local flySpeedSlider = createSlider(MovementTab, "Fly Speed", 20, 150, 50, function(value)
    flySpeed = value
end)

-- Speed Feature
local speedEnabled = false
local speedValue = 50

local speedToggle = createToggle(MovementTab, "Speed Hack", function(state)
    speedEnabled = state
    
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = state and speedValue or 16
    end
end)

local speedSlider = createSlider(MovementTab, "WalkSpeed", 16, 100, 50, function(value)
    speedValue = value
    if speedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = value
    end
end)

-- Infinite Jump (One-time activation)
local infiniteJumpEnabled = false
local infiniteJumpConnection

local infiniteJumpButton = Instance.new("TextButton")
infiniteJumpButton.Name = "InfiniteJumpButton"
infiniteJumpButton.Size = UDim2.new(1, -20, 0, 40)
infiniteJumpButton.Position = UDim2.new(0, 10, 0, 10 + (65 * (#MovementTab:GetChildren() - 1)))
infiniteJumpButton.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
infiniteJumpButton.BackgroundTransparency = 0.7
infiniteJumpButton.BorderSizePixel = 0
infiniteJumpButton.Text = "TOGGLE INFINITE JUMP"
infiniteJumpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
infiniteJumpButton.Font = Enum.Font.GothamBold
infiniteJumpButton.TextSize = 14
infiniteJumpButton.Parent = MovementTab

UICorner:Clone().Parent = infiniteJumpButton

infiniteJumpButton.MouseButton1Click:Connect(function()
    infiniteJumpEnabled = not infiniteJumpEnabled
    
    if infiniteJumpEnabled then
        infiniteJumpButton.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
        infiniteJumpButton.Text = "INFINITE JUMP: ON"
        
        if infiniteJumpConnection then
            infiniteJumpConnection:Disconnect()
        end
        
        infiniteJumpConnection = UIS.JumpRequest:Connect(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    else
        infiniteJumpButton.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
        infiniteJumpButton.Text = "TOGGLE INFINITE JUMP"
        
        if infiniteJumpConnection then
            infiniteJumpConnection:Disconnect()
            infiniteJumpConnection = nil
        end
    end
end)

-- ESP Feature
local espEnabled = false
local espHighlights = {}

local function updateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if espEnabled then
                if player.Character then
                    if not espHighlights[player] then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = "RLKESP"
                        highlight.FillColor = Color3.fromRGB(255, 50, 50)
                        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                        highlight.FillTransparency = 0.5
                        highlight.OutlineTransparency = 0
                        highlight.Parent = player.Character
                        espHighlights[player] = highlight
                    end
                end
            else
                if espHighlights[player] then
                    espHighlights[player]:Destroy()
                    espHighlights[player] = nil
                end
            end
        end
    end
end

local espToggle = createToggle(VisualTab, "ESP", function(state)
    espEnabled = state
    updateESP()
end)

-- Character Added Events
local function onCharacterAdded(character)
    if flyEnabled then
        flyToggle.Set(true)
    end
    
    if speedEnabled then
        character:WaitForChild("Humanoid").WalkSpeed = speedValue
    end
    
    if infiniteJumpEnabled then
        infiniteJumpConnection = UIS.JumpRequest:Connect(function()
            character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end)
    end
    
    character:WaitForChild("Humanoid").Died:Connect(function()
        if infiniteJumpConnection then
            infiniteJumpConnection:Disconnect()
            infiniteJumpConnection = nil
        end
    end)
end

LocalPlayer.CharacterAdded:Connect(onCharacterAdded)

-- Player Added Events
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if espEnabled then
            local highlight = Instance.new("Highlight")
            highlight.Name = "RLKESP"
            highlight.FillColor = Color3.fromRGB(255, 50, 50)
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.FillTransparency = 0.5
            highlight.OutlineTransparency = 0
            highlight.Parent = character
            espHighlights[player] = highlight
        end
    end)
end)

-- Initial ESP Setup
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer and player.Character then
        if espEnabled then
            local highlight = Instance.new("Highlight")
            highlight.Name = "RLKESP"
            highlight.FillColor = Color3.fromRGB(255, 50, 50)
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.FillTransparency = 0.5
            highlight.OutlineTransparency = 0
            highlight.Parent = player.Character
            espHighlights[player] = highlight
        end
    end
end

-- Fly Movement Handler
RunService.Heartbeat:Connect(function()
    if flyEnabled and flyBodyVelocity and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
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
            velocity = velocity.Unit * flySpeed
        end
        
        flyBodyVelocity.Velocity = velocity
        flyBodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
    end
end)

-- Initialization
if LocalPlayer.Character then
    onCharacterAdded(LocalPlayer.Character)
end

-- Auto-update canvas size for tabs
for _, tab in ipairs(Tabs:GetChildren()) do
    if tab:IsA("ScrollingFrame") then
        tab:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
            tab.CanvasSize = UDim2.new(0, 0, 0, tab.UIListLayout.AbsoluteContentSize.Y + 20)
        end)
    end
end
