-- Fly Script com botão para celular (compatível com anti-cheat)
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local flying = false
local speed = 50
local fixedHeight = 60
local control = {F = 0, B = 0, L = 0, R = 0}
local spawnPosition = hrp.Position

-- Interface botão
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "FlyUI"
screenGui.ResetOnSpawn = false

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 100, 0, 40)
toggleButton.Position = UDim2.new(1, -110, 0.85, 0)
toggleButton.AnchorPoint = Vector2.new(0, 0)
toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
toggleButton.Text = "Fly: OFF"
toggleButton.TextScaled = true
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Parent = screenGui

-- Voo
local function fly()
	if flying then return end
	flying = true
	humanoid.PlatformStand = true

	RunService.RenderStepped:Connect(function()
		if not flying then return end

		local rayOrigin = hrp.Position
		local rayDirection = Vector3.new(0, -200, 0)
		local raycastParams = RaycastParams.new()
		raycastParams.FilterDescendantsInstances = {character}
		raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

		local raycastResult = Workspace:Raycast(rayOrigin, rayDirection, raycastParams)
		local groundY = raycastResult and raycastResult.Position.Y or 0
		local desiredY = groundY + fixedHeight

		local camLook = Workspace.CurrentCamera.CFrame.LookVector
		local moveDir = Vector3.new(control.L + control.R, 0, control.F + control.B)
		moveDir = Workspace.CurrentCamera.CFrame:VectorToWorldSpace(moveDir)
		moveDir = Vector3.new(moveDir.X, 0, moveDir.Z).Unit * speed

		local vertical = Vector3.new(0, (desiredY - hrp.Position.Y) * 2, 0)

		-- Move o personagem como se estivesse voando
		humanoid:Move(moveDir + vertical, true)
	end)
end

-- Parar voo
local function stopFlying()
	flying = false
	humanoid.PlatformStand = false
end

toggleButton.MouseButton1Click:Connect(function()
	if flying then
		stopFlying()
		toggleButton.Text = "Fly: OFF"
		toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
	else
		fly()
		toggleButton.Text = "Fly: ON"
		toggleButton.BackgroundColor3 = Color3.fromRGB(30, 200, 30)
	end
end)

-- Controles
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then control.F = -1 end
	if input.KeyCode == Enum.KeyCode.S then control.B = 1 end
	if input.KeyCode == Enum.KeyCode.A then control.L = -1 end
	if input.KeyCode == Enum.KeyCode.D then control.R = 1 end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then control.F = 0 end
	if input.KeyCode == Enum.KeyCode.S then control.B = 0 end
	if input.KeyCode == Enum.KeyCode.A then control.L = 0 end
	if input.KeyCode == Enum.KeyCode.D then control.R = 0 end
end)
