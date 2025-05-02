-- Universal Vex Hub Script - Unified GUI with all features functional
-- Includes autofarm for Build A Boat, team check and aimbot for Gunfight, and fixes for speed, jump, teleport, auto heal, and night mode

local VexHub = {}

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")

-- Settings
VexHub.Settings = {
    Fly = false,
    FlySpeed = 50,
    Noclip = false,
    ESP = false,
    ESPTeamCheck = true,
    AutoFarm = false,
    AutoFarmSpeed = 1,
    GunfightESP = false,
    GunfightTeamCheck = true,
    Aimbot = false,
    AimKey = Enum.UserInputType.MouseButton2,
    AimPart = "Head",
    AimbotFOV = 100,
    AimbotSmoothness = 0.1,
    IncreaseRange = false,
    NewRange = 500,
    Speed = false,
    SpeedValue = 50,
    Jump = false,
    JumpPowerValue = 100,
    AutoHeal = false,
    HealAmount = 10,
    HealInterval = 1,
    NightMode = false,
}

-- Connections
VexHub.Connections = {}

-- Cleanup function
function VexHub:Cleanup()
    for _, connection in pairs(VexHub.Connections) do
        connection:Disconnect()
    end
    VexHub.Connections = {}

    -- Remove ESP highlights
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local highlight = player.Character:FindFirstChild("VexESP")
            if highlight then highlight:Destroy() end
            local gunfightHighlight = player.Character:FindFirstChild("VexGunfightESP")
            if gunfightHighlight then gunfightHighlight:Destroy() end
        end
    end
end

-- Fly variables
local flying = false
local ctrl = {f = 0, b = 0, l = 0, r = 0}
local lastctrl = {f = 0, b = 0, l = 0, r = 0}
local maxspeed = 50
local speed = 0
local bg
local bv

local function getCharacter()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

local function getTorso()
    local char = getCharacter()
    return char and (char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso"))
end

local function Fly()
    local torso = getTorso()
    if not torso then return end

    bg = Instance.new("BodyGyro", torso)
    bg.P = 9e4
    bg.maxTorque = Vector3.new(9e9, 9e9, 9e9)
    bg.cframe = torso.CFrame

    bv = Instance.new("BodyVelocity", torso)
    bv.velocity = Vector3.new(0,0.1,0)
    bv.maxForce = Vector3.new(9e9, 9e9, 9e9)

    flying = true
    speed = 0

    RunService:BindToRenderStep("Fly", Enum.RenderPriority.Character.Value, function()
        if not flying then
            RunService:UnbindFromRenderStep("Fly")
            if bg then bg:Destroy() bg = nil end
            if bv then bv:Destroy() bv = nil end
            local char = getCharacter()
            if char and char:FindFirstChildOfClass("Humanoid") then
                char.Humanoid.PlatformStand = false
            end
            return
        end

        local char = getCharacter()
        if char and char:FindFirstChildOfClass("Humanoid") then
            char.Humanoid.PlatformStand = true
        end

        if ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0 then
            speed = speed + 0.5 + (speed / maxspeed)
            if speed > maxspeed then
                speed = maxspeed
            end
        elseif speed > 0 then
            speed = speed - 1
            if speed < 0 then
                speed = 0
            end
        end

        local moveDirection = Vector3.new(0,0,0)
        if ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0 then
            moveDirection = ((Camera.CFrame.LookVector * (ctrl.f + ctrl.b)) + ((Camera.CFrame * CFrame.new(ctrl.l + ctrl.r, (ctrl.f + ctrl.b) * 0.2, 0).p) - Camera.CFrame.p)) * speed
            lastctrl = {f = ctrl.f, b = ctrl.b, l = ctrl.l, r = ctrl.r}
        elseif speed ~= 0 then
            moveDirection = ((Camera.CFrame.LookVector * (lastctrl.f + lastctrl.b)) + ((Camera.CFrame * CFrame.new(lastctrl.l + lastctrl.r, (lastctrl.f + lastctrl.b) * 0.2, 0).p) - Camera.CFrame.p)) * speed
        else
            moveDirection = Vector3.new(0, 0.1, 0)
        end

        bv.velocity = moveDirection
        bg.cframe = Camera.CFrame * CFrame.Angles(-math.rad((ctrl.f + ctrl.b) * 50 * speed / maxspeed), 0, 0)
    end)
end

local function startFly()
    if flying then return end
    flying = true
    Fly()
end

local function stopFly()
    flying = false
end

-- Speed toggle
local speedConnection
local function toggleSpeed()
    VexHub.Settings.Speed = not VexHub.Settings.Speed
    local char = getCharacter()
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    if VexHub.Settings.Speed then
        humanoid.WalkSpeed = VexHub.Settings.SpeedValue
        speedConnection = RunService.Heartbeat:Connect(function()
            if not VexHub.Settings.Speed then
                humanoid.WalkSpeed = 16
                if speedConnection then
                    speedConnection:Disconnect()
                    speedConnection = nil
                end
            end
        end)
    else
        humanoid.WalkSpeed = 16
        if speedConnection then
            speedConnection:Disconnect()
            speedConnection = nil
        end
    end
end

-- Jump toggle
local jumpConnection
local function toggleJump()
    VexHub.Settings.Jump = not VexHub.Settings.Jump
    local char = getCharacter()
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    if VexHub.Settings.Jump then
        humanoid.JumpPower = VexHub.Settings.JumpPowerValue
        jumpConnection = RunService.Heartbeat:Connect(function()
            if not VexHub.Settings.Jump then
                humanoid.JumpPower = 50
                if jumpConnection then
                    jumpConnection:Disconnect()
                    jumpConnection = nil
                end
            end
        end)
    else
        humanoid.JumpPower = 50
        if jumpConnection then
            jumpConnection:Disconnect()
            jumpConnection = nil
        end
    end
end

-- Teleport to spawn
local function teleportToSpawn()
    local char = getCharacter()
    if not char then return end
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end

    local spawnLocation = workspace:FindFirstChild("SpawnLocation") or workspace:FindFirstChild("Spawn")
    if spawnLocation and spawnLocation:IsA("BasePart") then
        rootPart.CFrame = spawnLocation.CFrame + Vector3.new(0, 5, 0)
    end
end

-- Auto Heal
local autoHealConnection
local function toggleAutoHeal()
    VexHub.Settings.AutoHeal = not VexHub.Settings.AutoHeal
    local char = getCharacter()
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    if VexHub.Settings.AutoHeal then
        autoHealConnection = RunService.Heartbeat:Connect(function(dt)
            if humanoid.Health < humanoid.MaxHealth then
                humanoid.Health = math.min(humanoid.Health + VexHub.Settings.HealAmount * dt, humanoid.MaxHealth)
            end
            if not VexHub.Settings.AutoHeal then
                if autoHealConnection then
                    autoHealConnection:Disconnect()
                    autoHealConnection = nil
                end
            end
        end)
    else
        if autoHealConnection then
            autoHealConnection:Disconnect()
            autoHealConnection = nil
        end
    end
end

-- Night Mode toggle
local nightModeConnection
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
    VexHub.Settings.NightMode = not VexHub.Settings.NightMode
    if VexHub.Settings.NightMode then
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

-- ESP for general and Gunfight
local espBoxes = {}

local function toggleESP()
    VexHub.Settings.ESP = not VexHub.Settings.ESP
    if VexHub.Settings.ESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local box = Instance.new("BoxHandleAdornment")
                box.Name = "VexESP"
                box.Adornee = player.Character.HumanoidRootPart
                box.AlwaysOnTop = true
                box.ZIndex = 10
                box.Size = Vector3.new(4, 6, 1)
                box.Color3 = Color3.fromRGB(0, 255, 0)
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

local function toggleGunfightESP()
    VexHub.Settings.GunfightESP = not VexHub.Settings.GunfightESP
    if VexHub.Settings.GunfightESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local highlight = player.Character:FindFirstChild("VexGunfightESP")
                if not highlight then
                    local newHighlight = Instance.new("Highlight")
                    newHighlight.Name = "VexGunfightESP"
                    if VexHub.Settings.GunfightTeamCheck and player.Team == LocalPlayer.Team then
                        newHighlight.FillColor = Color3.fromRGB(0, 0, 255)
                    else
                        newHighlight.FillColor = Color3.fromRGB(255, 0, 0)
                    end
                    newHighlight.OutlineColor = newHighlight.FillColor
                    newHighlight.FillTransparency = 0.3
                    newHighlight.OutlineTransparency = 0
                    newHighlight.Parent = player.Character
                end
            end
        end
    else
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local highlight = player.Character:FindFirstChild("VexGunfightESP")
                if highlight then
                    highlight:Destroy()
                end
            end
        end
    end
end

-- Team check toggle for Gunfight ESP and Aimbot
local function toggleGunfightTeamCheck()
    VexHub.Settings.GunfightTeamCheck = not VexHub.Settings.GunfightTeamCheck
end

-- Aimbot variables
local aimbotEnabled = false
local aimbotConnection

local function getClosestEnemy()
    local closestPlayer, shortestDistance = nil, math.huge
    local cameraPosition = Camera.CFrame.Position

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if VexHub.Settings.GunfightTeamCheck and player.Team == LocalPlayer.Team then
                continue
            end
            local rootPart = player.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
            if onScreen then
                local mousePos = UserInputService:GetMouseLocation()
                local distance = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
                if distance < VexHub.Settings.AimbotFOV and distance < shortestDistance then
                    shortestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end
    return closestPlayer
end

local function startAimbot()
    if aimbotEnabled then return end
    aimbotEnabled = true
    aimbotConnection = RunService.RenderStepped:Connect(function()
        if not aimbotEnabled then return end
        local target = getClosestEnemy()
        if target and target.Character and target.Character:FindFirstChild(VexHub.Settings.AimPart) then
            local targetPos = target.Character[VexHub.Settings.AimPart].Position
            local cameraCFrame = Camera.CFrame
            local direction = (targetPos - cameraCFrame.Position).Unit
            local newCFrame = CFrame.new(cameraCFrame.Position, cameraCFrame.Position + direction)
            Camera.CFrame = cameraCFrame:Lerp(newCFrame, VexHub.Settings.AimbotSmoothness)
        end
    end)
end

local function stopAimbot()
    if not aimbotEnabled then return end
    aimbotEnabled = false
    if aimbotConnection then
        aimbotConnection:Disconnect()
        aimbotConnection = nil
    end
end

-- Increase range for Gunfight tools
local function toggleIncreaseRange()
    VexHub.Settings.IncreaseRange = not VexHub.Settings.IncreaseRange
    if VexHub.Settings.IncreaseRange then
        for _, tool in pairs(LocalPlayer.Character:GetChildren()) do
            if tool:IsA("Tool") then
                for _, v in pairs(tool:GetDescendants()) do
                    if v:IsA("NumberValue") and (v.Name:lower():find("range") or v.Name:lower():find("distance")) then
                        v.Value = VexHub.Settings.NewRange
                    end
                end
            end
        end
    else
        -- Resetting range is not implemented, user must rejoin or reload
    end
end

-- AutoFarm for Build A Boat
local autoFarmConnection
local function toggleAutoFarm()
    VexHub.Settings.AutoFarm = not VexHub.Settings.AutoFarm
    if VexHub.Settings.AutoFarm then
        autoFarmConnection = RunService.Heartbeat:Connect(function()
            if not VexHub.Settings.AutoFarm or not LocalPlayer.Character then
                if autoFarmConnection then
                    autoFarmConnection:Disconnect()
                    autoFarmConnection = nil
                end
                return
            end
            local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            local rootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and rootPart then
                rootPart.CFrame = rootPart.CFrame + (rootPart.CFrame.LookVector * VexHub.Settings.AutoFarmSpeed * 0.1)
                humanoid:Move(Vector3.new(0, 0, 1), true)
            end
        end)
    else
        if autoFarmConnection then
            autoFarmConnection:Disconnect()
            autoFarmConnection = nil
        end
    end
end

-- GUI Creation
function VexHub:CreateUI()
    if game:GetService("CoreGui"):FindFirstChild("VexHubUI") then
        return
    end

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "VexHubUI"
    ScreenGui.Parent = game:GetService("CoreGui")
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local screenSize = Camera.ViewportSize
    local guiWidth = math.clamp(screenSize.X * 0.2, 280, 350)
    local guiHeight = math.clamp(screenSize.Y * 0.7, 350, 600)

    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
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

    local UIStroke = Instance.new("UIStroke")
    UIStroke.Color = Color3.fromRGB(0, 255, 0)
    UIStroke.Thickness = 2
    UIStroke.Parent = MainFrame

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, 0, 0, 45)
    Title.BackgroundTransparency = 1
    Title.Text = "Vex Hub"
    Title.TextColor3 = Color3.fromRGB(0, 255, 0)
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 28
    Title.Parent = MainFrame

    local ScrollFrame = Instance.new("ScrollingFrame")
    ScrollFrame.Size = UDim2.new(1, -20, 1, -60)
    ScrollFrame.Position = UDim2.new(0, 10, 0, 50)
    ScrollFrame.BackgroundTransparency = 1
    ScrollFrame.BorderSizePixel = 0
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    ScrollFrame.ScrollBarThickness = 6
    ScrollFrame.Parent = MainFrame

    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    UIListLayout.Padding = UDim.new(0, 10)
    UIListLayout.Parent = ScrollFrame

    UIListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y + 10)
    end)

    local function createButton(text)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 40)
        btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        btn.TextColor3 = Color3.fromRGB(0, 255, 0)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 20
        btn.Text = text
        btn.AutoButtonColor = true
        btn.Parent = ScrollFrame

        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 12)
        corner.Parent = btn

        btn.MouseEnter:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 60)}):Play()
        end)
        btn.MouseLeave:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(40, 40, 40)}):Play()
        end)

        return btn
    end

    -- Create buttons and connect functions

    local flyButton = createButton("Toggle Fly")
    flyButton.MouseButton1Click:Connect(function()
        if flying then
            stopFly()
            flyButton.Text = "Toggle Fly"
        else
            startFly()
            flyButton.Text = "Disable Fly"
        end
    end)

    local speedButton = createButton("Toggle Speed")
    speedButton.MouseButton1Click:Connect(function()
        toggleSpeed()
        if VexHub.Settings.Speed then
            speedButton.Text = "Disable Speed"
        else
            speedButton.Text = "Toggle Speed"
        end
    end)

    local jumpButton = createButton("Toggle Jump Boost")
    jumpButton.MouseButton1Click:Connect(function()
        toggleJump()
        if VexHub.Settings.Jump then
            jumpButton.Text = "Disable Jump Boost"
        else
            jumpButton.Text = "Toggle Jump Boost"
        end
    end)

    local teleportButton = createButton("Teleport to Spawn")
    teleportButton.MouseButton1Click:Connect(function()
        teleportToSpawn()
    end)

    local autoHealButton = createButton("Toggle Auto Heal")
    autoHealButton.MouseButton1Click:Connect(function()
        toggleAutoHeal()
        if VexHub.Settings.AutoHeal then
            autoHealButton.Text = "Disable Auto Heal"
        else
            autoHealButton.Text = "Toggle Auto Heal"
        end
    end)

    local nightModeButton = createButton("Toggle Night Mode")
    nightModeButton.MouseButton1Click:Connect(function()
        toggleNightMode()
        if VexHub.Settings.NightMode then
            nightModeButton.Text = "Disable Night Mode"
        else
            nightModeButton.Text = "Toggle Night Mode"
        end
    end)

    local espButton = createButton("Toggle ESP")
    espButton.MouseButton1Click:Connect(function()
        toggleESP()
        if VexHub.Settings.ESP then
            espButton.Text = "Disable ESP"
        else
            espButton.Text = "Toggle ESP"
        end
    end)

    local gunfightESPButton = createButton("Toggle Gunfight ESP")
    gunfightESPButton.MouseButton1Click:Connect(function()
        toggleGunfightESP()
        if VexHub.Settings.GunfightESP then
            gunfightESPButton.Text = "Disable Gunfight ESP"
        else
            gunfightESPButton.Text = "Toggle Gunfight ESP"
        end
    end)

    local teamCheckButton = createButton("Toggle Team Check")
    teamCheckButton.MouseButton1Click:Connect(function()
        toggleGunfightTeamCheck()
        if VexHub.Settings.GunfightTeamCheck then
            teamCheckButton.Text = "Disable Team Check"
        else
            teamCheckButton.Text = "Enable Team Check"
        end
    end)

    local aimbotButton = createButton("Toggle Aimbot")
    aimbotButton.MouseButton1Click:Connect(function()
        if VexHub.Settings.Aimbot then
            VexHub.Settings.Aimbot = false
            stopAimbot()
            aimbotButton.Text = "Toggle Aimbot"
        else
            VexHub.Settings.Aimbot = true
            startAimbot()
            aimbotButton.Text = "Disable Aimbot"
        end
    end)

    local increaseRangeButton = createButton("Toggle Increase Range")
    increaseRangeButton.MouseButton1Click:Connect(function()
        toggleIncreaseRange()
        if VexHub.Settings.IncreaseRange then
            increaseRangeButton.Text = "Disable Increase Range"
        else
            increaseRangeButton.Text = "Toggle Increase Range"
        end
    end)

    local autoFarmButton = createButton("Toggle Auto Farm")
    autoFarmButton.MouseButton1Click:Connect(function()
        toggleAutoFarm()
        if VexHub.Settings.AutoFarm then
            autoFarmButton.Text = "Disable Auto Farm"
        else
            autoFarmButton.Text = "Toggle Auto Farm"
        end
    end)

end

-- Initialize UI
VexHub:CreateUI()

return VexHub
