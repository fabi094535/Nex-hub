--==================================================
-- OPEN SOURCE PET SCANNER HUB
-- Para usar no seu próprio jogo Roblox
--==================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Remove versão anterior
local old = playerGui:FindFirstChild("OpenSourcePetHub")
if old then
	old:Destroy()
end

--==================================================
-- CONFIGURAÇÃO
--==================================================

local CONFIG = {
	ScanFolder = workspace,

	MaxPets = 20,

	-- Nomes que podem representar valor
	ValueNames = {
		"Value",
		"Price",
		"Cost",
		"PetValue",
		"Cash",
		"Money"
	},

	-- Nomes que podem representar raridade
	RarityNames = {
		"Rarity",
		"Rare",
		"Tier"
	}
}

--==================================================
-- GUI
--==================================================

local gui = Instance.new("ScreenGui")
gui.Name = "OpenSourcePetHub"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(620, 450)
main.Position = UDim2.fromScale(0.5, 0.5)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.BackgroundColor3 = Color3.fromRGB(7, 15, 16)
main.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 16)
corner.Parent = main

local border = Instance.new("UIStroke")
border.Color = Color3.fromRGB(35, 170, 110)
border.Thickness = 2
border.Parent = main

--==================================================
-- HEADER
--==================================================

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 65)
header.BackgroundColor3 = Color3.fromRGB(9, 23, 24)
header.Parent = main

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 16)
headerCorner.Parent = header

local title = Instance.new("TextLabel")
title.BackgroundTransparency = 1
title.Text = "OPEN SOURCE PET HUB"
title.TextColor3 = Color3.fromRGB(245, 255, 250)
title.Font = Enum.Font.GothamBold
title.TextSize = 21
title.Size = UDim2.new(1, -130, 1, 0)
title.Position = UDim2.fromOffset(18, 0)
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

--==================================================
-- BOTÕES DO HEADER
--==================================================

local minimize = Instance.new("TextButton")
minimize.Size = UDim2.fromOffset(40, 40)
minimize.Position = UDim2.new(1, -95, 0, 12)
minimize.BackgroundColor3 = Color3.fromRGB(15, 40, 35)
minimize.Text = "—"
minimize.TextColor3 = Color3.new(1,1,1)
minimize.TextSize = 22
minimize.Font = Enum.Font.GothamBold
minimize.Parent = header

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 10)
minCorner.Parent = minimize

local close = Instance.new("TextButton")
close.Size = UDim2.fromOffset(40, 40)
close.Position = UDim2.new(1, -48, 0, 12)
close.BackgroundColor3 = Color3.fromRGB(45, 20, 22)
close.Text = "×"
close.TextColor3 = Color3.new(1,1,1)
close.TextSize = 25
close.Font = Enum.Font.GothamBold
close.Parent = header

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 10)
closeCorner.Parent = close

--==================================================
-- ÁREA PRINCIPAL
--==================================================

local content = Instance.new("Frame")
content.BackgroundTransparency = 1
content.Position = UDim2.fromOffset(15, 80)
content.Size = UDim2.new(1, -30, 1, -95)
content.Parent = main

local scanButton = Instance.new("TextButton")
scanButton.Size = UDim2.fromOffset(160, 42)
scanButton.Position = UDim2.fromOffset(0, 0)
scanButton.BackgroundColor3 = Color3.fromRGB(15, 55, 43)
scanButton.Text = "🔎 SCAN PETS"
scanButton.TextColor3 = Color3.new(1,1,1)
scanButton.TextSize = 15
scanButton.Font = Enum.Font.GothamBold
scanButton.Parent = content

local scanCorner = Instance.new("UICorner")
scanCorner.CornerRadius = UDim.new(0, 10)
scanCorner.Parent = scanButton

local status = Instance.new("TextLabel")
status.BackgroundTransparency = 1
status.Position = UDim2.fromOffset(175, 0)
status.Size = UDim2.new(1, -175, 0, 42)
status.Text = "Aguardando scan..."
status.TextColor3 = Color3.fromRGB(170, 190, 185)
status.TextSize = 14
status.Font = Enum.Font.Gotham
status.TextXAlignment = Enum.TextXAlignment.Left
status.Parent = content

--==================================================
-- LISTA
--==================================================

local list = Instance.new("ScrollingFrame")
list.Position = UDim2.fromOffset(0, 55)
list.Size = UDim2.new(1, 0, 1, -55)
list.BackgroundColor3 = Color3.fromRGB(5, 12, 13)
list.BorderSizePixel = 0
list.ScrollBarThickness = 5
list.CanvasSize = UDim2.new(0,0,0,0)
list.Parent = content

local listCorner = Instance.new("UICorner")
listCorner.CornerRadius = UDim.new(0, 12)
listCorner.Parent = list

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 6)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = list

--==================================================
-- FUNÇÕES
--==================================================

local function getNumber(object)

	for _, name in ipairs(CONFIG.ValueNames) do

		local attribute = object:GetAttribute(name)

		if typeof(attribute) == "number" then
			return attribute
		end

		local child = object:FindFirstChild(name)

		if child then

			if child:IsA("NumberValue") or
				child:IsA("IntValue") then

				return child.Value
			end

		end
	end

	return 0
end

local function getRarity(object)

	for _, name in ipairs(CONFIG.RarityNames) do

		local attribute = object:GetAttribute(name)

		if attribute ~= nil then
			return tostring(attribute)
		end

		local child = object:FindFirstChild(name)

		if child and child:IsA("StringValue") then
			return child.Value
		end
	end

	return "Desconhecido"
end

local function looksLikePet(object)

	local name = string.lower(object.Name)

	local keywords = {
		"pet",
		"egg",
		"dog",
		"cat",
		"dragon",
		"bird",
		"spider",
		"mantis",
		"legendary",
		"mythic"
	}

	for _, word in ipairs(keywords) do

		if string.find(name, word, 1, true) then
			return true
		end

	end

	-- Também considera objetos com valor/raridade
	if getNumber(object) > 0 then
		return true
	end

	return false
end

local function clearList()

	for _, child in ipairs(list:GetChildren()) do

		if child:IsA("Frame") then
			child:Destroy()
		end

	end

end

local function createPetRow(data, index)

	local row = Instance.new("Frame")

	row.Size = UDim2.new(1, -10, 0, 62)
	row.BackgroundColor3 = Color3.fromRGB(10, 25, 26)
	row.LayoutOrder = index
	row.Parent = list

	local rowCorner = Instance.new("UICorner")
	rowCorner.CornerRadius = UDim.new(0, 10)
	rowCorner.Parent = row

	local number = Instance.new("TextLabel")
	number.BackgroundTransparency = 1
	number.Size = UDim2.fromOffset(45, 62)
	number.Text = "#" .. index
	number.TextColor3 = Color3.fromRGB(150, 170, 165)
	number.TextSize = 15
	number.Font = Enum.Font.GothamBold
	number.Parent = row

	local name = Instance.new("TextLabel")
	name.BackgroundTransparency = 1
	name.Position = UDim2.fromOffset(50, 7)
	name.Size = UDim2.new(0.45, 0, 0, 27)
	name.Text = data.Name
	name.TextColor3 = Color3.fromRGB(240, 250, 245)
	name.TextSize = 16
	name.Font = Enum.Font.GothamBold
	name.TextXAlignment = Enum.TextXAlignment.Left
	name.Parent = row

	local rarity = Instance.new("TextLabel")
	rarity.BackgroundTransparency = 1
	rarity.Position = UDim2.fromOffset(50, 34)
	rarity.Size = UDim2.new(0.45, 0, 0, 20)
	rarity.Text = data.Rarity
	rarity.TextColor3 = Color3.fromRGB(170, 100, 255)
	rarity.TextSize = 12
	rarity.Font = Enum.Font.Gotham
	rarity.TextXAlignment = Enum.TextXAlignment.Left
	rarity.Parent = row

	local value = Instance.new("TextLabel")
	value.BackgroundTransparency = 1
	value.Position = UDim2.new(0.55, 0, 0, 7)
	value.Size = UDim2.new(0.25, 0, 0, 25)
	value.Text = "$" .. tostring(data.Value)
	value.TextColor3 = Color3.fromRGB(50, 230, 105)
	value.TextSize = 15
	value.Font = Enum.Font.GothamBold
	value.TextXAlignment = Enum.TextXAlignment.Right
	value.Parent = row

	local select = Instance.new("TextButton")
	select.Size = UDim2.fromOffset(85, 36)
	select.Position = UDim2.new(1, -95, 0, 13)
	select.BackgroundColor3 = Color3.fromRGB(20, 60, 48)
	select.Text = "SELECIONAR"
	select.TextColor3 = Color3.new(1,1,1)
	select.TextSize = 11
	select.Font = Enum.Font.GothamBold
	select.Parent = row

	local selectCorner = Instance.new("UICorner")
	selectCorner.CornerRadius = UDim.new(0, 8)
	selectCorner.Parent = select

	select.MouseButton1Click:Connect(function()

		status.Text =
			"Selecionado: " ..
			data.Name ..
			" | " ..
			data.Rarity

	end)

end

--==================================================
-- SCANNER
--==================================================

local function scanPets()

	clearList()

	status.Text = "Escaneando..."

	local pets = {}

	for _, object in ipairs(CONFIG.ScanFolder:GetDescendants()) do

		if object:IsA("Model")
			or object:IsA("Folder")
			or object:IsA("BasePart") then

			if looksLikePet(object) then

				table.insert(pets, {
					Object = object,
					Name = object.Name,
					Value = getNumber(object),
					Rarity = getRarity(object)
				})

			end

		end

	end

	-- Maior valor primeiro
	table.sort(pets, function(a,b)

		return a.Value > b.Value

	end)

	local count = math.min(#pets, CONFIG.MaxPets)

	for i = 1, count do
		createPetRow(pets[i], i)
	end

	list.CanvasSize =
		UDim2.fromOffset(0, count * 68)

	status.Text =
		"Encontrados: " .. tostring(#pets)

end

scanButton.MouseButton1Click:Connect(scanPets)

--==================================================
-- 1º / 2º PLANO
--==================================================

local minimized = false

minimize.MouseButton1Click:Connect(function()

	minimized = not minimized

	if minimized then

		-- Segundo plano/minimizado
		content.Visible = false

		main.Size =
			UDim2.fromOffset(300, 65)

	else

		-- Primeiro plano/aberto
		content.Visible = true

		main.Size =
			UDim2.fromOffset(620, 450)

	end

end)

close.MouseButton1Click:Connect(function()
	gui:Destroy()
end)

--==================================================
-- ARRASTAR JANELA
--==================================================

local dragging = false
local dragStart
local startPosition

header.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		dragging = true
		dragStart = input.Position
		startPosition = main.Position

	end

end)

header.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		dragging = false

	end

end)

UserInputService.InputChanged:Connect(function(input)

	if not dragging then
		return
	end

	if input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch then

		local delta =
			input.Position - dragStart

		main.Position =
			UDim2.new(
				startPosition.X.Scale,
				startPosition.X.Offset + delta.X,

				startPosition.Y.Scale,
				startPosition.Y.Offset + delta.Y
			)

	end

end)

print("Open Source Pet Hub carregado!")
