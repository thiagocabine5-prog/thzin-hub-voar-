--[[ 
Crie um LocalScript em StarterPlayerScripts.
Tela de carregamento RGB + HUB móvel com botão de ativar/desativar voo.
Voo agora segue corretamente a direção da câmera (W = frente, S = trás, etc).
--]]

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- RGB effect function
local function getRainbowColor(t)
    local r = math.sin(t) * 0.5 + 0.5
    local g = math.sin(t + 2) * 0.5 + 0.5
    local b = math.sin(t + 4) * 0.5 + 0.5
    return Color3.new(r, g, b)
end

-- Create Loading Screen
local loadingGui = Instance.new("ScreenGui")
loadingGui.IgnoreGuiInset = true
loadingGui.Name = "ThzinLoadingGui"
loadingGui.ResetOnSpawn = false

local loadingText = Instance.new("TextLabel")
loadingText.Parent = loadingGui
loadingText.Size = UDim2.new(1,0,1,0)
loadingText.Position = UDim2.new(0,0,0,0)
loadingText.BackgroundTransparency = 1
loadingText.Text = "thzin hub"
loadingText.Font = Enum.Font.GothamBlack
loadingText.TextScaled = true
loadingText.TextStrokeTransparency = 0.2
loadingText.TextStrokeColor3 = Color3.new(0,0,0)
loadingText.TextColor3 = Color3.fromRGB(255,255,255)

loadingGui.Parent = game:GetService("CoreGui")

-- Animate RGB loading text and remove after 2.5s
spawn(function()
    local t = 0
    while t < 2.5 do
        loadingText.TextColor3 = getRainbowColor(os.clock()*2)
        t = t + task.wait()
    end
    loadingGui:Destroy()
end)

-- Wait for character to load
if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
    player.CharacterAdded:Wait()
end

-- Create HUB GUI (smaller and draggable)
local hubGui = Instance.new("ScreenGui")
hubGui.Name = "ThzinHubGui"
hubGui.IgnoreGuiInset = true
hubGui.ResetOnSpawn = false
hubGui.Parent = game:GetService("CoreGui")

local hubFrame = Instance.new("Frame")
hubFrame.Size = UDim2.new(0,260,0,130)
hubFrame.Position = UDim2.new(0.5,-130,0.5,-65)
hubFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
hubFrame.BackgroundTransparency = 0.1
hubFrame.BorderSizePixel = 0
hubFrame.AnchorPoint = Vector2.new(0.5,0.5)
hubFrame.Parent = hubGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0,16)
corner.Parent = hubFrame

-- Draggable logic
local dragging, dragInput, dragStart, startPos

hubFrame.Active = true
hubFrame.Draggable = false -- Deprecated, so we implement custom drag
hubFrame.Selectable = true

hubFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = hubFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

hubFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        hubFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local hubTitle = Instance.new("TextLabel")
hubTitle.Parent = hubFrame
hubTitle.Size = UDim2.new(1,0,0,40)
hubTitle.Position = UDim2.new(0,0,0,0)
hubTitle.BackgroundTransparency = 1
hubTitle.Text = "thzin hub"
hubTitle.Font = Enum.Font.GothamBlack
hubTitle.TextScaled = true
hubTitle.TextStrokeTransparency = 0.2
hubTitle.TextStrokeColor3 = Color3.new(0,0,0)
hubTitle.TextColor3 = Color3.new(1,1,1)

local flyBtn = Instance.new("TextButton")
flyBtn.Parent = hubFrame
flyBtn.Size = UDim2.new(0.7,0,0,38)
flyBtn.Position = UDim2.new(0.15,0,0.62,0)
flyBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
flyBtn.Text = "Ativar Voo"
flyBtn.Font = Enum.Font.GothamSemibold
flyBtn.TextScaled = true
flyBtn.TextColor3 = Color3.fromRGB(255,255,255)
flyBtn.AutoButtonColor = true

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0,10)
btnCorner.Parent = flyBtn

-- Animate RGB titles
spawn(function()
    while true do
        local color = getRainbowColor(os.clock()*2)
        hubTitle.TextColor3 = color
        loadingText.TextColor3 = color
        task.wait()
    end
end)

-- Fly Logic
local flying = false
local flySpeed = 50
local flyConn1, flyConn2
local bv, bg

local function getHRP()
    local char = player.Character or player.CharacterAdded:Wait()
    return char:FindFirstChild("HumanoidRootPart")
end

local function startFly()
    if flying then return end
    local hrp = getHRP()
    if not hrp then return end
    flying = true
    flyBtn.Text = "Desativar Voo"

    bv = Instance.new("BodyVelocity")
    bv.Velocity = Vector3.new(0,0,0)
    bv.MaxForce = Vector3.new(1,1,1) * 1e5
    bv.Parent = hrp

    bg = Instance.new("BodyGyro")
    bg.MaxTorque = Vector3.new(1,1,1) * 1e5
    bg.P = 1e4
    bg.CFrame = hrp.CFrame
    bg.Parent = hrp

    local cam = workspace.CurrentCamera
    local moveVec = Vector3.new(0,0,0)

    flyConn1 = UIS.InputBegan:Connect(function(input, processed)
        if processed then return end
        if input.KeyCode == Enum.KeyCode.W then moveVec = moveVec + Vector3.new(0,0,1) end
        if input.KeyCode == Enum.KeyCode.S then moveVec = moveVec + Vector3.new(0,0,-1) end
        if input.KeyCode == Enum.KeyCode.A then moveVec = moveVec + Vector3.new(-1,0,0) end
        if input.KeyCode == Enum.KeyCode.D then moveVec = moveVec + Vector3.new(1,0,0) end
        if input.KeyCode == Enum.KeyCode.Space then moveVec = moveVec + Vector3.new(0,1,0) end
        if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.LeftShift then moveVec = moveVec + Vector3.new(0,-1,0) end
    end)
    flyConn2 = UIS.InputEnded:Connect(function(input, processed)
        if input.KeyCode == Enum.KeyCode.W then moveVec = moveVec - Vector3.new(0,0,1) end
        if input.KeyCode == Enum.KeyCode.S then moveVec = moveVec - Vector3.new(0,0,-1) end
        if input.KeyCode == Enum.KeyCode.A then moveVec = moveVec - Vector3.new(-1,0,0) end
        if input.KeyCode == Enum.KeyCode.D then moveVec = moveVec - Vector3.new(1,0,0) end
        if input.KeyCode == Enum.KeyCode.Space then moveVec = moveVec - Vector3.new(0,1,0) end
        if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.LeftShift then moveVec = moveVec - Vector3.new(0,-1,0) end
    end)

    -- Loop de voo corrigido (direção correta)
    spawn(function()
        while flying and hrp and bv and bg do
            if moveVec.Magnitude > 0 then
                local camCF = cam.CFrame
                -- DIREÇÃO CORRETA: W = frente, S = trás, A = esquerda, D = direita
                local dir = (camCF.LookVector * moveVec.Z + camCF.RightVector * moveVec.X)
                dir = dir + Vector3.new(0, moveVec.Y, 0)
                bv.Velocity = dir.Unit * flySpeed
            else
                bv.Velocity = Vector3.new(0,0,0)
            end
            bg.CFrame = cam.CFrame
            task.wait()
        end
    end)
end

local function stopFly()
    if not flying then return end
    flying = false
    flyBtn.Text = "Ativar Voo"
    if bv then bv:Destroy() bv = nil end
    if bg then bg:Destroy() bg = nil end
    if flyConn1 then flyConn1:Disconnect() flyConn1 = nil end
    if flyConn2 then flyConn2:Disconnect() flyConn2 = nil end
end

flyBtn.MouseButton1Click:Connect(function()
    if flying then
        stopFly()
    else
        startFly()
    end
end)

UIS.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.V then
        if flying then
            stopFly()
        else
            startFly()
        end
    end
end)

-- Reset on respawn
player.CharacterAdded:Connect(function(char)
    char:WaitForChild("HumanoidRootPart")
    stopFly()
end)
