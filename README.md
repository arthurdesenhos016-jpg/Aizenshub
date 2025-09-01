-- LocalScript em StarterPlayerScripts
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Configurações
local guiEnabled = true
local isSpeedOn, isJumpOn, isNoClipOn, isESPOn, isFlyOn = false, false, false, false, false
local savedConfig = {}
local defaultSpeed, defaultJump

local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
defaultSpeed, defaultJump = humanoid.WalkSpeed, humanoid.JumpPower

local flySpeed = 50
local flyBodyVelocity

-- Função NoClip
local function applyNoClip()
    if isNoClipOn and character then
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end

-- ESP
local espItems = {}
local function applyESP()
    -- Limpar ESP antigo
    for _, item in pairs(espItems) do
        if item.Adornee then item:Destroy() end
    end
    espItems = {}

    if not isESPOn then return end

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = plr.Character.HumanoidRootPart
            -- Caixa
            local box = Instance.new("BoxHandleAdornment")
            box.Adornee = hrp
            box.Size = Vector3.new(2, 5, 1)
            box.Color3 = Color3.fromRGB(255,0,0)
            box.Transparency = 0.5
            box.AlwaysOnTop = true
            box.ZIndex = 10
            box.Parent = LocalPlayer.PlayerGui
            table.insert(espItems, box)
            -- Nome
            local nameTag = Instance.new("BillboardGui")
            nameTag.Adornee = hrp
            nameTag.Size = UDim2.new(0,100,0,50)
            nameTag.AlwaysOnTop = true
            nameTag.Parent = LocalPlayer.PlayerGui
            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1,0,1,0)
            label.BackgroundTransparency = 1
            label.Text = plr.Name
            label.TextColor3 = Color3.fromRGB(255,255,0)
            label.Font = Enum.Font.GothamBold
            label.TextScaled = true
            label.Parent = nameTag
            table.insert(espItems, nameTag)
        end
    end
end

-- Fly
local function applyFly()
    if isFlyOn and character and humanoid then
        if not flyBodyVelocity then
            flyBodyVelocity = Instance.new("BodyVelocity")
            flyBodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
            flyBodyVelocity.Velocity = Vector3.new(0,0,0)
            flyBodyVelocity.Parent = character:FindFirstChild("HumanoidRootPart")
        end
    else
        if flyBodyVelocity then
            flyBodyVelocity:Destroy()
            flyBodyVelocity = nil
        end
    end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    applyNoClip()
    applyESP()

    -- Fly
    if isFlyOn and flyBodyVelocity and humanoid then
        local moveVec = Vector3.new()
        local cam = workspace.CurrentCamera
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveVec = moveVec + cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveVec = moveVec - cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveVec = moveVec - cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveVec = moveVec + cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveVec = moveVec + Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveVec = moveVec - Vector3.new(0,1,0) end
        if moveVec.Magnitude > 0 then
            flyBodyVelocity.Velocity = moveVec.Unit * flySpeed
        else
            flyBodyVelocity.Velocity = Vector3.new(0,0,0)
        end
    end
end)

-- Criar GUI
local function createGUI()
    if LocalPlayer.PlayerGui:FindFirstChild("TutuHubGUI") then
        return LocalPlayer.PlayerGui.TutuHubGUI
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "TutuHubGUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0,300,0,350)
    mainFrame.Position = UDim2.new(0,20,0,100)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = screenGui

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1,0,0,30)
    title.BackgroundTransparency = 1
    title.Text = "Aizens hub"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 20
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.Parent = mainFrame

    local function createButton(name, y, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0,120,0,30)
        btn.Position = UDim2.new(0,10,0,y)
        btn.Text = name.." OFF"
        btn.BackgroundColor3 = Color3.fromRGB(200,0,0)
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        btn.Parent = mainFrame
        btn.MouseButton1Click:Connect(function()
            local active = callback()
            if active then
                btn.Text = name.." ON"
                btn.BackgroundColor3 = Color3.fromRGB(0,200,0)
            else
                btn.Text = name.." OFF"
                btn.BackgroundColor3 = Color3.fromRGB(200,0,0)
            end
        end)
        return btn
    end

    local function createTextBox(y, placeholder, default)
        local tb = Instance.new("TextBox")
        tb.Size = UDim2.new(0,100,0,30)
        tb.Position = UDim2.new(0,150,0,y)
        tb.PlaceholderText = placeholder
        tb.Text = default or ""
        tb.BackgroundColor3 = Color3.fromRGB(50,50,50)
        tb.TextColor3 = Color3.fromRGB(255,255,255)
        tb.Parent = mainFrame
        return tb
    end

    -- Speed
    local speedInput = createTextBox(50,"Speed",tostring(defaultSpeed))
    createButton("Speed",50,function()
        isSpeedOn = not isSpeedOn
        local s = tonumber(speedInput.Text) or defaultSpeed
        humanoid.WalkSpeed = isSpeedOn and s or defaultSpeed
        savedConfig.speed = s
        return isSpeedOn
    end)

    -- Jump
    local jumpInput = createTextBox(90,"Jump",tostring(defaultJump))
    createButton("Jump",90,function()
        isJumpOn = not isJumpOn
        local j = tonumber(jumpInput.Text) or defaultJump
        humanoid.JumpPower = isJumpOn and j or defaultJump
        savedConfig.jump = j
        return isJumpOn
    end)

    -- NoClip
    createButton("NoClip",130,function()
        isNoClipOn = not isNoClipOn
        return isNoClipOn
    end)

    -- ESP
    createButton("ESP",170,function()
        isESPOn = not isESPOn
        return isESPOn
    end)

    -- Fly
    local flyInput = createTextBox(210,"FlySpeed","50")
    createButton("Fly",210,function()
        isFlyOn = not isFlyOn
        local val = tonumber(flyInput.Text) or 50
        flySpeed = val
        applyFly()
        return isFlyOn
    end)

    -- Botão fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0,30,0,30)
    closeButton.Position = UDim2.new(1,-35,0,0)
    closeButton.Text = "X"
    closeButton.Font = Enum.Font.GothamBold
    closeButton.TextSize = 18
    closeButton.BackgroundColor3 = Color3.fromRGB(200,0,0)
    closeButton.TextColor3 = Color3.fromRGB(255,255,255)
    closeButton.Parent = mainFrame
    closeButton.MouseButton1Click:Connect(function()
        screenGui.Enabled = false
        guiEnabled = false
    end)

    return screenGui
end

-- Cria GUI
local gui = createGUI()

-- Tecla K
UserInputService.InputBegan:Connect(function(input,gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.K then
        gui.Enabled = not gui.Enabled
        guiEnabled = gui.Enabled
    end
end)

-- Reaplicar humanoid após respawn
LocalPlayer.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    humanoid.WalkSpeed = savedConfig.speed or defaultSpeed
    humanoid.JumpPower = savedConfig.jump or defaultJump
end)

