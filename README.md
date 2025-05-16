--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

--// Referências
local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local Tool = script.Parent

--// Configurações
local ParteParaMirar = "Head" -- ou "HumanoidRootPart"
local DANO = 50
local DISTANCIA_MAXIMA = 500
local AimbotAtivo = false

--// Interface: botão Aimbot mobile
local gui = Instance.new("ScreenGui")
gui.Name = "AimbotUI"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local botao = Instance.new("TextButton")
botao.Size = UDim2.new(0, 120, 0, 50)
botao.Position = UDim2.new(1, -130, 1, -60)
botao.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
botao.TextColor3 = Color3.new(1, 1, 1)
botao.Text = "Aimbot: OFF"
botao.Font = Enum.Font.SourceSansBold
botao.TextSize = 20
botao.Parent = gui

--// Função: encontrar jogador mais próximo
local function JogadorMaisProximo()
	local menorDistancia = math.huge
	local alvoMaisProximo = nil

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(ParteParaMirar) then
			local parte = player.Character[ParteParaMirar]
			local distancia = (parte.Position - Camera.CFrame.Position).Magnitude
			if distancia < menorDistancia then
				menorDistancia = distancia
				alvoMaisProximo = parte
			end
		end
	end

	return alvoMaisProximo
end

--// Aimbot
RunService.RenderStepped:Connect(function()
	if AimbotAtivo then
		local alvo = JogadorMaisProximo()
		if alvo then
			local direcao = (alvo.Position - Camera.CFrame.Position).Unit
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + direcao)
		end
	end
end)

botao.MouseButton1Click:Connect(function()
	AimbotAtivo = not AimbotAtivo
	botao.Text = AimbotAtivo and "Aimbot: ON" or "Aimbot: OFF"
	botao.BackgroundColor3 = AimbotAtivo and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(30, 30, 30)
end)

--// Função: detectar o primeiro inimigo na direção da câmera, ignorando paredes
local function DetectarAlvoAtravessandoParedes()
	local origem = Camera.CFrame.Position
	local direcao = Camera.CFrame.LookVector * DISTANCIA_MAXIMA

	-- RaycastParams para ignorar paredes e só detectar jogadores
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Blacklist
	params.FilterDescendantsInstances = {LocalPlayer.Character}

	local resultado = Workspace:Raycast(origem, direcao, params)

	-- Mesmo se não bater em nada, vamos verificar manualmente quem está na linha do tiro
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
			local humanoidPart = player.Character.HumanoidRootPart
			local direcaoAlvo = (humanoidPart.Position - origem).Unit
			local distancia = (humanoidPart.Position - origem).Magnitude

			-- Verifica se o ângulo entre onde a câmera aponta e o alvo é pequeno (está na frente)
			local angulo = math.deg(math.acos(direcaoAlvo:Dot(Camera.CFrame.LookVector)))
			if angulo < 5 and distancia <= DISTANCIA_MAXIMA then
				return player.Character.Humanoid
			end
		end
	end

	return nil
end

--// Quando atira
Tool.Activated:Connect(function()
	local alvo = DetectarAlvoAtravessandoParedes()
	if alvo then
		alvo:TakeDamage(DANO)
	end
end)
