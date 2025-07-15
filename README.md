-- RLK HUB - Roblox Lua client script with modern GUI using Roblox GUI objects
-- Features: Fly, Walkspeed, ESP, Aimbot, Infinite Jump, GUI toggle, Notifications

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local StarterGui = game:GetService("StarterGui")

-- Wait for character and camera to load
repeat wait() until LocalPlayer and LocalPlayer.Character and Camera

-- Feature states
local flyEnabled = false
local flySpeed = 50
local bodyVelocity

local walkspeedEnabled = false
local walkspeedValue = 16

local espEnabled = false
local espHighlights = {}

local aimbotEnabled = false
local infiniteJumpEnabled = false

-- Notifications
local function notify(text)
    StarterGui:SetCore("SendNotification", {
        Title = "RLK HUB";
        Text = text;
        Duration = 3;
    })
end

-- Create main ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RLKHubGui"
screenGui.ResetOnSpawn = false
screenGui.Enabled = false
screenGui.Parent = game.CoreGui or game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")

-- Main frame
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 400, 0, 300)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Title label
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
titleLabel.BorderSizePixel = 0
titleLabel.Text = "RLK HUB"
titleLabel.TextColor3 = Color3.new(1,1,1)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 24
titleLabel.Parent = mainFrame

-- Tabs frame
local tabsFrame = Instance.new("Frame")
tabsFrame.Size = UDim2.new(1, 0, 0, 40)
tabsFrame.Position = UDim2.new(0, 0, 0, 40)
tabsFrame.BackgroundTransparency = 1
tabsFrame.Parent = mainFrame

local tabs = {"Steal a brainrot", "#1 game", "Universal"}
local currentTab = "Steal a brainrot"
local tabButtons = {}

local function clearContent()
    for _, child in pairs(mainFrame:GetChildren()) do
        if child.Name == "Content" then
            child:Destroy()
        end
    end
end

local function createTabButton(name, position)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 130, 1, 0)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.BorderSizePixel = 0
    button.Text = name
    button.TextColor3 = Color3.new(1,1,1)
    button.Font = Enum.Font.Gotham
    button.TextSize = 18
    button.Parent = tabsFrame

    button.MouseButton1Click:Connect(function()
        currentTab = name
        updateTabs()
        updateContent()
    end)

    return button
end

local function updateTabs()
    for name, button in pairs(tabButtons) do
        if name == currentTab then
            button.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        else
            button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        end
    end
end

local function createToggle(parent, text, position, initialState, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, -20, 0, 30)
    frame.Position = position
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.Position = UDim2.new(0, 0, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.new(1,1,1)
    label.Font = Enum.Font.Gotham
    label.TextSize = 18
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0, 50, 0, 25)
    toggle.Position = UDim2.new(1, -50, 0, 2)
    toggle.BackgroundColor3 = initialState and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(150, 0, 0)
    toggle.BorderSizePixel = 0
    toggle.Text = initialState and "ON" or "OFF"
    toggle.TextColor3 = Color3.new(1,1,1)
    toggle.Font = Enum.Font.GothamBold
    toggle.TextSize = 16
    toggle.Parent = frame

    toggle.MouseButton1Click:Connect(function()
        local newState = not (toggle.Text == "ON")
        toggle.Text = newState and "ON" or "OFF"
        toggle.BackgroundColor3 = newState and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(150, 0, 0)
        callback(newState)
    end)

    return frame
end

local function createSlider(parent, text, position, min, max, initialValue, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, -20, 0, 40)
    frame.Position = position
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.7, 0, 0, 20)
    label.Position = UDim2.new(0, 0, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.new(1,1,1)
    label.Font = Enum.Font.Gotham
    label.TextSize = 18
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local sliderFrame = Instance.new("Frame")
    sliderFrame.Size = UDim2.new(1, 0, 0, 15)
    sliderFrame.Position = UDim2.new(0, 0, 0, 25)
    sliderFrame.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    sliderFrame.BorderSizePixel = 0
    sliderFrame.Parent = frame

    local sliderButton = Instance.new("TextButton")
    sliderButton.Size = UDim2.new(0, 20, 1, 0)
    sliderButton.Position = UDim2.new((initialValue - min) / (max - min), 0, 0, 0)
    sliderButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    sliderButton.BorderSizePixel = 0
    sliderButton.Text = ""
    sliderButton.Parent = sliderFrame

    local dragging = false

    sliderButton.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)

    sliderButton.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    sliderFrame.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local relativeX = math.clamp(input.Position.X - sliderFrame.AbsolutePosition.X, 0, sliderFrame.AbsoluteSize.X)
            local value = min + (relativeX / sliderFrame.AbsoluteSize.X) * (max - min)
            value = math.floor(value)
            sliderButton.Position = UDim2.new((value - min) / (max - min), 0, 0, 0)
            callback(value)
        end
    end)

    return frame
end

local function updateContent()
    clearContent()
    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "Content"
    contentFrame.Size = UDim2.new(1, -20, 1, -90)
    contentFrame.Position = UDim2.new(0, 10, 0, 90)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame

    if currentTab == "Steal a brainrot" then
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = "Steal a brainrot tab content placeholder"
        label.TextColor3 = Color3.new(1,1,1)
        label.Font = Enum.Font.Gotham
        label.TextSize = 20
        label.Parent = contentFrame
    elseif currentTab == "#1 game" then
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = "#1 game tab content placeholder"
        label.TextColor3 = Color3.new(1,1,1)
        label.Font = Enum.Font.Gotham
        label.TextSize = 20
        label.Parent = contentFrame
    elseif currentTab == "Universal" then
        -- Fly toggle
        createToggle(contentFrame, "Fly", UDim2.new(0, 0, 0, 0), flyEnabled, function(state)
            flyEnabled = state
            if flyEnabled then
                enableFly()
            else
                if bodyVelocity then
                    bodyVelocity:Destroy()
                    bodyVelocity = nil
                end
                notify("Fly disabled")
            end
        end)

        -- Walkspeed toggle
        createToggle(contentFrame, "Walkspeed", UDim2.new(0, 0, 0, 40), walkspeedEnabled, function(state)
            walkspeedEnabled = state
            if walkspeedEnabled then
                setWalkspeed(walkspeedValue)
            else
                setWalkspeed(16)
            end
        end)

        -- Walkspeed slider
        createSlider(contentFrame, "Walkspeed Value", UDim2.new(0, 0, 0, 80), 16, 100, walkspeedValue, function(value)
            walkspeedValue = value
            if walkspeedEnabled then
                setWalkspeed(walkspeedValue)
            end
        end)

        -- ESP toggle
        createToggle(contentFrame, "ESP", UDim2.new(0, 0, 0, 120), espEnabled, function(state)
            espEnabled = state
            if espEnabled then
                enableESP()
            else
                disableESP()
            end
        end)

        -- Aimbot toggle
        createToggle(contentFrame, "Aimbot", UDim2.new(0, 0, 0, 160), aimbotEnabled, function(state)
            aimbotEnabled = state
        end)

        -- Infinite Jump toggle
        createToggle(contentFrame, "Infinite Jump", UDim2.new(0, 0, 0, 200), infiniteJumpEnabled, function(state)
            infiniteJumpEnabled = state
        end)
    end
end

-- Fly functions
local function enableFly()
    local character = LocalPlayer.Character
    if not character then return end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    if bodyVelocity then bodyVelocity:Destroy() end
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.Parent = hrp
    notify("Fly enabled")
end

-- Fly movement control
RunService.Heartbeat:Connect(function()
    if flyEnabled and bodyVelocity then
        local direction = Vector3.new(0,0,0)
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            direction = direction + Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            direction = direction - Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            direction = direction - Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            direction = direction + Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            direction = direction + Vector3.new(0,1,0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
            direction = direction - Vector3.new(0,1,0)
        end
        if direction.Magnitude > 0 then
            bodyVelocity.Velocity = direction.Unit * flySpeed
        else
            bodyVelocity.Velocity = Vector3.new(0,0,0)
        end
    end
end)

-- Walkspeed control
local function setWalkspeed(value)
    local character = LocalPlayer.Character
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = value
    end
end

-- ESP functions
local function createHighlight(player)
    local highlight = Instance.new("Highlight")
    highlight.Adornee = player.Character
    highlight.FillColor = Color3.fromRGB(0, 255, 0)
    highlight.OutlineColor = Color3.fromRGB(0, 255, 0)
    highlight.Parent = player.Character
    return highlight
end

local function enableESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            if not espHighlights[player] then
                espHighlights[player] = createHighlight(player)
            end
        end
    end
    notify("ESP enabled")
end

local function disableESP()
    for player, highlight in pairs(espHighlights) do
        if highlight then
            highlight:Destroy()
        end
    end
    espHighlights = {}
    notify("ESP disabled")
end

-- Aimbot functions
local function getClosestTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local pos = player.Character.HumanoidRootPart.Position
            local screenPos, onScreen = Camera:WorldToViewportPoint(pos)
            if onScreen then
                local mousePos = UserInputService:GetMouseLocation()
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    closestPlayer = player
                end
            end
        end
    end
    return closestPlayer
end

RunService.Heartbeat:Connect(function()
    if aimbotEnabled then
        local target = getClosestTarget()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = target.Character.HumanoidRootPart
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, hrp.Position)
        end
    end
end)

-- Infinite jump
UserInputService.JumpRequest:Connect(function()
    if infiniteJumpEnabled then
        local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- GUI toggle keybind (RightShift)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        screenGui.Enabled = not screenGui.Enabled
        notify("GUI toggled: "..tostring(screenGui.Enabled))
    end
end)

-- Initialize tabs
for i, tabName in ipairs(tabs) do
    local pos = UDim2.new(0, (i-1)*135, 0, 0)
    tabButtons[tabName] = createTabButton(tabName, pos)
end

updateTabs()
updateContent()

print("RLK HUB script loaded, press RightShift to toggle GUI")
