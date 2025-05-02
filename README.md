-- Vex Hub - Universal Script
-- Versão: 2.0
-- Corrigido: GUI e funcionalidades

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

-- Funções universais
function VexHub:ToggleFly()
    VexHub.Settings.Universal.Fly = not VexHub.Settings.Universal.Fly
    
    if VexHub.Settings.Universal.Fly then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
        bodyVelocity.Parent = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        
        table.insert(VexHub.Connections, RunService.Heartbeat:Connect(function()
            if not VexHub.Settings.Universal.Fly or not LocalPlayer.Character then
                VexHub:Cleanup()
                return
            end
            
            local rootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                local newVelocity = Vector3.new(0, 0, 0)
                
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    newVelocity = newVelocity + (Camera.CFrame.LookVector * VexHub.Settings.Universal.FlySpeed)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    newVelocity = newVelocity - (Camera.CFrame.LookVector * VexHub.Settings.Universal.FlySpeed)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    newVelocity = newVelocity - (Camera.CFrame.RightVector * VexHub.Settings.Universal.FlySpeed)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    newVelocity = newVelocity + (Camera.CFrame.RightVector * VexHub.Settings.Universal.FlySpeed)
                end
                
                rootPart.Velocity = Vector3.new(newVelocity.X, rootPart.Velocity.Y, newVelocity.Z)
                
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    bodyVelocity.Velocity = Vector3.new(0, VexHub.Settings.Universal.FlySpeed, 0)
                elseif UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                    bodyVelocity.Velocity = Vector3.new(0, -VexHub.Settings.Universal.FlySpeed, 0)
                else
                    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
                end
            end
        end))
    else
        VexHub:Cleanup()
    end
end

function VexHub:ToggleNoclip()
    VexHub.Settings.Universal.Noclip = not VexHub.Settings.Universal.Noclip
    
    if VexHub.Settings.Universal.Noclip then
        table.insert(VexHub.Connections, RunService.Stepped:Connect(function()
            if not VexHub.Settings.Universal.Noclip or not LocalPlayer.Character then
                VexHub:Cleanup()
                return
            end
            
            for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end))
    else
        VexHub:Cleanup()
    end
end

function VexHub:ToggleESP()
    VexHub.Settings.Universal.ESP = not VexHub.Settings.Universal.ESP
    
    if not VexHub.Settings.Universal.ESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local highlight = player.Character:FindFirstChild("VexESP")
                if highlight then
                    highlight:Destroy()
                end
            end
        end
        return
    end
    
    local function createESP(player)
        if player == LocalPlayer then return end
        
        local character = player.Character or player.CharacterAdded:Wait()
        local highlight = Instance.new("Highlight")
        highlight.Name = "VexESP"
        highlight.FillColor = VexHub.Settings.Universal.ESPColor
        highlight.OutlineColor = VexHub.Settings.Universal.ESPColor
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.Parent = character
    end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if player.Character then
                createESP(player)
            end
            table.insert(VexHub.Connections, player.CharacterAdded:Connect(function(character)
                createESP(player)
            end))
        end
    end
    
    table.insert(VexHub.Connections, Players.PlayerAdded:Connect(function(player)
        table.insert(VexHub.Connections, player.CharacterAdded:Connect(function(character)
            createESP(player)
        end))
    end))
end

-- Funções específicas para Build A Boat For Treasure
function VexHub:ToggleBABAutoFarm()
    if game.PlaceId ~= 537413528 then
        warn("Este recurso só está disponível em Build A Boat For Treasure")
        return
    end
    
    VexHub.Settings.BuildABoat.AutoFarm = not VexHub.Settings.BuildABoat.AutoFarm
    
    if VexHub.Settings.BuildABoat.AutoFarm then
        table.insert(VexHub.Connections, RunService.Heartbeat:Connect(function()
            if not VexHub.Settings.BuildABoat.AutoFarm or not LocalPlayer.Character then
                VexHub:Cleanup()
                return
            end
            
            local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            local rootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            
            if humanoid and rootPart then
                rootPart.CFrame = rootPart.CFrame + (rootPart.CFrame.LookVector * VexHub.Settings.BuildABoat.FarmSpeed * 0.1)
                humanoid:Move(Vector3.new(0, 0, 1), true)
            end
        end))
    else
        VexHub:Cleanup()
    end
end

-- Funções específicas para Gunfight Arena
function VexHub:ToggleGunfightESP()
    if game.PlaceId ~= 14518422161 then
        warn("Este recurso só está disponível em Gunfight Arena")
        return
    end
    
    VexHub.Settings.Gunfight.ESP = not VexHub.Settings.Gunfight.ESP
    
    if not VexHub.Settings.Gunfight.ESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local highlight = player.Character:FindFirstChild("VexGunfightESP")
                if highlight then
                    highlight:Destroy()
                end
            end
        end
        return
    end
    
    local function createGunfightESP(player)
        if player == LocalPlayer then return end
        
        local character = player.Character or player.CharacterAdded:Wait()
        local highlight = Instance.new("Highlight")
        highlight.Name = "VexGunfightESP"
        
        if VexHub.Settings.Gunfight.TeamCheck and player.Team == LocalPlayer.Team then
            highlight.FillColor = Color3.fromRGB(0, 0, 255)
        else
            highlight.FillColor = Color3.fromRGB(255, 0, 0)
        end
        
        highlight.OutlineColor = highlight.FillColor
        highlight.FillTransparency = 0.3
        highlight.OutlineTransparency = 0
        highlight.Parent = character
    end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if player.Character then
                createGunfightESP(player)
            end
            table.insert(VexHub.Connections, player.CharacterAdded:Connect(function(character)
                createGunfightESP(player)
            end))
        end
    end
    
    table.insert(VexHub.Connections, Players.PlayerAdded:Connect(function(player)
        table.insert(VexHub.Connections, player.CharacterAdded:Connect(function(character)
            createGunfightESP(player)
        end))
    end))
end

function VexHub:ToggleAimbot()
    if game.PlaceId ~= 14518422161 then
        warn("Este recurso só está disponível em Gunfight Arena")
        return
    end
    
    VexHub.Settings.Gunfight.Aimbot = not VexHub.Settings.Gunfight.Aimbot
    
    if VexHub.Settings.Gunfight.Aimbot then
        table.insert(VexHub.Connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            
            if input.UserInputType == VexHub.Settings.Gunfight.AimKey then
                local closestPlayer, closestDistance = nil, math.huge
                local localRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                
                if not localRoot then return end
                
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character then
                        local character = player.Character
                        local humanoid = character:FindFirstChildOfClass("Humanoid")
                        local rootPart = character:FindFirstChild("HumanoidRootPart")
                        local head = character:FindFirstChild("Head")
                        
                        if humanoid and humanoid.Health > 0 and rootPart and head then
                            if VexHub.Settings.Gunfight.TeamCheck and player.Team == LocalPlayer.Team then
                                continue
                            end
                            
                            local screenPoint, onScreen = Camera:WorldToViewportPoint(head.Position)
                            local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
                            local magnitude = distance.Magnitude
                            
                            if onScreen and magnitude < VexHub.Settings.Gunfight.FOV and magnitude < closestDistance then
                                closestDistance = magnitude
                                closestPlayer = player
                            end
                        end
                    end
                end
                
                if closestPlayer then
                    local targetPart = closestPlayer.Character:FindFirstChild(VexHub.Settings.Gunfight.AimPart) or closestPlayer.Character:FindFirstChild("Head") or closestPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if targetPart then
                        local aimPosition = targetPart.Position
                        if VexHub.Settings.Gunfight.Smoothness > 0 then
                            local currentLook = Camera.CFrame.LookVector
                            local targetLook = (aimPosition - Camera.CFrame.Position).Unit
                            local smoothedLook = currentLook:Lerp(targetLook, VexHub.Settings.Gunfight.Smoothness)
                            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + smoothedLook)
                        else
                            Camera.CFrame = CFrame.new(Camera.CFrame.Position, aimPosition)
                        end
                    end
                end
            end
        end))
    else
        VexHub:Cleanup()
    end
end

function VexHub:IncreaseGunRange()
    if game.PlaceId ~= 14518422161 then
        warn("Este recurso só está disponível em Gunfight Arena")
        return
    end
    
    VexHub.Settings.Gunfight.IncreaseRange = not VexHub.Settings.Gunfight.IncreaseRange
    
    if VexHub.Settings.Gunfight.IncreaseRange then
        for _, tool in pairs(LocalPlayer.Character:GetChildren()) do
            if tool:IsA("Tool") then
                for _, v in pairs(tool:GetDescendants()) do
                    if v:IsA("NumberValue") and (v.Name:lower():find("range") or v.Name:lower():find("distance")) then
                        v.Value = VexHub.Settings.Gunfight.NewRange
                    end
                end
            end
        end
        
        table.insert(VexHub.Connections, LocalPlayer.CharacterAdded:Connect(function(character)
            table.insert(VexHub.Connections, character.ChildAdded:Connect(function(tool)
                if tool:IsA("Tool") then
                    for _, v in pairs(tool:GetDescendants()) do
                        if v:IsA("NumberValue") and (v.Name:lower():find("range") or v.Name:lower():find("distance")) then
                            v.Value = VexHub.Settings.Gunfight.NewRange
                        end
                    end
                end
            end))
        end))
    end
end

-- Interface do usuário
function VexHub:CreateUI()
    if not game:GetService("CoreGui"):FindFirstChild("VexHubUI") then
        local ScreenGui = Instance.new("ScreenGui")
        ScreenGui.Name = "VexHubUI"
        ScreenGui.Parent = game:GetService("CoreGui")
        ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

        local MainFrame = Instance.new("Frame")
        MainFrame.Name = "MainFrame"
        MainFrame.Size = UDim2.new(0, 350, 0, 400)
        MainFrame.Position = UDim2.new(0.5, -175, 0.5, -200)
        MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
        MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        MainFrame.BorderSizePixel = 0
        MainFrame.Parent = ScreenGui

        local Title = Instance.new("TextLabel")
        Title.Name = "Title"
        Title.Text = "Vex Hub"
        Title.Size = UDim2.new(1, 0, 0, 40)
        Title.Position = UDim2.new(0, 0, 0, 0)
        Title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        Title.TextColor3 = Color3.fromRGB(255, 255, 255)
        Title.Font = Enum.Font.GothamBold
        Title.TextSize = 18
        Title.Parent = MainFrame

        local TabButtons = Instance.new("Frame")
        TabButtons.Name = "TabButtons"
        TabButtons.Size = UDim2.new(1, 0, 0, 30)
        TabButtons.Position = UDim2.new(0, 0, 0, 40)
        TabButtons.BackgroundTransparency = 1
        TabButtons.Parent = MainFrame

        local UniversalTab = Instance.new("TextButton")
        UniversalTab.Name = "UniversalTab"
        UniversalTab.Text = "Universal"
        UniversalTab.Size = UDim2.new(0.33, 0, 1, 0)
        UniversalTab.Position = UDim2.new(0, 0, 0, 0)
        UniversalTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        UniversalTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        UniversalTab.Font = Enum.Font.Gotham
        UniversalTab.TextSize = 14
        UniversalTab.Parent = TabButtons

        local BABTab = Instance.new("TextButton")
        BABTab.Name = "BABTab"
        BABTab.Text = "Build A Boat"
        BABTab.Size = UDim2.new(0.34, 0, 1, 0)
        BABTab.Position = UDim2.new(0.33, 0, 0, 0)
        BABTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        BABTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        BABTab.Font = Enum.Font.Gotham
        BABTab.TextSize = 14
        BABTab.Parent = TabButtons

        local GunfightTab = Instance.new("TextButton")
        GunfightTab.Name = "GunfightTab"
        GunfightTab.Text = "Gunfight"
        GunfightTab.Size = UDim2.new(0.33, 0, 1, 0)
        GunfightTab.Position = UDim2.new(0.67, 0, 0, 0)
        GunfightTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        GunfightTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        GunfightTab.Font = Enum.Font.Gotham
        GunfightTab.TextSize = 14
        GunfightTab.Parent = TabButtons

        local TabContent = Instance.new("Frame")
        TabContent.Name = "TabContent"
        TabContent.Size = UDim2.new(1, 0, 1, -70)
        TabContent.Position = UDim2.new(0, 0, 0, 70)
        TabContent.BackgroundTransparency = 1
        TabContent.Parent = MainFrame

        -- Conteúdo da aba Universal
        local UniversalContent = Instance.new("ScrollingFrame")
        UniversalContent.Name = "UniversalContent"
        UniversalContent.Size = UDim2.new(1, 0, 1, 0)
        UniversalContent.Position = UDim2.new(0, 0, 0, 0)
        UniversalContent.BackgroundTransparency = 1
        UniversalContent.ScrollBarThickness = 5
        UniversalContent.AutomaticCanvasSize = Enum.AutomaticSize.Y
        UniversalContent.Visible = true
        UniversalContent.Parent = TabContent

        local FlyToggle = Instance.new("TextButton")
        FlyToggle.Name = "FlyToggle"
        FlyToggle.Text = "Fly: OFF"
        FlyToggle.Size = UDim2.new(0.9, 0, 0, 40)
        FlyToggle.Position = UDim2.new(0.05, 0, 0, 10)
        FlyToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        FlyToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        FlyToggle.Font = Enum.Font.Gotham
        FlyToggle.TextSize = 14
        FlyToggle.Parent = UniversalContent

        local NoclipToggle = Instance.new("TextButton")
        NoclipToggle.Name = "NoclipToggle"
        NoclipToggle.Text = "Noclip: OFF"
        NoclipToggle.Size = UDim2.new(0.9, 0, 0, 40)
        NoclipToggle.Position = UDim2.new(0.05, 0, 0, 60)
        NoclipToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        NoclipToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        NoclipToggle.Font = Enum.Font.Gotham
        NoclipToggle.TextSize = 14
        NoclipToggle.Parent = UniversalContent

        local ESPToggle = Instance.new("TextButton")
        ESPToggle.Name = "ESPToggle"
        ESPToggle.Text = "ESP: OFF"
        ESPToggle.Size = UDim2.new(0.9, 0, 0, 40)
        ESPToggle.Position = UDim2.new(0.05, 0, 0, 110)
        ESPToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        ESPToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        ESPToggle.Font = Enum.Font.Gotham
        ESPToggle.TextSize = 14
        ESPToggle.Parent = UniversalContent

        -- Conteúdo da aba Build A Boat
        local BABContent = Instance.new("ScrollingFrame")
        BABContent.Name = "BABContent"
        BABContent.Size = UDim2.new(1, 0, 1, 0)
        BABContent.Position = UDim2.new(0, 0, 0, 0)
        BABContent.BackgroundTransparency = 1
        BABContent.ScrollBarThickness = 5
        BABContent.AutomaticCanvasSize = Enum.AutomaticSize.Y
        BABContent.Visible = false
        BABContent.Parent = TabContent

        local AutoFarmToggle = Instance.new("TextButton")
        AutoFarmToggle.Name = "AutoFarmToggle"
        AutoFarmToggle.Text = "Auto Farm: OFF"
        AutoFarmToggle.Size = UDim2.new(0.9, 0, 0, 40)
        AutoFarmToggle.Position = UDim2.new(0.05, 0, 0, 10)
        AutoFarmToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        AutoFarmToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        AutoFarmToggle.Font = Enum.Font.Gotham
        AutoFarmToggle.TextSize = 14
        AutoFarmToggle.Parent = BABContent

        -- Conteúdo da aba Gunfight
        local GunfightContent = Instance.new("ScrollingFrame")
        GunfightContent.Name = "GunfightContent"
        GunfightContent.Size = UDim2.new(1, 0, 1, 0)
        GunfightContent.Position = UDim2.new(0, 0, 0, 0)
        GunfightContent.BackgroundTransparency = 1
        GunfightContent.ScrollBarThickness = 5
        GunfightContent.AutomaticCanvasSize = Enum.AutomaticSize.Y
        GunfightContent.Visible = false
        GunfightContent.Parent = TabContent

        local GunfightESPToggle = Instance.new("TextButton")
        GunfightESPToggle.Name = "GunfightESPToggle"
        GunfightESPToggle.Text = "ESP: OFF"
        GunfightESPToggle.Size = UDim2.new(0.9, 0, 0, 40)
        GunfightESPToggle.Position = UDim2.new(0.05, 0, 0, 10)
        GunfightESPToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        GunfightESPToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        GunfightESPToggle.Font = Enum.Font.Gotham
        GunfightESPToggle.TextSize = 14
        GunfightESPToggle.Parent = GunfightContent

        local TeamCheckToggle = Instance.new("TextButton")
        TeamCheckToggle.Name = "TeamCheckToggle"
        TeamCheckToggle.Text = "Team Check: ON"
        TeamCheckToggle.Size = UDim2.new(0.9, 0, 0, 40)
        TeamCheckToggle.Position = UDim2.new(0.05, 0, 0, 60)
        TeamCheckToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        TeamCheckToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        TeamCheckToggle.Font = Enum.Font.Gotham
        TeamCheckToggle.TextSize = 14
        TeamCheckToggle.Parent = GunfightContent

        local AimbotToggle = Instance.new("TextButton")
        AimbotToggle.Name = "AimbotToggle"
        AimbotToggle.Text = "Aimbot: OFF"
        AimbotToggle.Size = UDim2.new(0.9, 0, 0, 40)
        AimbotToggle.Position = UDim2.new(0.05, 0, 0, 110)
        AimbotToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        AimbotToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        AimbotToggle.Font = Enum.Font.Gotham
        AimbotToggle.TextSize = 14
        AimbotToggle.Parent = GunfightContent

        local IncreaseRangeToggle = Instance.new("TextButton")
        IncreaseRangeToggle.Name = "IncreaseRangeToggle"
        IncreaseRangeToggle.Text = "Increase Range: OFF"
        IncreaseRangeToggle.Size = UDim2.new(0.9, 0, 0, 40)
        IncreaseRangeToggle.Position = UDim2.new(0.05, 0, 0, 160)
        IncreaseRangeToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        IncreaseRangeToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        IncreaseRangeToggle.Font = Enum.Font.Gotham
        IncreaseRangeToggle.TextSize = 14
        IncreaseRangeToggle.Parent = GunfightContent

        -- Conectando os botões
        FlyToggle.MouseButton1Click:Connect(function()
            VexHub:ToggleFly()
            FlyToggle.Text = "Fly: " .. (VexHub.Settings.Universal.Fly and "ON" or "OFF")
        end)
        
        NoclipToggle.MouseButton1Click:Connect(function()
            VexHub:ToggleNoclip()
            NoclipToggle.Text = "Noclip: " .. (VexHub.Settings.Universal.Noclip and "ON" or "OFF")
        end)
        
        ESPToggle.MouseButton1Click:Connect(function()
            VexHub:ToggleESP()
            ESPToggle.Text = "ESP: " .. (VexHub.Settings.Universal.ESP and "ON" or "OFF")
        end)
        
        AutoFarmToggle.MouseButton1Click:Connect(function()
            VexHub:ToggleBABAutoFarm()
            AutoFarmToggle.Text = "Auto Farm: " .. (VexHub.Settings.BuildABoat.AutoFarm and "ON" or "OFF")
        end)
        
        GunfightESPToggle.MouseButton1Click:Connect(function()
            VexHub:ToggleGunfightESP()
            GunfightESPToggle.Text = "ESP: " .. (VexHub.Settings.Gunfight.ESP and "ON" or "OFF")
        end)
        
        TeamCheckToggle.MouseButton1Click:Connect(function()
            VexHub.Settings.Gunfight.TeamCheck = not VexHub.Settings.Gunfight.TeamCheck
            TeamCheckToggle.Text = "Team Check: " .. (VexHub.Settings.Gunfight.TeamCheck and "ON" or "OFF")
        end)
        
        AimbotToggle.MouseButton1Click:Connect(function()
            VexHub:ToggleAimbot()
            AimbotToggle.Text = "Aimbot: " .. (VexHub.Settings.Gunfight.Aimbot and "ON" or "OFF")
        end)
        
        IncreaseRangeToggle.MouseButton1Click:Connect(function()
            VexHub:IncreaseGunRange()
            IncreaseRangeToggle.Text = "Increase Range: " .. (VexHub.Settings.Gunfight.IncreaseRange and "ON" or "OFF")
        end)
        
        -- Lógica de troca de abas
        UniversalTab.MouseButton1Click:Connect(function()
            UniversalContent.Visible = true
            BABContent.Visible = false
            GunfightContent.Visible = false
            
            UniversalTab.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
            BABTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            GunfightTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        end)
        
        BABTab.MouseButton1Click:Connect(function()
            UniversalContent.Visible = false
            BABContent.Visible = true
            GunfightContent.Visible = false
            
            UniversalTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            BABTab.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
            GunfightTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        end)
        
        GunfightTab.MouseButton1Click:Connect(function()
            UniversalContent.Visible = false
            BABContent.Visible = false
            GunfightContent.Visible = true
            
            UniversalTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            BABTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            GunfightTab.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        end)
        
        -- Tornar a janela arrastável
        local dragging
        local dragInput
        local dragStart
        local startPos
        
        local function update(input)
            local delta = input.Position - dragStart
            MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
        
        Title.InputBegan:Connect(function(input)
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
        
        Title.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement then
                dragInput = input
            end
        end)
        
        UserInputService.InputChanged:Connect(function(input)
            if input == dragInput and dragging then
                update(input)
            end
        end)
        
        -- Fechar GUI quando o script for reiniciado
        table.insert(VexHub.Connections, game:GetService("CoreGui").ChildRemoved:Connect(function(child)
            if child.Name == "VexHubUI" then
                VexHub:Cleanup()
            end
        end))
    end
end

-- Inicialização
VexHub:CreateUI()

return VexHub
