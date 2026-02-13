local player = game.Players.LocalPlayer
local RunService = game:GetService("RunService")

local KEY_CORRETA = "FF4xMlK"

-- CONFIG
local alturaMaxima = 21
local velocidade = 17

-- ESTADOS
local ativo = false
local platform
local connection
local godAtivo = false
local godConnection

---------------------------------------------------
-- FUNÇÃO CRIAR GUI PLATAFORMA
---------------------------------------------------

local function criarPlataformaGUI()

	local gui = Instance.new("ScreenGui")
	gui.Name = "trikerx_main"
	gui.ResetOnSpawn = false
	gui.Parent = player:WaitForChild("PlayerGui")

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0,240,0,130)
	frame.Position = UDim2.new(0.5,-120,0.6,0)
	frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
	frame.Active = true
	frame.Draggable = true
	frame.Parent = gui
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0,20)

	-- RAINBOW
	local stroke = Instance.new("UIStroke")
	stroke.Thickness = 3
	stroke.Parent = frame

	task.spawn(function()
		while gui.Parent do
			for i=0,1,0.01 do
				stroke.Color = Color3.fromHSV(i,1,1)
				task.wait(0.03)
			end
		end
	end)

	-- TITULO
	local titulo = Instance.new("TextLabel")
	titulo.Size = UDim2.new(1,0,0,35)
	titulo.Text = "TRIKER X"
	titulo.BackgroundTransparency = 1
	titulo.TextColor3 = Color3.new(1,1,1)
	titulo.TextScaled = true
	titulo.Font = Enum.Font.GothamBold
	titulo.Parent = frame

	-- BOTÃO PLATAFORMA
	local botao = Instance.new("TextButton")
	botao.Size = UDim2.new(0,150,0,45)
	botao.Position = UDim2.new(0.08,0,0.6,-5)
	botao.Text = "PLATAFORMA"
	botao.TextScaled = true
	botao.Font = Enum.Font.GothamBold
	botao.TextColor3 = Color3.new(1,1,1)
	botao.BackgroundColor3 = Color3.fromRGB(0,170,255)
	botao.Parent = frame
	Instance.new("UICorner", botao).CornerRadius = UDim.new(0,18)

	-- BOTÃO GOD PEQUENO REDONDO
	local godBotao = Instance.new("TextButton")
	godBotao.Size = UDim2.new(0,45,0,45)
	godBotao.Position = UDim2.new(0.75,0,0.6,-5)
	godBotao.Text = "G"
	godBotao.TextScaled = true
	godBotao.Font = Enum.Font.GothamBold
	godBotao.TextColor3 = Color3.new(1,1,1)
	godBotao.BackgroundColor3 = Color3.fromRGB(170,0,0)
	godBotao.Parent = frame
	Instance.new("UICorner", godBotao).CornerRadius = UDim.new(1,0) -- totalmente redondo

	---------------------------------------------------
	-- PLATAFORMA
	---------------------------------------------------

	local function ativar()
		if ativo then return end
		ativo = true
		botao.BackgroundColor3 = Color3.fromRGB(0,200,100)

		local character = player.Character or player.CharacterAdded:Wait()
		local hrp = character:WaitForChild("HumanoidRootPart")

		platform = Instance.new("Part")
		platform.Size = Vector3.new(10,1.2,10)
		platform.Material = Enum.Material.Neon
		platform.Color = Color3.fromRGB(0,170,255)
		platform.Anchored = true
		platform.CanCollide = true
		platform.Parent = workspace

		local baseY = hrp.Position.Y - 3
		local alturaAtual = 0

		connection = RunService.RenderStepped:Connect(function()
			if not ativo then return end

			if alturaAtual < alturaMaxima then
				alturaAtual += velocidade * 0.1
				if alturaAtual > alturaMaxima then
					alturaAtual = alturaMaxima
				end
			end

			platform.Position = Vector3.new(
				hrp.Position.X,
				baseY + alturaAtual,
				hrp.Position.Z
			)

			local topo = platform.Position.Y + (platform.Size.Y / 2)
			if hrp.Position.Y < topo + 2.8 then
				hrp.CFrame = CFrame.new(
					hrp.Position.X,
					topo + 2.8,
					hrp.Position.Z
				)
				hrp.AssemblyLinearVelocity = Vector3.zero
			end
		end)
	end

	local function desativar()
		ativo = false
		botao.BackgroundColor3 = Color3.fromRGB(0,170,255)

		if connection then connection:Disconnect() end
		if platform then platform:Destroy() end
	end

	botao.MouseButton1Click:Connect(function()
		if ativo then desativar() else ativar() end
	end)

	---------------------------------------------------
	-- GOD MODE
	---------------------------------------------------

	godBotao.MouseButton1Click:Connect(function()
		local character = player.Character or player.CharacterAdded:Wait()
		local humanoid = character:WaitForChild("Humanoid")

		if not godAtivo then
			godAtivo = true
			godBotao.BackgroundColor3 = Color3.fromRGB(0,200,0)

			godConnection = humanoid.HealthChanged:Connect(function()
				if humanoid.Health < humanoid.MaxHealth then
					humanoid.Health = humanoid.MaxHealth
				end
			end)
		else
			godAtivo = false
			godBotao.BackgroundColor3 = Color3.fromRGB(170,0,0)
			if godConnection then godConnection:Disconnect() end
		end
	end)
end

---------------------------------------------------
-- GUI KEY PRIMEIRO
---------------------------------------------------

local keyGui = Instance.new("ScreenGui")
keyGui.Name = "trikerx_key"
keyGui.ResetOnSpawn = false
keyGui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,240,0,130)
frame.Position = UDim2.new(0.5,-120,0.6,0)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.Active = true
frame.Draggable = true
frame.Parent = keyGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,20)

-- RAINBOW
local stroke = Instance.new("UIStroke")
stroke.Thickness = 3
stroke.Parent = frame

task.spawn(function()
	while keyGui.Parent do
		for i=0,1,0.01 do
			stroke.Color = Color3.fromHSV(i,1,1)
			task.wait(0.03)
		end
	end
end)

local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1,0,0,35)
titulo.Text = "DIGITE A KEY"
titulo.BackgroundTransparency = 1
titulo.TextColor3 = Color3.new(1,1,1)
titulo.TextScaled = true
titulo.Font = Enum.Font.GothamBold
titulo.Parent = frame

local box = Instance.new("TextBox")
box.Size = UDim2.new(0,160,0,45)
box.Position = UDim2.new(0.5,-80,0.6,-5)
box.PlaceholderText = "Coloque a Key"
box.TextScaled = true
box.Font = Enum.Font.GothamBold
box.TextColor3 = Color3.new(1,1,1)
box.BackgroundColor3 = Color3.fromRGB(0,170,255)
box.Parent = frame
Instance.new("UICorner", box).CornerRadius = UDim.new(0,18)

box.FocusLost:Connect(function()
	if box.Text == KEY_CORRETA then
		keyGui:Destroy()
		criarPlataformaGUI()
	else
		box.Text = ""
		box.PlaceholderText = "KEY ERRADA"
	end
end)
