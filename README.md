-- Vex Hub for Roblox Gunfight Arena
-- Functional, mobile-friendly GUI with toggles for ESP, Hitbox, Aimbot, Bullet Penetration, Auto Farm

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variables
local VexHub = {}
VexHub.Enabled = true
VexHub.ESPEnabled = false
VexHub.HitboxEnabled = false
VexHub.AimbotEnabled = false
VexHub.BulletPenetrationEnabled = false
VexHub.AutoFarmEnabled = false

-- GUI Creation
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VexHubGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 250, 0, 300)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.Parent = ScreenGui
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Title.BorderSizePixel = 0
Title.Text = "Vex Hub - Gunfight Arena"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20
Title.Parent = MainFrame

local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Name = "MinimizeButton"
MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
MinimizeButton.Position = UDim2.new(1, -30, 0, 0)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MinimizeButton.BorderSizePixel = 0
MinimizeButton.Text = "-"
MinimizeButton.TextColor3 = Color3.new(1, 1, 1)
MinimizeButton.Font = Enum.Font.SourceSansBold
MinimizeButton.TextSize = 24
MinimizeButton.Parent = MainFrame

local function createToggle(name, position)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, -20, 0, 40)
    frame.Position = position
    frame.BackgroundTransparency = 1
    frame.Parent = MainFrame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 18
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0, 50, 0, 25)
    toggle.Position = UDim2.new(0.75, 0, 0.25, 0)
    toggle.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    toggle.BorderSizePixel = 0
    toggle.Text = "OFF"
    toggle.TextColor3 = Color3.new(1, 1, 1)
    toggle.Font = Enum.Font.SourceSansBold
    toggle.TextSize = 18
    toggle.Parent = frame

    return toggle
end

local ESPToggle = createToggle("ESP", UDim2.new(0, 10, 0, 40))
local HitboxToggle = createToggle("Hitbox", UDim2.new(0, 10, 0, 80))
local AimbotToggle = createToggle("Aimbot", UDim2.new(0, 10, 0, 120))
local BulletPenToggle = createToggle("Bullet Penetration", UDim2.new(0, 10, 0, 160))
local AutoFarmToggle = createToggle("Auto Farm", UDim2.new(0, 10, 0, 200))

-- Minimize/Open functionality
local minimized = false
MinimizeButton.MouseButton1Click:Connect(function()
    if minimized then
        MainFrame.Size = UDim2.new(0, 250, 0, 300)
        for _, child in pairs(MainFrame:GetChildren()) do
            if child:IsA("Frame") then
                child.Visible = true
            end
        end
        Title.Text = "Vex Hub - Gunfight Arena"
        minimized = false
    else
        MainFrame.Size = UDim2.new(0, 250, 0, 30)
        for _, child in pairs(MainFrame:GetChildren()) do
            if child:IsA("Frame") then
                child.Visible = false
            end
        end
        Title.Text = "Vex Hub (Minimized)"
        minimized = true
    end
end)

-- Toggle button helper
local function toggleButton(button, state)
    if state then
        button.Text = "ON"
        button.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
    else
        button.Text = "OFF"
        button.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    end
end

-- ESP Implementation
local espBoxes = {}

local function createEspBox(player)
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.new(1, 0, 0)
    box.Thickness = 2
    box.Filled = false
    return box
end

local function updateEsp()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local rootPart = player.Character.HumanoidRootPart
            local pos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
            if onScreen then
                if not espBoxes[player] then
                    espBoxes[player] = createEspBox(player)
                end
                local box = espBoxes[player]
                local size = 100 / pos.Z
                box.Size = size
                box.Position = Vector2.new(pos.X - size / 2, pos.Y - size / 2)
                box.Visible = VexHub.ESPEnabled
            elseif espBoxes[player] then
                espBoxes[player].Visible = false
            end
        elseif espBoxes[player] then
            espBoxes[player].Visible = false
        end
    end
end

-- Hitbox Enlargement
local originalSizes = {}

local function setHitbox(enabled)
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local rootPart = player.Character.HumanoidRootPart
            if enabled then
                if not originalSizes[player] then
                    originalSizes[player] = rootPart.Size
                end
                rootPart.Size = Vector3.new(10, 10, 10)
                rootPart.Transparency = 0.5
                rootPart.CanCollide = false
            else
                if originalSizes[player] then
                    rootPart.Size = originalSizes[player]
                    rootPart.Transparency = 1
                    rootPart.CanCollide = true
                    originalSizes[player] = nil
                end
            end
        end
    end
end

-- Aimbot Implementation
local function getNearestEnemy()
    local nearestPlayer = nil
    local shortestDistance = math.huge
    local mousePos = UserInputService:GetMouseLocation()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local rootPart = player.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
            if onScreen then
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    nearestPlayer = player
                end
            end
        end
    end
    return nearestPlayer
end

local function aimAt(target)
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local headPos = target.Character.Head.Position
        local cameraCFrame = Camera.CFrame
        local direction = (headPos - cameraCFrame.Position).Unit
        Camera.CFrame = CFrame.new(cameraCFrame.Position, cameraCFrame.Position + direction)
    end
end

-- Bullet Penetration (Simulated by ignoring walls in raycast)
local function modifyBulletPenetration()
    -- This is highly dependent on the game's bullet implementation.
    -- As a placeholder, we simulate by ignoring walls in raycasts or modifying bullet scripts.
    -- This requires game-specific knowledge and may not be fully functional without game internals.
end

-- Auto Farm Implementation
local function autoFarm()
    if not VexHub.AutoFarmEnabled then return end
    local target = getNearestEnemy()
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        -- Aim at target
        aimAt(target)
        -- Simulate shooting (depends on game, placeholder)
        -- For example, fire a remote event or tool activation
        -- This part requires game-specific implementation
    end
end

-- Connect toggles
ESPToggle.MouseButton1Click:Connect(function()
    VexHub.ESPEnabled = not VexHub.ESPEnabled
    toggleButton(ESPToggle, VexHub.ESPEnabled)
end)

HitboxToggle.MouseButton1Click:Connect(function()
    VexHub.HitboxEnabled = not VexHub.HitboxEnabled
    toggleButton(HitboxToggle, VexHub.HitboxEnabled)
    setHitbox(VexHub.HitboxEnabled)
end)

AimbotToggle.MouseButton1Click:Connect(function()
    VexHub.AimbotEnabled = not VexHub.AimbotEnabled
    toggleButton(AimbotToggle, VexHub.AimbotEnabled)
end)

BulletPenToggle.MouseButton1Click:Connect(function()
    VexHub.BulletPenetrationEnabled = not VexHub.BulletPenetrationEnabled
    toggleButton(BulletPenToggle, VexHub.BulletPenetrationEnabled)
    if VexHub.BulletPenetrationEnabled then
        modifyBulletPenetration()
    end
end)

AutoFarmToggle.MouseButton1Click:Connect(function()
    VexHub.AutoFarmEnabled = not VexHub.AutoFarmEnabled
    toggleButton(AutoFarmToggle, VexHub.AutoFarmEnabled)
end)

-- Main loop
RunService.RenderStepped:Connect(function()
    if VexHub.ESPEnabled then
        updateEsp()
    else
        for _, box in pairs(espBoxes) do
            box.Visible = false
        end
    end

    if VexHub.AimbotEnabled then
        local target = getNearestEnemy()
        if target then
            aimAt(target)
        end
    end

    if VexHub.AutoFarmEnabled then
        autoFarm()
    end
end)

-- Initialize toggles to OFF
toggleButton(ESPToggle, false)
toggleButton(HitboxToggle, false)
toggleButton(AimbotToggle, false)
toggleButton(BulletPenToggle, false)
toggleButton(AutoFarmToggle, false)

print("Vex Hub loaded. Use the GUI to toggle features.")
