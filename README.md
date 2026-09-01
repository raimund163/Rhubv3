--[[
    Nome do Script: Rhub (Versão Funcional Otimizada)
    Plataforma: Roblox / Executores Executor Lua (Delta, etc.)
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local Settings = {
    Aimbot = {
        Enabled = false,
        TeamCheck = true,
        WallCheck = true,
        FOV = 120,
        Prediction = true
    },
    ESP = {
        Enabled = false,
        Box = false,
        HealthBar = false,
        Skeleton = false,
        TeamCheck = true
    }
}

-- Criar ScreenGui de forma segura
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RhubGUI"
ScreenGui.ResetOnSpawn = false

pcall(function()
    if syn and syn.protect_gui then
        syn.protect_gui(ScreenGui)
        ScreenGui.Parent = game:GetService("CoreGui")
    else
        ScreenGui.Parent = game:GetService("CoreGui")
    end
end)

if not ScreenGui.Parent then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Botão Flutuante (Abre/Fecha)
local ToggleButton = Instance.new("TextButton", ScreenGui)
ToggleButton.Name = "ToggleRhub"
ToggleButton.Size = UDim2.new(0, 42, 0, 42)
ToggleButton.Position = UDim2.new(0, 15, 0.4, 0)
ToggleButton.BackgroundColor3 = Color3.fromRGB(24, 25, 30)
ToggleButton.Text = "RH"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 13

local ToggleCorner = Instance.new("UICorner", ToggleButton)
ToggleCorner.CornerRadius = UDim.new(0, 8)

-- Janela Principal
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 400, 0, 230)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -115)
MainFrame.BackgroundColor3 = Color3.fromRGB(18, 19, 23)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false

local MainCorner = Instance.new("UICorner", MainFrame)
MainCorner.CornerRadius = UDim.new(0, 8)

-- Sistema de Arrastar (Dragify) universal para mobile e PC
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
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
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

ToggleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Título
local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, -40, 0, 30)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "Rhub - Painel Principal"
Title.TextColor3 = Color3.fromRGB(240, 240, 245)
Title.TextSize = 13
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Minimizar / Fechar Rápido (-)
local CloseBtn = Instance.new("TextButton", MainFrame)
CloseBtn.Size = UDim2.new(0, 24, 0, 24)
CloseBtn.Position = UDim2.new(1, -30, 0, 3)
CloseBtn.BackgroundColor3 = Color3.fromRGB(30, 32, 38)
CloseBtn.Text = "-"
CloseBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14

local CloseCorner = Instance.new("UICorner", CloseBtn)
CloseCorner.CornerRadius = UDim.new(0, 4)

CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

-- Menu Lateral Esquerdo
local LeftMenu = Instance.new("Frame", MainFrame)
LeftMenu.Size = UDim2.new(0, 110, 1, -40)
LeftMenu.Position = UDim2.new(0, 8, 0, 34)
LeftMenu.BackgroundColor3 = Color3.fromRGB(24, 25, 30)

local LeftCorner = Instance.new("UICorner", LeftMenu)
LeftCorner.CornerRadius = UDim.new(0, 6)

-- Painel Central de Opções
local CenterPanel = Instance.new("Frame", MainFrame)
CenterPanel.Size = UDim2.new(1, -126, 1, -40)
CenterPanel.Position = UDim2.new(0, 122, 0, 34)
CenterPanel.BackgroundColor3 = Color3.fromRGB(24, 25, 30)

local CenterCorner = Instance.new("UICorner", CenterPanel)
CenterCorner.CornerRadius = UDim.new(0, 6)

local AimbotSection = Instance.new("ScrollingFrame", CenterPanel)
AimbotSection.Size = UDim2.new(1, 0, 1, 0)
AimbotSection.BackgroundTransparency = 1
AimbotSection.ScrollBarThickness = 2
AimbotSection.Visible = true
AimbotSection.CanvasSize = UDim2.new(0, 0, 0, 180)

local ESPSection = Instance.new("ScrollingFrame", CenterPanel)
ESPSection.Size = UDim2.new(1, 0, 1, 0)
ESPSection.BackgroundTransparency = 1
ESPSection.ScrollBarThickness = 2
ESPSection.Visible = false
ESPSection.CanvasSize = UDim2.new(0, 0, 0, 200)

local function createMenuButton(name, posY, targetFrame)
    local btn = Instance.new("TextButton", LeftMenu)
    btn.Size = UDim2.new(1, -8, 0, 28)
    btn.Position = UDim2.new(0, 4, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(34, 36, 44)
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(210, 210, 215)
    btn.Font = Enum.Font.GothamMedium
    btn.TextSize = 11

    local btnCorner = Instance.new("UICorner", btn)
    btnCorner.CornerRadius = UDim.new(0, 4)

    btn.MouseButton1Click:Connect(function()
        AimbotSection.Visible = (targetFrame == AimbotSection)
        ESPSection.Visible = (targetFrame == ESPSection)
    end)
end

createMenuButton("Aimbot", 6, AimbotSection)
createMenuButton("ESP", 38, ESPSection)

local function createToggle(parent, name, posY, callback)
    local label = Instance.new("TextLabel", parent)
    label.Size = UDim2.new(0, 140, 0, 26)
    label.Position = UDim2.new(0, 8, 0, posY)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.fromRGB(180, 180, 185)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.Gotham
    label.TextSize = 11

    local toggleBtn = Instance.new("TextButton", parent)
    toggleBtn.Size = UDim2.new(0, 36, 0, 18)
    toggleBtn.Position = UDim2.new(1, -42, 0, posY + 4)
    toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 42, 50)
    toggleBtn.Text = "OFF"
    toggleBtn.TextColor3 = Color3.fromRGB(220, 70, 70)
    toggleBtn.Font = Enum.Font.GothamBold
    toggleBtn.TextSize = 10

    local tCorner = Instance.new("UICorner", toggleBtn)
    tCorner.CornerRadius = UDim.new(0, 4)

    local state = false
    toggleBtn.MouseButton1Click:Connect(function()
        state = not state
        if state then
            toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 160, 80)
            toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            toggleBtn.Text = "ON"
        else
            toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 42, 50)
            toggleBtn.TextColor3 = Color3.fromRGB(220, 70, 70)
            toggleBtn.Text = "OFF"
        end
        callback(state)
    end)
end

-- Botões do Aimbot
createToggle(AimbotSection, "Ativar Aimbot", 6, function(v) Settings.Aimbot.Enabled = v end)
createToggle(AimbotSection, "Team Check", 36, function(v) Settings.Aimbot.TeamCheck = v end)
createToggle(AimbotSection, "Wall Check", 66, function(v) Settings.Aimbot.WallCheck = v end)
createToggle(AimbotSection, "Predição Dinâmica", 96, function(v) Settings.Aimbot.Prediction = v end)

-- Botões do ESP
createToggle(ESPSection, "Ativar ESP", 6, function(v) Settings.ESP.Enabled = v end)
createToggle(ESPSection, "ESP Box", 36, function(v) Settings.ESP.Box = v end)
createToggle(ESPSection, "Barra de Vida (Esquerda)", 66, function(v) Settings.ESP.HealthBar = v end)
createToggle(ESPSection, "Esqueleto (R6/R15)", 96, function(v) Settings.ESP.Skeleton = v end)
createToggle(ESPSection, "Team Check", 126, function(v) Settings.ESP.TeamCheck = v end)

-- Círculo de FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Transparency = 0.5
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1
FOVCircle.NumSides = 32
FOVCircle.Radius = Settings.Aimbot.FOV
FOVCircle.Filled = false

local ESPObjects = {}

local function getRootPart(character)
    return character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso") or character:FindFirstChild("UpperTorso")
end

local function isVisible(targetPart)
    if not Settings.Aimbot.WallCheck then return true end
    local origin = Camera.CFrame.Position
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Blacklist
    rayParams.FilterDescendantsInstances = {LocalPlayer.Character, Camera}
    local result = workspace:Raycast(origin, targetPart.Position - origin, rayParams)
    if result then
        return result.Instance:IsDescendantOf(targetPart.Parent)
    end
    return true
end

local function getClosestTarget()
    local closestTarget = nil
    local shortestDist = Settings.Aimbot.FOV

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            local rootPart = getRootPart(player.Character)

            if humanoid and humanoid.Health > 0 and rootPart then
                if not Settings.Aimbot.TeamCheck or player.Team ~= LocalPlayer.Team then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
                    if onScreen then
                        local mousePos = UserInputService:GetMouseLocation()
                        local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude

                        if dist < shortestDist and isVisible(rootPart) then
                            shortestDist = dist
                            closestTarget = rootPart
                        end
                    end
                end
            end
        end
    end
    return closestTarget
end

local function createESP(player)
    local drawings = {
        Box = Drawing.new("Square"),
        HealthBarBg = Drawing.new("Square"),
        HealthBarFill = Drawing.new("Square"),
        SkeletonLines = {}
    }

    drawings.Box.Visible = false
    drawings.Box.Thickness = 1
    drawings.Box.Color = Color3.fromRGB(255, 255, 255)
    drawings.Box.Filled = false

    drawings.HealthBarBg.Visible = false
    drawings.HealthBarBg.Thickness = 1
    drawings.HealthBarBg.Color = Color3.fromRGB(0, 0, 0)
    drawings.HealthBarBg.Filled = true

    drawings.HealthBarFill.Visible = false
    drawings.HealthBarFill.Thickness = 1
    drawings.HealthBarFill.Color = Color3.fromRGB(0, 255, 0)
    drawings.HealthBarFill.Filled = true

    ESPObjects[player] = drawings
end

local function removeESP(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do
            if typeof(obj) == "table" then
                for _, line in pairs(obj) do line:Remove() end
            else
                obj:Remove()
            end
        end
        ESPObjects[player] = nil
    end
end

Players.PlayerAdded:Connect(createESP)
Players.PlayerRemoving:Connect(removeESP)
for _, p in ipairs(Players:GetPlayers()) do if p ~= LocalPlayer then createESP(p) end end

local function updateSkeleton(character, drawings)
    local r15Connections = {
        {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
        {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
        {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
        {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
        {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}
    }
    local r6Connections = {
        {"Head", "Torso"}, {"Torso", "Left Arm"}, {"Torso", "Right Arm"},
        {"Torso", "Left Leg"}, {"Torso", "Right Leg"}
    }

    local isR15 = character:FindFirstChild("UpperTorso") ~= nil
    local connections = isR15 and r15Connections or r6Connections

    while #drawings.SkeletonLines < #connections do
        local l = Drawing.new("Line")
        l.Visible = false
        l.Thickness = 1
        l.Color = Color3.fromRGB(255, 255, 255)
        table.insert(drawings.SkeletonLines, l)
    end

    for i, conn in ipairs(connections) do
        local p1 = character:FindFirstChild(conn[1])
        local p2 = character:FindFirstChild(conn[2])
        local line = drawings.SkeletonLines[i]

        if p1 and p2 then
            local pos1, on1 = Camera:WorldToViewportPoint(p1.Position)
            local pos2, on2 = Camera:WorldToViewportPoint(p2.Position)

            if on1 or on2 then
                line.From = Vector2.new(pos1.X, pos1.Y)
                line.To = Vector2.new(pos2.X, pos2.Y)
                line.Visible = true
            else
                line.Visible = false
            end
        else
            line.Visible = false
        end
    end

    for i = #connections + 1, #drawings.SkeletonLines do
        drawings.SkeletonLines[i].Visible = false
    end
end

-- Loop de Renderização Principal
RunService.RenderStepped:Connect(function()
    FOVCircle.Position = UserInputService:GetMouseLocation()
    FOVCircle.Radius = Settings.Aimbot.FOV
    FOVCircle.Visible = Settings.Aimbot.Enabled

    if Settings.Aimbot.Enabled then
        local target = getClosestTarget()
        if target then
            local targetPos = target.Position
            if Settings.Aimbot.Prediction then
                local distance = (Camera.CFrame.Position - targetPos).Magnitude
                if distance >= 800 then
                    local velocity = target.Velocity or Vector3.new(0, 0, 0)
                    targetPos = targetPos + (velocity * 0.22)
                end
            end
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
        end
    end

    for player, drawings in pairs(ESPObjects) do
        local char = player.Character
        local root = char and getRootPart(char)
        local hum = char and char:FindFirstChildOfClass("Humanoid")

        local showESP = Settings.ESP.Enabled and char and root and hum and hum.Health > 0
        if showESP and Settings.ESP.TeamCheck and player.Team == LocalPlayer.Team then
            showESP = false
        end

        if showESP then
            local vector, onScreen = Camera:WorldToViewportPoint(root.Position)
            if onScreen then
                local head = char:FindFirstChild("Head")
                local headPos = head and Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0)) or vector
                local legPos = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))

                local height = math.abs(headPos.Y - legPos.Y)
                local width = height / 2

                if Settings.ESP.Box then
                    drawings.Box.Size = Vector2.new(width, height)
                    drawings.Box.Position = Vector2.new(vector.X - width / 2, headPos.Y)
                    drawings.Box.Visible = true
                else
                    drawings.Box.Visible = false
                end

                if Settings.ESP.HealthBar and Settings.ESP.Box then
                    local healthPercent = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
                    local barHeight = height * healthPercent

                    drawings.HealthBarBg.Size = Vector2.new(3, height)
                    drawings.HealthBarBg.Position = Vector2.new(vector.X - width / 2 - 6, headPos.Y)
                    drawings.HealthBarBg.Visible = true

                    drawings.HealthBarFill.Size = Vector2.new(1, barHeight)
                    drawings.HealthBarFill.Position = Vector2.new(vector.X - width / 2 - 5, headPos.Y + (height - barHeight))
                    drawings.HealthBarFill.Color = Color3.fromRGB(255 * (1 - healthPercent), 255 * healthPercent, 0)
                    drawings.HealthBarFill.Visible = true
                else
                    drawings.HealthBarBg.Visible = false
                    drawings.HealthBarFill.Visible = false
                end

                if Settings.ESP.Skeleton then
                    updateSkeleton(char, drawings)
                else
                    for _, l in pairs(drawings.SkeletonLines) do l.Visible = false end
                end
            else
                drawings.Box.Visible = false
                drawings.HealthBarBg.Visible = false
                drawings.HealthBarFill.Visible = false
                for _, l in pairs(drawings.SkeletonLines) do l.Visible = false end
            end
        else
            drawings.Box.Visible = false
            drawings.HealthBarBg.Visible = false
            drawings.HealthBarFill.Visible = false
            for _, l in pairs(drawings.SkeletonLines) do l.Visible = false end
        end
    end
end)

print("Rhub Hub carregado e pronto para uso!")
EspBox = false,
EspHealth = false,
EspSkeleton = false,
RigType = "R15" -- "R15" ou "R6"}
-- [[ FUNÇÃO DE SEGURANÇA PARA ARREDONDAMENTO DE CANTOS ]] 
---- Evita travamentos caso o executor não tenha suporte completo para UICornerlocal function ApplyCorner(instance, radius)local success, _ = pcall(function()local corner = Instance.new("UICorner")corner.CornerRadius = UDim.new(0, radius)corner.Parent = instanceend)return successend-- [[ CRIAÇÃO DA ESTRUTURA DA GUI ]] --local ScreenGui = Instance.new("ScreenGui")ScreenGui.Name = "PremiumCheatGUI_V2"ScreenGui.ResetOnSpawn = falseScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling-- Injeção segura na CoreGui ou fallback para PlayerGuilocal success, _ = pcall(function()ScreenGui.Parent = CoreGuiend)if not success thenScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")end-- Botão Flutuante para Abrir/Fechar a Interfacelocal ToggleButton = Instance.new("TextButton")ToggleButton.Name = "MenuToggle"ToggleButton.Size = UDim2.new(0, 140, 0, 42)ToggleButton.Position = UDim2.new(0, 25, 0, 25)ToggleButton.BackgroundColor3 = UI_THEME.SidebarToggleButton.BorderSizePixel = 0ToggleButton.Text = "Menu: Ativo"ToggleButton.TextColor3 = UI_THEME.TextActiveToggleButton.Font = Enum.Font.SourceSansBoldToggleButton.TextSize = 15ToggleButton.Parent = ScreenGuiApplyCorner(ToggleButton, 8)local MainFrame = Instance.new("Frame")MainFrame.Name = "MainFrame"MainFrame.Size = UDim2.new(0, 580, 0, 380)MainFrame.Position = UDim2.new(0.5, -290, 0.5, -190)MainFrame.BackgroundColor3 = UI_THEME.BackgroundMainFrame.BorderSizePixel = 0MainFrame.Visible = trueMainFrame.Parent = ScreenGuiApplyCorner(MainFrame, 10)-- Sistema de Arrastar com Efeito Suave (Smooth Drag)local dragging, dragInput, dragStart, startPoslocal function updateDrag(input)local delta = input.Position - dragStartlocal targetPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)TweenService:Create(MainFrame, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = targetPos}):Play()endMainFrame.InputBegan:Connect(function(input)if input.UserInputType == Enum.UserInputType.MouseButton1 thendragging = truedragStart = input.PositionstartPos = MainFrame.Positioninput.Changed:Connect(function()if input.UserInputState == Enum.UserInputState.End thendragging = falseendend)endend)MainFrame.InputChanged:Connect(function(input)if input.UserInputType == Enum.UserInputType.MouseMovement thendragInput = inputendend)UserInputService.InputChanged:Connect(function(input)if input == dragInput and dragging thenupdateDrag(input)endend)ToggleButton.MouseButton1Click:Connect(function()MainFrame.Visible = not MainFrame.VisibleToggleButton.Text = MainFrame.Visible and "Menu: Ativo" or "Menu: Oculto"end)-- Painel Lateral Esquerdo (Menu de Navegação)local Sidebar = Instance.new("Frame")Sidebar.Name = "Sidebar"Sidebar.Size = UDim2.new(0, 160, 1, 0)Sidebar.BackgroundColor3 = UI_THEME.SidebarSidebar.BorderSizePixel = 0Sidebar.Parent = MainFrameApplyCorner(Sidebar, 10)-- Obstruir cantos direitos do painel lateral para manter o design limpolocal SidebarPatch = Instance.new("Frame")SidebarPatch.Size = UDim2.new(0, 10, 1, 0)SidebarPatch.Position = UDim2.new(1, -10, 0, 0)SidebarPatch.BackgroundColor3 = UI_THEME.SidebarSidebarPatch.BorderSizePixel = 0SidebarPatch.Parent = Sidebar-- Logo / Título do Menulocal LogoLabel = Instance.new("TextLabel")LogoLabel.Size = UDim2.new(1, 0, 0, 50)LogoLabel.BackgroundTransparency = 1LogoLabel.Text = "PREMIUM CHEAT"LogoLabel.TextColor3 = UI_THEME.AccentLogoLabel.Font = Enum.Font.SourceSansBoldLogoLabel.TextSize = 18LogoLabel.Parent = Sidebar-- Painel Conteúdo Direitolocal ContentFrame = Instance.new("Frame")ContentFrame.Name = "ContentFrame"ContentFrame.Size = UDim2.new(1, -160, 1, 0)ContentFrame.Position = UDim2.new(0, 160, 0, 0)ContentFrame.BackgroundTransparency = 1ContentFrame.Parent = MainFrame-- Avatar de Noob Estilizado (Lado Direito)local NoobContainer = Instance.new("Frame")NoobContainer.Size = UDim2.new(0, 110, 0, 110)NoobContainer.Position = UDim2.new(1, -130, 0, 20)NoobContainer.BackgroundColor3 = UI_THEME.SidebarNoobContainer.BorderSizePixel = 0NoobContainer.Parent = ContentFrameApplyCorner(NoobContainer, 12)local NoobAvatar = Instance.new("ImageLabel")NoobAvatar.Size = UDim2.new(1, -10, 1, -10)NoobAvatar.Position = UDim2.new(0, 5, 0, 5)NoobAvatar.BackgroundTransparency = 1NoobAvatar.Image = "rbxassetid://134149021" -- ID de Recurso do Noob ClássicoNoobAvatar.Parent = NoobContainer-- Título da Página de Configuração Dinâmicalocal PageTitle = Instance.new("TextLabel")PageTitle.Size = UDim2.new(1, -160, 0, 40)PageTitle.Position = UDim2.new(0, 20, 0, 15)PageTitle.BackgroundTransparency = 1PageTitle.Text = "Configurações de Aimbot"PageTitle.TextColor3 = UI_THEME.TextActivePageTitle.Font = Enum.Font.SourceSansBoldPageTitle.TextSize = 22PageTitle.TextAlign = Enum.TextAlign.LeftPageTitle.Parent = ContentFrame-- Containers para Alternar Abas (Aimbot / ESP)local AimbotPage = Instance.new("Frame")AimbotPage.Size = UDim2.new(1, 0, 1, -60)AimbotPage.Position = UDim2.new(0, 0, 0, 60)AimbotPage.BackgroundTransparency = 1AimbotPage.Visible = trueAimbotPage.Parent = ContentFramelocal EspPage = Instance.new("Frame")EspPage.Size = UDim2.new(1, 0, 1, -60)EspPage.Position = UDim2.new(0, 0, 0, 60)EspPage.BackgroundTransparency = 1EspPage.Visible = falseEspPage.Parent = ContentFrame-- [[ DRAWING API (VETORES DO FOV E SNAPLINE) ]] --local FovCircle = Drawing.new("Circle")FovCircle.Color = Color3.fromRGB(0, 162, 255)FovCircle.Thickness = 1.2FovCircle.NumSides = 64FovCircle.Radius = Settings.FovRadiusFovCircle.Filled = falseFovCircle.Visible = truelocal SnapLine = Drawing.new("Line")SnapLine.Color = Color3.fromRGB(255, 235, 59)SnapLine.Thickness = 1.5SnapLine.Visible = false-- [[ MÓDULO MATEMÁTICO DO AIMBOT ]] --local function GetClosestTarget()local closestPlayer = nillocal shortestDistance = math.hugefor _, player in pairs(Players:GetPlayers()) do
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
end-- Loop de Atualização do Aimbot e Linha de DirecionamentoRunService.RenderStepped:Connect(function()local mousePos = UserInputService:GetMouseLocation()FovCircle.Position = mousePosFovCircle.Radius = Settings.FovRadiusif Settings.AimbotEnabled then
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

            -- Travamento da Mira (Ao manter pressionado o Botão Direito do Mouse)
            if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPosition)
            end

            -- Desenhar linha até à cabeça/parte alvo se o Wall Check estiver ativo ou livre
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
end)-- [[ SISTEMA AVANÇADO DE VISUAIS (ESP ENGINE) ]] --local function CreatePlayerESP(player)local Box = Drawing.new("Square")Box.Visible = falseBox.Color = Color3.fromRGB(255, 40, 40)Box.Thickness = 1.5Box.Filled = falselocal HealthBar = Drawing.new("Line")
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
                -- Tamanho dinâmico adaptativo conforme a distância (Longe, Médio ou Perto)
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

                -- Barra de Vida e Valor Numérico no lado esquerdo da Box
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

                -- Esqueleto Dinâmico (R6 / R15)
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

                    -- Desenho das linhas do esqueleto
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
endPlayers.PlayerAdded:Connect(CreatePlayerESP)for _, p in pairs(Players:GetPlayers()) do CreatePlayerESP(p) end-- [[ CONSTRUTOR DE BOTÕES COM PROTEÇÃO DE CANTOS ]] --local function CreateButton(text, pos, size, parent, callback)local btn = Instance.new("TextButton")btn.Size = sizebtn.Position = posbtn.BackgroundColor3 = UI_THEME.ButtonBgbtn.TextColor3 = UI_THEME.TextMutedbtn.Text = textbtn.Font = Enum.Font.SourceSansBoldbtn.TextSize = 14btn.BorderSizePixel = 0btn.Parent = parentApplyCorner(btn, 6)

btn.MouseEnter:Connect(function()
    TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = UI_THEME.ButtonHover, TextColor3 = UI_THEME.TextActive}):Play()
end)
btn.MouseLeave:Connect(function()
    TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = UI_THEME.ButtonBg, TextColor3 = UI_THEME.TextMuted}):Play()
end)

btn.MouseButton1Click:Connect(function()
    callback(btn)
end)
return btn
end
-- Botões de Menu na Barra Lateral (Alternar Abas)CreateButton("Config. Aimbot", UDim2.new(0, 10, 0, 70), UDim2.new(1, -20, 0, 36), Sidebar, function()PageTitle.Text = "Configurações de Aimbot"AimbotPage.Visible = trueEspPage.Visible = falseend)CreateButton("Visuals (ESP)", UDim2.new(0, 10, 0, 115), UDim2.new(1, -20, 0, 36), Sidebar, function()PageTitle.Text = "Configurações Visuais (ESP)"AimbotPage.Visible = falseEspPage.Visible = trueend)-- CONTROLES DA ABA: AIMBOTlocal AimToggle = CreateButton("Aimbot: DESATIVADO", UDim2.new(0, 20, 0, 10), UDim2.new(0, 180, 0, 32), AimbotPage, function(b)Settings.AimbotEnabled = not Settings.AimbotEnabledb.Text = Settings.AimbotEnabled and "Aimbot: ATIVADO" or "Aimbot: DESATIVADO"end)local TeamToggle = CreateButton("Team Check: INATIVO", UDim2.new(0, 20, 0, 50), UDim2.new(0, 180, 0, 32), AimbotPage, function(b)Settings.TeamCheck = not Settings.TeamCheckb.Text = Settings.TeamCheck and "Team Check: ATIVO" or "Team Check: INATIVO"end)local WallToggle = CreateButton("Wall Check: INATIV
