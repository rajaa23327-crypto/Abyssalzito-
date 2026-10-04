local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

local uiParent = (gethui and gethui()) or game:GetService("CoreGui")

if uiParent:FindFirstChild("PainelAuraDelta") then
	uiParent.PainelAuraDelta:Destroy()
end

-- ==========================================
-- CRIANDO A INTERFACE (UI)
-- ==========================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PainelAuraDelta"
ScreenGui.Parent = uiParent

-- ==========================================
-- MENU PRINCIPAL (FRAME)
-- ==========================================
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 220, 0, 260)
Frame.Position = UDim2.new(0.5, -110, 0.5, -130)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = Frame

-- Título Atualizado
local Titulo = Instance.new("TextLabel")
Titulo.Size = UDim2.new(1, 0, 0, 40)
Titulo.BackgroundTransparency = 1
Titulo.Text = "Customized Glows"
Titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
Titulo.Font = Enum.Font.GothamBold
Titulo.TextSize = 14
Titulo.Parent = Frame

-- ==========================================
-- BOTÃO DE MINIMIZAR (BOLINHA 🤟)
-- ==========================================
local Bolinha = Instance.new("TextButton")
Bolinha.Size = UDim2.new(0, 45, 0, 45)
Bolinha.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Bolinha.Text = "🤟"
Bolinha.TextSize = 22
Bolinha.Visible = false -- Fica invisível até você fechar o menu
Bolinha.Parent = ScreenGui

local BolinhaCorner = Instance.new("UICorner")
BolinhaCorner.CornerRadius = UDim.new(1, 0) -- Deixa perfeitamente redondo (bolinha)
BolinhaCorner.Parent = Bolinha

-- Botão de Fechar/Minimizar (O 'X' no menu)
local BotaoFechar = Instance.new("TextButton")
BotaoFechar.Size = UDim2.new(0, 30, 0, 30)
BotaoFechar.Position = UDim2.new(1, -35, 0, 5)
BotaoFechar.BackgroundTransparency = 1
BotaoFechar.Text = "X"
BotaoFechar.TextColor3 = Color3.fromRGB(255, 50, 50)
BotaoFechar.Font = Enum.Font.GothamBold
BotaoFechar.TextSize = 16
BotaoFechar.Parent = Frame

-- Lógica para Minimizar
BotaoFechar.MouseButton1Click:Connect(function()
	-- Faz a bolinha aparecer exatamente onde o menu estava
	Bolinha.Position = UDim2.new(0, Frame.AbsolutePosition.X + 87, 0, Frame.AbsolutePosition.Y + 10)
	Frame.Visible = false
	Bolinha.Visible = true
end)

-- Lógica para Maximizar (Abrir de volta) sem atrapalhar o arrastar
local tempoClique = 0
Bolinha.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		tempoClique = tick() -- Marca a hora que tocou
	end
end)

Bolinha.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		-- Se tocou rápido (menos de 0.2 segundos), ele abre o menu. Se demorou, foi porque arrastou!
		if tick() - tempoClique < 0.2 then
			Frame.Visible = true
			Bolinha.Visible = false
		end
	end
end)


-- ==========================================
-- SISTEMA DE ROLAGEM E INPUTS (Scroll)
-- ==========================================
local Scroll = Instance.new("ScrollingFrame")
Scroll.Size = UDim2.new(1, 0, 1, -45)
Scroll.Position = UDim2.new(0, 0, 0, 40)
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.ScrollBarThickness = 4
Scroll.CanvasSize = UDim2.new(0, 0, 0, 350)
Scroll.Parent = Frame

local function criarInput(nome, placeholder, posY)
	local TextBox = Instance.new("TextBox")
	TextBox.Name = nome
	TextBox.Size = UDim2.new(0, 180, 0, 35)
	TextBox.Position = UDim2.new(0.5, -90, 0, posY)
	TextBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
	TextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
	TextBox.PlaceholderText = placeholder
	TextBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
	TextBox.Font = Enum.Font.Gotham
	TextBox.TextSize = 12
	TextBox.Text = ""
	TextBox.Parent = Scroll
	
	local Corner = Instance.new("UICorner")
	Corner.CornerRadius = UDim.new(0, 5)
	Corner.Parent = TextBox
	return TextBox
end

local inputId = criarInput("InputID", "ID (Ex: 12345)", 5)
local inputX = criarInput("InputX", "Tam X (Ex: 6)", 45)
local inputY = criarInput("InputY", "Tam Y (Ex: 0.1)", 85)
local inputZ = criarInput("InputZ", "Tam Z (Ex: 6)", 125)

local inputPosX = criarInput("InputPosX", "Posição X (Lados): 0", 175)
local inputPosY = criarInput("InputPosY", "Posição Y (Altura): 0", 215)
local inputPosZ = criarInput("InputPosZ", "Posição Z (Frente): 0", 255)

local BotaoAplicar = Instance.new("TextButton")
BotaoAplicar.Size = UDim2.new(0, 180, 0, 35)
BotaoAplicar.Position = UDim2.new(0.5, -90, 0, 305)
BotaoAplicar.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
BotaoAplicar.TextColor3 = Color3.fromRGB(255, 255, 255)
BotaoAplicar.Font = Enum.Font.GothamBold
BotaoAplicar.Text = "Aplicar Glow"
BotaoAplicar.TextSize = 14
BotaoAplicar.Parent = Scroll

local BtnCorner = Instance.new("UICorner")
BtnCorner.CornerRadius = UDim.new(0, 5)
BtnCorner.Parent = BotaoAplicar

-- ==========================================
-- SISTEMA PARA ARRASTAR (DRAGGABLE)
-- ==========================================
-- Lógica para arrastar o Painel Principal (Segurando o Título)
local dragging, dragInput, dragStart, startPos

Titulo.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = Frame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then dragging = false end
		end)
	end
end)

Titulo.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

-- Lógica para arrastar a Bolinha
local bolinhaDragging, bolinhaDragInput, bolinhaDragStart, bolinhaStartPos
Bolinha.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		bolinhaDragging = true
		bolinhaDragStart = input.Position
		bolinhaStartPos = Bolinha.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then bolinhaDragging = false end
		end)
	end
end)
Bolinha.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		bolinhaDragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		Frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
	if input == bolinhaDragInput and bolinhaDragging then
		local delta = input.Position - bolinhaDragStart
		Bolinha.Position = UDim2.new(bolinhaStartPos.X.Scale, bolinhaStartPos.X.Offset + delta.X, bolinhaStartPos.Y.Scale, bolinhaStartPos.Y.Offset + delta.Y)
	end
end)

-- ==========================================
-- LÓGICA DO GLOW (IMAGEM)
-- ==========================================
BotaoAplicar.MouseButton1Click:Connect(function()
	local character = player.Character
	if not character then return end
	
	local hrp = character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	local idDaImagem = inputId.Text:match("%d+")
	local sizeX = tonumber(inputX.Text) or 6
	local sizeY = tonumber(inputY.Text) or 0.1
	local sizeZ = tonumber(inputZ.Text) or 6

	local posX = tonumber(inputPosX.Text) or 0
	local posY = tonumber(inputPosY.Text) or 0
	local posZ = tonumber(inputPosZ.Text) or 0

	local velhaAura = character:FindFirstChild("AuraImagemPart")
	if velhaAura then velhaAura:Destroy() end

	local auraPart = Instance.new("Part")
	auraPart.Name = "AuraImagemPart"
	auraPart.Anchored = false
	auraPart.CanCollide = false
	auraPart.Massless = true
	auraPart.Size = Vector3.new(sizeX, sizeY, sizeZ)
	auraPart.Transparency = 1 
	
	local surfaceGui = Instance.new("SurfaceGui")
	surfaceGui.Name = "ImagemGui"
	surfaceGui.Face = Enum.NormalId.Top
	surfaceGui.LightInfluence = 0 
	surfaceGui.SizingMode = Enum.SurfaceGuiSizingMode.PixelsPerStud
	surfaceGui.PixelsPerStud = 50
	surfaceGui.Parent = auraPart

	local imageLabel = Instance.new("ImageLabel")
	imageLabel.Size = UDim2.new(1, 0, 1, 0)
	imageLabel.BackgroundTransparency = 1 
	
	if idDaImagem then
		imageLabel.Image = "rbxthumb://type=Asset&id=" .. idDaImagem .. "&w=420&h=420"
	end
	
	imageLabel.Parent = surfaceGui
	
	local weld = Instance.new("Weld")
	weld.Part0 = hrp
	weld.Part1 = auraPart
	weld.C0 = CFrame.new(posX, -2.9 + posY, posZ) 
	weld.Parent = auraPart
	
	auraPart.Parent = character
end)
