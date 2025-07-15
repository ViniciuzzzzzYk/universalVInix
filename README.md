-- RLK HUB - Modern Roblox Lua client script for Swift PC executor with GUI tabs and features
-- Features: Fly, Walkspeed, ESP, Aimbot, Infinite Jump, GUI toggle, Notifications

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

-- Wait for character and camera to load
repeat wait() until LocalPlayer and LocalPlayer.Character and Camera

-- GUI state
local guiVisible = false
local currentTab = "Steal a brainrot"

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
    print("[RLK HUB Notification] "..text)
    -- Could add Drawing API notification here if desired
end

-- Create Drawing API GUI elements
local Drawing = Drawing

-- Main window
local windowSize = Vector2.new(400, 300)
local windowPos = Vector2.new((Camera.ViewportSize.X - windowSize.X) / 2, (Camera.ViewportSize.Y - windowSize.Y) / 2)

local window = Drawing.new("Square")
window.Size = windowSize
window.Position = windowPos
window.Color = Color3.fromRGB(30, 30, 30)
window.Filled = true
window.Visible = false
window.Transparency = 0.9
window.ZIndex = 999

-- Title text
local title = Drawing.new("Text")
title.Text = "RLK HUB"
title.Size = 24
title.Position = windowPos + Vector2.new(10, 10)
title.Color = Color3.fromRGB(255, 255, 255)
title.Visible = false
title.Center = false
title.Outline = true
title.OutlineColor = Color3.new(0, 0, 0)
title.ZIndex = 1000

-- Tabs
local tabs = {"Steal a brainrot", "#1 game", "Universal"}
local tabButtons = {}

local tabStartX = windowPos.X + 10
local tabStartY = windowPos.Y + 40
local tabWidth = 120
local tabHeight = 30

for i, tabName in ipairs(tabs) do
    local tabButton = Drawing.new("Square")
    tabButton.Size = Vector2.new(tabWidth, tabHeight)
    tabButton.Position = Vector2.new(tabStartX + (i-1)*(tabWidth + 10), tabStartY)
    tabButton.Color = Color3.fromRGB(50, 50, 50)
    tabButton.Filled = true
    tabButton.Visible = false
    tabButton.Transparency = 0.9
    tabButton.ZIndex = 1000

    local tabText = Drawing.new("Text")
    tabText.Text = tabName
    tabText.Size = 18
    tabText.Position = tabButton.Position + Vector2.new(10, 5)
    tabText.Color = Color3.fromRGB(255, 255, 255)
    tabText.Visible = false
    tabText.Center = false
    tabText.Outline = true
    tabText.OutlineColor = Color3.new(0, 0, 0)
    tabText.ZIndex = 1001

    tabButtons[tabName] = {button = tabButton, text = tabText}
end

-- Content area
local contentStartX = windowPos.X + 10
local contentStartY = windowPos.Y + 80
local contentWidth = windowSize.X - 20
local contentHeight = windowSize.Y - 90

-- Universal tab options UI elements
local universalOptions = {}

local function createToggleOption(name, posY)
    local box = Drawing.new("Square")
    box.Size = Vector2.new(20, 20)
    box.Position = Vector2.new(contentStartX, contentStartY + posY)
    box.Color = Color3.fromRGB(70, 70, 70)
    box.Filled = true
    box.Visible = false
    box.Transparency = 0.9
    box.ZIndex = 1000

    local check = Drawing.new("Text")
    check.Text = ""
    check.Size = 18
    check.Position = box.Position + Vector2.new(3, 0)
    check.Color = Color3.fromRGB(0, 255, 0)
    check.Visible = false
    check.Center = false
    check.Outline = true
    check.OutlineColor = Color3.new(0, 0, 0)
    check.ZIndex = 1001

    local label = Drawing.new("Text")
    label.Text = name
    label.Size = 18
    label.Position = Vector2.new(box.Position.X + 30, box.Position.Y - 2)
    label.Color = Color3.fromRGB(255, 255, 255)
    label.Visible = false
    label.Center = false
    label.Outline = true
    label.OutlineColor = Color3.new(0, 0, 0)
    label.ZIndex = 1001

    return {box = box, check = check, label = label, state = false}
end

universalOptions.fly = createToggleOption("Fly", 0)
universalOptions.walkspeed = createToggleOption("Walkspeed", 30)
universalOptions.esp = createToggleOption("ESP", 60)
universalOptions.aimbot = createToggleOption("Aimbot", 90)
universalOptions.infiniteJump = createToggleOption("Infinite Jump", 120)

-- Walkspeed slider UI
local walkspeedSlider = Drawing.new("Square")
walkspeedSlider.Size = Vector2.new(150, 20)
walkspeedSlider.Position = Vector2.new(contentStartX + 100, contentStartY + 30)
walkspeedSlider.Color = Color3.fromRGB(70, 70, 70)
walkspeedSlider.Filled = true
walkspeedSlider.Visible = false
walkspeedSlider.Transparency = 0.9
walkspeedSlider.ZIndex = 1000

local walkspeedHandle = Drawing.new("Square")
walkspeedHandle.Size = Vector2.new(10, 20)
walkspeedHandle.Position = walkspeedSlider.Position
walkspeedHandle.Color = Color3.fromRGB(0, 255, 0)
walkspeedHandle.Filled = true
walkspeedHandle.Visible = false
walkspeedHandle.Transparency = 0.9
walkspeedHandle.ZIndex = 1001

local walkspeedValueText = Drawing.new("Text")
walkspeedValueText.Text = tostring(walkspeedValue)
walkspeedValueText.Size = 18
walkspeedValueText.Position = Vector2.new(walkspeedSlider.Position.X + walkspeedSlider.Size.X + 10, walkspeedSlider.Position.Y - 2)
walkspeedValueText.Color = Color3.fromRGB(255, 255, 255)
walkspeedValueText.Visible = false
walkspeedValueText.Center = false
walkspeedValueText.Outline = true
walkspeedValueText.OutlineColor = Color3.new(0, 0, 0)
walkspeedValueText.ZIndex = 1001

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

local function disableFly()
    if bodyVelocity then
        bodyVelocity:Destroy()
        bodyVelocity = nil
        notify("Fly disabled")
    end
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
        guiVisible = not guiVisible
        window.Visible = guiVisible
        title.Visible = guiVisible
        for _, tab in pairs(tabButtons) do
            tab.button.Visible = guiVisible
            tab.text.Visible = guiVisible
        end
        for _, option in pairs(universalOptions) do
            option.box.Visible = false
            option.check.Visible = false
            option.label.Visible = false
        end
        walkspeedSlider.Visible = false
        walkspeedHandle.Visible = false
        walkspeedValueText.Visible = false
        if guiVisible and currentTab == "Universal" then
            for _, option in pairs(universalOptions) do
                option.box.Visible = true
                option.check.Visible = option.state
                option.label.Visible = true
            end
            walkspeedSlider.Visible = true
            walkspeedHandle.Visible = true
            walkspeedValueText.Visible = true
        end
        notify("GUI toggled: "..tostring(guiVisible))
    end
end)

-- Tab click detection
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if guiVisible and input.UserInputType == Enum.UserInputType.MouseButton1 then
        local mousePos = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
        -- Check tabs
        for tabName, tab in pairs(tabButtons) do
            local pos = tab.button.Position
            local size = tab.button.Size
            if mousePos.X >= pos.X and mousePos.X <= pos.X + size.X and mousePos.Y >= pos.Y and mousePos.Y <= pos.Y + size.Y then
                currentTab = tabName
                -- Update tab colors
                for tn, t in pairs(tabButtons) do
                    if tn == currentTab then
                        t.button.Color = Color3.fromRGB(100, 100, 100)
                    else
                        t.button.Color = Color3.fromRGB(50, 50, 50)
                    end
                end
                -- Update visible options
                for _, option in pairs(universalOptions) do
                    option.box.Visible = false
                    option.check.Visible = false
                    option.label.Visible = false
                end
                walkspeedSlider.Visible = false
                walkspeedHandle.Visible = false
                walkspeedValueText.Visible = false
                if currentTab == "Universal" then
                    for _, option in pairs(universalOptions) do
                        option.box.Visible = true
                        option.check.Visible = option.state
                        option.label.Visible = true
                    end
                    walkspeedSlider.Visible = true
                    walkspeedHandle.Visible = true
                    walkspeedValueText.Visible = true
                end
                notify("Switched to tab: "..currentTab)
                break
            end
        end
        -- Check universal options toggle boxes
        if currentTab == "Universal" then
            for name, option in pairs(universalOptions) do
                local pos = option.box.Position
                local size = option.box.Size
                if mousePos.X >= pos.X and mousePos.X <= pos.X + size.X and mousePos.Y >= pos.Y and mousePos.Y <= pos.Y + size.Y then
                    option.state = not option.state
                    option.check.Text = option.state and "✔" or ""
                    option.check.Visible = option.state
                    -- Apply feature state changes
                    if name == "fly" then
                        flyEnabled = option.state
                        if flyEnabled then enableFly() else disableFly() end
                    elseif name == "walkspeed" then
                        walkspeedEnabled = option.state
                        if walkspeedEnabled then
                            setWalkspeed(walkspeedValue)
                        else
                            setWalkspeed(16)
                        end
                    elseif name == "esp" then
                        espEnabled = option.state
                        if espEnabled then enableESP() else disableESP() end
                    elseif name == "aimbot" then
                        aimbotEnabled = option.state
                    elseif name == "infiniteJump" then
                        infiniteJumpEnabled = option.state
                    end
                    notify(name.." toggled: "..tostring(option.state))
                    break
                end
            end
        end
        -- Check walkspeed slider drag
        if currentTab == "Universal" then
            local sliderPos = walkspeedSlider.Position
            local sliderSize = walkspeedSlider.Size
            if mousePos.X >= sliderPos.X and mousePos.X <= sliderPos.X + sliderSize.X and mousePos.Y >= sliderPos.Y and mousePos.Y <= sliderPos.Y + sliderSize.Y then
                local relativeX = mousePos.X - sliderPos.X
                local newSpeed = math.floor((relativeX / sliderSize.X) * 100)
                if newSpeed < 16 then newSpeed = 16 end
                if newSpeed > 100 then newSpeed = 100 end
                walkspeedValue = newSpeed
                walkspeedValueText.Text = tostring(walkspeedValue)
                walkspeedHandle.Position = Vector2.new(sliderPos.X + (relativeX - walkspeedHandle.Size.X/2), sliderPos.Y)
                if walkspeedEnabled then
                    setWalkspeed(walkspeedValue)
                end
                notify("Walkspeed set to "..walkspeedValue)
            end
        end
    end
end)

-- Initialize tab colors
for tn, t in pairs(tabButtons) do
    if tn == currentTab then
        t.button.Color = Color3.fromRGB(100, 100, 100)
    else
        t.button.Color = Color3.fromRGB(50, 50, 50)
    end
end

print("RLK HUB script loaded, press RightShift to toggle GUI")
