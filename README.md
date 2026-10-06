-- STREAMING_CHUNK:Initializing main services and variables...
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
​local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
​-- Configurações e Estados
local ESPEnabled = true
local WindowDoorESPEnabled = true
​-- STREAMING_CHUNK:Creating the Graphical User Interface (GUI)...
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "NaoDeixaiEleEntrarGUI"
ScreenGui.ResetOnSpawn = false
​-- Proteção contra detecção em alguns executors ou uso no CoreGui
pcall(function()
ScreenGui.Parent = CoreGui
end)
if not ScreenGui.Parent then
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end
​-- Detectar se é mobile ou PC para ajustar o tamanho da GUI
local isMobile = UserInputService and UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
-- Como UserInputService pode não estar disponível direto, checamos via TouchEnabled ou tamanho de tela
pcall(function()
local guiSize = isMobile and UDim2.new(0, 160, 0, 90) or UDim2.new(0, 220, 0, 120)
end)
​local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = isMobile and UDim2.new(0, 150, 0, 85) or UDim2.new(0, 220, 0, 120)
MainFrame.Position = UDim2.new(0.05, 0, 0.1, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui
​local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame
​local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "TitleLabel"
TitleLabel.Size = UDim2.new(1, 0, 0, 25)
TitleLabel.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Text = "Não Deixai Ele Entrar"
TitleLabel.TextSize = isMobile and 10 or 12
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.Parent = MainFrame
​local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = TitleLabel
​-- STREAMING_CHUNK:Creating GUI toggle buttons...
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0.9, 0, 0, 24)
ToggleButton.Position = UDim2.new(0.05, 0, 0.35, 0)
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Text = "ESP: ON"
ToggleButton.TextSize = isMobile and 11 or 13
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.Parent = MainFrame
​local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 6)
ToggleCorner.Parent = ToggleButton
​local StatusLabel = Instance.new("TextLabel")
StatusLabel.Name = "StatusLabel"
StatusLabel.Size = UDim2.new(1, 0, 0, 20)
StatusLabel.Position = UDim2.new(0, 0, 0.7, 0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
StatusLabel.Text = "Verde: Fechado | Vermelho: Aberto"
StatusLabel.TextSize = isMobile and 8 or 10
StatusLabel.Font = Enum.Font.SourceSansItalic
StatusLabel.Parent = MainFrame
​-- STREAMING_CHUNK:Implementing Toggle Logic...
ToggleButton.MouseButton1Click:Connect(function()
ESPEnabled = not ESPEnabled
if ESPEnabled then
ToggleButton.Text = "ESP: ON"
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
else
ToggleButton.Text = "ESP: OFF"
ToggleButton.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
end
end)
​-- STREAMING_CHUNK:Setting up ESP visual holders and drawing utilities...
local ESPObjects = {}
​local function createDrawing(objType, properties)
local drawing = Drawing.new(objType)
for k, v in pairs(properties) do
drawing[k] = v
end
return drawing
end
​-- Limpar ou atualizar ESP de entidades, portas e janelas
local function updateESP()
-- Remover objetos antigos se necessário ou gerenciar por tabela
for _, obj in pairs(ESPObjects) do
if obj and obj.Remove then
obj:Remove()
end
end
ESPObjects = {}
​if not ESPEnabled then return end
​-- Procurar monstro, portas e janelas no Workspace
-- Nota: Os nomes exatos dependem de como o jogo foi programado, buscamos por padrões comuns ou filhos
for _, descendant in ipairs(Workspace:GetDescendants()) do
local nameLower = string.lower(descendant.Name)
​-- Detecção do Monstro (ajuste o nome se necessário, ex: "Monster", "Entity", "Moster")
local isMonster = nameLower:find("monster") or nameLower:find("entity") or nameLower:find("creature") or nameLower:find("monstro")
local isDoorOrWindow = nameLower:find("door") or nameLower:find("window") or nameLower:find("porta") or nameLower:find("janela")
​if (isMonster or isDoorOrWindow) and descendant:IsA("BasePart") then
local primaryPart = descendant
​-- Tentar achar parte principal caso seja Model
if descendant.Parent and descendant.Parent:IsA("Model") then
primaryPart = descendant.Parent.PrimaryPart or descendant.Parent:FindFirstChildWhichIsA("BasePart") or descendant
end
​if primaryPart then
local vector, onScreen = Camera:WorldToViewportPoint(primaryPart.Position)
​if onScreen then
local box = Drawing.new("Square")
box.Visible = true
box.Size = Vector2.new(40, 60)
box.Position = Vector2.new(vector.X - 20, vector.Y - 30)
box.Filled = false
box.Thickness = 2
​local label = Drawing.new("Text")
label.Visible = true
label.Size = 14
label.Center = true
label.Outline = true
label.Font = 2
​if isMonster then
box.Color = Color3.fromRGB(255, 0, 0) -- Monstro sempre vermelho chamativo
label.Color = Color3.fromRGB(255, 0, 0)
label.Text = "[ MONSTRO ]"
label.Position = Vector2.new(vector.X, vector.Y - 45)
else
-- Lógica para Portas e Janelas: Verde fechada, Vermelho aberta
-- Verificamos se está aberta checando propriedades físicas ou nome/transparência/canCollide
local isOpen = false
if descendant:IsA("Model") then
-- Checar se alguma parte está aberta ou sem colisão
for _, p in ipairs(descendant:GetChildren()) do
if p:IsA("BasePart") and (not p.CanCollide or p.Transparency > 0.5) then
isOpen = true
end
end
else
if not descendant.CanCollide or descendant.Transparency > 0.5 then
isOpen = true
end
end
​if isOpen then
box.Color = Color3.fromRGB(255, 0, 0) -- Aberta = Vermelho
label.Color = Color3.fromRGB(255, 0, 0)
label.Text = "ABERTA"
else
box.Color = Color3.fromRGB(0, 255, 0) -- Fechada = Verde
label.Color = Color3.fromRGB(0, 255, 0)
label.Text = "FECHADA"
end
label.Position = Vector2.new(vector.X, vector.Y - 25)
end
​table.insert(ESPObjects, box)
table.insert(ESPObjects, label)
end
end
end
end
end
​-- STREAMING_CHUNK:Connecting ESP loop to RenderStepped...
RunService.RenderStepped:Connect(function()
pcall(updateESP)
end)
​print("Script Não Deixai Ele Entrar carregado com sucesso!")
