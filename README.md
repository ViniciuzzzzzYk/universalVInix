-- Updated Universal GUI script with responsive size, vertical tabs, minimize feature, game detection, and enhanced aimbot

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera
local StarterGui = game:GetService("StarterGui")

-- Detect if player is in Gunfight Arena by PlaceId
local GUNFIGHT_ARENA_PLACEID = 14518422161
local inGunfightArena = (game.PlaceId == GUNFIGHT_ARENA_PLACEID)

-- Responsive GUI size based on screen size
local screenSize = workspace.CurrentCamera.ViewportSize
local guiWidth = math.clamp(screenSize.X * 0.25, 250, 350)
local guiHeight = math.clamp(screenSize.Y * 0.6, 300, 450)

-- Create ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "UniversalGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

-- Main Frame (responsive size and draggable)
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, guiWidth, 0, guiHeight)
MainFrame.Position = UDim2.new(0.5, -guiWidth/2, 0.5, -guiHeight/2)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BackgroundTransparency = 0.1
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui
MainFrame.Active = true
MainFrame.Draggable = true

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 15)
UICorner.Parent = MainFrame

-- Title Label
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "VEX Universal"
Title.TextColor3 = Color3.fromRGB(0, 255, 0)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 26
Title.Parent = MainFrame

-- Vertical Tab buttons container on left side
local TabContainer = Instance.new("Frame")
TabContainer.Size = UDim2.new(0, 120, 1, -40)
TabContainer.Position = UDim2.new(0, 0, 0, 40)
TabContainer.BackgroundTransparency = 1
TabContainer.Parent = MainFrame

local function createTabButton(text, positionY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.Position = UDim2.new(0, 0, 0, positionY)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.fromRGB(200, 200, 200)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 18
    btn.Text = text
    btn.AutoButtonColor = true
    btn.Parent = TabContainer

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = btn

    -- Hover effect
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 60)}):Play()
    end)
    btn.MouseLeave:Connect(function()
        if btn.BackgroundColor3 ~= Color3.fromRGB(80, 80, 80) then
            TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(40, 40, 40)}):Play()
        end
    end)

    return btn
end

local UniversalTabButton = createTabButton("Universal", 0)
local GameTabButton
if inGunfightArena then
    GameTabButton = createTabButton("Gunfight Arena", 50)
end

-- Content frames for tabs
local UniversalTab = Instance.new("Frame")
UniversalTab.Size = UDim2.new(1, -120, 1, -40)
UniversalTab.Position = UDim2.new(0, 120, 0, 40)
UniversalTab.BackgroundTransparency = 1
UniversalTab.Parent = MainFrame

local GameTab
if inGunfightArena then
    GameTab = Instance.new("Frame")
    GameTab.Size = UDim2.new(1, -120, 1, -40)
    GameTab.Position = UDim2.new(0, 120, 0, 40)
    GameTab.BackgroundTransparency = 1
    GameTab.Visible = false
    GameTab.Parent = MainFrame
end

-- Minimize button (small green V icon)
local MinimizedButton = Instance.new("TextButton")
MinimizedButton.Size = UDim2.new(0, 40, 0, 40)
MinimizedButton.Position = UDim2.new(0, 10, 0, 10)
MinimizedButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
MinimizedButton.TextColor3 = Color3.fromRGB(0, 0, 0)
MinimizedButton.Font = Enum.Font.GothamBold
MinimizedButton.TextSize = 24
MinimizedButton.Text = "V"
MinimizedButton.Visible = false
MinimizedButton.Parent = ScreenGui

MinimizedButton.MouseButton1Click:Connect(function()
    MinimizedButton.Visible = false
    MainFrame.Visible = true
end)

-- Tab switching logic
local function setActiveTab(tabButton, tabFrame)
    UniversalTabButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    UniversalTabButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    if inGunfightArena and GameTabButton then
        GameTabButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        GameTabButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    end

    tabButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    tabButton.TextColor3 = Color3.fromRGB(255, 255, 255)

    UniversalTab.Visible = false
    if inGunfightArena and GameTab then
        GameTab.Visible = false
    end
    tabFrame.Visible = true
end

UniversalTabButton.MouseButton1Click:Connect(function()
    setActiveTab(UniversalTabButton, UniversalTab)
end)

if inGunfightArena and GameTabButton then
    GameTabButton.MouseButton1Click:Connect(function()
        setActiveTab(GameTabButton, GameTab)
    end)
end

setActiveTab(UniversalTabButton, UniversalTab)

-- Utility function to create buttons inside tabs
local function createButton(text, positionY, parent)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 35)
    btn.Position = UDim2.new(0, 10, 0, positionY)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 18
    btn.Text = text
    btn.AutoButtonColor = true
    btn.Parent = parent

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn

    -- Hover effect
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(70, 70, 70)}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 50, 50)}):Play()
    end)

    return btn
end

-- Variables for features
local flying = false
local flySpeed = 50
local bodyVelocity

local invisible = false
local espEnabled = false
local espBoxes = {}

local speedEnabled = false
local defaultWalkSpeed = 16
local speedValue = 50

local jumpEnabled = false
local defaultJumpPower = 50
local jumpPowerValue = 100

local autoHealEnabled = false
local healAmount = 10
local healInterval = 1

local nightModeEnabled = false
local defaultLightingSettings = {
    Brightness = Lighting.Brightness,
    Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient,
    ClockTime = Lighting.ClockTime,
}

local nightLightingSettings = {
    Brightness = 0.2,
    Ambient = Color3.fromRGB(20, 20, 20),
    OutdoorAmbient = Color3.fromRGB(10, 10, 10),
    ClockTime = 0,
}

local antiAFKEnabled = false
local VirtualUser = game:GetService("VirtualUser")

local wallClipEnabled = false

-- Helper functions
local function getCharacter()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

local function getRootPart()
    local char = getCharacter()
    return char and char:FindFirstChild("HumanoidRootPart")
end

-- Fly feature (fixed)
local function startFly()
    if flying then return end
    flying = true
    local rootPart = getRootPart()
    if not rootPart then return end

    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.Parent = rootPart

    RunService:BindToRenderStep("Fly", Enum.RenderPriority.Character.Value, function()
        local moveDirection = Vector3.new(0, 0, 0)
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end

        if moveDirection.Magnitude > 0 then
            bodyVelocity.Velocity = moveDirection.Unit * flySpeed
        else
            bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        end
    end)
end

local function stopFly()
    if not flying then return end
    flying = false
    RunService:UnbindFromRenderStep("Fly")
    if bodyVelocity then
        bodyVelocity:Destroy()
        bodyVelocity = nil
    end
end

-- Wall clipping feature
local function toggleWallClip()
    wallClipEnabled = not wallClipEnabled
    local character = getCharacter()
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    humanoid:ChangeState(wallClipEnabled and Enum.HumanoidStateType.Physics or Enum.HumanoidStateType.GettingUp)
end

-- Invisibility feature
local function toggleInvisibility()
    invisible = not invisible
    local character = getCharacter()
    if not character then return end
    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") then
            part.Transparency = invisible and 1 or 0
            if part:FindFirstChildOfClass("Decal") then
                part:FindFirstChildOfClass("Decal").Transparency = invisible and 1 or 0
            end
        elseif part:IsA("Accessory") and part:FindFirstChild("Handle") then
            part.Handle.Transparency = invisible and 1 or 0
        end
    end
end

-- ESP feature
local espBoxes = {}
local function createESPBox(player)
    local box = Instance.new("BoxHandleAdornment")
    box.Adornee = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = Vector3.new(4, 6, 1)
    box.Color3 = Color3.new(1, 0, 0)
    box.Transparency = 0.5
    box.Parent = Camera
    return box
end

local espEnabled = false
local function toggleESP()
    espEnabled = not espEnabled
    if espEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                espBoxes[player] = createESPBox(player)
            end
        end
        Players.PlayerAdded:Connect(function(player)
            if espEnabled then
                espBoxes[player] = createESPBox(player)
            end
        end)
        Players.PlayerRemoving:Connect(function(player)
            if espBoxes[player] then
                espBoxes[player]:Destroy()
                espBoxes[player] = nil
            end
        end)
    else
        for _, box in pairs(espBoxes) do
            box:Destroy()
        end
        espBoxes = {}
    end
end

-- Speed toggle
local speedEnabled = false
local defaultWalkSpeed = 16
local speedValue = 50

local function toggleSpeed()
    local character = getCharacter()
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    speedEnabled = not speedEnabled
    if speedEnabled then
        humanoid.WalkSpeed = speedValue
    else
        humanoid.WalkSpeed = defaultWalkSpeed
    end
end

-- Jump boost toggle
local jumpEnabled = false
local defaultJumpPower = 50
local jumpPowerValue = 100

local function toggleJump()
    local character = getCharacter()
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    jumpEnabled = not jumpEnabled
    if jumpEnabled then
        humanoid.JumpPower = jumpPowerValue
    else
        humanoid.JumpPower = defaultJumpPower
    end
end

-- Teleport to spawn
local function teleportToSpawn()
    local character = getCharacter()
    if not character then return end
    local rootPart = getRootPart()
    if not rootPart then return end
    local spawnLocation = workspace:FindFirstChild("SpawnLocation") or workspace:FindFirstChild("Spawn")
    if spawnLocation and spawnLocation:IsA("BasePart") then
        rootPart.CFrame = spawnLocation.CFrame + Vector3.new(0, 5, 0)
    else
        rootPart.CFrame = CFrame.new(0, 10, 0)
    end
end

-- Auto heal toggle
local autoHealEnabled = false
local healAmount = 10
local healInterval = 1

local function autoHeal()
    while autoHealEnabled do
        local character = getCharacter()
        if character then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health < humanoid.MaxHealth then
                humanoid.Health = math.min(humanoid.Health + healAmount, humanoid.MaxHealth)
            end
        end
        wait(healInterval)
    end
end

local function toggleAutoHeal()
    autoHealEnabled = not autoHealEnabled
    if autoHealEnabled then
        coroutine.wrap(autoHeal)()
    end
end

-- Night mode toggle
local nightModeEnabled = false
local defaultLightingSettings = {
    Brightness = Lighting.Brightness,
    Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient,
    ClockTime = Lighting.ClockTime,
}

local nightLightingSettings = {
    Brightness = 0.2,
    Ambient = Color3.fromRGB(20, 20, 20),
    OutdoorAmbient = Color3.fromRGB(10, 10, 10),
    ClockTime = 0,
}

local function toggleNightMode()
    nightModeEnabled = not nightModeEnabled
    if nightModeEnabled then
        Lighting.Brightness = nightLightingSettings.Brightness
        Lighting.Ambient = nightLightingSettings.Ambient
        Lighting.OutdoorAmbient = nightLightingSettings.OutdoorAmbient
        Lighting.ClockTime = nightLightingSettings.ClockTime
    else
        Lighting.Brightness = defaultLightingSettings.Brightness
        Lighting.Ambient = defaultLightingSettings.Ambient
        Lighting.OutdoorAmbient = defaultLightingSettings.OutdoorAmbient
        Lighting.ClockTime = defaultLightingSettings.ClockTime
    end
end

-- Anti-AFK feature
local antiAFKEnabled = false
local function toggleAntiAFK()
    antiAFKEnabled = not antiAFKEnabled
    if antiAFKEnabled then
        LocalPlayer.Idled:Connect(function()
            VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            wait(1)
            VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        end)
    end
end

-- Team check feature for Gunfight Arena
local teamCheckEnabled = false
local function isEnemy(player)
    if not teamCheckEnabled then
        return true
    end
    local localTeam = LocalPlayer.Team
    if not localTeam then
        return true
    end
    return player.Team ~= localTeam
end

-- Aimbot feature for Gunfight Arena with increased range and closest to crosshair targeting
local aimbotEnabled = false
local aimbotFOV = 150 -- increased degrees for wider range
local aimbotSmoothness = 0.25

local function getClosestEnemy()
    local closestPlayer = nil
    local shortestDistance = math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and isEnemy(player) then
            local screenPos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            if onScreen then
                local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                local playerPos = Vector2.new(screenPos.X, screenPos.Y)
                local distance = (playerPos - mousePos).Magnitude
                if distance < shortestDistance and distance <= aimbotFOV then
                    shortestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end
    return closestPlayer
end

RunService:BindToRenderStep("Aimbot", Enum.RenderPriority.Camera.Value + 1, function()
    if aimbotEnabled then
        local target = getClosestEnemy()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local targetPos = target.Character.HumanoidRootPart.Position
            local cameraCFrame = Camera.CFrame
            local direction = (targetPos - cameraCFrame.Position).Unit
            local newCFrame = CFrame.new(cameraCFrame.Position, cameraCFrame.Position + direction)
            Camera.CFrame = cameraCFrame:Lerp(newCFrame, aimbotSmoothness)
        end
    end
end)

-- Minimize button functionality
local function minimizeGUI()
    MainFrame.Visible = false
    MinimizedButton.Visible = true
end

local function maximizeGUI()
    MainFrame.Visible = true
    MinimizedButton.Visible = false
end

-- Add minimize button to MainFrame
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
MinimizeButton.Position = UDim2.new(1, -35, 0, 5)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
MinimizeButton.TextColor3 = Color3.fromRGB(0, 0, 0)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 20
MinimizeButton.Text = "-"
MinimizeButton.Parent = MainFrame

MinimizeButton.MouseButton1Click:Connect(minimizeGUI)

MinimizedButton.MouseButton1Click:Connect(maximizeGUI)

-- Create buttons for Universal tab
local flyButton = createButton("Toggle Fly", 10, UniversalTab)
local invisButton = createButton("Toggle Invisibility", 60, UniversalTab)
local espButton = createButton("Toggle ESP", 110, UniversalTab)
local speedButton = createButton("Toggle Speed", 160, UniversalTab)
local jumpButton = createButton("Toggle Jump Boost", 210, UniversalTab)
local teleportButton = createButton("Teleport to Spawn", 260, UniversalTab)
local autoHealButton = createButton("Toggle Auto Heal", 310, UniversalTab)
local nightModeButton = createButton("Toggle Night Mode", 360, UniversalTab)
local antiAFKButton = createButton("Toggle Anti-AFK", 410, UniversalTab)
local wallClipButton = createButton("Toggle Wall Clip", 460, UniversalTab)

-- Create buttons for Gunfight Arena tab if applicable
local teamCheckButton
local aimbotButton
if inGunfightArena and GameTab then
    teamCheckButton = createButton("Toggle Team Check", 10, GameTab)
    aimbotButton = createButton("Toggle Aimbot", 60, GameTab)
end

-- Button connections
flyButton.MouseButton1Click:Connect(function()
    if flying then
        stopFly()
        flyButton.Text = "Toggle Fly"
    else
        startFly()
        flyButton.Text = "Disable Fly"
    end
end)

invisButton.MouseButton1Click:Connect(function()
    toggleInvisibility()
    if invisible then
        invisButton.Text = "Disable Invisibility"
    else
        invisButton.Text = "Toggle Invisibility"
    end
end)

espButton.MouseButton1Click:Connect(function()
    toggleESP()
    if espEnabled then
        espButton.Text = "Disable ESP"
    else
        espButton.Text = "Toggle ESP"
    end
end)

speedButton.MouseButton1Click:Connect(function()
    toggleSpeed()
    if speedEnabled then
        speedButton.Text = "Disable Speed"
    else
        speedButton.Text = "Toggle Speed"
    end
end)

jumpButton.MouseButton1Click:Connect(function()
    toggleJump()
    if jumpEnabled then
        jumpButton.Text = "Disable Jump Boost"
    else
        jumpButton.Text = "Toggle Jump Boost"
    end
end)

teleportButton.MouseButton1Click:Connect(function()
    teleportToSpawn()
end)

autoHealButton.MouseButton1Click:Connect(function()
    toggleAutoHeal()
    if autoHealEnabled then
        autoHealButton.Text = "Disable Auto Heal"
    else
        autoHealButton.Text = "Toggle Auto Heal"
    end
end)

nightModeButton.MouseButton1Click:Connect(function()
    toggleNightMode()
    if nightModeEnabled then
        nightModeButton.Text = "Disable Night Mode"
    else
        nightModeButton.Text = "Toggle Night Mode"
    end
end)

antiAFKButton.MouseButton1Click:Connect(function()
    toggleAntiAFK()
    if antiAFKEnabled then
        antiAFKButton.Text = "Disable Anti-AFK"
    else
        antiAFKButton.Text = "Toggle Anti-AFK"
    end
end)

wallClipButton.MouseButton1Click:Connect(function()
    toggleWallClip()
    if wallClipEnabled then
        wallClipButton.Text = "Disable Wall Clip"
    else
        wallClipButton.Text = "Toggle Wall Clip"
    end
end)

if inGunfightArena and GameTab then
    teamCheckButton.MouseButton1Click:Connect(function()
        teamCheckEnabled = not teamCheckEnabled
        if teamCheckEnabled then
            teamCheckButton.Text = "Disable Team Check"
        else
            teamCheckButton.Text = "Enable Team Check"
        end
    end)

    aimbotButton.MouseButton1Click:Connect(function()
        aimbotEnabled = not aimbotEnabled
        if aimbotEnabled then
            aimbotButton.Text = "Disable Aimbot"
        else
            aimbotButton.Text = "Enable Aimbot"
        end
    end)
end

-- Anti-detection note:
-- This script uses minimal events and simple methods to reduce detection risk.
-- However, no script can guarantee full undetectability on Roblox.
-- Use with caution and at your own risk.
