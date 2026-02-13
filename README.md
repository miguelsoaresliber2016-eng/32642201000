local player = game.Players.LocalPlayer
local RunService = game:GetService("RunService")

local ativo = false
local platform
local connection

local alturaMaxima = 12
local velocidade = 4 -- 🚀 SPEED 4

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "trikerx"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,220,0,120)
frame.Position = UDim2.new(0.5,-110,0.6,0)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.Active = true
frame.Draggable = true
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,20)

-- 🌈 BORDA ARCO-ÍRIS
local stroke = Instance.new("UIStroke")
stroke.Thickness = 3
stroke.Parent = frame

task.spawn(function()
	while true do
		for i = 0, 1, 0.01 do
			stroke.Color = Color3.fromHSV(i,1,1)
			task.wait(0.03)
		end
	end
end)

local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1,0,0,35)
titulo.Text = "TRIKER X"
titulo.BackgroundTransparency = 1
titulo.TextColor3 = Color3.new(1,1,1)
titulo.TextScaled = true
titulo.Font = Enum.Font.GothamBold
titulo.Parent = frame

local botao = Instance.new("TextButton")
botao.Size = UDim2.new(0,160,0,45)
botao.Position = UDim2.new(0.5,-80,0.6,-5)
botao.Text = "PLATAFORMA"
botao.TextScaled = true
botao.Font = Enum.Font.GothamBold
botao.TextColor3 = Color3.new(1,1,1)
botao.BackgroundColor3 = Color3.fromRGB(0,170,255)
botao.Parent = frame
Instance.new("UICorner", botao).CornerRadius = UDim.new(0,18)

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
			alturaAtual += velocidade
			if alturaAtual > alturaMaxima then
				alturaAtual = alturaMaxima
			end
		end

		platform.Position = Vector3.new(
			hrp.Position.X,
			baseY + alturaAtual,
			hrp.Position.Z
		)

		-- Anti atravessar
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

	if connection then
		connection:Disconnect()
		connection = nil
	end

	if platform then
		platform:Destroy()
		platform = nil
	end
end

botao.MouseButton1Click:Connect(function()
	if ativo then
		desativar()
	else
		ativar()
	end
end)
