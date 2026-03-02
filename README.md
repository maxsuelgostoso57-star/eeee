-- ╔══════════════════════════════════════════════════════════╗
-- ║  🎯 AUTO FARM - VOA → TRAVA → ATACA → PRÓXIMO          ║
-- ║  Botão 🎯 na tela ou pressione K para abrir/fechar      ║
-- ╚══════════════════════════════════════════════════════════╝

-- ========== SERVIÇOS ==========
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ========== VARIÁVEIS GLOBAIS ==========
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- ========== ESTADOS DO AUTO FARM ==========
-- "idle"   = procurando NPC
-- "flying" = voando em direção ao NPC
-- "locked" = travado atacando o NPC
local FarmState = "idle"
local CurrentTarget = nil
local lockedCF = nil

-- ========== CONFIGURAÇÕES ==========
local Config = {
    AutoFarm     = false,
    PullNPCs     = false,
    ManualLock   = false,
    FlySpeed     = 250,
    AttackDistance = 100,
    FlyHeight    = 10,
    ArriveDistance = 8,
    PullSpeed    = 15,
    PullDistance = 50,
    PullStopDistance = 10,
    ClickDelay   = 0,
    MenuOpen     = false
}

-- ========== CONEXÕES ==========
local Connections = {
    autoFarm   = nil,
    noclip     = nil,
    pullNPCs   = nil,
    manualLock = nil,
}

-- ========== RESPAWN ==========
Player.CharacterAdded:Connect(function(newChar)
    Character = newChar
    Humanoid  = newChar:WaitForChild("Humanoid")
    RootPart  = newChar:WaitForChild("HumanoidRootPart")
    FarmState = "idle"
    CurrentTarget = nil
    lockedCF = nil
    wait(0.3)
    if Config.AutoFarm then startAutoFarm() end
end)

-- ========== GUI ==========
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutoFarmGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

-- ===== BOTÃO FLUTUANTE =====
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 50, 0, 50)
ToggleButton.Position = UDim2.new(0, 15, 0.5, -25)
ToggleButton.BackgroundColor3 = Color3.fromRGB(100, 200, 255)
ToggleButton.BorderSizePixel = 0
ToggleButton.Text = "🎯"
ToggleButton.TextSize = 22
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.ZIndex = 10
ToggleButton.Parent = ScreenGui
Instance.new("UICorner", ToggleButton).CornerRadius = UDim.new(1, 0)
local tbStroke = Instance.new("UIStroke", ToggleButton)
tbStroke.Color = Color3.fromRGB(255, 255, 255)
tbStroke.Thickness = 2
tbStroke.Transparency = 0.5

local tbDrag, tbDragStart, tbStartPos, tbDragInput, tbMoved = false, nil, nil, nil, false
ToggleButton.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        tbDrag = true; tbDragStart = i.Position; tbStartPos = ToggleButton.Position
        i.Changed:Connect(function()
            if i.UserInputState == Enum.UserInputState.End then tbDrag = false end
        end)
    end
end)
ToggleButton.InputChanged:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseMovement then tbDragInput = i end
end)

-- ===== FRAME PRINCIPAL =====
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 310, 0, 455)
MainFrame.Position = UDim2.new(0, 75, 0.5, -227)
MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
local mfStroke = Instance.new("UIStroke", MainFrame)
mfStroke.Color = Color3.fromRGB(100, 200, 255)
mfStroke.Thickness = 1.5
mfStroke.Transparency = 0.5

local mDrag, mDragInput, mDragStart, mStartPos = false, nil, nil, nil
MainFrame.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        mDrag = true; mDragStart = i.Position; mStartPos = MainFrame.Position
        i.Changed:Connect(function()
            if i.UserInputState == Enum.UserInputState.End then mDrag = false end
        end)
    end
end)
MainFrame.InputChanged:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseMovement then mDragInput = i end
end)

RunService.RenderStepped:Connect(function()
    if tbDrag and tbDragInput then
        local d = tbDragInput.Position - tbDragStart
        if d.Magnitude > 5 then tbMoved = true end
        ToggleButton.Position = UDim2.new(tbStartPos.X.Scale, tbStartPos.X.Offset + d.X, tbStartPos.Y.Scale, tbStartPos.Y.Offset + d.Y)
    end
    if mDrag and mDragInput then
        local d = mDragInput.Position - mDragStart
        MainFrame.Position = UDim2.new(mStartPos.X.Scale, mStartPos.X.Offset + d.X, mStartPos.Y.Scale, mStartPos.Y.Offset + d.Y)
    end
end)

-- ===== HEADER =====
local Header = Instance.new("Frame", MainFrame)
Header.Size = UDim2.new(1, 0, 0, 55)
Header.BackgroundColor3 = Color3.fromRGB(12, 12, 22)
Header.BorderSizePixel = 0
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 12)
local hFix = Instance.new("Frame", Header)
hFix.Size = UDim2.new(1, 0, 0, 12); hFix.Position = UDim2.new(0, 0, 1, -12)
hFix.BackgroundColor3 = Color3.fromRGB(12, 12, 22); hFix.BorderSizePixel = 0

local Title = Instance.new("TextLabel", Header)
Title.Size = UDim2.new(1, -60, 0, 30); Title.Position = UDim2.new(0, 15, 0, 8)
Title.BackgroundTransparency = 1; Title.Text = "🎯 AUTO FARM"
Title.TextColor3 = Color3.fromRGB(100, 200, 255); Title.Font = Enum.Font.GothamBold
Title.TextSize = 18; Title.TextXAlignment = Enum.TextXAlignment.Left

local Sub = Instance.new("TextLabel", Header)
Sub.Size = UDim2.new(1, -60, 0, 14); Sub.Position = UDim2.new(0, 15, 0, 37)
Sub.BackgroundTransparency = 1; Sub.Text = "K para fechar  •  Arraste para mover"
Sub.TextColor3 = Color3.fromRGB(100, 100, 125); Sub.Font = Enum.Font.Gotham
Sub.TextSize = 9; Sub.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = Instance.new("TextButton", Header)
CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -38, 0, 14)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 55, 55); CloseBtn.BorderSizePixel = 0
CloseBtn.Text = "✕"; CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.TextSize = 13
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(1, 0)

-- ===== STATUS BAR =====
local StatusLabel = Instance.new("TextLabel", MainFrame)
StatusLabel.Size = UDim2.new(1, -30, 0, 24)
StatusLabel.Position = UDim2.new(0, 15, 0, 63)
StatusLabel.BackgroundColor3 = Color3.fromRGB(22, 22, 38)
StatusLabel.BorderSizePixel = 0
StatusLabel.Text = "⏸ Aguardando..."
StatusLabel.TextColor3 = Color3.fromRGB(160, 160, 185)
StatusLabel.Font = Enum.Font.GothamBold
StatusLabel.TextSize = 11
StatusLabel.TextXAlignment = Enum.TextXAlignment.Center
Instance.new("UICorner", StatusLabel).CornerRadius = UDim.new(0, 6)

local function setStatus(txt, col)
    StatusLabel.Text = txt
    StatusLabel.TextColor3 = col or Color3.fromRGB(160, 160, 185)
end

-- ===== FUNÇÃO CRIAR TOGGLE =====
local function makeToggle(txt, posY)
    local b = Instance.new("TextButton", MainFrame)
    b.Size = UDim2.new(1, -30, 0, 38); b.Position = UDim2.new(0, 15, 0, posY)
    b.BackgroundColor3 = Color3.fromRGB(42, 42, 55); b.BorderSizePixel = 0
    b.Text = "❌ " .. txt .. ": OFF"
    b.TextColor3 = Color3.fromRGB(225, 225, 225)
    b.Font = Enum.Font.GothamBold; b.TextSize = 13
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
    return b
end

local AutoFarmToggle   = makeToggle("Auto Farm",     95)
local PullNPCsToggle   = makeToggle("Puxar NPCs",   143)
local ManualLockToggle = makeToggle("Travar Manual", 191)

-- ===== SEPARADOR =====
local Sep1 = Instance.new("Frame", MainFrame)
Sep1.Size = UDim2.new(1, -30, 0, 1); Sep1.Position = UDim2.new(0, 15, 0, 239)
Sep1.BackgroundColor3 = Color3.fromRGB(50, 50, 72); Sep1.BorderSizePixel = 0

-- ===== SLIDER ALTURA =====
local HeightLabel = Instance.new("TextLabel", MainFrame)
HeightLabel.Size = UDim2.new(1, -30, 0, 18); HeightLabel.Position = UDim2.new(0, 15, 0, 247)
HeightLabel.BackgroundTransparency = 1; HeightLabel.Text = "📏 Altura acima do NPC: 10 studs"
HeightLabel.TextColor3 = Color3.fromRGB(165, 165, 190); HeightLabel.Font = Enum.Font.GothamBold
HeightLabel.TextSize = 11; HeightLabel.TextXAlignment = Enum.TextXAlignment.Left

local SliderBG = Instance.new("Frame", MainFrame)
SliderBG.Size = UDim2.new(1, -30, 0, 8); SliderBG.Position = UDim2.new(0, 15, 0, 271)
SliderBG.BackgroundColor3 = Color3.fromRGB(35, 35, 50); SliderBG.BorderSizePixel = 0
Instance.new("UICorner", SliderBG).CornerRadius = UDim.new(1, 0)

local SliderFill = Instance.new("Frame", SliderBG)
SliderFill.Size = UDim2.new(0.5, 0, 1, 0); SliderFill.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
SliderFill.BorderSizePixel = 0; Instance.new("UICorner", SliderFill).CornerRadius = UDim.new(1, 0)

local SliderBtn = Instance.new("TextButton", SliderBG)
SliderBtn.Size = UDim2.new(0, 18, 0, 18); SliderBtn.Position = UDim2.new(0.5, -9, 0.5, -9)
SliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255); SliderBtn.BorderSizePixel = 0
SliderBtn.Text = ""; SliderBtn.AutoButtonColor = false
Instance.new("UICorner", SliderBtn).CornerRadius = UDim.new(1, 0)

local sDrag = false
SliderBtn.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then sDrag = true end end)
SliderBG.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then sDrag = true end end)

RunService.RenderStepped:Connect(function()
    if sDrag then
        local rel = math.clamp((UserInputService:GetMouseLocation().X - SliderBG.AbsolutePosition.X) / SliderBG.AbsoluteSize.X, 0, 1)
        Config.FlyHeight = math.floor(rel * 20)
        SliderFill.Size = UDim2.new(rel, 0, 1, 0); SliderBtn.Position = UDim2.new(rel, -9, 0.5, -9)
        HeightLabel.Text = string.format("📏 Altura acima do NPC: %d studs", Config.FlyHeight)
    end
end)

local HBF = Instance.new("Frame", MainFrame)
HBF.Size = UDim2.new(1, -30, 0, 22); HBF.Position = UDim2.new(0, 15, 0, 285); HBF.BackgroundTransparency = 1

local function makeQBtn(parent, lbl, posX, cb)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(0.23, 0, 1, 0); b.Position = UDim2.new(posX, 0, 0, 0)
    b.BackgroundColor3 = Color3.fromRGB(35, 35, 50); b.BorderSizePixel = 0
    b.Text = lbl; b.TextColor3 = Color3.fromRGB(165, 165, 190)
    b.Font = Enum.Font.Gotham; b.TextSize = 10
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 5)
    b.MouseButton1Click:Connect(cb)
end

makeQBtn(HBF, "0",  0,    function() Config.FlyHeight=0;  SliderFill.Size=UDim2.new(0,0,1,0);    SliderBtn.Position=UDim2.new(0,-9,0.5,-9);    HeightLabel.Text="📏 Altura acima do NPC: 0 studs"  end)
makeQBtn(HBF, "5",  0.26, function() Config.FlyHeight=5;  SliderFill.Size=UDim2.new(0.25,0,1,0); SliderBtn.Position=UDim2.new(0.25,-9,0.5,-9); HeightLabel.Text="📏 Altura acima do NPC: 5 studs"  end)
makeQBtn(HBF, "10", 0.52, function() Config.FlyHeight=10; SliderFill.Size=UDim2.new(0.5,0,1,0);  SliderBtn.Position=UDim2.new(0.5,-9,0.5,-9);  HeightLabel.Text="📏 Altura acima do NPC: 10 studs" end)
makeQBtn(HBF, "20", 0.78, function() Config.FlyHeight=20; SliderFill.Size=UDim2.new(1,0,1,0);    SliderBtn.Position=UDim2.new(1,-9,0.5,-9);     HeightLabel.Text="📏 Altura acima do NPC: 20 studs" end)

local Sep2 = Instance.new("Frame", MainFrame)
Sep2.Size = UDim2.new(1, -30, 0, 1); Sep2.Position = UDim2.new(0, 15, 0, 315)
Sep2.BackgroundColor3 = Color3.fromRGB(50, 50, 72); Sep2.BorderSizePixel = 0

-- ===== SLIDER PULL SPEED =====
local PullSpeedLabel = Instance.new("TextLabel", MainFrame)
PullSpeedLabel.Size = UDim2.new(1, -30, 0, 18); PullSpeedLabel.Position = UDim2.new(0, 15, 0, 323)
PullSpeedLabel.BackgroundTransparency = 1; PullSpeedLabel.Text = "🧲 Velocidade de Puxar: 15"
PullSpeedLabel.TextColor3 = Color3.fromRGB(165, 165, 190); PullSpeedLabel.Font = Enum.Font.GothamBold
PullSpeedLabel.TextSize = 11; PullSpeedLabel.TextXAlignment = Enum.TextXAlignment.Left

local SliderPBG = Instance.new("Frame", MainFrame)
SliderPBG.Size = UDim2.new(1, -30, 0, 8); SliderPBG.Position = UDim2.new(0, 15, 0, 347)
SliderPBG.BackgroundColor3 = Color3.fromRGB(35, 35, 50); SliderPBG.BorderSizePixel = 0
Instance.new("UICorner", SliderPBG).CornerRadius = UDim.new(1, 0)

local SliderPFill = Instance.new("Frame", SliderPBG)
SliderPFill.Size = UDim2.new(0, 0, 1, 0); SliderPFill.BackgroundColor3 = Color3.fromRGB(255, 120, 160)
SliderPFill.BorderSizePixel = 0; Instance.new("UICorner", SliderPFill).CornerRadius = UDim.new(1, 0)

local SliderPBtn = Instance.new("TextButton", SliderPBG)
SliderPBtn.Size = UDim2.new(0, 18, 0, 18); SliderPBtn.Position = UDim2.new(0, -9, 0.5, -9)
SliderPBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255); SliderPBtn.BorderSizePixel = 0
SliderPBtn.Text = ""; SliderPBtn.AutoButtonColor = false
Instance.new("UICorner", SliderPBtn).CornerRadius = UDim.new(1, 0)

local sPDrag = false
SliderPBtn.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then sPDrag = true end end)
SliderPBG.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then sPDrag = true end end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then sDrag = false; sPDrag = false end
end)

RunService.RenderStepped:Connect(function()
    if sPDrag then
        local rel = math.clamp((UserInputService:GetMouseLocation().X - SliderPBG.AbsolutePosition.X) / SliderPBG.AbsoluteSize.X, 0, 1)
        Config.PullSpeed = math.floor(15 + (rel * 85))
        SliderPFill.Size = UDim2.new(rel, 0, 1, 0); SliderPBtn.Position = UDim2.new(rel, -9, 0.5, -9)
        PullSpeedLabel.Text = string.format("🧲 Velocidade de Puxar: %d", Config.PullSpeed)
    end
end)

local PSBFrame = Instance.new("Frame", MainFrame)
PSBFrame.Size = UDim2.new(1, -30, 0, 22); PSBFrame.Position = UDim2.new(0, 15, 0, 361); PSBFrame.BackgroundTransparency = 1

local function setPullSpeed(v)
    Config.PullSpeed = v
    local rel = (v - 15) / 85
    SliderPFill.Size = UDim2.new(rel, 0, 1, 0); SliderPBtn.Position = UDim2.new(rel, -9, 0.5, -9)
    PullSpeedLabel.Text = string.format("🧲 Velocidade de Puxar: %d", v)
end

makeQBtn(PSBFrame, "15",  0,    function() setPullSpeed(15)  end)
makeQBtn(PSBFrame, "30",  0.26, function() setPullSpeed(30)  end)
makeQBtn(PSBFrame, "50",  0.52, function() setPullSpeed(50)  end)
makeQBtn(PSBFrame, "100", 0.78, function() setPullSpeed(100) end)

-- ===== INFO =====
local InfoLabel = Instance.new("TextLabel", MainFrame)
InfoLabel.Size = UDim2.new(1, -30, 0, 52); InfoLabel.Position = UDim2.new(0, 15, 0, 396)
InfoLabel.BackgroundTransparency = 1
InfoLabel.Text = "🔄 FLUXO AUTO FARM:\n✈️ Voa até NPC  →  🔒 Trava posição  →  ⚔️ Ataca\n💀 NPC morreu  →  próximo NPC  →  repete ∞"
InfoLabel.TextColor3 = Color3.fromRGB(90, 185, 90); InfoLabel.Font = Enum.Font.GothamBold
InfoLabel.TextSize = 10; InfoLabel.TextWrapped = true
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left

-- ========== TOGGLE MENU ==========
local function toggleMenu()
    Config.MenuOpen = not Config.MenuOpen
    MainFrame.Visible = Config.MenuOpen
    ToggleButton.BackgroundColor3 = Config.MenuOpen and Color3.fromRGB(60, 200, 100) or Color3.fromRGB(100, 200, 255)
    ToggleButton.Text = Config.MenuOpen and "✕" or "🎯"
end

ToggleButton.MouseButton1Click:Connect(function()
    if not tbMoved then toggleMenu() end
    tbMoved = false
end)
CloseBtn.MouseButton1Click:Connect(toggleMenu)
UserInputService.InputBegan:Connect(function(i, gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.K then toggleMenu() end
end)

-- ========== FUNÇÕES AUXILIARES ==========
local function notify(title, text)
    game:GetService("StarterGui"):SetCore("SendNotification", { Title=title, Text=text, Duration=2 })
end

local function IsAlive(char)
    return char and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0
end

-- ========== ENCONTRAR NPCs ==========
local function FindNearestNPC()
    local nearest, shortest = nil, math.huge
    local folder = workspace:FindFirstChild("Enemies")
    if folder then
        for _, e in ipairs(folder:GetChildren()) do
            if IsAlive(e) and e:FindFirstChild("HumanoidRootPart") then
                local d = (RootPart.Position - e.HumanoidRootPart.Position).Magnitude
                if d < shortest then shortest = d; nearest = e end
            end
        end
    end
    return nearest
end

local function Find3NearestNPCs()
    local list = {}
    local folder = workspace:FindFirstChild("Enemies")
    if folder then
        for _, e in ipairs(folder:GetChildren()) do
            if IsAlive(e) and e:FindFirstChild("HumanoidRootPart") then
                local d = (RootPart.Position - e.HumanoidRootPart.Position).Magnitude
                if d <= Config.PullDistance then
                    table.insert(list, {npc=e, distance=d})
                end
            end
        end
    end
    table.sort(list, function(a,b) return a.distance < b.distance end)
    local res = {}
    for i = 1, math.min(3, #list) do res[i] = list[i].npc end
    return res
end

-- ========== PULL NPC ==========
local function PullNPC(npc, index)
    if not npc or not npc:FindFirstChild("HumanoidRootPart") or not RootPart then return end
    local npcRoot = npc.HumanoidRootPart
    local npcH = npc:FindFirstChild("Humanoid")
    local offsets = {Vector3.new(0,0,0), Vector3.new(3,0,0), Vector3.new(-3,0,0)}
    local off = offsets[index] or Vector3.new(0,0,0)
    local targetPos = Vector3.new(RootPart.Position.X + off.X, RootPart.Position.Y - 3, RootPart.Position.Z + off.Z)
    local dist = (npcRoot.Position - targetPos).Magnitude
    if dist <= Config.PullStopDistance then
        if npcH then npcH.WalkSpeed = 0; npcH.JumpPower = 0 end
        return
    end
    local dir = (targetPos - npcRoot.Position).Unit
    local amount = math.min(dist * 0.5, Config.PullSpeed)
    pcall(function() npcRoot.CFrame = CFrame.new(npcRoot.Position + (dir * amount)) end)
    if npcH then npcH.WalkSpeed = 0; npcH.JumpPower = 0 end
end

-- ========== NOCLIP ==========
local function SetNoclip(state)
    if state then
        if Connections.noclip then return end
        Connections.noclip = RunService.Stepped:Connect(function()
            if not Character then return end
            for _, p in pairs(Character:GetDescendants()) do
                if p:IsA("BasePart") then pcall(function() p.CanCollide = false end) end
            end
        end)
    else
        if Connections.noclip then Connections.noclip:Disconnect(); Connections.noclip = nil end
        if Character then
            for _, p in pairs(Character:GetDescendants()) do
                if p:IsA("BasePart") then pcall(function() p.CanCollide = true end) end
            end
        end
    end
end

-- ========== VOO ==========
local function FlyToward(targetPos)
    if not RootPart then return end
    local g = RootPart:FindFirstChild("AFGyro")
    if not g then
        g = Instance.new("BodyGyro"); g.Name = "AFGyro"
        g.MaxTorque = Vector3.new(9e9, 9e9, 9e9); g.P = 30000; g.Parent = RootPart
    end
    local v = RootPart:FindFirstChild("AFVel")
    if not v then
        v = Instance.new("BodyVelocity"); v.Name = "AFVel"
        v.MaxForce = Vector3.new(9e9, 9e9, 9e9); v.Parent = RootPart
    end
    local dir = (targetPos - RootPart.Position).Unit
    g.CFrame = CFrame.new(RootPart.Position, targetPos)
    v.Velocity = dir * Config.FlySpeed
end

local function StopFlying()
    if not RootPart then return end
    local g = RootPart:FindFirstChild("AFGyro"); if g then g:Destroy() end
    local v = RootPart:FindFirstChild("AFVel");  if v then v:Destroy() end
end

local function HoverHere()
    if not RootPart then return end
    local v = RootPart:FindFirstChild("AFVel")
    if not v then
        v = Instance.new("BodyVelocity"); v.Name = "AFVel"
        v.MaxForce = Vector3.new(9e9, 9e9, 9e9); v.Parent = RootPart
    end
    v.Velocity = Vector3.new(0, 0, 0)
end

-- ========== TRAVAR POSIÇÃO ==========
local function LockHere()
    lockedCF = RootPart.CFrame
end

local function ApplyLock()
    if lockedCF and RootPart then
        pcall(function()
            RootPart.CFrame = lockedCF
            RootPart.Velocity = Vector3.new(0, 0, 0)
            RootPart.RotVelocity = Vector3.new(0, 0, 0)
        end)
    end
end

-- ========== FAST ATTACK ==========
local function getRemote(path)
    local ok, r = pcall(function()
        local parts = path:split("/")
        local obj = ReplicatedStorage
        for _, p in ipairs(parts) do obj = obj:WaitForChild(p, 3) end
        return obj
    end)
    return ok and r or nil
end

local RegisterAttack = getRemote("Modules/Net/RE/RegisterAttack")
local RegisterHit    = getRemote("Modules/Net/RE/RegisterHit")

local function DoAttack()
    if not IsAlive(Character) then return end
    local enemies, part1 = {}, nil
    local folder = workspace:FindFirstChild("Enemies")
    if folder then
        for _, e in ipairs(folder:GetChildren()) do
            local head = e:FindFirstChild("Head")
            if head and IsAlive(e) and Player:DistanceFromCharacter(head.Position) < Config.AttackDistance and e ~= Character then
                table.insert(enemies, {e, head}); part1 = head
            end
        end
    end
    local weapon = Character:FindFirstChildOfClass("Tool")
    if weapon and weapon:FindFirstChild("LeftClickRemote") then
        for _, ed in ipairs(enemies) do
            local e = ed[1]
            if e and e:FindFirstChild("HumanoidRootPart") and RootPart then
                local dir = (e.HumanoidRootPart.Position - Character:GetPivot().Position).Unit
                pcall(function() weapon.LeftClickRemote:FireServer(dir, 1) end)
            end
        end
    elseif #enemies > 0 and RegisterAttack and RegisterHit then
        pcall(function()
            RegisterAttack:FireServer(Config.ClickDelay)
            RegisterHit:FireServer(part1, enemies)
        end)
    end
end

-- ===========================================================
-- ========== AUTO FARM - MÁQUINA DE ESTADOS ==========
-- ===========================================================
--
-- idle   → encontra NPC → muda para "flying"
-- flying → voa até o NPC → ao chegar, trava posição → muda para "locked"
-- locked → trava posição + ataca → NPC morreu → muda para "idle"
--
-- Ciclo infinito: idle → flying → locked → idle → ...
--
function startAutoFarm()
    if Connections.autoFarm then return end
    Config.AutoFarm = true
    FarmState = "idle"
    CurrentTarget = nil
    lockedCF = nil
    SetNoclip(true)

    Connections.autoFarm = RunService.Heartbeat:Connect(function()
        if not Config.AutoFarm or not IsAlive(Character) or not RootPart then return end

        -- ---- IDLE: procurando próximo alvo ----
        if FarmState == "idle" then
            local npc = FindNearestNPC()
            if npc then
                CurrentTarget = npc
                FarmState = "flying"
                setStatus("✈️ Voando para NPC...", Color3.fromRGB(100, 190, 255))
            else
                setStatus("🔍 Procurando NPCs...", Color3.fromRGB(220, 190, 60))
                StopFlying()
            end

        -- ---- FLYING: voando até o alvo ----
        elseif FarmState == "flying" then
            -- NPC sumiu/morreu durante o voo → volta para idle
            if not IsAlive(CurrentTarget) then
                FarmState = "idle"; CurrentTarget = nil
                StopFlying()
                return
            end

            local targetPos = CurrentTarget.HumanoidRootPart.Position + Vector3.new(0, Config.FlyHeight, 0)
            local dist = (RootPart.Position - targetPos).Magnitude

            if dist > Config.ArriveDistance then
                FlyToward(targetPos) -- continua voando
            else
                -- CHEGOU! Para de voar e trava a posição
                HoverHere()
                LockHere()
                FarmState = "locked"
                setStatus("🔒 Travado! Atacando NPC...", Color3.fromRGB(255, 90, 90))
            end

        -- ---- LOCKED: travado atacando ----
        elseif FarmState == "locked" then
            -- Manter posição travada todo frame
            ApplyLock()

            -- NPC morreu → próximo ciclo
            if not IsAlive(CurrentTarget) then
                lockedCF = nil
                CurrentTarget = nil
                FarmState = "idle"
                setStatus("✅ NPC morto! Indo ao próximo...", Color3.fromRGB(80, 220, 80))
                return
            end

            -- Atacar
            local weapon = Character:FindFirstChildOfClass("Tool")
            if weapon and weapon.ToolTip ~= "Gun" then
                DoAttack()
            end
        end
    end)

    AutoFarmToggle.Text = "✅ Auto Farm: ON"
    AutoFarmToggle.BackgroundColor3 = Color3.fromRGB(35, 135, 35)
    notify("🎯 Auto Farm ATIVADO", "Voa → Trava → Ataca → Próximo ∞")
end

function stopAutoFarm()
    Config.AutoFarm = false
    FarmState = "idle"
    CurrentTarget = nil
    lockedCF = nil

    if Connections.autoFarm then Connections.autoFarm:Disconnect(); Connections.autoFarm = nil end
    StopFlying()
    SetNoclip(false)

    AutoFarmToggle.Text = "❌ Auto Farm: OFF"
    AutoFarmToggle.BackgroundColor3 = Color3.fromRGB(42, 42, 55)
    setStatus("⏸ Aguardando...", Color3.fromRGB(160, 160, 185))
    notify("❌ Auto Farm OFF", "Desativado")
end

-- ========== PUXAR NPCs ==========
function startPullNPCs()
    if Connections.pullNPCs then return end
    Config.PullNPCs = true
    Connections.pullNPCs = RunService.Heartbeat:Connect(function()
        if not Config.PullNPCs or not IsAlive(Character) or not RootPart then return end
        local npcs = Find3NearestNPCs()
        for i, npc in ipairs(npcs) do
            pcall(function() PullNPC(npc, i) end)
        end
    end)
    PullNPCsToggle.Text = "✅ Puxar NPCs: ON"
    PullNPCsToggle.BackgroundColor3 = Color3.fromRGB(35, 135, 35)
    notify("🧲 Puxar NPCs ATIVADO", "3 NPCs agrupados embaixo de você!")
end

function stopPullNPCs()
    Config.PullNPCs = false
    if Connections.pullNPCs then Connections.pullNPCs:Disconnect(); Connections.pullNPCs = nil end
    local folder = workspace:FindFirstChild("Enemies")
    if folder then
        for _, e in ipairs(folder:GetChildren()) do
            local h = e:FindFirstChild("Humanoid")
            if h then pcall(function() h.WalkSpeed = 16; h.JumpPower = 50 end) end
        end
    end
    PullNPCsToggle.Text = "❌ Puxar NPCs: OFF"
    PullNPCsToggle.BackgroundColor3 = Color3.fromRGB(42, 42, 55)
    notify("❌ Puxar NPCs OFF", "Desativado")
end

-- ========== TRAVAR MANUAL ==========
function startManualLock()
    if Connections.manualLock then return end
    Config.ManualLock = true
    if FarmState ~= "locked" then lockedCF = RootPart.CFrame end
    Connections.manualLock = RunService.Heartbeat:Connect(function()
        if not Config.ManualLock or not RootPart then return end
        if FarmState ~= "locked" then
            pcall(function()
                RootPart.CFrame = lockedCF
                RootPart.Velocity = Vector3.new(0, 0, 0)
                RootPart.RotVelocity = Vector3.new(0, 0, 0)
            end)
        end
    end)
    ManualLockToggle.Text = "✅ Travar Manual: ON"
    ManualLockToggle.BackgroundColor3 = Color3.fromRGB(175, 45, 45)
    notify("🔒 Posição TRAVADA", "Player parado no lugar!")
end

function stopManualLock()
    Config.ManualLock = false
    if Connections.manualLock then Connections.manualLock:Disconnect(); Connections.manualLock = nil end
    if FarmState ~= "locked" then lockedCF = nil end
    ManualLockToggle.Text = "❌ Travar Manual: OFF"
    ManualLockToggle.BackgroundColor3 = Color3.fromRGB(42, 42, 55)
    notify("🔓 Posição DESTRAVADA", "Pode se mover!")
end

-- ========== EVENTOS DOS BOTÕES ==========
AutoFarmToggle.MouseButton1Click:Connect(function()
    if Config.AutoFarm then stopAutoFarm() else startAutoFarm() end
end)
PullNPCsToggle.MouseButton1Click:Connect(function()
    if Config.PullNPCs then stopPullNPCs() else startPullNPCs() end
end)
ManualLockToggle.MouseButton1Click:Connect(function()
    if Config.ManualLock then stopManualLock() else startManualLock() end
end)

-- ========== INICIALIZAÇÃO ==========
notify("🎯 AUTO FARM CARREGADO", "Clique em 🎯 ou pressione K para abrir!")

print("╔══════════════════════════════════════════╗")
print("║   ✅ AUTO FARM - FLUXO AUTOMÁTICO        ║")
print("╠══════════════════════════════════════════╣")
print("║  ✈️  Voa até o NPC                       ║")
print("║  🔒 Trava posição ao chegar               ║")
print("║  ⚔️  Ataca automaticamente                ║")
print("║  💀 NPC morreu → vai pro próximo          ║")
print("║  🔄 Repete infinitamente                  ║")
print("╠══════════════════════════════════════════╣")
print("║  🎯 Botão flutuante ou K para abrir       ║")
print("╚══════════════════════════════════════════╝")
