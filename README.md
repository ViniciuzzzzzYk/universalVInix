-- Universal Vex Hub Script - Single GUI with all features integrated
-- Includes BAB Farm (teleport autofarm), ESP with team check and aimbot, and other features unified

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
    ESP = false,
    ESPTeamCheck = true,
    BABFarm = false,
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

-- Fly variables and improved movement
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
            moveDirection = ((Camera.CFrame.LookVector * (ctrl.f + ctrl.b)) + ((Camera.CFrame * CFrame.new(ctrl.l + ctrl.r, 0, (ctrl.f + ctrl.b) * 0.2).p) - Camera.CFrame.p)) * speed
            lastctrl = {f = ctrl.f, b = ctrl.b, l = ctrl.l, r = ctrl.r}
        elseif speed ~= 0 then
            moveDirection = ((Camera.CFrame.LookVector * (lastctrl.f + lastctrl.b)) + ((Camera.CFrame * CFrame.new(lastctrl.l + lastctrl.r, 0, (lastctrl.f + lastctrl.b) * 0.2).p) - Camera.CFrame.p)) * speed
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

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        local key = input.KeyCode
        if key == Enum.KeyCode.W then
            ctrl.f = 1
        elseif key == Enum.KeyCode.S then
            ctrl.b = -1
        elseif key == Enum.KeyCode.A then
            ctrl.l = -1
        elseif key == Enum.KeyCode.D then
            ctrl.r = 1
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        local key = input.KeyCode
        if key == Enum.KeyCode.W then
            ctrl.f = 0
        elseif key == Enum.KeyCode.S then
            ctrl.b = 0
        elseif key == Enum.KeyCode.A then
            ctrl.l = 0
        elseif key == Enum.KeyCode.D then
            ctrl.r = 0
        end
    end
end)

-- BAB Farm (Build A Boat) teleport autofarm
local BABFarmConnection
local BABFarmLocations = {
    CFrame.new(-61, 70.739624, 125),
    CFrame.new(-55.7065125, 70.739624, 9492.35645),
    CFrame.new(-55.7065125, -360.739624, 9492.35645),
}

local function toggleBABFarm()
    VexHub.Settings.BABFarm = not VexHub.Settings.BABFarm
    local char = getCharacter()
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not hum or not rootPart then return end

    if VexHub.Settings.BABFarm then
        BABFarmConnection = RunService.Heartbeat:Connect(function()
            if not VexHub.Settings.BABFarm or not LocalPlayer.Character then
                if BABFarmConnection then
                    BABFarmConnection:Disconnect()
                    BABFarmConnection = nil
                end
                return
            end
            for _, location in ipairs(BABFarmLocations) do
                if not VexHub.Settings.BABFarm then break end
                local tween = TweenService:Create(rootPart, TweenInfo.new(3), {CFrame = location})
                tween:Play()
                tween.Completed:Wait()
                wait(0.1)
                if hum.Health == 0 then break end
            end
            if hum.Health > 0 then
                workspace.ClaimRiverResultsGold:FireServer()
            end
        end)
    else
        if BABFarmConnection then
            BABFarmConnection:Disconnect()
            BABFarmConnection = nil
        end
    end
end

-- ESP for general and Gunfight with team check integrated
local espBoxes = {}

local function toggleESP()
    VexHub.Settings.ESP = not VexHub.Settings.ESP
    if VexHub.Settings.ESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                if VexHub.Settings.ESPTeamCheck and player.Team == LocalPlayer.Team then
                    -- Skip teammates if team check enabled
                else
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
        end
    else
        for _, box in pairs(espBoxes) do
            box:Destroy()
        end
        espBoxes = {}
    end
end

-- Gunfight ESP integrated with team check
local function toggleGunfightESP()
    VexHub.Settings.GunfightESP = not VexHub.Settings.GunfightESP
    if VexHub.Settings.GunfightESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                if VexHub.Settings.GunfightTeamCheck and player.Team == LocalPlayer.Team then
                    -- Skip teammates if team check enabled
                else
                    local highlight = player.Character:FindFirstChild("VexGunfightESP")
                    if not highlight then
                        local newHighlight = Instance.new("Highlight")
                        newHighlight.Name = "VexGunfightESP"
                        newHighlight.FillColor = Color3.fromRGB(255, 0, 0)
                        newHighlight.OutlineColor = newHighlight.FillColor
                        newHighlight.FillTransparency = 0.3
                        newHighlight.OutlineTransparency = 0
                        newHighlight.Parent = player.Character
                    end
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

-- Team check toggle for ESP and Aimbot
local function toggleTeamCheck()
    VexHub.Settings.ESPTeamCheck = not VexHub.Settings.ESPTeamCheck
    VexHub.Settings.GunfightTeamCheck = VexHub.Settings.ESPTeamCheck
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

-- GUI Creation single unified GUI
function VexHub:CreateUI()
    if game:GetService("CoreGui"):FindFirstChild("VexHubUI") then
        return
    end

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "VexHubUI"
    ScreenGui.Parent = game:GetService("CoreGui")
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local screenSize = Camera.ViewportSize
    local guiWidth = math.clamp(screenSize.X * 0.22, 280, 350)
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

    -- Minimize button
    local MinimizeButton = Instance.new("TextButton")
    MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
    MinimizeButton.Position = UDim2.new(1, -40, 0, 10)
    MinimizeButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    MinimizeButton.TextColor3 = Color3.fromRGB(0, 0, 0)
    MinimizeButton.Font = Enum.Font.GothamBold
    MinimizeButton.TextSize = 20
    MinimizeButton.Text = "-"
    MinimizeButton.Parent = MainFrame

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

    MinimizeButton.MouseButton1Click:Connect(function()
        MainFrame.Visible = false
        MinimizedButton.Visible = true
    end)

    MinimizedButton.MouseButton1Click:Connect(function()
        MainFrame.Visible = true
        MinimizedButton.Visible = false
    end)

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
        VexHub.Settings.Speed = not VexHub.Settings.Speed
        local char = getCharacter()
        if not char then return end
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end

        if VexHub.Settings.Speed then
            humanoid.WalkSpeed = VexHub.Settings.SpeedValue
            speedButton.Text = "Disable Speed"
        else
            humanoid.WalkSpeed = 16
            speedButton.Text = "Toggle Speed"
        end
    end)

    local jumpButton = createButton("Toggle Jump Boost")
    jumpButton.MouseButton1Click:Connect(function()
        VexHub.Settings.Jump = not VexHub.Settings.Jump
        local char = getCharacter()
        if not char then return end
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end

        if VexHub.Settings.Jump then
            humanoid.JumpPower = VexHub.Settings.JumpPowerValue
            jumpButton.Text = "Disable Jump Boost"
        else
            humanoid.JumpPower = 50
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
        toggleTeamCheck()
        if VexHub.Settings.ESPTeamCheck then
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

    local BABFarmButton = createButton("Toggle BAB Farm")
    BABFarmButton.MouseButton1Click:Connect(function()
        toggleBABFarm()
        if VexHub.Settings.BABFarm then
            BABFarmButton.Text = "Disable BAB Farm"
        else
            BABFarmButton.Text = "Toggle BAB Farm"
        end
    end)
end

-- Initialize UI
VexHub:CreateUI()

return VexHub
