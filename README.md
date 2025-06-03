-- Gunfight Simulator Auto Script (No GUI)
-- Features: Auto Farm, ESP, Aimbot with team detection, automatic operation
-- WARNING: Use at your own risk. For educational purposes only.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- Settings
local AimbotEnabled = true
local ESPEnabled = true
local AutoFarmEnabled = true
local WallhackEnabled = true
local AutoDodgeEnabled = true -- new toggle for auto dodging
local AutoTeleportEnabled = false -- new toggle for teleporting on players (high ban risk)
local SpeedBoostValue = 16
local NormalSpeed = 16

-- Aimbot with team detection and closest to crosshair targeting
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

-- Auto farm logic placeholder
local function autoFarm()
    -- Implement game-specific auto farm logic here
    print("Auto Farm: Running auto farm logic")
end

-- Function to simulate bullet penetration (wallhack)
local function canShootThroughWalls(targetPosition)
    -- Simplified: always return true to simulate bullet penetration
    -- For more advanced, raycast ignoring walls can be implemented
    return true
end

-- Function to dodge incoming projectiles
local function autoDodge()
    local Character = LocalPlayer.Character
    if not Character then return end
    local HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
    if not HumanoidRootPart then return end

    for _, projectile in pairs(workspace:GetChildren()) do
        if projectile.Name == "Bullet" or projectile.Name == "Projectile" then
            if projectile:IsA("BasePart") then
                local distance = (projectile.Position - HumanoidRootPart.Position).Magnitude
                if distance < 20 then -- dodge if projectile is close
                    -- Move character perpendicular to projectile velocity
                    local velocity = projectile.Velocity
                    local dodgeDirection = Vector3.new(-velocity.Z, 0, velocity.X).Unit
                    local dodgePosition = HumanoidRootPart.Position + dodgeDirection * 10
                    HumanoidRootPart.CFrame = CFrame.new(dodgePosition)
                    break
                end
            end
        end
    end
end

-- Function to teleport on top of closest player (high ban risk)
local function autoTeleportOnPlayer()
    local target = getClosestTarget()
    if target and target.Character then
        local targetPart = target.Character:FindFirstChild("HumanoidRootPart") or target.Character:FindFirstChild("UpperTorso")
        if targetPart then
            local Character = LocalPlayer.Character
            if Character and Character:FindFirstChild("HumanoidRootPart") then
                Character.HumanoidRootPart.CFrame = targetPart.CFrame * CFrame.new(0, 3, 0)
            end
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
            if targetPart and canShootThroughWalls(targetPart.Position) then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPart.Position)
            end
        end
    end

    -- ESP
    if ESPEnabled then
        updateEsp()
    end

    -- Auto Dodge
    if AutoDodgeEnabled then
        autoDodge()
    end

    -- Auto Teleport (high ban risk)
    if AutoTeleportEnabled then
        autoTeleportOnPlayer()
    end

    -- Auto Farm
    if AutoFarmEnabled then
        autoFarm()
    end
end)

print("Gunfight Simulator Auto Script loaded.")
