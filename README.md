local ReplicatedStorage = game:GetService("ReplicatedStorage")
local slapEvent = ReplicatedStorage:WaitForChild("DefenderSlapEvent")

local Players = game:GetService("Players")

local COOLDOWN_TIME = 2 -- segundos
local playerCooldowns = {}

slapEvent.OnServerEvent:Connect(function(player, force)
	if not force or type(force) ~= "number" then return end
	
	-- Checar cooldown
	if playerCooldowns[player] and tick() - playerCooldowns[player] < COOLDOWN_TIME then
		return
	end
	playerCooldowns[player] = tick()
	
	local character = player.Character
	if not character then return end
	
	local hrp = character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	
	-- Raycast para frente para achar alvo
	local rayOrigin = hrp.Position
	local rayDirection = hrp.CFrame.LookVector * 7
	
	local raycastParams = RaycastParams.new()
	raycastParams.FilterDescendantsInstances = {character}
	raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
	
	local raycastResult = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
	
	if raycastResult and raycastResult.Instance then
		local hitPart = raycastResult.Instance
		local hitCharacter = hitPart:FindFirstAncestorOfClass("Model")
		if hitCharacter then
			local humanoid = hitCharacter:FindFirstChildOfClass("Humanoid")
			local targetHRP = hitCharacter:FindFirstChild("HumanoidRootPart")
			
			if humanoid and targetHRP and hitCharacter ~= character then
				-- Aplicar força do slap
				local direction = (targetHRP.Position - hrp.Position).Unit
				targetHRP.Velocity = direction * force * 50 -- multiplicar pra dar um impacto bom
				
				-- Pode adicionar som ou efeito visual aqui, se quiser
			end
		end
	end
end)
