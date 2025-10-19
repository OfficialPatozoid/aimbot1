-- LocalScript (StarterGui)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local Enabled = false
local MaxDistance = 300
local FOVRadius = 150
local Smoothness = 0.2

-- GUI base
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotUI"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- === CÍRCULO FIXO (no centro da tela) ===
local fovCircle = Instance.new("Frame")
fovCircle.Size = UDim2.new(0,FOVRadius*2,0,FOVRadius*2)
fovCircle.AnchorPoint = Vector2.new(0.5,0.5)
fovCircle.Position = UDim2.new(0.5,0,0.5,0)
fovCircle.BackgroundTransparency = 1
fovCircle.BorderSizePixel = 0
fovCircle.Parent = screenGui

local circleStroke = Instance.new("UIStroke")
circleStroke.Thickness = 2
circleStroke.Color = Color3.fromRGB(255,60,150)
circleStroke.Transparency = 0.25
circleStroke.Parent = fovCircle

local circleCorner = Instance.new("UICorner")
circleCorner.CornerRadius = UDim.new(1,0)
circleCorner.Parent = fovCircle

-- === PAINEL LUXO ===
local panel = Instance.new("Frame")
panel.Size = UDim2.new(0,250,0,170)
panel.Position = UDim2.new(0,20,0,20)
panel.BackgroundColor3 = Color3.fromRGB(20,20,20)
panel.BackgroundTransparency = 0.2
panel.BorderSizePixel = 0
panel.Parent = screenGui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0,18)

-- Glow externo
local panelStroke = Instance.new("UIStroke")
panelStroke.Thickness = 2
panelStroke.Color = Color3.fromRGB(255,70,200)
panelStroke.Transparency = 0.4
panelStroke.Parent = panel

-- Gradiente neon
local grad = Instance.new("UIGradient")
grad.Color = ColorSequence.new{
	ColorSequenceKeypoint.new(0, Color3.fromRGB(60,0,80)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(20,20,20))
}
grad.Rotation = 45
grad.Parent = panel

task.spawn(function()
	while task.wait(3) do
		grad.Rotation = (grad.Rotation + 60) % 360
	end
end)

-- === TÍTULO PROFISSIONAL ===
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,32)
title.BackgroundTransparency = 1
title.Text = "⚡ AIMBOT ULTRA"
title.TextColor3 = Color3.fromRGB(255,100,200)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.Parent = panel

local titleStroke = Instance.new("UIStroke")
titleStroke.Thickness = 2
titleStroke.Color = Color3.fromRGB(0,0,0)
titleStroke.Parent = title

-- === BOTÃO TOGGLE ===
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0.9,0,0,36)
toggleBtn.Position = UDim2.new(0.05,0,0,38)
toggleBtn.BackgroundColor3 = Color3.fromRGB(90,0,90)
toggleBtn.Text = "Aimbot: OFF"
toggleBtn.TextColor3 = Color3.new(1,1,1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 16
toggleBtn.AutoButtonColor = true
toggleBtn.Parent = panel
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0,12)

-- Função criar caixas de texto estilosas
local function newBox(y,text,value)
	local lbl = Instance.new("TextLabel")
	lbl.Size = UDim2.new(0.5,-10,0,24)
	lbl.Position = UDim2.new(0.05,0,0,y)
	lbl.BackgroundTransparency = 1
	lbl.Text = text
	lbl.TextColor3 = Color3.fromRGB(255,200,255)
	lbl.Font = Enum.Font.Gotham
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	lbl.TextSize = 14
	lbl.Parent = panel

	local box = Instance.new("TextBox")
	box.Size = UDim2.new(0.4,0,0,24)
	box.Position = UDim2.new(0.55,0,0,y)
	box.Text = tostring(value)
	box.BackgroundColor3 = Color3.fromRGB(35,0,50)
	box.TextColor3 = Color3.new(1,1,1)
	box.Font = Enum.Font.GothamBold
	box.TextSize = 14
	box.ClearTextOnFocus = false
	box.Parent = panel
	Instance.new("UICorner", box).CornerRadius = UDim.new(0,8)

	local bs = Instance.new("UIStroke")
	bs.Color = Color3.fromRGB(255,100,255)
	bs.Thickness = 1.2
	bs.Parent = box

	return box
end

local fovBox = newBox(85,"FOV:",FOVRadius)
local distBox = newBox(115,"Distância:",MaxDistance)

-- === FUNÇÕES AIMBOT ===
local function isValidTarget(player)
	if not player or player == LocalPlayer or not player.Character then return false end
	local hrp = player.Character:FindFirstChild("HumanoidRootPart")
	local hum = player.Character:FindFirstChildOfClass("Humanoid")
	if not hrp or not hum or hum.Health <= 0 then return false end
	if (hrp.Position - Camera.CFrame.Position).Magnitude > MaxDistance then return false end
	return true
end

local function getClosestTarget()
	local best, dist = nil, math.huge
	local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
	for _,pl in ipairs(Players:GetPlayers()) do
		if isValidTarget(pl) then
			local hrp = pl.Character.HumanoidRootPart
			local pos, visible = Camera:WorldToViewportPoint(hrp.Position)
			if visible then
				local vec = Vector2.new(pos.X,pos.Y)
				local diff = (vec - center).Magnitude
				if diff < dist and diff <= FOVRadius then
					dist = diff
					best = pl
				end
			end
		end
	end
	return best
end

local function aimAt(target)
	if not target or not target.Character then return end
	local hrp = target.Character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	local camPos = Camera.CFrame.Position
	local look = CFrame.new(camPos, hrp.Position)
	Camera.CFrame = Camera.CFrame:Lerp(look, Smoothness)
end

-- Atualizar valores
fovBox.FocusLost:Connect(function()
	local n = tonumber(fovBox.Text)
	if n and n > 20 and n < 800 then
		FOVRadius = n
		fovCircle.Size = UDim2.new(0,FOVRadius*2,0,FOVRadius*2)
	else
		fovBox.Text = tostring(FOVRadius)
	end
end)

distBox.FocusLost:Connect(function()
	local n = tonumber(distBox.Text)
	if n and n > 50 and n < 10000 then
		MaxDistance = n
	else
		distBox.Text = tostring(MaxDistance)
	end
end)

-- Toggle botão
toggleBtn.MouseButton1Click:Connect(function()
	Enabled = not Enabled
	toggleBtn.Text = Enabled and "Aimbot: ON" or "Aimbot: OFF"
	toggleBtn.BackgroundColor3 = Enabled and Color3.fromRGB(0,150,0) or Color3.fromRGB(90,0,90)
end)

-- Loop render
RunService.RenderStepped:Connect(function()
	if not Enabled then
		circleStroke.Transparency = 0.5
		return
	end
	local t = getClosestTarget()
	if t then
		aimAt(t)
		circleStroke.Transparency = 0
	else
		circleStroke.Transparency = 0.5
	end
end)
