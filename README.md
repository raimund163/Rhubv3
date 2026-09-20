        -- [[ CONFIGURAÇÕES INICIAIS E VARIÁVEIS ]]
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

-- Configurações mutáveis pela GUI
local Config = {
    AimbotEnabled = false,
    TeamCheck = false,
    WallCheck = false,
    Prediction = false,
    TargetPart = "Head", -- "Head", "Torso", "Foot" (Adaptável R6/R15)
    FOV = 10,
    
    ESPEnabled = false,
    ESPBoxes = false,
    ESPHealth = false,
    ESPSkeleton = false,
    RigType = "R15" -- "R15" ou "R6"
}

-- Armazenar objetos de desenho (Drawing API) para limpeza/atualização rápida
local Cache = {
    Boxes = {},
    HealthBars = {},
    Skeletons = {}
}

-- Criando o Círculo do FOV usando Drawing API fixado no centro da mira
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Thickness = 1.5
FOVCircle.NumSides = 60
FOVCircle.Radius = Config.FOV
FOVCircle.Filled = false
FOVCircle.Visible = true

-- Linha do Alvo (Tracer que "fuma" uma linha até o inimigo)
local TargetLine = Drawing.new("Line")
TargetLine.Color = Color3.fromRGB(255, 255, 0)
TargetLine.Thickness = 2
TargetLine.Visible = false

---
-- [[ INTERFACE GRÁFICA (GUI) DE ALTA QUALIDADE ]]
---

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomMenuByGemini"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Inserindo na CoreGui ou PlayerGui dependendo do exploit/ambiente
pcall(function() ScreenGui.Parent = CoreGui end)
if not ScreenGui.Parent then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

-- Botão de Abrir/Fechar flutuante
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 120, 0, 40)
ToggleBtn.Position = UDim2.new(0, 10, 0, 10)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleBtn.Text = "Abrir/Fechar"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Font = Enum.Font.SourceSansBold
ToggleBtn.TextSize = 16
ToggleBtn.BorderSizePixel = 1
ToggleBtn.Parent = ScreenGui

-- Menu Principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 560, 0, 360)
MainFrame.Position = UDim2.new(0.5, -280, 0.5, -180)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderSizePixel = 2
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

-- Função de arrastar para o Menu Principal
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end
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
        update(input)
    end
end)

ToggleBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Painel Lateral Esquerdo (Abas)
local LeftPanel = Instance.new("Frame")
LeftPanel.Size = UDim2.new(0, 130, 1, 0)
LeftPanel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
LeftPanel.BorderSizePixel = 0
LeftPanel.Parent = MainFrame

-- Conteúdos das Abas
local AimbotContent = Instance.new("Frame")
AimbotContent.Size = UDim2.new(0, 260, 1, 0)
AimbotContent.Position = UDim2.new(0, 130, 0, 0)
AimbotContent.BackgroundTransparency = 1
AimbotContent.Parent = MainFrame

local ESPContent = Instance.new("Frame")
ESPContent.Size = UDim2.new(0, 260, 1, 0)
ESPContent.Position = UDim2.new(0, 130, 0, 0)
ESPContent.BackgroundTransparency = 1
ESPContent.Visible = false
ESPContent.Parent = MainFrame

-- Botões de Aba (Menu Esquerdo)
local TabAimbot = Instance.new("TextButton")
TabAimbot.Size = UDim2.new(1, 0, 0, 50)
TabAimbot.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
TabAimbot.Text = "AIMBOT"
TabAimbot.TextColor3 = Color3.fromRGB(255, 255, 255)
TabAimbot.Font = Enum.Font.SourceSansBold
TabAimbot.TextSize = 16
TabAimbot.BorderSizePixel = 0
TabAimbot.Parent = LeftPanel

local TabESP = Instance.new("TextButton")
TabESP.Size = UDim2.new(1, 0, 0, 50)
TabESP.Position = UDim2.new(0, 0, 0, 55)
TabESP.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
TabESP.Text = "ESP"
TabESP.TextColor3 = Color3.fromRGB(200, 200, 200)
TabESP.Font = Enum.Font.SourceSansBold
TabESP.TextSize = 16
TabESP.BorderSizePixel = 0
TabESP.Parent = LeftPanel

TabAimbot.MouseButton1Click:Connect(function()
    AimbotContent.Visible = true
    ESPContent.Visible = false
    TabAimbot.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    TabESP.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
end)
TabESP.MouseButton1Click:Connect(function()
    AimbotContent.Visible = false
    ESPContent.Visible = true
    TabESP.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    TabAimbot.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
end)

-- [[ GERADOR DE BOTÕES LIGA/DESLIGA ]]
local function createToggle(name, pos, parent, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 240, 0, 35)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    btn.Text = name .. ": DESLIGADO"
    btn.TextColor3 = Color3.fromRGB(255, 100, 100)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 14
    btn.BorderSizePixel = 1
    
    local enabled = false
    btn.MouseButton1Click:Connect(function()
        enabled = not enabled
        btn.Text = name .. ": " .. (enabled and "LIGADO" or "DESLIGADO")
        btn.TextColor3 = enabled and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
        btn.BackgroundColor3 = enabled and Color3.fromRGB(45, 45, 45) or Color3.fromRGB(35, 35, 35)
        callback(enabled)
    end)
    btn.Parent = parent
    return btn
end

-- Botões de Configurações do Aimbot
createToggle("Ativar Aimbot", UDim2.new(0, 10, 0, 15), AimbotContent, function(v) Config.AimbotEnabled = v end)
createToggle("Team Check", UDim2.new(0, 10, 0, 55), AimbotContent, function(v) Config.TeamCheck = v end)
createToggle("Wall Check", UDim2.new(0, 10, 0, 95), AimbotContent, function(v) Config.WallCheck = v end)
createToggle("Previsão de Tiro", UDim2.new(0, 10, 0, 135), AimbotContent, function(v) Config.Prediction = v end)

-- Ajustes do tamanho do FOV (Mais e Menos)
local FovLabel = Instance.new("TextLabel")
FovLabel.Size = UDim2.new(0, 120, 0, 30)
FovLabel.Position = UDim2.new(0, 70, 0, 175)
FovLabel.Text = "AIMFOV: " .. Config.FOV
FovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
FovLabel.Font = Enum.Font.SourceSansBold
FovLabel.TextSize = 14
FovLabel.BackgroundTransparency = 1
FovLabel.Parent = AimbotContent

local MinusBtn = Instance.new("TextButton")
MinusBtn.Size = UDim2.new(0, 50, 0, 30)
MinusBtn.Position = UDim2.new(0, 10, 0, 175)
MinusBtn.Text = "-"
MinusBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MinusBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
MinusBtn.Font = Enum.Font.SourceSansBold
MinusBtn.TextSize = 18
MinusBtn.MouseButton1Click:Connect(function()
    Config.FOV = math.max(10, Config.FOV - 10)
    FovLabel.Text = "AIMFOV: " .. Config.FOV
    FOVCircle.Radius = Config.FOV
end)
MinusBtn.Parent = AimbotContent

local PlusBtn = Instance.new("TextButton")
PlusBtn.Size = UDim2.new(0, 50, 0, 30)
PlusBtn.Position = UDim2.new(0, 200, 0, 175)
PlusBtn.Text = "+"
PlusBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
PlusBtn.TextColor3 = Color3.fromRGB(100, 255, 100)
PlusBtn.Font = Enum.Font.SourceSansBold
PlusBtn.TextSize = 18
PlusBtn.MouseButton1Click:Connect(function()
    Config.FOV = math.min(600, Config.FOV + 10)
    FovLabel.Text = "AIMFOV: " .. Config.FOV
    FOVCircle.Radius = Config.FOV
end)
PlusBtn.Parent = AimbotContent

-- Seleção de Travamento por Parte do Corpo (Cabeça, Tronco, Pé)
local PartLabel = Instance.new("TextLabel")
PartLabel.Size = UDim2.new(0, 240, 0, 20)
PartLabel.Position = UDim2.new(0, 10, 0, 215)
PartLabel.Text = "Parte Alvo: Cabeça"
PartLabel.TextColor3 = Color3.fromRGB(0, 255, 255)
PartLabel.Font = Enum.Font.SourceSansBold
PartLabel.TextSize = 14
PartLabel.BackgroundTransparency = 1
PartLabel.Parent = AimbotContent

local PartHead = Instance.new("TextButton")
PartHead.Size = UDim2.new(0, 75, 0, 30)
PartHead.Position = UDim2.new(0, 10, 0, 240)
PartHead.Text = "Cabeça"
PartHead.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
PartHead.TextColor3 = Color3.fromRGB(255, 255, 255)
PartHead.Font = Enum.Font.SourceSansBold
PartHead.MouseButton1Click:Connect(function() 
    Config.TargetPart = "Head" 
    PartLabel.Text = "Parte Alvo: Cabeça" 
end)
PartHead.Parent = AimbotContent

local PartTorso = Instance.new("TextButton")
PartTorso.Size = UDim2.new(0, 75, 0, 30)
PartTorso.Position = UDim2.new(0, 92, 0, 240)
PartTorso.Text = "Tronco"
PartTorso.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
PartTorso.TextColor3 = Color3.fromRGB(255, 255, 255)
PartTorso.Font = Enum.Font.SourceSansBold
PartTorso.MouseButton1Click:Connect(function() 
    Config.TargetPart = "Torso" 
    PartLabel.Text = "Parte Alvo: Tronco" 
end)
PartTorso.Parent = AimbotContent

local PartFoot = Instance.new("TextButton")
PartFoot.Size = UDim2.new(0, 75, 0, 30)
PartFoot.Position = UDim2.new(0, 175, 0, 240)
PartFoot.Text = "Pé"
PartFoot.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
PartFoot.TextColor3 = Color3.fromRGB(255, 255, 255)
PartFoot.Font = Enum.Font.SourceSansBold
PartFoot.MouseButton1Click:Connect(function() 
    Config.TargetPart = "Foot" 
    PartLabel.Text = "Parte Alvo: Pé" 
end)
PartFoot.Parent = AimbotContent


-- [[ ABA DE CONFIGURAÇÃO DO ESP ]]
createToggle("Ativar ESP", UDim2.new(0, 10, 0, 15), ESPContent, function(v) Config.ESPEnabled = v end)
createToggle("Mostrar Box Dinâmico", UDim2.new(0, 10, 0, 55), ESPContent, function(v) Config.ESPBoxes = v end)
createToggle("Mostrar Vida (Esquerda)", UDim2.new(0, 10, 0, 95), ESPContent, function(v) Config.ESPHealth = v end)
createToggle("Mostrar Esqueleto", UDim2.new(0, 10, 0, 135), ESPContent, function(v) Config.ESPSkeleton = v end)

-- Painel para Selecionar Esqueleto de R6 ou R15
local RigLabel = Instance.new("TextLabel")
RigLabel.Size = UDim2.new(0, 240, 0, 20)
RigLabel.Position = UDim2.new(0, 10, 0, 180)
RigLabel.Text = "Esqueleto Selecionado: R15"
RigLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
RigLabel.Font = Enum.Font.SourceSansBold
RigLabel.TextSize = 14
RigLabel.BackgroundTransparency = 1
RigLabel.Parent = ESPContent

local BtnR6 = Instance.new("TextButton")
BtnR6.Size = UDim2.new(0, 115, 0, 35)
BtnR6.Position = UDim2.new(0, 10, 0, 205)
BtnR6.Text = "Selecionar R6"
BtnR6.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
BtnR6.TextColor3 = Color3.fromRGB(255, 255, 255)
BtnR6.Font = Enum.Font.SourceSansBold
BtnR6.MouseButton1Click:Connect(function() 
    Config.RigType = "R6" 
    RigLabel.Text = "Esqueleto Selecionado: R6"
end)
BtnR6.Parent = ESPContent

local BtnR15 = Instance.new("TextButton")
BtnR15.Size = UDim2.new(0, 115, 0, 35)
BtnR15.Position = UDim2.new(0, 135, 0, 205)
BtnR15.Text = "Selecionar R15"
BtnR15.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
BtnR15.TextColor3 = Color3.fromRGB(255, 255, 255)
BtnR15.Font = Enum.Font.SourceSansBold
BtnR15.MouseButton1Click:Connect(function() 
    Config.RigType = "R15" 
    RigLabel.Text = "Esqueleto Selecionado: R15"
end)
BtnR15.Parent = ESPContent


-- [[ LADO DIREITO: VIEWPORT 3D COM AVATAR NOOB ]]
local RightPanel = Instance.new("Frame")
RightPanel.Size = UDim2.new(0, 170, 1, 0)
RightPanel.Position = UDim2.new(1, -170, 0, 0)
RightPanel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
RightPanel.BorderSizePixel = 0
RightPanel.Parent = MainFrame

-- Viewport Frame para o Boneco 3D
local ViewportFrame = Instance.new("ViewportFrame")
ViewportFrame.Size = UDim2.new(1, 0, 1, 0)
ViewportFrame.BackgroundTransparency = 1
ViewportFrame.Parent = RightPanel

local vCamera = Instance.new("Camera")
vCamera.FieldOfView = 50
ViewportFrame.CurrentCamera = vCamera
vCamera.Parent = ViewportFrame

-- Montando o Modelo Noob 3D de alta compatibilidade
local NoobModel = Instance.new("Model")
NoobModel.Name = "NoobModel"

local function createMeshPart(name, size, color, pos)
    local p = Instance.new("Part")
    p.Name = name
    p.Size = size
    p.Color = color
    p.Position = pos
    p.Anchored = true
    p.Material = Enum.Material.SmoothPlastic
    p.Parent = NoobModel
    return p
end

-- Posicionando os membros do Noob na GUI
local nHead = createMeshPart("Head", Vector3.new(1.2, 1.2, 1.2), Color3.fromRGB(253, 234, 141), Vector3.new(0, 1.5, 0))
local nTorso = createMeshPart("Torso", Vector3.new(2, 2, 1), Color3.fromRGB(13, 105, 172), Vector3.new(0, 0, 0))
local nLArm = createMeshPart("Left Arm", Vector3.new(1, 2, 1), Color3.fromRGB(253, 234, 141), Vector3.new(-1.6, 0, 0))
local nRArm = createMeshPart("Right Arm", Vector3.new(1, 2, 1), Color3.fromRGB(253, 234, 141), Vector3.new(1.6, 0, 0))
local nLLeg = createMeshPart("Left Leg", Vector3.new(1, 2, 1), Color3.fromRGB(102, 153, 102), Vector3.new(-0.6, -2, 0))
local nRLeg = createMeshPart("Right Leg", Vector3.new(1, 2, 1), Color3.fromRGB(102, 153, 102), Vector3.new(0.6, -2, 0))

-- Rosto Clássico do Noob
local faceDecal = Instance.new("Decal")
faceDecal.Texture = "rbxassetid://15229013" -- Face clássica sorrindo
faceDecal.Face = Enum.NormalId.Front
faceDecal.Parent = nHead

NoobModel.Parent = ViewportFrame
vCamera.CFrame = CFrame.new(Vector3.new(0, 0.2, 5.5), nTorso.Position)


---
-- [[ SISTEMA DE BUSCA, VALIDAÇÃO E FILTRO DE PERSONAGENS ]]
---

-- Localiza de forma extremamente robusta o modelo do Personagem do Jogador no Workspace
local function getValidCharacter(player)
    if not player then return nil end
    
    -- Busca primária: Propriedade nativa do Player
    local character = player.Character
    
    -- Busca secundária: Se o jogo gerou um modelo customizado com o nome do jogador no Workspace
    if not character then
        character = workspace:FindFirstChild(player.Name)
    end
    
    -- Validação de existência física e do Humanoid necessário
    if character and character:IsA("Model") then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        local hrp = character:FindFirstChild("HumanoidRootPart")
        
        if humanoid and hrp and humanoid.Health > 0 then
            return character, humanoid, hrp
        end
    end
    
    return nil
end

-- Mapeia partes do corpo com base em R6 ou R15 de forma flexível e recursiva
local function getTargetBodyPart(character)
    if not character then return nil end
    
    if Config.TargetPart == "Head" then
        return character:FindFirstChild("Head")
    elseif Config.TargetPart == "Torso" then
        return character:FindFirstChild("UpperTorso") or character:FindFirstChild("Torso") or character:FindFirstChild("LowerTorso")
    elseif Config.TargetPart == "Foot" then
        return character:FindFirstChild("LeftFoot") or character:FindFirstChild("Left Leg") or character:FindFirstChild("RightFoot") or character:FindFirstChild("Right Leg")
    end
    
    -- Fallback inteligente caso a parte padrão não exista
    return character:FindFirstChild("Head") or character:FindFirstChildOfClass("Part")
end

-- Raycast robusto para detecção de paredes (Wall Check funcional)
local function isVisibleThroughWall(targetPart, character)
    if not Config.WallCheck then return true end
    if not targetPart then return false end
    
    local origin = Camera.CFrame.Position
    local direction = targetPart.Position - origin
    
    local raycastParams = RaycastParams.new()
    -- Inclui o próprio personagem e o do alvo na lista de exclusão do raio físico
    local myChar = LocalPlayer.Character or workspace:FindFirstChild(LocalPlayer.Name)
    raycastParams.FilterDescendantsInstances = {myChar, character}
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    raycastParams.IgnoreWater = true
    
    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
    
    -- Se nada interceptou o trajeto físico, o alvo está visível
    return raycastResult == nil
end

-- Seleciona o melhor inimigo dentro do Círculo do FOV (calculado em relação ao centro da mira/tela)
local function getClosestTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local screenCenter = Camera.ViewportSize / 2

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            -- Team Check
            if Config.TeamCheck and player.Team == LocalPlayer.Team then continue end
            
            -- Detecta e valida o personagem utilizando nossa nova função dedicada
            local char, humanoid, hrp = getValidCharacter(player)
            if char then
                local part = getTargetBodyPart(char)
                if part and isVisibleThroughWall(part, char) then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                    if onScreen then
                        -- Distância medida a partir do centro da tela (mira física)
                        local distance = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                        
                        if distance < shortestDistance and distance <= Config.FOV then
                            closestPlayer = player
                            shortestDistance = distance
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end


-- [[ SISTEMA COMPARTIMENTADO DO ESP (BOX / VIDA / ESQUELETO R6 E R15) ]]

-- Criar recursos Drawing API para um jogador
local function applyESP(player)
    local Box = Drawing.new("Square")
    Box.Color = Color3.fromRGB(255, 0, 0)
    Box.Thickness = 1.5
    Box.Filled = false
    Box.Visible = false

    local HealthBar = Drawing.new("Line")
    HealthBar.Color = Color3.fromRGB(0, 255, 0)
    HealthBar.Thickness = 2
    HealthBar.Visible = false

    -- Esqueleto dinâmico: Array de Linhas conectadas
    local Bones = {}
    for i = 1, 15 do -- Máximo de 15 ossos mapeados
        local line = Drawing.new("Line")
        line.Color = Color3.fromRGB(255, 255, 255)
        line.Thickness = 1.5
        line.Visible = false
        table.insert(Bones, line)
    end

    Cache.Boxes[player] = Box
    Cache.HealthBars[player] = HealthBar
    Cache.Skeletons[player] = Bones

    local connection
    connection = RunService.RenderStepped:Connect(function()
        -- Verificando existência de dados válidos do alvo com nossa nova validação
        local char, humanoid, hrp = getValidCharacter(player)
        
        if not player or not char then
            if not Players:FindFirstChild(player.Name) then
                Box:Remove()
                HealthBar:Remove()
                for _, line in ipairs(Bones) do line:Remove() end
                Cache.Boxes[player] = nil
                Cache.HealthBars[player] = nil
                Cache.Skeletons[player] = nil
                connection:Disconnect()
            else
                Box.Visible = false
                HealthBar.Visible = false
                for _, line in ipairs(Bones) do line.Visible = false end
            end
            return
        end

        -- Se o ESP global estiver desligado
        if not Config.ESPEnabled then
            Box.Visible = false
            HealthBar.Visible = false
            for _, line in ipairs(Bones) do line.Visible = false end
            return
        end

        local hrpPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

        if onScreen then
            -- Métrica de distância dinâmicamente redimensionada (Longe, Médio ou Perto)
            local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
            local scale = (1 / distance) * 1000
            local w = 3 * scale
            local h = 4.5 * scale

            -- 1. BOX ESP DINÂMICO
            if Config.ESPBoxes then
                Box.Size = Vector2.new(w, h)
                Box.Position = Vector2.new(hrpPos.X - w / 2, hrpPos.Y - h / 2)
                
                -- Altera cor da box conforme proximidade (Longe = Verde, Médio = Amarelo, Perto = Vermelho)
                if distance > 100 then
                    Box.Color = Color3.fromRGB(0, 255, 0) -- Longe
                elseif distance > 40 then
                    Box.Color = Color3.fromRGB(255, 255, 0) -- Médio
                else
                    Box.Color = Color3.fromRGB(255, 0, 0) -- Perto
                end
                Box.Visible = true
            else
                Box.Visible = false
            end

            -- 2. BARRA DE VIDA TOTAL
            if Config.ESPHealth then
                local percent = humanoid.Health / humanoid.MaxHealth
                local barX = hrpPos.X - w / 2 - 5
                local barY = hrpPos.Y + h / 2

                HealthBar.From = Vector2.new(barX, barY)
                HealthBar.To = Vector2.new(barX, barY - (h * percent))
                HealthBar.Color = Color3.fromRGB(255 * (1 - percent), 255 * percent, 0)
                HealthBar.Visible = true
            else
                HealthBar.Visible = false
            end

            -- 3. ESQUELETO TOTALMENTE FUNCIONAL (R6 / R15)
            if Config.ESPSkeleton then
                local boneIndex = 1
                local function connectParts(part1, part2)
                    if char:FindFirstChild(part1) and char:FindFirstChild(part2) then
                        local p1, onScreen1 = Camera:WorldToViewportPoint(char[part1].Position)
                        local p2, onScreen2 = Camera:WorldToViewportPoint(char[part2].Position)

                        if onScreen1 and onScreen2 and Bones[boneIndex] then
                            Bones[boneIndex].From = Vector2.new(p1.X, p1.Y)
                            Bones[boneIndex].To = Vector2.new(p2.X, p2.Y)
                            Bones[boneIndex].Visible = true
                            boneIndex = boneIndex + 1
                        end
                    end
                end

                -- Limpar linhas extras não utilizadas
                for i = 1, #Bones do Bones[i].Visible = false end

                if Config.RigType == "R15" then
                    -- Conexões R15
                    connectParts("Head", "UpperTorso")
                    connectParts("UpperTorso", "LowerTorso")
                    -- Braço Esquerdo
                    connectParts("UpperTorso", "LeftUpperArm")
                    connectParts("LeftUpperArm", "LeftLowerArm")
                    connectParts("LeftLowerArm", "LeftHand")
                    -- Braço Direito
                    connectParts("UpperTorso", "RightUpperArm")
                    connectParts("RightUpperArm", "RightLowerArm")
                    connectParts("RightLowerArm", "RightHand")
                    -- Perna Esquerda
                    connectParts("LowerTorso", "LeftUpperLeg")
                    connectParts("LeftUpperLeg", "LeftLowerLeg")
                    connectParts("LeftLowerLeg", "LeftFoot")
                    -- Perna Direita
                    connectParts("LowerTorso", "RightUpperLeg")
                    connectParts("RightUpperLeg", "RightLowerLeg")
                    connectParts("RightLowerLeg", "RightFoot")
                else
                    -- Conexões R6 Básicas
                    connectParts("Head", "Torso")
                    connectParts("Torso", "Left Arm")
                    connectParts("Torso", "Right Arm")
                    connectParts("Torso", "Left Leg")
                    connectParts("Torso", "Right Leg")
                end
            else
                for _, line in ipairs(Bones) do line.Visible = false end
            end

        else
            Box.Visible = false
            HealthBar.Visible = false
            for _, line in ipairs(Bones) do line.Visible = false end
        end
    end)
end

-- Conectar todos os jogadores na entrada ou no carregamento inicial
for _, p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then applyESP(p) end
end
Players.PlayerAdded:Connect(function(p)
    if p ~= LocalPlayer then applyESP(p) end
end)


-- [[ LOOP DE ATUALIZAÇÃO DO AIMBOT ]]
-- Para evitar o conflito com as mecânicas internas de armas do Rivals (que destravam a mira),
-- usamos o BindToRenderStep com prioridade de execução CAMERA + 1.
-- Desta forma, a câmera é atualizada pós-processamento de recoil das armas, de forma 100% estática.

local function handleAimbotBypass()
    -- Define o centro exato da tela (mira física) para o círculo do FOV
    local screenCenter = Camera.ViewportSize / 2
    FOVCircle.Position = screenCenter
    
    local target = getClosestTarget()
    
    if target then
        local char, humanoid, hrp = getValidCharacter(target)
        if char then
            local targetPart = getTargetBodyPart(char)
            if targetPart then
                local aimPos = targetPart.Position
                
                -- Motor de Previsão Cinemática de Movimento
                if Config.Prediction then
                    local velocity = hrp.AssemblyLinearVelocity
                    local dt = 0.035
                    aimPos = aimPos + (velocity * dt)
                end
                
                -- Linha Fumegante (Tracer com origem no CENTRO DA MIRA/TELA até o alvo selecionado)
                local screenPos, onScreen = Camera:WorldToViewportPoint(aimPos)
                if onScreen and (not Config.WallCheck or isVisibleThroughWall(targetPart, char)) then
                    TargetLine.From = screenCenter
                    TargetLine.To = Vector2.new(screenPos.X, screenPos.Y)
                    TargetLine.Visible = true
                else
                    TargetLine.Visible = false
                end
                
                -- Travar a mira instantaneamente de forma AUTOMÁTICA (Sem precisar segurar botão nenhum)
                -- Vinculado em alta prioridade pós-camera do Rivals!
                if Config.AimbotEnabled then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, aimPos)
                end
            end
        else
            TargetLine.Visible = false
        end
    else
        TargetLine.Visible = false
    end
end

-- Desconecta conexões anteriores do mesmo nome se existirem para evitar sobreposição
pcall(function()
    RunService:UnbindFromRenderStep("RivalsAimbotLockBypass")
end)

-- Registrando o loop com prioridade absoluta
RunService:BindToRenderStep("RivalsAimbotLockBypass", Enum.RenderPriority.Camera.Value + 1, handleAimbotBypass)
