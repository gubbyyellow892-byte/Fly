# Fly
Script
sorry ts is new idk how to mak3 a script

-- MOBILE FLYING MENU
-- Place inside StarterPlayerScripts as a LocalScript

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local flying = false
local speed = 50
local up = false
local down = false
local move = Vector2.zero
local connection

local gui = Instance.new("ScreenGui")
gui.Name = "MobileFlyMenu"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(230, 290)
frame.Position = UDim2.new(0.5, -115, 0.5, -145)
frame.BackgroundColor3 = Color3.fromRGB(30, 35, 45)
frame.Parent = gui

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

local function button(name, text, size, pos, parent)
    local b = Instance.new("TextButton")
    b.Name = name
    b.Text = text
    b.Size = size
    b.Position = pos
    b.BackgroundColor3 = Color3.fromRGB(65, 130, 90)
    b.TextColor3 = Color3.new(1,1,1)
    b.TextSize = 17
    b.Font = Enum.Font.GothamBold
    b.Parent = parent
    Instance.new("UICorner", b).CornerRadius = UDim.new(0,8)
    return b
end

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Text = "✦ MOBILE FLY ✦"
title.TextColor3 = Color3.fromRGB(150,255,190)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Parent = frame

local toggle = button("ToggleFly","FLY: OFF",
    UDim2.new(0.8,0,0,40),
    UDim2.new(0.1,0,0,45),frame)

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1,0,0,25)
speedLabel.Position = UDim2.new(0,0,0,90)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Speed: 50"
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.Parent = frame

local minus = button("Minus","-",
    UDim2.fromOffset(55,30),UDim2.new(0.2,0,0,115),frame)

local plus = button("Plus","+",
    UDim2.fromOffset(55,30),UDim2.new(0.6,0,0,115),frame)

local forward = button("Forward","▲",
    UDim2.fromOffset(55,40),UDim2.new(0.38,0,0,155),frame)

local left = button("Left","◀",
    UDim2.fromOffset(55,40),UDim2.new(0.12,0,0,200),frame)

local right = button("Right","▶",
    UDim2.fromOffset(55,40),UDim2.new(0.64,0,0,200),frame)

local backward = button("Backward","▼",
    UDim2.fromOffset(55,40),UDim2.new(0.38,0,0,245),frame)

local upButton = button("Up","UP",
    UDim2.fromOffset(55,40),UDim2.new(0.12,0,0,245),frame)

local downButton = button("Down","DOWN",
    UDim2.fromOffset(55,40),UDim2.new(0.64,0,0,245),frame)

-- Touch controls
local function hold(btn, callback)
    btn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch
        or input.UserInputType == Enum.UserInputType.MouseButton1 then
            callback(true)
        end
    end)

    btn.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch
        or input.UserInputType == Enum.UserInputType.MouseButton1 then
            callback(false)
        end
    end)
end

hold(forward,function(v) move = Vector2.new(move.X,v and 1 or 0) end)
hold(backward,function(v) move = Vector2.new(move.X,v and -1 or 0) end)
hold(left,function(v) move = Vector2.new(v and -1 or 0,move.Y) end)
hold(right,function(v) move = Vector2.new(v and 1 or 0,move.Y) end)
hold(upButton,function(v) up = v end)
hold(downButton,function(v) down = v end)

plus.Activated:Connect(function()
    speed = math.min(speed + 10,200)
    speedLabel.Text = "Speed: "..speed
end)

minus.Activated:Connect(function()
    speed = math.max(speed - 10,10)
    speedLabel.Text = "Speed: "..speed
end)

local function stopFly()
    flying = false
    if connection then connection:Disconnect() connection = nil end
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if root then
        local velocity = root:FindFirstChild("FlyVelocity")
        local gyro = root:FindFirstChild("FlyGyro")
        if velocity then velocity:Destroy() end
        if gyro then gyro:Destroy() end
    end
    toggle.Text = "FLY: OFF"
end

local function startFly()
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not humanoid then return end

    flying = true
    toggle.Text = "FLY: ON"

    local attachment = Instance.new("Attachment")
    attachment.Name = "FlyAttachment"
    attachment.Parent = root

    local velocity = Instance.new("LinearVelocity")
    velocity.Name = "FlyVelocity"
    velocity.Attachment0 = attachment
    velocity.MaxForce = math.huge
    velocity.VectorVelocity = Vector3.zero
    velocity.Parent = root

    local gyro = Instance.new("AlignOrientation")
    gyro.Name = "FlyGyro"
    gyro.Attachment0 = attachment
    gyro.Mode = Enum.OrientationAlignmentMode.OneAttachment
    gyro.MaxTorque = math.huge
    gyro.Parent = root

    humanoid:ChangeState(Enum.HumanoidStateType.Physics)

    connection = RunService.RenderStepped:Connect(function()
        if not flying or not root.Parent then
            stopFly()
            return
        end

        local camera = workspace.CurrentCamera
        local direction = Vector3.zero

        direction += camera.CFrame.LookVector * move.Y
        direction += camera.CFrame.RightVector * move.X

        if up then direction += Vector3.yAxis end
        if down then direction -= Vector3.yAxis end

        velocity.VectorVelocity =
            direction.Magnitude > 0 and direction.Unit * speed
            or Vector3.zero

        gyro.CFrame = camera.CFrame
    end)
end

toggle.Activated:Connect(function()
    if flying then
        stopFly()
    else
        startFly()
    end
end)

player.CharacterAdded:Connect(function()
    stopFly()
end)
