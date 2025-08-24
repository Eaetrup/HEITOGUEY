local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

local function criarUICorner(obj)
	local uic = Instance.new("UICorner")
	uic.CornerRadius = UDim.new(0, 8)
	uic.Parent = obj
end

-- Criar GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "JuniorMenu"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 600, 0, 400)
frame.Position = UDim2.new(0.5, -300, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.Visible = false
frame.Active = true
frame.Draggable = true
criarUICorner(frame)

local titulo = Instance.new("TextLabel", frame)
titulo.Size = UDim2.new(1, 0, 0, 40)
titulo.Text = "Junior Menu"
titulo.BackgroundTransparency = 1
titulo.TextColor3 = Color3.fromRGB(0, 255, 255)
titulo.Font = Enum.Font.GothamBold
titulo.TextScaled = true

-- Variáveis
local espAtivo = false
local speedAtivo = false
local noclipAtivo = false
local speedValue = 50
local aimbotAtivo = false
local alcanceAimbot = 70
local aimConn = nil
local espObjects = {}

local abas = {"Visual", "Aimbot", "Speed", "Noclip", "Teleport"}
local abasFrames = {}
local abaAtual = "Visual"

for i, nome in ipairs(abas) do
	local botao = Instance.new("TextButton", frame)
	botao.Size = UDim2.new(0, 120, 0, 40)
	botao.Position = UDim2.new(0, 0, 0, 40 * (i - 1) + 50)
	botao.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
	botao.Text = nome
	botao.Font = Enum.Font.GothamBold
	botao.TextColor3 = Color3.new(1,1,1)
	botao.TextScaled = true
	criarUICorner(botao)

	local conteudo = Instance.new("Frame", frame)
	conteudo.Position = UDim2.new(0, 130, 0, 50)
	conteudo.Size = UDim2.new(1, -140, 1, -60)
	conteudo.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
	conteudo.Visible = (nome == abaAtual)
	criarUICorner(conteudo)
	abasFrames[nome] = conteudo

	botao.MouseButton1Click:Connect(function()
		for _, fr in pairs(abasFrames) do fr.Visible = false end
		conteudo.Visible = true
		abaAtual = nome
	end)
end

-- ABA VISUAL (ESP)
local visual = abasFrames["Visual"]
local opcoesESP = { box = false, vida = false, arma = false }

local function limparESP()
	for player, objetos in pairs(espObjects) do
		for _, obj in pairs(objetos) do
			if obj and obj.Parent then
				obj:Destroy()
			end
		end
	end
	espObjects = {}
end

local function aplicarESP()
	limparESP()
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Team ~= LocalPlayer.Team and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local char = p.Character
			local objetos = {}

			if opcoesESP.box then
				local hl = Instance.new("Highlight", char)
				hl.FillColor = Color3.fromRGB(255, 0, 0)
				hl.OutlineColor = Color3.fromRGB(255, 255, 255)
				hl.FillTransparency = 0.5
				hl.Adornee = char
				table.insert(objetos, hl)
			end

			if opcoesESP.vida and char:FindFirstChild("Humanoid") then
				local gui = Instance.new("BillboardGui", char)
				gui.Size = UDim2.new(4,0,0.4,0)
				gui.StudsOffset = Vector3.new(0,5,0)
				gui.AlwaysOnTop = true
				gui.Adornee = char:FindFirstChild("HumanoidRootPart")

				local fundo = Instance.new("Frame", gui)
				fundo.Size = UDim2.new(1,0,1,0)
				fundo.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

				local barra = Instance.new("Frame", fundo)
				barra.Size = UDim2.new(char.Humanoid.Health / char.Humanoid.MaxHealth, 0, 1, 0)
				barra.BackgroundColor3 = Color3.fromRGB(0, 255, 0)

				table.insert(objetos, gui)
			end

			if opcoesESP.arma then
				local tool = "Nenhuma"
				for _, v in ipairs(char:GetChildren()) do
					if v:IsA("Tool") then tool = v.Name break end
				end

				local gui = Instance.new("BillboardGui", char)
				gui.Size = UDim2.new(0, 200, 0, 25)
				gui.StudsOffset = Vector3.new(0, 6, 0)
				gui.AlwaysOnTop = true
				gui.Adornee = char:FindFirstChild("Head")

				local txt = Instance.new("TextLabel", gui)
				txt.Size = UDim2.new(1, 0, 1, 0)
				txt.Text = "🔫 " .. tool
				txt.BackgroundTransparency = 1
				txt.TextColor3 = Color3.fromRGB(255, 255, 255)
				txt.Font = Enum.Font.GothamBold
				txt.TextScaled = true

				table.insert(objetos, gui)
			end

			espObjects[p] = objetos
		end
	end
end

local function criarToggleVisual(nome, ordem, chave)
	local btn = Instance.new("TextButton", visual)
	btn.Size = UDim2.new(0, 220, 0, 35)
	btn.Position = UDim2.new(0, 20, 0, 20 + ((ordem - 1) * 45))
	btn.Text = "🔘 " .. nome
	btn.Font = Enum.Font.Gotham
	btn.TextScaled = true
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	criarUICorner(btn)

	btn.MouseButton1Click:Connect(function()
		opcoesESP[chave] = not opcoesESP[chave]
		btn.Text = (opcoesESP[chave] and "🟢 " or "🔘 ") .. nome
		
		-- Atualizar variável global espAtivo
		espAtivo = opcoesESP.box or opcoesESP.vida or opcoesESP.arma
		
		aplicarESP()
	end)
end

criarToggleVisual("Caixa (Box)", 1, "box")
criarToggleVisual("Mostrar Vida", 2, "vida")
criarToggleVisual("Mostrar Arma", 3, "arma")

-- Atualiza ESP a cada 1 segundo enquanto ativo
task.spawn(function()
	while true do
		wait(1)
		if espAtivo then
			aplicarESP()
		end
	end
end)

-- Atualiza ESP quando jogador entra ou respawna
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		wait(1)
		if espAtivo then
			aplicarESP()
		end
	end)
end)

for _, player in pairs(Players:GetPlayers()) do
	player.CharacterAdded:Connect(function()
		wait(1)
		if espAtivo then
			aplicarESP()
		end
	end)
end

-- ABA AIMBOT
local aimbot = abasFrames["Aimbot"]
local aimConn = nil
local aimbotAtivo = false

local function iniciarAimbot()
	if aimbotAtivo then return end
	aimbotAtivo = true

	aimConn = RunService.RenderStepped:Connect(function()
		if not aimbotAtivo then return end
		local nearest, bestScore = nil, math.huge
		local camera = workspace.CurrentCamera
		local mousePos = UserInputService:GetMouseLocation()

		for _, player in pairs(Players:GetPlayers()) do
			if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team then
				local char = player.Character
				if char and char:FindFirstChild("Head") and char:FindFirstChild("Humanoid") then
					if char.Humanoid.Health > 0 then
						local head = char.Head
						local screenPos, onScreen = camera:WorldToViewportPoint(head.Position)

						if onScreen then
							local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
							local worldDist = (camera.CFrame.Position - head.Position).Magnitude
							local score = screenDist + worldDist * 2

							if score < bestScore and screenDist < 100 and worldDist < alcanceAimbot then
								bestScore = score
								nearest = head
							end
						end
					end
				end
			end
		end

		if nearest then
			camera.CFrame = CFrame.new(camera.CFrame.Position, nearest.Position)
		end
	end)
end

local function desativarAimbot()
	aimbotAtivo = false
	if aimConn then
		aimConn:Disconnect()
		aimConn = nil
	end
end

local function botaoAimbot(txt, y, callback)
	local btn = Instance.new("TextButton", aimbot)
	btn.Size = UDim2.new(0, 240, 0, 40)
	btn.Position = UDim2.new(0, 20, 0, y)
	btn.Text = txt
	btn.Font = Enum.Font.GothamBold
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	btn.TextScaled = true
	criarUICorner(btn)
	btn.MouseButton1Click:Connect(callback)
end

botaoAimbot("Aimbot Curto (70)", 20, function()
	alcanceAimbot = 70
	desativarAimbot()
	iniciarAimbot()
end)

botaoAimbot("Aimbot Longo (300)", 80, function()
	alcanceAimbot = 300
	desativarAimbot()
	iniciarAimbot()
end)

botaoAimbot("Desativar Aimbot", 140, desativarAimbot)

-- ABA SPEED
local speed = abasFrames["Speed"]
local speedAtivo = false
local speedValue = 50

local entrada = Instance.new("TextBox", speed)
entrada.Size = UDim2.new(0, 100, 0, 35)
entrada.Position = UDim2.new(0, 20, 0, 20)
entrada.PlaceholderText = "Velocidade"
entrada.Text = tostring(speedValue)
entrada.Font = Enum.Font.Gotham
entrada.TextScaled = true
entrada.ClearTextOnFocus = false
criarUICorner(entrada)

entrada.FocusLost:Connect(function()
	local val = tonumber(entrada.Text)
	if val and val >= 1 then
		speedValue = val
	else
		speedValue = 16
		entrada.Text = "16"
	end
end)

local toggleSpeed = Instance.new("TextButton", speed)
toggleSpeed.Size = UDim2.new(0, 140, 0, 35)
toggleSpeed.Position = UDim2.new(0, 140, 0, 20)
toggleSpeed.Text = "🔘 Ativar Speed"
toggleSpeed.Font = Enum.Font.GothamBold
toggleSpeed.TextScaled = true
toggleSpeed.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleSpeed.TextColor3 = Color3.new(1, 1, 1)
criarUICorner(toggleSpeed)

toggleSpeed.MouseButton1Click:Connect(function()
	speedAtivo = not speedAtivo
	toggleSpeed.Text = speedAtivo and "🟢 Speed ON" or "🔘 Ativar Speed"
	local char = LocalPlayer.Character
	if char and char:FindFirstChild("Humanoid") then
		char.Humanoid.WalkSpeed = speedAtivo and speedValue or 16
	end
end)

RunService.Stepped:Connect(function()
	local char = LocalPlayer.Character
	if speedAtivo and char and char:FindFirstChild("Humanoid") then
		if char.Humanoid.WalkSpeed ~= speedValue then
			char.Humanoid.WalkSpeed = speedValue
		end
	end
end)

-- ABA NOCLIP
local noclip = abasFrames["Noclip"]
local noclipAtivo = false

local toggleNoclip = Instance.new("TextButton", noclip)
toggleNoclip.Size = UDim2.new(0, 250, 0, 40)
toggleNoclip.Position = UDim2.new(0, 20, 0, 20)
toggleNoclip.Text = "🔘 Ativar Noclip"
toggleNoclip.Font = Enum.Font.GothamBold
toggleNoclip.TextScaled = true
toggleNoclip.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
toggleNoclip.TextColor3 = Color3.new(1, 1, 1)
criarUICorner(toggleNoclip)

toggleNoclip.MouseButton1Click:Connect(function()
	noclipAtivo = not noclipAtivo
	toggleNoclip.Text = noclipAtivo and "🟢 Noclip ON" or "🔘 Ativar Noclip"
end)

RunService.Stepped:Connect(function()
	if noclipAtivo and LocalPlayer.Character then
		for _, p in ipairs(LocalPlayer.Character:GetDescendants()) do
			if p:IsA("BasePart") then p.CanCollide = false end
		end
	end
end)

-- ABA TELEPORT
local teleport = abasFrames["Teleport"]
local teleportAtivo = false

local toggleTeleport = Instance.new("TextButton", teleport)
toggleTeleport.Size = UDim2.new(0, 260, 0, 40)
toggleTeleport.Position = UDim2.new(0, 20, 0, 20)
toggleTeleport.Text = "🔘 Ativar Teleporte"
toggleTeleport.Font = Enum.Font.GothamBold
toggleTeleport.TextScaled = true
toggleTeleport.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
toggleTeleport.TextColor3 = Color3.new(1, 1, 1)
criarUICorner(toggleTeleport)

local infoTeleport = Instance.new("TextLabel", teleport)
infoTeleport.Size = UDim2.new(1, -40, 0, 40)
infoTeleport.Position = UDim2.new(0, 20, 0, 70)
infoTeleport.BackgroundTransparency = 1
infoTeleport.TextColor3 = Color3.fromRGB(0, 255, 255)
infoTeleport.Font = Enum.Font.GothamBold
infoTeleport.TextScaled = true
infoTeleport.Text = "Clique no mapa para teleportar (só funciona ativado)"

toggleTeleport.MouseButton1Click:Connect(function()
	teleportAtivo = not teleportAtivo
	toggleTeleport.Text = teleportAtivo and "🟢 Teleporte ATIVADO" or "🔘 Ativar Teleporte"
end)

UserInputService.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if teleportAtivo and input.UserInputType == Enum.UserInputType.MouseButton1 then
		local mouse = LocalPlayer:GetMouse()
		local char = LocalPlayer.Character
		if mouse and char and char:FindFirstChild("HumanoidRootPart") then
			local target = mouse.Hit
			if target and target.Position then
				char:MoveTo(Vector3.new(target.Position.X, target.Position.Y + 3, target.Position.Z))
			end
		end
	end
end)

-- Abrir/Fechar menu com Delete
UserInputService.InputBegan:Connect(function(input, gpe)
	if not gpe and input.KeyCode == Enum.KeyCode.Delete then
		frame.Visible = not frame.Visible
	end
end)
