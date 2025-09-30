--// DeltaEditor Movement & Basic UI Module
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- SETTINGS
local settings = {
    flyEnabled = false,
    flySpeed = 50,
    noclipEnabled = false,
    walkSpeed = 16,
    jumpPower = 50,
    bunnyJump = false,
    tpDistance = 50,
    mapBrightness = 1,
}

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 400, 0, 500)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(35,35,35)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = ScreenGui

-- Scroll Frame
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1,0,1,0)
scrollFrame.CanvasSize = UDim2.new(0,0,2,0)
scrollFrame.ScrollBarThickness = 8
scrollFrame.Parent = mainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0,5)
UIListLayout.Parent = scrollFrame

-- Helpers
local function createButton(name, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 35)
    btn.Text = name
    btn.BackgroundColor3 = Color3.fromRGB(70,70,70)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Parent = scrollFrame
    btn.MouseButton1Click:Connect(callback)
end

local function createToggle(name, variable)
    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(1, -20, 0, 35)
    toggle.Text = name.." [OFF]"
    toggle.BackgroundColor3 = Color3.fromRGB(70,70,70)
    toggle.TextColor3 = Color3.fromRGB(255,255,255)
    toggle.Parent = scrollFrame
    toggle.MouseButton1Click:Connect(function()
        settings[variable] = not settings[variable]
        toggle.Text = name.." ["..(settings[variable] and "ON" or "OFF").."]"
    end)
end

local function createSlider(name, variable, min, max)
    local sliderFrame = Instance.new("Frame")
    sliderFrame.Size = UDim2.new(1, -20, 0, 40)
    sliderFrame.BackgroundTransparency = 1
    sliderFrame.Parent = scrollFrame

    local sliderLabel = Instance.new("TextLabel")
    sliderLabel.Size = UDim2.new(1,0,0,20)
    sliderLabel.Text = name.." : "..settings[variable]
    sliderLabel.TextColor3 = Color3.fromRGB(255,255,255)
    sliderLabel.BackgroundTransparency = 1
    sliderLabel.Parent = sliderFrame

    local slider = Instance.new("TextButton")
    slider.Size = UDim2.new(1,0,0,20)
    slider.Position = UDim2.new(0,0,0,20)
    slider.Text = ""
    slider.BackgroundColor3 = Color3.fromRGB(100,100,100)
    slider.Parent = sliderFrame

    local dragging = false
    slider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
        end
    end)
    slider.InputEnded:Connect(function(input)
        dragging = false
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local mouseX = input.Position.X
            local framePos = sliderFrame.AbsolutePosition.X
            local frameSize = sliderFrame.AbsoluteSize.X
            local percent = math.clamp((mouseX - framePos)/frameSize,0,1)
            settings[variable] = math.floor(min + (max-min)*percent)
            sliderLabel.Text = name.." : "..settings[variable]
        end
    end)
end

-- CREATE CONTROLS
createToggle("Fly", "flyEnabled")
createSlider("Fly Speed", "flySpeed", 1, 100)
createToggle("NoClip", "noclipEnabled")
createSlider("WalkSpeed", "walkSpeed", 16, 100)
createSlider("Jump Power", "jumpPower", 50, 500)
createToggle("Bunny Jump", "bunnyJump")
createSlider("TP Distance", "tpDistance", 10, 500)
createButton("TP Forward", function()
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = hrp.CFrame + hrp.CFrame.LookVector * settings.tpDistance
    end
end)
createButton("Respawn", function()
    if LocalPlayer.Character then LocalPlayer:LoadCharacter() end
end)
createButton("Rejoin", function()
    -- 実装はゲーム依存
end)
createButton("Leave", function()
    -- 実装はゲーム依存
end)
createSlider("Map Brightness", "mapBrightness", 0, 10)

-- RUN SERVICE
RunService.RenderStepped:Connect(function()
    if LocalPlayer.Character then
        local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if settings.flyEnabled and hrp then
            hrp.Velocity = Vector3.new(0,0,0)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then hrp.CFrame = hrp.CFrame + hrp.CFrame.LookVector * (settings.flySpeed/10) end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then hrp.CFrame = hrp.CFrame - hrp.CFrame.LookVector * (settings.flySpeed/10) end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then hrp.CFrame = hrp.CFrame - hrp.CFrame.RightVector * (settings.flySpeed/10) end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then hrp.CFrame = hrp.CFrame + hrp.CFrame.RightVector * (settings.flySpeed/10) end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then hrp.CFrame = hrp.CFrame + Vector3.new(0,settings.flySpeed/10,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then hrp.CFrame = hrp.CFrame - Vector3.new(0,settings.flySpeed/10,0) end
        end
        if settings.noclipEnabled and humanoid then
            for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
        if humanoid then
            humanoid.WalkSpeed = settings.walkSpeed
            humanoid.JumpPower = settings.jumpPower
            if settings.bunnyJump and humanoid.FloorMaterial ~= Enum.Material.Air then
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end
    end
end)
-- Combat & ESP Module (DeltaEditor-friendly)
-- Place as LocalScript in StarterPlayerScripts
-- Client-side visual/assist only. No server manipulation.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Workspace = game:GetService("Workspace")

-- ================= SETTINGS =================
local cfg = {
    espOn = false,
    showName = true,
    showHealth = false,
    showDistance = true,
    wallhackAlpha = 0.6,        -- 0..1 (1 = opaque, 0 = invisible) we use LocalTransparencyModifier = 1 - alpha
    aimbotStrength = 30,       -- 0..100 (0 = off)
    aimbotRange = 100,         -- studs/meters
    aimbotRedCircle = false,
    aimbotLines = false,
    aimbotIgnoreSameTeam = true,
    aimbotNearestPriority = true,
    uiPosition = UDim2.new(0.7, 0, 0.06, 0),
    uiColor = Color3.fromRGB(255, 150, 120),
}

-- ================= GUI BUILD =================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CombatHelperGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Name = "Main"
main.Size = UDim2.new(0, 360, 0, 380)
main.Position = cfg.uiPosition
main.BackgroundColor3 = Color3.fromRGB(40,40,50)
main.BorderSizePixel = 0
main.Parent = screenGui
main.Active = true

-- header (drag)
local header = Instance.new("TextLabel", main)
header.Size = UDim2.new(1,0,0,28)
header.BackgroundTransparency = 1
header.Text = "Combat Helper"
header.TextColor3 = Color3.fromRGB(240,240,240)
header.Font = Enum.Font.SourceSansBold
header.TextSize = 16
header.TextXAlignment = Enum.TextXAlignment.Left
header.TextYAlignment = Enum.TextYAlignment.Center
header.Padding = UDim.new(0,8)

-- minimize & close
local btnMin = Instance.new("TextButton", main)
btnMin.Size = UDim2.new(0,28,0,24)
btnMin.Position = UDim2.new(1, -60, 0, 2)
btnMin.Text = "—"
btnMin.Font = Enum.Font.SourceSansBold
btnMin.TextSize = 18

local btnClose = Instance.new("TextButton", main)
btnClose.Size = UDim2.new(0,28,0,24)
btnClose.Position = UDim2.new(1, -30, 0, 2)
btnClose.Text = "X"
btnClose.Font = Enum.Font.SourceSansBold
btnClose.TextSize = 14

-- scrolling frame for controls
local scroll = Instance.new("ScrollingFrame", main)
scroll.Size = UDim2.new(1, -12, 1, -40)
scroll.Position = UDim2.new(0,6,0,36)
scroll.CanvasSize = UDim2.new(0,0,3,0)
scroll.ScrollBarThickness = 8
scroll.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", scroll)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0,6)

-- helper builders
local function makeLabel(text)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, 0, 0, 20)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(230,230,230)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Font = Enum.Font.SourceSans
    lbl.TextSize = 14
    return lbl
end

local function makeToggle(text, start)
    local fr = Instance.new("Frame")
    fr.Size = UDim2.new(1,0,0,28)
    fr.BackgroundTransparency = 1

    local lbl = Instance.new("TextLabel", fr)
    lbl.Size = UDim2.new(0.6,0,1,0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(220,220,220)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Font = Enum.Font.SourceSans
    lbl.TextSize = 14

    local btn = Instance.new("TextButton", fr)
    btn.Size = UDim2.new(0, 80, 0, 22)
    btn.Position = UDim2.new(1, -84, 0, 3)
    btn.Text = start and "ON" or "OFF"
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 14

    return fr, btn
end

local function makeNumberCtrl(name, min, max, init)
    local fr = Instance.new("Frame")
    fr.Size = UDim2.new(1,0,0,34)
    fr.BackgroundTransparency = 1

    local lbl = Instance.new("TextLabel", fr)
    lbl.Size = UDim2.new(0.5,0,1,0)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = Color3.fromRGB(220,220,220)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Font = Enum.Font.SourceSans
    lbl.TextSize = 14

    local minus = Instance.new("TextButton", fr)
    minus.Size = UDim2.new(0,30,0,24)
    minus.Position = UDim2.new(0.5, 8, 0, 5)
    minus.Text = "−"
    minus.Font = Enum.Font.SourceSansBold

    local val = Instance.new("TextLabel", fr)
    val.Size = UDim2.new(0,60,0,24)
    val.Position = UDim2.new(0.5, 48, 0, 5)
    val.BackgroundTransparency = 1
    val.Text = tostring(init)
    val.Font = Enum.Font.SourceSansBold
    val.TextSize = 14

    local plus = Instance.new("TextButton", fr)
    plus.Size = UDim2.new(0,30,0,24)
    plus.Position = UDim2.new(0.5, 116, 0, 5)
    plus.Text = "+"
    plus.Font = Enum.Font.SourceSansBold

    return fr, minus, plus, val
end

-- ================= BUILD UI CONTROLS =================
-- ESP toggles
local espFrame, espBtn = makeToggle("ESP 全表示", cfg.espOn)
espFrame.Parent = scroll

local nameFrame, nameBtn = makeToggle("Name 表示", cfg.showName)
nameFrame.Parent = scroll

local hpFrame, hpBtn = makeToggle("体力表示", cfg.showHealth)
hpFrame.Parent = scroll

local distFrame, distBtn = makeToggle("距離表示", cfg.showDistance)
distFrame.Parent = scroll

-- wallhack control
local wallLabel = makeLabel("Wallhack 透過度")
wallLabel.Parent = scroll
local wallFrame, wallMinus, wallPlus, wallVal = makeNumberCtrl("Wallhack Alpha (0-100)", 0, 100, math.floor(cfg.wallhackAlpha*100))
wallFrame.Parent = scroll

-- Aimbot controls
local aimFrame, aimBtn = makeToggle("Aimbot ON/OFF", cfg.aimbotStrength > 0)
aimFrame.Parent = scroll

local aStrFrame, aStrMinus, aStrPlus, aStrVal = makeNumberCtrl("Aimbot 強度 (0-100)", 0, 100, cfg.aimbotStrength)
aStrFrame.Parent = scroll

local aRangeFrame, arMinus, arPlus, arVal = makeNumberCtrl("Aimbot 範囲 (m)", 10, 1000, cfg.aimbotRange)
aRangeFrame.Parent = scroll

local aOpt1Frame, aOpt1Btn = makeToggle("aimbot追加1: 赤い円表示", cfg.aimbotRedCircle)
aOpt1Frame.Parent = scroll
local aOpt2Frame, aOpt2Btn = makeToggle("aimbot追加2: 線表示", cfg.aimbotLines)
aOpt2Frame.Parent = scroll
local aOpt3Frame, aOpt3Btn = makeToggle("aimbot追加3: 同チーム除外", cfg.aimbotIgnoreSameTeam)
aOpt3Frame.Parent = scroll
local aOpt4Frame, aOpt4Btn = makeToggle("aimbot追加4: 近距離優先", cfg.aimbotNearestPriority)
aOpt4Frame.Parent = scroll

-- Player List
local plLabel = makeLabel("Player Lists")
plLabel.Parent = scroll
local plContainer = Instance.new("Frame", scroll)
plContainer.Size = UDim2.new(1,0,0,160)
plContainer.BackgroundTransparency = 1
local plScroll = Instance.new("ScrollingFrame", plContainer)
plScroll.Size = UDim2.new(1,0,1,0)
plScroll.CanvasSize = UDim2.new(0,0,2,0)
plScroll.ScrollBarThickness = 6
local plLayout = Instance.new("UIListLayout", plScroll)
plLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- helper: create player row
local function makePlayerRow(pl)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -8, 0, 28)
    btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    btn.TextColor3 = Color3.fromRGB(240,240,240)
    btn.Text = pl.Name
    btn.Parent = plScroll

    local menu = Instance.new("Frame") -- small inline options
    menu.Size = UDim2.new(0, 120, 0, 28)
    menu.Position = UDim2.new(1, -124, 0, 0)
    menu.BackgroundTransparency = 1
    menu.Parent = btn

    local tBtn = Instance.new("TextButton", menu)
    tBtn.Size = UDim2.new(0, 36, 1, 0); tBtn.Position = UDim2.new(0,0,0,0)
    tBtn.Text = "TP"
    tBtn.Font = Enum.Font.SourceSansBold
    tBtn.TextSize = 12

    local tgtBtn = Instance.new("TextButton", menu)
    tgtBtn.Size = UDim2.new(0, 36, 1, 0); tgtBtn.Position = UDim2.new(0,40,0,0)
    tgtBtn.Text = "TGT"
    tgtBtn.Font = Enum.Font.SourceSansBold
    tgtBtn.TextSize = 12

    local pinBtn = Instance.new("TextButton", menu)
    pinBtn.Size = UDim2.new(0, 36, 1, 0); pinBtn.Position = UDim2.new(0,80,0,0)
    pinBtn.Text = "PIN"
    pinBtn.Font = Enum.Font.SourceSansBold
    pinBtn.TextSize = 12

    -- safe actions:
    tBtn.MouseButton1Click:Connect(function()
        local myChar = LocalPlayer.Character
        if myChar and myChar:FindFirstChild("HumanoidRootPart") and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
            myChar.HumanoidRootPart.CFrame = pl.Character.HumanoidRootPart.CFrame + Vector3.new(0,3,0)
        end
    end)

    tgtBtn.MouseButton1Click:Connect(function()
        -- set as visual target (client-side)
        selectedTarget = pl
    end)

    pinBtn.MouseButton1Click:Connect(function()
        -- create a persistent marker on that player's HRP (client-side)
        if pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = pl.Character.HumanoidRootPart
            if not hrp:FindFirstChild("CH_Pin") then
                local gui = Instance.new("BillboardGui", hrp)
                gui.Name = "CH_Pin"; gui.AlwaysOnTop = true; gui.Size = UDim2.new(0,120,0,24); gui.StudsOffset = Vector3.new(0,4,0)
                local lbl = Instance.new("TextLabel", gui); lbl.Size = UDim2.new(1,0,1,0); lbl.BackgroundTransparency = 1; lbl.Text = "Pinned: "..pl.Name; lbl.TextColor3 = Color3.new(1,0.6,0.6)
            else
                hrp.CH_Pin:Destroy()
            end
        end
    end)

    return btn
end

-- build initial player list
local function rebuildPlayerList()
    plScroll:ClearAllChildren()
    for _, pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer then
            makePlayerRow(pl)
        end
    end
    plScroll.CanvasSize = UDim2.new(0,0,0, (#Players:GetPlayers()-1) * 30)
end

Players.PlayerAdded:Connect(rebuildPlayerList)
Players.PlayerRemoving:Connect(rebuildPlayerList)
rebuildPlayerList()

-- ================= UI INTERACTIONS BINDINGS =================
-- toggles wiring
espBtn.MouseButton1Click:Connect(function() cfg.espOn = not cfg.espOn; espBtn.Text = cfg.espOn and "ON" or "OFF" end)
nameBtn.MouseButton1Click:Connect(function() cfg.showName = not cfg.showName; nameBtn.Text = cfg.showName and "ON" or "OFF" end)
hpBtn.MouseButton1Click:Connect(function() cfg.showHealth = not cfg.showHealth; hpBtn.Text = cfg.showHealth and "ON" or "OFF" end)
distBtn.MouseButton1Click:Connect(function() cfg.showDistance = not cfg.showDistance; distBtn.Text = cfg.showDistance and "ON" or "OFF" end)

aimBtn.MouseButton1Click:Connect(function()
    if cfg.aimbotStrength > 0 then
        cfg.aimbotStrength = 0
        aimBtn.Text = "OFF"
    else
        cfg.aimbotStrength = tonumber(aStrVal.Text) or 30
        aimBtn.Text = "ON"
    end
end)

aOpt1Btn.MouseButton1Click:Connect(function() cfg.aimbotRedCircle = not cfg.aimbotRedCircle; aOpt1Btn.Text = cfg.aimbotRedCircle and "ON" or "OFF" end)
aOpt2Btn.MouseButton1Click:Connect(function() cfg.aimbotLines = not cfg.aimbotLines; aOpt2Btn.Text = cfg.aimbotLines and "ON" or "OFF" end)
aOpt3Btn.MouseButton1Click:Connect(function() cfg.aimbotIgnoreSameTeam = not cfg.aimbotIgnoreSameTeam; aOpt3Btn.Text = cfg.aimbotIgnoreSameTeam and "ON" or "OFF" end)
aOpt4Btn.MouseButton1Click:Connect(function() cfg.aimbotNearestPriority = not cfg.aimbotNearestPriority; aOpt4Btn.Text = cfg.aimbotNearestPriority and "ON" or "OFF" end)

-- wallhack numeric wiring
wallVal.Text = tostring(math.floor(cfg.wallhackAlpha*100))
wallMinus.MouseButton1Click:Connect(function()
    local v = math.max(0, math.floor((cfg.wallhackAlpha*100) - 1))
    cfg.wallhackAlpha = v/100
    wallVal.Text = tostring(v)
end)
wallPlus.MouseButton1Click:Connect(function()
    local v = math.min(100, math.floor((cfg.wallhackAlpha*100) + 1))
    cfg.wallhackAlpha = v/100
    wallVal.Text = tostring(v)
end)

-- aimbot strength wiring
aStrVal.Text = tostring(cfg.aimbotStrength)
aStrMinus.MouseButton1Click:Connect(function()
    cfg.aimbotStrength = math.max(0, cfg.aimbotStrength - 1); aStrVal.Text = tostring(cfg.aimbotStrength)
end)
aStrPlus.MouseButton1Click:Connect(function()
    cfg.aimbotStrength = math.min(100, cfg.aimbotStrength + 1); aStrVal.Text = tostring(cfg.aimbotStrength)
end)

-- aimbot range wiring
arVal.Text = tostring(cfg.aimbotRange)
arMinus.MouseButton1Click:Connect(function()
    cfg.aimbotRange = math.max(10, cfg.aimbotRange - 5); arVal.Text = tostring(cfg.aimbotRange)
end)
arPlus.MouseButton1Click:Connect(function()
    cfg.aimbotRange = math.min(2000, cfg.aimbotRange + 5); arVal.Text = tostring(cfg.aimbotRange)
end)

-- minimize & close behavior
local minimized = false
btnMin.MouseButton1Click:Connect(function()
    minimized = not minimized
    for _, v in ipairs(scroll:GetChildren()) do
        if v ~= layout then v.Visible = not minimized end
    end
end)
btnClose.MouseButton1Click:Connect(function() screenGui.Enabled = false end)

-- header drag (mobile friendly)
local dragging = false
local dragStart, startPos
header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- ================= HELPER FUNCTIONS =================
local espMap = {}      -- player -> billboard
local redCircles = {}  -- player -> sphere part
local connectLines = {}-- player -> part (thin stretched part for line)
local selectedTarget = nil

local function createESPForPlayer(pl)
    if not pl.Character or not pl.Character:FindFirstChild("HumanoidRootPart") then return end
    local hrp = pl.Character.HumanoidRootPart
    -- create billboard if not exists
    if not hrp:FindFirstChild("CH_ESP") then
        local bill = Instance.new("BillboardGui")
        bill.Name = "CH_ESP"; bill.Adornee = hrp; bill.AlwaysOnTop = true
        bill.Size = UDim2.new(0,140,0,48); bill.StudsOffset = Vector3.new(0,3,0)
        bill.Parent = hrp
        local label = Instance.new("TextLabel", bill)
        label.Name = "Label"; label.Size = UDim2.new(1,0,1,0); label.BackgroundTransparency = 1
        label.TextColor3 = Color3.fromRGB(255,255,200); label.Font = Enum.Font.SourceSansBold; label.TextSize = 14
    end
    espMap[pl] = hrp.CH_ESP
end

local function removeESP(pl)
    if espMap[pl] and espMap[pl].Parent then
        espMap[pl]:Destroy()
    end
    espMap[pl] = nil
end

local function updateAllESPs()
    for _, pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
            if cfg.espOn then
                createESPForPlayer(pl)
                local lbl = pl.Character.HumanoidRootPart.CH_ESP.Label
                local nameText = cfg.showName and pl.Name or ""
                local hpText = ""
                if cfg.showHealth then
                    local hum = pl.Character:FindFirstChildOfClass("Humanoid")
                    if hum then hpText = " | HP:" .. math.floor(hum.Health) end
                end
                local distText = ""
                if cfg.showDistance and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local d = (pl.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                    distText = " | " .. tostring(math.floor(d)) .. "m"
                end
                lbl.Text = nameText .. hpText .. distText
            else
                if pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") and pl.Character.HumanoidRootPart:FindFirstChild("CH_ESP") then
                    pl.Character.HumanoidRootPart.CH_ESP:Destroy()
                end
            end
        end
    end
end

local function applyWallhackTransparency()
    for _, pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character then
            for _, part in pairs(pl.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    -- LocalTransparencyModifier = 1 - alpha (we invert to keep behavior intuitive)
                    part.LocalTransparencyModifier = 1 - cfg.wallhackAlpha
                end
            end
        end
    end
end

-- create/destroy red circle at hrp
local function ensureRedCircle(pl)
    if not pl.Character or not pl.Character:FindFirstChild("HumanoidRootPart") then return end
    local hrp = pl.Character.HumanoidRootPart
    if redCircles[pl] and redCircles[pl].Parent then return end
    local part = Instance.new("Part")
    part.Name = "CH_RedCircle"
    part.Size = Vector3.new(4,0.2,4)
    part.Anchored = true
    part.CanCollide = false
    part.Material = Enum.Material.Neon
    part.Color = Color3.fromRGB(255, 50, 50)
    part.Parent = Workspace
    redCircles[pl] = part
end

local function removeRedCircle(pl)
    if redCircles[pl] and redCircles[pl].Parent then redCircles[pl]:Destroy() end
    redCircles[pl] = nil
end

-- small function to create/adjust a thin part as a line between two positions
local function setLineForPlayer(pl, targetPos)
    if not pl.Character or not pl.Character:FindFirstChild("HumanoidRootPart") then return end
    if not connectLines[pl] or not connectLines[pl].Parent then
        local p = Instance.new("Part")
        p.Name = "CH_Line"
        p.Anchored = true
        p.CanCollide = false
        p.Size = Vector3.new(0.2,0.2,1)
        p.Material = Enum.Material.Neon
        p.Color = Color3.fromRGB(255, 100, 100)
        p.Parent = Workspace
        connectLines[pl] = p
    end
    local origin = connectLines[pl]
    local p1 = pl.Character.HumanoidRootPart.Position
    local p2 = targetPos
    local dist = (p2 - p1).Magnitude
    origin.Size = Vector3.new(0.15, 0.15, math.max(0.1, dist))
    origin.CFrame = CFrame.new((p1 + p2)/2, p2) * CFrame.new(0,0,-dist/2)
end

local function clearLine(pl)
    if connectLines[pl] and connectLines[pl].Parent then connectLines[pl]:Destroy() end
    connectLines[pl] = nil
end

-- ================= Aimbot TARGETING =================
local function validTarget(pl)
    if not pl.Character or not pl.Character:FindFirstChild("HumanoidRootPart") then return false end
    if cfg.aimbotIgnoreSameTeam and pl.Team and LocalPlayer.Team and pl.Team == LocalPlayer.Team then
        return false
    end
    local hrp = pl.Character.HumanoidRootPart
    local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myRoot then return false end
    local d = (hrp.Position - myRoot.Position).Magnitude
    return d <= cfg.aimbotRange
end

local function findBestTarget()
    local best, bestDist = nil, math.huge
    for _, pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and validTarget(pl) then
            local d = (pl.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            if cfg.aimbotNearestPriority then
                if d < bestDist then bestDist = d; best = pl end
            else
                -- else use a weaker distance priority combined with angle (not implemented heavy)
                if d < bestDist then bestDist = d; best = pl end
            end
        end
    end
    return best
end

-- ================= MAIN LOOP =================
local lastTick = tick()
RunService.RenderStepped:Connect(function(dt)
    -- update ESPs
    updateAllESPs()
    -- apply wallhack transparency locally
    applyWallhackTransparency()

    -- maintain red circles and lines if enabled
    for _, pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer then
            if cfg.aimbotRedCircle then
                ensureRedCircle(pl)
                if redCircles[pl] and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
                    redCircles[pl].CFrame = CFrame.new(pl.Character.HumanoidRootPart.Position - Vector3.new(0, 1.1, 0))
                    redCircles[pl].Size = Vector3.new(4, 0.2, 4)
                end
            else
                removeRedCircle(pl)
            end

            if cfg.aimbotLines then
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
                    setLineForPlayer(pl, LocalPlayer.Character.HumanoidRootPart.Position)
                end
            else
                clearLine(pl)
            end
        end
    end

    -- AIMBOT: camera nudge when user holds right mouse button
    if cfg.aimbotStrength > 0 and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local target = findBestTarget()
            if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                local targetPos = target.Character.HumanoidRootPart.Position
                local cam = workspace.CurrentCamera
                -- compute desired look CFrame
                local desired = CFrame.new(cam.CFrame.Position, targetPos)
                local strength = math.clamp(cfg.aimbotStrength / 100, 0, 1)
                -- lerp camera a bit toward target each frame
                workspace.CurrentCamera.CFrame = cam.CFrame:Lerp(desired, strength * math.min(0.5, dt*60))
            end
        end
    end

    lastTick = tick()
end)

-- cleanup on character removal
Players.PlayerRemoving:Connect(function(pl)
    removeESP(pl); removeRedCircle(pl); clearLine(pl)
end)
Players.PlayerAdded:Connect(function(pl)
    -- will be handled by updateAllESPs on next frame; optionally rebuild player list externally
end)

-- initial apply
updateAllESPs()
applyWallhackTransparency()

-- ========== END ==========
