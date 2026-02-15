local player = game.Players.LocalPlayer
local RunService = game:GetService("RunService")

local KEY_CORRETA = "FF4"
local ADM_KEY = "Qw12As"

local antiLagAtivo = false
local threshold = 1
local lastCF
local lastVel
local stopFlag = false
local antiLagHeartbeatConn
local antiLagPropConn
local antiLagDiedConn

local antiLagInactiveColor = Color3.fromRGB(64, 0, 128)
local antiLagActiveColor = Color3.fromRGB(200, 162, 255)

-- Variáveis de estado dos botões
local plataformaAtiva = false
local godAtivo = false
local plataformaObj = nil
local godConnection = nil
local plataformaConnection = nil

-- Configurações da plataforma (copiadas do primeiro script)
local alturaMaxima = 14
local velocidade = 14

-- FUNÇÃO GOD MODE (copiada do primeiro script)
local function toggleGodMode()
    local char = player.Character
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    godAtivo = not godAtivo

    if godAtivo then
        -- LIGAR God Mode
        if godConnection then godConnection:Disconnect() end
        godConnection = humanoid.HealthChanged:Connect(function()
            humanoid.Health = humanoid.MaxHealth
        end)
        humanoid.Health = humanoid.MaxHealth -- garantir que comece cheio
        print("God LIGADO")
    else
        -- DESLIGAR God Mode
        if godConnection then
            godConnection:Disconnect()
            godConnection = nil
        end
        print("God DESLIGADO")
    end
end

-- FUNÇÃO PLATAFORMA (copiada do primeiro script)
local function togglePlataforma()
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then
        return
    end
    local hrp = char.HumanoidRootPart

    if not plataformaAtiva then
        -- CRIAR PLATAFORMA
        plataformaObj = Instance.new("Part")
        plataformaObj.Name = "PlataformaTemp"
        plataformaObj.Size = Vector3.new(10, 1.2, 10)  -- tamanho do primeiro script
        plataformaObj.Position = hrp.Position - Vector3.new(0, 3, 0)
        plataformaObj.Anchored = true
        plataformaObj.CanCollide = true
        plataformaObj.Material = Enum.Material.Neon
        plataformaObj.Color = Color3.fromRGB(0, 170, 255)  -- cor do primeiro script
        plataformaObj.Transparency = 0.3
        plataformaObj.Parent = workspace

        local baseY = plataformaObj.Position.Y
        local alturaAtual = 0

        -- CONEXÃO PARA SUBIR E SEGUIR
        plataformaConnection = RunService.RenderStepped:Connect(function()
            if not plataformaObj or not plataformaObj.Parent then
                -- Se a plataforma foi destruída, desconecta
                if plataformaConnection then
                    plataformaConnection:Disconnect()
                    plataformaConnection = nil
                end
                return
            end

            -- Atualiza altura (sobe até o limite)
            if alturaAtual < alturaMaxima then
                alturaAtual = alturaAtual + velocidade * 0.1  -- 0.1 é o tempo médio de um RenderStepped? Ajuste fino
                if alturaAtual > alturaMaxima then
                    alturaAtual = alturaMaxima
                end
            end

            -- Posiciona a plataforma embaixo do jogador, na altura atual
            plataformaObj.Position = Vector3.new(
                hrp.Position.X,
                baseY + alturaAtual,
                hrp.Position.Z
            )

            -- Empurra o jogador para cima da plataforma (como no primeiro script)
            local topo = plataformaObj.Position.Y + (plataformaObj.Size.Y / 2)
            if hrp.Position.Y < topo + 2.8 then
                hrp.CFrame = CFrame.new(hrp.Position.X, topo + 2.8, hrp.Position.Z)
                hrp.AssemblyLinearVelocity = Vector3.zero
            end
        end)

        plataformaAtiva = true
    else
        -- DESTRUIR PLATAFORMA
        if plataformaObj then
            plataformaObj:Destroy()
            plataformaObj = nil
        end
        if plataformaConnection then
            plataformaConnection:Disconnect()
            plataformaConnection = nil
        end
        plataformaAtiva = false
    end
end

---------------------------------------------------
-- FUNÇÃO GUI PRINCIPAL
---------------------------------------------------

local function criarGUI(isAdm)

    local gui = Instance.new("ScreenGui")
    gui.ResetOnSpawn = false
    gui.Parent = player:WaitForChild("PlayerGui")

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0,240,0,150)
    frame.Position = UDim2.new(0.5,-120,0.6,0)
    frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
    frame.Active = true
    frame.Draggable = true
    frame.Parent = gui
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0,20)

    -- BORDA ARCO IRIS
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
    titulo.Text = isAdm and "Adm Trk" or "TRIKER X"
    titulo.BackgroundTransparency = 1
    titulo.TextColor3 = Color3.new(1,1,1)
    titulo.TextScaled = true
    titulo.Font = Enum.Font.GothamBold
    titulo.Parent = frame

    -- BOTÃO -
    local minimizar = Instance.new("TextButton")
    minimizar.Size = UDim2.new(0,30,0,30)
    minimizar.Position = UDim2.new(1,-35,0,2)
    minimizar.Text = "-"
    minimizar.TextScaled = true
    minimizar.Font = Enum.Font.GothamBold
    minimizar.TextColor3 = Color3.new(1,1,1)
    minimizar.BackgroundColor3 = Color3.fromRGB(60,60,60)
    minimizar.Parent = frame
    Instance.new("UICorner", minimizar).CornerRadius = UDim.new(1,0)

    local minimizado = false

    ---------------------------------------------------
    -- MODO NORMAL
    ---------------------------------------------------
    if not isAdm then

        -- BOTÃO ADM
        local admBotao = Instance.new("TextButton")
        admBotao.Size = UDim2.new(0,30,0,30)
        admBotao.Position = UDim2.new(0,5,0,2)
        admBotao.Text = "A"
        admBotao.TextScaled = true
        admBotao.Font = Enum.Font.GothamBold
        admBotao.TextColor3 = Color3.new(1,1,1)
        admBotao.BackgroundColor3 = Color3.fromRGB(40,40,40)
        admBotao.Parent = frame
        Instance.new("UICorner", admBotao).CornerRadius = UDim.new(1,0)

        local admStroke = Instance.new("UIStroke")
        admStroke.Thickness = 2
        admStroke.Parent = admBotao

        task.spawn(function()
            while gui.Parent do
                for i=0,1,0.01 do
                    admStroke.Color = Color3.fromHSV(i,1,1)
                    task.wait(0.03)
                end
            end
        end)

        admBotao.MouseButton1Click:Connect(function()
            criarAdmKeyGUI()
        end)

        -- BOTÃO PLATAFORMA (com cores do primeiro script)
        local plataformaBtn = Instance.new("TextButton")
        plataformaBtn.Size = UDim2.new(0,130,0,45)
        plataformaBtn.Position = UDim2.new(0.05,0,0.6,-5)
        plataformaBtn.Text = "PLATAFORMA"
        plataformaBtn.TextScaled = true
        plataformaBtn.Font = Enum.Font.GothamBold
        plataformaBtn.TextColor3 = Color3.new(1,1,1)
        plataformaBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)  -- Azul inativo
        plataformaBtn.Parent = frame
        Instance.new("UICorner", plataformaBtn).CornerRadius = UDim.new(0,18)

        plataformaBtn.MouseButton1Click:Connect(function()
            togglePlataforma()
            if plataformaAtiva then
                plataformaBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)  -- Verde ativo
                plataformaBtn.Text = "PLATAFORMA ON"
            else
                plataformaBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)  -- Azul inativo
                plataformaBtn.Text = "PLATAFORMA"
            end
        end)

        -- BOTÃO GOD (com cores do primeiro script)
        local godBtn = Instance.new("TextButton")
        godBtn.Size = UDim2.new(0,45,0,45)
        godBtn.Position = UDim2.new(0.75,0,0.6,-5)
        godBtn.Text = "G"
        godBtn.TextScaled = true
        godBtn.Font = Enum.Font.GothamBold
        godBtn.TextColor3 = Color3.new(1,1,1)
        godBtn.BackgroundColor3 = Color3.fromRGB(170, 0, 0)  -- Vermelho inativo
        godBtn.Parent = frame
        Instance.new("UICorner", godBtn).CornerRadius = UDim.new(1,0)

        godBtn.MouseButton1Click:Connect(function()
            toggleGodMode()
            if godAtivo then
                godBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)  -- Verde ativo
                godBtn.Text = "GOD ON"
            else
                godBtn.BackgroundColor3 = Color3.fromRGB(170, 0, 0)  -- Vermelho inativo
                godBtn.Text = "G"
            end
        end)

        -- Botão minimizar
        minimizar.MouseButton1Click:Connect(function()
            minimizado = not minimizado
            plataformaBtn.Visible = not minimizado
            godBtn.Visible = not minimizado
            admBotao.Visible = not minimizado
            frame.Size = minimizado and UDim2.new(0,240,0,40) or UDim2.new(0,240,0,150)
            minimizar.Text = minimizado and "+" or "-"
        end)

    ---------------------------------------------------
    -- MODO ADM
    ---------------------------------------------------
    else
        local antiLagBotao = Instance.new("TextButton")
        antiLagBotao.Size = UDim2.new(0,70,0,45)
        antiLagBotao.Position = UDim2.new(0.5,-35,0.6,-5)
        antiLagBotao.Text = "AL"
        antiLagBotao.TextScaled = true
        antiLagBotao.Font = Enum.Font.GothamBold
        antiLagBotao.TextColor3 = Color3.new(1,1,1)
        antiLagBotao.BackgroundColor3 = antiLagInactiveColor
        antiLagBotao.Parent = frame
        Instance.new("UICorner", antiLagBotao).CornerRadius = UDim.new(0,15)

        minimizar.MouseButton1Click:Connect(function()
            minimizado = not minimizado
            antiLagBotao.Visible = not minimizado
            frame.Size = minimizado and UDim2.new(0,240,0,40) or UDim2.new(0,240,0,150)
            minimizar.Text = minimizado and "+" or "-"
        end)

        -- LÓGICA AL (inalterada)
        local function cleanup()
            if antiLagHeartbeatConn then antiLagHeartbeatConn:Disconnect() end
            if antiLagPropConn then antiLagPropConn:Disconnect() end
            if antiLagDiedConn then antiLagDiedConn:Disconnect() end
        end

        local function setup()
            local char = player.Character
            if not char then return end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if not hrp then return end

            lastCF = hrp.CFrame
            lastVel = hrp.AssemblyLinearVelocity
            stopFlag = false

            antiLagHeartbeatConn = RunService.Heartbeat:Connect(function()
                if not antiLagAtivo or stopFlag then return end
                lastCF = hrp.CFrame
                lastVel = hrp.AssemblyLinearVelocity
            end)

            antiLagPropConn = hrp:GetPropertyChangedSignal("CFrame"):Connect(function()
                if not antiLagAtivo then return end
                local dist = (hrp.CFrame.Position - lastCF.Position).Magnitude
                if dist > threshold then
                    stopFlag = true
                    hrp.CFrame = lastCF
                    hrp.AssemblyLinearVelocity = lastVel
                    task.wait(0.01)
                    stopFlag = false
                end
            end)
        end

        antiLagBotao.MouseButton1Click:Connect(function()
            antiLagAtivo = not antiLagAtivo
            if antiLagAtivo then
                antiLagBotao.BackgroundColor3 = antiLagActiveColor
                setup()
            else
                antiLagBotao.BackgroundColor3 = antiLagInactiveColor
                cleanup()
            end
        end)
    end
end

---------------------------------------------------
-- GUI KEY
---------------------------------------------------

local function criarKeyGUI(key, isAdm)

    local keyGui = Instance.new("ScreenGui")
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

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 0.6
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
    titulo.Text = isAdm and "digite a key ADM" or "digite a key"
    titulo.BackgroundTransparency = 1
    titulo.TextScaled = true
    titulo.Font = Enum.Font.GothamBold
    titulo.TextColor3 = Color3.new(1,1,1)
    titulo.Parent = frame

    local minimizar = Instance.new("TextButton")
    minimizar.Size = UDim2.new(0,30,0,30)
    minimizar.Position = UDim2.new(1,-35,0,2)
    minimizar.Text = "-"
    minimizar.TextScaled = true
    minimizar.Font = Enum.Font.GothamBold
    minimizar.TextColor3 = Color3.new(1,1,1)
    minimizar.BackgroundColor3 = Color3.fromRGB(60,60,60)
    minimizar.Parent = frame
    Instance.new("UICorner", minimizar).CornerRadius = UDim.new(1,0)

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

    local minimizado = false

    minimizar.MouseButton1Click:Connect(function()
        minimizado = not minimizado
        box.Visible = not minimizado
        frame.Size = minimizado and UDim2.new(0,240,0,40) or UDim2.new(0,240,0,130)
        minimizar.Text = minimizado and "+" or "-"
    end)

    box.FocusLost:Connect(function()
        if box.Text == key then
            keyGui:Destroy()
            criarGUI(isAdm)
        else
            box.Text = ""
            box.PlaceholderText = "KEY ERRADA"
        end
    end)
end

---------------------------------------------------
-- INICIO
---------------------------------------------------

function criarAdmKeyGUI()
    criarKeyGUI(ADM_KEY, true)
end

criarKeyGUI(KEY_CORRETA, false)
