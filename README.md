-- ====================================================================
-- SCRIPT DE RODAS COM PRESETS EMBUTIDOS + CAMBER ESQUERDA/DIREITA + MINIMIZAR
-- ====================================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

---------------------------------------------------------------------
-- PRESETS DE RODAS CONFIGURADOS
---------------------------------------------------------------------
local WHEEL_PRESETS = {
    ["Kranze Cerb's"] = [[RL;Aro parte de fora;7459763245;7459794427;0;0;0;0;0;0;0.61;0.67;0.67;255;255;255;0;0|RL;Pneu;5937413583;4648056401;0;0;0;0;0;90;1.04;1.1;1.04;0;0;0;0;0|RL;Aro Interna 1;7459766430;7459794427;-0.3;0;-0.02;0;0;0;0.68;0.68;0.68;255;255;255;0;0|RL;Aro interna 2;7459768360;7459794427;-0.3;0;0.48;0;0;0;0.68;0.68;0.68;255;255;255;0;0|RL;Aro Meio;7459760785;7459803417;-0.44;0;0;0;0;0;0.8;0.8;0.8;255;255;255;0;0|RL;Parafusos;7459769752;7459794427;-0.3;-0.02;0.005;0;0;0;0.67;0.67;0.67;255;255;255;0;0|RR;Aro parte de fora;7459763245;7459794427;0;0;0;0;0;0;0.61;0.67;0.67;255;255;255;0;0|RR;Pneu;5937413583;4648056401;0;0;0;0;0;90;1.04;1.1;1.04;0;0;0;0;0|RR;Aro Interna 1;7459766430;7459794427;-0.3;0;-0.02;0;0;0;0.68;0.68;0.68;255;255;255;0;0|RR;Aro interna 2;7459768360;7459794427;-0.3;0;0.48;0;0;0;0.68;0.68;0.68;255;255;255;0;0|RR;Aro Meio;7459760785;7459803417;-0.44;0;0;0;0;0;0.8;0.8;0.8;255;255;255;0;0|RR;Parafusos;7459769752;7459794427;-0.3;-0.02;0.005;0;0;0;0.67;0.67;0.67;255;255;255;0;0|FR;Aro parte de fora;7459763245;7459794427;0;0;0;0;0;0;0.61;0.67;0.67;255;255;255;0;0|FR;Pneu;5937413583;4648056401;0;0;0;0;0;90;1.04;1.1;1.04;0;0;0;0;0|FR;Aro Interna 1;7459766430;7459794427;-0.3;0;-0.02;0;0;0;0.68;0.68;0.68;255;255;255;0;0|FR;Aro interna 2;7459768360;7459794427;-0.3;0;0.48;0;0;0;0.68;0.68;0.68;255;255;255;0;0|FR;Aro Meio;7459760785;7459803417;-0.44;0;0;0;0;0;0.8;0.8;0.8;255;255;255;0;0|FR;Parafusos;7459769752;7459794427;-0.3;-0.02;0.005;0;0;0;0.67;0.67;0.67;255;255;255;0;0|FL;Aro parte de fora;7459763245;7459794427;0;0;0;0;0;0;0.61;0.67;0.67;255;255;255;0;0|FL;Pneu;5937413583;4648056401;0;0;0;0;0;90;1.04;1.1;1.04;0;0;0;0;0|FL;Aro Interna 1;7459766430;7459794427;-0.3;0;-0.02;0;0;0;0.68;0.68;0.68;255;255;255;0;0|FL;Aro interna 2;7459768360;7459794427;-0.3;0;0.48;0;0;0;0.68;0.68;0.68;255;255;255;0;0|FL;Aro Meio;7459760785;7459803417;-0.44;0;0;0;0;0;0.8;0.8;0.8;255;255;255;0;0|FL;Parafusos;7459769752;7459794427;-0.3;-0.02;0.005;0;0;0;0.67;0.67;0.67;255;255;255;0;0]],
    ["Roda 2"]         = [[RR;e uma roda trouxa;3108766844;3108766958;0;0;0;0;0;0;0.25;0.25;0.25;255;255;255;0;0|FL;roda do abysall trouxa;3108766844;3108766958;0;0;0;0;0;0;0.25;0.25;0.25;255;255;255;0;0|FR;aro;3108766844;3108766958;0;0;0;0;0;0;0.25;0.25;0.25;255;255;255;0;0|RL;roda;3108766844;3108766958;0;0;0;0;0;0;0.25;0.25;0.25;255;255;255;0;0]]
}

local currentWheelData = WHEEL_PRESETS["Kranze Cerb's"]

---------------------------------------------------------------------
-- CONFIGURAÇÕES DE POSICIONAMENTO E CAMBER SEPARADO
---------------------------------------------------------------------
local frontPos = { X = 0, Y = 0, Z = 0, RotY = 0 }
local rearPos  = { X = 0, Y = 0, Z = 0, RotY = 0 }
local camberPos = { Left = 0, Right = 0 }

local basePositions = {
    ["FL"] = Vector3.new(-2.5, 0, -4.5),
    ["FR"] = Vector3.new( 2.5, 0, -4.5),
    ["RL"] = Vector3.new(-2.5, 0,  4.5),
    ["RR"] = Vector3.new( 2.5, 0,  4.5),
}

local wheelWelds = {}
local baseAnchors = {}
local wheelsModel = nil

---------------------------------------------------------------------
-- DETECTAR CARRO OU JOGADOR
---------------------------------------------------------------------
local function getTargetPart()
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local carFolder = workspace:FindFirstChild("PlayerCarMeshes")
    if carFolder then
        local carModel = carFolder:FindFirstChild(LocalPlayer.Name .. "_Car")
        if carModel and carModel:FindFirstChild("CarRoot") then
            return carModel.CarRoot
        end
    end
    return hrp
end

---------------------------------------------------------------------
-- ATUALIZAR POSIÇÕES E ROTAÇÕES
---------------------------------------------------------------------
local function updateWheelPositions()
    for posKey, baseVector in pairs(basePositions) do
        local weld = wheelWelds[posKey]
        if weld then
            local isFront = (posKey == "FL" or posKey == "FR")
            local isLeft  = (posKey == "FL" or posKey == "RL")

            local config = isFront and frontPos or rearPos
            local camber = isLeft and camberPos.Left or camberPos.Right
            local multX  = isLeft and -1 or 1

            local finalPos = Vector3.new(
                baseVector.X + (config.X * multX),
                baseVector.Y + config.Y,
                baseVector.Z + config.Z
            )

            -- Rotação de 180 no Y para o lado Direito para o aro virar para fora
            local sideFlipY = isLeft and 0 or 180

            local finalRot = CFrame.Angles(0, math.rad(sideFlipY), 0)
                           * CFrame.Angles(0, math.rad(config.RotY * multX), math.rad(camber))

            weld.C0 = CFrame.new(finalPos) * finalRot
        end
    end
end

---------------------------------------------------------------------
-- CARREGAR RODAS NO MAPA
---------------------------------------------------------------------
local function loadWheels(dataString)
    if wheelsModel then wheelsModel:Destroy() end
    wheelWelds = {}
    baseAnchors = {}

    local targetPart = getTargetPart()

    wheelsModel = Instance.new("Model")
    wheelsModel.Name = LocalPlayer.Name .. "_Wheels"
    wheelsModel.Parent = workspace

    for posKey, posVector in pairs(basePositions) do
        local anchor = Instance.new("Part")
        anchor.Name = "Anchor_" .. posKey
        anchor.Size = Vector3.new(1, 1, 1)
        anchor.Transparency = 1
        anchor.CanCollide = false
        anchor.Anchored = false
        anchor.Massless = true
        anchor.CFrame = targetPart.CFrame * CFrame.new(posVector)
        anchor.Parent = wheelsModel

        local weld = Instance.new("Weld")
        weld.Part0 = targetPart
        weld.Part1 = anchor
        weld.C0 = CFrame.new(posVector)
        weld.Parent = anchor

        baseAnchors[posKey] = anchor
        wheelWelds[posKey] = weld
    end

    local partsList = string.split(dataString, "|")
    for _, partInfo in ipairs(partsList) do
        local data = string.split(partInfo, ";")
        if #data >= 18 then
            local posKey    = data[1]
            local pName     = data[2]
            local meshId    = data[3]
            local textureId = data[4]
            local posX      = tonumber(data[5]) or 0
            local posY      = tonumber(data[6]) or 0
            local posZ      = tonumber(data[7]) or 0
            local rotX      = tonumber(data[8]) or 0
            local rotY      = tonumber(data[9]) or 0
            local rotZ      = tonumber(data[10]) or 0
            local scaleX    = tonumber(data[11]) or 1
            local scaleY    = tonumber(data[12]) or 1
            local scaleZ    = tonumber(data[13]) or 1
            local colorR    = tonumber(data[14]) or 255
            local colorG    = tonumber(data[15]) or 255
            local colorB    = tonumber(data[16]) or 255
            local trans     = tonumber(data[17]) or 0
            local reflect   = tonumber(data[18]) or 0

            local parentAnchor = baseAnchors[posKey]
            if parentAnchor then
                local p = Instance.new("Part")
                p.Name = posKey .. "_" .. pName
                p.Size = Vector3.new(1, 1, 1)
                p.CanCollide = false
                p.CanTouch = false
                p.CanQuery = false
                p.Anchored = false
                p.Massless = true
                p.Transparency = trans
                p.Reflectance = reflect
                p.Color = Color3.fromRGB(colorR, colorG, colorB)
                p.Parent = wheelsModel

                local m = Instance.new("SpecialMesh")
                m.MeshType = Enum.MeshType.FileMesh
                if meshId ~= "" then m.MeshId = "rbxassetid://" .. meshId end
                if textureId ~= "" then m.TextureId = "rbxassetid://" .. textureId end
                m.Scale = Vector3.new(scaleX, scaleY, scaleZ)
                m.Parent = p

                local partCFrame = CFrame.new(posX, posY, posZ) * CFrame.Angles(
                    math.rad(rotX),
                    math.rad(rotY),
                    math.rad(rotZ)
                )

                local w = Instance.new("Weld")
                w.Part0 = parentAnchor
                w.Part1 = p
                w.C0 = partCFrame
                w.Parent = p

                p.CFrame = parentAnchor.CFrame * partCFrame
            end
        end
    end

    updateWheelPositions()
end

RunService.RenderStepped:Connect(function()
    if wheelsModel and wheelsModel.Parent then
        for _, p in ipairs(wheelsModel:GetChildren()) do
            if p:IsA("BasePart") and not string.find(p.Name, "Anchor_") then
                p.LocalTransparencyModifier = p.Transparency
            end
        end
    end
end)

---------------------------------------------------------------------
-- INTERFACE GRÁFICA (UI)
---------------------------------------------------------------------
local screenGui = PlayerGui:FindFirstChild("AdvancedWheelsUI") or Instance.new("ScreenGui")
screenGui.Name = "AdvancedWheelsUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

for _, c in ipairs(screenGui:GetChildren()) do c:Destroy() end

local fullSize = UDim2.new(0, 320, 0, 470)
local miniSize = UDim2.new(0, 320, 0, 32)
local isMinimized = false

local frame = Instance.new("Frame")
frame.Size = fullSize
frame.Position = UDim2.new(0.02, 0, 0.35, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
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
title.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
title.Text = "🛞 CONTROLE DE RODAS E CAMBER"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 15
title.Parent = frame

-- BOTÃO DE MINIMIZAR (-)
local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 24, 0, 24)
minBtn.Position = UDim2.new(1, -28, 0, 4)
minBtn.Text = "-"
minBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 75)
minBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minBtn.Font = Enum.Font.SourceSansBold
minBtn.TextSize = 18
minBtn.ZIndex = 5
minBtn.Parent = frame

local minBtnCorner = Instance.new("UICorner")
minBtnCorner.CornerRadius = UDim.new(0, 4)
minBtnCorner.Parent = minBtn

local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, 0, 1, -32)
scroll.Position = UDim2.new(0, 0, 0, 32)
scroll.BackgroundTransparency = 1
scroll.CanvasSize = UDim2.new(0, 0, 0, 680)
scroll.ScrollBarThickness = 6
scroll.Parent = frame

-- EVENTO DO BOTÃO MINIMIZAR
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

-- SEÇÃO PRESETS (BOTOES PRONTOS)
local presetTitle = Instance.new("TextLabel")
presetTitle.Size = UDim2.new(0.9, 0, 0, 20)
presetTitle.Position = UDim2.new(0.05, 0, 0.01, 0)
presetTitle.Text = "--- PRESETS DE RODA ---"
presetTitle.TextColor3 = Color3.fromRGB(255, 215, 0)
presetTitle.BackgroundTransparency = 1
presetTitle.Font = Enum.Font.SourceSansBold
presetTitle.TextSize = 14
presetTitle.Parent = scroll

local btnKranze = Instance.new("TextButton")
btnKranze.Size = UDim2.new(0.43, 0, 0, 28)
btnKranze.Position = UDim2.new(0.05, 0, 0.045, 0)
btnKranze.Text = "Kranze Cerb's"
btnKranze.BackgroundColor3 = Color3.fromRGB(50, 100, 180)
btnKranze.TextColor3 = Color3.fromRGB(255, 255, 255)
btnKranze.Font = Enum.Font.SourceSansBold
btnKranze.Parent = scroll

local btnRoda2 = Instance.new("TextButton")
btnRoda2.Size = UDim2.new(0.43, 0, 0, 28)
btnRoda2.Position = UDim2.new(0.52, 0, 0.045, 0)
btnRoda2.Text = "Roda 2"
btnRoda2.BackgroundColor3 = Color3.fromRGB(150, 80, 40)
btnRoda2.TextColor3 = Color3.fromRGB(255, 255, 255)
btnRoda2.Font = Enum.Font.SourceSansBold
btnRoda2.Parent = scroll

btnKranze.MouseButton1Click:Connect(function()
    currentWheelData = WHEEL_PRESETS["Kranze Cerb's"]
    loadWheels(currentWheelData)
end)

btnRoda2.MouseButton1Click:Connect(function()
    currentWheelData = WHEEL_PRESETS["Roda 2"]
    loadWheels(currentWheelData)
end)

-- CAMPO PARA STRING PERSONALIZADA
local boxTitle = Instance.new("TextLabel")
boxTitle.Size = UDim2.new(0.9, 0, 0, 20)
boxTitle.Position = UDim2.new(0.05, 0, 0.09, 0)
boxTitle.Text = "Ou cole outra String de Roda:"
boxTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
boxTitle.BackgroundTransparency = 1
boxTitle.Font = Enum.Font.SourceSans
boxTitle.TextXAlignment = Enum.TextXAlignment.Left
boxTitle.Parent = scroll

local textBox = Instance.new("TextBox")
textBox.Size = UDim2.new(0.9, 0, 0, 26)
textBox.Position = UDim2.new(0.05, 0, 0.12, 0)
textBox.PlaceholderText = "Cole aqui outra string de roda..."
textBox.Text = ""
textBox.ClearTextOnFocus = false
textBox.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
textBox.TextColor3 = Color3.fromRGB(255, 255, 255)
textBox.Font = Enum.Font.SourceSans
textBox.TextSize = 12
textBox.Parent = scroll

local btnLoad = Instance.new("TextButton")
btnLoad.Size = UDim2.new(0.9, 0, 0, 24)
btnLoad.Position = UDim2.new(0.05, 0, 0.16, 0)
btnLoad.Text = "🔄 CARREGAR STRING DA CAIXA"
btnLoad.BackgroundColor3 = Color3.fromRGB(40, 140, 60)
btnLoad.TextColor3 = Color3.fromRGB(255, 255, 255)
btnLoad.Font = Enum.Font.SourceSansBold
btnLoad.Parent = scroll

btnLoad.MouseButton1Click:Connect(function()
    if textBox.Text ~= "" then
        currentWheelData = textBox.Text
        loadWheels(currentWheelData)
    end
end)

-- FUNÇÃO GERADORA DE CONTROLES
local function addSectionHeader(text, topPosY)
    local h = Instance.new("TextLabel")
    h.Size = UDim2.new(0.9, 0, 0, 22)
    h.Position = UDim2.new(0.05, 0, topPosY, 0)
    h.Text = "--- " .. text .. " ---"
    h.TextColor3 = Color3.fromRGB(255, 215, 0)
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
        updateWheelPositions()
    end

    btnM.MouseButton1Click:Connect(function() modify(-1) end)
    btnP.MouseButton1Click:Connect(function() modify(1) end)
end

-- SEÇÃO RODAS DA FRENTE
addSectionHeader("RODAS DA FRENTE (FL / FR)", 0.22)
createControlRow("Largura (X)", 0.26, frontPos, "X", 0.2, false)
createControlRow("Altura (Y)",  0.30, frontPos, "Y", 0.2, false)
createControlRow("Frente/Trás (Z)", 0.34, frontPos, "Z", 0.2, false)
createControlRow("Girar Y (Direção)", 0.38, frontPos, "RotY", 5, true)

-- SEÇÃO RODAS DE TRÁS
addSectionHeader("RODAS DE TRÁS (RL / RR)", 0.45)
createControlRow("Largura (X)", 0.49, rearPos, "X", 0.2, false)
createControlRow("Altura (Y)",  0.53, rearPos, "Y", 0.2, false)
createControlRow("Frente/Trás (Z)", 0.57, rearPos, "Z", 0.2, false)
createControlRow("Girar Y (Direção)", 0.61, rearPos, "RotY", 5, true)

-- SEÇÃO CAMBER POR LADO
addSectionHeader("CAMBER (POR LADO)", 0.68)
createControlRow("Camber Esquerda (FL/RL)", 0.72, camberPos, "Left", 2, true)
createControlRow("Camber Direita (FR/RR)",  0.76, camberPos, "Right", 2, true)

-- CARREGA A RODA PADRÃO
loadWheels(currentWheelData)
