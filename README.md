# Puffer-basic-Speed-GUI-
local speedUI = {}
speedUI.enabled = true
speedUI.speed = 0
speedUI.maxSpeed = 200
speedUI.acceleration = 50
speedUI.friction = 30

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = game.Players.LocalPlayer
if not player then
	return
end

local playerGui = player:WaitForChild("PlayerGui", 5)
if not playerGui then
	return
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpeedUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 300, 0, 350)
mainFrame.Position = UDim2.new(0, 20, 0.5, -175)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

local headerBar = Instance.new("Frame")
headerBar.Name = "HeaderBar"
headerBar.Size = UDim2.new(1, 0, 0, 40)
headerBar.Position = UDim2.new(0, 0, 0, 0)
headerBar.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
headerBar.BorderSizePixel = 0
headerBar.Parent = mainFrame

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 8)
headerCorner.Parent = headerBar

local headerLabel = Instance.new("TextLabel")
headerLabel.Name = "HeaderLabel"
headerLabel.Size = UDim2.new(1, -20, 1, 0)
headerLabel.Position = UDim2.new(0, 10, 0, 0)
headerLabel.BackgroundTransparency = 1
headerLabel.BorderSizePixel = 0
headerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
headerLabel.TextSize = 16
headerLabel.Font = Enum.Font.GothamBold
headerLabel.Text = "SPEED"
headerLabel.TextXAlignment = Enum.TextXAlignment.Left
headerLabel.Parent = headerBar

local speedDisplay = Instance.new("TextLabel")
speedDisplay.Name = "SpeedDisplay"
speedDisplay.Size = UDim2.new(1, -20, 0, 100)
speedDisplay.Position = UDim2.new(0, 10, 0, 50)
speedDisplay.BackgroundTransparency = 1
speedDisplay.BorderSizePixel = 0
speedDisplay.TextColor3 = Color3.fromRGB(255, 255, 255)
speedDisplay.TextSize = 56
speedDisplay.Font = Enum.Font.GothamBold
speedDisplay.Text = "0.0"
speedDisplay.Parent = mainFrame

local barBg = Instance.new("Frame")
barBg.Name = "BarBg"
barBg.Size = UDim2.new(1, -20, 0, 25)
barBg.Position = UDim2.new(0, 10, 0, 160)
barBg.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
barBg.BorderSizePixel = 0
barBg.Parent = mainFrame

local barCorner = Instance.new("UICorner")
barCorner.CornerRadius = UDim.new(0, 4)
barCorner.Parent = barBg

local barFill = Instance.new("Frame")
barFill.Name = "BarFill"
barFill.Size = UDim2.new(0, 0, 1, 0)
barFill.Position = UDim2.new(0, 0, 0, 0)
barFill.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
barFill.BorderSizePixel = 0
barFill.Parent = barBg

local fillCorner = Instance.new("UICorner")
fillCorner.CornerRadius = UDim.new(0, 4)
fillCorner.Parent = barFill

local statsFrame = Instance.new("Frame")
statsFrame.Name = "StatsFrame"
statsFrame.Size = UDim2.new(1, -20, 0, 100)
statsFrame.Position = UDim2.new(0, 10, 0, 200)
statsFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
statsFrame.BorderSizePixel = 0
statsFrame.Parent = mainFrame

local statsCorner = Instance.new("UICorner")
statsCorner.CornerRadius = UDim.new(0, 4)
statsCorner.Parent = statsFrame

local fpsLabel = Instance.new("TextLabel")
fpsLabel.Name = "FPSLabel"
fpsLabel.Size = UDim2.new(1, 0, 0, 20)
fpsLabel.Position = UDim2.new(0, 0, 0, 5)
fpsLabel.BackgroundTransparency = 1
fpsLabel.BorderSizePixel = 0
fpsLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
fpsLabel.TextSize = 12
fpsLabel.Font = Enum.Font.Gotham
fpsLabel.Text = "0"
fpsLabel.Parent = statsFrame

local maxSpeedLabel = Instance.new("TextLabel")
maxSpeedLabel.Name = "MaxSpeedLabel"
maxSpeedLabel.Size = UDim2.new(1, 0, 0, 20)
maxSpeedLabel.Position = UDim2.new(0, 0, 0, 30)
maxSpeedLabel.BackgroundTransparency = 1
maxSpeedLabel.BorderSizePixel = 0
maxSpeedLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
maxSpeedLabel.TextSize = 12
maxSpeedLabel.Font = Enum.Font.Gotham
maxSpeedLabel.Text = "200"
maxSpeedLabel.Parent = statsFrame

local currentLabel = Instance.new("TextLabel")
currentLabel.Name = "CurrentLabel"
currentLabel.Size = UDim2.new(1, 0, 0, 20)
currentLabel.Position = UDim2.new(0, 0, 0, 55)
currentLabel.BackgroundTransparency = 1
currentLabel.BorderSizePixel = 0
currentLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
currentLabel.TextSize = 12
currentLabel.Font = Enum.Font.Gotham
currentLabel.Text = "0.0"
currentLabel.Parent = statsFrame

local frameCounter = 0
local frameTimeBuffer = 0
local lastSecond = tick()
local running = true
local connection
local ancestryConnection
local characterRemovedConnection
local playerRemovedConnection
local lastTime = nil
local EPSILON = 0.001
local CLEANUP_GUARD = false

local function cleanup()
	if CLEANUP_GUARD then
		return
	end
	CLEANUP_GUARD = true
	
	running = false
	
	if connection then
		pcall(function()
			connection:Disconnect()
		end)
		connection = nil
	end
	
	if ancestryConnection then
		pcall(function()
			ancestryConnection:Disconnect()
		end)
		ancestryConnection = nil
	end
	
	if characterRemovedConnection then
		pcall(function()
			characterRemovedConnection:Disconnect()
		end)
		characterRemovedConnection = nil
	end
	
	if playerRemovedConnection then
		pcall(function()
			playerRemovedConnection:Disconnect()
		end)
		playerRemovedConnection = nil
	end
	
	pcall(function()
		if screenGui and screenGui.Parent then
			screenGui:Destroy()
		end
	end)
	
	player = nil
	playerGui = nil
	screenGui = nil
	mainFrame = nil
	speedDisplay = nil
	barBg = nil
	barFill = nil
	fpsLabel = nil
	currentLabel = nil
	maxSpeedLabel = nil
end

function speedUI.update(deltaTime)
	if not running or not screenGui or not screenGui.Parent then
		cleanup()
		return
	end
	
	if speedUI.maxSpeed <= 0 then
		return
	end
	
	local barBgWidth = 1
	if barBg and barBg.Parent then
		local absSize = barBg.AbsoluteSize.X
		if absSize and absSize > 0 then
			barBgWidth = absSize
		else
			barBgWidth = 280
		end
	end
	
	barBgWidth = math.max(barBgWidth, 1)
	
	local isKeyUp = false
	local isKeyDown = false
	
	local success = pcall(function()
		isKeyUp = UserInputService:IsKeyDown(Enum.KeyCode.Up)
		isKeyDown = UserInputService:IsKeyDown(Enum.KeyCode.Down)
	end)
	
	if not success then
		isKeyUp = false
		isKeyDown = false
	end
	
	if isKeyUp then
		speedUI.speed = speedUI.speed + speedUI.acceleration * deltaTime
	elseif isKeyDown then
		speedUI.speed = speedUI.speed - speedUI.acceleration * deltaTime
	else
		speedUI.speed = speedUI.speed - speedUI.friction * deltaTime
	end
	
	speedUI.speed = math.clamp(speedUI.speed, 0, speedUI.maxSpeed)
	
	if math.abs(speedUI.speed) < EPSILON then
		speedUI.speed = 0
	end
	
	if speedUI.speed ~= speedUI.speed then
		speedUI.speed = 0
	end
	
	local ratio = speedUI.speed / speedUI.maxSpeed
	if ratio ~= ratio then
		ratio = 0
	end
	ratio = math.clamp(ratio, 0, 1)
	
	local fillWidth = ratio * barBgWidth
	if fillWidth ~= fillWidth then
		fillWidth = 0
	end
	
	if barFill and barFill.Parent then
		barFill.Size = UDim2.new(0, fillWidth, 1, 0)
		
		if ratio < 0.33 then
			barFill.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
		elseif ratio < 0.66 then
			barFill.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
		else
			barFill.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
		end
	end
	
	if speedDisplay and speedDisplay.Parent then
		speedDisplay.Text = string.format("%.1f", math.max(speedUI.speed, 0))
	end
	
	if currentLabel and currentLabel.Parent then
		currentLabel.Text = string.format("%.1f", math.max(speedUI.speed, 0))
	end
	
	frameTimeBuffer = frameTimeBuffer + deltaTime
	frameCounter = frameCounter + 1
	
	local currentTime = tick()
	if currentTime - lastSecond >= 1 then
		local fpsValue = 0
		if frameTimeBuffer > 0 then
			fpsValue = math.floor(frameCounter / frameTimeBuffer + 0.5)
		end
		fpsValue = math.max(fpsValue, 0)
		
		if fpsLabel and fpsLabel.Parent then
			fpsLabel.Text = tostring(fpsValue)
		end
		
		frameCounter = 0
		frameTimeBuffer = 0
		lastSecond = currentTime
	end
end

characterRemovedConnection = player.CharacterRemoving:Connect(function()
	cleanup()
end)

playerRemovedConnection = game.Players.PlayerRemoving:Connect(function(removedPlayer)
	if removedPlayer == player then
		cleanup()
	end
end)

ancestryConnection = screenGui.AncestryChanged:Connect(function(child, parent)
	if parent == nil then
		cleanup()
	end
end)

connection = RunService.RenderStepped:Connect(function()
	if CLEANUP_GUARD or not running then
		cleanup()
		return
	end
	
	if not screenGui or not screenGui.Parent then
		cleanup()
		return
	end
	
	if not player or not player.Parent then
		cleanup()
		return
	end
	
	if not lastTime then
		lastTime = tick()
		return
	end
	
	local currentTime = tick()
	local deltaTime = currentTime - lastTime
	lastTime = currentTime
	
	if deltaTime <= 0 or deltaTime > 1 or deltaTime ~= deltaTime then
		lastTime = tick()
		return
	end
	
	deltaTime = math.min(deltaTime, 0.5)
	
	local success = pcall(function()
		speedUI.update(deltaTime)
	end)
	
	if not success then
		cleanup()
	end
end)

return speedUI
