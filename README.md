-- RLK HUB (LocalScript) - Paste into StarterPlayerScripts
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
MainFrame.Size = UDim2.new(0, 350, 0, 400)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -200)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BackgroundTransparency = 0.2
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = RLKHub

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local TopBar = Instance.new("Frame")
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 30)
TopBar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "RLK HUB v1.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.Parent = TopBar

local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 0)
CloseButton.BackgroundTransparency = 1
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 14
CloseButton.Parent = TopBar

local TabButtons = Instance.new("Frame")
TabButtons.Name = "TabButtons"
TabButtons.Size = UDim2.new(0, 100, 1, -30)
TabButtons.Position = UDim2.new(0, 0, 0, 30)
TabButtons.BackgroundTransparency = 1
TabButtons.Parent = MainFrame

local Tabs = Instance.new("Frame")
Tabs.Name = "Tabs"
Tabs.Size = UDim2.new(1, -100, 1, -30)
Tabs.Position = UDim2.new(0, 100, 0, 30)
Tabs.BackgroundTransparency = 1
Tabs.Parent = MainFrame

-- Create Tabs
local function createTab(name)
    local button = Instance.new("TextButton")
    button.Name = name.."Tab"
    button.Size = UDim2.new(1, 0, 0, 40)
    button.Position = UDim2.new(0, 0, 0, (40 * (#TabButtons:GetChildren() - 1)))
    button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    button.BackgroundTransparency = 0.5
    button.BorderSizePixel = 0
    button.Text = name
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 12
    button.Parent = TabButtons
    
    local tab = Instance.new("ScrollingFrame")
    tab.Name = name
    tab.Size = UDim2.new(1, 0, 1, 0)
    tab.Position = UDim2.new(0, 0, 0, 0)
    tab.BackgroundTransparency = 1
    tab.Visible = false
    tab.ScrollBarThickness = 3
    tab.AutomaticCanvasSize = Enum.AutomaticSize.Y
    tab.Parent = Tabs
    
    UICorner:Clone().Parent = button
    
    return tab
end

-- Movement Tab
local MovementTab = createTab("Movement")
local VisualTab = createTab("Visual")

-- Make first tab visible
MovementTab.Visible = true

-- Tab Switching
for _, button in ipairs(TabButtons:GetChildren()) do
    if button:IsA("TextButton") then
        button.MouseButton1Click:Connect(function()
            for _, tab in ipairs(Tabs:GetChildren()) do
                if tab:IsA("ScrollingFrame") then
                    tab.Visible = false
                end
            end
            Tabs[button.Name:gsub("Tab", "")].Visible = true
            
            -- Animate button click
            button.BackgroundTransparency = 0.3
            task.wait(0.1)
            button.BackgroundTransparency = 0.5
        end)
    end
end

-- Toggle GUI Visibility
local guiVisible = true
CloseButton.MouseButton1Click:Connect(function()
    guiVisible = not guiVisible
    MainFrame.Visible = guiVisible
end)

UIS.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.RightShift and not gameProcessed then
        guiVisible = not guiVisible
        MainFrame.Visible = guiVisible
    end
end)

-- Modern Toggle Function
local function createToggle(parent, name, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Name = name.."Toggle"
    toggleFrame.Size = UDim2.new(1, -20, 0, 30)
    toggleFrame.Position = UDim2.new(0, 10, 0, 10 + (35 * (#parent:GetChildren() - 1)))
    toggleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    toggleFrame.BackgroundTransparency = 0.5
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = parent
    
    UICorner:Clone().Parent = toggleFrame
    
    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Name = "Label"
    toggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    toggleLabel.Position = UDim2.new(0, 10, 0, 0)
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.Text = name
    toggleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    toggleLabel.Font = Enum.Font.Gotham
    toggleLabel.TextSize = 12
    toggleLabel.Parent = toggleFrame
    
    local toggleButton = Instance.new("TextButton")
    toggleButton.Name = "ToggleButton"
    toggleButton.Size = UDim2.new(0, 50, 0, 20)
    toggleButton.Position = UDim2.new(1, -60, 0.5, -10)
    toggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    toggleButton.BorderSizePixel = 0
    toggleButton.Text = ""
    toggleButton.Parent = toggleFrame
    
    local toggleCircle = Instance.new("Frame")
    toggleCircle.Name = "Circle"
    toggleCircle.Size = UDim2.new(0, 16, 0, 16)
    toggleCircle.Position = UDim2.new(0, 2, 0.5, -8)
    toggleCircle.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
    toggleCircle.BorderSizePixel = 0
    toggleCircle.Parent = toggleButton
    
    UICorner:Clone().Parent = toggleButton
    UICorner:Clone().Parent = toggleCircle
    
    local state = false
    
    local function updateToggle()
        if state then
            TweenService:Create(toggleCircle, TweenInfo.new(0.2), {Position = UDim2.new(1, -18, 0.5, -8)}):Play()
            TweenService:Create(toggleButton, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0, 170, 255)}):Play()
        else
            TweenService:Create(toggleCircle, TweenInfo.new(0.2), {Position = UDim2.new(0, 2, 0.5, -8)}):Play()
            TweenService:Create(toggleButton, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 60)}):Play()
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
        end
    }
end

-- Fly Feature
local flyToggle = createToggle(MovementTab, "Fly", function(state)
    if state then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(0, 0, 0)
        bodyVelocity.Parent = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        
        LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Physics)
        
        local flyConnection
        flyConnection = RunService.Heartbeat:Connect(function()
            if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") or not bodyVelocity then
                flyConnection:Disconnect()
                return
            end
            
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
                velocity = velocity.Unit * 50
            end
            
            bodyVelocity.Velocity = velocity
            bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
        end)
    else
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            for _, v in ipairs(LocalPlayer.Character.HumanoidRootPart:GetChildren()) do
                if v:IsA("BodyVelocity") then
                    v:Destroy()
                end
            end
            LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
        end
    end
end)

-- Speed Feature
local speedToggle = createToggle(MovementTab, "Speed Hack", function(state)
    if state then
        LocalPlayer.Character.Humanoid.WalkSpeed = 50
    else
        LocalPlayer.Character.Humanoid.WalkSpeed = 16
    end
end)

-- Infinite Jump
local infiniteJumpToggle = createToggle(MovementTab, "Infinite Jump", function(state)
    if state then
        LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        LocalPlayer.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
    else
        LocalPlayer.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
    end
end)

-- ESP Feature
local espToggle = createToggle(VisualTab, "ESP", function(state)
    if state then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local highlight = Instance.new("Highlight")
                highlight.Name = "RLKESP"
                highlight.FillColor = Color3.fromRGB(255, 0, 0)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.FillTransparency = 0.5
                highlight.Parent = player.Character
            end
        end
        
        Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function(character)
                if espToggle then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "RLKESP"
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.FillTransparency = 0.5
                    highlight.Parent = character
                end
            end)
        end)
    else
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                for _, v in ipairs(player.Character:GetChildren()) do
                    if v.Name == "RLKESP" then
                        v:Destroy()
                    end
                end
            end
        end
    end
end)

-- Character Added Events
LocalPlayer.CharacterAdded:Connect(function(character)
    if speedToggle then
        character:WaitForChild("Humanoid").WalkSpeed = speedToggle and 50 or 16
    end
    
    if infiniteJumpToggle then
        character:WaitForChild("Humanoid"):SetStateEnabled(Enum.HumanoidStateType.Jumping, not infiniteJumpToggle)
    end
end)

-- Initialization
task.spawn(function()
    repeat task.wait() until LocalPlayer.Character
    if speedToggle then speedToggle.Set(false) end
    if infiniteJumpToggle then infiniteJumpToggle.Set(false) end
    if flyToggle then flyToggle.Set(false) end
    if espToggle then espToggle.Set(false) end
end)
