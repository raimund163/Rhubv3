-- [[ SERVIÇOS DO ROBLOX ]] --
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- [[ DEFINIÇÃO DE CORES E ESTILOS ]] --
local UI_THEME = {
    Background = Color3.fromRGB(18, 18, 20),
    Sidebar = Color3.fromRGB(24, 24, 28),
    Accent = Color3.fromRGB(0, 162, 255),
    AccentHover = Color3.fromRGB(0, 130, 200),
    TextActive = Color3.fromRGB(255, 255, 255),
    TextMuted = Color3.fromRGB(150, 150, 160),
    ButtonBg = Color3.fromRGB(32, 32, 38),
    ButtonHover = Color3.fromRGB(40, 40, 48),
    Border = Color3.fromRGB(40, 40, 45)
}

-- [[ CONFIGURAÇÕES GERAIS ]] --
local Settings = {
    AimbotEnabled = false,
    TeamCheck = false,
    WallCheck = false,
    Prediction = false,
    PredictionFactor = 0.145, -- Tempo estimado de projétil/ping para cálculo físico
    FovRadius = 100,
    TargetPart = "Head", -- "Head" (Cabeça), "Torso" (Tronco), "LeftFoot" (Pé)
    
    EspEnabled = false,
    EspBox = false,
    EspHealth = false,
    EspSkeleton = false,
    RigType = "R15" -- "R15" ou "R6"
}

-- [[ CRIAÇÃO DA ESTRUTURA DA GUI ]] --
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PremiumCheatGUI_V2"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Injeção segura na CoreGui ou PlayerGui
local success, _ = pcall(function()
    ScreenGui.Parent = CoreGui
end)
if not success then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Botão Flutuante para Abrir/Fechar a Interface
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "MenuToggle"
ToggleButton.Size = UDim2.new(0, 140, 0, 42)
ToggleButton.Position = UDim2.new(0, 25, 0, 25)
ToggleButton.BackgroundColor3 = UI_THEME.Sidebar
ToggleButton.BorderSizePixel = 0
ToggleButton.Text = "Menu: Ativo"
ToggleButton.TextColor3 = UI_THEME.TextActive
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 15
ToggleButton.Parent = ScreenGui

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 8)
ToggleCorner.Parent = ToggleButton

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 580, 0, 380)
MainFrame.Position = UDim2.new(0.5, -290, 0.5, -190)
MainFrame.BackgroundColor3 = UI_THEME.Background
MainFrame.BorderSizePixel = 0
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

-- Sistema de Arrastar com Efeito Suave (Smooth Drag)
local dragging, dragInput, dragStart, startPos
local function updateDrag(input)
    local delta = input.Position - dragStart
    local targetPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    TweenService:Create(MainFrame, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = targetPos}):Play()
end

MainFrame.InputBegan:Connect(function(input)
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

MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        updateDrag(input)
    end
end)

ToggleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
    ToggleButton.Text = MainFrame.Visible and "Menu: Ativo" or "Menu: Oculto"
end)

-- Painel Lateral Esquerdo (Menu de Navegação)
local Sidebar = Instance.new("Frame")
Sidebar.Name = "Sidebar"
Sidebar.Size = UDim2.new(0, 160, 1, 0)
Sidebar.BackgroundColor3 = UI_THEME.Sidebar
Sidebar.BorderSizePixel = 0
Sidebar.Parent = MainFrame

local SidebarCorner = Instance.new("UICorner")
SidebarCorner.CornerRadius = UDim.new(0, 10)
SidebarCorner.Parent = Sidebar

-- Obstruir cantos direitos do painel lateral para manter o design clean
local SidebarPatch = Instance.new("Frame")
SidebarPatch.Size = UDim2.new(0, 10, 1, 0)
SidebarPatch.Position = UDim2.new(1, -10, 0, 0)
SidebarPatch.BackgroundColor3 = UI_THEME.Sidebar
SidebarPatch.BorderSizePixel = 0
SidebarPatch.Parent = Sidebar

-- Logo / Título do Menu
local LogoLabel = Instance.new("TextLabel")
LogoLabel.Size = UDim2.new(1, 0, 0, 50)
LogoLabel.BackgroundTransparency = 1
LogoLabel.Text = "PREMIUM CHEAT"
LogoLabel.TextColor3 = UI_THEME.Accent
LogoLabel.Font = Enum.Font.SourceSansBold
LogoLabel.TextSize = 18
LogoLabel.Parent = Sidebar

-- Painel Conteúdo Direito
local ContentFrame = Instance.new("Frame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -160, 1, 0)
ContentFrame.Position = UDim2.new(0, 160, 0, 0)
ContentFrame.BackgroundTransparency = 1
ContentFrame.Parent = MainFrame

-- Avatar de Noob Estilizado
local NoobContainer = Instance.new("Frame")
NoobContainer.Size = UDim2.new(0, 110, 0, 110)
NoobContainer.Position = UDim2.new(1, -130, 0, 20)
NoobContainer.BackgroundColor3 = UI_THEME.Sidebar
NoobContainer.BorderSizePixel = 0
NoobContainer.Parent = ContentFrame

local NoobCorner = Instance.new("UICorner")
NoobCorner.CornerRadius = UDim.new(0, 12)
NoobCorner.Parent = NoobContainer

local NoobAvatar = Instance.new("ImageLabel")
NoobAvatar.Size = UDim2.new(1, -10, 1, -10)
NoobAvatar.Position = UDim2.new(0, 5, 0, 5)
NoobAvatar.BackgroundTransparency = 1
NoobAvatar.Image = "rbxassetid://134149021" -- ID de Recurso Clássico Noob do Roblox
NoobAvatar.Parent = NoobContainer

-- Título da Página de Configuração Dinâmica
local PageTitle = Instance.new("TextLabel")
PageTitle.Size = UDim2.new(1, -160, 0, 40)
PageTitle.Position = UDim2.new(0, 20, 0, 15)
PageTitle.BackgroundTransparency = 1
PageTitle.Text = "Menu Principal"
PageTitle.TextColor3 = UI_THEME.TextActive
PageTitle.Font = Enum.Font.SourceSansBold
PageTitle.TextSize = 22
PageTitle.TextAlign = Enum.TextAlign.Left
PageTitle.Parent = ContentFrame

-- Containers para Alternar Abas (Aimbot / ESP)
local AimbotPage = Instance.new("Frame")
AimbotPage.Size = UDim2.new(1, 0, 1, -60)
AimbotPage.Position = UDim2.new(0, 0, 0, 60)
AimbotPage.BackgroundTransparency = 1
AimbotPage.Visible = true
AimbotPage.Parent = ContentFrame

local EspPage = Instance.new("Frame")
EspPage.Size = UDim2.new(1, 0, 1, -60)
EspPage.Position = UDim2.new(0, 0, 0, 60)
EspPage.BackgroundTransparency = 1
EspPage.Visible = false
EspPage.Parent = ContentFrame

-- [[ DRAWING API (VETORES DO FOV E SNAPLINE) ]] --
local FovCircle = Drawing.new("Circle")
FovCircle.Color = Color3.fromRGB(0, 162, 255)
FovCircle.Thickness = 1.2
FovCircle.NumSides = 64
FovCircle.Radius = Settings.FovRadius
FovCircle.Filled = false
FovCircle.Visible = true

local SnapLine = Drawing.new("Line")
SnapLine.Color = Color3.fromRGB(255, 235, 59)
SnapLine.Thickness = 1.5
SnapLine.Visible = false

-- [[ MÓDULO MATEMÁTICO DO AIMBOT ]] --
local function GetClosestTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            -- Mapeamento inteligente de esqueleto R15/R6
            local partName = Settings.TargetPart
            if partName == "Torso" then
                partName = player.Character:FindFirstChild("UpperTorso") and "UpperTorso" or "Torso"
            elseif partName == "LeftFoot" then
                partName = player.Character:FindFirstChild("LeftFoot") and "LeftFoot" or "Left Leg"
            end

            local targetPart = player.Character:FindFirstChild(partName)
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")

            if targetPart and humanoid and humanoid.Health > 0 then
                -- Verificação de Equipa (Team Check)
                if Settings.TeamCheck and player.Team == LocalPlayer.Team then continue end

                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)

                if onScreen then
                    -- Verificação de Linha de Visão / Paredes (Wall Check via Raycast)
                    if Settings.WallCheck then
                        local raycastParams = RaycastParams.new()
                        raycastParams.FilterType = Enum.RaycastFilterType.Exclude
                        raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, player.Character}
                        
                        local raycastResult = workspace:Raycast(
                            Camera.CFrame.Position, 
                            targetPart.Position - Camera.CFrame.Position, 
                            raycastParams
                        )
                        if raycastResult then continue end -- Bloqueado por obstáculo
                    end

                    -- Distância Bidimensional em relação ao Cursor
                    local mousePos = UserInputService:GetMouseLocation()
                    local distanceToCursor = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude

                    if distanceToCursor < shortestDistance and distanceToCursor <= Settings.FovRadius then
                        closestPlayer = player
                        shortestDistance = distanceToCursor
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- Loop de Atualização do Aimbot e Mecânica de Linha de Fov
RunService.RenderStepped:Connect(function()
    local mousePos = UserInputService:GetMouseLocation()
    FovCircle.Position = mousePos
    FovCircle.Radius = Settings.FovRadius

    if Settings.AimbotEnabled then
        local target = GetClosestTarget()
        if target and target.Character then
            local partName = Settings.TargetPart
            if partName == "Torso" then
                partName = target.Character:FindFirstChild("UpperTorso") and "UpperTorso" or "Torso"
            elseif partName == "LeftFoot" then
                partName = target.Character:FindFirstChild("LeftFoot") and "LeftFoot" or "Left Leg"
            end

            local part = target.Character:FindFirstChild(partName)
            if part then
                local targetPosition = part.Position

                -- Previsão Física de Movimento (Prediction Engine)
                if Settings.Prediction and target.Character:FindFirstChild("HumanoidRootPart") then
                    local velocity = target.Character.HumanoidRootPart.AssemblyLinearVelocity
                    targetPosition = targetPosition + (velocity * Settings.PredictionFactor)
                end

                -- Travamento Automático da Mira (Ao manter pressionado o Botão Direito do Mouse)
                if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPosition)
                end

                -- Linha de Tração Dinâmica do FOV até o Alvo (Snapline)
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

-- [[ SISTEMA AVANÇADO DE VISUAIS (ESP ENGINE) ]] --
local function CreatePlayerESP(player)
    -- Elementos do Desenho
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Color = Color3.fromRGB(255, 40, 40)
    Box.Thickness = 1.5
    Box.Filled = false

    local HealthBar = Drawing.new("Line")
    HealthBar.Visible = false
    HealthBar.Color = Color3.fromRGB(0, 255, 100)
    HealthBar.Thickness = 2.5

    local HealthText = Drawing.new("Text")
    HealthText.Visible = false
    HealthText.Color = Color3.fromRGB(255, 255, 255)
    HealthText.Size = 13
    HealthText.Center = true
    HealthText.Outline = true

    -- Mapeamento de Esqueletos para R6/R15
    local SkeletonLines = {}
    local maxSkeletonBones = 16
    for i = 1, maxSkeletonBones do
        local line = Drawing.new("Line")
        line.Color = Color3.fromRGB(255, 255, 255)
        line.Thickness = 1.0
        line.Visible = false
        table.insert(SkeletonLines, line)
    end

    local connection
    connection = RunService.RenderStepped:Connect(function()
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChildOfClass("Humanoid") and player.Character:FindFirstChildOfClass("Humanoid").Health > 0 and player ~= LocalPlayer then
            if Settings.EspEnabled then
                local hrp = player.Character.HumanoidRootPart
                local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

                if onScreen then
                    -- Dimensionamento Dinâmico Inteligente com base na Distância (Perto, Médio, Longe)
                    local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
                    local factor = math.clamp(1000 / distance, 4, 120)
                    local width = 45 * factor
                    local height = 65 * factor

                    local boxTopLeft = Vector2.new(screenPos.X - width/2, screenPos.Y - height/2)

                    -- Renderização da Caixa ESP
                    if Settings.EspBox then
                        Box.Size = Vector2.new(width, height)
                        Box.Position = boxTopLeft
                        Box.Visible = true
                    else
                        Box.Visible = false
                    end

                    -- Barra e Texto de Vida no Lado Esquerdo da Box
                    if Settings.EspHealth then
                        local hpPercent = humanoid.Health / humanoid.MaxHealth
                        local barHeight = height * hpPercent
                        
                        HealthBar.From = Vector2.new(boxTopLeft.X - 6, boxTopLeft.Y + height)
                        HealthBar.To = Vector2.new(boxTopLeft.X - 6, boxTopLeft.Y + height - barHeight)
                        HealthBar.Color = Color3.fromRGB(255 * (1 - hpPercent), 255 * hpPercent, 0)
                        HealthBar.Visible = true

                        HealthText.Text = math.floor(humanoid.Health) .. " HP"
                        HealthText.Position = Vector2.new(boxTopLeft.X - 25, boxTopLeft.Y + (height/2) - 6)
                        HealthText.Visible = true
                    else
                        HealthBar.Visible = false
                        HealthText.Visible = false
                    end

                    -- Motor de Desenho de Esqueleto Integrado (R6 vs R15)
                    if Settings.EspSkeleton then
                        local rig = player.Character
                        local bones = {}

                        if Settings.RigType == "R15" then
                            local jointNames = {
                                {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
                                {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
                                {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
                                {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
                                {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}
                            }
                            for _, joint in ipairs(jointNames) do
                                if rig:FindFirstChild(joint[1]) and rig:FindFirstChild(joint[2]) then
                                    table.insert(bones, {rig[joint[1]].Position, rig[joint[2]].Position})
                                end
                            end
                        else -- Esquema R6
                            local jointNames = {
                                {"Head", "Torso"},
                                {"Torso", "Left Arm"}, {"Torso", "Right Arm"},
                                {"Torso", "Left Leg"}, {"Torso", "Right Leg"}
                            }
                            for _, joint in ipairs(jointNames) do
                                if rig:FindFirstChild(joint[1]) and rig:FindFirstChild(joint[2]) then
                                    table.insert(bones, {rig[joint[1]].Position, rig[joint[2]].Position})
                                end
                            end
                        end

                        -- Plotagem de Linhas do Esqueleto
                        for idx, line in ipairs(SkeletonLines) do
                            local bonePair = bones[idx]
                            if bonePair then
                                local posA, onScreenA = Camera:WorldToViewportPoint(bonePair[1])
                                local posB, onScreenB = Camera:WorldToViewportPoint(bonePair[2])
                                if onScreenA and onScreenB then
                                    line.From = Vector2.new(posA.X, posA.Y)
                                    line.To = Vector2.new(posB.X, posB.Y)
                                    line.Visible = true
                                else
                                    line.Visible = false
                                end
                            else
                                line.Visible = false
                            end
                        end
                    else
                        for _, line in ipairs(SkeletonLines) do line.Visible = false end
                    end
                else
                    Box.Visible = false
                    HealthBar.Visible = false
                    HealthText.Visible = false
                    for _, line in ipairs(SkeletonLines) do line.Visible = false end
                end
            else
                Box.Visible = false
                HealthBar.Visible = false
                HealthText.Visible = false
                for _, line in ipairs(SkeletonLines) do line.Visible = false end
            end
        else
            Box.Visible = false
            HealthBar.Visible = false
            HealthText.Visible = false
            for _, line in ipairs(SkeletonLines) do line.Visible = false end
            if not player.Parent then
                Box:Remove()
                HealthBar:Remove()
                HealthText:Remove()
                for _, line in ipairs(SkeletonLines) do line:Remove() end
                connection:Disconnect()
            end
        end
    end)
end

Players.PlayerAdded:Connect(CreatePlayerESP)
for _, p in pairs(Players:GetPlayers()) do CreatePlayerESP(p) end

-- [[ SISTEMA DE CONSTRUÇÃO DE INTERFACE DO CONTEÚDO ]] --
local function CreateButton(text, pos, size, parent, callback)
    local btn = Instance.new("TextButton")
    btn.Size = size
    btn.Position = pos
    btn.BackgroundColor3 = UI_THEME.ButtonBg
    btn.TextColor3 = UI_THEME.TextMuted
    btn.Text = text
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 14
    btn.BorderSizePixel = 0
    btn.Parent = parent

    local corner = Instance.new("UIC
