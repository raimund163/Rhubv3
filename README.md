-- [[ SERVIÇOS DO ROBLOX ]] --
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- [[ CONFIGURAÇÕES GERAIS (ESTADO DA GUI) ]] --
local Settings = {
    AimbotEnabled = false,
    TeamCheck = false,
    WallCheck = false,
    Prediction = false,
    FovRadius = 100,
    TargetPart = "Head", -- Head, Torso/UpperTorso, LeftFoot
    
    EspEnabled = false,
    EspBox = false,
    EspHealth = false,
    RigType = "R15"
}

-- [[ CRIAÇÃO DA GUI ]] --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomCheatGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Tenta colocar na CoreGui para segurança, se não conseguir vai para PlayerGui
local success, err = pcall(function()
    ScreenGui.Parent = game:GetService("CoreGui")
end)
if not success then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Botão de Abrir / Fechar
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 120, 0, 40)
ToggleButton.Position = UDim2.new(0, 20, 0, 20)
ToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Text = "Menu: ON"
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 16
ToggleButton.Parent = ScreenGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 550, 0, 350)
MainFrame.Position = UDim2.new(0.5, -275, 0.5, -175)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

-- Sistema de Arrastar a GUI
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)
MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

ToggleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
    ToggleButton.Text = MainFrame.Visible and "Menu: ON" or "Menu: OFF"
end)

-- Painéis da GUI
local LeftPanel = Instance.new("Frame")
LeftPanel.Size = UDim2.new(0, 150, 1, 0)
LeftPanel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
LeftPanel.BorderSizePixel = 0
LeftPanel.Parent = MainFrame

local RightPanel = Instance.new("Frame")
RightPanel.Size = UDim2.new(1, -150, 1, 0)
RightPanel.Position = UDim2.new(0, 150, 0, 0)
RightPanel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
RightPanel.BorderSizePixel = 0
RightPanel.Parent = MainFrame

-- Avatar de Noob na Direita
local NoobAvatar = Instance.new("ImageLabel")
NoobAvatar.Size = UDim2.new(0, 90, 0, 90)
NoobAvatar.Position = UDim2.new(1, -110, 0, 20)
NoobAvatar.BackgroundTransparency = 1
NoobAvatar.Image = "rbxassetid://134149021"
NoobAvatar.Parent = RightPanel

local SectionTitle = Instance.new("TextLabel")
SectionTitle.Size = UDim2.new(1, -20, 0, 30)
SectionTitle.Position = UDim2.new(0, 10, 0, 10)
SectionTitle.BackgroundTransparency = 1
SectionTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
SectionTitle.Text = "Configurações Globais"
SectionTitle.Font = Enum.Font.SourceSansBold
SectionTitle.TextSize = 18
SectionTitle.TextAlign = Enum.TextAlign.Left
SectionTitle.Parent = RightPanel

-- [[ DRAWING API (FOV E LINHA) ]] --
local FovCircle = Drawing.new("Circle")
FovCircle.Color = Color3.fromRGB(255, 0, 0)
FovCircle.Thickness = 1
FovCircle.NumSides = 64
FovCircle.Radius = Settings.FovRadius
FovCircle.Filled = false
FovCircle.Visible = true

local SnapLine = Drawing.new("Line")
SnapLine.Color = Color3.fromRGB(255, 255, 0)
SnapLine.Thickness = 1.5
SnapLine.Visible = false

-- [[ LÓGICA DE ALVO DO AIMBOT ]] --
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            -- Valida a parte do corpo com base no Rig
            local partName = Settings.TargetPart
            if partName == "Torso" and player.Character:FindFirstChild("UpperTorso") then
                partName = "UpperTorso"
            end
            
            local targetPart = player.Character:FindFirstChild(partName)
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            
            if targetPart and humanoid and humanoid.Health > 0 then
                if Settings.TeamCheck and player.Team == LocalPlayer.Team then continue end
                
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                
                if onScreen then
                    if Settings.WallCheck then
                        local raycastParams = RaycastParams.new()
                        raycastParams.FilterType = Enum.RaycastFilterType.Exclude
                        raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, player.Character}
                        local raycastResult = workspace:Raycast(Camera.CFrame.Position, targetPart.Position - Camera.CFrame.Position, raycastParams)
                        if raycastResult then continue end 
                    end
                    
                    local mousePos = UserInputService:GetMouseLocation()
                    local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    
                    if distance < shortestDistance and distance <= Settings.FovRadius then
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- Loop do Aimbot e Trava de Mira (Botão Direito do Mouse)
RunService.RenderStepped:Connect(function()
    local mousePos = UserInputService:GetMouseLocation()
    FovCircle.Position = mousePos
    FovCircle.Radius = Settings.FovRadius
    
    if Settings.AimbotEnabled then
        local target = GetClosestPlayer()
        if target and target.Character then
            local partName = Settings.TargetPart
            if partName == "Torso" and target.Character:FindFirstChild("UpperTorso") then partName = "UpperTorso" end
            
            local part = target.Character:FindFirstChild(partName)
            if part then
                local targetPosition = part.Position
                
                if Settings.Prediction and target.Character:FindFirstChild("HumanoidRootPart") then
                    targetPosition = targetPosition + (target.Character.HumanoidRootPart.AssemblyLinearVelocity * 0.145)
                end
                
                -- Se segurar o botão direito do mouse, trava a câmera
                if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPosition)
                end
                
                -- Linha saindo do fov até a cabeça/alvo (Apenas se o inimigo estiver visível no wall check)
                local screenPos, _ = Camera:WorldToViewportPoint(targetPosition)
                SnapLine.From = mousePos
                SnapLine.To = Vector2.new(screenPos.X, screenPos.Y)
                SnapLine.Visible = true
            else
                SnapLine.Visible = false
            end
        else
            SnapLine.Visible = false
        end
    else
        SnapLine.Visible = false
    end
end)

-- [[ SISTEMA ESP COM BOX DINÂMICA ]] --
local function CreateEsp(player)
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Color = Color3.fromRGB(255, 0, 0)
    Box.Thickness = 15 -- Espessura visível
    Box.Filled = false

    local HealthText = Drawing.new("Text")
    HealthText.Visible = false
    HealthText.Color = Color3.fromRGB(0, 255, 0)
    HealthText.Size = 14
    HealthText.Center = false
    HealthText.Outline = true

    local connection
    connection = RunService.RenderStepped:Connect(function()
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChildOfClass("Humanoid") and player.Character:FindFirstChildOfClass("Humanoid").Health > 0 and player ~= LocalPlayer then
            if Settings.EspEnabled then
                local hrp = player.Character.HumanoidRootPart
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                
                if onScreen then
                    -- Tamanho baseado na distância (Perto, Médio, Longe)
                    local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
                    local sizeModifier = math.clamp(1000 / distance, 5, 150)
                    local boxWidth = 40 * sizeModifier
                    local boxHeight = 60 * sizeModifier

                    if Settings.EspBox then
                        Box.Size = Vector2.new(boxWidth, boxHeight)
                        Box.Position = Vector2.new(screenPos.X - boxWidth / 2, screenPos.Y - boxHeight / 2)
                        Box.Visible = true
                    else
                        Box.Visible = false
                    end

                    if Settings.EspHealth then
                        HealthText.Text = "HP: " .. math.floor(player.Character:FindFirstChildOfClass("Humanoid").Health)
                        HealthText.Position = Vector2.new(screenPos.X - (boxWidth / 2) - 45, screenPos.Y - (boxHeight / 2))
                        HealthText.Visible = true
                    else
                        HealthText.Visible = false
                    end
                else
                    Box.Visible = false
                    HealthText.Visible = false
                end
            else
                Box.Visible = false
                HealthText.Visible = false
            end
        else
            Box.Visible = false
            HealthText.Visible = false
            if not player.Parent then
                Box:Remove()
                HealthText:Remove()
                connection:Disconnect()
            end
        end
    end)
end

Players.PlayerAdded:Connect(CreateEsp)
for _, p in pairs(Players:GetPlayers()) do CreateEsp(p) end

-- [[ INTERATIVIDADE DOS BOTÕES ]] --
local function CreateSubButton(text, pos, parent, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 130, 0, 30)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Text = text
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 14
    btn.Parent = parent
    btn.MouseButton1Click:Connect(function()
        callback(btn)
    end)
    return btn
end

-- Alternar Abas
CreateSubButton("Aimbot Menu", UDim2.new(0, 10, 0, 20), LeftPanel, function() SectionTitle.Text = "Configurações de Aimbot" end)
CreateSubButton("ESP Menu", UDim2.new(0, 10, 0, 60), LeftPanel, function() SectionTitle.Text = "Configurações de Visuais (ESP)" end)

-- Botões de Função (Aimbot)
local AimBtn = CreateSubButton("Aimbot: OFF", UDim2.new(0, 10, 0, 50), RightPanel, function(b)
    Settings.AimbotEnabled = not Settings.AimbotEnabled
    b.Text = Settings.AimbotEnabled and "Aimbot: ON" or "Aimbot: OFF"
end)

local TeamBtn = CreateSubButton("Team Check: OFF", UDim2.new(0, 10, 0, 90), RightPanel, function(b)
    Settings.TeamCheck = not Settings.TeamCheck
    b.Text = Settings.TeamCheck and "Team Check: ON" or "Team Check: OFF"
end)

local WallBtn = CreateSubButton("Wall Check: OFF", UDim2.new(0, 10, 0, 130), RightPanel, function(b)
    Settings.WallCheck = not Settings.WallCheck
    b.Text = Settings.WallCheck and "Wall Check: ON" or "Wall Check: OFF"
end)

local PredBtn = CreateSubButton("Previsão: OFF", UDim2.new(0, 10, 0, 170), RightPanel, function(b)
    Settings.Prediction = not Settings.Prediction
    b.Text = Settings.Prediction and "Previsão: ON" or "Previsão: OFF"
end)

-- Modificadores de FOV (+ e -)
local FovLabel = Instance.new("TextLabel")
FovLabel.Size = UDim2.new(0, 100, 0, 30)
FovLabel.Position = UDim2.new(0, 10, 0, 210)
FovLabel.Text = "FOV: " .. Settings.FovRadius
FovLabel.TextColor3 = Color3.fromRGB(255,255,255)
FovLabel.BackgroundTransparency = 1
FovLabel.Parent = RightPanel

CreateSubButton("+", UDim2.new(0, 115, 0, 210), RightPanel, function()
    Settings.FovRadius = Settings.FovRadius + 15
    FovLabel.Text = "FOV: " .. Settings.FovRadius
end)

CreateSubButton("-", UDim2.new(0, 150, 0, 210), RightPanel, function()
    if Settings.FovRadius > 15 then
        Settings.FovRadius = Settings.FovRadius - 15
        FovLabel.Text = "FOV: " .. Settings.FovRadius
    end
end)

-- Seletor de Osso Alvo
local BoneBtn = CreateSubButton("Alvo: Cabeça", UDim2.new(0, 10, 0, 250), RightPanel, function(b)
    if Settings.TargetPart == "Head" then
        Settings.TargetPart = "Torso"
        b.Text = "Alvo: Tronco"
    elseif Settings.TargetPart == "Torso" then
        Settings.TargetPart = "LeftFoot"
        b.Text = "Alvo: Pé"
    else
        Settings.TargetPart = "Head"
        b.Text = "Alvo: Cabeça"
    end
end)

-- Botões do ESP
local EspBtn = CreateSubButton("Ativar ESP: OFF", UDim2.new(0, 200, 0, 250), RightPanel, function(b)
    Settings.EspEnabled = not Settings.EspEnabled
    Settings.EspBox = Settings.EspEnabled
    Settings.EspHealth = Settings.EspEnabled
    b.Text = Settings.EspEnabled and "Ativar ESP: ON" or "Ativar ESP: OFF"
end)

local RigBtn = CreateSubButton("Rig: R15", UDim2.new(0, 200, 0, 210), RightPanel, function(b)
    if Settings.RigType == "R15" then
        Settings.RigType = "R6"
        b.Text = "Rig: R6"
    else
        Settings.RigType = "R15"
        b.Text = "Rig: R15"
    end
end)
