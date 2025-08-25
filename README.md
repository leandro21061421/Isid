
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

local attacking = false
local humanoid = nil
local maxDistance = 30   -- distância máxima para considerar inimigos
local attackRange = 10   -- distância mínima para atacar (só bate se estiver bem perto)

-- ===== GUI =====
local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "AutoAttackGui"
screenGui.ResetOnSpawn = false -- não some ao morrer

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 180)
frame.Position = UDim2.new(0.5, -110, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
frame.Parent = screenGui

local attackButton = Instance.new("TextButton")
attackButton.Size = UDim2.new(1, -20, 0, 40)
attackButton.Position = UDim2.new(0, 10, 0, 10)
attackButton.Text = "Auto Attack: OFF"
attackButton.TextColor3 = Color3.fromRGB(255,255,255)
attackButton.BackgroundColor3 = Color3.fromRGB(70,70,70)
attackButton.Parent = frame

local distanceBox = Instance.new("TextBox")
distanceBox.Size = UDim2.new(1, -20, 0, 40)
distanceBox.Position = UDim2.new(0, 10, 0, 60)
distanceBox.PlaceholderText = "Distância Máxima (ex: 30)"
distanceBox.Text = tostring(maxDistance)
distanceBox.TextColor3 = Color3.fromRGB(255,255,255)
distanceBox.BackgroundColor3 = Color3.fromRGB(50,50,50)
distanceBox.ClearTextOnFocus = false
distanceBox.Parent = frame

local rangeBox = Instance.new("TextBox")
rangeBox.Size = UDim2.new(1, -20, 0, 40)
rangeBox.Position = UDim2.new(0, 10, 0, 110)
rangeBox.PlaceholderText = "Distância Ataque (ex: 10)"
rangeBox.Text = tostring(attackRange)
rangeBox.TextColor3 = Color3.fromRGB(255,255,255)
rangeBox.BackgroundColor3 = Color3.fromRGB(50,50,50)
rangeBox.ClearTextOnFocus = false
rangeBox.Parent = frame

distanceBox.FocusLost:Connect(function()
	if tonumber(distanceBox.Text) then
		maxDistance = tonumber(distanceBox.Text)
	end
end)

rangeBox.FocusLost:Connect(function()
	if tonumber(rangeBox.Text) then
		attackRange = tonumber(rangeBox.Text)
	end
end)

-- ===== Função para achar inimigo mais próximo =====
local function getClosestEnemy()
	local closest, dist = nil, math.huge
	if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return nil end

	local root = LocalPlayer.Character.HumanoidRootPart

	for _, npc in pairs(workspace:GetChildren()) do
		if npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") and npc ~= LocalPlayer.Character then
			if npc.Humanoid.Health > 0 then
				local mag = (npc.HumanoidRootPart.Position - root.Position).Magnitude
				if mag < dist and mag <= maxDistance then
					dist = mag
					closest = npc
				end
			end
		end
	end
	return closest, dist
end

-- ===== Atacar =====
RunService.Heartbeat:Connect(function()
	if attacking then
		local target, dist = getClosestEnemy()
		if target and dist <= attackRange then
			local tool = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool")
			if tool then
				tool:Activate() -- só ataca se o inimigo estiver na range configurada
			end
		end
	end
end)

-- ===== Botão liga/desliga =====
attackButton.MouseButton1Click:Connect(function()
	attacking = not attacking
	if attacking then
		attackButton.Text = "Auto Attack: ON"
	else
		attackButton.Text = "Auto Attack: OFF"
	end
end)

-- ===== Atualiza humanoid sempre que renascer =====
local function updateHumanoid()
	if LocalPlayer.Character then
		humanoid = LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid")
	end
end

LocalPlayer.CharacterAdded:Connect(function()
	task.wait(1) -- espera carregar
	updateHumanoid()
end)

updateHumanoid()
