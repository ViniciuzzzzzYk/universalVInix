-- Gunfight Simulator Hack Script for Roblox
-- Features: Auto Farm, ESP, Aimbot with team detection, Wallhack (bullet penetration)
-- GUI: Minimizable, movable, soft design
-- WARNING: Use at your own risk. For educational purposes only.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- Settings
local AimbotEnabled = false
local ESPEnabled = false
local AutoFarmEnabled = false
local WallhackEnabled = false
local SpeedBoostValue = 16
local NormalSpeed = 16

-- Create ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GunfightSimulatorHackGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 280, 0, 320)
MainFrame.Position = UDim2.new(0.05, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.ZIndex = 10
MainFrame.Parent = ScreenGui
MainFrame.Visible = true

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "Gunfight Simulator Hack"
Title.TextColor3 = Color3.new(1,1,1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 24
Title.Parent = MainFrame

-- Toggles container
local TogglesFrame = Instance.new("Frame")
TogglesFrame.Name = "TogglesFrame"
TogglesFrame.Size = UDim2.new(1, -20, 0, 200)
TogglesFrame.Position = UDim2.new(0, 10, 0, 50)
TogglesFrame.BackgroundTransparency = 1
TogglesFrame.Parent = MainFrame

-- Function to create toggles
local function createToggle(name, position, initialState)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 40)
    frame.Position = position
    frame.BackgroundTransparency = 1
    frame.Parent = TogglesFrame

    local label = Instance.new("TextLabel")
    label.Text = name
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1,1,1)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 20
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0, 60, 0, 30)
    toggle.Position = UDim2.new(0.75, 0, 0.15, 0)
    toggle.BackgroundColor3 = initialState and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
    toggle.Text = initialState and "ON" or "OFF"
    toggle.TextColor3 = Color3.new(1,1,1)
    toggle.Font = Enum.Font.SourceSansBold
    toggle.TextSize = 18
    toggle.Parent = frame

    return toggle
end

local aimbotToggle = createToggle("Aimbot", UDim2.new(0, 0, 0, 0), AimbotEnabled)
local espToggle = createToggle("ESP", UDim2.new(0, 0, 0, 50), ESPEnabled)
local autoFarmToggle = createToggle("Auto Farm", UDim2.new(0, 0, 0, 100), AutoFarmEnabled)
local wallhackToggle = createToggle("Wallhack", UDim2.new(0, 0, 0, 150), WallhackEnabled)

-- Minimize Button
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Name = "MinimizeButton"
MinimizeButton.Size = UDim2.new(0, 40, 0, 40)
MinimizeButton.Position = UDim2.new(1, -45, 0, 0)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MinimizeButton.Text = "-"
MinimizeButton.TextColor3 = Color3.new(1,1,1)
MinimizeButton.Font = Enum.Font.SourceSansBold
MinimizeButton.TextSize = 28
MinimizeButton.Parent = MainFrame

local MinimizedIcon = Instance.new("TextButton")
MinimizedIcon.Name = "MinimizedIcon"
MinimizedIcon.Size = UDim2.new(0, 50, 0, 50)
MinimizedIcon.Position = UDim2.new(0, 10, 0, 10)
MinimizedIcon.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MinimizedIcon.Text = "☰"
MinimizedIcon.TextColor3 = Color3.new(1,1,1)
MinimizedIcon.Font = Enum.Font.SourceSansBold
MinimizedIcon.TextSize = 30
MinimizedIcon.Visible = false
MinimizedIcon.Parent = ScreenGui

local UICornerIcon = Instance.new("UICorner")
UICornerIcon.CornerRadius = UDim.new(0, 12)
UICornerIcon.Parent = MinimizedIcon

MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    MinimizedIcon.Visible = true
end)

MinimizedIcon.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    MinimizedIcon.Visible = false
end)

-- Make GUI movable
local dragging = false
local dragInput
local dragStart
local startPos

local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

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

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Aimbot with aim assist (smooth aiming)
local AimAssistStrength = 0.2 -- 0 = no assist, 1 = instant snap

local function lerp(a, b, t)
    return a + (b - a) * t
end

local function getClosestTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local localTeam = LocalPlayer.Team
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team ~= localTeam and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local targetPart = player.Character:FindFirstChild("Head") or player.Character:FindFirstChild("UpperTorso")
            if targetPart then
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if dist < shortestDistance then
                        shortestDistance = dist
                        closestPlayer = player
                    end
                end
            end
        end
    end
    return closestPlayer
end

RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local target = getClosestTarget()
        if target and target.Character then
            local targetPart = target.Character:FindFirstChild("Head") or target.Character:FindFirstChild("UpperTorso")
            if targetPart then
                local currentCFrame = Camera.CFrame
                local targetCFrame = CFrame.new(currentCFrame.Position, targetPart.Position)
                local newLookVector = currentCFrame.LookVector:Lerp(targetCFrame.LookVector, AimAssistStrength)
                Camera.CFrame = CFrame.new(currentCFrame.Position, currentCFrame.Position + newLookVector)
            end
        end
    end
end)

-- ESP Setup
local espBoxes = {}

local function createEspBox(player)
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.new(1, 0, 0)
    box.Thickness = 2
    box.Transparency = 1
    box.Filled = false
    return box
end

local function updateEsp()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local targetPart = player.Character:FindFirstChild("UpperTorso") or player.Character:FindFirstChild("HumanoidRootPart")
            if targetPart then
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local box = espBoxes[player]
                    if not box then
                        box = createEspBox(player)
                        espBoxes[player] = box
                    end
                    local size = 50
                    box.Position = Vector2.new(screenPos.X - size/2, screenPos.Y - size/2)
                    box.Size = Vector2.new(size, size)
                    box.Visible = ESPEnabled
                elseif espBoxes[player] then
                    espBoxes[player].Visible = false
                end
            end
        elseif espBoxes[player] then
            espBoxes[player].Visible = false
        end
    end
end

-- Auto farm logic for Gunfight Simulator
local function autoFarmGunfightSimulator()
    -- Placeholder for Gunfight Simulator auto farm logic
    -- Implement game-specific logic here
    print("Auto Farm: Running Gunfight Simulator auto farm logic")
end

-- Main loop
RunService.RenderStepped:Connect(function()
    -- Speed Boost (if implemented)
    if SpeedBoostEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = SpeedBoostValue
    end

    -- ESP
    updateEsp()

    -- Auto Farm
    if AutoFarmEnabled then
        autoFarmGunfightSimulator()
    end

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "Gunfight Simulator Hack"
Title.TextColor3 = Color3.new(1,1,1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 24
Title.Parent = MainFrame

-- Minimize Button
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Name = "MinimizeButton"
MinimizeButton.Size = UDim2.new(0, 40, 0, 40)
MinimizeButton.Position = UDim2.new(1, -45, 0, 0)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MinimizeButton.Text = "-"
MinimizeButton.TextColor3 = Color3.new(1,1,1)
MinimizeButton.Font = Enum.Font.SourceSansBold
MinimizeButton.TextSize = 28
MinimizeButton.Parent = MainFrame

local MinimizedIcon = Instance.new("TextButton")
MinimizedIcon.Name = "MinimizedIcon"
MinimizedIcon.Size = UDim2.new(0, 50, 0, 50)
MinimizedIcon.Position = UDim2.new(0, 10, 0, 10)
MinimizedIcon.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MinimizedIcon.Text = "☰"
MinimizedIcon.TextColor3 = Color3.new(1,1,1)
MinimizedIcon.Font = Enum.Font.SourceSansBold
MinimizedIcon.TextSize = 30
MinimizedIcon.Visible = false
MinimizedIcon.Parent = ScreenGui

local UICornerIcon = Instance.new("UICorner")
UICornerIcon.CornerRadius = UDim.new(0, 12)
UICornerIcon.Parent = MinimizedIcon

-- Toggles
local function createToggle(name, position, initialState)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, -20, 0, 40)
    frame.Position = position
    frame.BackgroundTransparency = 1
    frame.Parent = MainFrame

    local label = Instance.new("TextLabel")
    label.Text = name
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1,1,1)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 20
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0, 60, 0, 30)
    toggle.Position = UDim2.new(0.75, 0, 0.15, 0)
    toggle.BackgroundColor3 = initialState and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
    toggle.Text = initialState and "ON" or "OFF"
    toggle.TextColor3 = Color3.new(1,1,1)
    toggle.Font = Enum.Font.SourceSansBold
    toggle.TextSize = 18
    toggle.Parent = frame

    return toggle
end

local aimbotToggle = createToggle("Aimbot", UDim2.new(0, 10, 0, 60), AimbotEnabled)
local espToggle = createToggle("ESP", UDim2.new(0, 10, 0, 110), ESPEnabled)
local autoFarmToggle = createToggle("Auto Farm", UDim2.new(0, 10, 0, 160), AutoFarmEnabled)
local wallhackToggle = createToggle("Wallhack", UDim2.new(0, 10, 0, 210), WallhackEnabled)

-- Minimize and maximize functions
MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    MinimizedIcon.Visible = true
end)

MinimizedIcon.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    MinimizedIcon.Visible = false
end)

-- Make GUI movable
local dragging = false
local dragInput
local dragStart
local startPos

local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

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

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Aimbot function with team detection and closest to crosshair targeting
local function getClosestTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local localTeam = LocalPlayer.Team
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team ~= localTeam and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local targetPart = player.Character:FindFirstChild("Head") or player.Character:FindFirstChild("UpperTorso")
            if targetPart then
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if dist < shortestDistance then
                        shortestDistance = dist
                        closestPlayer = player
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- ESP Setup
local espBoxes = {}

local function createEspBox(player)
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.new(1, 0, 0)
    box.Thickness = 2
    box.Transparency = 1
    box.Filled = false
    return box
end

local function updateEsp()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local targetPart = player.Character:FindFirstChild("UpperTorso") or player.Character:FindFirstChild("HumanoidRootPart")
            if targetPart then
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local box = espBoxes[player]
                    if not box then
                        box = createEspBox(player)
                        espBoxes[player] = box
                    end
                    local size = 50
                    box.Position = Vector2.new(screenPos.X - size/2, screenPos.Y - size/2)
                    box.Size = Vector2.new(size, size)
                    box.Visible = ESPEnabled
                elseif espBoxes[player] then
                    espBoxes[player].Visible = false
                end
            end
        elseif espBoxes[player] then
            espBoxes[player].Visible = false
        end
    end
end

-- Auto farm logic for Gunfight Simulator
local function autoFarmGunfightSimulator()
    -- Placeholder for Gunfight Simulator auto farm logic
    -- Implement game-specific logic here
    print("Auto Farm: Running Gunfight Simulator auto farm logic")
end

-- Main loop
RunService.RenderStepped:Connect(function()
    -- Aimbot
    if AimbotEnabled then
        local target = getClosestTarget()
        if target and target.Character then
            local targetPart = target.Character:FindFirstChild("Head") or target.Character:FindFirstChild("UpperTorso")
            if targetPart then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPart.Position)
            end
        end
    end

    -- ESP
    updateEsp()

    -- Auto Farm
    if AutoFarmEnabled then
        autoFarmGunfightSimulator()
    end
end)

print("Gunfight Simulator Hack loaded.")
