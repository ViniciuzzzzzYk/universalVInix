-- Roblox FPS Hack Menu with GUI, Aimbot, ESP, Speed Boost, and Minimizable Menu
-- WARNING: Use at your own risk. This script is for educational purposes only.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- Settings
local AimbotEnabled = false
local ESPEnabled = false
local SpeedBoostEnabled = false
local WallhackEnabled = false -- new toggle for bullet penetration
local AutoFarmEnabled = false -- new toggle for auto farm
local SpeedBoostValue = 50 -- default walk speed when speed boost is enabled
local NormalSpeed = 16 -- default Roblox walk speed

-- Create ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "HackMenuGui"
ScreenGui.ResetOnSpawn = false

local CoreGui = game:GetService("CoreGui")
local PlayerGui = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")

-- Try to parent to CoreGui, fallback to PlayerGui if fails
local success, err = pcall(function()
    ScreenGui.Parent = CoreGui
end)
if not success then
    ScreenGui.Parent = PlayerGui
end

print("HackMenuGui parented to: ", ScreenGui.Parent.Name)

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 250, 0, 320) -- smaller size
MainFrame.Position = UDim2.new(0.05, 0, 0.15, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.ZIndex = 10
MainFrame.Parent = ScreenGui
MainFrame.Visible = true

-- Aimbot toggle button on screen corner
local AimbotToggleButton = Instance.new("TextButton")
AimbotToggleButton.Name = "AimbotToggleButton"
AimbotToggleButton.Size = UDim2.new(0, 40, 0, 40)
AimbotToggleButton.Position = UDim2.new(1, -50, 0.5, -20)
AimbotToggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
AimbotToggleButton.TextColor3 = Color3.new(1,1,1)
AimbotToggleButton.Font = Enum.Font.SourceSansBold
AimbotToggleButton.TextSize = 18
AimbotToggleButton.Text = "Aim ON"
AimbotToggleButton.Parent = ScreenGui
AimbotToggleButton.ZIndex = 10

AimbotToggleButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    if AimbotEnabled then
        AimbotToggleButton.Text = "Aim ON"
        aimbotToggle.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
        aimbotToggle.Text = "ON"
    else
        AimbotToggleButton.Text = "Aim OFF"
        aimbotToggle.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
        aimbotToggle.Text = "OFF"
    end
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

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "Universal Roblox Hack Menu"
Title.TextColor3 = Color3.new(1,1,1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 26
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

-- Game Selection Dropdown
local GameSelectionLabel = Instance.new("TextLabel")
GameSelectionLabel.Text = "Select Game for Auto Farm"
GameSelectionLabel.Size = UDim2.new(1, -20, 0, 30)
GameSelectionLabel.Position = UDim2.new(0, 10, 0, 320)
GameSelectionLabel.BackgroundTransparency = 1
GameSelectionLabel.TextColor3 = Color3.new(1,1,1)
GameSelectionLabel.Font = Enum.Font.SourceSans
GameSelectionLabel.TextSize = 18
GameSelectionLabel.TextXAlignment = Enum.TextXAlignment.Left
GameSelectionLabel.Parent = MainFrame

local GameSelectionDropdown = Instance.new("TextButton")
GameSelectionDropdown.Name = "GameSelectionDropdown"
GameSelectionDropdown.Size = UDim2.new(1, -20, 0, 30)
GameSelectionDropdown.Position = UDim2.new(0, 10, 0, 350)
GameSelectionDropdown.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
GameSelectionDropdown.TextColor3 = Color3.new(1,1,1)
GameSelectionDropdown.Font = Enum.Font.SourceSans
GameSelectionDropdown.TextSize = 18
GameSelectionDropdown.Text = "Unnamed Shooter"
GameSelectionDropdown.Parent = MainFrame

local DropdownOpen = false
local DropdownList = Instance.new("Frame")
DropdownList.Size = UDim2.new(1, -20, 0, 100)
DropdownList.Position = UDim2.new(0, 10, 0, 380)
DropdownList.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
DropdownList.Visible = false
DropdownList.Parent = MainFrame

local UICornerDropdown = Instance.new("UICorner")
UICornerDropdown.CornerRadius = UDim.new(0, 8)
UICornerDropdown.Parent = DropdownList

local games = {"Unnamed Shooter", "Build A Boat For Treasure", "Other Game"}

local function closeDropdown()
    DropdownList.Visible = false
    DropdownOpen = false
end

local function openDropdown()
    DropdownList.Visible = true
    DropdownOpen = true
end

GameSelectionDropdown.MouseButton1Click:Connect(function()
    if DropdownOpen then
        closeDropdown()
    else
        openDropdown()
    end
end)

for i, gameName in ipairs(games) do
    local option = Instance.new("TextButton")
    option.Size = UDim2.new(1, 0, 0, 30)
    option.Position = UDim2.new(0, 0, 0, (i-1)*30)
    option.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    option.TextColor3 = Color3.new(1,1,1)
    option.Font = Enum.Font.SourceSans
    option.TextSize = 18
    option.Text = gameName
    option.Parent = DropdownList

    option.MouseButton1Click:Connect(function()
        GameSelectionDropdown.Text = gameName
        closeDropdown()
    end)
end

-- UI Corner for rounded edges
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "Roblox FPS Hack Menu"
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
MinimizedIcon.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MinimizedIcon.Text = "☰"
MinimizedIcon.TextColor3 = Color3.new(1,1,1)
MinimizedIcon.Font = Enum.Font.SourceSansBold
MinimizedIcon.TextSize = 30
MinimizedIcon.Visible = false
MinimizedIcon.Parent = ScreenGui

local UICornerIcon = Instance.new("UICorner")
UICornerIcon.CornerRadius = UDim.new(0, 10)
UICornerIcon.Parent = MinimizedIcon

-- Toggles and Buttons
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
local speedToggle = createToggle("Speed Boost", UDim2.new(0, 10, 0, 160), SpeedBoostEnabled)
local wallhackToggle = createToggle("Wallhack", UDim2.new(0, 10, 0, 210), WallhackEnabled)
local autoFarmToggle = createToggle("Auto Farm", UDim2.new(0, 10, 0, 260), AutoFarmEnabled)

-- Speed Slider
local speedLabel = Instance.new("TextLabel")
speedLabel.Text = "Speed Value"
speedLabel.Size = UDim2.new(0, 100, 0, 30)
speedLabel.Position = UDim2.new(0, 10, 0, 310)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.Font = Enum.Font.SourceSans
speedLabel.TextSize = 18
speedLabel.Parent = MainFrame

local speedValueLabel = Instance.new("TextLabel")
speedValueLabel.Text = tostring(SpeedBoostValue)
speedValueLabel.Size = UDim2.new(0, 40, 0, 30)
speedValueLabel.Position = UDim2.new(0, 120, 0, 310)
speedValueLabel.BackgroundTransparency = 1
speedValueLabel.TextColor3 = Color3.new(1,1,1)
speedValueLabel.Font = Enum.Font.SourceSansBold
speedValueLabel.TextSize = 18
speedValueLabel.Parent = MainFrame

local speedSlider = Instance.new("TextBox")
speedSlider.Size = UDim2.new(0, 100, 0, 30)
speedSlider.Position = UDim2.new(0, 170, 0, 310)
speedSlider.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
speedSlider.TextColor3 = Color3.new(1,1,1)
speedSlider.Font = Enum.Font.SourceSans
speedSlider.TextSize = 18
speedSlider.Text = tostring(SpeedBoostValue)
speedSlider.ClearTextOnFocus = false
speedSlider.Parent = MainFrame

-- Functions for toggles
aimbotToggle.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    aimbotToggle.BackgroundColor3 = AimbotEnabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
    aimbotToggle.Text = AimbotEnabled and "ON" or "OFF"
end)

espToggle.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    espToggle.BackgroundColor3 = ESPEnabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
    espToggle.Text = ESPEnabled and "ON" or "OFF"
end)

speedToggle.MouseButton1Click:Connect(function()
    SpeedBoostEnabled = not SpeedBoostEnabled
    speedToggle.BackgroundColor3 = SpeedBoostEnabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
    speedToggle.Text = SpeedBoostEnabled and "ON" or "OFF"
    if not SpeedBoostEnabled then
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = NormalSpeed
        end
    end
end)

wallhackToggle.MouseButton1Click:Connect(function()
    WallhackEnabled = not WallhackEnabled
    wallhackToggle.BackgroundColor3 = WallhackEnabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
    wallhackToggle.Text = WallhackEnabled and "ON" or "OFF"
end)

autoFarmToggle.MouseButton1Click:Connect(function()
    AutoFarmEnabled = not AutoFarmEnabled
    autoFarmToggle.BackgroundColor3 = AutoFarmEnabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
    autoFarmToggle.Text = AutoFarmEnabled and "ON" or "OFF"
end)

speedSlider.FocusLost:Connect(function(enterPressed)
    local val = tonumber(speedSlider.Text)
    if val and val >= 16 and val <= 200 then
        SpeedBoostValue = val
        speedValueLabel.Text = tostring(SpeedBoostValue)
        if SpeedBoostEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = SpeedBoostValue
        end
    else
        speedSlider.Text = tostring(SpeedBoostValue)
    end
end)

-- Minimize and maximize functions
MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    MinimizedIcon.Visible = true
end)

MinimizedIcon.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    MinimizedIcon.Visible = false
end)

-- Aimbot function with team check and closest to crosshair targeting
local function getClosestTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local localTeam = nil
    if LocalPlayer.Team then
        localTeam = LocalPlayer.Team
    end
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
=======
-- Auto farm logic for Grow a Garden
local function autoFarmGrowAGarden()
    -- Example implementation for Grow a Garden auto farm
    print("Auto Farm: Running Grow a Garden auto farm logic")

    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer

    -- Example remote events (replace with actual game remotes)
    local PlantSeedEvent = ReplicatedStorage:FindFirstChild("PlantSeed")
    local WaterPlantEvent = ReplicatedStorage:FindFirstChild("WaterPlant")
    local HarvestPlantEvent = ReplicatedStorage:FindFirstChild("HarvestPlant")
    local SellCropsEvent = ReplicatedStorage:FindFirstChild("SellCrops")

    -- Plant seeds logic
    if PlantSeedEvent then
        -- Logic to plant best seeds at available spots
        -- This is a placeholder, actual implementation depends on game specifics
        PlantSeedEvent:FireServer("BestSeedType", Vector3.new(0,0,0))
    end

    -- Water plants logic
    if WaterPlantEvent then
        -- Use raycast to detect dry soil and water
        WaterPlantEvent:FireServer()
    end

    -- Harvest plants logic
    if HarvestPlantEvent then
        -- Harvest mature plants with delay
        HarvestPlantEvent:FireServer()
    end

    -- Sell crops logic
    if SellCropsEvent then
        SellCropsEvent:FireServer()
    end
end

-- Auto farm logic for Build A Boat For Treasure
local function autoFarmBuildABoat()
    print("Auto Farm: Running Build A Boat For Treasure auto farm logic")

    local PathfindingService = game:GetService("PathfindingService")
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local Character = LocalPlayer.Character
    local Humanoid = Character and Character:FindFirstChildOfClass("Humanoid")

    if not Character or not Humanoid then return end

    -- Example target positions (replace with actual game positions)
    local resourcePositions = {
        Vector3.new(0,0,0),
        Vector3.new(10,0,10),
        Vector3.new(20,0,20),
    }

    for _, pos in ipairs(resourcePositions) do
        local path = PathfindingService:CreatePath()
        path:ComputeAsync(Character.HumanoidRootPart.Position, pos)
        if path.Status == Enum.PathStatus.Success then
            local waypoints = path:GetWaypoints()
            for _, waypoint in ipairs(waypoints) do
                Humanoid:MoveTo(waypoint.Position)
                Humanoid.MoveToFinished:Wait()
                wait(math.random(0.2, 1.5)) -- random delay to avoid detection
            end
            -- Simulate click to collect resource
            -- This is a placeholder, actual implementation depends on game specifics
            print("Collected resource at ", pos)
        end
    end

    -- Anti-AFK: subtle movements
    if tick() % 60 < 1 then
        Humanoid.Jump = true
        wait(0.1)
        Humanoid.Jump = false
    end
end

-- Auto farm logic for Gunfight Arena
local function autoFarmGunfightArena()
    print("Auto Farm: Running Gunfight Arena auto farm logic")

    -- Placeholder for Gunfight Arena auto farm
    -- Implement game-specific logic here
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

    -- Speed Boost
    if SpeedBoostEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = SpeedBoostValue
    end

    -- ESP
    updateEsp()

    -- Auto Farm: automatically fire at closest target if enabled
    if AutoFarmEnabled then
        local selectedGame = GameSelectionDropdown.Text
        if selectedGame == "Unnamed Shooter" and AimbotEnabled then
            local target = getClosestTarget()
            if target and target.Character then
                -- Simulate firing at target
                print("Auto Farm: Firing at target " .. target.Name)
                -- TODO: Implement actual firing logic based on game specifics
            end
        elseif selectedGame == "Grow a Garden" then
            autoFarmGrowAGarden()
        elseif selectedGame == "Build A Boat For Treasure" then
            autoFarmBuildABoat()
        elseif selectedGame == "Gunfight Arena" then
            autoFarmGunfightArena()
        else
            -- Other games or no auto farm
        end
    end
end)

-- ESP Setup tailored for Unnamed Shooter game
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
            -- Use UpperTorso or HumanoidRootPart for ESP box position
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

    -- Speed Boost
    if SpeedBoostEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = SpeedBoostValue
    end

    -- ESP
    updateEsp()

    -- Auto Farm: automatically fire at closest target if enabled
    if AutoFarmEnabled and AimbotEnabled then
        local target = getClosestTarget()
        if target and target.Character then
            -- Simulate firing at target
            -- This part depends on the game's shooting implementation
            -- For Unnamed Shooter, we can try to fire a remote event or simulate mouse click
            -- Here we just print for demonstration
            print("Auto Farm: Firing at target " .. target.Name)
            -- TODO: Implement actual firing logic based on game specifics
        end
    end
end)

-- Responsive GUI scaling
local function onResize()
    local screenSize = workspace.CurrentCamera.ViewportSize
    MainFrame.Size = UDim2.new(0, math.clamp(screenSize.X * 0.3, 250, 400), 0, math.clamp(screenSize.Y * 0.5, 300, 450))
    MinimizedIcon.Position = UDim2.new(0, 10, 0, screenSize.Y - 60)
end

workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(onResize)
onResize()

-- Anti-cheat bypass note:
-- This script uses basic techniques and obfuscation is recommended for better bypass.
-- Use with caution and test in your environment.

print("Roblox FPS Hack Menu loaded.")
