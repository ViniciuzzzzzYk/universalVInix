-- Vex Hub - Universal Script
-- Versão: 2.0
-- GUI redesign inspired by Banana Hub style with tabs, minimize/maximize, and mobile/PC responsiveness

local VexHub = {}

-- Configurações iniciais
VexHub.Settings = {
    Universal = {
        Fly = false,
        FlySpeed = 50,
        Noclip = false,
        ESP = false,
        ESPColor = Color3.fromRGB(0, 255, 0)
    },
    BuildABoat = {
        AutoFarm = false,
        FarmSpeed = 1
    },
    Gunfight = {
        ESP = false,
        TeamCheck = true,
        Aimbot = false,
        AimKey = Enum.UserInputType.MouseButton2,
        AimPart = "Head",
        FOV = 100,
        Smoothness = 0.1,
        IncreaseRange = false,
        NewRange = 500
    }
}

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Lighting = game:GetService("Lighting")

-- Conexões
VexHub.Connections = {}

-- Função para limpar conexões
function VexHub:Cleanup()
    for _, connection in pairs(VexHub.Connections) do
        connection:Disconnect()
    end
    VexHub.Connections = {}
    
    -- Limpar ESPs
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local esp = player.Character:FindFirstChild("VexESP")
            if esp then esp:Destroy() end
        end
    end
end

-- GUI Creation and redesign
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
    local guiHeight = math.clamp(screenSize.Y * 0.7, 350, 500)

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

    -- Tab container
    local TabContainer = Instance.new("Frame")
    TabContainer.Size = UDim2.new(0, 120, 1, -45)
    TabContainer.Position = UDim2.new(0, 0, 0, 45)
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
    local BuildABoatTabButton = createTabButton("Build A Boat", 50)
    local GunfightTabButton = createTabButton("Gunfight", 100)

    -- Content frames for tabs
    local UniversalTab = Instance.new("Frame")
    UniversalTab.Size = UDim2.new(1, -120, 1, -45)
    UniversalTab.Position = UDim2.new(0, 120, 0, 45)
    UniversalTab.BackgroundTransparency = 1
    UniversalTab.Parent = MainFrame

    local BuildABoatTab = Instance.new("Frame")
    BuildABoatTab.Size = UDim2.new(1, -120, 1, -45)
    BuildABoatTab.Position = UDim2.new(0, 120, 0, 45)
    BuildABoatTab.BackgroundTransparency = 1
    BuildABoatTab.Visible = false
    BuildABoatTab.Parent = MainFrame

    local GunfightTab = Instance.new("Frame")
    GunfightTab.Size = UDim2.new(1, -120, 1, -45)
    GunfightTab.Position = UDim2.new(0, 120, 0, 45)
    GunfightTab.BackgroundTransparency = 1
    GunfightTab.Visible = false
    GunfightTab.Parent = MainFrame

    -- ScrollFrames for each tab
    local UniversalScroll = Instance.new("ScrollingFrame")
    UniversalScroll.Size = UDim2.new(1, -20, 1, -20)
    UniversalScroll.Position = UDim2.new(0, 10, 0, 10)
    UniversalScroll.BackgroundTransparency = 1
    UniversalScroll.BorderSizePixel = 0
    UniversalScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    UniversalScroll.ScrollBarThickness = 6
    UniversalScroll.Parent = UniversalTab

    local BuildABoatScroll = Instance.new("ScrollingFrame")
    BuildABoatScroll.Size = UDim2.new(1, -20, 1, -20)
    BuildABoatScroll.Position = UDim2.new(0, 10, 0, 10)
    BuildABoatScroll.BackgroundTransparency = 1
    BuildABoatScroll.BorderSizePixel = 0
    BuildABoatScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    BuildABoatScroll.ScrollBarThickness = 6
    BuildABoatScroll.Parent = BuildABoatTab

    local GunfightScroll = Instance.new("ScrollingFrame")
    GunfightScroll.Size = UDim2.new(1, -20, 1, -20)
    GunfightScroll.Position = UDim2.new(0, 10, 0, 10)
    GunfightScroll.BackgroundTransparency = 1
    GunfightScroll.BorderSizePixel = 0
    GunfightScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    GunfightScroll.ScrollBarThickness = 6
    GunfightScroll.Parent = GunfightTab

    local UniversalLayout = Instance.new("UIListLayout")
    UniversalLayout.SortOrder = Enum.SortOrder.LayoutOrder
    UniversalLayout.Padding = UDim.new(0, 10)
    UniversalLayout.Parent = UniversalScroll

    local BuildABoatLayout = Instance.new("UIListLayout")
    BuildABoatLayout.SortOrder = Enum.SortOrder.LayoutOrder
    BuildABoatLayout.Padding = UDim.new(0, 10)
    BuildABoatLayout.Parent = BuildABoatScroll

    local GunfightLayout = Instance.new("UIListLayout")
    GunfightLayout.SortOrder = Enum.SortOrder.LayoutOrder
    GunfightLayout.Padding = UDim.new(0, 10)
    GunfightLayout.Parent = GunfightScroll

    UniversalLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        UniversalScroll.CanvasSize = UDim2.new(0, 0, 0, UniversalLayout.AbsoluteContentSize.Y + 10)
    end)

    BuildABoatLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        BuildABoatScroll.CanvasSize = UDim2.new(0, 0, 0, BuildABoatLayout.AbsoluteContentSize.Y + 10)
    end)

    GunfightLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        GunfightScroll.CanvasSize = UDim2.new(0, 0, 0, GunfightLayout.AbsoluteContentSize.Y + 10)
    end)

    -- Utility function to create buttons inside a parent frame
    local function createButton(text, parent)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 40)
        btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        btn.TextColor3 = Color3.fromRGB(0, 255, 0)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 20
        btn.Text = text
        btn.AutoButtonColor = true
        btn.Parent = parent

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

    local aimbotFOV = 100 -- default aimbot range
    local aimbotRangeIncreased = false

    -- Helper functions
    local function getCharacter()
        return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    end

    local function getRootPart()
        local char = getCharacter()
        return char and char:FindFirstChild("HumanoidRootPart")
    end

    -- Fly feature
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

    -- Wall clipping feature
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

    -- ESP feature
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

    -- Aimbot feature
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

    -- Create buttons for Universal tab
    local flyButton = createButton("Toggle Fly", UniversalScroll)
    local invisButton = createButton("Toggle Invisibility", UniversalScroll)
    local espButton = createButton("Toggle ESP", UniversalScroll)
    local speedButton = createButton("Toggle Speed", UniversalScroll)
    local jumpButton = createButton("Toggle Jump Boost", UniversalScroll)
    local teleportButton = createButton("Teleport to Spawn", UniversalScroll)
    local autoHealButton = createButton("Toggle Auto Heal", UniversalScroll)
    local nightModeButton = createButton("Toggle Night Mode", UniversalScroll)
    local antiAFKButton = createButton("Toggle Anti-AFK", UniversalScroll)
    local wallClipButton = createButton("Toggle Wall Clip", UniversalScroll)

    -- Create buttons for Gunfight Arena tab
    local teamCheckButton = createButton("Toggle Team Check", GunfightScroll)
    local aimbotButton = createButton("Toggle Aimbot", GunfightScroll)
    local increaseRangeButton = createButton("Increase Aimbot Range", GunfightScroll)

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
        if aimbotFOV == 100 then
            aimbotFOV = 200
            increaseRangeButton.Text = "Decrease Aimbot Range"
        else
            aimbotFOV = 100
            increaseRangeButton.Text = "Increase Aimbot Range"
        end
    end)

    -- Anti-detection note:
    -- This script uses minimal events and simple methods to reduce detection risk.
    -- However, no script can guarantee full undetectability on Roblox.
    -- Use with caution and at your own risk.
end

-- Initialize UI
VexHub:CreateUI()

return VexHub
