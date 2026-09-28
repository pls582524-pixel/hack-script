-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- Configuration & State
local scriptEnabled = false
local fovRadius = 150 -- Default FOV radius in pixels
local lockedTarget = nil

-- Create UI Elements
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PandaControlPanel"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 220, 0, 180)
MainFrame.Position = UDim2.new(0.05, 0, 0.1, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true -- Allows moving the UI around easily
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.Text = "Control Panel"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 16
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- Toggle Button
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0.9, 0, 0, 35)
ToggleButton.Position = UDim2.new(0.05, 0, 0.25, 0)
ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
ToggleButton.Text = "Status: OFF"
ToggleButton.TextColor3 = Color3.fromRGB(255, 100, 100)
ToggleButton.TextSize = 14
ToggleButton.Font = Enum.Font.GothamMedium
ToggleButton.Parent = MainFrame

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 6)
ToggleCorner.Parent = ToggleButton

-- FOV Label / Display
local FovLabel = Instance.new("TextLabel")
FovLabel.Size = UDim2.new(0.9, 0, 0, 30)
FovLabel.Position = UDim2.new(0.05, 0, 0.5, 0)
FovLabel.BackgroundTransparency = 1
FovLabel.Text = "FOV Radius: " .. fovRadius
FovLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
FovLabel.TextSize = 13
FovLabel.Font = Enum.Font.Gotham
FovLabel.Parent = MainFrame

-- Target Lock Indicator
local TargetLabel = Instance.new("TextLabel")
TargetLabel.Size = UDim2.new(0.9, 0, 0, 30)
TargetLabel.Position = UDim2.new(0.05, 0, 0.7, 0)
TargetLabel.BackgroundTransparency = 1
TargetLabel.Text = "Target: None"
TargetLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
TargetLabel.TextSize = 13
TargetLabel.Font = Enum.Font.Gotham
TargetLabel.Parent = MainFrame

-- Visual FOV Circle using Drawing API (or UI alternative)
-- Note: Drawing API works natively in client viewports for testing
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Transparency = 0.7
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1
FOVCircle.Radius = fovRadius
FOVCircle.Filled = false

-- Toggle Functionality
ToggleButton.MouseButton1Click:Connect(function()
	scriptEnabled = not scriptEnabled
	if scriptEnabled then
		ToggleButton.Text = "Status: ON"
		ToggleButton.TextColor3 = Color3.fromRGB(100, 255, 100)
		FOVCircle.Visible = true
	else
		ToggleButton.Text = "Status: OFF"
		ToggleButton.TextColor3 = Color3.fromRGB(255, 100, 100)
		FOVCircle.Visible = false
		lockedTarget = nil
		TargetLabel.Text = "Target: None"
	end
end)

-- Find closest target within FOV
local function getClosestTargetInFOV()
	local closestTarget = nil
	local shortestDistance = fovRadius
	local mousePos = Vector2.new(Mouse.X, Mouse.Y)

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
			if humanoid and humanoid.Health > 0 then
				local rootPart = player.Character.HumanoidRootPart
				local screenPoint, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
				
				if onScreen then
					local screenPos = Vector2.new(screenPoint.X, screenPoint.Y)
					local distance = (screenPos - mousePos).Magnitude
					
					if distance < shortestDistance then
						shortestDistance = distance
						closestTarget = rootPart
					end
				end
			end
		end
	end
	return closestTarget
end

-- Main Loop
RunService.RenderStepped:Connect(function()
	if not scriptEnabled then return end
	
	-- Update FOV circle position to mouse coordinates
	FOVCircle.Position = Vector2.new(Mouse.X, Mouse.Y + 36) -- Offset for top bar if needed
	
	-- Target Lock logic (Hold Right Mouse Button to lock)
	if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
		if not lockedTarget then
			lockedTarget = getClosestTargetInFOV()
		end
		
		if lockedTarget then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, lockedTarget.Position)
			TargetLabel.Text = "Target: Locked"
			TargetLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
		end
	else
		lockedTarget = nil
		TargetLabel.Text = "Target: Searching..."
		TargetLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
	end
end)
# hack-script
arono
