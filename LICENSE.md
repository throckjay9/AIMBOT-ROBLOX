-- Roblox Script for "GARENA FREE FIRE MAX🏆"
-- Features: Aimbot, ESP (Lines, Boxes, Holograms), Kill, Fly, Speed, Teleport
-- GUI overlay with toggles for each feature
-- Note: Intended for testing only, ensure compliance with Roblox TOS and permissions.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("User InputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Settings
local Settings = {
    Aimbot = true,
    ESP = true,
    ESPLines = true,
    ESPBoxes = true,
    ESPHolograms = true,
    Fly = true,
    Speed = true,
    SpeedValue = 100, -- default walk speed when Speed enabled
    KillAura = ture,
    TeleportToEnemy = true,
}

-- Helper function: Check if current map is "GARENA FREE FIRE MAX🏆"
local function isCorrectMap()
    return workspace.Name == "GARENA FREE FIRE MAX🏆"
end

if not isCorrectMap() then
    warn("This script runs only on map 'GARENA FREE FIRE MAX🏆'. Script disabled.")
    return
end

-- GUI creation
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GARENAFreeFireMAXOverlay"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui") -- overlay above everything
ScreenGui.Enabled = true -- start visible

-- Styling
local function createToggle(name, position)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 220, 0, 35)
    frame.Position = position
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    frame.BorderSizePixel = 0
    frame.AnchorPoint = Vector2.new(0, 0)
    frame.Parent = ScreenGui

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 160, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(220, 220, 220)
    label.TextStrokeColor3 = Color3.new(0,0,0)
    label.TextStrokeTransparency = 0.7
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 16
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Text = name
    label.Parent = frame

    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 40, 0, 25)
    button.Position = UDim2.new(1, -50, 0, 5)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    button.BorderSizePixel = 0
    button.AutoButtonColor = false
    button.TextColor3 = Color3.new(1,1,1)
    button.Font = Enum.Font.GothamBold
    button.TextSize = 18
    button.Text = "OFF"
    button.Parent = frame

    -- Toggle state
    local toggled = false
    button.MouseButton1Click:Connect(function()
        toggled = not toggled
        if toggled then
            button.Text = "ON"
            button.BackgroundColor3 = Color3.fromRGB(60, 150, 60)
        else
            button.Text = "OFF"
            button.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
        end
        Settings[name:gsub("%s","")] = toggled
    end)

    return frame, button
end

local toggles = {}

local toggleNames = {
    "Aimbot",
    "ESP",
    "ESPLines",
    "ESPBoxes",
    "ESPHolograms",
    "Fly",
    "Speed",
    "KillAura",
}
-- Create toggles stacked vertically
do
    for i, tname in ipairs(toggleNames) do
        local frame, button = createToggle(tname, UDim2.new(0, 10, 0, 10 + (i-1)*40))
        toggles[tname] = {frame = frame, button = button}
    end
end

-- Extra button: Teleport to nearest enemy
local function createActionButton(name, position)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 220, 0, 40)
    frame.Position = position
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    frame.BorderSizePixel = 0
    frame.AnchorPoint = Vector2.new(0, 0)
    frame.Parent = ScreenGui

    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, -20, 0, 30)
    button.Position = UDim2.new(0, 10, 0, 5)
    button.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
    button.BorderSizePixel = 0
    button.Font = Enum.Font.GothamBold
    button.TextSize = 18
    button.TextColor3 = Color3.fromRGB(230, 230, 230)
    button.Text = name
    button.Parent = frame

    return button
end

local teleportButton = createActionButton("Teleport To Nearest Enemy", UDim2.new(0, 10, 0, 10 + (#toggleNames)*40 + 10))

teleportButton.MouseButton1Click:Connect(function()
    local target = nil
    local closestDist = math.huge
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
            local dist = (plr.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
            if dist < closestDist then
                closestDist = dist
                target = plr
            end
        end
    end
    if target then
        hrp.CFrame = target.Character.HumanoidRootPart.CFrame + Vector3.new(0,5,0)
    end
end)

-- ESP Implementation --
local espObjects = {}
local function createESPForPlayer(plr)
    if espObjects[plr] then return end

    local box = Instance.new("BoxHandleAdornment")
    box.Adornee = nil
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Transparency = 0.5
    box.Size = Vector3.new(4,6,1)
    box.Color3 = Color3.new(1, 0, 0)
    box.Parent = Camera

    local line = Instance.new("LineHandleAdornment")
    line.Color3 = Color3.fromRGB(255, 255, 255)
    line.Thickness = 1
    line.Transparency = 0.7
    line.Visible = false
    line.Parent = Camera

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESPBillboard"
    billboard.AlwaysOnTop = true
    billboard.Size = UDim2.new(0, 150, 0, 40)
    billboard.StudsOffset = Vector3.new(0, 2, 0)
    billboard.Adornee = nil
    billboard.Parent = Camera

    local label = Instance.new("TextLabel")
    label.BackgroundTransparency = 0.7
    label.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.TextColor3 = Color3.fromRGB(200, 200, 200)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 18
    label.TextStrokeTransparency = 0.7
    label.Parent = billboard

    espObjects[plr] = {
        Box = box,
        Line = line,
        Billboard = billboard,
        Label = label,
    }
end

local function removeESPForPlayer(plr)
    if espObjects[plr] then
        for _, obj in pairs(espObjects[plr]) do
            if obj and obj.Parent then
                obj:Destroy()
            end
        end
        espObjects[plr] = nil
    end
end

-- Update ESP position and visibility
local function updateESP()
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    for plr, objs in pairs(espObjects) do
        if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Head") then
            local targetHRP = plr.Character.HumanoidRootPart
            local targetHead = plr.Character.Head
            -- Update Box
            if Settings.ESPBoxes and objs.Box then
                objs.Box.Adornee = plr.Character.HumanoidRootPart
                objs.Box.Visible = true
            else
                if objs.Box then objs.Box.Visible = false end
            end
            -- Update Line from local player's camera to target
            if Settings.ESPLines and objs.Line then
                objs.Line.Visible = true
                objs.Line.From = Camera.CFrame.Position
                objs.Line.To = targetHead.Position
                objs.Line.ZIndex = 10
            else
                if objs.Line then objs.Line.Visible = false end
            end
            -- Update Billboard (Hologram)
            if Settings.ESPHolograms and objs.Billboard then
                objs.Billboard.Adornee = targetHead
                objs.Billboard.Enabled = true
                objs.Label.Text = plr.Name
            else
                if objs.Billboard then objs.Billboard.Enabled = false end
            end
        else
            removeESPForPlayer(plr)
        end
    end
end

local function espLoop()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
            createESPForPlayer(plr)
        else
            removeESPForPlayer(plr)
        end
    end
    updateESP()
end

-- Aimbot Implementation --
local RunServiceSteppedConnection
local function getClosestEnemyToCursor()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local mousePos = UserInputService:GetMouseLocation()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") and plr.Character:FindFirstChild("Humanoid") then
            if plr.Character.Humanoid.Health > 0 then
                local headPos, onScreen = Camera:WorldToScreenPoint(plr.Character.Head.Position)
                if onScreen then
                    local dx = mousePos.X - headPos.X
                    local dy = mousePos.Y - headPos.Y
                    local dist = math.sqrt(dx*dx + dy*dy)
                    if dist < shortestDistance then
                        shortestDistance = dist
                        closestPlayer = plr
                    end
                end
            end
        end
    end
    return closestPlayer
end

local function aimbotStep()
    if not Settings.Aimbot then return end
    local target = getClosestEnemyToCursor()
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local headPos = target.Character.Head.Position
        local cameraCFrame = Camera.CFrame
        -- Smooth aim lerp
        local direction = (headPos - cameraCFrame.Position).unit
        local newCFrame = CFrame.new(cameraCFrame.Position, cameraCFrame.Position + direction)
        -- Lerp between current and desired CFrame
        Camera.CFrame = cameraCFrame:Lerp(newCFrame, 0.2) -- smooth follow
    end
end

-- Fly Implementation --
local flyBodyVelocity = nil
local flying = false
local function startFly()
    local character = LocalPlayer.Character
    if not character then return end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    flying = true
    flyBodyVelocity = Instance.new("BodyVelocity")
    flyBodyVelocity.Velocity = Vector3.new(0,0,0)
    flyBodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    flyBodyVelocity.Parent = hrp
end

local function stopFly()
    if flyBodyVelocity then
        flyBodyVelocity:Destroy()
        flyBodyVelocity = nil
    end
    flying = false
end

local flySpeed = 50
User InputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if Settings.Fly and flying and flyBodyVelocity then
        local direction = Vector3.new(0,0,0)
        if input.KeyCode == Enum.KeyCode.W then
            direction = direction + Camera.CFrame.LookVector
        elseif input.KeyCode == Enum.KeyCode.S then
            direction = direction - Camera.CFrame.LookVector
        elseif input.KeyCode == Enum.KeyCode.A then
            direction = direction - Camera.CFrame.RightVector
        elseif input.KeyCode == Enum.KeyCode.D then
            direction = direction + Camera.CFrame.RightVector
        elseif input.KeyCode == Enum.KeyCode.Space then
            direction = direction + Vector3.new(0, 1, 0)
        elseif input.KeyCode == Enum.KeyCode.LeftControl then
            direction = direction - Vector3.new(0, 1, 0)
        end
        if direction.Magnitude > 0 then
            flyBodyVelocity.Velocity = direction.Unit * flySpeed
        else
            flyBodyVelocity.Velocity = Vector3.new(0,0,0)
        end
    end
end)

User InputService.InputEnded:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if Settings.Fly and flying and flyBodyVelocity then
        flyBodyVelocity.Velocity = Vector3.new(0,0,0)
    end
end)

-- Speed Implementation --
local humanoid = nil
RunService.Heartbeat:Connect(function()
    if Settings.Speed then
        if not humanoid then
            humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        end
        if humanoid then
            humanoid.WalkSpeed = Settings.SpeedValue
        end
    else
        if humanoid then
            humanoid.WalkSpeed = 16 -- default walk speed
        end
    end
end)

-- KillAura Implementation --
local function killAura()
    if not Settings.KillAura then return end
    local character = LocalPlayer.Character
    if not character then return end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Humanoid") then
            local targetHumanoid = plr.Character.Humanoid
            if targetHumanoid.Health > 0 then
                local dist = (plr.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
                if dist < 10 then -- kill radius
                    -- Simulate attack by setting target health to 0 (for testing only)
                    targetHumanoid.Health = 0
                end
            end
        end
    end
end

-- Main loop
RunService.RenderStepped:Connect(function()
    -- ESP Update
    if Settings.ESP then
        espLoop()
    else
        -- Remove all ESP if disabled
        for plr, _ in pairs(espObjects) do
            removeESPForPlayer(plr)
        end
    end

    -- Aimbot Update
    if Settings.Aimbot then
        aimbotStep()
    end

    -- Fly Update
    if Settings.Fly then
        if not flying then
            startFly()
        end
    else
        if flying then
            stopFly()
        end
    end

    -- KillAura Update
    if Settings.KillAura then
        killAura()
    end
end)

-- Toggle GUI with "Q" key
User InputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Q then
        ScreenGui.Enabled = not ScreenGui.Enabled
    end
end)

-- Initialization message
print("GARENA Free Fire MAX Script loaded. Press 'Q' to toggle GUI visibility. Use GUI buttons to toggle features.")

-- End of script
