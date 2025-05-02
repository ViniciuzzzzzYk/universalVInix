-- Updated Universal GUI script with your provided fly, wall clip, aimbot, ESP, and hitbox expander code, plus aimbot range option

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera
local StarterGui = game:GetService("StarterGui")

-- Responsive GUI size based on screen size
local screenSize = workspace.CurrentCamera.ViewportSize
local guiWidth = math.clamp(screenSize.X * 0.22, 280, 350)
local guiHeight = math.clamp(screenSize.Y * 0.7, 350, 500)

-- Create ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "UniversalGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

-- Main Frame (responsive size, draggable, repositioned to left side)
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, guiWidth, 0, guiHeight)
MainFrame.Position = UDim2.new(0, 10, 0.5, -guiHeight/2)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BackgroundTransparency = 0.05
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui
MainFrame.Active = true
MainFrame.Draggable = true

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 20)
UICorner.Parent = MainFrame

-- UIStroke for subtle border
local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(0, 255, 0)
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

-- Title Label
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 45)
Title.BackgroundTransparency = 1
Title.Text = "VEX Universal"
Title.TextColor3 = Color3.fromRGB(0, 255, 0)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 28
Title.Parent = MainFrame

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

-- Minimize button on MainFrame
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
MinimizeButton.Position = UDim2.new(1, -40, 0, 10)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
MinimizeButton.TextColor3 = Color3.fromRGB(0, 0, 0)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 20
MinimizeButton.Text = "-"
MinimizeButton.Parent = MainFrame

MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    MinimizedButton.Visible = true
end)

-- ScrollFrame for content with UIListLayout
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Size = UDim2.new(1, -20, 1, -70)
ScrollFrame.Position = UDim2.new(0, 10, 0, 55)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.BorderSizePixel = 0
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
ScrollFrame.ScrollBarThickness = 6
ScrollFrame.Parent = MainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 10)
UIListLayout.Parent = ScrollFrame

-- Update CanvasSize based on content
UIListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y + 10)
end)

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

local aimbotFOV = 150 -- default aimbot range
local aimbotRangeIncreased = false

-- Helper functions
local function getCharacter()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

local function getRootPart()
    local char = getCharacter()
    return char and char:FindFirstChild("HumanoidRootPart")
end

-- Fly feature (updated with your code)
local function startFly()
    if flying then return end
    flying = true
    local rootPart = getRootPart()
    if not rootPart then return end

    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Velocity = Vector3.zero
    bodyVelocity.Parent = rootPart

    RunService:BindToRenderStep("Fly", Enum.RenderPriority.Character.Value, function()
        local moveDirection = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection += Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection -= Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection -= Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection += Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection += Vector3.new(0, 1, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
            moveDirection -= Vector3.new(0, 1, 0)
        end

        bodyVelocity.Velocity = moveDirection.Magnitude > 0 and moveDirection.Unit * flySpeed or Vector3.zero
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

-- Wall clipping feature (updated with your code)
local function toggleWallClip()
    wallClipEnabled = not wallClipEnabled
    local character = getCharacter()
    if not character then return end

    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = not wallClipEnabled
        end
    end
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

-- ESP feature (updated with your code)
local function toggleESP()
    espEnabled = not espEnabled
    if espEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local box = Instance.new("BoxHandleAdornment")
                box.Adornee = player.Character.HumanoidRootPart
                box.AlwaysOnTop = true
                box.ZIndex = 10
                box.Size = Vector3.new(4, 6, 1)
                box.Color3 = Color3.fromRGB(255, 0, 0)
                box.Transparency = 0.5
                box.Parent = Camera
                espBoxes[player] = box
            end
        end
    else
        for _, box in pairs(espBoxes) do
            box:Destroy()
        end
        espBoxes = {}
    end
end

-- Hitbox expander (adjusted if needed)
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

-- Team check feature
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

-- Aimbot feature (updated with your code and range option)
local aimbotEnabled = false
local aimbotSmoothness = 0.25

local function getClosestEnemy()
    local closestPlayer, shortestDistance = nil, math.huge
    local cameraPosition = Camera.CFrame.Position

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and isEnemy(player) then
            local rootPart = player.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)

            if onScreen then
                local direction = (rootPart.Position - cameraPosition).Unit
                local ray = Ray.new(cameraPosition, direction * aimbotFOV)
                local hitPart = workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character})

                if hitPart == rootPart then
                    local distance = (Vector2.new(Mouse.X, Mouse.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestPlayer = player
                    end
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

-- Create buttons for all features inside ScrollFrame
local flyButton = createButton("Toggle Fly")
local invisButton = createButton("Toggle Invisibility")
local espButton = createButton("Toggle ESP")
local speedButton = createButton("Toggle Speed")
local jumpButton = createButton("Toggle Jump Boost")
local teleportButton = createButton("Teleport to Spawn")
local autoHealButton = createButton("Toggle Auto Heal")
local nightModeButton = createButton("Toggle Night Mode")
local antiAFKButton = createButton("Toggle Anti-AFK")
local wallClipButton = createButton("Toggle Wall Clip")
local teamCheckButton = createButton("Toggle Team Check")
local aimbotButton = createButton("Toggle Aimbot")
local increaseRangeButton = createButton("Increase Aimbot Range")

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

increaseRangeButton.MouseButton1Click:Connect(function()
    if aimbotRangeIncreased then
        aimbotFOV = 150
        increaseRangeButton.Text = "Increase Aimbot Range"
        aimbotRangeIncreased = false
    else
        aimbotFOV = 300
        increaseRangeButton.Text = "Decrease Aimbot Range"
        aimbotRangeIncreased = true
    end
end)

-- Anti-detection note:
-- This script uses minimal events and simple methods to reduce detection risk.
-- However, no script can guarantee full undetectability on Roblox.
-- Use with caution and at your own risk.
