-- ====================================================================
-- SCRIPT DE BODY-SWAP: CAPTURA O CARRO DO JOGO, OCULTA LATARIA E APLICA MESH
-- ====================================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local currentCarData = ""
local attachedCarModel = nil
local customMeshHolder = nil
local isBodyHidden = false

---------------------------------------------------------------------
-- CONFIGURAÇÕES DE OFFSET E ROTAÇÃO DA MESH SOBRE O CARRO
---------------------------------------------------------------------
local meshOffset = { X = 0, Y = 0, Z = 0 }
local meshRotation = { Pitch = 0, Yaw = 0, Roll = 0 }

---------------------------------------------------------------------
-- FUNÇÃO PARA ATUALIZAR A POSIÇÃO DA MESH NO CARRO
---------------------------------------------------------------------
local function updateMeshTransform()
    if not customMeshHolder or not attachedCarModel then return end
    local rootPart = attachedCarModel.PrimaryPart or attachedCarModel:FindFirstChildWhichIsA("BasePart")
    if not rootPart then return end

    local weld = customMeshHolder:FindFirstChild("MeshWeld")
    if weld then
        local posCFrame = CFrame.new(meshOffset.X, meshOffset.Y, meshOffset.Z)
        local rotCFrame = CFrame.Angles(
            math.rad(meshRotation.Pitch),
            math.rad(meshRotation.Yaw),
            math.rad(meshRotation.Roll)
        )
        weld.C0 = posCFrame * rotCFrame
    end
end

---------------------------------------------------------------------
-- CARREGAR E APLICAR A MESH NO CARRO CAPTURADO
---------------------------------------------------------------------
local function applyCustomMeshToCar(dataString)
    local char = LocalPlayer.Character
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    local seat = humanoid and humanoid.SeatPart

    if not seat or not seat:IsA("VehicleSeat") and not seat:IsA("Seat") then
        warn("⚠️ Você precisa estar sentado em um carro para aplicar a mesh!")
        return
    end

    attachedCarModel = seat.Parent
    
    -- Define uma PrimaryPart se não houver
    if not attachedCarModel.PrimaryPart then
        attachedCarModel.PrimaryPart = seat
    end

    -- 1. Ocultar a lataria original do carro (mantendo rodas e colisão de física)
    for _, part in ipairs(attachedCarModel:GetDescendants()) do
        if part:IsA("BasePart") and part ~= seat then
            local nameLower = part.Name:lower()
            -- Se não for roda, deixa invisível
            if not string.find(nameLower, "wheel") and not string.find(nameLower, "pneu") and not string.find(nameLower, "rim") then
                part.Transparency = 1
                -- Opcional: part.CanCollide = false (se quiser que só a mesh colida ou deixe a física padrão)
            end
        end
    end
    isBodyHidden = true

    -- 2. Limpar mesh anterior se já existir
    if customMeshHolder then customMeshHolder:Destroy() end

    -- 3. Criar container para a nova mesh
    customMeshHolder = Instance.new("Model")
    customMeshHolder.Name = "CustomMeshBody"
    customMeshHolder.Parent = attachedCarModel

    local rootPart = Instance.new("Part")
    rootPart.Name = "MeshRoot"
    rootPart.Size = Vector3.new(2, 1, 4)
    rootPart.Transparency = 1
    rootPart.CanCollide = false
    rootPart.Anchored = false
    rootPart.Parent = customMeshHolder

    local weld = Instance.new("Weld")
    weld.Name = "MeshWeld"
    weld.Part0 = attachedCarModel.PrimaryPart
    weld.Part1 = rootPart
    weld.C0 = CFrame.new(meshOffset.X, meshOffset.Y, meshOffset.Z) * CFrame.Angles(
        math.rad(meshRotation.Pitch), math.rad(meshRotation.Yaw), math.rad(meshRotation.Roll)
    )
    weld.Parent = customMeshHolder

    if dataString == "" or not dataString then return end

    -- 4. Interpretar a string da lataria e criar as peças
    local partsList = string.split(dataString, "|")
    for _, partInfo in ipairs(partsList) do
        local data = string.split(partInfo, ";")
        if #data >= 17 then
            local pName     = data[1]
            local meshId    = data[2]
            local textureId = data[3]
            local posX      = tonumber(data[4]) or 0
            local posY      = tonumber(data[5]) or 0
            local posZ      = tonumber(data[6]) or 0
            local rotX      = tonumber(data[7]) or 0
            local rotY      = tonumber(data[8]) or 0
            local rotZ      = tonumber(data[9]) or 0
            local scaleX    = tonumber(data[10]) or 1
            local scaleY    = tonumber(data[11]) or 1
            local scaleZ    = tonumber(data[12]) or 1
            local colorR    = tonumber(data[13]) or 255
            local colorG    = tonumber(data[14]) or 255
            local colorB    = tonumber(data[15]) or 255
            local trans     = tonumber(data[16]) or 0
            local reflect   = tonumber(data[17]) or 0

            local p = Instance.new("Part")
            p.Name = pName
            p.Size = Vector3.new(1, 1, 1)
            p.CanCollide = false
            p.CanTouch = false
            p.CanQuery = false
            p.Anchored = false
            p.Massless = true
            p.Transparency = trans
            p.Reflectance = reflect
            p.Color = Color3.fromRGB(colorR, colorG, colorB)
            p.Parent = customMeshHolder

            local m = Instance.new("SpecialMesh")
            m.MeshType = Enum.MeshType.FileMesh
            if meshId ~= "" then m.MeshId = "rbxassetid://" .. meshId end
            if textureId ~= "" then m.TextureId = "rbxassetid://" .. textureId end
            m.Scale = Vector3.new(scaleX, scaleY, scaleZ)
            m.Parent = p

            local offsetCFrame = CFrame.new(posX, posY, posZ) * CFrame.Angles(
                math.rad(rotX), math.rad(rotY), math.rad(rotZ)
            )

            local w = Instance.new("Weld")
            w.Part0 = rootPart
            w.Part1 = p
            w.C0 = offsetCFrame
            w.Parent = p
        end
    end
    
    print("🚗 Custom Mesh aplicada com sucesso sobre o carro do jogo!")
end

---------------------------------------------------------------------
-- INTERFACE GRÁFICA (UI FLUTUANTE COM MINIMIZAR)
---------------------------------------------------------------------
local screenGui = PlayerGui:FindFirstChild("CarSkinChangerUI") or Instance.new("ScreenGui")
screenGui.Name = "CarSkinChangerUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

for _, c in ipairs(screenGui:GetChildren()) do c:Destroy() end

local fullSize = UDim2.new(0, 320, 0, 460)
local miniSize = UDim2.new(0, 320, 0, 32)
local isMinimized = false

local frame = Instance.new("Frame")
frame.Size = fullSize
frame.Position = UDim2.new(0.02, 0, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 25, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.ClipsDescendants = true
frame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 32)
title.BackgroundColor3 = Color3.fromRGB(35, 45, 55)
title.Text = "🚗 BODY-SWAP CAR MESH"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 15
title.Parent = frame

-- BOTÃO DE MINIMIZAR (-)
local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 24, 0, 24)
minBtn.Position = UDim2.new(1, -28, 0, 4)
minBtn.Text = "-"
minBtn.BackgroundColor3 = Color3.fromRGB(60, 70, 85)
minBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minBtn.Font = Enum.Font.SourceSansBold
minBtn.TextSize = 18
minBtn.ZIndex = 5
minBtn.Parent = frame

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 4)
minCorner.Parent = minBtn

local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, 0, 1, -32)
scroll.Position = UDim2.new(0, 0, 0, 32)
scroll.BackgroundTransparency = 1
scroll.CanvasSize = UDim2.new(0, 0, 0, 620)
scroll.ScrollBarThickness = 6
scroll.Parent = frame

minBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    if isMinimized then
        scroll.Visible = false
        frame.Size = miniSize
        minBtn.Text = "+"
    else
        scroll.Visible = true
        frame.Size = fullSize
        minBtn.Text = "-"
    end
end)

-- CAMPO DE STRING
local boxTitle = Instance.new("TextLabel")
boxTitle.Size = UDim2.new(0.9, 0, 0, 20)
boxTitle.Position = UDim2.new(0.05, 0, 0.02, 0)
boxTitle.Text = "Cole a String da Custom Mesh:"
boxTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
boxTitle.BackgroundTransparency = 1
boxTitle.Font = Enum.Font.SourceSans
boxTitle.TextXAlignment = Enum.TextXAlignment.Left
boxTitle.Parent = scroll

local textBox = Instance.new("TextBox")
textBox.Size = UDim2.new(0.9, 0, 0, 28)
textBox.Position = UDim2.new(0.05, 0, 0.06, 0)
textBox.PlaceholderText = "Cole a string aqui..."
textBox.Text = ""
textBox.ClearTextOnFocus = false
textBox.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
textBox.TextColor3 = Color3.fromRGB(255, 255, 255)
textBox.Font = Enum.Font.SourceSans
textBox.TextSize = 12
textBox.Parent = scroll

local btnLoad = Instance.new("TextButton")
btnLoad.Size = UDim2.new(0.9, 0, 0, 28)
btnLoad.Position = UDim2.new(0.05, 0, 0.12, 0)
btnLoad.Text = "🚗 APLICAR NO CARRO ATUAL"
btnLoad.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
btnLoad.TextColor3 = Color3.fromRGB(255, 255, 255)
btnLoad.Font = Enum.Font.SourceSansBold
btnLoad.Parent = scroll

btnLoad.MouseButton1Click:Connect(function()
    if textBox.Text ~= "" then
        currentCarData = textBox.Text
        applyCustomMeshToCar(currentConfigData or currentCarData)
    end
end)

-- CRIADOR DE LINHAS DE CONTROLE
local function addSectionHeader(text, topPosY)
    local h = Instance.new("TextLabel")
    h.Size = UDim2.new(0.9, 0, 0, 22)
    h.Position = UDim2.new(0.05, 0, topPosY, 0)
    h.Text = "--- " .. text .. " ---"
    h.TextColor3 = Color3.fromRGB(0, 200, 255)
    h.BackgroundTransparency = 1
    h.Font = Enum.Font.SourceSansBold
    h.TextSize = 14
    h.Parent = scroll
end

local function createControlRow(label, topPosY, targetTable, key, step, isRot)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(0.4, 0, 0, 22)
    lbl.Position = UDim2.new(0.05, 0, topPosY, 0)
    lbl.Text = label .. ":"
    lbl.TextColor3 = Color3.fromRGB(220, 220, 220)
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.SourceSans
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = scroll

    local btnM = Instance.new("TextButton")
    btnM.Size = UDim2.new(0, 35, 0, 22)
    btnM.Position = UDim2.new(0.48, 0, topPosY, 0)
    btnM.Text = "-" .. step
    btnM.BackgroundColor3 = Color3.fromRGB(160, 50, 50)
    btnM.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnM.Font = Enum.Font.SourceSansBold
    btnM.Parent = scroll

    local btnP = Instance.new("TextButton")
    btnP.Size = UDim2.new(0, 35, 0, 22)
    btnP.Position = UDim2.new(0.62, 0, topPosY, 0)
    btnP.Text = "+" .. step
    btnP.BackgroundColor3 = Color3.fromRGB(50, 160, 50)
    btnP.TextColor3 = Color3.fromRGB(255, 255, 255)
    btnP.Font = Enum.Font.SourceSansBold
    btnP.Parent = scroll

    local val = Instance.new("TextLabel")
    val.Size = UDim2.new(0, 45, 0, 22)
    val.Position = UDim2.new(0.76, 0, topPosY, 0)
    val.Text = "0"
    val.TextColor3 = Color3.fromRGB(255, 255, 255)
    val.BackgroundTransparency = 1
    val.Font = Enum.Font.SourceSansBold
    val.Parent = scroll

    local function modify(dir)
        targetTable[key] = math.floor((targetTable[key] + (dir * step)) * 100) / 100
        val.Text = tostring(targetTable[key]) .. (isRot and "°" or "")
        updateMeshTransform()
    end

    btnM.MouseButton1Click:Connect(function() modify(-1) end)
    btnP.MouseButton1Click:Connect(function() modify(1) end)
end

-- SEÇÕES DE AJUSTE DA MESH SOBRE O CARRO
addSectionHeader("AJUSTE DE POSIÇÃO DA MESH", 0.20)
createControlRow("Lado (X)",    0.25, meshOffset, "X", 0.2, false)
createControlRow("Altura (Y)",  0.30, meshOffset, "Y", 0.2, false)
createContextRow = createControlRow
createControlRow("Frente (Z)",  0.35, meshOffset, "Z", 0.2, false)

addSectionHeader("AJUSTE DE ROTAÇÃO DA MESH", 0.43)
createControlRow("Inclinção (Pitch)", 0.48, meshRotation, "Pitch", 5, true)
createControlRow("Girar (Yaw)",        0.53, meshRotation, "Yaw", 5, true)
createControlRow("Lado (Roll)",        0.58, meshRotation, "Roll", 5, true)
