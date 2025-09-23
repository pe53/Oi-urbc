-- LocalScript (colar em StarterGui)
-- Painel laranja/preto/cinza — Aba Player intacta + Aba ESP com Name/Box/Health/Distance
-- Foi escrito para ser robusto e minimizar erros comuns.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- layout / constantes
local DEFAULT_JUMP = 50
local DEFAULT_SPEED = 16
local LEFT_PADDING = 10
local BOTTOM_OFFSET = 260
local TAB_WIDTH = 120
local TOP_HEIGHT = 40
local FLY_URL = "https://rawscripts.net/raw/Universal-Script-Gui-Fly-v3-37111"

-- estados (ABA PLAYER) — mantidos e inicializados
local jumpEnabled = false
local infJumpEnabled = false
local speedEnabled = false
local noclipEnabled = false
local infJumpConn = nil
local noclipConn = nil
local flyAtivo = false
local flyCreatedList = nil

-- remove GUI antiga (se existir)
local old = playerGui:FindFirstChild("PainelUI")
if old then old:Destroy() end

-- ScreenGui (pai no PlayerGui)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PainelUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Main frame (visual laranja/preto/cinza)
local main = Instance.new("Frame")
main.Name = "Main"
main.Size = UDim2.new(0.6, 0, 0.7, 0)
main.Position = UDim2.new(0.5, 0, 0.55, 0)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
main.Parent = screenGui

-- Top bar (laranja)
local topBar = Instance.new("Frame", main)
topBar.Size = UDim2.new(1, 0, 0, TOP_HEIGHT)
topBar.Position = UDim2.new(0, 0, 0, 0)
topBar.BackgroundColor3 = Color3.fromRGB(255, 140, 0)

local title = Instance.new("TextLabel", topBar)
title.Size = UDim2.new(1, -120, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Painel"
title.TextColor3 = Color3.fromRGB(0,0,0)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
title.TextXAlignment = Enum.TextXAlignment.Left

local removerBtn = Instance.new("TextButton", topBar)
removerBtn.Size = UDim2.new(0, 40, 1, 0)
removerBtn.Position = UDim2.new(1, -90, 0, 0)
removerBtn.BackgroundColor3 = Color3.fromRGB(100,100,100)
removerBtn.Text = "-"
removerBtn.TextColor3 = Color3.fromRGB(1,1,1)
removerBtn.Font = Enum.Font.SourceSansBold
removerBtn.TextSize = 24

local fecharBtn = Instance.new("TextButton", topBar)
fecharBtn.Size = UDim2.new(0, 40, 1, 0)
fecharBtn.Position = UDim2.new(1, -45, 0, 0)
fecharBtn.BackgroundColor3 = Color3.fromRGB(200,50,50)
fecharBtn.Text = "X"
fecharBtn.TextColor3 = Color3.fromRGB(1,1,1)
fecharBtn.Font = Enum.Font.SourceSansBold

-- Bolinha para reabrir/minimizar (igual ao que você usava)
local openDot = Instance.new("TextButton", screenGui)
openDot.Size = UDim2.new(0, 50, 0, 50)
openDot.AnchorPoint = Vector2.new(0,1)
openDot.Position = UDim2.new(0, LEFT_PADDING, 1, -BOTTOM_OFFSET)
openDot.BackgroundColor3 = Color3.fromRGB(255,140,0)
openDot.Text = "O"
openDot.TextColor3 = Color3.fromRGB(0,0,0)
openDot.Font = Enum.Font.SourceSansBold
openDot.Visible = false

-- Left panel (abas) com scroll
local leftPanel = Instance.new("ScrollingFrame", main)
leftPanel.Name = "LeftPanel"
leftPanel.Size = UDim2.new(0, TAB_WIDTH, 1, -TOP_HEIGHT)
leftPanel.Position = UDim2.new(0, 0, 0, TOP_HEIGHT)
leftPanel.BackgroundColor3 = Color3.fromRGB(100,100,100)
leftPanel.ScrollBarThickness = 6
leftPanel.AutomaticCanvasSize = Enum.AutomaticSize.Y

local leftLayout = Instance.new("UIListLayout", leftPanel)
leftLayout.SortOrder = Enum.SortOrder.LayoutOrder
leftLayout.Padding = UDim.new(0,6)
local leftPad = Instance.new("UIPadding", leftPanel)
leftPad.PaddingTop = UDim.new(0,6)
leftPad.PaddingLeft = UDim.new(0,6)

-- Content area (direita)
local contentArea = Instance.new("Frame", main)
contentArea.Name = "ContentArea"
contentArea.Size = UDim2.new(1, -TAB_WIDTH, 1, -TOP_HEIGHT)
contentArea.Position = UDim2.new(0, TAB_WIDTH, 0, TOP_HEIGHT)
contentArea.BackgroundColor3 = Color3.fromRGB(80,80,80)

-- tabela de abas
local tabs = {}

-- cria aba (botão lateral + ScrollingFrame no conteúdo)
local function createTab(tabName)
local btn = Instance.new("TextButton")
btn.Size = UDim2.new(1, 0, 0, 40)
btn.BackgroundColor3 = Color3.fromRGB(120,120,120)
btn.TextColor3 = Color3.fromRGB(1,1,1)
btn.Font = Enum.Font.SourceSansBold
btn.TextSize = 18
btn.Text = tabName
btn.LayoutOrder = #leftPanel:GetChildren() + 1
btn.Parent = leftPanel

local frame = Instance.new("ScrollingFrame")  
frame.Size = UDim2.new(1, 0, 1, 0)  
frame.Position = UDim2.new(0, 0, 0, 0)  
frame.BackgroundTransparency = 1  
frame.ScrollBarThickness = 6  
frame.AutomaticCanvasSize = Enum.AutomaticSize.Y  
frame.Visible = false  
frame.Parent = contentArea  

local layout = Instance.new("UIListLayout", frame)  
layout.SortOrder = Enum.SortOrder.LayoutOrder  
layout.Padding = UDim.new(0, 8)  
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center  

local pad = Instance.new("UIPadding", frame)  
pad.PaddingTop = UDim.new(0, 12)  
pad.PaddingBottom = UDim.new(0, 12)  

btn.MouseButton1Click:Connect(function()  
    for _, t in ipairs(tabs) do  
        t.frame.Visible = false  
    end  
    frame.Visible = true  
end)  

table.insert(tabs, {name = tabName, button = btn, frame = frame})  
return frame

end

-- helpers para controles (usando UIListLayout para posicionar)
local function createWideButton(parent, text)
local bt = Instance.new("TextButton")
bt.Size = UDim2.new(0.9, 0, 0, 36)
bt.BackgroundColor3 = Color3.fromRGB(255,140,0)
bt.Text = text
bt.TextColor3 = Color3.fromRGB(0,0,0)
bt.Font = Enum.Font.SourceSansBold
bt.TextSize = 18
bt.Parent = parent
bt.LayoutOrder = #parent:GetChildren() + 1
return bt
end

local function createWideTextBox(parent, placeholder)
local tb = Instance.new("TextBox")
tb.Size = UDim2.new(0.9, 0, 0, 36)
tb.BackgroundColor3 = Color3.fromRGB(255,255,255)
tb.Text = ""
tb.PlaceholderText = placeholder or ""
tb.TextColor3 = Color3.fromRGB(0,0,0)
tb.Font = Enum.Font.SourceSans
tb.TextSize = 18
tb.Parent = parent
tb.LayoutOrder = #parent:GetChildren() + 1
tb.ClearTextOnFocus = false
return tb
end

-- =========================
-- ABA: PLAYER (NÃO ALTEREI A LÓGICA) — uso a mesma estrutura funcional
-- =========================
local playerTab = createTab("Player")
playerTab.Visible = true

-- Jump (igual ao que estava)
local jumpBox = createWideTextBox(playerTab, "Valor do Jump (ex: 100)")
local jumpBtn = createWideButton(playerTab, "Jump (Ativar/Desativar)")
jumpBtn.MouseButton1Click:Connect(function()
local char = player.Character
if not char then return end
local humanoid = char:FindFirstChildOfClass("Humanoid")
if not humanoid then return end
if not jumpEnabled then
local val = tonumber(jumpBox.Text) or DEFAULT_JUMP
humanoid.UseJumpPower = true
humanoid.JumpPower = val
jumpEnabled = true
jumpBtn.Text = "Jump (Ativado)"
else
humanoid.JumpPower = DEFAULT_JUMP
jumpEnabled = false
jumpBtn.Text = "Jump (Desativado)"
end
end)

-- Jump Infinito (mesma ideia)
local infJumpBox = createWideTextBox(playerTab, "Altura do Jump Infinito")
local infJumpBtn = createWideButton(playerTab, "Jump Infinito (Ativar/Desativar)")
infJumpBtn.MouseButton1Click:Connect(function()
if not infJumpEnabled then
infJumpEnabled = true
infJumpBtn.Text = "Jump Infinito (Ativado)"
if not infJumpConn then
infJumpConn = UserInputService.JumpRequest:Connect(function()
if infJumpEnabled and player.Character then
local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
if humanoid then
local val = tonumber(infJumpBox.Text) or DEFAULT_JUMP
humanoid.UseJumpPower = true
humanoid.JumpPower = val
humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
end
end
end)
end
else
infJumpEnabled = false
infJumpBtn.Text = "Jump Infinito (Desativado)"
if infJumpConn then pcall(function() infJumpConn:Disconnect() end) infJumpConn = nil end
end
end)

-- Speed
local speedBox = createWideTextBox(playerTab, "Velocidade do Player")
local speedBtn = createWideButton(playerTab, "Speed (Ativar/Desativar)")
speedBtn.MouseButton1Click:Connect(function()
local char = player.Character
if not char then return end
local humanoid = char:FindFirstChildOfClass("Humanoid")
if not humanoid then return end
if not speedEnabled then
local val = tonumber(speedBox.Text) or DEFAULT_SPEED
humanoid.WalkSpeed = val
speedEnabled = true
speedBtn.Text = "Speed (Ativado)"
else
humanoid.WalkSpeed = DEFAULT_SPEED
speedEnabled = false
speedBtn.Text = "Speed (Desativado)"
end
end)

-- Gravity (aba Player)
local gravityBox = createWideTextBox(playerTab, tostring(Workspace.Gravity))
local gravityBtn = createWideButton(playerTab, "Definir Gravidade")
gravityBtn.MouseButton1Click:Connect(function()
local val = tonumber(gravityBox.Text)
if val then Workspace.Gravity = val end
end)

-- NoClip
local noclipBtn = createWideButton(playerTab, "NoClip (Ativar/Desativar)")
noclipBtn.MouseButton1Click:Connect(function()
if not noclipEnabled then
noclipEnabled = true
noclipBtn.Text = "NoClip (Desativar)"
if not noclipConn then
noclipConn = RunService.Stepped:Connect(function()
if noclipEnabled and player.Character then
for _, part in pairs(player.Character:GetDescendants()) do
if part:IsA("BasePart") then pcall(function() part.CanCollide = false end) end
end
end
end)
end
else
noclipEnabled = false
noclipBtn.Text = "NoClip (Ativar)"
if noclipConn then pcall(function() noclipConn:Disconnect() end) noclipConn = nil end
if player.Character then
for _, part in pairs(player.Character:GetDescendants()) do
if part:IsA("BasePart") then pcall(function() part.CanCollide = true end) end
end
end
end
end)

-- Função utilitária de limpeza do Fly (mantida)
local function tryCleanupFly()
local function checkAndDestroy(container)
for _, child in pairs(container:GetChildren()) do
if child:IsA("ScreenGui") or child:IsA("Frame") or child:IsA("Folder") then
local lname = tostring(child.Name):lower()
if lname:find("fly") or lname:find("universal") or lname:find("flying") then
pcall(function() child:Destroy() end)
else
for _, d in pairs(child:GetDescendants()) do
if d:IsA("TextLabel") or d:IsA("TextButton") or d:IsA("TextBox") then
local txt = tostring(d.Text or d.PlaceholderText or ""):lower()
if txt:find("fly") or txt:find("voar") then
pcall(function() child:Destroy() end); break
end
end
end
end
end
end
end
pcall(function() checkAndDestroy(playerGui) end)
pcall(function() checkAndDestroy(game:GetService("CoreGui")) end)
if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
pcall(function() player.Character.HumanoidRootPart.Anchored = false end)
end
if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
pcall(function() player.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = DEFAULT_SPEED end)
pcall(function() player.Character:FindFirstChildOfClass("Humanoid").JumpPower = DEFAULT_JUMP end)
end
end

-- Fly (botão) — mesmíssima ideia que você usava
local flyBtn = createWideButton(playerTab, "Fly")
flyBtn.MouseButton1Click:Connect(function()
if not flyAtivo then
flyAtivo = true
flyBtn.Text = "Desativar Fly"
local ok, content = pcall(function() return game:HttpGet(FLY_URL) end)
if not ok or not content or content == "" then
flyBtn.Text = "Fly (Erro baixar)"
flyAtivo = false
task.delay(2, function() if not flyAtivo then flyBtn.Text = "Fly" end end)
return
end
local ok2, err = pcall(function() loadstring(content)() end)
if not ok2 then
flyBtn.Text = "Fly (Erro executar)"; flyAtivo = false
task.delay(2, function() if not flyAtivo then flyBtn.Text = "Fly" end end)
warn("[Painel] Erro ao executar fly:", err); return
end
flyCreatedList = {}
task.wait(0.5)
for _, g in pairs(playerGui:GetChildren()) do
if g:IsA("ScreenGui") and (string.find(string.lower(g.Name), "fly") or string.find(string.lower(g.Name), "universal")) then
table.insert(flyCreatedList, g)
end
end
for _, g in pairs(game:GetService("CoreGui"):GetChildren()) do
if g:IsA("ScreenGui") and (string.find(string.lower(g.Name), "fly") or string.find(string.lower(g.Name), "universal")) then
table.insert(flyCreatedList, g)
end
end
if #flyCreatedList == 0 then flyCreatedList = nil end
else
flyAtivo = false
flyBtn.Text = "Fly"
if flyCreatedList then
for _, g in ipairs(flyCreatedList) do pcall(function() g:Destroy() end) end
flyCreatedList = nil
end
tryCleanupFly()
end
end)

-- =========================
-- ABA: ESP (só mexi aqui — as 4 funções: Name / Box / Health / Distance)
-- =========================
local espTab = createTab("ESP")
local espEnabled = { Name = false, Box = false, Health = false, Distance = false }

local function createEspButton(parent, text, key)
local btn = createWideButton(parent, text)
btn.MouseButton1Click:Connect(function()
espEnabled[key] = not espEnabled[key]
btn.Text = (espEnabled[key] and "[ON] "..text) or ("[OFF] "..text)
end)
return btn
end

createEspButton(espTab, "ESP Name", "Name")
createEspButton(espTab, "ESP Box", "Box")
createEspButton(espTab, "ESP Health", "Health")
createEspButton(espTab, "ESP Distance", "Distance")

-- Detecta se Drawing API existe (muitos executores suportam)
local drawingSupported = false
pcall(function()
local test = Drawing.new("Text")
if test and test.Remove then
test:Remove()
drawingSupported = true
end
end)

local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local drawings = {} -- keyed by player

local function createDrawingForPlayer(pl)
if not drawingSupported then return end
if drawings[pl] then return end
local ok, obj = pcall(function()
local t = {}
t.Name = Drawing.new("Text")
t.Box = Drawing.new("Square")
t.Health = Drawing.new("Text")
t.Distance = Drawing.new("Text")

t.Name.Size = 13; t.Name.Color = Color3.fromRGB(0,255,0); t.Name.Center = true; t.Name.Outline = true  
    t.Box.Thickness = 1; t.Box.Color = Color3.fromRGB(255,255,255); t.Box.Filled = false  
    t.Health.Size = 13; t.Health.Color = Color3.fromRGB(255,0,0); t.Health.Center = true; t.Health.Outline = true  
    t.Distance.Size = 13; t.Distance.Color = Color3.fromRGB(0,200,255); t.Distance.Center = true; t.Distance.Outline = true  
    return t  
end)  
if ok and obj then drawings[pl] = obj else drawings[pl] = nil end

end

local function removeDrawingForPlayer(pl)
if not drawings[pl] then return end
pcall(function()
for _, d in pairs(drawings[pl]) do if d and d.Remove then d:Remove() end end
end)
drawings[pl] = nil
end

Players.PlayerRemoving:Connect(function(p) removeDrawingForPlayer(p) end)

RunService.RenderStepped:Connect(function()
if not LocalPlayer or not LocalPlayer.Character then return end
local localRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
for _, pl in ipairs(Players:GetPlayers()) do
if pl ~= LocalPlayer and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
createDrawingForPlayer(pl)
local root = pl.Character.HumanoidRootPart
local humanoid = pl.Character:FindFirstChildOfClass("Humanoid")
local pos, onScreen = Camera:WorldToViewportPoint(root.Position)

local d = drawings[pl]  
        if d then  
            -- Name  
            d.Name.Visible = espEnabled.Name and onScreen  
            if d.Name.Visible then  
                pcall(function() d.Name.Text = pl.Name; d.Name.Position = Vector2.new(pos.X, pos.Y - 40) end)  
            end  

            -- Box  
            d.Box.Visible = espEnabled.Box and onScreen  
            if d.Box.Visible then  
                pcall(function()  
                    local dist = localRoot and (localRoot.Position - root.Position).Magnitude or 20  
                    local height = math.clamp(2000 / (dist + 1), 30, 180)  
                    local width = math.max(20, height / 2)  
                    d.Box.Size = Vector2.new(width, height)  
                    d.Box.Position = Vector2.new(pos.X - width/2, pos.Y - height/2)  
                end)  
            end  

            -- Health  
            d.Health.Visible = espEnabled.Health and onScreen  
            if d.Health.Visible and humanoid then  
                pcall(function() d.Health.Text = "HP: "..math.floor(humanoid.Health); d.Health.Position = Vector2.new(pos.X, pos.Y + 40) end)  
            end  

            -- Distance  
            d.Distance.Visible = espEnabled.Distance and onScreen  
            if d.Distance.Visible and localRoot then  
                pcall(function()  
                    local distance = (localRoot.Position - root.Position).Magnitude  
                    d.Distance.Text = "["..math.floor(distance).."m]"  
                    d.Distance.Position = Vector2.new(pos.X, pos.Y + 55)  
                end)  
            end  
        end  
    else  
        removeDrawingForPlayer(pl)  
    end  
end

end)

-- Topbar actions (mantidos)
fecharBtn.MouseButton1Click:Connect(function()
main.Visible = false
openDot.Visible = true
end)

removerBtn.MouseButton1Click:Connect(function()
-- tenta desconectar conexões de Jump/NoClip/Fly e destroi GUI
infJumpEnabled = false
noclipEnabled = false
if infJumpConn then pcall(function() infJumpConn:Disconnect() end) infJumpConn = nil end
if noclipConn then pcall(function() noclipConn:Disconnect() end) noclipConn = nil end
if flyAtivo then flyAtivo = false; tryCleanupFly() end
screenGui:Destroy()
end)

openDot.MouseButton1Click:Connect(function()
main.Visible = true
openDot.Visible = false
end)

-- garante posição correta da bolinha
local function atualizarPosicaoBolinha()
openDot.Position = UDim2.new(0, LEFT_PADDING, 1, -BOTTOM_OFFSET)
end
screenGui:GetPropertyChangedSignal("AbsoluteSize"):Connect(atualizarPosicaoBolinha)
task.defer(atualizarPosicaoBolinha)

-- mostra a aba Player por padrão (garante compatibilidade)
for _, t in ipairs(tabs) do
t.frame.Visible = false
end
if tabs[1] then tabs[1].frame.Visible = true end

