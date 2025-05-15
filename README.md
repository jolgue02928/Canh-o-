local ts = game:GetService("TweenService")
local rs = game:GetService("RunService")
local plr = game.Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local hum = char:WaitForChild("Humanoid")
local startCF = hrp.CFrame

local function isUnanchored(m)
	for _, p in pairs(m:GetDescendants()) do
		if p:IsA("BasePart") and not p.Anchored then
			return true
		end
	end
	return false
end

local function findCannon()
	local exclude = nil
	local fort = workspace:FindFirstChild("FortConstitution")
	if fort then
		exclude = fort:FindFirstChild("Cannon", true)
	end
	for _, d in ipairs(workspace:GetDescendants()) do
		if d:IsA("Model") and d.Name == "Cannon" and d ~= exclude then
			return d
		end
	end
	return nil
end

local tween = ts:Create(hrp, TweenInfo.new(20, Enum.EasingStyle.Linear), { CFrame = CFrame.new(-9, 3, -50000) })
tween:Play()

local conn
conn = rs.RenderStepped:Connect(function()
	local can = findCannon()
	if can and isUnanchored(can) then
		local seat = can:FindFirstChildWhichIsA("VehicleSeat", true)
		if seat and not seat.Occupant then
			tween:Cancel()
			conn:Disconnect()
			hrp.CFrame = seat.CFrame
			seat:Sit(hum)

			task.delay(1, function()
				if hum.Sit then
					hum.Sit = false
					hum:ChangeState(Enum.HumanoidStateType.Jumping)
				end
				task.delay(1, function()
					seat:Sit(hum)
					task.delay(1, function()
						hrp.CFrame = startCF
						hum.JumpPower = 0
						hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
						task.wait(1)
						_G.CannonDone = true
					end)
				end)
			end)
		end
	end
end)
