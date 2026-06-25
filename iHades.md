--[[
    ╔══════════════════════════════════════╗
    ║           IHadesHD Script            ║
    ╚══════════════════════════════════════╝
]]

-- ============================================================
-- SERVIÇOS
-- ============================================================
local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService       = game:GetService("RunService")
local CoreGui          = game:GetService("CoreGui")
local Debris           = game:GetService("Debris")
local StarterGui       = game:GetService("StarterGui")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ============================================================
-- VARIÁVEIS GLOBAIS
-- ============================================================
local uiVisible = true

-- Glitch
getgenv().GlideEnabled = false
getgenv().GlideSpeed   = 350
getgenv().GlideTime    = 0.20
local JumpForce     = 300
local LastShootTime = 0

-- Fly V3
local flyActive = false
local flySpeed  = 400
local FlyVel    = nil

-- Hitbox Expander
getgenv().HitboxSize         = 15
getgenv().HitboxTransparency = 0.9
getgenv().HitboxStatus       = false
getgenv().TeamCheck          = false

-- ============================================================
-- INTERFACE GRÁFICA
-- ============================================================
local guiParent = (gethui and gethui()) or CoreGui

if guiParent:FindFirstChild("IHadesHDUI") then
    guiParent.IHadesHDUI:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name         = "IHadesHDUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent       = guiParent

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size                   = UDim2.new(0, 250, 0, 300)
MainFrame.Position               = UDim2.new(0, 40, 0.5, -150)
MainFrame.BackgroundColor3       = Color3.fromRGB(5, 5, 5)
MainFrame.BackgroundTransparency = 0.05
MainFrame.BorderSizePixel        = 0
MainFrame.Active                 = true
MainFrame.Draggable              = true
MainFrame.Parent                 = ScreenGui

do
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 12)
    c.Parent = MainFrame

    local s = Instance.new("UIStroke")
    s.Color     = Color3.fromRGB(40, 40, 40)
    s.Thickness = 2
    s.Parent    = MainFrame
end

-- Título
local Title = Instance.new("TextLabel")
Title.Size               = UDim2.new(1, 0, 0, 30)
Title.Position           = UDim2.new(0, 0, 0, 10)
Title.BackgroundTransparency = 1
Title.Text               = "IHadesHD"
Title.TextColor3         = Color3.fromRGB(255, 255, 255)
Title.Font               = Enum.Font.GothamBlack
Title.TextSize           = 22
Title.Parent             = MainFrame

-- ============================================================
-- BARRA DE ABAS
-- ============================================================
local TabBar = Instance.new("Frame")
TabBar.Size                  = UDim2.new(0.95, 0, 0, 25)
TabBar.Position              = UDim2.new(0.025, 0, 0, 44)
TabBar.BackgroundTransparency = 1
TabBar.Parent                = MainFrame

local function createTabButton(text, xPos, width)
    local btn = Instance.new("TextButton")
    btn.Size             = UDim2.new(width or 0.31, 0, 1, 0)
    btn.Position         = UDim2.new(xPos, 0, 0, 0)
    btn.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    btn.Text             = text
    btn.TextColor3       = Color3.fromRGB(255, 255, 255)
    btn.Font             = Enum.Font.GothamBold
    btn.TextSize         = 8
    btn.Parent           = TabBar
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    return btn
end

local TabGlitchBtn = createTabButton("GLITCH", 0)
local TabFlyBtn    = createTabButton("FLY V3", 0.345)
local TabHitboxBtn = createTabButton("HITBOX", 0.69)

-- ============================================================
-- PÁGINAS
-- ============================================================
local Pages = Instance.new("Frame")
Pages.Size                  = UDim2.new(1, 0, 0, 160)
Pages.Position              = UDim2.new(0, 0, 0, 75)
Pages.BackgroundTransparency = 1
Pages.Parent                = MainFrame

local GlitchPage = Instance.new("Frame")
GlitchPage.Size                  = UDim2.new(1, 0, 1, 0)
GlitchPage.BackgroundTransparency = 1
GlitchPage.Parent                = Pages

local FlyPage = Instance.new("Frame")
FlyPage.Size                  = UDim2.new(1, 0, 1, 0)
FlyPage.BackgroundTransparency = 1
FlyPage.Parent                = Pages

local HitboxPage = Instance.new("Frame")
HitboxPage.Size                  = UDim2.new(1, 0, 1, 0)
HitboxPage.BackgroundTransparency = 1
HitboxPage.Parent                = Pages

local function showPage(page)
    GlitchPage.Visible = (page == GlitchPage)
    FlyPage.Visible    = (page == FlyPage)
    HitboxPage.Visible = (page == HitboxPage)

    local ACTIVE   = Color3.fromRGB(60, 60, 60)
    local INACTIVE = Color3.fromRGB(15, 15, 15)

    TabGlitchBtn.BackgroundColor3 = (page == GlitchPage) and ACTIVE or INACTIVE
    TabFlyBtn.BackgroundColor3    = (page == FlyPage)    and ACTIVE or INACTIVE
    TabHitboxBtn.BackgroundColor3 = (page == HitboxPage) and ACTIVE or INACTIVE
end

showPage(GlitchPage)

TabGlitchBtn.MouseButton1Click:Connect(function() showPage(GlitchPage) end)
TabFlyBtn.MouseButton1Click:Connect(function()    showPage(FlyPage)    end)
TabHitboxBtn.MouseButton1Click:Connect(function() showPage(HitboxPage) end)

-- ============================================================
-- HELPER: CAMPO DE INPUT COM LABEL
-- ============================================================
local function createInputField(labelText, defaultVal, yPos, parentPage)
    local lbl = Instance.new("TextLabel")
    lbl.Size               = UDim2.new(0.5, 0, 0, 25)
    lbl.Position           = UDim2.new(0, 15, 0, yPos)
    lbl.BackgroundTransparency = 1
    lbl.Text               = labelText
    lbl.TextColor3         = Color3.fromRGB(255, 255, 255)
    lbl.TextXAlignment     = Enum.TextXAlignment.Left
    lbl.Font               = Enum.Font.GothamMedium
    lbl.TextSize           = 12
    lbl.Parent             = parentPage

    local bg = Instance.new("Frame")
    bg.Size             = UDim2.new(0.42, 0, 0, 24)
    bg.Position         = UDim2.new(0.53, -5, 0, yPos)
    bg.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    bg.BorderSizePixel  = 0
    bg.Parent           = parentPage
    Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 6)

    local input = Instance.new("TextBox")
    input.Size               = UDim2.new(1, 0, 1, 0)
    input.BackgroundTransparency = 1
    input.TextColor3         = Color3.fromRGB(255, 255, 255)
    input.Text               = tostring(defaultVal)
    input.Font               = Enum.Font.GothamMedium
    input.TextSize           = 12
    input.Parent             = bg

    return input
end

-- ============================================================
-- ABA 1: GLITCH
-- ============================================================
local GlitchToggleBtn = Instance.new("TextButton")
GlitchToggleBtn.Size             = UDim2.new(0.9, 0, 0, 30)
GlitchToggleBtn.Position         = UDim2.new(0.05, 0, 0, 5)
GlitchToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 20, 40)
GlitchToggleBtn.Text             = "GLITCH SYSTEM: OFF"
GlitchToggleBtn.TextColor3       = Color3.fromRGB(255, 80, 80)
GlitchToggleBtn.Font             = Enum.Font.GothamBold
GlitchToggleBtn.TextSize         = 12
GlitchToggleBtn.Parent           = GlitchPage
Instance.new("UICorner", GlitchToggleBtn).CornerRadius = UDim.new(0, 6)

local GlitchSpeedInput = createInputField("Velocidade:",   getgenv().GlideSpeed, 45,  GlitchPage)
local GlitchDurInput   = createInputField("Duração:",      getgenv().GlideTime,  80,  GlitchPage)
local JumpPowerInput   = createInputField("G-Jump Power:", JumpForce,            115, GlitchPage)

local function updateGlitchButtonVisual()
    local on = getgenv().GlideEnabled
    GlitchToggleBtn.Text             = on and "GLITCH SYSTEM: ON"  or "GLITCH SYSTEM: OFF"
    GlitchToggleBtn.TextColor3       = on and Color3.fromRGB(80, 255, 120) or Color3.fromRGB(255, 80, 80)
    GlitchToggleBtn.BackgroundColor3 = on and Color3.fromRGB(50, 25, 70)  or Color3.fromRGB(30, 20, 40)
end

GlitchToggleBtn.MouseButton1Click:Connect(function()
    getgenv().GlideEnabled = not getgenv().GlideEnabled
    updateGlitchButtonVisual()
end)

-- ============================================================
-- ABA 2: FLY V3
-- ============================================================
local FlyToggleBtn = Instance.new("TextButton")
FlyToggleBtn.Size             = UDim2.new(0.9, 0, 0, 30)
FlyToggleBtn.Position         = UDim2.new(0.05, 0, 0, 5)
FlyToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 20, 40)
FlyToggleBtn.Text             = "FLY SYSTEM (H): OFF"
FlyToggleBtn.TextColor3       = Color3.fromRGB(255, 80, 80)
FlyToggleBtn.Font             = Enum.Font.GothamBold
FlyToggleBtn.TextSize         = 12
FlyToggleBtn.Parent           = FlyPage
Instance.new("UICorner", FlyToggleBtn).CornerRadius = UDim.new(0, 6)

local FlySpeedInput = createInputField("Fly Speed:", flySpeed, 45, FlyPage)

local function updateFlyState(state)
    flyActive = state

    local on = flyActive
    FlyToggleBtn.Text             = on and "FLY SYSTEM (H): ON"  or "FLY SYSTEM (H): OFF"
    FlyToggleBtn.TextColor3       = on and Color3.fromRGB(80, 255, 120) or Color3.fromRGB(255, 80, 80)
    FlyToggleBtn.BackgroundColor3 = on and Color3.fromRGB(50, 25, 70)  or Color3.fromRGB(30, 20, 40)

    local hum = player.Character and player.Character:FindFirstChild("Humanoid")

    if not flyActive then
        if FlyVel then
            FlyVel:Destroy()
            FlyVel = nil
        end
        if hum then hum.PlatformStand = false end
    elseif hum then
        hum.PlatformStand = true
    end
end

FlyToggleBtn.MouseButton1Click:Connect(function()
    updateFlyState(not flyActive)
end)

-- ============================================================
-- ABA 3: HITBOX EXPANDER
-- ============================================================
local HitboxToggleBtn = Instance.new("TextButton")
HitboxToggleBtn.Size             = UDim2.new(0.9, 0, 0, 30)
HitboxToggleBtn.Position         = UDim2.new(0.05, 0, 0, 5)
HitboxToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 20, 40)
HitboxToggleBtn.Text             = "HITBOX: OFF"
HitboxToggleBtn.TextColor3       = Color3.fromRGB(255, 80, 80)
HitboxToggleBtn.Font             = Enum.Font.GothamBold
HitboxToggleBtn.TextSize         = 12
HitboxToggleBtn.Parent           = HitboxPage
Instance.new("UICorner", HitboxToggleBtn).CornerRadius = UDim.new(0, 6)

local HitboxSizeInput  = createInputField("Tamanho:",       getgenv().HitboxSize,         45,  HitboxPage)
local HitboxTransInput = createInputField("Transparência:", getgenv().HitboxTransparency, 80,  HitboxPage)

local TeamCheckBtn = Instance.new("TextButton")
TeamCheckBtn.Size             = UDim2.new(0.9, 0, 0, 30)
TeamCheckBtn.Position         = UDim2.new(0.05, 0, 0, 115)
TeamCheckBtn.BackgroundColor3 = Color3.fromRGB(30, 20, 40)
TeamCheckBtn.Text             = "TEAM CHECK: OFF"
TeamCheckBtn.TextColor3       = Color3.fromRGB(255, 80, 80)
TeamCheckBtn.Font             = Enum.Font.GothamBold
TeamCheckBtn.TextSize         = 12
TeamCheckBtn.Parent           = HitboxPage
Instance.new("UICorner", TeamCheckBtn).CornerRadius = UDim.new(0, 6)

local function updateHitboxVisual()
    local on = getgenv().HitboxStatus
    HitboxToggleBtn.Text             = on and "HITBOX: ON"  or "HITBOX: OFF"
    HitboxToggleBtn.TextColor3       = on and Color3.fromRGB(80, 255, 120) or Color3.fromRGB(255, 80, 80)
    HitboxToggleBtn.BackgroundColor3 = on and Color3.fromRGB(50, 25, 70)  or Color3.fromRGB(30, 20, 40)
end

local function updateTeamCheckVisual()
    local on = getgenv().TeamCheck
    TeamCheckBtn.Text             = on and "TEAM CHECK: ON"  or "TEAM CHECK: OFF"
    TeamCheckBtn.TextColor3       = on and Color3.fromRGB(80, 255, 120) or Color3.fromRGB(255, 80, 80)
    TeamCheckBtn.BackgroundColor3 = on and Color3.fromRGB(50, 25, 70)  or Color3.fromRGB(30, 20, 40)
end

HitboxToggleBtn.MouseButton1Click:Connect(function()
    getgenv().HitboxStatus = not getgenv().HitboxStatus
    updateHitboxVisual()
end)

TeamCheckBtn.MouseButton1Click:Connect(function()
    getgenv().TeamCheck = not getgenv().TeamCheck
    updateTeamCheckVisual()
end)

-- ============================================================
-- FOOTER / INFO
-- ============================================================
local InfoFrame = Instance.new("Frame")
InfoFrame.Size                   = UDim2.new(0.9, 0, 0, 40)
InfoFrame.Position               = UDim2.new(0.05, 0, 1, -50)
InfoFrame.BackgroundColor3       = Color3.fromRGB(0, 0, 0)
InfoFrame.BackgroundTransparency = 0.4
InfoFrame.BorderSizePixel        = 0
InfoFrame.Parent                 = MainFrame
Instance.new("UICorner", InfoFrame).CornerRadius = UDim.new(0, 6)

local InfoLabel = Instance.new("TextLabel")
InfoLabel.Size               = UDim2.new(1, 0, 1, 0)
InfoLabel.BackgroundTransparency = 1
InfoLabel.Text               = "[H] Fly  |  [G] Pulo  |  [K] Menu  |  Hitbox Aba"
InfoLabel.TextColor3         = Color3.fromRGB(255, 255, 255)
InfoLabel.Font               = Enum.Font.Gotham
InfoLabel.TextSize           = 9
InfoLabel.LineHeight         = 1.3
InfoLabel.TextWrapped        = true
InfoLabel.Parent             = InfoFrame

-- ============================================================
-- SINCRONIZAÇÃO DOS INPUTS
-- ============================================================
local function ResetJump()
    local hum = player.Character and player.Character:FindFirstChild("Humanoid")
    if hum then
        hum.JumpPower    = 50
        hum.UseJumpPower = true
    end
end

ResetJump()
player.CharacterAdded:Connect(ResetJump)

GlitchSpeedInput.FocusLost:Connect(function()
    getgenv().GlideSpeed = tonumber(GlitchSpeedInput.Text) or getgenv().GlideSpeed
end)
GlitchDurInput.FocusLost:Connect(function()
    getgenv().GlideTime = tonumber(GlitchDurInput.Text) or getgenv().GlideTime
end)
JumpPowerInput.FocusLost:Connect(function()
    JumpForce = tonumber(JumpPowerInput.Text) or JumpForce
end)
FlySpeedInput.FocusLost:Connect(function()
    flySpeed = tonumber(FlySpeedInput.Text) or flySpeed
end)
HitboxSizeInput.FocusLost:Connect(function()
    getgenv().HitboxSize = tonumber(HitboxSizeInput.Text) or getgenv().HitboxSize
end)
HitboxTransInput.FocusLost:Connect(function()
    getgenv().HitboxTransparency = tonumber(HitboxTransInput.Text) or getgenv().HitboxTransparency
end)

-- ============================================================
-- LÓGICA DO GLITCH: DETECÇÃO DE FERRAMENTA
-- ============================================================
local function SetupToolDetection(character)
    character.ChildAdded:Connect(function(child)
        if child:IsA("Tool") and (child.Name:find("Soul") or child.Name:find("Guitar")) then
            child.Activated:Connect(function()
                if getgenv().GlideEnabled then
                    LastShootTime = tick()
                end
            end)
        end
    end)
end

if player.Character then SetupToolDetection(player.Character) end
player.CharacterAdded:Connect(SetupToolDetection)

-- ============================================================
-- TECLAS DE ATALHO
-- ============================================================
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    -- [K] Alternar visibilidade do menu
    if input.KeyCode == Enum.KeyCode.K then
        uiVisible = not uiVisible
        MainFrame.Visible = uiVisible

    -- [H] Alternar voo
    elseif input.KeyCode == Enum.KeyCode.H then
        updateFlyState(not flyActive)

    -- [G] Super pulo
    elseif input.KeyCode == Enum.KeyCode.G then
        local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.Velocity = Vector3.new(hrp.Velocity.X, 0, hrp.Velocity.Z)
            hrp.Velocity += Vector3.new(0, JumpForce, 0)
        end
    end
end)

-- ============================================================
-- LOOP DE RENDER: FLY V3 + HITBOX EXPANDER
-- ============================================================
local function resetHRP(hrp)
    hrp.Size         = Vector3.new(2, 2, 1)
    hrp.Transparency = 1
    hrp.BrickColor   = BrickColor.new("Medium stone grey")
    hrp.Material     = Enum.Material.Plastic
end

RunService.RenderStepped:Connect(function()
    -- >> FLY V3 <<
    local char = player.Character
    if flyActive and char and char:FindFirstChild("HumanoidRootPart") then
        local root = char.HumanoidRootPart

        if not FlyVel or FlyVel.Parent ~= root then
            if FlyVel then FlyVel:Destroy() end
            FlyVel          = Instance.new("BodyVelocity")
            FlyVel.Name     = "SkyFly"
            FlyVel.MaxForce = Vector3.new(9e9, 9e9, 9e9)
            FlyVel.Parent   = root
        end

        local dir = Vector3.zero
        local UIS = UserInputService

        if UIS:IsKeyDown(Enum.KeyCode.W) then dir += camera.CFrame.LookVector  end
        if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= camera.CFrame.LookVector  end
        if UIS:IsKeyDown(Enum.KeyCode.A) then dir -= camera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then dir += camera.CFrame.RightVector end

        FlyVel.Velocity = dir * flySpeed
    end

    -- >> HITBOX EXPANDER <<
    for _, v in ipairs(Players:GetPlayers()) do
        if v == player then continue end

        pcall(function()
            local hrp = v.Character.HumanoidRootPart

            if getgenv().HitboxStatus then
                local sameTeam = v.Team ~= nil
                    and player.Team ~= nil
                    and v.Team == player.Team

                if not getgenv().TeamCheck or not sameTeam then
                    local s = getgenv().HitboxSize
                    hrp.Size         = Vector3.new(s, s, s)
                    hrp.Transparency = getgenv().HitboxTransparency
                    hrp.BrickColor   = BrickColor.new("Really black")
                    hrp.Material     = Enum.Material.Neon
                    hrp.CanCollide   = false
                else
                    resetHRP(hrp)
                end
            else
                resetHRP(hrp)
            end
        end)
    end
end)

-- ============================================================
-- LOOP DE HEARTBEAT: GLITCH SYSTEM
-- ============================================================
RunService.Heartbeat:Connect(function()
    if not getgenv().GlideEnabled then return end

    local char = player.Character
    if not (char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid")) then
        return
    end

    local hrp = char.HumanoidRootPart
    local hum = char.Humanoid

    if (tick() - LastShootTime < 1.5) and hrp.Velocity.Magnitude > 20 then
        if hrp:FindFirstChild("OmniGlide") then
            hrp.OmniGlide:Destroy()
        end

        local dir = hum.MoveDirection
        if dir.Magnitude == 0 then
            local look = camera.CFrame.LookVector
            dir = Vector3.new(look.X, 0, look.Z).Unit
        else
            dir = dir.Unit
        end

        local bv    = Instance.new("BodyVelocity")
        bv.Name     = "OmniGlide"
        bv.MaxForce = Vector3.new(100000, 0, 100000)
        bv.Velocity = dir * getgenv().GlideSpeed
        bv.Parent   = hrp

        Debris:AddItem(bv, getgenv().GlideTime)
        LastShootTime = 0
    end
end)

-- ============================================================
-- NOTIFICAÇÃO DE CARREGAMENTO
-- ============================================================
StarterGui:SetCore("SendNotification", {
    Title    = "IHadesHD",
    Text     = "Script carregado com sucesso!",
    Duration = 5,
})
